---
title: unity-shader-texture
comments: true
tags:
  - unity
  - shader
  - texture
date: 2019-02-22 11:13:51
---



#####纹理采样

纹理名_ST，其中ST是缩放scale和平移translation的缩写。
_MainTex_ST可以让我们得到该纹理的缩放和平移(偏移)值。
_MainTex_ST.xy存储的是缩放值， _MainTex_ST.zw存储的是偏移值。
在unity中可以在材质面板中调节纹理的平铺tiling(缩放)和偏移offset{平移}
TRANSFORM_TEX将会根据纹理计算变换后的纹理坐标
tex2D函数进行纹理采样

#####纹理的属性

属性调整面板设置纹理类型 wrap mode repeat和clamp 
##### 滤波模式FilterMode
支持三种模式Point Bilinear Trilinear得到的图片滤波效果依次提升，性能消耗越大，会影响
放大或缩小时得到的图片质量。比如把64x64纹理贴在512x512大小平面时，需要放大纹理。

1. Point模式使用最近邻滤波nearest neighbor，在放大或者缩小时，它的采样像素数目通常只有一个
2. Bilinear滤波使用了线性滤波，对于每个目标像素，它会找到四个邻近像素，然后进行线性插值混合后得到最终像素，因此图像看起来模糊了
3. Trilinear



##### 放大纹理

##### 缩小纹理

##### 多级渐远纹理mipmapping 

在一个很小空间里有很多东西。是提前用滤波处理得到很多更小的图像，随着远离摄像机，使用较小的纹理。这样可以通过增加空间提高性能。如果在导入纹理时设置了这个GenerateMipMaps，将会生成多级渐远纹理。

##### Shader示例

```c
//普通纹理采样
Shader "shader/texture"{
    Properties{
        _MainTex("MainTex", 2D) = "white"{}
        _Color("MainColor", Color)  = (1,1,1,1)
        _Specular("Specular", Color) =(1,1,1,1)
        _Gloss("Gloss", Range(8,256)) = 20
    }
    SubShader{
        Pass{
            Tags {"LightMode"="ForwardBase"}
            CGPROGRAM
                #pragma vertex vert
                #pragma fragment frag
                #include "Lighting.cginc"
                
                fixed4    _Color;
            	sampler2D _MainTex;
            	fixed4	  _Specular;
            	float     _Gloss;
            	float4    _MainTex_ST;
            
            struct a2v {
                float4 vertex      : POSITION;
                float3 normal      : NORMAL;
                float4 texcoord    : TEXCOORD0;
            };
            struct v2f{
            	float4 pos 		   : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos    : TEXCOORD1;
                float4 uv		   : TEXCOORD2;
            };
            
            v2f vert(a2v v){
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(_Object2World, v.vertex).xyz;
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                return o;
            }
            
            float4 frag(v2f i) : SV_Target
            {
                //归一化
                fixed3  worldNormal = normalize(i.worldNormal);
                fixed3  worldLightDir = normalize(UnityWorldLightDir(i.worldPos));
                //
                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(worldNormal, halfDir)), _Gloss);
                return fixed4(ambient + diffuse + specular, 1.0);
            }
            
            CGEND
        }
        FallBack "Specular"
    }
}
```



