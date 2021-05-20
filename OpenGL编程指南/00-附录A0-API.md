### 第1章 OpenGL概述

```c
void glCreateVertexArrays(GLsizei n, GLuint* arrays);
```

**描述**：

返回n个未使用的对象名到arrays中，用作顶点数组对象

返回的名字可以用来分配更多的缓存对象，并且它们已经使用未初始化的顶点数组集合的默认状态进行了数值的初始化

**错误**：

如果n是负数，产生GL_INVALID_VALUE错误

```c
void glBindVertexArray(GLuint array);
```

**描述**：

若输入的变量array非0且是glCreateVertexArrays()所返回的，就会激活这个顶点数组对象，并且直接影响对象中所保存的顶点数组状态。

若输入的变量array为0。那么OpenGL将不再使用之前绑定的顶点数组

**错误**：

若不是glCreateVertexArrays()所返回的数值，或者已经被glDeleteVertexArrays()函数释放了，产生GL_INVALIS_OPERATION错误

```c
void glDeleteVertexArrays(GLsizei n, const GLuint* arrays);
```

**描述**：

释放n个在arrays中定义的对象。释放后对应的对象名可再次用作顶点数组名。

若绑定的数组已经释放，则当前绑定的顶点数组对象被重设为0，并且不再存在当前对象，在arrays中未使用的名称都会被释放，但是当前顶点数组状态不会发生任何变化

```c
GLboolean gllsVertexArray(GLuint array);
```

**描述**：

若array是一个已经用glCreateVertexArrays()创建且没有被删除的顶点数组对象的名称，返回GL_TURE

若array为0或者不是任何顶点数组对象的名称，返回GL_FALSE

```c
void glCreateBuffers(GLsizei n, GLuint* buffers);
```

**描述**：

返回n个当前未使用的缓存对象名称，并保存到buffers数组中

返回到buffers中的名称不一定是连续的整型数据。0是一个保留的缓存对象名称，永远都不会分配该值

**错误**：

若n是负数，则产生GL_INVALID_VALUE错误

```c
void glBindBuffer(GLenum target, GLuint buffer);
```

**描述**：

若绑定到一个已经创建的缓存对象，则该对象成为当前target中被激活的缓存对象

若绑定的buffer为0，则不对当前target使用任何缓存对象

**target数值**：

GL_ARRAY_BUFFER、GL_ATOMIC_COUNTER_BUFFER、GL_ELEMENT_ARRAY_BUFFER、GL_PIXEL_PACK_BUFFER、GL_PIXEL_UNPACK_BUFFER、GL_COPY_READ_BUFFER、GL_COPY_WEITE_WRITE_BUFFER、GL_SHADER_STORAGE_BUFFER、GL_QUERY_RESULT_BUFFER、GL_DRAW_INDIRECT_BUFFER、GL_TRANSFORM_FEEDBACK_BUFFER、GL_UNIFORM_BUFFER

```c
void glDeleteBuffers(GLsizei n, GLuint* buffers);
```

**描述**：

释放n个保存在buffers数组中的缓存对象，被释放的缓存对象可以重用

若释放的缓存对象已经被绑定，则该对象的所有绑定将会重置为默认的缓存对象。

若释放不存在的缓存对象或缓存对象为0，则为无效操作也不会产生错误

```c
GLboolean glIsBuffer(GLuint buffer);
```

**描述**：

若buffer是一个已经分配且没有释放的缓存对象名称，则返回GL_TURE

若buffer为0或不是缓存对象的名称，则返回GL_FALSE

```c
void glNamedBufferStorage(GLuint buffer, GLsizeiptr size, const void* date, GLbitfield flags);
```

**描述**：

在OpenGL服务端内存中分配size个存储单元（通常为byte），用于存储数据或者索引。

**参数**：

size：存储数据的总数量，等于data中存储的元素的总数乘以单位元素的存储空间

data：要么是一个客户端内存的指针，以便初始化缓存对象，要么是NULL。若指针合法，则会有size大小的数据从客户端传输到服务端。若为NULL，则保留size的大小的未初始化数据，以备后用

flags：提供了缓存中存储的数据相关的用途信息

**flags数值**：

GL_DYNAMIC_STORAGE_BIT、GL_MAP_READ_BIT、GL_MAP_WEITE_BIT、GL_MAP_PERSISTENT_BIT、GL_MAP_COHERENT_BIT、GL_CLIENT_STORAGE_BIT

**错误**：

若size大小超过了服务端能分配的额度，产生GL_OUT_OF_MEMORY错误

若flags包含的不是可用模式，产生GL_INVALID_VALUE错误

**其他**：

作用于名为buffer的缓存区域，不需要设置target参数

```c
void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid* pointer);
```

**描述**：

设置着色器中index位置对应的数据值

**参数**：

pointer：缓存对象中，从其实位置开始计算的数组数据的偏移值，使用基本的系统单位（一般为byte）

size：每个顶点需要更新的分量数目，1、2、3、4或GL_BGRA

type：指定数组中元素数据类型

normalized：设置顶点数据在存储前是否需要进行归一化

stride：数组中每两个元素键的大小偏移值（一般以byte为单位），若为0，则数据紧密封装在一起

**其他**：

只要在内存中数据是规范组织的（保存在一个连续的数组中，不使用其他基于节点如链表的容器），可以用此函数告诉OpenGL如何直接从内存中获取数据

```c
void glEnableVertexAttribArray(GLuint index);
```

**描述**：

启用与index索引相关联的顶点数组，index是一个介于0至GL_MAX_VERTEX_ATTRIBS-1的值

```c
void glDisableVertexAttribArray(GLuint index);
```

**描述**：

禁用与index索引相关联的顶点数组，index是一个介于0至GL_MAX_VERTEX_ATTRIBS-1的值

```c
GLboolean glIsEnables(GLenum capability);
```

**描述**：

判断是否启用当前指定的模式

### 第2章 着色器基础



#### 2.1 Uniform、Buffer块

```c
GLint glGetUniformLocation(GLuint program, const char* name);
```

**描述**：

返回着色器程序中的uniform变量name对应的索引值，若name与启用的着色器程序中的所有uniform变量都不相符，或name是一个内部保留的着色器变量名称，返回-1

**参数：**

name：以NULL结尾的字符串，不存在空格。可以是单一的变量名称，数组中的一个元素（变量名加方括号加索引值），结构体的域变量（变量名加一点加结构体成员名）。对于uniform变量数组，可以直接使用数组名称来获取数组中的第一个元素

**其他：**

除非重新链接着色器程序（glLinkProgram()），否则返回值不会发生变化

```c
void glCreateBuffers(GLsizei n, GLuint* buffers);
```

**描述**：

返回n个当前未使用的缓存对象名称，并保存到buffers数组中

返回到buffers中的名称不一定是连续的整型数据。0是一个保留的缓存对象名称，永远都不会分配该值

**错误**：

若n是负数，则产生GL_INVALID_VALUE错误

```c
void glUniform{1234}{fdi ui}(GLint location, TYPE value);
void glUniform{1234}{fdi ui}v(GLint location, GLsizei count, const TYPE* values);
void glUniformMartrix{234}{fd}v(GLint location, GLsizei count, GLboolean transpose, const GLfloat* values);
void glUniformMarix{2x3,2x4,3x2,3x4,4x2,4x3}{fd}v(GLint location, GLsizei count, GLboolean transpose, const GLfloat* values);
```

**描述**：

设置与location索引位置对应的uniform变量的值，其中向量形式的函数会载入count个数据的集合，根据调用方式读入1~4个值，并写入location位置的uniform变量，若location是数组的起始索引值，那么数组之后的连续count个元素都会被载入

**其他：**

后缀有f的形式可以用来载入单精度类型的浮点数、float类型的向量、数组、向量数组以及布尔数据

