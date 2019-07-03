# 1. pipeline中的深度数据

**深度**是渲染管线当中的重要概念，它控制着三维世界中物体渲染的遮挡关系。我们以传统的foward render为例，三角形提交draw call以后，顶点进入vertex shader，经过`model-view-projection matrix`变换到**齐次裁剪空间坐标系**（又叫规则观察体(Canonical View Volume)中，通过透视除法（硬件自动完成）进入pixel shader（当然，现代的gpu都会在进入pixel shader之前提供Early-Z kill，后文详述）  

![](https://upload-images.jianshu.io/upload_images/8944607-29e4431b70faf900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/619/format/webp)

z_ealy_z_流程.png

通过pixel shader上色以后，就使用到了本文的主角**深度**，深度测试阶段（Z-test）所有顶点都处在**NDC**坐标系（也就是整个世界规范到xy = {-1， 1}， z = {0， 1}的长方体中），如下图。  

![](https://upload-images.jianshu.io/upload_images/8944607-1919790e1350ddd2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600/format/webp)

ndc_cvv_介绍.jpg

这么折腾一通，对于Z-test的意义就是所有的顶点position.z深度值都规范到{0，1}之间，这也就给比较遮挡关系提供了便利，距离越近的点才能渲染到render texture中，距离太远的话当前像素就可以丢弃了。距离相机越近的点z值越小，假如我们使用如下的pixel shader来表示深度图

```java
fixed4 frag (v2f i) : SV_Target
    {
        float depth = tex2D(_CameraDepthTexture, i.uv).r;
        float4 col = float4(depth, depth, depth, 1);
        return col
    }
```

由于rgb一致，所以深度图都是以灰度图呈现。

---

## 2. UNITY 获取深度图 -- camera的内置depth texture

Camera可以生成depth texture, depth+normals texture，这些内置数据可以用于延迟渲染以及shadow map，本文主要讨论深度图，其他概念暂且摁下不表。

获取Camera内置深度图的介绍的比较多，demo可以参考[这个例子，github需要翻墙](https://link.jianshu.com/?t=https://github.com/giangchau92/Unity-Depth-Buffer-Visualization)，本文也使用这个场景做其他获取方式的介绍。

内置的深度图名字为_CameraDepthTexture，我们在使用它的时候需要遵循如下流程

#### 抓取内置depth texture的获取基本流程如下~

> 1.  获取深度图需要指定Camera， 设置抓取深度图的Camera.depthTextureMode = DepthTextureMode.**Depth**;
> 2.  声明一个rendertexture rt用于保存深度图，rt = RenderTexture.GetTemporary(**width, height, 0**);
> 3.  在Camera的Gameobj身上挂载一个script，重载 **OnRenderImage**方法
> 4.  **OnRenderImage**中 Graphics.Blit(source, dest, DepthRender);
> 5.  保证场景中希望写入深度图的gameobj，它们上色的shader必须有 **FallBack "Diffuse"**代码块，否则深度图不会将他的深度数据写入

流程1说明需要向camera中保存depth tetxure到_CameraDepthTexture中。  
流程2是保存深度图的容器  
当场景绘制完成以后，我们使用blit方法将数据拷贝到dest中去，**请注意！！_CameraDepthTexture与blit（src，dest）中的src，dest位置顺序没有关系，_CameraDepthTexture中一直存在深度数据，_MainTex的数据则只来自src**  
流程5是一个潜规则，原文是这样的[Depth texture is rendered using the same shader passes as used for shadow caster rendering (ShadowCaster pass type). ](https://link.jianshu.com/?t=https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html)。

简单来说内置深度数据来自shadow map计算时使用的一个特殊pass，我们必须设置这个pass才能让内置depth buffer完成数据填充，**FallBack "Diffuse"**在这个时候就起到了作用，貌似自己写的其他pass时不会在获取内置深度时调用。大坑一个，所以自己写的shader结构应该保留**FallBack "Diffuse"**：

```bash
Shader "Custom/yourshader" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
    }
    SubShader 
    {       
        Tags { "RenderType"="Opaque" }

        Pass
        {           
        }
    } 
    FallBack "Diffuse"
}
```

然后我们来看流程4的DepthRender，它的作用就是从_CameraDepthTexture获取depth输出给blit的dest，完整代码如下

```objectivec
Shader "Unlit/depth_render"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "black" {}
    }
    SubShader
    {
        // No culling or depth
        Cull Off ZWrite Off ZTest Always

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            sampler2D _CameraDepthTexture;
            sampler2D _MainTex;

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                float4 col = float4(1, 1, 1, 1);

                // 内置 深度获取
                float depth = tex2D(_CameraDepthTexture, i.uv).r;
                col = float4(depth, depth, depth, 1 );
                return col;
            }
            ENDCG
        }
    }
}
```

其中获取深度还可以使用unity的内置宏SAMPLE_DEPTH_TEXTURE，一般情况下就是取texture的r分量即可  
**\#define SAMPLE_DEPTH_TEXTURE(sampler, uv) (tex2D(sampler, uv).r)**

`tips -- 由于是image effect，我们关闭深度测试和裁剪**Cull Off ZWrite Off ZTest Always**， 关于内置宏，请自行下载unity-build-in-shader查看`

![](https://upload-images.jianshu.io/upload_images/8944607-f85b77610146963a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

深度图2.png

左边是scene view 右边是深度图，我们可以得出这样的结论

### 物体距离摄像机越近，内置深度图的深度值就越大（越接近1 - 白色）

---

## 3. UNITY 获取深度图 -- 创建一个专门的depth camera

上文使用的是unity的内置方法获取深度图，那么我们通过自己的shader将vertex的z值写入rt，当然就可以让一个camera专门渲染深度了！！~~~

渲染深度数据的shader如下

```objectivec
Shader "Unlit/deptVisual"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag           
            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return fixed4(i.vertex.z, i.vertex.z, i.vertex.z, 1);
            }
            ENDCG
        }
    }
}
```

我们看一下渲染结果  

![](https://upload-images.jianshu.io/upload_images/8944607-1fffc3f5249ce069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/791/format/webp)

深度图3.png

但是这个方法带来一个问题，难道我们为了获取深度数据，难道还要让所有的game obj 替换成上述的material吗？答案当然是否定的，这时候就需要介绍一个camera的大杀器了

#### 替换渲染**Camera.renderwithshader()**

> 讲一个例子，假如场景中存在两个camera，renderCamera和depthCamera，场景中其他的gameobj都是默认的材质。那么我们为了不影响renderCamera的正常渲染，depthCamera如何不修改game obj的材质但是又能使用上述的deptVisual-shader进行深度图提取呢？方式就是**RenderWithShader(Shader replaceShader, string replacementTag)**  
> 它需要两个参数，头一个是替换执行的shader，很好理解，第二个是决定如何替换的规则，它一般都是使用"RenderType"。我们从unity doc可以知道每一个subshader都可以设置RenderType
>
> #### Tags { "RenderType"="Opaque" }
>
> 这里rendertype的具体用法就是：replaceShader中所有RenderType的值 = originShader中RenderType的值的，都会进行替换渲染，也就是本来用originShader subshader 渲染的，因为originShader subshader 的RenderType与replaceShader subshader的一致，渲染都改为使用replaceShader subshader。  
> **假如第二个参数 = ""**,则此摄像机本次渲染的所有物体都会使用replaceShader 进行渲染。

replaceShader 中Tags { "RenderType"="Opaque" }，就是所有的不透明物体都是用replaceShader渲染一次。我们使用renderwithshader把所有不透明的物体进行深度提取（半透明物体的深度zwrite off）。

unity的官网里已经有了相关代码，c# 代码如下

```cpp
using UnityEngine;
 using System.Collections;

 [RequireComponent(typeof(Camera))]
 public class DepthRenderer : MonoBehaviour {

     GameObject depthCamera=null;
     Shader replacementShader=null;

     // Use this for initialization
     void Start () 
     {
         depthCamera=new GameObject();
         depthCamera.AddComponent<Camera>();
         depthCamera.camera.enabled=false;
         depthCamera.hideFlags=HideFlags.HideAndDontSave;

         depthCamera.camera.CopyFrom(camera);
         depthCamera.camera.cullingMask=1<<0; // default layer for now
         depthCamera.camera.clearFlags=CameraClearFlags.Depth;

         replacementShader=Shader.Find("RenderDepth");
         if (replacementShader==null)
         {
             Debug.LogError("could not find 'RenderDepth' shader");
         }
     }

     // Update is called once per frame
     void OnPreRender () 
     {
         if (replacementShader!=null)
         {
             Camera camCopy=depthCamera.camera;

             // copy position and location;
             camCopy.transform.position=camera.transform.position;
             camCopy.transform.rotation=camera.transform.rotation;

             camCopy.RenderWithShader(replacementShader, "RenderType");
         }
     }
 }
```

depthCamera 设置为false是为了手动RenderWithShader， 所以渲染深度图的时机就可以自己控制了啊！！！！

`tips -- 因为深度图是灰度图，假如渲染出来的rt全是default color，那么有可能是camera的远近裁剪面太大，导致颜色看不清楚，可以自行修改裁剪面的值进行测试`

---

## 4. UNITY 的Graphics API介绍 & Depth Stencil Test 注意事项

使用unity进行image post effect时，我们会遇到大量的Graphics Api，记录几个常用的

#### Graphics .Blit(Texture source, RenderTexture dest, Material mat)

Blit其实姐可以理解为遍历source所有像素，使用mat的shader进行处理后，拷贝到dest中去。它的实现方法我猜就是正交投影下绘制一个覆盖整个相机的长方体，然后通过已有数据进行处理。  
可以肯定的是  
**Blit sets dest as the render target, sets source _MainTex property on the material, and draws a full-screen quad.**  
_MainTex 使用的数据来自source，写入的位置肯定是dest，当dest = null的时候，写入[帧缓存](https://link.jianshu.com/?t=https://docs.unity3d.com/ScriptReference/Graphics.Blit.html)。  
**当mat存在模板测试代码的时候，参考的模板数据来自dest**，毕竟每个rendertexture都可能同时存在colorbuffer & depthbuffer

#### Graphics.Blit(Texture source, Material mat）

这个函数还有一个重要的重载的形式，如上  
它的写入目标是**Graphics.activeColorBuffer**。depth & stencil buffer使用的是**Graphics.activeDepthBuffer**。  
因此，我们使用这个方法的时候一定要设置好渲染目标

#### Graphics.SetRenderTarget(rt1.colorBuffer, rt2.depthBuffer);

这个api控制接下来的blit渲染中color 以及 depth & stencil buffer使用不同rt的数据，写入color的目标是rt1，测试使用的depth buffer的数据来源是rt2（**当然zwrite 打开我认为rt2深度数据也会修改，待测**）  
_这个api感觉android不能用，也就是不能分开设置rendertarget_

> 讲一个关于模板测试的例子  
> // source 中模板已经有数据了，现在希望对模板中特殊值的像素进行颜色xxx的后处理，那么就是  
> OnRenderImage(RenderTexture source, RenderTexture destination)  
> Graphics.Blit(source, TempBuffer);  
> Graphics.Blit(TempBuffer, source，PostRender); // 假设postrender会进行模板测试  
> 原因就是blit中_MainTex使用的是TempBuffer，然而深度测试 & 模板测试数据使用的是source，先把TempBuffer的颜色拷贝过来  
> 一般的blit后处理深度测试都关了，我猜测depth 打开的话使用的也是dest的深度buffer，没有印证过