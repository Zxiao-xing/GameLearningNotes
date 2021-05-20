### 3.1 Unity Shader概述

#### 3.1.1 材质和Unity Shader

- 在Unity中需要配合使用材质和Unity Shader才能达到需要的效果，常见流程为：
  1. 创建一个材质
  2. 创建一个Unity Shader，并赋给上一步中创建的材质
  3. 把材质赋给要渲染的对象
  4. 在材质面板中调整Unity Shader的属性以得到满意的结果

#### 3.1.2 Unity中的材质

- Unity中的材质需要结合一个GameObject的Mesh或Particle Systems组价来工作
- 在Unity5.x版本中，默认情况下一个新建的材质将使用Unity内置的Standard Shader，这是一种基于物理渲染的着色器

#### 3.1.3 Unity中的Shader

- Unity5.2及以上版本中，Unity一共提供了4种类Unity Shader模板以供选择：
  1. Standard Surface Shader：产生一个包含了标准光照模型（使用基于物理的渲染方法）的表面着色器模板
  2. Unlit Shader：产生一个不包含光照但包含雾效的基本顶点、片元着色器
  3. Image Effect Shader：实现各种屏幕后处理效果提供了一个基本模板
  4. Compute Shader：产生一种特殊的Shader文件，旨在利用GPU的并行性来进行一些与常规渲染流水线无关的计算
- Unity Shader本质上就是一个文本文件，其拥有导入设置面板：
  - Default Maps指定该Unity Shader使用的默认纹理，当第一次使用时纹理就会自动被赋予到相应的属性上
  - 下放面板显示和该UnityShader相关信息，如是什么类型着色器，标签设置等

### 3.2 Unity Shader的基础：ShaderLab

- Unity Shader是给学习和编写着色器的一个抽象，而和抽象打交道的途径是使用Unity提供的一种专门为Unity Shader服务的语言：ShaderLab。Unity中所有的Unity Shader都是使用ShaderLab来编写的，是Unity提供的编写Unity Shader的一种说明性语言，使用了一些嵌套在花括号内部的语义来描述一个Unity Shader文件的结构，这些结构包含了许多渲染所需的数据。Unity在背后会根据使用的平台将这些结构编译成真正的代码和Shader文件

### 3.3 Unity Shader的结构

#### 3.3.1 给Shader起名

- 每个Unity Shader文件的第一行都需要通过Shader语义来指定名字，其通过一个字符串定义，当作为材质选择使用的UnityShader时，这些名称就会出现在材质面板的下拉类表中。通过斜杠“/"就可以控制Unity Shader在材质面板中出现的位置

  ```
  Shader "Custom/MyShader";		//在Shader->Custom->MyShader位置
  ```

#### 3.3.2 材质和Unity Shader的桥梁：Properties

- Properties语义块中包含了一系列属性，这些属性将出现在材质面板中，这些属性是为了在材质面板中能更方便的调整这些属性。格式如下

  ```
  Properties{
  	Name ("display name", PropertyType) = DefaultValue
  	...
  }
  ```

  1. Name：属性的名字，通常由下划线开始，在Shader中访问时使用

  2. display name：材质面板上的名字

  3. PropertyType：属性的类型

     | 属性类型        | 默认值的定义语法                 |
     | --------------- | -------------------------------- |
     | Int             | number                           |
     | Float           | number                           |
     | Range(min, max) | number                           |
     | Color           | (number, number, number, number) |
     | Vector          | (number, number, number, number) |
     | 2D              | "defaulttexture"{}               |
     | Cube            | "defaulttexture"{}               |
     | 3D              | "defaulttexture"{}               |

     defaulttexture要么是空，要么是内置的纹理名称，花括号用于指定一些纹理属性，在Unity5.0只拿可以使用一些选项来控制固定管线中的纹理坐标的生成，但在Unity5.0以后这些选项被移除了，若要实现类似功能需要自己咋顶点着色器中编写相应纹理坐标的代码

  4. DefaultValue：默认值

- Unity允许重载默认的材质编辑面板以提供更多自定义的数据类型

- 为了在Shader中可以访问这些属性，需要在Cg代码片中定义和这些属性类型相匹配的边浪，即使不在Properties语义块中声明这些属性，也可以直接在Cg代码片中定义这些变量，此时可以通过脚本向Shader中传递这些属性

#### 3.3.3 重量级成员：SubShader