后缀有d的形式可以用来载入双精度类型的标准、向量和数组

后缀有i的形式可以用来更新打个有符号整型、有符号整型向量、数组、向量数组，还可以使用其载入独立纹理采样器或者纹理数组、布尔类型的标量、向量和数组

后缀有ui的形式用来载入无符号整型标量、向量、数组

对于Matrix{234}系列来说可以从values中读取2x2，3x3或4x4个值来构成矩阵

对于Martrix{2x3,2x4,3x2,3x4,4x2,4x3}系列来说，可以从values中读取对应矩阵维度的数值并构成矩阵，若transpose设为GL_TRUE，则values中的数据是以行主序，否则以列主序

```c
GLuint glGetUniformBlockIndex(GLuint program, const char* uniformBlockName);
```

**描述**：

返回program中秉承为uniformBlockName的uniform块中的索引值

**错误**：

若uniformBlockName不是一个合法的uniform块，则返回GL_INVALID_INDEX

```c
GLuint glGetUniformBlockIndex(GLuint program, const char* uniformBlockName);
```

**描述**：

返回program中秉承为uniformBlockName的uniform块中的索引值

**错误**：

若uniformBlockName不是一个合法的uniform块，则返回GL_INVALID_INDEX

```c
void glBindBufferRange(GLenum target, GLuint index, GLuint buffer, GLintptr offset, GLsizeiptr size);
void glBindBUfferBase(GLenum target, GLuint index, GLuint buffer)
```

**描述**：

将缓存对象buffer与索引为index的命名uniform块关联起来

**参数：**

target：支持索引的某个缓存绑定目标

index：uniform块的索引

offset：uniform缓存映射的起始索引

size：uniform缓存映射的大小

**其他：**

glBindBufferBase()等价于offset=0，size为缓存对象大小的glBindBufferRange()

**错误**：

若size<0，offset+size大于缓存大小，offset或size不是4的倍数，index<0或index大于target设置的绑定目标所支持的最大索引数，产生GL_INVALID_VALUE错误

```c
GLint glUniformBlockBinding(GLuint program, GLuint uniformBlockIndex, GLuint uniformBlockBinding);
```

**描述**：

显示的将块uniformBlockIndex绑定到uniformBlockBinding

```c
void glGetUniformIndices(GLuint program, GLsizei uniformCount, const char** uniformNames, GLuint* uniformIndices);
```

**描述**：

返回所有uniformCount个uniform变量的索引位置，变量的名称通过字符串数组uniformNames指定，程序返回值保存在数组uniformIndices中

**其他：**

uniformNames中的每个名称都是以NULL来结尾的，并且uniformNames和uniformIndices的数组元素数都是uniformCount个

**错误**：

若在uniformNames中给出的某个名称不是当前启用的uniform变量名称，那么uniformIndices中对应位置将会记录为GL_INVALID_INDEX



#### 2.2 着色器对象的



```c
GLuint glCreateShader(GLenum type);
```

**描述**：

分配一个着色器对象，返回一个非零的整数值

**参数：**

type：GL_VERTEX_SHADER、GL_FRAGMENT_SHADER、GL_TESS_CONTOL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER或Gl_COMPUTE_SHADER

**错误**：

若返回值为0，则说明发生错误

```c
void glShaderSource(GLuint shader, GLsizei count, const GLchar** string, const GLint* length);
```

**描述**：

将着色器源代码关联到一个着色器对象shader上

**参数：**

string：由count行GLchar类型的字符串组成的数组，用于表示着色器源代码数据，可以是以NULL结尾的，也可以不是

length：若为NULL，假设string给出的每行字符串都是以NULL结尾的，否则length中必须有count个元素，分别表示string中对应行的长度。若该数组中某个值是整数，那么表示对应的字符串中的字符数，若为负数则string中对应行假设以NULL结尾

```c
void glCompileShader(GLuint shader);
```

**描述**：

编译着色器的源代码

**其他：**

结果可以用glGetShaderiv(GL_COMPILE_STATUE)查询

```c
void glGetShaderInfoLog(GLuint shader, GLsizei bufSize, GLsizei* length, char* infoLog);
```

**描述**：

返回shader最后编译结果，返回的日志信息是一个以NULL结尾的字符串，保存在infoLog缓存中，长度为length个字符串，返回的最大值通过bufSize来定义。若length为NULL，将不会返回infoLog的大小

```c
GLuint glCreateProgram(void);
```

**描述**：

创建一个空的着色器程序，返回值是一个非零的整数

**错误**：

返回值为零，则发生了错误

```c
void glAttachShader(GLuint program, GLuint shader);
```

**描述**：

将着色器对象shader关联到着色器程序program上

**其他：**

着色器独享的功能只要经过程序的成功连接才能使用。着色器对象可以同时关联到多个不同的着色器程序上

```c
void glDetachShader(GLuint program, GLuint shader);
```

**描述**：

移除着色器对象shader与着色器程序program的关联

**其他：**

若着色器已经被标记为要删除的对象，然后接触关联，则会被即时删除

```c
void glLinkProgram(GLuint program);
```

**描述**：

处理所有与program关联的着色器对象来生成一个完整的着色器程序

**其他：**

结果可以用glGetProgramiv(GL_LINK_STATUS)查询

```c
void glGetProgramInfoLog(GLuint program, GLsizei bufSize, GLsizei* length, char* infoLog);
```

**描述**：

返回最后一次program链接的日志信息，返回的字符串以BULL结尾，长度为length个字符，保存在infoLog缓存中，log可返回的最大值通过bufSize指定，若length为NULL，则不再返回infoLog的长度

```c
void glUseProgram(GLuint program);
```

**描述**：

使用链接过的着色器程序program，若为0，则所有当前使用的着色器都会被清除。若该程序没有绑定任何着色器，结果未定义但不会产生错误

```c
void glDeleteShader(GLuint shader);
```

**描述**：

删除着色器对象shader，若shader已经链接到一个或多个激活的着色器程序上，则将标识为可删除，当对应着色器程序不再使用的时候，就会自动删除这个对象

```c
void glDeleteProgram(GLuint program);
```

**描述**：

立即删除一个当前没有在任何环境中使用的着色器程序program，若程序正在被某个环境使用，则等空闲时再删除

```c
GLboolean glIsShader(GLuint shader);
```

**描述**：

若shader是一个通过glCreateShader()生成的着色器对象名称，且没有被删除则返回GL_TRUE，若shader是0或不是着色器对象名称的非零值，则返回GL_FALSE

```c
GLboolean glIsProgram(GLuint program);
```

**描述**：

若program是一个通过glCreateProgram()生成的程序对象名称，且没有被删除则返回GL_TRUE，若program是0或不是着程序名称的非零值，则返回GL_FALSE

#### 2.3 选择着色器程序

```c
GLuint glGetSubroutineUniformLocation(GLuint program, GLenum shadertype, const char* name);
```

**描述**：

返回名为name的子程序uniform的位置，相应的着色器阶段通过shadertype指定。若name不是一个激活的子程序uniform，返回-1

**参数：**

name：以NULL结尾的字符串

shadertype：GL_VERTEX_SHADER、GL_TESS_CONTROL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER或GL_FRAGMENT_SHADER

**错误**：

若program不是一个可用的着色器程序，生成一个GL_INVALID_OPERATION错误

```c
GLuint glGetSubroutineIndex(GLuint program, GLenum shadertype, const char* name);
```

**描述**：

从程序program返回name所对应的着色器函数的索引，相应的着色器阶段通过shadertype指定。若name不是shadertype着色器的一个活动子程序，返回GGL_INVALID_INDEX

**参数：**

name：以NULL结尾的字符串

shadertype：GL_VERTEX_SHADER、GL_TESS_CONTROL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER或GL_FRAGMENT_SHADER

