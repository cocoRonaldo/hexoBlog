---
title: 光 光照模型 漫反射
comments: true
tags:
  - unity
  - shader
date: 2019-02-21 11:07:15
---



[TOC]



#### 光

1. 方向 l表示方式
2. 辐射度 irradiance, 垂直于l的单位面积上单位时间内穿过的能量就是辐射度
3. 颜色

##### 吸收absorption

不改变 方向
改变 光的密度和颜色

##### 散射scattering

改变光的方向
不改变光的颜色和密度

1. 折射 refraction(透射transmission)     通过漫反射diffuse来模拟，有多少光线被折射、吸收、和散射出表面
2. 反射 reflection                                      通过高光反射来反应模拟specular

##### 出射度

辐射度和出射度之间满足一定线性关系，这个比值就是材质的漫反射和高光反射属性



#### 标准光照模型

通常光有以下四部分组成

1. 自发光emissive  $c_{emissive}$  直接用了该材质的颜色 $c_{missive}=m_{emissive}$  
2. 高光反射specular $c_{specular}​$
3. 漫反射diffuse $c_{diffuse}$ 是那些被物体表面随机散射到各个方向的辐射度，各个位置都有完全随机，用兰伯特定律来模拟：反射光线的强度和表面法线和光源方向之间的夹角的余弦值成正比$c_{diffuse}=(c_{light}·m_{diffuse})max(0,n·I)$ n为表面法线，I是指向光源的单位矢量，m_diffuse漫反射颜色，c_light光源颜色
4. 环境光ambient  $c_{ambient}$  近似模拟间接光照  $c_{ambient}=g_{ambient}$ 内置变量UNITY_LIGHTMODEL_AMBIENT可以得到环境光的颜色和强度信息

##### Phong模型泊松模型

1. 表面法线 n

2. 入射角 I

3. 反射角 r

4. 视角方向 v

这是模型基础，只要知道一个就可以求得其他a·b=|a||b|cosa

r+I=2n(n·I) 所以r=2n(n·I)-I  

泊松模型 :

$C_{specular} = C_{light}m_{specular}max(0, v·r)^{m_{gloss}}​$

其中m_gloss为光泽度也被成为反光度，用于控制高光区域的亮点，m_gloss越大，亮点越小
m_specular是材质的高光反射颜色用于控制该材质对于高光反射的强度和颜色
c_light是光源的颜色和强度

##### Blinn模型

思路是避免求反射角引入 视角和入射角相加再归一化h=(v+I)/|v+i|,然后使用n和h的夹角来计算高光
$C_{specular} = C_{light}m_{specular}max(0, v·h)^{m_{gloss}}$

##### 半兰伯特模型

适用于漫反射光照模型，在平面某点漫反射光的光强与该反射点的法向量和入射光角度的余弦值成正比,这样在光照无法到达的区域，模型的外观通常是全黑的，没有任何明暗变化，是去了模型细节表现，为了改善这种情况，加了两个参数
cdiffuse = clight mdiffuse (a(ni) + b), 当a和b都等于0.5时候就是半兰伯特光照模型



```csharp
Shader "My/specular-pixel_level"
{
    Properties {
        _Diffuse("Diffuse", Color) = (1,1,1,1)
        _Specular("Specular", Color) = (1,1,1,1)
        _Gloss("Gloss", Range(8,256)) = 20
    }

    SubShader{
        Pass{
            Tag{"LightMode"="ForwardBase"}

            CGPROGRAM
                #pragma vertex vert
                #pragma fragment frag

                #include "Lighting.cginc"

                fixed4 _Diffuse;
                fixed4 _Specular;
                float  _Gloss;

                struct a2v{
                    float4 vertex : POSITION;
                    float3 normal : NORMAL;
                };

                struct v2f{
                    float4 pos : SV_POSITION;
                    float3 worldNormal : TEXCOORD0;
                    float3 worldPos : TEXCOORD1;
                };

                v2f vert(a2v v){
                    v2f o;
                    o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                    o.worldNormal = mul(v.normal, _World2Object);
                    o.worldPos = mul(_Object2World, v.vertex).xyz;
                    return o;
                }

                fixed4 frag(v2f i) : SV_Target{
                                     
                    fixed3 ambient = UNITY_LIGHTMODE_AMBIENT.xyz;

                    //计算法线
                    fixed3 worldNormal = normalize(mul(v.normal, 	(float3x3)_World2Object));
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                    //计算漫反射
                    fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
                    //计算高光反射
                    fixed3 reflectDir = normalize(reflect(-WorldLightDir, worldNormal));
                    //得到观察视角
                    fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(_Object2World, v.vertex).xyz);
                    fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
                    o.color = ambient + diffuse + specular;
                    
                    
                    return fixed4(i.color, 1.0);
                }

            ENDCG
        }
    }
    Fallback "specular"
}
```



