# Unity shader查询手册——选项设置

## LOD
``` glsl
1:LOD Level of Detail, 根据LOD来设置使用不同版本的Shader;
2:着色器中给SubShader一个LOD值，程序来设置这个shader的LOD值，只有第一个小于等于LOD值subShader才会被执行;
3: 每个shader最多只会有一个SubShader被使用;
4: 通过Shader maximumLOD来设置最大的LOD值;
5: 设置全局的LOD值，Shader.globalMaximumLOD;
6: Unity内置着色器分LOD等级:
    (1)VertexLit kind of shaders 100
     (2) Decal, Reflective VertexLit 150
     (3)Diffuse 200
     (4)Difuse Detail  250
     (5) Bumped, Specular   300
     (6) BumpedSpecular  400
     (7) Parallax   500
     (8) Parallax Specular 600
```

## 渲染队列
>渲染队列的数值决定了Unity在渲染场景物体时的先后顺序,关闭深度测试的情况下;
``` glsl
1:渲染队列标签可选值:
      (1)Background 背景,对应的值为1000;
      (2) Geometry(default) 几何体对应的值为2000, 这个队列是默认的渲染队列,大多数不透明的物体;
     (3)AlphaTest Alpha测试,对应值为2450, alpha测试的几何体使用这种队列,它是独立于 Geometry的队列,它可以在所有固体对象绘制后更有效的渲染采用Alpha测试的对象;
     (4)Transparent:透明，对应值3000, 这个渲染队列在Geometry被渲染，采用从后向前的次序;
    任何有alpha混合的对象都在这个队列里面渲染;
     (5) Overlay 覆盖对应值为4000, 这个渲染队列是最后渲染的物体;
2: Unity 渲染模式: 普通物体从前向后, Alpha从后向前;
```

## 混合模式
```glsl
1:在所有计算完成后，决定当前的计算结果输出到帧缓冲区时，如何混合源和目标,通常用来绘制半透明的物体;
2: Blend Off 关闭混合
3: Blend 源因子，目标因子: 配置并开启混合，产生的颜色和因子相乘，然后两个颜色相加
4: Blend 源因子,目标因子, 源因子A，目标因子A： 源因子与目标因子用户混合颜色值，源因子A，与目标因子A，用于混合alpha
5: BlendOp操作命令: 不是将颜色混合在一起，而是对他们进行操作，主要有:
Min, Max, Sub, RevSub
6:混合因子的类型：
One 使用源或目标色完全显示出来;                 OneMinusSrcColor 阶段值 * (1-源颜色的值)
Zero 删除源颜色或目标颜色;                             OneMinusSrcAlpha 阶段值 * (1-源颜色的Alpha值)
SrcColor 这个阶段的值*源颜色值;                    OneMinusDstColor 阶段值 * (1-目标颜色的值);
DstColor 这个阶段的值* 帧缓冲颜色值;         OneMinusDstAlha 阶段值 * (1-目标颜色Alpha值)
DstAlpha这个阶段的值 * 帧缓冲源Alpha值   SrcAlpha这个阶段的值 * 源颜色Alpha值
6:一般放在放在Pass通道里面
```

## Alpha测试
```
1:Alpha测试: 阻止片元被写到屏幕的最后机会, 最终渲染出来的颜色计算出来后可通过透明度和最后一个固定值比较，如果通过测试则绘制次片元，否则丢弃此片元;
2: AlphaTest Off/On: 开启/关闭Alpha测试,默认是关闭的;
3:比较测试值的模式:
    Greater >,  GEqual >=, Less <, LEqual <=, Equal ==, NotEqual !=, 
   Always (永远渲染), Never(从不渲染);
4: AlphaTest 条件 [变量] / 常数,  一般放在放在Pass通道里面;
```
## 深度测试
```
1:为了使近距离的物体挡住远距离的物体，当片元写入到缓冲的时候，需要将片元的深度值与缓冲区的深度值进行比较，测试成功写入帧缓冲区;
2: ZWrite  深度写开关, 控制是否将深度Z的片元写入缓冲区中，如果不绘制透明物体设置为On, 否则的话设置为Off，默认为On;
3: ZTest 深度测试模式: 设置深度测试的执行方式，默认为LEqual,深度测试的模式:
    Less <, Greater >, LEqual <= , GEqual >=, Equal ==, NotEqual !=, Always 总是绘制,关闭深度测试;
4: ZTest 条件 
5: 一般放在放在Pass通道里面;
```
## 通道遮罩
```
1:通道遮罩可以是开发人员指定渲染结果输出的通道，而不是通常情况下的RGBA四个通道;
2:可选的是RGBA的任意组合以及0，如果为0意味着不会写入到任何通道;
3: ColorMask RG ... ColorMask 0
```
## 面剔除
```
1:通过不渲染背对摄像机的几何体的面来提高性能优化错误，所有的几何体都包含正面和反面
2: 面剔除操作，大多数都是封闭的物体，所以不需要绘制背面;
3: 面剔除操作: 
       Cull Back: 不绘制背对摄像机的面，默认项
       Cull Front,  不绘制面向摄像机的面,
       Cull Off, 关闭面剔除操作
```