```c
GLuint glUniformSubroutinesuiv(GLenum shadertype， GLsizei count, const GLuint* indices);
```

**描述**：

设置count个着色器子程序uniform使用Indices数组中的值，相应的着色器阶段通过shadertype来指定。第i个子程序uniform对应indices[i]的值

**参数：**

name：以NULL结尾的字符串

shadertype：GL_VERTEX_SHADER、GL_TESS_CONTROL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER或GL_FRAGMENT_SHADER

**错误**：

若count不等于当前绑定程序的着色阶段shadertype的GL_ACTIVE_SUBROUTINE_UNIFORM_LOCATIONS值，产生GL_INVALID_VALUE错误

Indices中的所有值必须小于GL_ACTIVE_SUBROUTINES，否则产生一个GL_INVALID_VALUE错误

#### 2.4 使用独立的着色器对象

```c
void glProgramUniform{1234}{fdi ui}(GLuint program, Glint location, TYPE value);
void glProgramUniform{1234}{fd ui}v(GLuint program, GLint location, GLsizei count, const TYPE* values);
void glProgramUniformMatrix{234}{fd}v(GLuint program, GLint location, GLsizei count, const TYPE* values);
void glProgramUniformMatrix{2x3,2x4,3x2,3x4,4x2,4x3}{fd}v(GLuint program, GLint location, GLsizei count, GLboolean transpose, const GLfloat* values);
```

**描述**：

glProgramUniform\*()的使用和glUniform\*()的使用是一样的，唯一的区别是使用一个program参数来设置准备更新的uniform变量的着色器程序，优点是program可以不是当前绑定的程序

#### 2.5 SPIR-V

```c
void glShaderBinary(GLsizei count, const GLuint* shaders, enum binaryformat, const void* binary, GLsizei length);
```

**描述**：

若binaryformat设置为GL_SHADER_BINARY_FORMAT_SPIR_V_ARB，则binary中需要设置SPIR-V模块所关联的一组着色器对象。shaders包含一组着色器对象的句柄，大小为count。每个着色器对象句柄对应一个唯一的着色器类型，可以为GL_VERTEX_SHADER、GL_FRAGMENT_SHADER、GL_TESS_CONTROL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER、GL_COMPUTE_SHADER中的一种。binary指向一个合法SPIR-V模块的第一个字节，length包含了SPIR-V模块的字节长度

**其他：**

若成功使用SPIR-V模块，则shaders中的每个入口都可以从这个SPIR-V模块中获取入口点，这些着色器编译的状态会被设置为GL_FALSE

```c
void glSpecializeShader(GLuint shader, const char* pEntryPoint, GLuint numSpecializationConstants, const uint* pConstantIndex, const uint* pCOnstantValue);
```

**描述**：

设置SPIR-V模块中入口点的名字，并设置SPIR-V模块中转优化常量的值

**参数：**

shader：与SPIR-V模块管来呢的着色器对象的名字

PEntryPoint：一个UTF-8字符串指针，使用NULL阶段，表示SPIR-V模块中name着色器对应的入口点名称。若为空，默认字符串为"main"

numSpecializationConstants：本次调用过程中专有化常量的数量

pConstantIndex：数组的指针，包含了numSpecializationConstants个无符号整型数据

pCOnstantValue：无符号整型数组，对应的数据被用来设置专有化常量的值，所有位置由pConstantIndex中的数据决定。可以在该数组中使用浮点数常量并采用IEEE-754标准的表示方法

**其他：**

pConstantIndex没有引用的专有化变量在SPIR-V模块中仍然保留原有值，当专有化完成后，着色器编译状态将设置为GL_TRUE

### 第3章 OpenGL绘制方式

#### 3.1 OpenGL图元

```c
void glPointSize(GLfloat size);
```

**描述**：

设置固定的像素大小，若没有开启GL_PROGRAM_POINT_SIZE，则被用于设置点的大小

```c
void glLineWidth(GLfloat width);
```

**描述**：

设置线段的固定宽度，默认为1.0

**错误：**

当值小于等于0.0时，产生错误

```c
void glPolygonMode(GLenum face, GLenum mode);
```

**描述**：

控制多边形的正面与背面绘制模式

**参数：**

face：必须是GL_FRONT_AND_BACK

mode：GL_POIN（点集）、GL_LINE（轮廓线）、GL_FILL（填充模式），默认为填充模式

```c
void glFrontFace(GLenum mode);
```

**描述：**

控制多边形正面的判断方式，默认模式为GL_CCW，即多边形投影到窗口坐标系后，顶点按逆时针排序的面作为正面。GL_CW，采用顺时针方向的面将被认为是物体的正面

```c
void glCullFace(GLenum mode);
```

**描述：**

在转换到屏幕空间渲染之前，设置需要裁减哪一类多边形

**参数：**

mode：GL_FRONT（正面）、GL_BACK（背面）、GL_FRONT_AND_BACK（所有多边形）

**其他：**

该命令要生效使用glEnable()并设置参数为GL_CULL_FACE，设置该参数给glDisable()关闭

#### 3.2 OpenGL缓存数据

```c
void glCreateBuffers(Glsizei n, Gluint* buffers);
```

**描述：**

返回n个当前未使用的缓存对象名称，并保存到buffers数组中

```c
void glBindBuffer(GLenum target, GLuint buffer)
```

**描述：**

将名称为buffer的缓存对象绑定到target所指定的缓存结合点，若buffer是第一次被绑定，则对应的缓存对象也将同时被创建

**参数：**

target：必须是OpenGL支持的缓存绑定目标之一

buffer：必须是通过glCreateBuffers()分配的名称

```c
void glNamedBufferStorage(GLuint buffer, GLsizeiptr size, const void* data, GLbitfield flags);
```

**描述：**

为缓存对象buffer分配size字节的存储空间，若data不为NULL，则使用data所在的内存区域的内容初始化该空间。flags用来设置缓存的预期用途信息，该flags标识量在用户程序和OpenGL直接构建了协议，允许使用OpenGL尽可能极致的优化缓存的存储空间

```c
void glNamedBufferSubData(GLuint buffer, GLintptr offset, GLsizeiptr size, const void* data);
```

**描述：**

使用新的数据替换缓存对象buffer中的部分数据，缓存中从offset字节处开始需要使用地址为data、大小为size的数据块进行更新。缓存buffer中存储的数据必须经过glNamedBufferStorage()初始化，且标识量应设置为GL_DYNAMIC_STORAGE_BIT

**错误：**

若offset和size的总和超出了缓存对象绑定数据的范围，产生一个错误

```c++
void glClearNamedBufferData(GLuint buffer, GLenum internalformat, Glenum format, GLenum type, const void* data);
void glClearNamedBufferSubData(GLuint buffer, GLenum internalformat, GLintptr offset, GLsizeiptr size, GLenum format, GLenum type, const void* data);
```

**描述：**

清除缓存对象中所有或者部分数据，buffer缓存存储空间将使用data中存储的数据进行填充，format和type分别了data对应数据格式和类型。首先将数据转换到internalformat指定的格式，然后填充缓存数据的指定区域范围。对于glClearNamedBufferData()来说，整个区域都会被指定的数据所填充。而对于glClearNamedBufferSubData()，填充区域是通过offset和size来指定的，分别给出以字节为单位的起始偏移地址和大小

```c
void glCopyNamedBufferSubData(GLuint readBuffer, GLuint writeBuffer, GLintptr readoffset, GLintptr writeoffset, GLsizeiptr size);
```

**描述：**

将名为readBuffer的缓存对象的一部分存储数据拷贝到名为writeBuffer的缓存对象的数据区域上，readBuffer对应的数据从readoffset位置开始复制size个字节，拷贝到writeBuffer对应的writeoffset位置

**错误：**

若readoffset或writeoffset与size的和超出了范围，则OpenGL会产生一个GL_INVALID_VALUE错误

