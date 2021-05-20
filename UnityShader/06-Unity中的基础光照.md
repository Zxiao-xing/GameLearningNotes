### 6.1 我们是如何考到这个世界的

- 模拟真实光照环境来生成一张图像，需要考虑三种物理现象：
  1. 光线从官员中被发射出来
  2. 光线和场景中一些物体相交，一些光被物体吸收了，而另一些光被散射到其他方向
  3. 摄像机吸收了一些光，产生了一张图像

#### 6.1.1 光源

- 使用辐照度来量化光，对平行光来说，辐照度可以通过垂直于面的的单位面积上单位时间内穿过的能量获得

#### 6.1.2 吸收和散射

- 光线和物体相交的结果有两个：
  - 散射：只改变光线的方向，不改变光线的密度和颜色。经过散射后有两种方向：
    1. 折射：散射到物体内部
    2. 反射：散射到物体外部
  - 吸收：只改变光线的密度和颜色
- 使用高光反射部分表示物体表面是如何反射光线
- 使用漫反射部分描述有多少光线被折射、吸收和散射出表面

#### 6.1.3 着色

- 着色：根据材质属性、光源信息，使用一个等式去计算沿该某个观察方向的出射度的过程，该等式称为光照模型

### 6.2 标准光照模型

- 标准光照模型将进入到摄像机的光线分为4个部分，每个部分使用一种方法计算其贡献度：
  - 自发光：描述给定方向一个表面本身会向该方向发射多少辐射量
  - 高光反射：描述当光线从光源照射到模型表面时，该表面会在完全镜面反射方向散射多少能量
  - 漫反射：描述当光线从光源照射到模型表面时，该表面会向每个方向散射多少辐射量
  - 环境光：描述其他所有间接光照

### 6.3 Unity中的环境光和自发光

- 场景中的环境光可以通过Unity内置变量UNITY_LIGHTMODEL_AMBIENT得到环境光的颜色和强度信息

### 6.4 在Unity Shader中实现漫反射光照模型

#### 6.4.1 逐顶点光照

```
// 逐顶点漫反射光照
Shader "Unity Shader Book/Chapter 6/Diffuse Vertex Level"{

	Properties{
		_Diffuse ("Diffuse", Color) = (1, 1, 1, 1)
	}

	SubShader{
		Pass{
			//用于定义该Pass在Unity的光照流水线中的角色
			Tags { "LightMode" = "ForwardBase" }

			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"
			
			fixed4 _Diffuse;
			
			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};

			struct v2f {
				float4 pos : SV_POSITION;
				float3 color : COLOR;
			};

			v2f vert(a2v v) {
				v2f o;

				o.pos = UnityObjectToClipPos(v.vertex);

				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				//unity_WorldToObject：模型空间到世界空间的变换矩阵的逆
				fixed worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));

				fixed worldLight = normalize(_WorldSpaceLightPos0.xyz);
				//saturate：将参数截取到[0,1]
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));

				o.color = ambient + diffuse;

				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				return fixed4(i.color, 1.0);
			}

			ENDCG
		}
	}
	Fallback "Diffuse"
}
```

#### 6.4.2 逐像素光照

```
// 逐像素漫反射光照
Shader "Unity Shader Book/Chapter 6/Diffuse Vertex Level"{

	Properties{
		_Diffuse("Diffuse", Color) = (1, 1, 1, 1)
	}

		SubShader{
			Pass{
				Tags { "LightMode" = "ForwardBase" }

				CGPROGRAM

				#pragma vertex vert
				#pragma fragment frag

				#include "Lighting.cginc"

				fixed4 _Diffuse;

				struct a2v {
					float4 vertex : POSITION;
					float3 normal : NORMAL;
				};

				struct v2f {
					float4 pos : SV_POSITION;
					float3 worldNormal : TEXCOORD0;
				};

				v2f vert(a2v v) {
					v2f o;

					o.pos = UnityObjectToClipPos(v.vertex);

					o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

					return o;
				}

				fixed4 frag(v2f i) : SV_Target{
					fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
					
					fixed3 worldNormal = normalize(i.worldNormal);

					fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

					fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));

					fixed3 color = ambient + diffuse;

					return fixed4(color, 1.0);
				}

				ENDCG
			}
	}
		Fallback "Diffuse"
}
```

#### 6.4.3 半兰伯特模型

- 上述漫反射光照模型也被称为兰伯特光照模型，其符合兰伯特定律：在平面某点漫反射光的光强与该反射点的法向量和入射光角度的余弦值成正比

  兰伯特模型无法解决背光面明暗一样的问题

- 半兰伯特模型：
  $$
  c_{diffuse} = （c_{light}\dot{}m_{diffuse}）(\alpha(\hat{n}\dot{}\hat{l}) + \beta)
  $$
  即对结果进行了alpha倍的缩放加beta偏移，大多数情况下两者均为0.5。将[-1,1]范围映射到[0,1]

### 6.5 在Unity Shader中实现高光反射光照模型

#### 6.5.1 逐顶点光照

