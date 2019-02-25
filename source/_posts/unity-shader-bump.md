---
title: unity 凹凸映射
comments: true
tags:
  - unity
  - shader
date: 2019-02-25 09:46:57
---



##### 凹凸映射

凹凸映射的目的是使用一张纹理来修改模型表面的法线，让模型看起来凹凸不平。

##### 高度纹理

存储的是强度值，它用于表示模型表面局部的海拔高度，颜色越浅表明该位置的表面越向外凸起，颜色越深表明该位置越向里凹，可以很直观的从高度图里明确的知道一个模型表面的凹凸情况。缺点是计算更加复杂，在实时计算中不能直接得到表面法线，而需要由像素的灰度值计算而得，需要消耗很多性能。

#####法线纹理

存储的是表面的法线方向，由于法线方向的分量范围在[-1, 1],而像素的分布范围是[0,1]因此需要做一个映射，pixel=（normal + 1)/2 反映射的过程就是norma=pixel x 2 - 1

##### 模型空间的法线纹理

修改后的模型空间中的表面法线存储在一张纹理中，按照模型自带的坐标空间，模型空间
由于每个点存储的法线方向是各异的，所以看起来是**五颜六色**的。
优点：

- 实现简单，更加直观
- 在纹理坐标的缝合处和尖锐的边角部分，可见的缝隙较少，可以提供平滑的边界

##### 切线空间的法线纹理

对于模型的每一个顶点，它都有一个属于自己的切线空间，这个切线空间的原点就是该顶点本身，z轴是顶点的法线方向(n), x轴是顶点的切线方向(t)，y轴可由法线和切线叉乘得到，也被称为副切线b。
切线空间下的法线纹理看起来几乎都是浅蓝色的，纹理其实是存储了各自的切线空间中的法线扰动方向。如果一个点的法线方向不变，那么在它的切线空间中，新的法线方向就是z轴方向(0,0,1)经过映射后存储在纹理中RGB(0.5,0.5,1)为浅蓝色。所以切线空间下的法线纹理看起来是一大片**浅蓝色**的

优点：

- 自由度高。可以用在别的网格中。
- 可以进行UV动画。可以移动一个纹理的UV坐标来实现一个凹凸移动的效果
- 可以重用法线纹理。比如一个砖块可以用一张法线纹理就可以用到所有的6个面上
- 可压缩。z总是法线方向，因此可以只存储xy方向的值。

#####Shader实现

```c#
Shader "My/bump"{
    Properties{
    	_Color("Color", Color)           = (1,1,1,1)
        _MainTex("MainTex", 2D)          = "white"{}
        _BumpMap("Normal Map", 2D)       = "bump" {}
        _BumpScale("Bump Scale", Float)  = 1.0
		_Specular("Specular", Color)     = (1,1,1,1)
        _Gloss("Gloss", Range(8.0, 256)) = 20
    }
    
    SubShader{
        Pass{
            Tag { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Lighting.cginc"
            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            float  _BumpScale;
            fixed4 _Specular;
            float  _Gloss;
            
            struct a2v{
                float4 vertex 	: POSITION;
                float3 normal 	: NORMAL;
                float4 tagent 	: TANGENT; //xyzw w来确定第三个坐标轴的副切线方向性
                float4 texcoord : TEXCOORD0;
            };
            struct v2f{
            	float4 pos  : SV_POSITION;
                float4 uv   : TEXCOORD0;
                float3 lightDir : TEXCOORD1;
                float3 viewDir  : TEXCOORD2;
            };
            
            v2f vert(a2v v){
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //转换到切线空间
                TANGENT_SPACE_ROTATION;
                o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
                o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target{
                fixed3 tangentLightDir = normalize(i.lightDir);
                fixed3 tagentViewDir   = normalize(i.viewDir);
                
                fixed4 packedNormal = tex2D(_BumpMap, i.uv.zw); //得到的是压缩过的法线
                fixed3 tangentNormal;
                tangentNormal = UnpackNormal(packedNormal);     //得到正确的法线
                tangentNormal.xy *= _BumpScale;
                tangentNormal.z = sqrt(1 - saturate(dot(tangentNormal.xy, 	tangentNormal.xy)));
                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));
                fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss);
                return fixed4(ambient + diffuse + specular, 1.0);
            }
            
            ENDCG
        }
        
    }
    FallBack "diffuse"
}

```



世界空间下计算光照模型；需要在片元着色器中把法线方向从切线空间变换到世界空间下。
在顶点着色器计算从切线空间到世界空间的变换矩阵，并把它传递给片元着色器。变换矩阵的计算可以由顶点的切线、副切线和法线在世界空间下的表示来得到，最后在片元着色器中把法线纹理中的法线方向从切线空间变换到世界空间。

```c#
Shader "My/NormalMapWorldSpace"{
    Properties{
    	_Color("Color", Color)           = (1,1,1,1)
        _MainTex("MainTex", 2D)          = "white"{}
        _BumpMap("Normal Map", 2D)       = "bump" {}
        _BumpScale("Bump Scale", Float)  = 1.0
		_Specular("Specular", Color)     = (1,1,1,1)
        _Gloss("Gloss", Range(8.0, 256)) = 20
    }
    
    SubShader{
        Pass{
            Tag { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Lighting.cginc"
            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            float  _BumpScale;
            fixed4 _Specular;
            float  _Gloss;
            
            struct a2v{
                float4 vertex 	: POSITION;
                float3 normal 	: NORMAL;
                float4 tagent 	: TANGENT; //xyzw w来确定第三个坐标轴的副切线方向性
                float4 texcoord : TEXCOORD0;
            };
            struct v2f{
            	float4 pos   : SV_POSITION;
                float4 uv    : TEXCOORD0;
               	float4 TtoW0 : TEXCOORD1;
                float4 TtoW1 : TEXCOORD2;
                float4 TtoW2 : TEXCOORD3;
            };
            
            v2f vert(a2v v){
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP, v.vertex);  //剪裁空间
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
         		float3 worldPos = mul(_Object2World, v.vertex).xyz;
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed3 worldTangent = UnityObjectToWolrdDir(v.tangent.xyz);
                fixed3 worldBinormal = cross(worldNormal, worldTagent) * v.tangent.w;
                
                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target{
              	//得到顶点的世界坐标
       			float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
                fixed3 viewDir  = normalize(UnityWorldSpaceViewDir(worldPos));
                
                //得到切线空间的法线
                fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw));
                bump.xy *= _BumpScale;
                bump.z = sqrt(1 - saturate(dot(bump.xy, bump.xy)));
                bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));
                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(bump, lightDir));
                fixed3 halfDir = normalize(lightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(bump, halfDir)), _Gloss);
                return fixed4(ambient + diffuse + specular, 1.0);
            }
            
            ENDCG
        }
        
    }
    FallBack "diffuse"
}
```