```c++
void glGetNamedBufferSubData(Glenum target, GLintptr offset, GLsizeiptr size, void* data);
```

**描述：**

返回前名为buffer的缓存对象中的部分或全部数据，起始数据的偏移字节位置为offset，回读的数据大小为size个字节，它们将从缓存的数据区域拷贝到data所指向的内存区域中

**错误：**

若缓存对象当前已经被映射，或offset和size的和超出了缓存对象数据区域的范围，则提示一个错误

```c++
void* glMapBuffer(GLenum target, GLenum access);
```

**描述：**

将前绑定到target的缓存对象的整个数据区域映射到客户端的地址空间中，之后可以根据给定的access策略，通过返回的指针对数据进行直接读或写的凑在哦

**错误：**

若OpenGL无法将缓存对象的数据映射出来，那么glMapBuffer()将产生一个错误并返回NULL

```c++
GLboolean glUnmapNamedBuffer(GLuint buffer);
```

**描述：**

解除glMapNamedBufferRange()针对缓存对象buffer创建的映射，若对象数据的内容在映射过程中没有发生损坏则返回GL_TRUE

```c++
void glFlushMappedNamedBufferRange(GLuint buffer, GLintptr offset, GLsizeiptr length);
```

**描述：**

通知OpenGL，映射缓存buffer中由offset和length所划分的区域已经发生了修改，需要立即更新到缓存对象的数据区域中

```c++
void glInvalidateBufferData(GLuint buffer);
void glInvalidateBufferSubData(GLuint buffer, GLintptr offset, GLsizeiptr length);
```

**描述：**

通知OpenGL，应用程序已经完成对缓存对象中给定范围内容的操作，因此可以随时根据实际情况抛弃数据

#### 3.3 顶点规范

```c++
void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid* pointer);
```

**描述：**

设置顶点属性在index位置可访问的数据值。

**参数：**

pointer：数组中第一组数据值的位置，以基本计算机代为度量，其由绑定到GL_ARRAY_BUFFER目标缓存对象中地址偏移量决定的

size：每个顶点中需要更新的元素个数，可选值包括1、2、3、4以及GL_BGRA（压缩格式）

type：数组中每个元素的数据类型

normalized：顶点数据是否需要在传递到顶点数组之前进行归一化处理

stride：数组中两个连续元素之间的偏移字节数

```c++
void glVertexAttribIPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const GLvoid* pointer);
```

**描述：**

与glVertexAtrribPointer类似，但type必须是整数类型中的一种，且不需要normalized参数

```c++
void glVertexAttribLpointer(GLuint index, GLint size, GLenum type, GLsizei stride, const GLvoid* pointer);
```

**描述：**

与glVertexAtrribPointer类似，对于传入顶点着色器的64位双精度浮点型顶点属性，type必须设置为GL_DOUBLE，且不需要normalized参数

```c++
void glVertexAttrib{1234}{fds}(GLuint index, TYPE values);
void glVertexAttrib{1234}{fds}v(GLuint index, const TYPE* values);
void glVertexAttrib4{bsifd ub us ui}v(GLuint index, const TYPE* values);
```

**描述：**

设置索引为index的顶点属性的静态值，若函数名称末尾没有v，最多可以指定4个参数值，即x、y、z、w参数，若有，则最多有4个参数值是保存在一个数组中传入的，其地址通过values指定，存储顺序依次是x、y、z和w

```c++
void glVertexAttrib4Nub(GLuint index, GLubyte x, GLubyte y, GLubyte z, GLubtye w);
void glVertexAttrib4N{bsi ub us ui}v(GLuint index, const TYPE* v);
```

**描述：**

设置索引index所对应的一个或多个顶点属性静态值，并在转换过程中将无符号参数归一化至[0, 1]有符号归一化至[-1, 1]

```c++
void glVertexAttribl{1234}{i ui}(GLuint index, TYPE values);
void glVertexAttribl{123}{i ui}v(GLuint index, const TYPE* values);
void glVertexAttribl4{bsi ub us ui}v(GLuint index, const TYPE* values);
```

**描述：**

设置一个或多个静态整型顶点属性静态值，用于索引index所对应的整型顶点属性值

```c++
void glVertexAttribL{1234}(GLuint index, TYPE values);
void glVertexAttribL{1234}v(GLuint index, const TYPE* values);
```

**描述：**

设置一个或多个静顶点属性静态值，用于索引index所对应的双精度顶点属性值

#### 3.4 OpenGL绘制命令

```c++
void glDrawArrays(GLenum mode, GLint first, GLsizei count);
```

**描述：**

使用数组元素建立的连续的几何图元序列，每个启用的数组中起始位置为first，结束位置为first+count-1

**参数：**

mode：表示构建图元的类型。必须是GL_TRIANGLES、GL_LINE_LOOP、GL_LINES、GL_POINTS之一

```c++
void glDrawElements(GLenum mode, GLsizei count, GLenum type, const GLvoid* indices);
```

**描述：**

使用count个元素来定义一系列几何图元，元素的索引值保存在一个绑定到GL_ELEMENT_ARRAY_BUFFER的缓存中

**参数：**

indices：元素数组缓存中的偏移地址，即索引数据开始的位置，单位为字节

type：元素数组缓存中索引数据的类型，必须是GL_UNSIGNED_BYTE、GL_UNSIGNED_SHORT、GL_UNSIGNED_INT之一

mode：表示构建图元的类型。必须是GL_TRIANGLES、GL_LINE_LOOP、GL_LINES、GL_POINTS之一

```c++
void glDrawElementsBaseVertex(GLenum mode, GLsizei count, GLenum type, const GLvoid* indices, GLint basevertex);
```

**描述：**

本质上和glDrawElements没有区别，但其第i个元素在传入绘制命令时，实际上读取的是各个顶点属性数组的第indices[i]+basevertex个元素

```c++
void glDrawRangeElements(GLenum mode, GLuint start, GLuint end, GLsizei count, GLenum type, const GLvoid* indices);
```

**描述：**

比之glDrawElements()更严格，相当于应用程序与OpenGL之间形成了一种约定，即数组缓存中所包含的任何一个索引值都会在start和end范围中

```c++
void glDrawRangeElementsBaseVertex(GLenum mode, GLuint start, GLuint end, GLsizei count, GLenum type, const GLvoid* indices, GLint basevertex);
```

**描述：**

首先检查元素数组缓存中保存的数据是否落入start和end之间，再对其添加basevertex基数

```c++
void glDrawArraysIndirect(GLenum mode, const GLvoid* indirect);
```

**描述：**

特性与glDrawArraysInstanced()完全一致，但绘制命令的参数是从绑定到GL_DRAW_INDIRECT_BUFFER的缓存中获取的结构器数据

**参数：**

indirect：记录间接绘制缓存中的偏移地址

mode：表示构建图元的类型。必须是GL_TRIANGLES、GL_LINE_LOOP、GL_LINES、GL_POINTS之一

```c++
void glDrawElementsIndirect(GLenum mode, GLenum type, const GLvoid* indirect);
```

**描述：**

本质上与glDrawElements()是一致的，但绘制命令的参数是从绑定到GL_DRAW_INDIRECT_BUFFER的缓存中获取的结构器数据

```c++
void glMultiDrawArrays(GLenum mode, const GLint* first, const GLint* count, GLsizei primcount)
```

**描述：**

在一个OpenGL函数调用过程中绘制多组几何图元集，first和count都是数组的形式，数组的每个元素都相当于一次glDrawArrays()调用，元素的总数由primcount决定

```c++
void glMultiDrawElements(GLenum mode, const GLint* count, GLenum type, const GLvoid* const* indices, GLsizei primcount);
```

**描述：**

在一个OpenGL函数调用过程中绘制多组几何图元集，first和indices都是数组的形式，数组的每个元素都相当于一次glDrawElements()调用，元素的总数由primcount决定