```
Shader "Unity Shader Book/Chapter 6/Specular VertexLevel"{

	Properties{
		_Diffuse("Diffuse", Color) = (1, 1, 1, 1)
		_Specular("Specular", Color) = (1, 1, 1, 1)
		_Gloss("Gloss", Range(8.0, 256)) = 20
	}

	SubShader{
		Pass{
			Tags {"LightMode" = "ForwardBase"}

			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};

			struct v2f {
				float4 pos : SV_POSITION;
				float3 color : COLOR;
			};

			v2f vert(a2v v) {
				v2f o;

				o.pos = UnityObjectToClipPos(v.vertex);

				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
				fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(unity_ObjectToWorld, v.vertex).xyz);
				fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir,viewDir)),_Gloss);
				o.color =  ambient + diffuse + specular;

				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				return fixed4(i.color, 1.0);
			}
			ENDCG
		}
	}
	Fallback "Specular"
}
```

#### 6.5.2 逐像素光照

```
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unity Shader Book/Chapter 6/Specular PixelLevel"{
	Properties{
		_Diffuse("Diffuse", Color) = (1, 1, 1, 1)
		_Specular("Specular", Color) = (1, 1, 1, 1)
		_Gloss("Gloss", Range(8.0, 256)) = 20
	}
	
	SubShader{
		Pass{
			Tags {"LightMode" = "ForwardBase"}
			
			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

			struct a2v{
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};

			struct v2f{
				float4 pos : SV_POSITION;
				float3 worldNormal : TEXCOORD0;
				float3 worldPos : TEXCOORD1;
			};

			v2f vert(a2v v) {
				v2f o;

				o.pos = UnityObjectToClipPos(v.vertex);

				o.worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));

				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				return o;
			}

			fixed4 frag(v2f i) : SV_Target {
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(i.worldNormal, worldLightDir));

				fixed3 reflectDir = normalize(reflect(-worldLightDir, i.worldNormal));

				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);

				fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);

				fixed3 color = ambient + diffuse + specular;

				return fixed4(color, 1.0);
			}

			ENDCG
		}
	}
	FallBack "Specular"

}
```

#### 6.5.3 Blinn-Phong光照模型

```
Shader "Unity Shader Book/Chapter 6/BlinnPhone"{
	Properties{
		_Diffuse("Diffuse", Color) = (1, 1, 1, 1)
		_Specular("Specular", Color) = (1, 1, 1, 1)
		_Gloss("Gloss", Range(8.0, 256)) = 20
	}

	SubShader{
		pass{
			Tags { "LightMode" = "ForwardBase" }

			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"

			fixed4 _Diffuse;
			fixed4 _Specular;
			float _Gloss;

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

				o.pos =  UnityObjectToClipPos(v.vertex);
				o.worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				return o;
			}

			fixed4 frag(v2f i) : SV_Target{
				
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				fixed3 diffuse = _LightColor0.xyz * _Diffuse.xyz * saturate(dot(i.worldNormal, worldLightDir));

				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz -  i.worldPos.xyz);
				fixed3 halfDir = normalize(worldLightDir + viewDir);

				fixed3 specular = _LightColor0.xzy * _Specular.xyz * pow( saturate(dot(i.worldNormal, halfDir)), _Gloss);
			
				fixed3 color = ambient + diffuse + specular;
				return fixed4(color, 1.0);
			}

			ENDCG
		}
	}
	FallBack "Specular"

}
```

### 6.6 使用Unity内置的函数

- UnityCG.cginc中一些常用的帮助函数

  | 函数名                                       | 描述                                                         |
  | -------------------------------------------- | ------------------------------------------------------------ |
  | float3 WorldSpaceViewDir(float4 v)           | 输入一个模型空间中的顶点位置，返回世界空间中从该点到摄像机的观察方向。内部实现使用了UnityWorldSpaceViewDir |
  | float3 UnityWorldSpaceViewDir(float4 v)      | 输入一个世界空间中的顶点位置，返回世界空间中从该点到摄像机的观察方向 |
  | float3 ObjSpaceViewDir(float4 v)             | 输入一个模型空间中的顶点位置，返回模型空间中从该点到摄像机的观察方向 |
  | float3 WorldSpaceLightDir(float4 v)          | 仅可用于前向渲染。输入一个模型空间中的顶点位置，返回世界空间中从该点到光源的光照方向。内部实现使用了UnityWorldSpaceLightDir |
  | float3 UnityWorldSpaceLightDir(float4 v)     | 仅可用于前向渲染。输入一个世界空间中的顶点位置，返回世界空间中从该点到光源的光照方向 |
  | float3 ObjSpaceLightDir(float4 v)            | 仅可用于前向渲染。输入一个模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向 |
  | float3 UnityObjectToWorldNormal(float3 norm) | 把法线方向从模型空间转换到世界空间中                         |
  | float3 UnityObjectToWorldDir(float3 dir)     | 把方向矢量从模型空间变换到世界空间中                         |
  | float3 UnityWorldToObjectDir(float3 dir)     | 把方向矢量从世界空间变换到模型空间中                         |

  函数没有保证得到的是单位矢量，所以在使用前应进行归一化。

  中间三个函数只能用于前向渲染，因为只有前向渲染这些函数内的光的位置变量才会被正确赋值

