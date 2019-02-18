---
title: Unity Shader基础
comments: true
tags:
  - unity
  - shader
date: 2019-02-18 14:02:25
---



##### Shader和材质
Shader必须赋值给材质球Material才能达到效果，材质球比如有shader才能显示出效果。
Unity可以创建的Shader有四种：

1. Standard Surface Shader
   包含标准光照模型的表面着色器模板

2. Unlit Shader
   不包含光照但包含雾效的基本的顶点片元着色器

3. Image Effect Shader
   实现各种屏幕后处理效果的一个基本模板

4. Compute Shader
   利用GPU的并行性来进行一些与常规渲染流水线无关的计算

##### ShaderLab语言

UnityShader都是使用ShaderLab来编写的，其语法Syntax如下：

1. shader名称
   Shader “Custom/Myshader”

2. SubShader
   至少包含一个SubShader
   有可选标签[Tags]和状态[RenderSetup]和通道Pass，Pass定义了一次完整的渲染流程，如果	pass数量多渲染性能下降

   - 渲染状态
     用于设置显卡的各种状态比如是否开启混合、深度测试，是否开启混合模式，是否开启深度写入

     

     | 状态名称 | 设置指令                                                     | 解释                               |
     | -------- | ------------------------------------------------------------ | ---------------------------------- |
     | Cull     | Cull Back \| Front \|Off                                     | 设置剔除模式：剔除背面，正面，关闭 |
     | ZTest    | ZTest Less Greater \| LEqual \| GEqual \| NotEqual \| Always | 设置深度测试时使用的函数           |
     | ZWrite   | ZWrite On \| Off                                             | 开启/关闭深度写入                  |
     | Blend    | Blend SrcFactor DstFactor                                    | 开启并设置混合模式                 |

   - 标签
     标签就是一些键值对（Key/Value Pair）是字符串类型，这是SubShader和渲染引擎之间的沟通桥梁，它用来告诉unity的渲染引擎希望怎样和何时来渲染这个对象。格式是Tags { “TagName1” = “Value1” “TagName2” = “Value2”}

     

     | 标签类型             | 说明                                                         | 例子                                  |
     | -------------------- | ------------------------------------------------------------ | ------------------------------------- |
     | Queue                | 控制渲染顺序，指定该物体属于哪一个渲染队列，通过这种方式可以保证所有的透明物体可以在所有不透明物体后面被渲染，也可以自定义渲染队列来控制渲染顺序.Backgroud1000 Geometry2000 AlphaTest 2450Transparent3000 Overlay | Tags { “Queue” = “Transparent”}       |
     | RenderType           | 对着色器进行分类，例如这是一个不透明的着色器或者是一个透明着色器，可以用于着色器替换功能 | Tags{“RenderType” = “Opaque”}         |
     | DisableBatching      | 指明是否对SubShader使用Unity的批处理功能                     | Tags{“DisableBatching” = “True”}      |
     | ForceNoShadowCasting | 控制使用该SubShader的物体是否会投射阴影                      | Tags{“ForceNoShadowCasting” = “True”} |
     | IgnoreProjector      | 使用该SubShader的物体将不会受Projector的影响，常用于半透明物体 | Tags{“IgnoreProjector” = “True”}      |
     | CanUseSpriteAtlas    | 当SubShader用于精灵sprites时，将该标签设为False              | Tags{“CanUseSpriteAtlas” = “False”}   |
     | PreviewType          | 指明材质面板将如何预览该材质，默认情况是下个材质球，可以通过把该标签设置为“Plane”，“SkyBox”来改变预览类型 | Tags{“PreviewType” = “Plane”}         |

3. Pass

   - Pass标签类型

     

   | 标签类型       | 说明                                                         | 用法                                     |
   | -------------- | ------------------------------------------------------------ | ---------------------------------------- |
   | LightMode      | 定义该通道在Unity的渲染流水线中的角色                        | Tags{“LightMode”=“ForwardBase”}          |
   | RequireOptions | 用于指定当满足某些条件时才渲染该Pass，他的值是有空格分隔的字符串 | Tags{“RequireOptions”=“SoftVegetation”}  |
   | 其他           | UsePass复用其他Pass                                          | GrabPass抓取屏幕并将结果存储在一张纹理中 |

   

   - pass语义块
     Pass{

     ​	Name “MyPass”

     ​	Tags{“LightMode”=“ForwardBase”}
     ​	UsePass “MyShader/SomePass”

   ​	}

   

4. Fallback
   Fallback “name” Fallback Off 使用更低级的shader吧

   

5. PropertyType属性类型和properties属性

   

   | 属性类型        | 默认值的语法定义                 | 例子                                   |
   | --------------- | -------------------------------- | -------------------------------------- |
   | Int             | number                           | _Int(“Int”, Int) = 2                   |
   | Float           | number                           | _Float(“Float”, Float) = 1.5           |
   | Range(min, max) | number                           | _Range(“Range”, Range(0.0, 5.0)) = 4.0 |
   | Color           | (number, number, number, number) | _Color(“Color”, Color) = (1,1,1,1)     |
   | Vector          | (number, number, number, number) | _Vector(“Vector”, Vector) = (1,2,3,1)  |
   | 2D              | “defaulttexture” {}              | _2D(“2D”, 2D) = “ ” {}                 |
   | Cube            | “defaulttexture” {}              | _Cube(“Cube”, Cube) = “white” {}       |
   | 3D              | “defaulttexture” {}              | _3D(“3D”, 3D) = “black” {}             |

   


