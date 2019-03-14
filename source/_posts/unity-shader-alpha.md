---
title: unity shader 透明效果
comments: true
tags:
  - unity
  - shader
  
date: 2019-02-25 17:18:03
---



实时渲染要实现透明效果，需要在渲染模型时控制它的透明通道，alpha channel
通常使用两种方法来实现透明效果：

##### 透明度测试 alpha test

只要一个片元的透明度不满足条件(通常是小于某个阈值)，那么它对应的片元就会被舍弃，被舍弃的片元将不会再进行任何处理，也不会对颜色缓冲产生任何影响，否则就按普通不透明物体的处理方式处理

```c#
void clip(float4 x)
{
    if(any ( x < 0))
        discard;
}
```



```c#
Shader "My/AlphaTest"{
    Properties{
        _Color("Main Tint", Color) = (1,1,1,1)
        _MainTex("MainTex", 2D) = "white" ()
        _CutOff("Alpha CutOff", Range(0, 1)) = 0.5
    }
    SubShader{
        Tags{"queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutOut"}
        Pass{
            Tags{"LightMode"="ForwardBase"}
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Lighting.cginc"
           	
            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed _CutOff;
            
             struct a2v{
                float4 vertex   : POSITION;
                float3 normal   : NORMAL;
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f{
                float4 pos  		:  SV_POSITION;
                float3 worldNormal  :  TEXCOORD0;
                float3 worldPos     :  TEXCOORD1;
                float2 uv			:  TEXCOORD2;
            };
            
            v2f vert(a2v v){
                v2f o;
                o.pos         = mul(UNITY_MATRIX_MVP, v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos    = mul(_Object2World, v.vertex).xyz;
                o.uv          = TRANSFORM_TEX(v.texcoord, _MainTex);
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target{
                fixed3 worldNormal   = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                fixed4 texColor      = tex2D(_MainTex, i.uv);
                clip(texColor.a - _Cutoff);
                fixed3 albedo = texColor.rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
                return fixed4(ambient + diffuse, 1.0);
            }
            
            ENDCG
        }
    }
    Fallback "VertexLit"
}
```

##### 透明度混合alpha blending

##### 深度缓冲 z-buffer

##### 渲染队列 render queue

##### 双面渲染原理

把双面渲染工作分成两个Pass：第一个pass只渲染背面，第二个pass只渲染正面，由于Unity会顺序执行Subshader中的各个Pass，因此我们可以保证背面总是在正面被渲染之前渲染，从而保证正确的深度渲染关系。

#####  总结

- 一共有两种方式可以实现，一个是控制透明度测试alphaTest 小于某个值就舍弃，简单粗暴，开启深度写入。
- 透明度混合，可以有两个通道第一个把背面的渲染出来，第二个把正面渲染出来 然后颜色混合,需要关闭深度写入

