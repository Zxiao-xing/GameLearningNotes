- Unity渲染工程师Aras认为将渲染过程分为顶点和像素层面是错误的，是一种不易理解的抽象，不符合人类的思考方式。认为应该分为表面着色器、光照模型、光照着色器层面。表面着色器定义模型表面的反射率、法线和高光，光照模型则选择使用兰伯特还是Blinn-Phong等，光照着色器负责计算光照衰减、阴影等，这样绝大部分时间只需要和表面着色器打交道

### 17.1 表面着色器的一个例子

```c++
Shader "Unity Shaders Book/Chapter 17/Bumped Diffuse"{
	Properties{
		_Color("Main Color", Color) = (1, 1, 1, 1)
		_MainTex("Base (RGB)", 2D) = "white" {}
		_BumpMap("Normalmap", 2D) = "bump" {}
	}

	SubShader{
		Tags { "RenderType" = "Opaque" }
		LOD 300

		CGPROGRAM

		#pragma surface surf Lambert 
		#pragma target 3.0

		sampler2D _MainTex;
		sampler2D _BumpMap;
		fixed4 _Color;

		struct Input{
			float2 uv_MainTex;
			float2 uv_BumpMap;
		}

		void surf(Input IN, inout SurfaceOutput o){
			fixed4 tex = tex2D(_MainTex, IN.uv_MainTex);
			o.Albedo = tex.rgb * _Color.rgb;
			o.Alpha = tex.a * _Color.a;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));
		}

		ENDCG
	}
	FallBack "Legacy Shaders/Diffuse"
}
```

- 表面着色器的CG代码可以直接也必须要写在SubShader块中，Unity会在背后生成多个Pass，可以在Tags设置表面着色器使用的标签
- 表面着色器最重要的两个部分是两个结构体以及其编译指令，结构体是表面着色器中不同函数之间信息传递的桥梁，而编译指令是和Unity沟通的重要手段

### 17.2 编译指令

- 编译质量最重要的作用是指明该表面着色器使用的表面函数和光照函数并设置一些可选参数，表面着色器的CG第一句代码往往是它的编译指令，编译指令的一般格式

  ```c++
  #pragma surface surfaceFunction lightModel [optionalparams]
  ```

  - #pragma surface用于指明该编译指令用于定义表面着色器
  - SurfaceFunction：表面函数
  - lightModel：光照模型
  - optionalparams：可选参数，控制表面着色器的一些行为

#### 17.2.1 表面函数

- 一个对象的表面属性定义了它的反射率、光滑度、透明度等值，而编译指令中的SurfaceFunction就用于定义这些表面属性，其函数格式是固定的：

  ```c++
  void surf(Input IN, inout SurfaceOutput o);
  void surf(Input IN, inout SurfaceOutputStandard o);
  void surf(Input IN, inout SurfaceOutputStandardSpecular o);
  ```

    SurfaceOutput、SurfaceOutputStandard和SurfaceOutputStandardSpecular都是Unity内置的结构体（后两个用于基于物理的渲染），需要配合不同的光照模型使用

  在表面函数中，会使用输入结构体Input IN来设置各种表面属性，并把这些属性存储在输出结构体  SurfaceOutput、SurfaceOutputStandard和SurfaceOutputStandardSpecular中，再传递给光照函数计算光照结果

#### 17.2.2 光照函数

- 光照函数会使用表面函数中设置的各种表面属性来应用某些光照模型，进而模拟物体表面的光照效果。

- Unity内置了基于物理的光照模型函数Standard和StandardSpecular（UnityPBSLighting.cginc），以及简单非基于物理的光照模型函数Lambert和BlinnPhong（LIghting.cginc）

- 也可以定义自己的光照函数，如

  ```c++
  half4 Lighting<Name>(SurfaceOutput s, half3 lightDir, half atten);
  ```

#### 17.2.3 其他可选参数

- 自定义的修改函数。除了表面函数和光照模型外，表面着色器还可以支持其他两种自定义的函数：顶点修改函数和最后颜色修改函数。顶点修改函数允许自定义一些顶点属性，若将顶点颜色传递给表面函数，修改顶点位置，实现某些顶点动画。最后的颜色修改函数则可以在颜色绘制到屏幕前，最后一次修改颜色值，如自定义雾效