- 每个Unity Shader文件中可以包含多个SubShader语义块，但至少要有一个

- 当Unity需要加载一个Unity Shader时会扫描所有的SubShader语义块，然后选择第一个能够在目标平台上运行的SubShader，若都不支持，就会使用Fallback语义指定的UnityShader

- Unity提供该语义的原因在于不同的显卡具有不同的能力，在高级显卡上使用计算复杂度较高的着色器以提供更出色的画面，在较旧的显卡上使用计算复杂度较低的着色器以保证性能

- SubShader语义块中包含的定义：

  ```
  SubShader{
  	[Tags]
  	
  	[ReaderSetup]
  	
  	Pass{
  	
  	}
  }
  ```

  1. Tags（可选）：标签，是一个键值对，键和值都是字符串类型，它是SubShader和渲染引擎之间沟通的桥梁，用来告诉Unity的渲染引擎如何以及何时渲染该对象

     ```
     Tags{"TagName"="Value", ...}
     ```

     | 标签类型             | 说明                                                         |
     | -------------------- | ------------------------------------------------------------ |
     | Queue                | 控制渲染顺序，指定该物体属于哪一个渲染队列。通过该方式可保证所有的透明物体可以在不透明物体后面渲染。也可以自定义使用的渲染队列来控制物体的渲染顺序 |
     | RenderType           | 对着色器进行分类，可以用于着色器替换功能                     |
     | DisableBatching      | 一些SubShader在使用Unity的批处理功能时会出现问题，可以通过该标签来直接指明是否使用批处理 |
     | ForceNoShadowCasting | 控制使用该SubShader的物体是否会投射阴影                      |
     | IgnoreProjector      | 若该标签值为true，则该SubShader的物体不会受Projector的影响，通常用于半透明物体 |
     | CanUseSpriteAtlas    | 当SubShader用于sprites时，该标签设为False                    |
     | PreviewType          | 指明材质面板将如何预览该材质，默认情况下材质将显示为球形     |

  2. RenderSetup（可选）：状态设置。ShaderLab提供了一系列渲染状态的设置指令，这些指令可以设置显卡的各种状态。当设置渲染状态时，会应用到所有的Pass

     | 常见状态名称 | 设置指令                                                    | 解释                                   |
     | ------------ | ----------------------------------------------------------- | -------------------------------------- |
     | Cull         | Cull Back\|Front\|Off                                       | 设置剔除模式，剔除背面、正面、关闭剔除 |
     | ZTest        | ZTest Less Greater\|LEqual\|GEqual\|Equal\|NotEqual\|Always | 设置深度测试时使用的函数               |
     | ZWrite       | ZWrite On\|Off                                              | 开启/关闭深度写入                      |
     | Blend        | Blend SrcFactor DstFactor                                   | 开启名设置混合模式                     |

  3. Pass：

     ```c++
     Pass{
     	[Name]
     	[Tags]
     	[RenderSetup]
     	//Other code
     }
     ```

     Name：Pass的名称，可以使用ShaderLab的UsePass命令来直接使用其他Unity Shader中的Pass，这样可以提高复用性，但由于Unity内部会把所有Pass名称转换为大写字母的表示，因此使用UsePass时必须使用大写形式的名称

     ```
     UsePass "MyShader/MYPASS"
     ```

     Tags：设置标签，不同于SubShader的标签，这些标签也是用于告诉渲染引擎希望怎样来渲染该物体

     | 标签类型       | 说明                                                         |
     | -------------- | ------------------------------------------------------------ |
     | LightMode      | 定义该Pass在Unity的渲染流水线中的角色                        |
     | RequireOptions | 用于指定当满足某些条件时才渲染该Pass，其值是一个由空格分隔的字符串 |

     RenderSetup：设置渲染状态，SubShader的状态设置同样适用于Pass，除此之外还可以使用固定管线的着色器命令

  除上述普通的Pass定义之外，还支持一些特殊的Pass：

  1. UsePass：用于复用其他Unity Shader的Pass
  2. GrabPass：负责抓取屏幕并将结果存储在一张纹理中用于后续的Pass处理

#### 3.3.4 留一条后路：Fallback

- 跟在各个SubShader语义块之后的可以是一个Fallback指令，用于告诉Unity若上面所有的SubShader在该显卡上都不能运行，就试试使用最低级的Shader

  ```
  Fallback "name"		//使用名称为name的SubShader
  Fallback off		//不使用任何SubShader
  ```

