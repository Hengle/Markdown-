
# Shader各种空间变换
> 前几天尝试写一个传送门的shader，发现自己对坐标之间的变换掌握的不够熟练，趁着这阵子想整理shader相关的知识点，先把各种空间及之间转换整理一下。

## **1 模型空间-世界空间-观察空间-裁剪空间**

建模时在模型空间进行，模型自带的坐标均为模型空间下的表示。   
当模型被放到世界坐标系中时，表达某个模型的位置使用的是世界空间下的坐标，所以模型上对应的某一个点，必须相应的转化为世界空间下的坐标。   
**从模型空间到世界空间的变换** 叫做 **模型变换**。   
Unity的Shader中，模型空间的坐标由Renderer直接提供，作为顶点着色器的输入，语义为POSITION。如：

```language-c
struct appdata
{
    float4 vertex : POSITION;
}
```

由于我们能看到的渲染图像均是通过摄像机得到的，为了方便后续裁剪、投影等操作，所以在将模型从模型空间变换到世界空间之后，还需要将其转换到观察空间。所谓的观察空间，就是以摄像机位置为原点，摄像机局部坐标轴为坐标轴的坐标系。   
**从世界空间到观察空间的变换**叫做**观察变换**（视图变换）。

坐标转换到观察空间后，由于直接使用摄像机的平截头体进行裁剪比较复杂（平截头体的边界方程求交困难），所以需要将其转化到裁剪空间。裁剪空间变换的思路是，对平截头体进行缩放，使近裁剪面和远裁剪面变成正方形，使坐标的w分量表示裁剪范围，此时，只需要简单的比较x,y,z和w分量的大小即可。   
**从观察空间到裁剪空间的变换**叫做**投影变换**。   
注意，虽然叫做投影变换，但是投影变换并没有进行真正的投影。

在顶点着色器中，模型、观察、裁剪空间的相关变换矩阵一般为以下几个：

别名               | 定义                                       | 含义      
---------------- | ---------------------------------------- | --------
UNITY_MATRIX_M   | unity_ObjectToWorld                      | 模型变换矩阵  
UNITY_MATRIX_V   | unity_MatrixV                            | 视图变换矩阵  
UNITY_MATRIX_P   | glstate_matrix_projection                | 投影变换矩阵  
UNITY_MATRIX_VP  | unity_MatrixVP                           | 视图投影变换矩阵
UNITY_MATRIX_MV  | mul(unity_MatrixV, unity_ObjectToWorld)  | 模型视图变换  
UNITY_MATRIX_MVP | mul(unity_MatrixVP, unity_ObjectToWorld) | 模型视图投影变换

注意，最新版本中（笔者用的是Unity5.6.3p4），官方建议将坐标点从模型空间转换到裁剪空间时，应使用UnityObjectToClipPos方法，该方法内部定义为：

```language-c
mul(UNITY_MATRIX_VP, mul(unity_ObjectToWorld, float4(pos, 1.0));
```

* 1

上述方法比起下面的顺序更有效率。

```language-c
mul(UNITY_MATRIX_MVP,  appdata.vertex);
```

* 1

在定点着色器中，输出即为裁剪空间下的坐标。

## **2 屏幕空间**

经过裁剪操作之后，我们需要将裁剪空间的坐标投影到屏幕空间。   
这里主要分成两个步骤：   
**1. 齐次除法**   
使用xyz分别处以w分量，得到NDC（归一化设备坐标），经过这一步，能够看到的坐标点变成了一个xy边长为1，z为-1到1的立方体。   
**2. 屏幕映射**    
使用NDC坐标和屏幕长宽像素直接映射，得到屏幕空间下的xy坐标。注意，虽然屏幕空间没有深度，但屏幕空间下的坐标仍然保留了z的深度值，可以进行深度检测或其他处理。

从裁剪空间到屏幕空间由unity直接进行。这里还要记住，从裁剪空间到屏幕空间，插值是硬件直接进行的。

之后我们从像素着色器中获得的语义为SV_POSITION的输入，其坐标基本没有太多用处了。   
如果希望获得此时的屏幕坐标，可以使用VPOS或者WPOS，在Unity中，这两者同义。当然，使用上述语义需要 

