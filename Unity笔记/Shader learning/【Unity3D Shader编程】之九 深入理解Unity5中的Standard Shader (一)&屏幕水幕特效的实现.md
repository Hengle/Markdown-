> 本文工程使用的Unity3D版本： 5.2.1   
>
>
>
>
>
>
> 概要：本文主要介绍了Unity5中的标准着色器，并且也涉及到了基于物理的着色、延迟渲染等高级着色技术，而在文章后半部分，也对屏幕水幕特效的实现方法进行了讲解与分析。
>
>
>
>
>
> 依然是附上一组本文配套工程的运行截图之后，便开始我们的正文。如下图。
>
>
>
> 打开水幕特效的效果图：
>
>
>
> ![](https://img-blog.csdn.net/20151101111232183?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
>
>
> 原始的城镇场景：
>
> ![](https://img-blog.csdn.net/20151101111246079?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
>
>
>
>
> 需要说明，这里的水幕特效是动态的水流效果。本来准备传GIF上来展示动态的效果，但受图片尺寸2M的限制，无法出色地表现出动态效果（帧数有限，图片清晰度也会大打折扣）。建议感兴趣的朋友们下载这里提供的游戏场景的exe，自己在机器上体验水幕效果，比看博文中贴出的图片效果好太多。
>
>
>
> [【可运行的本文配套exe游戏场景请点击这里下载】](http://yun.baidu.com/s/1nt7w5aL)
>
>
>
> 提示：在此游戏场景中按F键可以开关屏幕特效。
>
>
>
> 在文章末尾有更多的运行截图，并提供了源工程的下载。
>
>
>
> 好的，正文开始。
>
>
>
>
>
>
>
>
>
>
>
> # 一、认识Unity5中的Standard Shader
>
>
>
>
>
>
>
> Unity5中重点推出了一套基于物理的着色（Physically Based Shading，PBS）的多功能Shader，叫做标准着色器（Standard Shader）。这套Shader的设计初衷是化繁为简。想用这样的一个多功能Shader，来代替之前多种多样的Shader各司其职，对于不同的材质效果，需要不同的Shader的局面。
>
> Unity 5中目前有两个标准着色器，一个名为Standard，我们称它为标准着色器的标准版，另一个名为Standard（Specular Setup），我们称它为标准着色器的高光版，它们共同组成了一个完整的PBS光照明模型，且非常易于使用。其实这两个Shader基本差不多，只是有细微的属性参数上的区别。标准版这边的_Metallic（金属性）、_MetallicGlossMap（金属光泽贴图），被高光版的_SpecColor（高光颜色）、_SpecGlossMap（高光颜色法线贴图）所代替。
>
> 标准着色器主要是针对硬质表面（也就是建筑材质）而设计的，可以处理大多数现实世界的材质，例如石头、陶瓷、铜器、银器或橡胶等。同时，它也可以非常出色地处理一些非硬质表面的材质，例如皮肤、头发或布料等。
>
>
>
> 如下的一个Unity官方放出的名为VikingVillage的场景中，所有的物体材质都是使用的标准着色器，标准着色器的广泛适用性可见一斑。
>
>
>
> ![](https://img-blog.csdn.net/20151101111515884?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
>
>
> 而在Unity5的官方放出的演示demo视频中，也是用一个标准着色器，Standard（Specular Setup），就完成了游戏场景中绝大多数物体的材质显示。
>
> Unity5 Standard Shader的演示Demo可以看这里：
>
> http://www.iqiyi.com/w_19rquxac31.html
>
>
>
> Unity5 PBS技术的演示Demo截图：
>
> ![](https://img-blog.csdn.net/20151101111548685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
> 下面稍微对Standard shader中核心的概念——基于物理的着色做一个大概的了解。
>
>
>
>
>
>
>
>
>
> ## 1.1 基于物理的着色（Physically Based Shading）技术概览
>
>
>
>
>
>
>
> 基于物理的着色（Physically Based Shading，简称PBS）就是以某种方式模拟现实中材质和光照的相互作用的一种着色方法。这种方法在需要光照和材质更加直观和逼真地协同工作的场合下优势非常明显。基于物理的着色模拟了光线在现实中的行为，实现了在不同的光照条件下的逼真效果。为实现这种效果需要遵循各种物理原理，包括能量守恒（也就是物体反射出去的光量不可能超过所接收的光量），Fresnel反射（所有表面反射在掠射角（grazing angles）处更加强烈），以及物体表面是如何自我遮挡等原理。
>
> 其实，PBS技术在多年前于虚幻三游戏引擎中就已经被广泛使用，而Unity直到最近刚发布的第五代，才将此特性作为官方的特性实现出来。而在Unity5发布之前，Unity的Asset Store中，已经有不少的第三方插件，在Unity中实现了PBS技术。
>
>
>
> 关于PBS技术，知乎上有一个专栏，专门介绍PBS技术的一些相关原理，在这里推荐一下：
>
>
>
> [基于物理着色（一）](http://zhuanlan.zhihu.com/graphics/20091064)
>
> [基于物理着色（二）- Microfacet材质和多层材质](http://zhuanlan.zhihu.com/graphics/20119162)
>
> [基于物理着色（三）- Disney和UE4的实现](http://zhuanlan.zhihu.com/graphics/20122884)
>
>
>
> 这边贴一张专栏中的配套渲染图，效果非常出色：
>
> ![](https://img-blog.csdn.net/20151101111821822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
>
>
>
>
>
>
> ## 1.2 如何使用标准着色器
>
>
>
>
>
> 可以在Unity5中任意材质的Shader选项中的最前面看到两个Standard Shader字样的选项。其中第一个是普通版，第二个带Specularsetup是高光版。
>
> ![](https://img-blog.csdn.net/20151101111858712?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
>
>
>
>
> 标准着色器标准版材质界面：
>
> ![](https://img-blog.csdn.net/20151101111930623?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
> 标准着色器的高光版材质界面：
>
> ![](https://img-blog.csdn.net/20151105152108339?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
> 不难得知，标准着色器引入了新的材质编辑器，它使PBS的材质编辑工作比以前的非PBS材质更简单。新的编辑器不但简洁，还提供了材质的所有可能用到的选项。在新编辑器中，我们不需要选用不同的着色器来改变纹理通道；不会再出现 “texture unused, please choose another shader” 这样的提示；也不再需要通过切换着色器来改变混合模式。
>
> 所有的纹理通道都是备选的，无需强制使用，任何一个闲置通道的相关代码都会在编译时被优化掉，因此完全不用担心效率方面的问题。unity会根据我们输入到编辑器中的数据来生成正确的代码，并使整个过程保持高效。
>
>
>
> 另外，我们可以用ctrl+点击纹理的方式预览大图，并且还可以分别查看颜色和Alpha通道。
>
>
>
> ctrl+点击纹理，得到的纹理放大的预览：
>
> ![](https://img-blog.csdn.net/20151101191812463?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> OK，关于Unity5中的StandardShader的概念，大概先就讲这么多。更多细节可以参考Unity官方文档：
>
> http://docs.unity3d.com/Manual/shader-StandardShader.html
>
>
>
> 以及这里Unity5 Standard Shader的官方文档论坛翻译版：
>
> http://forum.china.unity3d.com/thread-897-1-1.html
>
>
>
>
>
>
>
>
>
> ## 1.3 理解标准着色器的组成
>
>
>
> 上面说了很多标准着色器的强大之处，其实，作为一个着色器，它也没有什么神秘的地方，无非就是两个Shader源文件，加上一堆CG头文件组成的两个功能稍微复杂全面一些的Shader而已。其中，两个Shader源文件里，又按渲染路径分为了很多的SubShader，每个SubShader里面又分为了很多Pass。而CG文件中，主要包含了Shader的支持函数，相关的宏等为Shader源文件提供支持的代码。
>
>
>
> Unity5中标准着色器的组成，归纳概括如下：
>
>
>
>
>
> * 两个Shader源文件
> * 七个CG头文件
> * 一个脚本文件（用于自定义材质编辑器UI）
>
>
>
>
>
> 下面分别对每个文件进行一个简单的介绍。
>
>
>
> ### 1)两个Shader源文件

> >

> * Stardard.shader着色器源文件 - 标准着色器的标准版
> * StardardSpecular.shader着色器源文件 - 标准着色器的高光版

> > ![](https://img-blog.csdn.net/20151101191917962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> #### 2)七个CG头文件
>
>   
>
>   
>
>
>
> * UnityStandardBRDF.cginc头文件-用于存放标准着色器处理BRDF材质属性相关的函数与宏
> * UnityStandardConfig.cginc头文件-用于存放标准着色器配置相关的代码（其实里面就几个宏）
> * UnityStandardCore.cginc头文件-用于存放标准着色器的主要代码（如顶点着色函数、片段着色函数等相关函数）
> * UnityStandardInput.cginc头文件-用于存放标准着色器输入结构相关的工具函数与宏
> * UnityStandardMeta.cginc头文件-用于存放标准着色器meta通道中会用到的工具函数与宏
> * UnityStandardShadow.cginc头文件-用于存放标准着色器阴影贴图采样相关的工具函数与宏
> * UnityStandardUtils.cginc头文件-用于存放标准着色器共用的一些工具函数

> > ![](https://img-blog.csdn.net/20151101191927534?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 在包括本文在内的接下来的几次更新中，本系列文章将试着花篇幅来剖析这两个Shader源文件，和依附的几个CG头文件的源码，从而一窥Unity5中新版Shader书写体系的究竟。
>
>
>
>
>
> #### 3)一个脚本文件
>
>   
>
>   
>
>
>
> * StandardShaderGUI.cs脚本文件——定义了特定的自定义编辑器UI界面

> > ![](https://img-blog.csdn.net/20151101191953962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 标准着色器对应材质的编辑器外观不同于一般的Shader，就是因为在Shader末尾书写了如下的代码：
>
>
>
>
>
>
> ```language-csharp
> //使用特定的自定义编辑器UI界面  
>CustomEditor"StandardShaderGUI"
> ```
>   
>
>
>   
>
>
>
>
>
>
> 标准着色器对应材质的编辑器外观如下：
>
> ![](https://img-blog.csdn.net/20151101175108195?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
>
>
>
>
>
>
> 一般的着色器对应材质的编辑器外观如下：
>
> ![](https://img-blog.csdn.net/20151101175115745?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
>
>
>
>
>
>
>
>
> # 二、Unity5标准着色器源代码剖析之一：架构分析篇
>
>
>
>
>
>
>
> 上文已经提到过，标准着色器源代码的剖析是一个小小的马拉松，完全解析起来篇幅会很长，所以本系列文章将对剖析的过程进行连载，此节为连载的第一部分。
>
>
>
> 在这里先贴出经过浅墨详细注释的标准着色器标准版的源代码，并对架构进行简单的分析，而对每个通道的剖析在稍后的更新中会进行。
>
>
>
>
> ```language-cpp
> //-----------------------------------------------【Shader说明】---------------------------------------------------//		 Unity5.2.1 Built-in Standard Shader//      2015年10月  Commented by  浅墨    //      更多内容或交流，请访问浅墨的博客：http://blog.csdn.net/poem_qianmo//--------------------------------------------------------------------------------------------------------------------- Shader "Standard"{	//------------------------------------【属性值】------------------------------------	Properties	{		//主颜色		_Color("Color", Color) = (1,1,1,1)		//主纹理		_MainTex("Albedo", 2D) = "white" {} 		//Alpha剔除值		_Cutoff("Alpha Cutoff", Range(0.0, 1.0)) = 0.5		//平滑、光泽度		_Glossiness("Smoothness", Range(0.0, 1.0)) = 0.5 		//金属性		[Gamma] _Metallic("Metallic", Range(0.0, 1.0)) = 0.0		//金属光泽纹理图		_MetallicGlossMap("Metallic", 2D) = "white" {} 		//凹凸的尺度		_BumpScale("Scale", Float) = 1.0		//法线贴图		_BumpMap("Normal Map", 2D) = "bump" {} 		//高度缩放尺度		_Parallax ("Height Scale", Range (0.005, 0.08)) = 0.02		//高度纹理图		_ParallaxMap ("Height Map", 2D) = "black" {} 		//遮挡强度		_OcclusionStrength("Strength", Range(0.0, 1.0)) = 1.0		//遮挡纹理图		_OcclusionMap("Occlusion", 2D) = "white" {} 		//自发光颜色		_EmissionColor("Color", Color) = (0,0,0)		//自发光纹理图		_EmissionMap("Emission", 2D) = "white" {}				//细节掩膜图		_DetailMask("Detail Mask", 2D) = "white" {} 		//细节纹理图		_DetailAlbedoMap("Detail Albedo x2", 2D) = "grey" {}		//细节法线贴图尺度		_DetailNormalMapScale("Scale", Float) = 1.0		//细节法线贴图		_DetailNormalMap("Normal Map", 2D) = "bump" {} 		//二级纹理的UV设置		[Enum(UV0,0,UV1,1)] _UVSec ("UV Set for secondary textures", Float) = 0 		//混合状态的定义		[HideInInspector] _Mode ("__mode", Float) = 0.0		[HideInInspector] _SrcBlend ("__src", Float) = 1.0		[HideInInspector] _DstBlend ("__dst", Float) = 0.0		[HideInInspector] _ZWrite ("__zw", Float) = 1.0	} 	//===========开始CG着色器语言编写模块===========	CGINCLUDE		//BRDF相关的一个宏		#define UNITY_SETUP_BRDF_INPUT MetallicSetup	//===========结束CG着色器语言编写模块===========	ENDCG  	//------------------------------------【子着色器1】------------------------------------	// 此子着色器用于Shader Model 3.0	//----------------------------------------------------------------------------------------	SubShader	{		//渲染类型设置：不透明		Tags { "RenderType"="Opaque" "PerformanceChecks"="False" } 		//细节层次设为：300		LOD 300				//--------------------------------通道1-------------------------------		// 正向基础渲染通道（Base forward pass）		// 处理方向光，自发光，光照贴图等 ...		Pass		{			//设置通道名称			Name "FORWARD"  			//于通道标签中设置光照模型为ForwardBase，正向渲染基础通道			Tags { "LightMode" = "ForwardBase" } 			//混合操作：源混合乘以目标混合			Blend [_SrcBlend] [_DstBlend]			// 根据_ZWrite参数，设置深度写入模式开关与否			ZWrite [_ZWrite] 			//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//着色器编译目标：Model 3.0			#pragma target 3.0 			//编译指令：不使用GLES渲染器编译			#pragma exclude_renderers gles						// ---------编译指令：着色器编译多样化--------			#pragma shader_feature _NORMALMAP			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON			#pragma shader_feature _EMISSION			#pragma shader_feature _METALLICGLOSSMAP 			#pragma shader_feature ___ _DETAIL_MULX2			#pragma shader_feature _PARALLAXMAP						//--------着色器编译多样化快捷指令------------			//编译指令：编译正向渲染基础通道（用于正向渲染中，应用环境光照、主方向光照和顶点/球面调和光照）所需的所有变体。			//这些变体用于处理不同的光照贴图类型、主要方向光源的阴影选项的开关与否			#pragma multi_compile_fwdbase			//编译指令：编译几个不同变种来处理不同类型的雾效(关闭/线性/指数/二阶指数/)			#pragma multi_compile_fog						//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertForwardBase			#pragma fragment fragForwardBase 			//包含辅助CG头文件			#include "UnityStandardCore.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		}		//--------------------------------通道2-------------------------------		// 正向附加渲染通道（Additive forward pass）		// 以每个光照一个通道的方式应用附加的逐像素光照		Pass		{			//设置通道名称			Name "FORWARD_DELTA" 			//于通道标签中设置光照模型为ForwardAdd，正向渲染附加通道			Tags { "LightMode" = "ForwardAdd" } 			//混合操作：源混合乘以1			Blend [_SrcBlend] One 			//附加通道中的雾效应该为黑色			Fog { Color (0,0,0,0) }  			//关闭深度写入模式			ZWrite Off			//设置深度测试模式：小于等于			ZTest LEqual 			//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//着色器编译目标：Model 3.0			#pragma target 3.0			//编译指令：不使用GLES渲染器编译			#pragma exclude_renderers gles 			// ---------编译指令：着色器编译多样化--------			#pragma shader_feature _NORMALMAP			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON			#pragma shader_feature _METALLICGLOSSMAP			#pragma shader_feature ___ _DETAIL_MULX2			#pragma shader_feature _PARALLAXMAP						//--------使用Unity内置的着色器编译多样化快捷指令------------			//编译指令：编译正向渲染基础通道所需的所有变体，但同时为上述通道的处理赋予了光照实时阴影的能力。			#pragma multi_compile_fwdadd_fullshadows			//编译指令：编译几个不同变种来处理不同类型的雾效(关闭/线性/指数/二阶指数/)			#pragma multi_compile_fog 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertForwardAdd			#pragma fragment fragForwardAdd 			//包含辅助CG头文件			#include "UnityStandardCore.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		// --------------------------------通道3-------------------------------		//  阴影渲染通道（Shadow Caster pass）		//  将将物体的深度渲染到阴影贴图或深度纹理中		Pass 		{			//设置通道名称			Name "ShadowCaster"			//于通道标签中设置光照模型为ShadowCaster。			//此光照模型代表着将物体的深度渲染到阴影贴图或深度纹理。			Tags { "LightMode" = "ShadowCaster" } 			//开启深入写入模式			ZWrite On 			//设置深度测试模式：小于等于			ZTest LEqual 			//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//着色器编译目标：Model 3.0			#pragma target 3.0 			//编译指令：不使用GLES渲染器编译			#pragma exclude_renderers gles			 			// ---------编译指令：着色器编译多样化--------			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON						//--------着色器编译多样化快捷指令------------			//进行阴影投射相关的多着色器变体的编译			#pragma multi_compile_shadowcaster 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertShadowCaster			#pragma fragment fragShadowCaster						//包含辅助CG头文件			#include "UnityStandardShadow.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		// --------------------------------通道4-------------------------------		//  延迟渲染通道（Deferred Render Pass）		Pass		{			//设置通道名称			Name "DEFERRED"			//于通道标签中设置光照模型为Deferred，延迟渲染通道			Tags { "LightMode" = "Deferred" } 			CGPROGRAM			#pragma target 3.0			// TEMPORARY: GLES2.0 temporarily disabled to prevent errors spam on devices without textureCubeLodEXT			#pragma exclude_renderers nomrt gles			 			//---------编译指令：着色器编译多样化（shader_feature）--------			#pragma shader_feature _NORMALMAP			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON			#pragma shader_feature _EMISSION			#pragma shader_feature _METALLICGLOSSMAP			#pragma shader_feature ___ _DETAIL_MULX2			#pragma shader_feature _PARALLAXMAP 			//---------编译指令：着色器编译多样化（multi_compile）--------			#pragma multi_compile ___ UNITY_HDR_ON			#pragma multi_compile LIGHTMAP_OFF LIGHTMAP_ON			#pragma multi_compile DIRLIGHTMAP_OFF DIRLIGHTMAP_COMBINED DIRLIGHTMAP_SEPARATE			#pragma multi_compile DYNAMICLIGHTMAP_OFF DYNAMICLIGHTMAP_ON						//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertDeferred			#pragma fragment fragDeferred 			//包含辅助CG头文件			#include "UnityStandardCore.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		// --------------------------------通道5-------------------------------		//元通道（Meta Pass）		//为全局光照（GI），光照贴图等技术提取相关参数，如（emission, albedo等参数值）		//此通道并不在常规的渲染过程中使用		Pass		{			//设置通道名称			Name "META"  			//于通道标签中设置光照模型为Meta			//（截止2015年10月22日，Unity 5.2.1的官方文档中并没有收录此光照模型，应该是Unity官方的疏漏）			Tags { "LightMode"="Meta" }			//关闭剔除操作			Cull Off 			//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vert_meta			#pragma fragment frag_meta 			//---------编译指令：着色器编译多样化--------			#pragma shader_feature _EMISSION			#pragma shader_feature _METALLICGLOSSMAP			#pragma shader_feature ___ _DETAIL_MULX2 			//包含辅助CG头文件			#include "UnityStandardMeta.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		}	} 	//------------------------------------【子着色器2】-----------------------------------	// 此子着色器用于Shader Model 2.0	//----------------------------------------------------------------------------------------	SubShader	{		//渲染类型设置：不透明		Tags { "RenderType"="Opaque" "PerformanceChecks"="False" }		//细节层次设为：150		LOD 150 		//--------------------------------通道1-------------------------------		// 正向基础渲染通道（Base forward pass）		// 处理方向光，自发光，光照贴图等 ...		Pass		{			//设置通道名称			Name "FORWARD" 			//于通道标签中设置光照模型为ForwardBase，正向渲染基础通道			Tags { "LightMode" = "ForwardBase" }			//混合操作：源混合乘以目标混合，即结果为两者的混合			Blend [_SrcBlend] [_DstBlend]			// 根据_ZWrite参数，设置深度写入模式开关与否			ZWrite [_ZWrite] 			//===========开启CG着色器语言编写模块===========			CGPROGRAM			//着色器编译目标：Model 2.0			#pragma target 2.0 			// ---------编译指令：着色器编译多样化--------			#pragma shader_feature _NORMALMAP			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON			#pragma shader_feature _EMISSION 			#pragma shader_feature _METALLICGLOSSMAP 			#pragma shader_feature ___ _DETAIL_MULX2			// SM2.0: NOT SUPPORTED shader_feature _PARALLAXMAP 			//跳过如下变体的编译，简化编译过程			#pragma skip_variants SHADOWS_SOFT DIRLIGHTMAP_COMBINED DIRLIGHTMAP_SEPARATE						//--------着色器编译多样化快捷指令------------			#pragma multi_compile_fwdbase			#pragma multi_compile_fog 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertForwardBase			#pragma fragment fragForwardBase 			//包含辅助CG头文件			#include "UnityStandardCore.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		//--------------------------------通道2-------------------------------		// 正向附加渲染通道（Additive forward pass）		// 以每个光照一个通道的方式应用附加的逐像素光照		Pass		{			//设置通道名称			Name "FORWARD_DELTA" 			//于通道标签中设置光照模型为ForwardAdd，正向渲染附加通道			Tags { "LightMode" = "ForwardAdd" } 			//混合操作：源混合乘以1			Blend [_SrcBlend] One 			//附加通道中的雾效应该为黑色			Fog { Color (0,0,0,0) }  			//关闭深度写入模式			ZWrite Off 			//设置深度测试模式：小于等于			ZTest LEqual 			//===========开启CG着色器语言编写模块===========			CGPROGRAM			//着色器编译目标：Model 2.0			#pragma target 2.0 			// ---------编译指令：着色器编译多样化--------			#pragma shader_feature _NORMALMAP			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON			#pragma shader_feature _METALLICGLOSSMAP			#pragma shader_feature ___ _DETAIL_MULX2 			//跳过一些变体的编译			// SM2.0: NOT SUPPORTED shader_feature _PARALLAXMAP			#pragma skip_variants SHADOWS_SOFT 			//--------使用Unity内置的着色器编译多样化快捷指令------------			//编译指令：编译正向渲染基础通道所需的所有变体，但同时为上述通道的处理赋予了光照实时阴影的能力。			#pragma multi_compile_fwdadd_fullshadows			//编译指令：编译几个不同变种来处理不同类型的雾效(关闭/线性/指数/二阶指数/)			#pragma multi_compile_fog 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertForwardAdd			#pragma fragment fragForwardAdd 			//包含辅助CG头文件			#include "UnityStandardCore.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		// --------------------------------通道3-------------------------------		//  阴影渲染通道（Shadow Caster pass）		//  将将物体的深度渲染到阴影贴图或深度纹理中		Pass 		{			//设置通道名称			Name "ShadowCaster" 			//于通道标签中设置光照模型为ShadowCaster。			//此光照模型代表着将物体的深度渲染到阴影贴图或深度纹理。			Tags { "LightMode" = "ShadowCaster" } 			//开启深入写入模式			ZWrite On 			//设置深度测试模式：小于等于			ZTest LEqual 			//===========开启CG着色器语言编写模块===========			CGPROGRAM			//着色器编译目标：Model 2.0			#pragma target 2.0 			//---------编译指令：着色器编译多样化（shader_feature）--------			#pragma shader_feature _ _ALPHATEST_ON _ALPHABLEND_ON _ALPHAPREMULTIPLY_ON						//编译指令：跳过某些变体的编译			#pragma skip_variants SHADOWS_SOFT 			//快捷编译指令：进行阴影投射相关的多着色器变体的编译			#pragma multi_compile_shadowcaster 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vertShadowCaster			#pragma fragment fragShadowCaster 			//包含辅助CG头文件			#include "UnityStandardShadow.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		} 		// --------------------------------通道4-------------------------------		//元通道（Meta Pass）		//为全局光照（GI），光照贴图等技术提取相关参数，如（emission, albedo等参数值）		//此通道并不在常规的渲染过程中使用		Pass		{			//设置通道名称			Name "META"  			//于通道标签中设置光照模型为Meta			//（截止2015年10月22日，Unity 5.2.1的官方文档中并没有收录此光照模型，应该是Unity官方的疏漏）			Tags { "LightMode"="Meta" }			//关闭剔除操作			Cull Off 			//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//编译指令：告知编译器顶点和片段着色函数的名称			#pragma vertex vert_meta			#pragma fragment frag_meta 			//---------编译指令：着色器编译多样化--------			#pragma shader_feature _EMISSION			#pragma shader_feature _METALLICGLOSSMAP			#pragma shader_feature ___ _DETAIL_MULX2 			//包含辅助CG头文件			#include "UnityStandardMeta.cginc" 			//===========结束CG着色器语言编写模块===========			ENDCG		}	} 	//回退Shader为顶点光照Shader	FallBack "VertexLit"	//使用特定的自定义编辑器UI界面	CustomEditor "StandardShaderGUI"}
> ```
>   
>
>
>
>
>
>
> 标准着色器的源代码部分相对于着色器的体量来说，算是有点长的，加上注释后有近500行。先稍微把它的架构稍微理一下。
>
> 标准着色器由两个SubShader组成。第一个SubShader用于处理Shader Model 3.0，有5个通道，第二个SubShader用于处理Shader Model 2.0, 有4个通道，
>
> 详细的架构如下图:
>
> ![](https://img-blog.csdn.net/20151101180146817?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
>
>
>
>
> OK，标准着色器的架构大致如上。今天就先讲这么多，上面代码的稍微有些看不懂没关系，从下次更新开始，就将深入到每个Pass的顶点和片段着色器函数之中，逐行注释与分析他们的指令细节。
>
>
>
> 而不难发现，标准着色器代码的第一个SubShader比第二个SubShader多出了一个延迟渲染通道（Deferred Render Pass），下面稍微提一下延迟渲染的基本概念。
>
>
>
>
>
>
>
>
>
>
>
>
>
> ## 三、关于延迟渲染（Deferred Render）
>
>
>
>
>
>
>
>
>
> 看过《GPU Gems2》的朋友们应该都有所了解，延迟渲染（Deferred Render，又称Deferred Shading）是次时代引擎必备的渲染方式之一。
>
> 需要注意，Deferred Render和Deferred Shading有一点细微的差别。
>
>
>
> * Deferred Shading：将所有的Shading全部转到Deferred阶段进行。
> * Deferred Render：只是有选择地将Lighting转到Deferred阶段进行。
>
>
>
> Unity中默认使用的是Deferred Shading。
>
>
>
> 而延迟渲染，一言以蔽之，就是将光照/渲染的计算延迟到第二步进行，避免多次渲染同一个像素，从而减少多余的计算操作，以提高渲染效率的一种先进的渲染方式。
>
>
>
> 延迟渲染最大的优势是可以实现同屏中数量众多的动态光源（十几到几十个），这在传统的渲染管线中是很难实现的。
>
>
>
> 更多延迟渲染的细节，这边不细说，只是提供一些链接，以作进一步了解延迟渲染的导论之用：
>
>
>
>
>
> * https://zh.wikipedia.org/wiki/%E5%BB%B6%E6%9C%9F%E7%9D%80%E8%89%B2
> * http://blog.csdn.net/noslopforever/article/details/3951273
> * http://blog.sina.com.cn/s/blog_458f871201017i06.html
> * http://www.cnblogs.com/lancidie/archive/2011/08/18/2144748.html
> * http://blog.csdn.net/pizi0475/article/details/7932920
> * http://www.cnblogs.com/wangchengfeng/p/3440097.html
> * http://blog.csdn.net/bugrunner/article/details/7436600
>
>
>
>
>
>
>
>
>
>
>
>
>
> # 四、屏幕水幕特效的实现
>
>
>
>
>
>
>
> 在上一篇文章中有提到过，Unity中的屏幕特效通常分为两部分来实现：
>
>
>
>
>
> * Shader实现部分
> * 脚本实现部分
>
>
>
>
>
> 下面依然是从这两个方面对本次的特效进行实现。
>
>
>
> 而在这之前，需要准备好一张水滴（水滴太多了也就成了水幕了）的效果图片（google“water drop”一下，稍微筛选一下就有了，最好是能找到或者自己加工成无缝衔接的），放置于我们特效的脚本实现文件目录附加的一个Resources的文件夹中，那么我们在脚本中适当的地方写上一句：
>
> Texture2 = Resources.Load("ScreenWaterDrop")as Texture2D;
>
>
>
> 就可以读取到这张图片了。
>
> 浅墨准备的图片如下（无水印本图片的可以在[浅墨的Github](https://github.com/QianMo/Awesome-Unity-Shader/tree/master/%E3%80%90%E6%B5%85%E5%A2%A8Unity3D%20Shader%E7%BC%96%E7%A8%8B%E3%80%91%E4%B9%8B%E4%B9%9D%20%E9%85%8D%E5%A5%97%E6%BA%90%E7%A0%81/ScreenWaterDropEffect/Resources)中找到，或者直接下文章末尾的项目工程）。
>
>
>
> ScreenWaterDrop.png：
>
> ![](https://img-blog.csdn.net/20151101180358328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
>
>
>
>
> ## 4.1 Shader实现部分
>
>
>
>
>
> 老规矩，先上详细注释的代码。
>
>
>
>
> ```language-cpp
> //-----------------------------------------------【Shader脚本说明】---------------------------------------------------//		 屏幕水幕特效的实现代码-Shader脚本部分//      2015年10月  Created by  浅墨//      更多内容或交流，请访问浅墨的博客：http://blog.csdn.net/poem_qianmo//--------------------------------------------------------------------------------------------------------------------- Shader "浅墨Shader编程/Volume9/ScreenWaterDropEffect"{	//------------------------------------【属性值】------------------------------------	Properties	{		//主纹理		_MainTex ("Base (RGB)", 2D) = "white" {}		//屏幕水滴的素材图		_ScreenWaterDropTex ("Base (RGB)", 2D) = "white" {}		//当前时间		_CurTime ("Time", Range(0.0, 1.0)) = 1.0		//X坐标上的水滴尺寸		_SizeX ("SizeX", Range(0.0, 1.0)) = 1.0		//Y坐标上的水滴尺寸		_SizeY ("SizeY", Range(0.0, 1.0)) = 1.0		//水滴的流动速度		_DropSpeed ("Speed", Range(0.0, 10.0)) = 1.0		//溶解度		_Distortion ("_Distortion", Range(0.0, 1.0)) = 0.87	} 	//------------------------------------【唯一的子着色器】------------------------------------	SubShader	{		Pass		{			//设置深度测试模式:渲染所有像素.等同于关闭透明度测试（AlphaTest Off）			ZTest Always						//===========开启CG着色器语言编写模块===========			CGPROGRAM 			//编译指令:告知编译器顶点和片段着色函数的名称			#pragma vertex vert			#pragma fragment frag			#pragma fragmentoption ARB_precision_hint_fastest			//编译指令: 指定着色器编译目标为Shader Model 3.0			#pragma target 3.0 			//包含辅助CG头文件			#include "UnityCG.cginc" 			//外部变量的声明			uniform sampler2D _MainTex;			uniform sampler2D _ScreenWaterDropTex;			uniform float _CurTime;			uniform float _DropSpeed;			uniform float _SizeX;			uniform float _SizeY;			uniform float _Distortion;			uniform float2 _MainTex_TexelSize;						//顶点输入结构			struct vertexInput			{				float4 vertex : POSITION;//顶点位置				float4 color : COLOR;//颜色值				float2 texcoord : TEXCOORD0;//一级纹理坐标			}; 			//顶点输出结构			struct vertexOutput			{				half2 texcoord : TEXCOORD0;//一级纹理坐标				float4 vertex : SV_POSITION;//像素位置				fixed4 color : COLOR;//颜色值			}; 			//--------------------------------【顶点着色函数】-----------------------------			// 输入：顶点输入结构体			// 输出：顶点输出结构体			//---------------------------------------------------------------------------------			vertexOutput vert(vertexInput Input)			{				//【1】声明一个输出结构对象				vertexOutput Output; 				//【2】填充此输出结构				//输出的顶点位置为模型视图投影矩阵乘以顶点位置，也就是将三维空间中的坐标投影到了二维窗口				Output.vertex = mul(UNITY_MATRIX_MVP, Input.vertex);				//输出的纹理坐标也就是输入的纹理坐标				Output.texcoord = Input.texcoord;				//输出的颜色值也就是输入的颜色值				Output.color = Input.color; 				//【3】返回此输出结构对象				return Output;			} 			//--------------------------------【片段着色函数】-----------------------------			// 输入：顶点输出结构体			// 输出：float4型的颜色值			//---------------------------------------------------------------------------------			fixed4 frag(vertexOutput Input) : COLOR			{				//【1】获取顶点的坐标值				float2 uv = Input.texcoord.xy; 				//【2】解决平台差异的问题。校正方向，若和规定方向相反，则将速度反向并加1				#if UNITY_UV_STARTS_AT_TOP				if (_MainTex_TexelSize.y < 0)					_DropSpeed = 1 - _DropSpeed;				#endif 				//【3】设置三层水流效果，按照一定的规律在水滴纹理上分别进行取样				float3 rainTex1 = tex2D(_ScreenWaterDropTex, float2(uv.x * 1.15* _SizeX, (uv.y* _SizeY *1.1) + _CurTime* _DropSpeed *0.15)).rgb / _Distortion;				float3 rainTex2 = tex2D(_ScreenWaterDropTex, float2(uv.x * 1.25* _SizeX - 0.1, (uv.y *_SizeY * 1.2) + _CurTime *_DropSpeed * 0.2)).rgb / _Distortion;				float3 rainTex3 = tex2D(_ScreenWaterDropTex, float2(uv.x* _SizeX *0.9, (uv.y *_SizeY * 1.25) + _CurTime * _DropSpeed* 0.032)).rgb / _Distortion; 				//【4】整合三层水流效果的颜色信息，存于finalRainTex中				float2 finalRainTex = uv.xy - (rainTex1.xy - rainTex2.xy - rainTex3.xy) / 3; 				//【5】按照finalRainTex的坐标信息，在主纹理上进行采样				float3 finalColor = tex2D(_MainTex, float2(finalRainTex.x, finalRainTex.y)).rgb; 				//【6】返回加上alpha分量的最终颜色值				return fixed4(finalColor, 1.0);  			} 			//===========结束CG着色器语言编写模块===========			ENDCG		}	}}
> ```
>
>
>   
>
>
>
>
> 屏幕特效Shader中真正有营养的核心代码，一般都是位于像素着色器中。也就是这里的frag函数中，稍微聊一聊。
>
>
>
> 第一步，先从顶点着色器输出结构体中获取顶点的坐标：
>
>
>
>
> ```language-cpp
> //【1】获取顶点的坐标值float2 uv = Input.texcoord.xy;
> ```
>
>
>   
>
>
> 第二步，因为需要水幕纹理在屏幕中从上向下滚动，而不同平台的原点位置是不同的，Direct3D原点在左上角，OpenGL原点在左下角。所以在这边需要对情况进行统一。
>
> 统一的方法里用到了UNITY_UV_STARTS_AT_TOP宏，它是Unity中内置的宏，其值 取为1 或0 ； 在纹理 V 坐标在“纹理顶部”为零的平台上值取 1。如Direct3D；而OpenGL 平台上取 0。
>
> 另外，这边还用到了MainTex_TexelSize变量。
>
> float2型的MainTex_TexelSize（其实是_TexelSize后缀，前面的名称会根据定义纹理变量的改变而改变）也是Unity中内置的一个变量，存放了纹理中单个像素的尺寸，也就是说，如果有一张2048 x 2048的纹理，那么x和y的取值都为1.0/2048.0。而且需要注意，当该纹理被Direct3D的抗锯齿操作垂直反转过后，xxx_TexelSize的值将为负数。
>
>
>
> 所以，第二步的代码就是如下：
>
>
>
>
> ```language-cpp
> //【2】解决平台差异的问题。校正方向，若和规定方向相反，则将速度反向并加1    #if UNITY_UV_STARTS_AT_TOP     if(_MainTex_TexelSize.y < 0)       _DropSpeed= 1 - _DropSpeed;           #endif
> ```
>   
>  
>
>
>
> 第三步，定义三层水流的纹理，按照不同的参数取值，进行采样：
>
>
>
>
> ```language-cpp
> //【3】设置三层水流效果，按照一定的规律在水滴纹理上分别进行取样float3 rainTex1 = tex2D(_ScreenWaterDropTex, float2(uv.x * 1.15* _SizeX, (uv.y* _SizeY*1.1) + _CurTime* _DropSpeed *0.15)).rgb / _Distortion;float3 rainTex2 = tex2D(_ScreenWaterDropTex, float2(uv.x * 1.25* _SizeX - 0.1, (uv.y*_SizeY * 1.2) + _CurTime *_DropSpeed * 0.2)).rgb / _Distortion;float3 rainTex3 = tex2D(_ScreenWaterDropTex, float2(uv.x* _SizeX *0.9, (uv.y *_SizeY *1.25) + _CurTime * _DropSpeed* 0.032)).rgb / _Distortion;
> ```
>   
>
>
>   
>
>
> 第四步，整合一下三层水流效果的颜色信息，用一个float2型的变量存放下来:
>
>
>
>
> ```language-cpp
> //【4】整合三层水流效果的颜色信息，存于finalRainTex中float2 finalRainTex = uv.xy - (rainTex1.xy - rainTex2.xy - rainTex3.xy) / 3;
> ```
>
>
>   
>
>
>
>
> 第五步，自然是在屏幕所在的纹理_MainTex中进行一次最终的采样，算出最终结果颜色值，存放于float3型的finalColor中。
>
>
>
>
> ```language-cpp
> //【5】按照finalRainTex的坐标信息，在主纹理上进行采样float3 finalColor = tex2D(_MainTex, float2(finalRainTex.x, finalRainTex.y)).rgb;
> ```
>
>
>
>
>
>
> 第六步，因为返回的是一个fixed4型的变量，rgba。所以需要给float3型的finalColor配上一个alpha分量，并返回：
>
> //【6】返回加上alpha分量的最终颜色值
>
>
>
>
> ```language-cpp
> return fixed4(finalColor, 1.0);
> ```
>
>
>
>
>
>
> OK，关于此Shader的实现细节，差不多就需要讲到这些。下面再看一下C#脚本文件的实现。
>
>
>
>
>
>
>
> ### 4.2 C#脚本实现部分
>
>
>
>
>
> C#脚本文件的实现并没有什么好讲的，唯一的地方，水滴纹理的载入，上面已经提到过了，实现代码如下：
>
>
>
>
> ```language-csharp
> //载入素材图 ScreenWaterDropTex = Resources.Load("ScreenWaterDrop") asTexture2D;
> ```
>   
>
>
>
>
> 下面就直接贴出详细注释的实现此特效的C#脚本：
>
>
>
>
> ```language-csharp
> //-----------------------------------------------【C#脚本说明】---------------------------------------------------//	屏幕水幕特效的实现代码-C#脚本部分//      2015年10月  Created by  浅墨//      更多内容或交流，请访问浅墨的博客：http://blog.csdn.net/poem_qianmo//---------------------------------------------------------------------------------------------------------------------  using UnityEngine;using System.Collections; [ExecuteInEditMode][AddComponentMenu("浅墨Shader编程/Volume9/ScreenWaterDropEffect")]public class ScreenWaterDropEffect : MonoBehaviour {    //-------------------变量声明部分-------------------    #region Variables     //着色器和材质实例    public Shader CurShader;//着色器实例    private Material CurMaterial;//当前的材质     //时间变量和素材图的定义    private float TimeX = 1.0f;//时间变量    private Texture2D ScreenWaterDropTex;//屏幕水滴的素材图     //可以在编辑器中调整的参数值    [Range(5, 64), Tooltip("溶解度")]    public float Distortion = 8.0f;    [Range(0, 7), Tooltip("水滴在X坐标上的尺寸")]    public float SizeX = 1f;    [Range(0, 7), Tooltip("水滴在Y坐标上的尺寸")]    public float SizeY = 0.5f;    [Range(0, 10), Tooltip("水滴的流动速度")]    public float DropSpeed = 3.6f;     //用于参数调节的中间变量    public static float ChangeDistortion;    public static float ChangeSizeX;    public static float ChangeSizeY;    public static float ChangeDropSpeed;    #endregion      //-------------------------材质的get&set----------------------------    #region MaterialGetAndSet    Material material    {        get        {            if (CurMaterial == null)            {                CurMaterial = new Material(CurShader);                CurMaterial.hideFlags = HideFlags.HideAndDontSave;            }            return CurMaterial;        }    }    #endregion      //-----------------------------------------【Start()函数】---------------------------------------------      // 说明：此函数仅在Update函数第一次被调用前被调用    //--------------------------------------------------------------------------------------------------------    void Start()    {        //依次赋值        ChangeDistortion = Distortion;        ChangeSizeX = SizeX;        ChangeSizeY = SizeY;        ChangeDropSpeed = DropSpeed;         //载入素材图        ScreenWaterDropTex = Resources.Load("ScreenWaterDrop") as Texture2D;         //找到当前的Shader文件        CurShader = Shader.Find("浅墨Shader编程/Volume9/ScreenWaterDropEffect");         //判断是否支持屏幕特效        if (!SystemInfo.supportsImageEffects)        {            enabled = false;            return;        }    }     //-------------------------------------【OnRenderImage()函数】------------------------------------      // 说明：此函数在当完成所有渲染图片后被调用，用来渲染图片后期效果    //--------------------------------------------------------------------------------------------------------    void OnRenderImage(RenderTexture sourceTexture, RenderTexture destTexture)    {        //着色器实例不为空，就进行参数设置        if (CurShader != null)        {            //时间的变化            TimeX += Time.deltaTime;            //时间大于100，便置0，保证可以循环            if (TimeX > 100) TimeX = 0;             //设置Shader中其他的外部变量            material.SetFloat("_CurTime", TimeX);            material.SetFloat("_Distortion", Distortion);            material.SetFloat("_SizeX", SizeX);            material.SetFloat("_SizeY", SizeY);            material.SetFloat("_DropSpeed", DropSpeed);            material.SetTexture("_ScreenWaterDropTex", ScreenWaterDropTex);             //拷贝源纹理到目标渲染纹理，加上我们的材质效果            Graphics.Blit(sourceTexture, destTexture, material);        }        //着色器实例为空，直接拷贝屏幕上的效果。此情况下是没有实现屏幕特效的        else        {            //直接拷贝源纹理到目标渲染纹理            Graphics.Blit(sourceTexture, destTexture);        }      }     //-----------------------------------------【OnValidate()函数】--------------------------------------      // 说明：此函数在编辑器中该脚本的某个值发生了改变后被调用    //--------------------------------------------------------------------------------------------------------    void OnValidate()    {        ChangeDistortion = Distortion;        ChangeSizeX = SizeX;        ChangeSizeY = SizeY;        ChangeDropSpeed = DropSpeed;    }      //-----------------------------------------【Update()函数】------------------------------------------      // 说明：此函数在每一帧中都会被调用    //--------------------------------------------------------------------------------------------------------     void Update()    {        //若程序在运行，进行赋值        if (Application.isPlaying)        {            //赋值            Distortion = ChangeDistortion;            SizeX = ChangeSizeX;            SizeY = ChangeSizeY;            DropSpeed = ChangeDropSpeed;        }        //找到对应的Shader文件，和纹理素材#if UNITY_EDITOR        if (Application.isPlaying != true)        {            CurShader = Shader.Find("浅墨Shader编程/Volume9/ScreenWaterDropEffect");            ScreenWaterDropTex = Resources.Load("ScreenWaterDrop") as Texture2D;         }#endif     }     //-----------------------------------------【OnDisable()函数】---------------------------------------      // 说明：当对象变为不可用或非激活状态时此函数便被调用      //--------------------------------------------------------------------------------------------------------    void OnDisable()    {        if (CurMaterial)        {            //立即销毁材质实例            DestroyImmediate(CurMaterial);        }     }}
> ```
>   
>
>
>
>
>
>
>
>
> OK，水幕屏幕特效实现部分大致就是这样，下面看一下运行效果的对比。
>
>
>
>
>
>
>
>
>
> ## 五、最终的效果展示
>
>
>
>
>
>
>
> 贴几张场景的效果图和使用了屏幕特效后的效果图。在试玩场景时，除了类似CS/CF的FPS游戏控制系统以外，还可以使用键盘上的按键【F】，开启或者屏幕特效。
>
> 本次的场景还是使用上次更新中放出的城镇，只是出生地点有所改变。为了有更多的时间专注于Shader书写本身，以后的博文配套场景采取不定期大更新的形式（两次、三次一换）。
>
>
>
>
>
> 城镇野外（with 屏幕水幕特效）：
>
> ![](https://img-blog.csdn.net/20151101190253808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
>
> 城镇野外（原始图）：![](https://img-blog.csdn.net/20151101190314088?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
>
> 山坡上（with 屏幕水幕特效）：
>
> ![](https://img-blog.csdn.net/20151101190330938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> 山坡上（原始图）：  
>
> ![](https://img-blog.csdn.net/20151101190516839?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> 城镇中（with 屏幕水幕特效）：  
>
> ![](https://img-blog.csdn.net/20151101190406420?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> 城镇中（原始图）
>
> ![](https://img-blog.csdn.net/20151101190614944?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> 石桥上（with 屏幕水幕特效）：  
>
> ![](https://img-blog.csdn.net/20151101190433219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
>
>
>
> 石桥上（原始图）：  
>
> ![](https://img-blog.csdn.net/20151101190652716?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
>
>
>
>
> 本次的更新大致如此，感谢各位捧场，我们下周再见。
>
>
>
>
>
> # 附： 本博文相关下载链接清单
>
>
>
>
>
> [【百度云】博文示例场景exe下载](http://yun.baidu.com/s/1nt7w5aL)
>
> [【百度云】包含博文示例场景所有资源与源码的unitypackage下载](http://pan.baidu.com/s/1kThcn2Z)
>
>
>
> [【Github】本文全部Shader源代码](https://github.com/QianMo/Awesome-Unity-Shader/tree/master/Volume%2009%20%E5%B1%8F%E5%B9%95%E6%B0%B4%E5%B9%95%E7%89%B9%E6%95%88Shader%26Standard%20Shader)  
>
>
>   
>
> 另附:若遇到导入unitypackage过程中进度条卡住的情况，不用慌，这是Unity5的一个bug。我也经常在导入工程时遇到。其实这个时候已经导入成功了，用资源管理器杀掉当前这个Unity的进程，再打开就行了。  
>
>
>
> ![](https://img-blog.csdn.net/20151101200203279?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
>
