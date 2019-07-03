> 这个作者大佬有很多关于shader的好文章可以看这里：
> [大佬shader案例](https://blog.csdn.net/zhangxiao13627093203/article/category/6317037)

一、介绍

    本篇要介绍的效果如图所示：角色在地形上行走，地形上始终会有一个动态圆环跟随角色，类似于阴影的一个效果。但是这个


“阴影”是我们可以自己控制它的效果的，如图2所示是通过修改后得到的效果图：圆环是有一定频率别的收缩，这种效果在游戏

中我们经常能看到。
二、实现
1，简单圆环版本
实现一的效果的Shader为
```
Shader "Custom/RadiusShader" {
	Properties {
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}
		
		_Center("Center", Vector) = (0,0,0,0)
		_Radius("Radius", Float) = 0.5
		_RadiusColor("Radius Color", Color) = (1,0,0,1)
		_RadiusWidth("Radius Width", Float) = 2
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		#pragma surface surf Standard fullforwardshadows
 
		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0
 
		sampler2D _MainTex;
		fixed4 _Color;
 
		float3 _Center;
		float _Radius;
		fixed4 _RadiusColor;
		float _RadiusWidth;
 
		struct Input {
			float2 uv_MainTex;
			float3 worldPos;
		};
 
		void surf(Input IN, inout SurfaceOutputStandard o) {
			float d = distance(_Center, IN.worldPos);
			if (d > _Radius && d < _Radius + _RadiusWidth)
				o.Albedo = _RadiusColor;
			else
				o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
		}
		ENDCG
	} 
	FallBack "Diffuse"
}
```
主要的思路其实就是在颜色输出的时候判断当前觉得位置一定半径距离范围，然后将这个范围内或者边上的颜色处理成我们预设的，代码中_Center 就是角色的世界坐标系的位置，这个位置通过C#代码在Update里给这个Shader传值，并且将半径和宽度值也
float d = distance(_Center, IN.worldPos);
			if (d > _Radius && d < _Radius + _RadiusWidth)
				o.Albedo = _RadiusColor;
			else
				o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
		}
一起传给Shader，代码如：
using UnityEngine;
 
[ExecuteInEditMode]
public class Radius : MonoBehaviour {
 
    public Material radiusMaterial;
    public float radius = 1;
    public Color color = Color.white;
 
    // Use this for initialization
    void Start () {
	
	}
	
    //每帧更新shader中的参数值
	// Update is called once per frame
	void Update () {
        radiusMaterial.SetVector("_Center", transform.position);
        radiusMaterial.SetFloat("_Radius", radius);
        radiusMaterial.SetColor("_RadiusColor", color);
    }
}
这种效果比较简单，有了这个基础我们再拓展就比较方便了，我自己实验了介绍里的第二种效果。
2、周期频率变化版本
其实核心的部分是类似的，在这无非就是对圆环的半径宽度做了一个随着时间的正弦函数的变化，并且将预设的颜色从圆心出做了
	float4 frag(vertexOutput input) : COLOR
			{
				float d = distance(_Center,input.worldPos.xyz);
				float tempRadiusWidth = _RadiusWidth*abs(sin(_Time.y*_RadiusSpeed));
 
				float4 texCol = tex2D(_MainTex, input.tex.xy);
				float4 col = texCol;
				if (d > _Radius&&d < _Radius + tempRadiusWidth)
				{
					col.rgb = texCol.rgb + _RadiusColor*pow(d / (_Radius + tempRadiusWidth), _RadiusPower);
				}
				else
				{
					col.rgb = texCol.rgb;
				}
				
				return col;
			}
渐变的处理，这样得到的效果会更加融合自然，最后在加上关照计算的处理就可以在实际的应用中使用了。完整的Shader代码如：
// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
 
// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
 
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
 