```c++
void glMultiDrawElementsBaseVertex(GLenum mode, cosnt GLint* count, GLenum type, const GLvoid* const* indices, GLsizei primcount, const GLint* baseVertex);
```

**描述：**

在一个OpenGL函数调用过程中绘制多组几何图元集，first、indices和baseVertex都是数组的形式，数组的每个元素都相当于一次glDrawElementsBaseVertex()调用，元素的总数由primcount决定

```c++
void glMultiDrawArraysIndirect(GLenum mode, const void* indirect, GLsizei drawcount, GLsizei stride);
```

**描述：**

绘制多组图元集，相关参数全部保存到缓存对象中，在glMultiDrawArraysIndirect()的一次调用当中，可以分发总共drawcount个独立的绘制命令，命令中的参数与glDrawArraysIndirect()所用的参数一致的，每个DrawArraysindirectCommand结构体之间的间隔都是stride个字节

```c++
void glMultiDrawElementsIndirect(GLenum mode, GLenum type, const void* indirect, GLsizei drawcount, GLsizei stride)
```

**描述：**

绘制多组图元集，相关参数全部保存到缓存对象中，在glMultiDrawElementsIndirect()的一次调用当中，可以分发总共drawcount个独立的绘制命令，命令中的参数与glDrawElementsIndirect()所用的参数一致的，每个DrawElementsIndirectCommand结构体之间的间隔都是stride个字节

```c++
void glPrimitiveRestartIndex(GLuint index);
```

**描述：**

设置一个顶点数组元素的索引值，用来指定渲染过程中从什么地方启动新的图元绘制，若在处理顶点数组元素索引的过程中遇到了一个符合该索引的数值，则系统不会处理其对应的顶点数据，而是终止当前的图元绘制，并且从下一个顶点重新开始渲染同一类型的图元集合

```c++
void glDrawArraysInstanced(GLenum mode, GLint first, GLsizei count, GLsizei primCount);
```

**描述：**

通过mode、first和count所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性

```c++
void glDrawElementsInstanced(GLenum mode, GLsizei count, GLenum type, const void* indices, GLsizei primCount);
```

**描述：**

通过mode、first和indices所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性

```c++
void glDrawElementsInstancedBaseVertex(GLenum mode, GLsizei count, GLenum type, const void* indices, GLsizei instanceCount, GLuint baseVertex);
```

**描述：**

通过mode、first、indices和baseVertex所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性

```c++
void glVertexAttribDivisor(GLuint index, GLuint divisor);
```

**描述：**

设置多实例渲染时，位于index位置的顶点着色器顶点属性是如何分配值到每个实例的。若divisor为0，则该属性的多实例特性被禁用，而其他值则表示顶点着色器每个divisor个实例都会分配一个新的属性值，默认情况下，每个顶点都会分配到一个独立的属性值

**参数：**

index：设置多实例特性的顶点属性的索引位置

```c++
void glDrawArraysInstancedBaseInstance(GLenum mode, GLint first, GLsizei count, GLsizei primCount, GLuint baseInstance);
```

**描述：**

通过mode、first和count所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性。baseInstance的值用来对实例化的顶点属性设置一个索引的偏移值，从而改变OpenGL取出的索引位置

```c++
void glDrawElementsInstancedBaseInstance(GLenum mode, GLsizei count, GLenum type, const void* indices, GLsizei primCount, GLuint baseInstance);
```

**描述：**

通过mode、first和indices所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性。baseInstance的值用来对实例化的顶点属性设置一个索引的偏移值，从而改变OpenGL取出的索引位置

```c++
void glDrawElementsInstancedBaseVertexBaseInstance(GLenum mode, GLsizei count, GLenum type, const void* indices, GLsizei instanceCount, GLuint baseVertex, GLuint baseInstance);
```

**描述：**

通过mode、first、indices和baseVertex所构成的几何图元集，绘制它的primCount个实例，对于每个实例，内置变量gl_InstanceID都会依次递增，新的数值会被传递到顶点着色器以区分不同实例的顶点属性。baseInstance的值用来对实例化的顶点属性设置一个索引的偏移值，从而改变OpenGL取出的索引位置

### 第4章 颜色、像素和片元



#### 4.2 缓存及其用途

```c++
void glClearBufferfv(GLenum buffer, GLint drawbuffer, const GLfloat *value);
```

**描述：**

清除buffer指定的第drawbuffer个缓存，并初始化为value数组的值

**参数：**

buffer：GL_COLOR（颜色缓存）、GL_DEPTH（深度缓存）

**其他：**

当清除深度缓存时，drawbuffer只能为0，因为只有一个深度缓存

```c++
void glClearBufferiv(GLenum buffer, GLint drawbuffer, const GLint* value);
void glClearBufferuiv(GLenum buffer, GLint drawbuffer, const GLuint* value);
void glClearBufferfi(GLenum buffer, GLint drawbuffer, GLfloat depth, GLint stencil);
```

**描述：**

glClearBufferiv()、glClearBufferuiv()用于清除整型类型的缓存，glClearBufferiv()可以清除模板缓存，并初始化为value数组的值

glClearBufferfi()可以同时清除深度和模板缓存，这个函数buffer必须为GL_DEPTH_STENCIL，且drawbuffer为0

```c++
void glColorMask(GLboolean red, GLboolean green, GLboolean blue, GLboolean alpha);
void glColorMaski(GLuint buffer, GLboolean red, GLboolean green, GLboolean blue, GLboolean alpha);
void glDepthMask(GLboolean flag);
void glStencilMask(GLboolean mask);
void glStencilMaskSeparate(GLenum face, GLuint mask);
```

**描述：**

设置用于控制写入不同缓存的掩码

glDepthMask()的flag为GL_TRUE，则深度缓存可以写入

glStencilMask()的mask参数用于和模板值进行按位与操作，若对应位操作结果为1，则像素的模板值可以写入

glStencilMaskSeparate()可以为多边形的正面和背面设置不同的模板掩码值

glColorMaski()可以对第buffer个缓存对象设置颜色掩码

所有GLboolean掩码的默认值为GL_TRUE，GLuint掩码的默认值为1

#### 4.4 片元的测试与操作

```c++
void glScissor(GLint x, GLint y, GLsizei width, GLsizei height);
```

**描述：**

设置剪切矩形的位置与大小，(x, y)是矩形左下角的坐标位置，width和height是矩形的高度和宽度

**其他：**

通过glEnable()和glDisable()设置参数为GL_SCISSOR_TEST开启或禁止。默认条件下，剪切矩形的大小与窗口大小相等，且剪切测试是关闭的

```c++
void glSampleCoverage(GLfloat value, GLboolean invert);
```

**描述：**

设置多重采样覆盖率的参数以正确解算alpha值。

**参数：**

value：若开启GL_SAMPLE_ALPHA_TO_CONVERAGE或GL_SAMPLE_CONVERAGE，则是一个临时的采样覆盖值。

invert：是一个布尔变量，用于设置这个临时覆盖值是否需要先进行位反转，然后再与片元覆盖率进行与操作



```c++
void glSampleMaski(GLuint index, GLbitfield mask);
```

**描述：**

设置一个32位的采样掩码mask，若当前帧缓存包含了多余32个采样数，则采样掩码的长度可能是多于32位大小的WORD字段组成，每一个WORD表示一个32位数据。当采样结果准备写入到帧缓存时，只有当前采样掩码值中对应位的数据才会被更新，其他数据将被丢弃

**参数：**

index：掩码本身的索引位置

mask：新的掩码

```c++
void glStencilFunc(GLenum func, GLint ref, GLuint mask);
void glStencilFuncSeparate(GLenum face, GLenum func, GLint ref, GLuint mask);
```

**描述：**

