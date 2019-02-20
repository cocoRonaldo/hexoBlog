---
title: unity-shader-variables
comments: true
tags:
  - unity
  - shader
date: 2019-02-20 14:30:59
---



#### Unity 内置的变换矩阵

| 变量名             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| UNITY_MATRIX_MVP   | 当前的模型观察投影矩阵，用于将顶点/矢量从模型空间变换到剪裁空间 |
| UNITY_MATRIX_MV    | 当前的模型观察投影矩阵，用于将顶点/矢量从模型空间变换观察空间 |
| UNITY_MATRIX_V     | 当前的观察矩阵，用于将顶点矢量从世界空间变换到观察空间       |
| UNITY_MATRIX_P     | 当前的投影矩阵，用于将顶点矢量从观察空间变换到投影空间       |
| UNITY_MATRIX_VP    | 当前的观察投影矩阵用于将顶点矢量从世界空间转换到剪裁空间     |
| UNITY_MATRIX_T_MV  | UNITY_MATRIX_MV的转置矩阵                                    |
| UNITY_MATRIX_IT_MV | UNITY_MATRIX_MV的逆转置矩阵，用于将法线从模型空间变换到观察空间，也可以用于得到UNITY_MATRIX_MV的逆矩阵 |
| _Object2World      | 当前的模型矩阵，用于将顶点方向矢量从模型空间变换到世界空间   |
| _World2Object      | _Object2World的逆矩阵，用于将顶点方向矢量从世界空间变换到模型空间 |

​				**unity内置的摄像机和屏幕参数**

| 变量名                    | 类型     | 描述                     |
| ------------------------- | -------- | ------------------------ |
| _WorldSpaceCameraPos      | float3   | 摄像机在世界空间中的     |
| unity_CameraProjection    | float4x4 | 摄像机的投影矩阵         |
| unity_CameraInvProjection | float4x4 | 摄像机的投影矩阵的逆矩阵 |
| ...                       | ...      | ...                      |



**Unity提供的CG/HLSL语义**

##### 语义

语义是CG/HLSL提供的语义semantics，实际上是赋值给shader的一些字符串，这些字符串表达了这个参数的含义，通过这种标记，让shader知道从哪里读取数据，并把数据输出到哪里

##### 系统数值语义 system-value

例如用SV_POSITION语义去修饰顶点着色器的输出变量pos，那边就表示pos包含了可用于光栅化的变换后的顶点坐标（即齐次剪裁空间中的坐标），用这些语义修饰的变量是不可以随便赋值的，因为流水线需要他们来完成特定的目的，例如渲染引擎会把用SV_POSITION修饰的变量经过光栅化后显示在屏幕上。

#####unity支持的语义

从应用阶段传递模型数据给顶点着色器时unity支持的语义

| 语义      | 描述                                                        |
| --------- | ----------------------------------------------------------- |
| POSITION  | 模型空间的顶点位置，通常是float4类型                        |
| NORMAL    | 顶点法线，通常是float3类型                                  |
| TANGENT   | 顶点切线，通常是float4类型                                  |
| TEXCOORDn | 顶点的纹理坐标，0表示第一组纹理，通常是float2或者float4类型 |
| COLOR     | 顶点颜色，通常是fixed4或float4类型                          |

从顶点着色器传递数据给片元着色器时的语义

| 语义                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| SV_POSITION         | 裁剪空间中的顶点坐标，结构体中必须包含一个用该语义修饰的变量。等同于dx9中的POSITION |
| COLOR0              | 通常用于输出第一组顶点颜色，但不是必需的                     |
| COLOR1              | 通常用于输出第二组顶点颜色，但不是必需的                     |
| TEXCOORD0~TEXCOORD7 | 通常用于输出纹理坐标，但不是必需的                           |

从片元着色器输出时支持的语义

| 语义      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| SV_Target | 输出值将会存储在渲染目标rendertarget中，等同于dx9的color语义， |



定义复杂的变量类型

```c++
struct v2f{
    float4 pos : SV_POSITION;
    fixed3 color0 : COLOR0;
    fixed4 color1 : COLOR1;
    half value0 : TEXCOORD0;
    float2 value1 : TEXTCOORD1;
}
```

一个语义可以用的寄存器只能处理4个浮点值(float)，如float3X4需要使用更多的空间，可以拆分为多个变量，每个变量存储了矩阵中的一行数据。