Shader "Unlit/RPGGroundRender"
{
	Properties{
		_MainTex("Albedo (RGB)", 2D) = "white" {}
	_Center("Center", Vector) = (0,0,0,0)
		_Radius("Radius",Float) = 0.5
		_RadiusColor("Radius Color",Color) = (1,0,0,1)
		_RadiusWidth("Radius Width",Float) = 2
		_RadiusPower("RadiusPower",Range(0,10)) = 2
		_RadiusSpeed("RadiusSpeed",Range(0,10)) = 50
	}
		SubShader
	{
		Tags { "RenderType" = "Opaque" "PreviewType"="Plane" }
		LOD 200
 
		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
 
			#pragma target 3.0
 
			#include "UnityCG.cginc"
 
			//光源的颜色，Unity的Lighting.cginc文件中包含这个
			float4 _LightColor0;
 
			sampler2D _MainTex;
			float3 _Center;
			float _Radius;
			fixed4 _RadiusColor;
			float _RadiusWidth;
			float _RadiusPower;
			float _RadiusSpeed;
 
			struct vertexInput {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
				float4 tex:TEXCOORD0;
			};
			struct vertexOutput {
				float4 pos:SV_POSITION;
				float4 worldPos : TEXCOORD1;
				float4 col : COLOR;
				float4 tex:TEXCOORD0;
			};
 
			//顶点程序部分增加了计算光照的部分
			vertexOutput vert(vertexInput input)
			{
				vertexOutput output;
 
				float4x4 modelMatrix = unity_ObjectToWorld;
				float4x4 modelMatrixInverse = unity_WorldToObject;
 
				float3 normalDirection = normalize(
					mul(float4(input.normal, 0.0), modelMatrixInverse).xyz);
				float3 lightDirection = normalize(_WorldSpaceLightPos0.xyz);
 
				float3 diffuseReflection = _LightColor0.rgb * max(0.0, dot(normalDirection, lightDirection));
 
				output.col = float4(diffuseReflection, 1.0);
				output.pos= UnityObjectToClipPos(input.vertex);
				output.worldPos = mul(unity_ObjectToWorld, input.vertex);
				output.tex = input.tex;
				return output;
			}
 
			float4 frag(vertexOutput input) : COLOR
			{
				float d = distance(_Center,input.worldPos.xyz);
				float tempRadiusWidth = _RadiusWidth*abs(sin(_Time.y*_RadiusSpeed));
 
				float4 texCol = tex2D(_MainTex, input.tex.xy);
				float4 col = texCol;
				if (d > _Radius&&d < _Radius + tempRadiusWidth)
				{
					col.rgb = texCol.rgb + _RadiusColor*pow(d / (_Radius + tempRadiusWidth), _RadiusPower);
				}
				else
				{
					col.rgb = texCol.rgb;
				}
				
				return col;
			}
			ENDCG
		}
	}
}

给Shader实时传数据的C#控制脚本代码：
using UnityEngine;
using System.Collections;
 
[ExecuteInEditMode]
public class RPGRenderControl : MonoBehaviour {
 
   
    public Material radiusMaterial;
    public Color color = Color.white;
    public float radius = 10f;
    public float radiusWidth = 2f;
    public float power = 5f;
    public float speed = 60f;
    // Use this for initialization
    void Start () {
	
	}
	
    
	// Update is called once per frame
    //保持渲染位置时刻都跟踪这个对象
	void Update () {
 
        radiusMaterial.SetVector("_Center", transform.position);
        radiusMaterial.SetColor("_RadiusColor", color);
        radiusMaterial.SetFloat("_Radius", radius);
        radiusMaterial.SetFloat("_RadiusWidth", radiusWidth);
        radiusMaterial.SetFloat("_RadiusPower", power);
        radiusMaterial.SetFloat("_RadiusSpeed", speed);
 
    }
}
三、总结
完整的工程在下面的这个链接处点击打开链接下载，原谅我可耻的骗取积分，手动捂脸