设置比较函数func、参考值ref以及掩码mask以完成模板测试。参考值将与mask参数进行与操作丢弃结果为0的位平面然后和模板缓存已有的值进行比较

glStencilFuncSeparate()允许为多边形的正面和背面单独设置模板函数参数

**参数：**

func：GL_NEVER、GL_ALWAYS、GL_LESS、GL_LEQUAL、GL_EQUAL、GL_GEQUAL、GL_GREATER、GL_NOTEQUAL

**其他：**

经过掩码操作的数据均被解析成非负数据。模板测试的开启和禁止是通过glEnable()和glDisable()设置参数GL_STENCIL_TEST开启和禁止。默认情况下，func为GL_ALWAYS，ref为0，mask所有位为1，且禁止模板测试

```c++
void glStencilOp(GLenum fail, GLenum zfail, Glenum zpass);
void glStencilOpSeparate(GLenum face, GLenum fail, GLenum zfail, Glenum zpass);
```

**描述：**

设置当片元通过或没有通过模板测试的时候，如何处理模板缓存中的数据。当片元没有通过模板测试，则执行fail函数，通过了模板测试但没有通过深度测试，则执行zfail函数，都通过了或通过模板测试没开启深度测试，则执行zpass函数。

glStencilOpSeparate()允许为多边形的正面和背面单独设置模板测试参数

**参数：**

fail、zfail、zpass：GL_KEEP（保持）、GL_ZERO（替换为0值）、GL_REPLACE（替换为参考值）、GL_INCR（使用饱和运算增加1）、GL_INCER_WRAP（不使用饱和运算增加1）、GL_DECR（使用饱和运算减少1）、GL_DECR_WRAP（不使用饱和运算减少1）、GL_INVERT（按位反转）

**其他：**

加1减1的值总是落在0到对应位的最大无符号整数值区间内

使用饱和运算是将模板值阶段在区间的极值上，不使用是数值超出区间后，变换到区间的另一端

默认情况下，三个函数都为GL_KEEP

```c++
void glDepthFunc(GLenum func);
```

**描述：**

设置深度测试的比较函数。对于任何输入的片元，若其z值与深度缓存中已有的值相比符合函数定义的条件，则测试通过。

**参数：**

func：GL_NEVER、GL_ALWAYS、GL_LESS、GL_LEQUAL、GL_EQUAL、GL_GEQUAL、GL_GREATER、GL_NOTEQUAL

**其他：**

默认的比较函数为GL_LESS

```c++
void glPolygonOffset(GLfloat factor, GLfloat units);
```

**描述：**

开启后，每个片元的深度值都会被修改，在执行深度测试之前添加一个计算偏移值：offset = m * factor + r * units。其中m是多边形的最大深度斜率（在光栅化过程中计算得到），r是两个不同深度之间的可识别的最小差值，是一个平台实现相关的常量

```c++
void glBlendFunc(GLenum srcfactor, GLenum destfactor);
void glBlendFunci(GLuint buffer, GLenum srcfactor, GLenum destfactor);
```

**描述：**

控制片元输出的颜色值和存储在帧缓存中的值来进行混合。

glBlendFunc()设置所有可绘制缓存的融混参数，glBlendFunci()设置缓存buffer的融混参数

**参数：**

srcfactor：源融混参数

destfactor：目标融混参数

源和目标融混参数：

| 枚举常量                    | RGB混融参数                  | Alpha混融参数 |
| --------------------------- | ---------------------------- | ------------- |
| GL_ZERO                     | (0, 0, 0)                    | 0             |
| GL_ONE                      | (1, 1, 1)                    | 1             |
| GL_SRC_COLOR                | (Rs, Gs, Bs)                 | As            |
| GL_ONE_MINUS_SRC_COLOR      | (1, 1, 1) - (Rs, Gs, Bs)     | 1-As          |
| GL_DST_COLOR                | (Rd, Gd, Bd)                 | Ad            |
| GL_ONE_MINUS_DST_COLOR      | (1, 1, 1) - (Rd, Gd, Bd)     | 1-Ad          |
| GL_SRC_ALPHA                | (As, As, As)                 | As            |
| GL_ONE_MINUS_SRC_ALPHA      | (1, 1, 1) - (As, As, As)     | 1-As          |
| GL_DST_ALPHA                | (Ad, Ad, Ad)                 | Ad            |
| GL_ONE_MINUS_DST_ALPHA      | (1, 1, 1) - (Ad, Ad, Ad)     | 1-Ad          |
| GL_CONSTANT_COLOR           | (Rc, Gc, Bc)                 | Ac            |
| GL_ONE_MINUS_CONSTANT_COLOR | (1, 1, 1) - (Rc, Gc, Bc)     | 1-Ac          |
| GL_CONSTANT_ALPHA           | (Ac, Ac, Ac)                 | Ac            |
| GL_ONE_MINUS_CONSTANT_ALPHA | (1, 1, 1) - (Ac, Ac, Ac)     | 1-Ac          |
| GL_SRC_ALPHA_SATURATE       | (f, f, f)，f = mid(As, 1-Ad) | 1             |
| GL_SRC1_COLOR               | (Rs1, Gs1, Bs1)              | As1           |
| GL_ONE_MINUS_SRC1_COLOR     | (1, 1, 1) - (Rs1, Gs1, Bs1)  | 1-As1         |
| GL_SRC1_ALPHA               | (As1, As1, As1)              | As1           |
| GL_ONE_MINUS_SRC1_ALPHA     | (1, 1, 1) - (As1, As1, As1)  | 1-As1         |

若使用GL_CONSTANT融混枚举量，则必须使用glBlendColor()来设置对应的常量颜色值

**其他：**

对于无符号和有符号的归一化帧缓存格式，融混参数将被分别限制在[0, 1]或[-1, 1]区间内。若帧缓存采用浮点数格式，则参数不存在上限和下限

```c++
void glBlendFuncSeparate(GLenum srcRGB, GLenum destRGB, GLenum srcAlpha, GLenum destAlpha);
void glBlendFuncSeparatei(GLuint buffer, GLenum srcRGB, GLenum destRGB, GLenum srcAlpha, GLenum destAlpha);
```

**描述：**

控制片元输出的颜色值和存储在帧缓存中的值来进行混合。允许RGB和alpha颜色分量使用不同的融混参数

glBlendFuncSeparate()设置所有可绘制缓存的融混参数，glBlendFuncSeparatei()设置缓存buffer的融混参数

**参数：**

srcRGB：RGB的源融混参数

destRGB：RGB的目标融混参数

srcAlpha：alpha的源融混参数

destAlpha：alpha的目标融混参数

```c++
void glBlendColor(GLclampf red, GLclampf green, GLclampf blue, GLclampf alpha);
```

**描述：**

若融混模式为GL_CONSTANT_COLOR，则设置一组红色、绿色、蓝色和alpha值作为常量颜色(Rc, Gc, Bc, Ac)

```c++
void glBlendEquation(GLenum mode);
void glBlendEquationi(GLuint buffer, GLenum mode);
```

**描述：**

设置帧缓存和源数据颜色之间的融混方法。

glBlendEquation()为所有缓存设置融混模式，glBlendEquationi()设置缓存buffer的融混模式

**参数：**

融混方程的数学操作符

| 融混模式参数             | 数学操作      |
| ------------------------ | ------------- |
| GL_FUNC_ADD              | CsS + CdD     |
| GL_FUNC_SUBTRACT         | CsS - CdD     |
| GL_FUNC_REVERSE_SUBTRACT | CdD - CsS     |
| GL_MIN                   | min(CsS, CdD) |
| GL_MAX                   | max(CsS, CdD) |

```c++
void glBlendEquationSeparate(GLenum modeRGB, GLenum modeAlpha);
void glBlendEquationiSeparate(GLuint buffer, GLenum modeRGB, GLenum modeAlpha);
```

