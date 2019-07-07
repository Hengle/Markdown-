# Unity shader查询手册——表面着色器
>1:表面着色器包括4个函数:
   (1): 顶点变换函数;
   (2): 表面着色函数;
   (3): 光照模型;
   (4): 最终颜色修改函数;
2: 表面着色器最终会被编译为一个复杂的顶点着色程序;

## 定义函数入口
```glsl
1:#pragma surface 入口函数名称 光照模型  [Options] 
2: suface 后面跟 表面着色的入口函数  surf(Input IN, inout SurfaceOutput o);
3: 光照模型： 
     (1)系统内置 Lambert(漫反射光照) BlinnPhong (高光光照)
     (2)自定义光照: 名字为Name 
           half4 Lighting<Name>(SurfaceOutput s, half3 lightDir, half atten);
          half4 Lighting<Name>(SurfaceOutput s, half3 lightDir, half3 viewDir, half atten);
          half4 Lighting<Name>(SurfaceOutput s, half4 light);
可选参数:
4: vertex: name vertex入口函数: 
void <Name> (inout appdata_full v) 只需改顶点着色器中的输入顶点数据;
half4 <Name>(inout appdata_full v, out Input o) 修改输入顶点数据,以及为表面着色器传递数据;
5: finalcolor: name 最终颜色修改函数:
    void <Name>(Input IN, SurfaceOutput o, inout fixed4 color);
```

## 其它可选参数
```
1:alpha: Alpha 混合模式，用户半透明着色器;
2: alphatest: varirableName Alpha测试模式，用户透明镂空着色器。
3: exclude_path:prepass 使用指定的渲染路径;
4: addshadow: 添加阴影投射器和集合通道;
5: dualforward: 将双重光照贴图用于正向渲染路径中;
6: fullforwardshadows 在正向渲染路径中支持的所有的阴影类型;
7: decal: add 附加印花着色器;
8: decal: blend 附加半透明印花着色器;
9: softvegetation 使用表面着色器，仅在Soft Vegetation 开启时被渲染;
10: noambient 不使用任何光照
11: novertexlights 在正向渲染中不适用球面调和光照或逐点光照;
12: nolightmap 在这个着色器上禁用光照贴图;
13: nodirlightmap 在这个着色器上禁用方向光照贴图;
14: noforwardadd 禁用正向渲染添加通道;
15: approxview: 对于有需要的着色器，逐顶点而不是逐像素计算规范化视线方向。
16: halfasview:  将半方向传递到光照函数中。
```
## Input 结构附加数据
```
Input:包含着色所需要的纹理坐标　uv纹理名字;使用第二张纹理是uv2纹理名字;
附加数据:
1:float3 viewDir  视图方向。
2:float4 color 每个顶点的颜色插值
3: float4 screenPos 屏幕空间中的位置。
4: float3 worldPos 世界坐标空间;   
5: float3 worldRef1 世界空间中的反射向量;
6: float3 worldNormal 世界空间中的法线向量;
7: float3 worldRef1; INTERNAL_DATA 世界坐标反射向量, 但必须表面着色写入o.Normal参数;
8: float3 worldNormal; INTERNAL_DATA 世界坐标法线向量, 但必须表面着色写入o.Normal参数;
```

## SurfaceOutput 结构体
```glsl
SurfaceOutput：
1: half3 Albedo: 漫反射的颜色值;
2: half3 Normal: 法线坐标;
3: half3 Emission; 自发光颜色;
4: half Specular;  镜面反射系数;
5: half Gloss; 光泽系数;
6: half Alpha; 透明度系数;
SurfaceOutputStandard：
7: half Smoothness;    // 0=粗糙, 1=光滑
8: half Metallic;    // 0=非金属, 1=金属
SurfaceOutputStandardSpecular:
    fixed3 Albedo;     
    fixed3 Specular;    
    fixed3 Normal;     
   half3 Emission;  
    half Smoothness;    // 0=粗糙, 1=光滑  
    half Occlusion;  // 遮挡(默认1)  
    fixed Alpha;  
```