```language-c
#pragma target 3.0
```


VPOS中的xy代表屏幕空间中的像素坐标，注意，这里的像素坐标是中心点坐标，不是整数，如屏幕分辨率为400*300，则x的范围为[0.5, 400.5]，y的范围为[0.5, 300.5]。z的范围为[0, 1]，0是近裁剪面，1是远裁剪面。w的范围需要根据摄像机投影类型划分，如果是透视投影，则其范围为[1/near，1/far]，如果是正交投影，则恒为1。   
这里还需要注意的是，如果使用VPOS或者WPOS作为fs的输入，就无法同时使用SV_POSITION作为输入，所以vs和fs需要写成如下方式：

```language-c
Shader "Unlit/Screen Position"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 3.0

            // note: no SV_POSITION in this struct
            struct v2f {
                float2 uv : TEXCOORD0;
            };

            v2f vert (
                float4 vertex : POSITION, // vertex position input
                float2 uv : TEXCOORD0, // texture coordinate input
                out float4 outpos : SV_POSITION // clip space position output
                )
            {
                v2f o;
                o.uv = uv;
                outpos = UnityObjectToClipPos(vertex);
                return o;
            }

            sampler2D _MainTex;

            fixed4 frag (v2f i, UNITY_VPOS_TYPE screenPos : VPOS) : SV_Target
            {
                // screenPos.xy will contain pixel integer coordinates.
                // use them to implement a checkerboard pattern that skips rendering
                // 4x4 blocks of pixels

                // checker value will be negative for 4x4 blocks of pixels
                // in a checkerboard pattern
                screenPos.xy = floor(screenPos.xy * 0.25) * 0.5;
                float checker = -frac(screenPos.r + screenPos.g);

                // clip HLSL instruction stops rendering a pixel if value is negative
                clip(checker);

                // for pixels that were kept, read the texture and output it
                fixed4 c = tex2D (_MainTex, i.uv);
                return c;
            }
            ENDCG
        }
    }
}
```

![这里写图片描述](https://docs.unity3d.com/uploads/SL/SemanticsScreenPosition.png)

摄像机和屏幕相关常用的内置变量如下表所示。

别名                             | 类型       | 含义
------------------------------ | -------- | --
_WorldSpaceCameraPos           | float3   |   
_ProjectionParams              | float4   |   
_ScreenParams                  | float4   |   
_ZBufferParams                 | float4   |   
unity_OrthoParams              | float4   |   
unity_CameraProjection         | float4x4 |   
unity_CameraInvProjection      | float4x4 |   
unity_CameraWroldClipPlanes[6] | float4   |   

## **3 视口空间**

熟悉OpenGL编程的同学都知道gl接口有一个glViewport，该接口其实就是确定视口空间。视口空间是一个将屏幕坐标对应到(0, 0)到(1, 1)范围内的空间。   
如果想得到视口空间坐标，则可以通过下面两种方法。

```language-c
fixed4 frag(float4 sp : VPOS) : SV_Target {
    return fixed(sp.xy/_ScreenParams.xy, 0.0, 1.0);
}
```

* 1
* 2
* 3

这里的_ScreenParams中保存了屏幕分辨率。   
另一种方法如下：

```language-c
struct vertOut {
    float4 pos : SV_POSITION;
    float4 srcPos : TEXCOORD0;
};

vertOut vert(appdata_base v) {
    vertOut o;
    o.pos = UnityObjectToClipPos(v.vertex);
    o.srcPos = ComputeScreenPos(o.pos);
    return o;
}

fixed4 frag(vertOut i) : SV_Target {
    float2 wcoord = (i.srcPos.xy/i.scrPos.w);
    return fixed4(wcoord, 0.0, 1.0);
}
```
这里，ComputeScreenPos并没有直接进行透视除法，原因是插值是线性的，必须在透视除法之后进行，所以，我们必须在fs中手动进行。

几篇不错的参考资料如下：   
http://blog.csdn.net/wangdingqiaoit/article/details/51594408   
http://blog.csdn.net/wangdingqiaoit/article/details/51589825