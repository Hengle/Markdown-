# Unity shader查询手册——基本语法

## 坐标空间
```
1:物体空间: 3D物体自己的坐标空间 
      一般设计时几何体以中心为原点,人物以双脚为原点;
2: 世界空间: 3D物体在场景中的世界坐标, 整个游戏场景的空间;
3: 摄像机空间: 以观察摄像机为原点的坐标系下的坐标空间;
4: 投影成像  3D坐标转换到屏幕空间;
```
## 坐标转换
``` glsl
1: transform.localToWorldMatrix   局部转世界的矩阵;
2: transfrom.worldToLocalMatrix  世界坐标转局部坐标矩阵;
    MultiplyPoint, MultiplyPoint3x4 MultiplayVector 来进行坐标变换;
4: shader中 左乘_World2Object矩阵来实现世界坐标转局部坐标变换;
5: shader中左乘_Object2World矩阵来实现局部转世界的转换; 
6: UNITY_MATRIX_MV 基本变换矩阵 x 摄像机矩阵;
7: UNITY_MATRIX_MVP 基本变换矩阵x摄像机矩阵x投影矩阵;
8: UNITY_MATRIX_V 摄像机矩阵;
9: UNITY_MATRIX_P 投影矩阵;
10: UNITY_MATRIX_VP摄像机矩阵x投影矩阵;
11: UNITY_MATRIX_T_MV (基本变换矩阵 x 摄像机矩阵) 转置矩阵;
12: UNITY_MATRIX_IT_MV(基本变换矩阵 x 摄像机矩阵) 的逆转置矩阵;
13: UNITY_MATRIX_TEXTURE0 纹理变化矩阵;
```

## 结构体表达式
``` glsl
1: struct name {
      类型 名字; 
       // 尽量不要使用;
       返回值 函数名称(参数) { // 如果成员函数里面使用，数据成员，该成员定义在函数前;
       }
};

2:输入语义与输出语义:
    语义: 一个阶段处理数据，然后传输给下一个阶段，那么每个阶段之间的接口, 例如：顶点处理器的输入数据是处于模型空间的顶点数据（位置、法向量），输出的是投影坐标和光照颜色；片段处理器要将光照颜色做为输入;C/C++用指针，而Cg通过语义绑定的形式;
    输入语义: 绑定接收参数,从上一个流水线获得参数;
    输出语义: 绑定输出参数到下一个流水线模块;
    语义: 入口函数上有意义(顶点着色入口,像素着色入口)，普通的函数无意义;
```

## 常用语义修饰
``` glsl
1:POSITION : 位置
2:TANGENT : 切线
3: NORMAL: 法线
4: TEXCOORD0: 第一套纹理
5: TEXCOORD1: 第二套纹理
6: TEXCOORD2: 第三套纹理
7: TEXCOORD3: 第四套纹理
8: COLOR: 颜色
```

## 基本类型表达式
```
1:语法和C语言类是,有对应的编译器,程序是给显卡运行;
2: 可以从渲染流水线中获得对应的输入;
3: 指定的输出能流入下一个流水线模块;
4: 操作符号和C语言一样，可以使用 +, -, * /  <, >, <=, >= 等运算;
5: Cg提供了float half double 浮点类型;
7: Cg 支持定点数 fixed来高效处理 某些小数;
8: Cg使用int来表示整数;
9: bool 数据类型来表示逻辑类型;
10:sampler*,纹理对象的句柄, sampler/1D/2D/3D/CUBE/RECT
11: 内置向量数据类型: float4(float, float, float, float), 向量长度不能超过4;
12: 内置矩阵数据类型: float1x1 float2x3 float4x3 float4x4;不能超过4x4;
13: 数组类型float a[10]; 10个float, float4 b[10], 10个float4;
14: 语义绑定 float4 a : POSITION,返回值也可以语义绑定;
```

## 标准内置函数
``` glsl
1:abs(num)绝对值;
2: 三角函数;
3: cross(a, b) 两个向量的叉积;
4: determinant(M)矩阵的行列式;
5: dot(a, b) 两个向量的点积;
6: floor(x)向下取整;
7: lerp(a, b, f), 在a, b之间线性插值;
8: log2(x) 基于2为底的x的对数;
9: mul(m, n): 矩阵x矩阵, 矩阵x向量, 向量x矩阵;
10: power(x, y) x的y次方;
11: radians(x) 度转弧度;
12: reflect(v, n) v 关于法线n的反射向量;
13: round(x) 靠近取整;
14: tex2D(smapler, x) 二维纹理查找
15: tex3Dproj(smapler, x) 投影三维纹理查找;
16: texCUBE 立方体贴图纹理查找;
17: distance() 计算点的距离; 
```
## 一些常用类型和数值
``` glsl
1: float4是内置向量 (x, y, z, w);   float4 a; 访问单独成员a.x, a.y, a.z, a.w;
2: fixed4 是内置向量(r, g, b, a);   fixed4 c; color.r, color.g, color.b, color.a;
3: float3是内置向量(x, y, z);
4: fixed3 是内置向量(r, g, b);
5: float2 是内置向量(x, y);
6: _Time: 自场景加载开始所经过的时间t，4个分量分别是 (t/20, t, t*2, t*3);
7: _SinTime:  t 是时间的正弦值，4个分量分别是 (t/8, t/4, t/2, t);
8: _CosTime: t 是时间的余弦值，4个分量分别是 (t/8, t/4, t/2, t);
9: unity_DeltaTime: dt 是时间增量，4个分量的值(dt, 1/dt, smoothDt,1/smoothDt),平滑时间，防止时间间隔起伏太大;
```


## Unity自带函数
```
1: 引用Unity自带的函数库: #include “UnityCG.cginc”  Unity-->Edit-->Data-->CGIncludes;
2: TRANSFORM_TEX: 根据顶点的纹理坐标，计算出对应的纹理的真正的UV坐标;
3: 使用属性的变量: 在shader里面需要使用属性变量还需要在shader中定义一下这个变量的类型和名字;
名字要保持一致;
4: 外部修改shader的编辑器上的参数值;
```