## GrabPass
```
1:使用抓屏通道, GrabPass {} 或 GrabPass { “ 纹理名称”}; _GrabTexture访问
2: 后续的Pass通道使用这个抓屏;
3: 编写案例
  (1): 创建一个顶点片元着色器;
  (2): 将这个着色器放到Overlay队列
  (3): 使用GrabPass通道截屏，并定义好变量来接收
  (3): 设置顶点的UV坐标;
    (4): 着色使用截图的纹理
```

## 常用的gcinc
```
1:cginc文件: 宏，帮助函数等，放在CGIncludes下面，开发人员可以开发自己的cginclude文件
2:常用的cginc文件: 
   HLSL.Support.cginc 协助多平台开发的一些宏等，自动包含
   UnityShaderVarirables.cginc 全局变量，自动包含；
   UnityCG.cginc 常用的帮助函数;
  AutoLight.cginc 光照和阴影功能；
  Lighting.cginc 表面着色器的光照模型;
  TerrainEngine.cginc 地形植被的光照着色函数;
```
## UnityCG.gcinc常用函数
```
1:UnityWorldSpaceViewDir: 给定对象空间的顶点位置朝向摄像机方向的世界坐标空间方向;
2: ObjSpaceViewDir: 给定对象空间的顶点位置朝向摄像机方向的对象空间方向;
3: ParallaxOffset: 计算用于视差法线贴图的UV偏移量;
4: Luminance: 将颜色转为亮度;
5: DecodeLightmap: 从光照贴图中解码颜色;
6: float EncodeFloatRGBA(float4 rgba): 将RGBA颜色编码为[0,1)的浮点数；
7: float4 DecodeFloatRGBA(float v): 将一个浮点数解码为RGBA的颜色;
8: UnityWorldSpaceLightDir 给定对象空间的顶点位置到光源的世界坐标空间方向;
9: ObjSpaceLightDir: 给定对象空间的顶点位置到光源的对象空间方向;
```
## Pass复用
```
1:编写过的pass可以重复使用,借助UsePass “ShaderPath/PASS_NAME”
2:PASS名字要大写;
3: Pass {
    name  “ONE” 
}
4: UsePass “Custom/ShaderName/ONE”
```
## 多平台编译
```glsl
1: 通过multi_compile编译多个版本的shader;
2: #pragma multi_compile MY_multi_1  MY_multi_2;
3: #ifdef MY_multi_1 #endif
4: Shader.EnableKeyword(“ MY_multi_1”);
5: Shader.DisableKeyword(“MY_multi_2”); 控制shader编译出不同的版本;
```
## 移动平台优化
```
1:  代码优化: 
    预先计算好对应的值 sqrt(2) --> 根号2 --> 1.414..;
    放心的使用向量相关操作，叉积,点击,基本都是硬件实现，很高效; 
    尽量减少函数调用减少开销;
2: 尽可能的计算放在顶点着色器中，顶点着色器的调用频率远低于片着色器；
3: 几何复杂度考量：在IOS平台视口内的顶点数不要超过100K个，IOS默认的缓冲区就是就是这么大，超过这个数字，底层会做一些操作消耗更多的资源；
4: 纹理大小为 2^n次方大小, 16, 64, 128, 256, 512, 1024;
5: 使用适当的数据类型float < half < fixed; 性能
6: 尽量慎用透明效果,透明效果GPU要逐像素渲染;
```