- 阴影。可以通过一些指令控制和阴影相关的代码。如addshadow函数为表面着色器生成一个阴影投射的Pass。通常情况下Unity可以直接在FallBack中找到通用的光照模式为ShadowCaster的Pass，从而将物体正确的渲染到深度和阴影纹理中。对于一些定义了顶点动画、透明度测试的物体，就需要对阴影的投射进行特殊处理，来为他们产生正确的阴影

  fullforwardshadows参数可以在前向渲染路径中支持所有光源类型的阴影。默认情况下只支持最重要的平行光的阴影效果，若需要让点光源或聚光灯在前向渲染中也可有阴影就可以添加该参数。若不想对使用该Shader的物体进行任何阴影计算，就可以使用noshadow参数禁用阴影

- 透明度混合和透明度测试：可以通过alpha和alphatest指令来控制透明度混合和透明度测试，如alphatest:VariableName指令会使用名为VariableName的变量来提出不满足条件的片元

- 光照：一些指令可以控制光照对物体的影响。如noambient让Unity不要应用任何环境光照或光照探针。novertexlights参数让Unity不要应用任何逐顶点光照。noforwardadd会去掉所有前向渲染中的额外Pass，即只会支持一个逐像素的平行光。还有用于控制光照烘焙、雾效模拟的参数如nolightmap、nofog等

- 控制代码的生成：一些执行可以控制由表面着色器自动生成的代码，默认情况下Unity会为一个表面着色器生成相应的前向渲染路径、延迟渲染路径使用的Pass，导致生成的Shader文件比较大。可以用exclude_path:defferd、exclude_path:forwar、exclude_path:prepass让Unity不需要为某些渲染路径生成代码

### 17.3 两个结构体

- 表面着色器之间的函数通过两个结构体传递信息：表面函数的输入结构体Input和存储了表面属性的结构体SurfaceOutput/SurfaceOutputStandard/SurfaceOutputStandardSpecular

#### 17.3.1 数据来源：Input结构体

- Input结构体包含了许多表面属性的数据额来源，所以会作为表面函数的输入结构体（若自定义了顶点修改函数，还会是顶点修改函数的输入结构体）。其支持很多内置的变量名，可以告知Unity需要使用的数据信息

  | 变量                              | 描述                                                         |
  | --------------------------------- | ------------------------------------------------------------ |
  | float3 viewDir                    | 包含了视角方向，可用于计算边缘光照                           |
  | 使用COLOR语义定义的float4变量     | 包含了插值后的逐顶点颜色                                     |
  | float4 screenPos                  | 包含了屏幕空间的坐标，可用于反射或屏幕特效                   |
  | float3 worldPos                   | 包含了世界空间下的位置                                       |
  | float3 worldRefl                  | 包含了世界空间下的反射方向。前提是没有修改表面法线           |
  | float3 worldRefl; INTERNAL_DATA   | 若修改了表面法线，需要使用该变量告诉Unity要基于修改后的法线计算世界空间下的反射方向。通过WorldReflectioVector(IN, o.Normal)来得到世界空间下的反射方向 |
  | float3 worldNormal                | 包含了世界空间的法线方向，前提是没有修改表面法线             |
  | float3 worldNormal; INTERNAL_DATA | 若修改了表面法线，需要使用该变量告诉Unity要基于修改后的法线计算世界空间下的法线方向。通过WorldNormalVector(IN, o.Normal)来得到世界空间下的法线方向 |

#### 17.3.2 表面属性：SurfaceOutput结构体

- SurfaceOutput/SurfaceOutputStandard/SurfaceOutputStandardSpecular会作为表面函数的输出，光照函数的输入进行各种光照计算。相比于Input，该结构体的变量是提前声明好的，不可增加也不会减少，没有赋值的就使用默认值
- 若使用非基于物理的光照模型，通常使用SurfaceOutput，若使用了基于物理的光照模型Standard或StandardSpecular，会分别使用SurfaceOutputStandard或SurfaceOutputStandardSpecular结构体。其中SurfaceOutputStandard用于默认的金属工作流程，对应了Standard光照函数。SurfaceOutputStandardSpecular用于高光工作流程，对应StandardSpecular光照函数

### 17.4 Unity背后做了什么