**描述：**

设置帧缓存和源数据颜色之间的融混方法，允许RGB和alpha颜色分量使用不同的融混模式

glBlendEquationSeparate()为所有缓存设置融混模式，glBlendEquationiSeparate()设置缓存buffer的融混模式

```c++
void glLogicOp(GLenum opcode);
```

**描述：**

对于给定的输入片元和当前颜色缓存的像素，选择需要执行的逻辑操作

**参数：**

| 参数             | 操作  | 参数            | 操作    |
| ---------------- | ----- | --------------- | ------- |
| GL_CLEAR         | 0     | GL_AND          | s&d     |
| GL_COPY          | s     | GL_OR           | s\|d    |
| GL_NOOP          | d     | GL_NAND         | ~(s&d)  |
| GL_SET           | 1     | GL_NOR          | ~(s\|d) |
| GL_COPY_INVERTED | ~s    | GL_XOR          | s^d     |
| GL_INVERT        | ~d    | GL_EQUIV        | ~(s^d)  |
| GL_AND_REVERSE   | s&~d  | GL_AND_INVERTED | ~s&d    |
| GL_OR_REVERSE    | s\|~d | GL_OR_INVERTED  | ~s\|d   |

```c++
void glCreateQueries(GLenum target, GLsizei n, GLuint* ids);
```

**描述：**

创建n个新的查询对象，它们使用时对应的目标由target决定，新的查询对象的名称被放置在ids数组中，其中保存的名称不一定是连续的整数数列

**其他：**

0是一个被保留的遮挡查询对象名称，永远不会返回0值作为有效的对象名

```c++
GLboolean glIsQuery(GLuint id);
```

**描述：**

若id是一个遮挡查询对象的名称，则返回GL_TRUE

```c++
void glBeginQuery(GLenum target, GLuint id);
```

**描述：**

启动遮挡查询操作

**参数：**

target：都与样本值的计算有关

- GL_SAMPLE_PASSED，会得到精确的片元数量，它们全都通过了逐片元的测试过程，使用该类型可能会降低OpenGL的性能，只要在需要精确值时使用
- GL_ANY_SAMPLES_PASSED，又叫非遮挡查询，得到一个近似的技术结果，该目标量唯一确保的是若没有任何片元能通过逐片元的测试，那么查询结果一定为0
- GL_ANY_SAMPLES_PASSED_CONSERVATIVE提供了一个比上一个更宽松的结果，只有OpenGL绝对确信不会有任何片元通过测试，查询的结果才为0。该类型是性能最高的也是最不精准的

id：用于标识本次遮挡查询操作

```c++
void glGetQueryObjectiv(GLenum id, GLenum pname, GLint* params);
void glGetQueryObjectuiv(GLenum id, GLenum pname, GLint* params);
```

**描述：**

获取遮挡查询对象的状态信息

**参数：**

id：遮挡查询物体名称

params：若为GL_QUERY_RESULT，则片元或样本的数量写入到params中

若为GL_QUERY_RESULT_AVAILABLE，且查询id的结果可以读取，则params为GL_TRUE，否则为GL_FLASE

**其他：**

遮挡查询操作结束后可能会有一点延迟才能获取结果

```c++
void glBeginConditionalRender(GLuint id, GLenum mode);
void glEndConditionalRender(void);
```

**描述：**

记录一系列OpenGL渲染命令，系统会根绝遮挡查询对象id的结果来决定是否自动抛弃他们

**参数：**

mode：设置OpenGL中要如何使用遮挡查询的结果

- GL_QUERY_WAIT，GPU将等待遮挡查询的结果返回，然后判断是否要继续进行渲染
- GL_QUERY_NO_WAIT，GPU可以不等待正当查询的结果返回就继续进行渲染，若结果还未返回的话，则将选择条件渲染区域内的一部分场景进行渲染
- GL_QUERY_BY_REGION_WAIT，GPU将判断片元的结果是否对条件渲染有所贡献，并等待这些结果渲染完成，同时会等待完整的遮挡查询结果返回
- GL_QUERY_BY_REGION_NO_WAIT，GPU会抛弃帧缓存中所有对遮挡查询没有贡献的区域，且即使结果还没有返回，也会开始渲染其他区域的内容

**错误：**

若id不是有效的遮挡查询对象，则产生GL_INVALID_VALUE错误

若条件渲染对列已经开始执行的时候再次调用glBeginConditionalRender()或在没有启用时调用glEndConditionalRender()，或id作为遮挡查询对象设置为GL_SAMPLES_PASSED以外的模式，或id对应的遮挡查询还在进行中，则产生GL_INVALID_OPERATION

#### 4.5 多重采样

```c++
void glGetMultisamplefv(GLenum pname, GLuint index, GLfloat* val);
```

**描述：**

若设置pname为GL_SAMPLE_POSITION，则返回第index个样本的位置信息存储在val参数中的一堆浮点数，位置信息区间为[0, 1]，即该样本相对于像素左下角位置的偏移值

**错误：**

若index大于等于系统所支持的样本数，则产生一个GL_INVALID_VALUE错误

```c++
void glMinSampleShading(GLfloat value);
```

**描述：**

设置每个像素中独立着色的样本值数量，value设置的是独立着色样本占总样本的比率，要限制在[0, 1]区间内，1.0表示所有的样本都会使用唯一一组采样数据

#### 4.6 逐图元的反走样

```c++
void glHint(GLenum target, GLenum hint);
```

**描述：**

控制OpenGL的一些具体特性

**参数：**

target：设置要控制的特性类型

| 参数                               | 描述                                               |
| ---------------------------------- | -------------------------------------------------- |
| GL_LINE_SMOOTH_HINT                | 线的反走样质量                                     |
| GL_POLYGON_SMOOTH_HINT             | 多边形边的反走样质量                               |
| GL_TEXTURE_COMPRESSION_HINT        | 纹理图像压缩的质量和性能                           |
| GL_FRAGMENT_SHADER_DERIVATIVE_HINT | 片元处理内置函数的导数精度，包括dFdx、dFdy、fwidth |

hint：GL_FASTEST（效率最高）、GL_NICEST（质量最高）、GL_DONT_CARE（没有偏好）

**其他：**

hint参数的解析方式与平台相关，有些OpenGL实现可能会完全忽略它们的影响

#### 4.7 像素数据的读取和拷贝

```c++
void glReadPixels(GLint x, GLint y, GLsizei width, GLsizei height, GLenum format, GLenum type, void* pixels);
```

**描述：**

从可读的帧缓存中读取像素数据，矩形区域返回的左下角定义为窗口坐标的(x, y)，尺寸为width和height，将数据数组保存到pixels。

**参数：**

format：指定读取的像素数据类型

| 枚举量                     | 像素格式         |
| -------------------------- | ---------------- |
| GL_RED、GL_RED_INTEGER     | 红色分量         |
| GL_GREEN、GL_GREEN_INTEGER | 绿色分量         |
| GL_BLUE、GL_BLUE_INTEGER   | 蓝色分量         |
| GL_ALPHA、GL_ALPHA_INTEGER | alpha分量        |
| GL_RG、GL_RG_INTEGER       | 红绿分量         |
| GL_RGB、GL_RGB_INTEGER     | 红绿蓝分量       |
| GL_RGBA、GL_RGBA_INTEGER   | 红绿蓝alpha分量  |
| GL_BGR、GL_BGR_INTEGER     | 蓝绿红分量       |
| GL_BGRA、GL_BGRA_INTEGER   | 蓝绿红alpha分量  |
| GL_STENCIL_INDEX           | 模板索引值       |
| GL_DEPTH_COMPONENT         | 深度分量         |
| GL_DEPTH_STENCIL           | 深度和模板索引值 |

type：指定每个元素的数据类型

```c++
void glClampColor(GLenum target, GLenum clamp)
```