- Fallback还会影响阴影的投射，在渲染阴影纹理时，Unity会在每个Unity Shader中寻找一个阴影投射的Pass，通常情况下不需要专门实现一个Pass，因为Fallback使用内置Shader中包含了一个通用的Pass

#### 3.3.5 ShaderLab中其他语义

- 一些不常用的语义：CustomEditor扩展编辑界面，Category对Unity Shader中的命令进行分组

### 3.4 Unity Shader的形式

- Unity Shader最重要的任务是指定各种着色器所需的代码，这些着色器的代码可以写在SubShader语义块中（表面着色器）或Pass语义块中（顶点/片元着色器和固定函数着色器）

#### 3.4.1 Unity的宠儿：表面着色器

- 表面着色器是Unity自己创造的一种着色器代码雷西兴，需要的代码量很少，Unity在背后做了很多工作，但渲染的代价比较大。其本质上仍要被转换为对应的顶点/片元着色器。其存在价值在于Unity为其处理了很多光照细节而不用开发者去考虑这些事
- 表面着色器被定义在SubShader语义块中的CGPROGRAM和ENDCG之间，因为表面着色器不需要开发者关心使用多少个Pass，每个Pass如何渲染等问题，Unity会在背后做好这些，开发者只需提供对应的数据，如纹理，使用什么光照模型
- CGPROGRAM和ENDCG之间的代码使用Cg/HLSL编写的，这里的Cg/HLSL是Unity经过封装后提供的，语法和标准的Cg/HLSL语法几乎一样，但还是有些不同，如某些原生的函数和用法在Unity中没有提供支持

#### 3.4.2 最聪明的孩子：顶点/片元着色器

- 顶点/片元着色器的代码也需要定义在CGPROGRAM和ENDCG之间，但是写在Pass语义块内，因为需要自己定义每个Pass需要使用的Shader代码

#### 3.4.3 被抛弃的角落：固定函数着色器

- 对于一些较旧的设备不支持可编程管线着色器，因此要使用固定函数着色器完成渲染。其被定义在Pass语义块中，需要使用ShaderLab的渲染设置命令来编写而非Cg/HLSL
- 在Unity5.2中，所有固定函数着色器都会在背后被Unity编译成对应的顶点/片元着色器，所以真正意义上的固定函数着色器已经不存在了

#### 3.4.4 选择哪种Unity Shader形式

- 建议：

  - 除非有明确的需要必须要使用固定函数着色器，否则使用可编程管线的着色器
  - 若要和各种光源打交道，可能更倾向于使用表面着色器，但需要小心在移动平台的性能表现
  - 若使用的光源数目非常少，那么使用顶点/片元着色器是一个更好的选择
  - 若有很多自定义的渲染效果，则使用顶点/片元着色器


### 3.6 答疑解惑

#### 3.6.1 Unity Shader不是真正的Shader

- Unity Shader实际指的是一个ShaderLab文件，硬盘上以.shader作为文件后缀的一种文件
- 在Unity Shader中可以做的事远多于一个传统意义的Shader：
  1. 在传统的Shader中，仅可以编写特定类型的Shader，在Unity Shader中，可以在同一个文件中同时包含需要的顶点着色器和片元着色器代码
  2. 传统的Shader中无法进行配置一些渲染设置，需要在应用程序阶段进行设置，而Unity Shader中通过一行特定的指令就可以完成设置
  3. 传统的Shader中，需要编写冗长的代码来设置着色器输入和输出，要小心的处理这些输入输出的位置对应关系。在Unity Shader中只需要在特定语句块中声明一些属性就可以依靠材质来方便的改变这些属性，且对于模型自带的数据，Unity Shader也提供了直接访问的方法，不需要开发者自行编码传给着色器
- Unity Shader的缺点：由于其高度封装性，可以编写的Shader类型和语法都被限制了，对于一些类型的Shader如曲面细分着色器，几何着色器等Unity的支持就相对差一些。对于一些高级的Shader语法Unity Shader也不支持

#### 3.6.3 我可以使用GLSL来写吗

- 若坚持使用GLSL来写，意味着放弃PC、Xbox360等仅支持DirectX的平台来说就要放弃了。方式是将GLSL代码嵌套在GLSLPROGRAM和ENDGLSL之间