- 以Unity以LightMode为ForwardBase的Pass为例：
  1. 直接将表面着色器中CGPROGMA和ENDCG之间的代码复制，其中包括了对Input结构体、表面函数、光照函数（若自定义）等变量和函数的定义，这些函数和变量在之后的处理过程中被当成正常的结构体和函数进行调用
  2. Unity会分析上述代码，并生成顶点着色器的输入v2f_surf结构体，用于顶点着色器和片元着色器之间进行数据传递。Unity会分析自定义函数中使用的变量，若需要就会在v2f_surf中生成相应变量。若在Input中定义了某些变量但Unity分析在后续代码发现未使用这些变量，则实际上不会再v2f_surf结构体中
  3. 生成顶点着色器
     - 若定义了顶点修改函数，Unity会首先调用顶点修改函数来修改顶点数据，或填充自定义的Input结构体中的变量，然后会分析顶点修改函数中修改的数据，在需要时通过Input结构体将修改结果存储到v2f_surf相应变量中
     - 计算v2f_surf中其他生成的变量值。主要包括顶点位置、纹理坐标、法线方向、逐顶点光照、光照纹理的采样坐标等。也可以通过编译指令来控制某些变量是否需要计算。最后将v2f_surf传递给片元着色器
  4. 生成片元着色器
     - 使用v2f_surf中的对应变量填充Input结构体
     - 调用自定义的表面函数填充SurfaceOutput结构体
     - 调用光照函数得到初始的颜色值，若是内置的Lambert或BlinnPhong光照函数，Unity还会计算动态全局光照，并添加到光照模型计算中
     - 进行其他的颜色叠加
     - 若自定义最后的颜色修改函数，Unity会调用它进行最后的颜色修改

### 17.5 表面着色器实例分析

```c++
Shader "Unity Shaders Book/Chapter 17/Normal Extrusion"{
	Properties{
		_ColorTint("Color Tint", Color) = (1, 1, 1, 1)
		_MainTex("Base (RGB)", 2D) = "white" {}
		_BumpMap("Normal Map", 2D) = "bump" {}
		_Amount("Extrusion Amount", Range(-0.5, 0.5)) = 0.1 
	}
	
	SubShader{
		Tags { "RenderType" = "Opaque" }
		LOD 300

		CGPROGRAM
		// addshadow参数告诉Unity生成该表面着色器对应的阴影投射Pass，默认Unity会为所有渲染路径生成相应pass，使用exclude_path排除相应的路径减少代码量
		// nometa参数取消对提取元数据的Pass的生成
		#pragma surface surf CustomLambert vertex:myvert finalcolor:mycolor addshadow exclude_path:deferred exclude_path:prepass nometa
		#pragma target 3.0

		fixed4 _ColorTint;
		sampler2D _MainTex;
		sampler2D _BumpMap;
		half _Amount;

		struct Input{
			float2 uv_MainTex;
			float2 uvBumpMap;
		};
		// 顶点修改函数，对顶点按法线位置进行膨胀
		void myvert(inout appdata_full v){
			v.vertex.xyz += v.normal * _Amount;
		}
		// 
		void surf(Input IN, inout SurfaceOutput o){
			fixed4 tex = tex2D(_MainTex, IN.uv_MainTex);
			o.Albedo = tex.rgb;
			o.Alpha = tex.a;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_MainTex));
		}
		// 光照函数，简单的兰伯特光照
		half4 LightingCustomLambert(SurfaceOutput s, half3 lightDir, half atten){
			half NdotL = dot(s.Normal, lightDir);
			half4 color;
			color.rgb = s.Albedo * _LightColor0.rgb * (NdotL * atten);
			color.a = s.Alpha;
			return color;
		}
		// 最后颜色修改函数，调整输出颜色
		void mycolor(Input IN, SurfaceOutput o, inout fixed4 color){
			color *= _ColorTint;
		}

		ENDCG
	}
	FallBack "Legacy Shaders/Diffuse"
}
```

### 17.6 Surface Shader的缺点

- 表面着色器只是Unity在顶点/片元着色器上提供的一种封装，是一种更高层的抽象。但任何在表面着色器中完成的事情，都可以在顶点/片元着色器中重现，但反过来不成立
- 表面着色器虽然可以快速实现各种光照效果，但失去了对各种优化和各种特效实现的看公职，且往往会对性能造成一定的影响。对于移动平台虽然提供给了相应的版本，但这些版本的Shader往往只是去掉了二娃IDE逐像素Pass，不计算全局光照和其他一些光照计算上的优化
- 表面着色器无法完成一些自定义的渲染效果

