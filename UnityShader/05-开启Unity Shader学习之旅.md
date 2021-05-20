### 5.2 一个最简单的顶点/片元着色器

#### 5.2.1 顶点/片元着色器的基本结构

```
Shader "Unity Shader Book/Chapter5/SimpleShader"
{
    SubShader{
        Pass{
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            float4 vert(float4 v : POSITION) : SV_POSITION{
            // 更新提示: 'mul(UNITY_MATRIX_MVP,*)' 替换为 'UnityObjectToClipPos(*)'
                return mul(UNITY_MATRIX_MVP, v);
            }
            fixed4 frag() : SV_Target{
                return fixed4(0.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

- #pragma	vertex	name和#pragma	fragment	name指定了哪个函数包含了顶点着色器，哪个函数包含了片元着色器
- 顶点函数是逐顶点执行的，其输入v包含了顶点的位置，通过POSITIN语义指定，返回值为float4类型的变量，它是在该顶点的裁剪空间中的位置。
- POSITION和SV_POSITION都是Cg/HLSL中的语义，是不可忽略的，这些语义将告诉系统用户需要哪些值以及用户的输入。POSITION告诉Unity把模型的顶点坐标填充到输入参数v中，SV_POSITION将告诉Unity顶点着色器的输出是裁剪空间中的顶点坐标
- UNITY_MATRIX_MVP是Unity内置的MVP矩阵
- SV_Target是HLSL中的一个系统语义，其告诉渲染器将用户的输入颜色存储到一个渲染目标中

#### 5.2.2 模型数据从哪里来

```
Shader "Unity Shader Book/Chapter5/SimpleShader"
{
    SubShader{
        Pass{
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag
            //applicaton to vertex shader
            struct a2v {
                //POSITION：使用模型空间的顶点坐标填充vertex变量
                float4 vertex : POSITION;
                //NORMAL：使用模型空间的法线向量填充normal变量
                float3 normal : NORMAL;
                //TEXCOORD0：使用模型的第一套纹理坐标填充texcoord变量
                float4 texcoord : TEXCOORD0;
            };

            float4 vert(a2v v) : SV_POSITION{
            	//对输入的顶点坐标进行mvp变换
                return UnityObjectToClipPos(v.vertex);
            }
            fixed4 frag() : SV_Target{
                return fixed4(0.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

- 对顶点着色器的输入，Unity支持的语义有：POSITION，TANGENT，NORMAL，TEXCOORDx，COLOR等，它们的数据是由材质的MeshRender组件提供，在每帧调用DrawCall时，MeshRender组件会把其负责渲染的模型数据发给UnityShader

#### 5.2.3 顶点着色器和片元着色器间如何通信

```
Shader "Unity Shader Book/Chapter5/SimpleShader"
{
    SubShader{
        Pass{
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag
            struct a2v {
                //POSITION：使用模型空间的顶点坐标填充vertex变量
                float4 vertex : POSITION;
                //NORMAL：使用模型空间的法线向量填充normal变量
                float3 normal : NORMAL;
                //TEXCOORD0：使用模型的第一套纹理坐标填充texcoord变量
                float4 texcoord : TEXCOORD0;
            };
            struct v2f {
                //SV_POSITION：包含了顶点在裁剪空间的位置信息
                float4 pos : SV_POSITION;
                //COLOR0语义可以用于存储颜色信息
                float3 color : COLOR0;
            };

            v2f vert(a2v v){
                //声明输出结构
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.color = v.normal * 0.5 + fixed3(0.5, 0.5, 0.5);
                return o;
            }
            fixed4 frag(v2f i) : SV_Target{
                    return fixed4(i.color, 1.0);
            }

            ENDCG
        }
    }
}
```

#### 5.2.4 如何使用属性

```
Shader "Unity Shader Book/Chapter5/SimpleShader"
{
	Properties{
		_Color("Color Tint", Color) = {1.0, 1.0, 1.0, 1.0}
	}
    SubShader{
        Pass{
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };
            struct v2f {
                float4 pos : SV_POSITION;
                float3 color : COLOR0;
            };

            v2f vert(a2v v){
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.color = v.normal * 0.5 + fixed3(0.5, 0.5, 0.5);
                return o;
            }
            fixed4 frag(v2f i) : SV_Target{
				fixed3 c = i.Color;
				c *= _Color.rgb;
                return fixed4(c, 1.0);
            }

            ENDCG
        }
    }
}
```

- ShaderLab属性类型和Cg变量类型的匹配关系

  | ShaderLab     | Cg变量类型            |
  | ------------- | --------------------- |
  | Color, Vector | float4, half4, fixed4 |
  | Range, Float  | float, half, fixed    |
  | 2D            | sampler2D             |
  | Cube          | samplerCube           |
  | 3D            | sampler3D             |

### 5.3 强大的援手：Unity提供的内置文件和变量

#### 5.3.1 内置的包含文件

- 包含文件类似于C++头文件的一种文件，文件后缀是.cginc，编写Shader时可以用#include把这些文件包含进来，这样可以使用Unity提供的一些非常有用的变量或帮助函数

- 可以在官方网站下载内置着色器来直接下载这些文件。

  - CGIncludes包含了所有的内置包含文件
  - DefaultResources包含了一些内置组件或功能所有要的UnityShader
  - DefaultResourcesExtra包含所有Unity中内置的UnityShader
  - Editor包含一个脚本文件，用于定义Standard Shader所用的材质面板

- CGIncludes中主要的包含文件以及主要用途

  | 文件名                     | 描述                                                         |
  | -------------------------- | ------------------------------------------------------------ |
  | UnityCG.cginc              | 包含了最常用的帮助函数、宏、结构体                           |
  | UnityShaderVariables.cginc | 在编译Unity Shader时被自动包含，包含很多内置的全局变量如UNITY_MATRIX_MVP |
  | Lighting.cginc             | 包含了各种内置的光照模型，若编写Surface Shader会自动包含     |
  | HLSLSupport.cginc          | 在编译Unity Shader被自动包含，声明了很多用于跨平台编译的宏和定义 |

- UnityCG.cginc一些常用的结构体

  | 名称         | 描述                   | 包含的变量                                       |
  | ------------ | ---------------------- | ------------------------------------------------ |
  | appdata_base | 可用于顶点着色器的输入 | 顶点位置、顶点法线、第一组纹理坐标               |
  | appdata_tan  | 可用于顶点着色器的输入 | 顶点位置、顶点切线、顶点法线、第一组纹理坐标     |
  | appdata_full | 可用于顶点着色器的输入 | 顶点位置、顶点切线、顶点法线、四组或更多纹理坐标 |
  | appdata_img  | 可用于顶点着色器的输入 | 顶点位置、第一组纹理坐标                         |
  | v2f_img      | 可用于顶点着色器的输出 | 裁剪空间中的位置、纹理坐标                       |

- UnityCG.cginc中一些常用的帮助函数

  | 函数名                                       | 描述                                                         |
  | -------------------------------------------- | ------------------------------------------------------------ |
  | float3 WorldSpaceViewDir(float4 v)           | 输入一个模型空间的顶点位置，返回世界空间中从该点到摄像机的观察方向 |
  | float3 ObjSpaceViewDir(float4 v)             | 输入一个模型空间中的顶点位置，返回模型空间中从该点到摄像机的观察方向 |
  | float3 WorldSpaceLightDir(float4 v)          | 仅可用于前向缓冲中。输入一个模型空间中的顶点位置，返回世界空间中从该点到光源的光照方向，未归一化 |
  | float3 ObjSpaceLightDir(float4 v)            | 仅可用于前向渲染中。输入一个模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向，没有归一化 |
  | float3 UnityObjectToWorldNormal(float3 norm) | 把法线方向从模型空间转换到世界空间中                         |
  | float3 UnityObjectToWorldDir(float3 dir)     | 把方向矢量从模型空间变换到世界空间中                         |
  | float3 UnityWorldToObjectDir(float3 dir)     | 把方向矢量从世界空间变换到模型空间中                         |


#### 5.3.2 内置的变量

- Unity提供了用于访问时间、光照、雾效和环境光等目的的变量，大多数位于UnityShaderVariables.cginc中。与光照相关的变量还会位于Lightning.cginc、AutoLight.cginc等文件中

### 5.4 Unity提供的Cg/HLSL语义

#### 5.4.1 什么是语义

- 语义实际上是一个赋给Shader输入和输出的字符串，其表达了参数的含义，让Shader知道从哪里读取数据，并把数据输出到哪里，他们在Cg/HLSL的Shader流水线是不可或缺的
- Unity并没有支持所有的语义
- 通常情况下，输入输出变量并不需要有特别的意义，可以自行决定这些变量的用途。Unity为了方便对模型数据的传输，对一些语义进行了特别的含义规定，如顶点着色器中输入变量TEXCOORD0，Unity会识别并将模型的第一组纹理坐标填充进去，但对于片元着色器则没有该限定，用户可以自己决定
- 在DirectX10以后有一种新的语义类型，即系统数值语义，以SV（sytem-value）开头，它们在渲染流水线中有特殊含义，描述的变量是不可以随便赋值的，流水线需要使用它们完成特定的目的。在大多数平台上，很多带SV和不带SV语义是等价的，但在某些平台必须用SV修饰，否则无法工作，所以

#### 5.4.2 Unity支持的语义

- 从应用阶段传递模型数据给顶点着色器时Unity使用的常用语义，Unity内部赋予了它们特殊的含义

  | 语义      | 描述                                   |
  | --------- | -------------------------------------- |
  | POSITION  | 模型空间中的顶点位置，通常为float4类型 |
  | NORMAL    | 顶点法线，通常是float3类型             |
  | TANGENT   | 顶点切线，通常是float4类型             |
  | TEXCORRDn | 该顶点的第n组纹理坐标                  |
  | COLOR     | 顶点颜色，通常是fixed4或float4类型     |

- 从顶点着色器传递数据给片元着色器时Unity支持的常用语义

  | 语义                | 描述                                                         |
  | ------------------- | ------------------------------------------------------------ |
  | SV_POSITION         | 裁剪空间中的顶点坐标，结构体中必须包含一个用该语义修饰的变量。等同于DirectX9中的POSITION，但最好使用SV_POSITION |
  | COLOR0              | 通常用于输出第一组顶点颜色，但不是必须的                     |
  | COLOR1              | 通常用于输出第二组顶点颜色，但不是必须的                     |
  | TEXCOORD0~TEXCOORD7 | 通常用于输出纹理坐标，但不是必须的                           |

- Unity支持的片元着色器的输出语义

  | 语义      | 描述                                                         |
  | --------- | ------------------------------------------------------------ |
  | SV_Target | 输出值将会存储到渲染目标中，等同于DirectX9中的COLOR语义，但最好使用SV_Target |

### 5.5 程序员的烦恼：Debug

#### 5.5.1 使用假彩色图像

- 假彩色图像：指的是用假彩色技术生成的一种图像，其对应的是照片这种真彩色图像。一张加彩色图像可以用于可视化一些数据
- 主要思想是把需要调试的变量映射到[0,1]，把它们作为颜色输出到屏幕上，然后通过屏幕上显式的像素颜色来判断这个值是否正确。由于颜色的分量范围在[0, 1]，因此需要小心处理需要调试的变量的范围，若已知它的值域范围，可以把它映射到[0, 1]之间再进行输出，若不知道一个变量的范围，则只能不停的实验，任何大于1的值会被设为1，小于0的值设为0

```
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Unity Shaders Book/Chapter 5/False Color"
{
    SubShader
    {
        Pass{
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct v2f {
                float4 pos : SV_POSITION;
                fixed4 color : COLOR0;
            };
            //appdata_full在UnityCG.cginc中，几乎包含了所哟逇模型数据
            v2f vert(appdata_full v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                //可视化法线方向
                //o.color = fixed4(v.normal * 0.5 + fixed3(0.5, 0.5, 0.5), 1.0);
                //可视化切线方向
                o.color = fixed4(v.tangent.xyz * 0.5 + fixed3(0.5, 0.5, 0.5), 1.0);
                //可视化副切线方向
                fixed3 binormal = cross(v.normal, v.tangent.xyz) * v.tangent.w;
                o.color = fixed4(binormal * 0.5 + fixed3(0.5, 0.5, 0.5), 1.0);
                //可视化第一组纹理坐标
                o.color = fixed4(v.texcoord.xy, 0.0, 1.0);
                //可视化第二组纹理坐标
                o.color = fixed4(v.texcoord1.xy, 0.0, 1.0);
                //可视化第一组纹理坐标的小数部分
                o.color = frac(v.texcoord);
                if (any(saturate(v.texcoord) - v.texcoord)) {
                    o.color.b = 0.5;
                }
                o.color.a = 1.0;
                //可视化第二组纹理坐标的小数部分
                o.color = frac(v.texcoord1);
                if (any(saturate(v.texcoord1) - v.texcoord1)) {
                    o.color.b = 0.5;
                }

                return o;
            }

            fixed frag(v2f i) : SV_Target{
                return i.color;
            }

            ENDCG

        }
    
    }
}
```

### 5.6 小心：渲染平台的差异

- Unity的有点之一是其强大的的跨平台性，写一份代码可以运行在很多平台上，绝大多数情况下，Unity隐藏了这些细节，但有时需要自己处理它们

#### 5.6.1 渲染纹理的坐标差异

- OpenGL和DirectX在水平方向上数值的变化方向是相同的，但在竖直方向上两者是相反的，OpenGL中（0，0）对应了屏幕左下角，但在DirectX对应左上角。大多数情况下该差异不会造成影响，但要使用渲染到纹理技术时，把屏幕图像渲染到一张纹理中时，若不采取任何措施的话，就会出现纹理翻转的情况，Unity在背后处理了这种翻转问题
- 若开启了抗锯齿（Edit->Project Settings->Quality->Anti Aliasing）并在此时使用了渲染到纹理技术，该情况下Unity先渲染得到屏幕图像，由硬件进行抗锯齿处理后得到一张渲染纹理供用户进行后续处理，此时在DirectX平台下得到的输入屏幕图像并不会被Unity翻转，因为对屏幕图像的采样坐标是需要符合DirectX平台规定的。若屏幕特效只需要处理一张渲染图像，仍然不需要在意纹理的翻转问题，在代用Graphics.Blit函数时，Unity已经对屏幕图像的采样坐标进行了处理，只需要按正常的采样过程处理屏幕图像即可。若同时处理多张渲染图像，这些图像在竖直方向的朝向可能是不同的，需要在顶点着色器中翻转某些渲染纹理的纵坐标，使之都符合DirectX平台的规则

#### 5.6.2 Shader的语法差异

- DirectX 9/11对Shader的语义更加严格，如初始化float4类型需要提供完全部参数，而不是提供一个参数。在表面着色器中，顶点函数没有对out参数的所有成员变量都进行初始化。且DirectX9/11也不支持在顶点着色器中使用tex2D函数，而使用tex2Dlod函数替代，且添加#pragma target 3.0，因为其是Shader Model 3.0的特性

#### 5.6.3 Shader的语义差异

- 应尽可能使用SV_POSITION来描述顶点着色器输出的顶点位置，使用SV_Target来描述片元着色器的输出颜色

### 5.7 Shader整洁之道

#### 5.7.1 float、half还是fixed

- Cg/HLSL中3种精度的数值类型

  | 类型  | 精度                                                     |
  | ----- | -------------------------------------------------------- |
  | float | 最高精度的浮点数，通常使用32位来存储                     |
  | half  | 中等精度的浮点值，通常使用16位来存储，范围是-60000-60000 |
  | fixed | 最顶精度的浮点数，通常使用11位来存储，范围是-2.0-2.0     |

  在不同平台和GPU上实际精度可能不同

  - 大多数桌面GPU会把所有计算都按最高的浮点精度进行计算，即这三个类型是等价的
  - 在移动平台GPU上会给出不同的进度范围，所以确保在移动平台上验证Shader是否正确
  - fixed精度实际上只在一些较旧的移动平台上使用，在大多数现代GPU上，内部将fixed和half当成同等精度对待

- 尽可能使用较低精度，其可以优化性能，在移动平台上尤为重要

#### 5.7.3 避免不必要的计算

- 若毫无节制的在Shader中（尤其是片元着色器）进行大量计算，则可能造成需要的临时寄存器数目或指令数目超过当前可支持的数目，使Unity报错

- 不同Shader Target、不同着色器阶段可使用的临时寄存器和指令数量是不同的，通常可以指定更高级的Shader Target消除错误

- Unity 支持的Shader Target（Unity 5.x）

  | 指令               | 描述                                                         |
  | ------------------ | ------------------------------------------------------------ |
  | #pragma target 2.0 | 默认的Shader Target等级，相当于Direct3D 9上的Shader Model 2.0，不支持对顶点纹理的采样，不支持显示的LOD纹理采样等 |
  | #pragma target 3.0 | 相当于Direct3D 9上的Shader Model 3.0，支持对顶点纹理的采样   |
  | #pragma target 4.0 | 相当于Direct3D 10上的Shader Model 4.0，支持几何着色器等      |
  | #pragma target 5.0 | 相当于Direct3D 11上的Shader Model 5.0                        |

   不同Unity版本支持的Shader Target种类不同

- Shader Model：微软提出的一套规范，决定了Shader中各个特性和能力，体现在Shader能使用的运算指令数目、寄存器个数等各个方面。Shader Model等级越高，Shader的能力就越大

#### 5.7.4 慎用分支和循环语句

- GPU使用了不同于CPU的技术来实现分支语句，GPU没有CPU的分支预测等技术，流程控制语句会降低GPU的并行处理能力
- 一些处理方法：
  - 尽量将流程控制语句向流水线上端移动，这样受到影响的计算更少
  - 分支判断语句中使用的条件变量最好是常数，即在Shader运行过程中不会发生变化
  - 每个分支中包含的操作指令数尽可能小，分支的嵌套层数尽可能少