**描述：**

控制浮点数和顶点数缓存的颜色值截断方式

**参数：**

target：GL_CLAMP_READ_COLOR开启阶段

clamp：GL_TRUE从缓存都会的颜色值将被截断到[0, 1]，Gl_FALSE不会进行阶段，GL_FIXED_ONLY只截断定点数数据而浮点数仍保留原始形式

#### 4.8 拷贝像素矩形

```c++
void glBlitFramebuffer(GLuint readFramebuffer, GLuint drawFramebuffer, GLint srcX0, GLint srcY0, GLint srcX1, GLint srcY1, GLint dstX0, GLint dstY0, GLint dstX1, GLint dstY1, GLbitfield mask, GLenum filter)
```

**描述：**

将矩形的像素数据从可读帧缓存的一处区域拷贝到可绘制帧缓存的另一处区域中，该过程可能存在自动缩放、反转、转换和滤波的操作。若当前存在多个颜色绘制缓存，则每个缓存都会进行一次源数据的拷贝

- 若矩形区域的大小不同，则filter设置插值的方法，GL_NEAREST（最近）、GL_LINEAR（插值）
- 若X1<X0或Y1<Y0，则读取或写入的是不同方向反转的图像
- 若源数据缓存和目标数据缓存的格式不同，则多数情况下自动进行转换操作。但若可读颜色缓存是浮点数，但写入的不是，或可读颜色缓存是整数类型，但写入的缓存不是，则产生GL_INVALID_OPERATION错误，且不会拷贝任何数据
- 若源缓存是多重采用的，但目标缓存不是，则所有的样本值都会归总到单一的像素值并存入目标缓存。若目标缓存是多重采样，源缓存不是，则源数据将进行多次拷贝以匹配样本质。若两个缓存都是多重采样，采样数相同则直接拷贝，不同则不会拷贝像素并产生GL_INVALID_OPERATION错误
- 若buffers中含有多余的类型值，或filter不是GL_NEAREST、GL_LINEAR产生GL_INVALID_VALUE错误

**参数：**

srcX0、srcY0、srcX1、srcY1：像素数据源的区域范围

dstX0、dstY0、dstX1、dstY1：目标区域的区域范围

### 第5章 视口变换、裁剪、剪切与反馈



#### 5.3 OpenGL变换

```c++
void glDepthRange(GLclampd near, GLclampd far);
void glDepthRangef(GLclampd near, GLclampd far);
```

**描述：**

设置z轴上的近平面位于near，远平面位于far，该函数定义了视口变换中z坐标的变化范围。近平面和远平面的值也就是深度缓存中所保存的最小值和最大值，默认情况下分别为0.0和1.0。该函数的参数的范围在[0, 1]之间

```c++
void glViewport(GLint x, GLint y, GLint width, GLint height);
```

**描述：**

在程序窗口定义一个矩形的像素区域，并且将最终渲染的图像映射到其中，(x, y)设置视口的左下角坐标，width和height分别设置视口矩形的像素大小。默认视口值设为(0, 0, winWidth, winHeight)，其中winWidth和winHeight是窗口的大小

```c++
void glClipControl(GLenum origin, GLenum depth);
```

**描述：**

设置剪切坐标到窗口坐标的映射方式，origin设置的是窗口坐标x和y的原点，depth设置剪切空间深度映射到glDepthRange()所设置的数值的方式

**参数：**

depth：GL_NEGATIVE_ONE_TO_ONE（窗口空间中的深度对应剪切空间中的[-1.0, 1.0]范围）、GL_ZERO_TO_ONE（窗口空间中的深度对应剪切空间中的[0.0, 1.0]范围）

origin：GL_LOWER_LEFT（剪切空间的xy坐标(-1.0, -1.0)对应窗口坐标的左下角，y轴正向对应上方向）、GL_UPPER_LEFT(剪切空间的xy坐标(-1,0, -1.0)对应窗口坐标的左上角，y轴正向对应下方向)

#### 5.4 transform feedback

```c++
void glCerateTransformFeedbacks(GLsiei n, GLuint* ids);
```

**描述：**

创建n个新的transform feedback对象，并且将生成的名称记录在数组ids中

```c++
void glBindTransformFeedback(GLenum target, GLuint id);
```

**描述：**

将第一个名称为id的transform feedback对象绑定到目标target上，目标的值必须是GL_TRANSFORM_FEEDBACK

```c++
GLboolean glIsTransformFeedback(GLenum id);
```

**描述：**

若id是一个已有的transform feedback对象的名称，则返回GL_TRUE

```c++
void glDeleteTransormFeedbacks(GLsiei n, const GLuint* ids);
```

**描述：**

删除n个transform feedback对象，其名称保存在数组ids中，若ids的某个元素不是transform feedback对象的名称或者尾0，则直接忽略

```c++
void glTransformFeedbackBufferBase(GLuint xfb, GLuint index, GLuint buffer);
```

**描述：**

将名为buffer的缓存对象绑定到名为xfb的transform feedback对象上，其索引有index设置，若index为0，则绑定到默认的transform feedback度夏宁的绑定点

```c++
void glTransformFeedbackBufferRange(GLuint xfb, GLuint index, GLuint buffer, GLintptr offset, GLsizeiptr size);
```

**描述：**

将缓存对象buffer的一部分绑定到名为xfb的transform feedback对象的绑定点索引index上，offset和size的单位均为字节它们定义了要绑定的缓存对象的范围

```c++
void glBindBuffersRange(GLenum target, GLuint first, GLsizei count, const GLuint *buffers, const GLintptr *offsets, const GLsizeiptr* sizes);
```

**描述：**

绑定来自一个或多个缓存的多个范围值，对应于target所指定的目标绑定点

**参数：**

first：绑定缓存范围的第一个索引值

count：绑定的数量

buffers：数组中的count个缓存名称，保存的数值采用字节方式设置

offsets：数组中count个起始地址偏移量，保存的数值采用字节方式设置

sizes：数组中count个绑定范围大小

**其他：**

若buffers为NULL，则offsets和sizes将被忽略，target的索引绑定点上所有的绑定关系会被删除

```c++
void API：glTransformFeedbackVaryings(GLuint program, GLsizei count, const GLchar** varyings, GLenum bufferMode);
```

**描述：**

设置使用varyings记录transform feedback的信息，所用的程序通过program来指定

**参数：**

count：设置varyings数组所包含的字符串的数量，它们存储的也是所有要捕获的变量的名称

bufferMode：设置的是捕获变量的模式，GL_SEPARATE_ATTRIBS（分离模式，每个变量都会记录到一个单独的缓存对象中）、GL_INTERLEAVED_ATTRIBS（交叉模式，所有变量一个接一个记录在绑定到当前transform feedback对象的第一个绑定点的缓存对象中）

```c++
void glBeginTransformFeedback(GLenum primitiveMode);
```

**描述：**

设置transform feedback准备记录的图元类型

**参数：**

primitiveMode：必须是Gl_POINTS、GL_LINES、GL_TRIANGLES之一

**其他：**

之后的绘制命令中的图元类型必须与这里相符，或存在几何着色器，其输出类型必须与这里相符

```c++
void glPauseTransformFeedback(void);
```

**描述：**

暂停transform feedback对变量的记录

**错误：**

若当前transform feedback没有启用或已经处于暂停状态，将产生错误

```c++
void glResumeTransformFeedback(void);
```

**描述：**

重新启用一个之前展厅的transform feedback过程

**错误：**

若transform feedback没有启用或已经启用但没有处于暂停状态，会产生一个错误

```c++
void glEndTransformFeedback(void);
```

**描述：**

完成transform feedback模式下的变量记录过程

### 第6章 纹理与帧缓存



```c++
void ();
```

**描述：**

```c++
void ();
```

**描述：**

```c++
void ();
```

**描述：**



