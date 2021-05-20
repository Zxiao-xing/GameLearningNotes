### 3.1 图元

- OpenGL可以支持很多种不同的图元类型，但都可以归结为三种类型：点、线、三角形。

  线和三角形图元类型可以再组合为条带、循环体（线）、扇面（三角形）

- 点、线和三角形是大部分图形硬件所支持的基础图元类型，OpenGL还支持其他一些图元类型，包括作为细分器输入的Patch类型以及作为几何着色器输入的邻接图元

**点**

- 点可以通过单一的顶点来表示，一个点是一个四维的齐次坐标值，点实际上不存在面积，在OpenGL中它是通过显示屏幕或绘制缓存上的一个四边形区域来模拟的
- 当绘制点图元时，OpenGL通过一系列光栅化规则来判断点所覆盖像素的位置，在OpenGL中只要某个采样值落入点在窗口坐标系中的四边形当中，就认为其被覆盖的像素位置。四边形区域的边长等于点的大小，是一个固定的状态，可以通过glPointSize()设置，也可以开启GL_PROGRAM_POINT_SIZE状态后在顶点、细分和几何着色器中通过内置变量gl_PointSize改变，若没有开启状态，则该内置变量的值会被忽略
- 默认点的大小为1.0，当渲染点的时候，默认除被剪切之外顶点都是屏幕上的一个像素，若点大小增加则会占据超过1个像素的区域
- 若使用OpenGL来渲染点，则点的每个片元都会执行片元着色器，在本质上每个点都是屏幕上的方形区域，而每个像素都可以使用不同的颜色进行着色，可以在片元着色器中通过解析纹理图实现点的着色。OpenGL的片元着色器提供gl_PointCoord内置变量，其中包含了当前片元在点区域内的位置信息，其值只对点的渲染有效。若将gl_PointCoord作为输入纹理坐标使用，则可以用位图和纹理替代简单的方块颜色，将结果进行alpha融混或直接discard关键字抛弃某些片元后，还可以创建不同形状的点精灵对象

**线、条带与循环线**

- OpenGL当中的线表示一条线段
- 独立的线可以通过一对顶点来表达，每个顶点表示线的一个端点，多线段可以进行链接来表示一系列的线段，可以是首尾闭合的，闭合的多段线叫做循环线，而没有首尾闭合的叫条带线
- 线从原理上来说也不存在面积，因此需要使用特殊规则来判断线段的光栅化会影响哪些像素值，该规则叫diamond exit规则。其假设每个像素区域内存在一个方形，除了尾端点外的线段穿过了该菱形，就表示该像素收到线段的影响
- diamond exit规则对于西西安段是有效的，OpenGL可以通过glLineWidth()设置线段的宽度大小，线段没有类似gl_PointSize的着色器变量进行设置
- 若线段的宽度大于1，则线段将被水平和垂直复制宽度大小的次数，若是Y主序（即垂直方向延伸），则复制过程是水平方向的，若是X主序，复制过程是垂直方向的
- 若没有开启反走样的话，OpenGL标准对于线段端点的表示方法以及线宽的光栅化方法是相对自由的，若开启了反走样，则线段将被视为沿着线方向对齐的矩形块，宽度等于当前设置的宽度

**三角形、条带与扇面**

- 三角形是由三个顶点的集合组成的，当分别渲染多个三角形的时候，每个三角形与其他三角形完全独立，渲染是通过三个顶点到屏幕的投影以及三条边的链接来完成的

- 若屏幕像素的采样值位于三条边的正侧半空间则受到了三角形的影响，若两个三角形共享了一条边则不可能有任何采样值同时位于两个三角形内

- OpenGL可以支持多种不同的光栅化算法，但对于共享边上的像素值设置却有严格的规定：

  - 两个三角形的共享边上的像素值因为同时被两者所覆盖，不可能不受到光照计算的影响
  - 两个三角形的共享边上的像素值，不可能收到多于一个三角形的光照计算的影响

  即OpenGL对于模型三角形共享边的光栅化过程不会产生任何裂缝，也不会产生重复的绘制

- 绘制三角形条带是前三个顶点构成第一个三角形，后继顶点将和前面的两个顶点构成新的三角形

  绘制三角形扇面，第一个顶点作为共享点存在，而后的每两个顶点都会与该共享点组成三角形

- OpenGL图元的模式标识：

  | 图元类型   | OpenGL枚举量      |
  | ---------- | ----------------- |
  | 点         | GL_POINTS         |
  | 线         | GL_LINES          |
  | 条带线     | GL_LINE_STRIP     |
  | 循环线     | GL_LINE_LOOP      |
  | 独立三角形 | GL_TRIANGLES      |
  | 三角形条带 | GL_TRIANGLE_STRIP |
  | 三角形扇面 | GL_TRIANGLE_FAN   |

- 一个多边形有正面和背面，当不同面朝向观察者时，渲染的结果可能是不一样的。默认情况下，正面背面的绘制方法是一致的，若要改变该属性，或仅使用轮廓线或顶点来进行绘制的话，可以调用GLPolygonMode()命令

- 从惯例来说，多边形正面的顶点在屏幕上应该是逆时针方向排列的，方向一致的多边形构建的实体表面在数学上称为有向流形，如球体、环形体、茶壶，而克林和莫比乌斯带不是

- 可以通过OpenGL函数glFrontFace()来反转背面，并重新设置多边形正面对应的方向

- 对于一个由不透明且方向一致的多边形组成的、完全封闭的模型表面来说，在背面的多边形都是不可见的，会被正面多边形遮挡。若位于模型的外侧可开启裁减来抛弃OpenGL中的背面多边形，若位于模型内，则只有背面多边形是可见的，若需要指示OpenGL自动抛弃正面或背面多边形，可以使用glCullFace()命令，同时通过glEnable开启裁减

- 判断多边形的面是正面还是背面，需要依赖这个多边形在窗口坐标系下的面积计算
  $$
  a = \frac{1}{2}\sum_{i=0}^{n-1}{x_iy_{(i+1)mod(n)}-x_{(i+1)mod(n)}y_i}
  $$
  其中i表达多边形的n个顶点中第i个顶点，x和y对应坐标x、y

  当设置逆时针为正面时，a>0对应的顶点位于正面

### 3.2 OpenGL缓存数据

- 几乎所有使用OpenGL完成的事情都用到了缓存buffers中的数据，OpenGL缓存表示为缓存对象

**创建与分配缓存**

- 步骤：

  1. 缓存对象使用GLuint的值来命名，其使用glCreateBuffers()来创建
  2. 创建之后没有连接到任何存储空间，需要使用glNamedBufferStorage()为每个对象分配存储空间。
  3. 将对象绑定到缓存目标。glBindBuffer()

- 缓存绑定的目标：

  | 目标                                      | 用途                                                         |
  | ----------------------------------------- | ------------------------------------------------------------ |
  | GL_ARRAY_BUFFER                           | 保存glVertexAttribPointer()设置的顶点数组数据                |
  | GL_COPY_READ_BUFFER、GL_COPY_WRITE_BUFFER | 用于拷贝缓存之间的数据，且不会引起OpenGL状态变化，也不会产生任何特殊调用 |
  | GL_DRAW_INDIRECT_BUFFER                   | 若采用间接绘制的方法，该目标用于存储绘制命令的参数           |
  | GL_ELEMENT_ARRAY_BUFFER                   | 可以包含顶点索引数据，以便glDrawElements()等索引形式的函数调用 |
  | GL_PIXEL_PACK_BUFFER                      | 用于从图像对象中读取数据                                     |
  | GL_PIXEL_UNPACK_BUFFER                    | 可以作为glTeXSubImage2D()等命令的数据源                      |
  | GL_TEXTURE_BUFFER                         | 直接绑定到纹理对象的缓存，提供操控纹理缓存的目标，还需要将缓存关联到纹理才能确保在着色器使用 |
  | GL_TRANSFORM_FEEDBACK_BUFFER              | 记录transform feedback属性数据                               |
  | GL_UNIFORM_BUFFER                         | 创建uniform缓存对象的缓存数据                                |

**向缓存输入和输出数据**

- 将数据输入和输出OpenGL缓存的方法很多：直接显式的传递数据、用新的数据替换缓存对象已有的部分数据、由OpenGL负责生成数据然后记录到缓存对象中等

- 像缓存对象中传递数据最简单的方法是在分配内存时读入数据，通过glNamedBufferStorage()

- glNamedBufferStorage()的缓存用途标识符

  | 标识符                 | 意义                                                         |
  | ---------------------- | ------------------------------------------------------------ |
  | GL_DYNAMIC_STORAGE_BIT | 缓存的内容可以随后通过glNamedBuffersSubData()直接进行修改。若未设置，则只能在GPU端完成，如通过着色器 |
  | GL_MAP_READ_BIT        | 可以映射缓存数据到CPU端进行读取，若未设置，该缓存调用glMapNamedBufferTange()获取读取权限会失败 |
  | GL_MAP_WRITE_BIT       | 可以映射魂村数据到CPU端进行写入，若未设置，该缓存调用glMapNamedBufferTange()获取写入权限会失败 |
  | GL_MAP_PERSISTENT_BIT  | 对缓存数据的映射时永久性的，在渲染的过程中始终有效，该标识必须在映射时同时设置才能创建永久映射表 |
  | GL_MAP_COHERENT_BIT    | 缓存数据在GPU端和CPU端映射将保持一致，该标识必须在映射时同时设置才能创建创建一致的映射表 |
  | GL_CLIENT_STORAGE_BIT  | 对不一致内存系统架构来说，有些内存可能从宿主机访问更为高效，而有些从GPU访问效率更高，在确保其他标识量都设置正确后，可以通过该标识量引导OpenGL优先选择CPU端进行访问提高效率 |

- 若需将数据进行紧凑的打包，并且存入一个足够大的缓存对象让OpenGL使用，在内存中数组之间可能是连续的也可能是不连续的，无法简单使用glNamedBufferStorage()来存储数据，以及一次性更新所有数据。需要引入新的glNamedBufferSubData()函数，若将两者结合起来使用，则可以对一个缓存对象进行分配和初始化，然后将数据更新到不同区块中

- 若希望将缓存对象的数据清除为一个已知的值，可以使用glClearNamedBufferData()或glClearNamedBufferSubData()函数，其允许初始化缓存对象中存储的数据，且不需要保留或清除任何一处系统内存

- 缓存对象中的数据可以使用glCopyNamedBufferSubData()函数互相进行拷贝，该函数对于较大魂村中的数据依次进行组装的做法不同，此时可以使用glNamedBufferStorage()将数据更新到独立的缓存当中，然后将这些缓存直接用glCopyNamedBufferSubData()拷贝到一个较大缓存中，或分配一系列缓存对象，然后循环两两操作，实现拷贝数据的叠加

- glCopyNamedBufferSubData()可以在两个目标对应的缓存之间拷贝数据，而GL_COPY_READ_BUFFER和GL_COPY_WRITE_BUFFER目标是为此存在，其不能用于其他OpenGL操作，并且若将缓存与之绑定，只用于拷贝和存储目的，不影响OpenGL的状态也不需要记录拷贝之前的目标区域，则整个操作过程保证是安全的

- 可以通过多种方式从缓存对象中回读数据：

  1. 使用GLGetNamedBufferSubData()函数，其可以从绑定到某个目标的缓存中回读数据，然后将它放置到应用程序保有的一处内存当中，若使用OpenGL生成了一些数据，且希望重新获取到它们的内容，则应该使用该函数
  2. 使用glGetBufferSubData()简单的将之前存入到缓存对象中的数据读回到内存中

**访问缓存的内容**

- glNamedBufferSubData()会将应用程序内存中的数据拷贝到OpenGL管理的内存中

  glCopyNamedBufferSubData()会将源缓存中的内容拷贝到另一个缓存或同一个缓存的不同位置

  glGetNamedBufferSubData()将魂村对象的数据拷贝到应用程序中

  它们都会导致OpenGL进行一次数据拷贝的操作

- 可以通过glMapBuffer()获取一个指针的形式，直接在应用程序中对OpenGL管理的内存进行访问。该函数返回一个指针，指向绑定到target的缓存对象的数据区域所对应的内存，该内存只是对应这个缓存对象本身，不一定是图形处理器用到的内存区域

- access参数指定了应用程序对于映射后的内存区域的使用方式：

  | 标识符        | 意义                                         |
  | ------------- | -------------------------------------------- |
  | GL_READ_ONLY  | 应用程序仅对OpenGL映射的内存区域执行读操作   |
  | GL_WRITE_ONLY | 应用程序仅对OpenGL映射的内存区域执行写操作   |
  | GL_READ_WRITE | 应用程序对OpenGL映射的内存区域执行读或写操作 |

- 当结束数据的读取或写入到缓存对象的操作后，必须使用glUnmapNamedBuffer()执行解除映射操作。若解除了缓存的映射，则之前写入到OpenGL映射内存中的数据将会重新对缓存可见

- 为了避免glMapBuffer()可能造成的缓存映射问题，glMapNamedBufferRange()函数使用额外标识符来更精准的设置访问模式，对于该函数，access位域中必须包含GL_MAP_READ_BIT和GL_MAP_WRITE_BIT中的一个或两个，以确认应用程序是否要对映射数据进行读写操作

- glMapNamedBufferRange()使用的标识符

  | 标识符                       | 意义                                                         |
  | ---------------------------- | ------------------------------------------------------------ |
  | GL_MAP_INVALIDATE_RANGE_BIT  | 给定的缓存区域内任何数据都可以被抛弃以及吴晓华，若给定区域范围内任何数据没有被随后写入的话，将变成未定义的数据。该标识符无法与GL_MAP_READ_BIT同时使用 |
  | GL_MAP_INVALIDATE_BUFFER_BIT | 缓存的整个内容都可以被抛弃和无效化，不再受到区域范围的设置影响，所有映射返回之外的数据都会变成未定义的状态，给定区域范围内任何数据没有被随后写入的话，将变成未定义的数据。该标识符无法与GL_MAP_READ_BIT同时使用 |
  | GL_MAP_FLUSH_EXPLICIT_BIT    | 应用程序将负责通知OpenGL映射范围内的哪个部分包含可用数据，其在glUnmapNamedBuffer()前调用。若缓存中较大范围内的数据都会被映射，而不是全部被应用程序写入，使用该标识符。该标识符必须和GL_MAP_WRITE_BIT结合使用，若该标识符没有被设置的话，glUnmapNamedBuffer()会自动刷新整个映射区域的内容 |
  | GL_MAP_UNSYNCHRONIZED_BIT    | 若该标识符没有设置，则OpenGL会等待所有正在处理的缓存访问操作结束，然后返回映射范围的内存，若设置，则不会尝试进行缓存同步操作 |

  - 若通过GL_MAP_INVALIDATE_RANGE_BIT或GL_MAP_INVALIDATE_BUFFER_BIT实现缓存数据的无效化，则意味着OpenGL可以对缓存对象中的任何已有的数据进行清理。除非确信要同时使用GL_MAP_WRITE_BIT对缓存进行写入操作，否则不要设置两个标识符中的任意一个。若设置GL_MAP_INVALIDATE_RANGE_BIT，目的是对某个区域的整体进行更新。若设置GL_MAP_INVALIDATE_BUFFER_BIT，意味着不再关心没有被映射的缓存区域的内容，或准备在后继的映射当中对缓存剩下的部分进行更新
  - GL_MAP_UNSYNCHRONIZED_BIT用于禁止OpenGL数据传输和使用时的自动同步机制，若确保之后逇操作可以在真正修改缓存内容前完成，且禁止同步功能以提升性能
  - GL_MAP_FLUSH_EXPLICIT_BIT表明了应用程序将通知OpenGL修改了缓存的哪些部分，再调用glUnmapNamedBuffer()。通知操作可以使用glFlushMappedNamedBufferRange(),可以对缓存对象中独立的或是互相重叠的映射范围多次调用该函数，映射范围必须通过glMapNamedBufferRange()加上GL_MAP_FLUSH_EXPLICIT_BIT来映射。执行该操作后，会假设OpenGL对于映射缓存对象中指定区域的修改已经完成，且开始执行一些相关操作

**丢弃缓存数据**

- 若要抛弃缓存对象的部分或全部数据，可以调用glInvalidateBufferData()或glInvalidateBufferSubData()函数
- 若调用glBufferData()并传入一个NULL指针，所实现的功能与glInvalidateBufferData()相似，都会通知OpenGL实现可以安全的抛弃缓存中的数据，但前者会重新分配内存区域，所有后者相对更优化。
- glInvalidateBufferSubData()是唯一一个可以抛弃缓存对象中的区域数据的方法

### 3.3 顶点规范

**深入讨论VertexAttribPointer**

- glVertexAttribPointer()的数据类型标识符

  | 标识符                         | OpenGL类型                        |
  | ------------------------------ | --------------------------------- |
  | GL_BYTE                        | GLbyte（有符号8位整型）           |
  | GL_UNSIGNED_BYTE               | GLubyte（无符号8位整型）          |
  | GL_SHORT                       | GLshort（有符号16位整型）         |
  | GL_UNSIGNED_SHORT              | GLushort（无符号16位整型）        |
  | GL_INT                         | GLint（有符号32位整型）           |
  | GL_UNSIGNED_INT                | GLuint（无符号32位整型）          |
  | GL_FIXED                       | GLfixed（有符号16位定点型）       |
  | GL_FLOAT                       | GLfloat（32位IEEE单精度浮点数）   |
  | GL_HALF_FLOAT                  | GLhalf（16位S1E5M10半精度浮点型） |
  | GL_DOUBLE                      | GLdouble（64位IEEE双精度浮点型）  |
  | GL_INT_2_10_10_10_REV          | GLuint（压缩数据类型）            |
  | GL_UNSIGNED_INT_2_10_10_10_REV | GLuint（压缩数据类型）            |

- 若glVertexAttribPointer()函数中的type传入了整数类型，则只能将这些数据类型存储到缓存对象的内存中，必须将这类数据转换为浮点数才能读取到浮点数的顶点属性中。可以使用normalize参数进行控制，若为GL_FALSE则整数将直接被强制转换为浮点数类型，然后传入到顶点着色器中，若为GL_TRUE，则数据在传入到顶点着色器之前需首先进行归一化，OpenGL使用一个固定的依赖于输入数据类型的常数去除每个元素。

  若为有符号的
  $$
  f = \frac{c}{2^b - 1}
  $$
  若为无符号的
  $$
  f = \frac{2c + 1}{2^b - 1}
  $$
  其中f就是浮点数值，c表示输入的整数分量，b表示数据类型的位数

- 浮点数会造成精度丢失，因此大范围的整数值不能直接使用浮点型属性传入顶点着色器，需要引入整型顶点属性：int、ivec2、ivec3、ivec4以及对应的无符号属性

- 通过GLVertexAttribIPointer()将整数传递到顶点属性中，其不会执行自动转换到浮点数的操作，type参数只能使用整型类型

- glVertexAttribLPointer()专门用于将属性数据加载到64位的双精度浮点型顶点属性中。若glVertexAttribPointer()使用GL_DOUBLE类型，实际在传递到顶点着色器之前会自动转换到32位单精度浮点型法师，即使目标顶点属性已经声明为双精度类型，前者可以保证精度

- GL_INT_2_10_10_10_REV以及GL_UNSIGNED_INT_2_10_10_10_REV标识符标识了一种有四个分量的数据格式，前三个分量占据10字节，第四个占据2字节，即压缩后的大小是一个32位单精度数据（GLuint）。这种数据排布方式对于法线等分类型的属性设置特别有益处，三个主要分量的大小均为10位，因此精度可以得到额外提高，且此时通常不需要达到半浮点数的进度级别，节约了内存空间和系统带宽，有助于提升程序性能

**静态顶点属性的规范**

- 在OpenGL从缓存中读取数据之前，必须使用glEnableVertexAttribPointerArray ()启用对应的顶点属性数组，若某个顶点属性对应的属性数组未启用，OpenGL会用静态顶点属性，每个顶点的静态顶点属性都是一个默认值。

- 若模型中所有的顶点或一部分顶点的属性值是相同的，使用一个常数值用来填充数据缓存，这是一种内存浪费和性能损失，所以可以禁止顶点属性数组，使用静态的顶点属性值来设置顶点属性

- 每个属性的静态顶点属性可以通过glVertexAttrib*()系列函数设置

  - 若顶点属性在顶点着色器中是一个浮点型的变量，则使用glVertexAttrib{1234}{fds}/{1234}{fds}v/{bsifd ub us ui}v()系列函数。这些函数会自动将非浮点数的输入参数强制转换为浮点数，然后传递到着色器中
  - 对于传入整型数值的情况，可以使用glVertexAttrib4N/4Nub()将数据根绝有符号无符号类型归一化到[0, 1]或[-1, 1]范围内。这些函数中输入参数仍会被转换到浮点数的形式，然后传递给顶点着色器，因此只能用来设置单精度浮点数类型的静态数据
  - 若属性变量必须为整数，则使用glAttribl{1234}{i ui}/{123}{i ui}v/4{bsi ub us ui}v
  - 若顶点属性声明为双精度浮点数类型，应使用glVertexAttribL{1234}/{1234}v

  若使用该系列函数，但传递给顶点属性的分量个数不足，则缺少的分量将自动填充为默认的值，对于w分量，默认值为1.0，而 y 和 z 分量的默认值为0.0，x分量不可能缺少

  若函数中包含的分量个数多余着色器中顶点属性的声明个数，则多余的部分将被抛弃

- 静态顶点属性值保存在当前VAO中，而不是程序对象

### 3.4 OpenGL绘制命令

- 大部分OpenGL绘制命令都是以Draw单词开始，绘制命令大致分为索引形式和非索引形式。索引形式的绘制需要用到绑定GL_ELEMENT_ARRAY_BUFFER的缓存对象中存储的索引数组，可以用来间接的对已经启用的顶点数组进行索引；非索引的绘制则不需要绑定，只需简单的按顺序读取顶点数据即可
- OpenGL中，最基本的非索引形式绘制命令是glDrawArrays()，最基本的索引形式绘制命令是glDrawElements()，这些函数都会从当前启用的数组中读取顶点信息，然后使用它们来构建mode指定的图元类型。前者只是直接将缓存对象中的定顶点属性按照自身的排列顺序直接取出并使用，后者使用了元素数组缓存中的索引数据来索引各个顶点属性数组。所有更复杂的OpenGL绘制函数，本质上基于这两个函数来完成功能实现
- glDrawElementsBaseVertex()可以将元素数组缓存中的索引数据进行一个固定数量的偏移
- glDrawRangeElements()与glDrawElements()类似，其是一种更严格的形式
- 可以使用功能组合来实现更高级的命令，例如glDrawRangeElementsBaseVertex()相当于glDrawElementsBaseVertex()和glDrawRangeElements()功能的组合形式
- 间接绘制函数，他们的参数不是直接从程序中得到，而是从缓存对象中获取的，若要使用必须先将一个缓存对象绑定到GL_DRAW_INDIRECT_BUFFER目标上，glDrawArrays()和glDrawElements()的间接版本分别为glDrawArraysIndirect()和glDrawElementsIndirect()
- 不以Draw开头的绘制命令，属于命令的多变量形式，包括glMultiDrawArrays()、glMultiDrawElements()、glMultiDrawElementsBaseVertex()，每个函数都记录了一个first参数的数组，一个count参数的数组，工作方式相当于对每个数组的元素，都会执行一次单一变量函数
- 若有大象的绘制内容需要处理，且相关参数已经保存到一个缓存对象中，可以直接使用glDrawArraysIndirect()或glDrawElementsIndirect()处理的话，也可以使用两者的多变量版本glMultiDrawArraysIndirect()或glDrawMultiElementsIndirect()

**图元的重启动**

- OpenGL支持在同一个渲染命令中进行图元重启动的功能，此时需要指定一个特殊的值，叫做图元重启动索引，OpenGL内部会对齐做特殊的处理，若绘制调用过程中遇到了这个重启动索引，那么将从这个索引之后的顶点开始。该索引的定义是通过glPrimitiveRestartIndex()完成的
- 若顶点的渲染需要在某一个glDrawElements()系列函数调用中完成，则可以用glPrimitiveRestartIndex()所指定的索引，并且检查这个索引值是否会出现在元素数组缓存中。必须启动图元重启动特性后才可以进行这种检查，通过GLEnable和glDisabled函数设置参数为GL_PRIMITIVE_RESTART来启动和关闭特性

**多实例渲染**

- 实例化或多实例渲染是一种连续执行多条相同的渲染命令的方法，且每个渲染命令所产生的结果都会有轻微的差异，这是一种非常有效的使用少量API调用来绘制大量几何体的方法，OpenGL中提供了多种机制允许着色器使用绘制的不同实例作为输入，并且对每个实例都赋予不同的顶点属性值
- 最简单的多实例渲染命令是glDrawArraysInstanced()，其对应glDrawArrays()。glDrawElementsInstanced()对应glDrawElements()，glDrawElementsInstancedBaseVertex()对应glDrawElementsBaseVertex()
- 多实例的顶点属性与正规的顶点属性是类似的，它们在顶点着色器中的声明和使用方式都是完全一致的
- glVertexAttribDivisor()函数用来启用多实例的顶点属性，主要是用来控制顶点属性的更新频率
- 可以在系统内部设置一个偏移值，以改变顶点缓存中得到实例化的顶点属性时的索引位置，在多实例版本中，实例的索引值通过一个额外的baseInstance参数设置。包括：glDrawArraysInstancedBaseInstance()、glDrawElementsInstancedBaseInstance()、glDrawElementsInstancedBaseVertexBaseInstance()。

### 示例

**例3.1**

```c++
//顶点位置
static const GLfloat positions[] = {
	-1.0f, -1.0f, 0.0f, 1.0f,
    1.0f, -1.0f, 0.0f, 1.0f,
    1.0f, 1.0f, 0.0f, 1.0f,
    -1.0f, 1.0f, 0.0f, 1.0f
};
//顶点颜色
static const Glfloat colors[] = {
    1.0f, 0.0f, 0.0f,
    0.0f, 1.0f, 0.0f,
    0.0f, 0.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
}
//缓存对象
GLuint buffer;
//创建缓存对象
glCreateBuffers(1, &buffer);
//分配空间
glNameBufferStorage(buffer, sizeof(positions) +sizeof(colors), nullptr, GL_DYNAMIC_STORAGE_BIT);
//将位置信息放置在偏移地址为0的位置
glNamedBufferSubData(buffer, 0, sizeof(positions), positions);
//放置在缓存中的颜色信息的偏移地址为当前填充大小值的位置
glNamedBufferSubData(buffer, sizeof(positions), sizeof(colors), 					positions);
```

**例3.2**

```c++
GLuint buffer;
FILE* f;
size_t filesize;

f = fopen("data.dat", "rb");
fseek(f, 0, SEEK_END);
filesize = ftell(f);
fseek(f, 0, SEEK_SET);
//生成缓存对象并绑定
glGenBuffers(1, &buffer);
glBindBuffer(GL_COPY_WRITE_BUFFER, buffer);
//分配缓存中存储的数据空间
glBufferData(GL_COPY_WRITE_BUFFER, (GLsizei)filesize, NULL, 				GL_STATIC_DRAW);
//映射缓存
void* data = glMapBuffer(GL_COPY_WRITE_BUFFER, GL_WRITE_ONLY);
//将文件读入缓存
fread(data, 1, filesize, f);
//解除映射
glUnMapBuffer(GL_COPY_WRITE_BUFFER);
fclose(f);
```

**例3.5**

```c++
//4个顶点
static const GLfloat vertex_positions[] = {
    -1.0f, -1.0f, 0.0f, 1.0f,
    1.0f, -1.0f, 0.0f, 1.0f,
    -1.0f, 1.0f, 0.0f, 1.0f,
    -1.0f, -1.0f, 0.0f, 1.0f,
};
//每个顶点的颜色
static const GLfloat vertex_colors[] = {
    1.0f, 1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 0.0f, 1.0f,
    1.0f, 0.0f, 1.0f, 1.0f,
    0.0f, 1.0f, 1.0f, 1.0f,
};
//三个索引值
static const GLushort vertex_indices[] = {
    0, 1, 2
};
//设置元素数组缓存
glGenBuffers(1, ebo);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo[0]);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(vertex_indices), 				vertex_indices, GL_STATIC_DRAW);
//设置顶点属性
glGenVertexArrays(1, vao);
glBindVertexArray(vao[0]);

glGenBuffers(1, vbo);
glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertex_positions) + 					sizeof(vertex_colors), NULLm GL_STATIC_DRAW);
glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(vertex_positions), 					vertex_positions);
glBufferSubData(GL_ARRAY_BUFFER, sizeof(vertex_positions), 						sizeof(vertex_colors), vertex_colors);
//使用glDrawArrays
mode_matrix = vmath::translation(-3.0f, 0.0f, -5.0f);
glUniformMatrix4fv(render_model_matrix_loc, 4, GL_FALSE, 							model_matrix);
glDrawArrays(GL_TRIANGLES, 0, 3);
//使用glDrawElements
model_matrix = vmath::translation(-1.0f, 0.0f, -5.0f);
glUniformMatrix4fv(render_model_martrix_loc, 4, GL_FALSE, 							model_matrix);
glDrawElements(GL_TRIANGLES, 3 GL_UNISIGNED_SHORT, NULL);
//使用glDrawElementsBaseVertex
model_matrix = vamth::traslation(1.0, 0.0f, -5.0f);
glUniformMatrix4fv(render_model_martrix_loc, 4, GL_FALSE, 							model_matrix);
glDrawElementsBaseVertex(GL_TRIANGLES, 3, GL_UNSIGNED_SHORT, NULL, 							1);
//使用glDrawArraysInstanced
model_matrix = vamth::traslation(3.0, 0.0f, -5.0f);
glUniformMatrix4fv(render_model_martrix_loc, 4, GL_FALSE, 							model_matrix);
glDrawArraysInstanced(GL_TRIANGLES, 0, 3, 1);
```

**例3.7**

```c++
//立方体的八个顶点，边长为2
static const GLfloat cube_positions[] = {
	-1.0f, -1.0f, -1.0f, 1.0f,
	-1.0f, -1.0f, 1.0f, 1.0f,
	-1.0f, 1.0f, -1.0f, 1.0f,
	-1.0f, 1.0f, 1.0f, 1.0f,
	1.0f, -1.0f, -1.0f, 1.0f,
	1.0f, -1.0f, 1.0f, 1.0f,
	1.0f, 1.0f, -1.0f, 1.0f,
	1.0f, 1.0f, 1.0f, 1.0f,
};
//每个顶点的颜色
static const GLfloat cube_colors[] = {
	1.0f, 1.0f, 1.0f, 1.0f,
	1.0f, 1.0f, 0.0f, 1.0f,
	1.0f, 0.0f, 1.0f, 1.0f,
	1.0f, 0.0f, 0.0f, 1.0f,
	0.0f, 1.0f, 1.0f, 1.0f,
	0.0f, 1.0f, 0.0f, 1.0f,
	0.0f, 0.0f, 1.0f, 1.0f,
	0.5f, 0.5f, 0.5f, 1.0f,
};
//三角形条带的索引
static const GLushort cube_indices[] = {
	0, 1, 2, 3, 6, 7, 4, 5,			//第一条
	0xFFFF,					//重启动索引
	2, 6, 0, 4, 1, 5, 3, 7
};
	//设置元素数组缓存
	GLuint* ebo;
	glGenBuffers(1, ebo);
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo[0]);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(cube_indices), cube_indices, GL_STATIC_DRAW);
	//设置顶点属性
	GLuint* vao;
	glGenVertexArrays(1, vao);
	glBindVertexArray(vao[0]);
	GLuint* vbo;
	glGenBuffers(1, vbo);
	glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
	glBufferData(GL_ARRAY_BUFFER, sizeof(cube_positions) + sizeof(cube_colors), NULL, GL_STATIC_DRAW);
	glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(cube_positions), cube_positions);
	glBufferSubData(GL_ARRAY_BUFFER, sizeof(cube_positions), sizeof(cube_colors), cube_colors);
	glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 0, NULL);
	glVertexAttribPointer(1, 4, GL_FLOAT, GL_FALSE, 0, (const GLvoid *)sizeof(cube_positions));
	glEnableVertexAttribArray(0);
	glEnableVertexAttribArray(1);
```

**例3.8**

```c++
	glBindVertexArray(vao[0]);
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo[0]);
#if USE_PRIMITIVE_RESTART
	//若开启图元重启动，则只需要调用一次绘制命令
	glEnable(GL_PRIMITIVE_RESTART);
	glPrimitiveRestartIndex(0xFFFF);
	glDrawElements(GL_TRIANGLE_STRIP, 17, GL_UNSIGNED_SHORT, NULL);
#else
	glDrawElements(GL_TRIANGLE_STRIP, 8, GL_UNSIGNED_SHORT, NULL);
	glDrawElements(GL_TRIANGLE_STRIP, 8, GL_UNSIGNED_SHORT, (const GLvoid*)(9 * sizeof(GLushort)));
#endif
```

**例3.9**

多实例顶点着色器属性

```glsl
#version 410 core
//位置和发现都是规则的顶点属性
layout (location = 0) in vec4 position;
layout (location = 1) in vec3 normal;
//颜色是一个逐实例的属性
layout (location = 2) in vec4 color;
//model_matrix是一个逐实例的变换矩阵，一个mat4会占据4个连续位置
//实际占据了3、4、5、6四个索引位
layout (location = 3) in mat4 model_matrix;
```

多实例顶点属性的设置

```c++
//prog是用于渲染的着色器对象，该步骤不是必须的，因为上面着色器设置了所有
//属性的位置
int position_loc = glGetAttribLocation(prog, "position");
int normal_loc = glGetAttribLocation(prog, "normal");
int color_loc = glGetAttribLocation(prog, "color");
int matrix_loc = glGetAttribLocation(prog, "model_matrix");
//配置正规的顶点属性数组
glBindBuffer(GL_ARRAY_BUFFER, position_buffer);
glVertexAttribArray(position_loc, 3, GL_FLOAT, GL_FALSE, 0, NULL);
glEnableVertexAttribArray(position_loc);
glBindBuffer(GL_ARRAY_BUFFER, normal_buffer);
glVertexAttribPointer(normal_loc, 3, GL_FLOAT, GL_FALSE, 0, NULL);
glEnableVertexAttribArray(normal_loc);
//设置颜色的数组
glBindBuffer(GL_ARRAY_BUFFER, color_buffer);
glVertexAttribPointer(color_loc, 4, GL_FLOAT, GL_FALSE, 0, NULL);
glEnableVertexAttribArray(color_loc);
//设置更新频率为1
glVertexAttribDivisor(color_loc, 1);

glBindBuffer(GL_ARRAY_BUFFER, model_matrix_buffer);
for(int i = 0; i < 4; i++){
    glVertexAttribPointer(matrix_loc+i, 4, GL_FLOAT, GL_FALSE, 					sizeof(mat4), (void *)(sizeof(vec4) + i));
    glEnableVertexAttribArray(matrix_loc + i);
    glVertexAttribDivisor(matrix_loc + i , 1);
}
```

多实例属性的顶点着色器

```glsl
//观察矩阵和投影矩阵在绘制过程中都是常数
uniform mat4 view_matrix;
uniform mat4 projection_matrix;
out VERTEX{
	vec3 normal;
	vec4 color;
}vertex;
void main(){
    //构建模型-视图矩阵
	mat4 model_view_matrix = view_matrix * model_matrix;
    //使用模型-视图变换，然后是投影变换
    gl_Position = ptojrvyion_matrix * (model_view_matrix * 							position);
    //使用模型-视图矩阵的左上3x3字矩阵变换法线
    vertex.normal = mat3(model_view_matrix) * normal;
    //将逐实例的颜色值春如片元着色器
    vertex.color = color;
}
```

多实例绘制

```c++
//映射缓存
mat4* matrices = (mat4*)glMapBuffer(GL_ARRAY_BUFFER, 							GL_WRITE_ONLY);
//设置每个实例的模型矩阵
for(n = 0; i < INSTANCE_COUNT; n++){
	float a = 50.0f * float(n) / 4.0f;
	float b = 50.0f * float(n) / 5.0f;
	float c = 50.0f * float(n) / 6.0f;
	matrices[n] = rotation(a + t * 360.0f, 1.0f, 0.0f, 0.0f) *
				rotation(b + t * 360.0f, 0.0f, 1.0f, 0.0f) *
				rotation(c + t * 360.0f, 0.0f, 0.0f, 1.0f) *
				translation(10.0f + a, 40.0f + b, 50.0f + c)
}
//完成后解除映射
glUnmapBuffer(GL_ARRAY_BUFFER);
//启用多用例的程序
glUseProgram(render_prog);
//设置观察矩阵和投影矩阵
mat4 view_matrix(translation(0.0f, 0.0f, -1500.0f) * 
			rotation(t * 360.0f * 2.0f, 0.0f, 1.0f, 0.0f));
mat4 projection_matrix(frustum(-1.0f, 1.0f, -aspect, aspect, 1.0f, 				5000.0f));
glUniformMatrix4fv(view_matrix_loc, 1, GL_FALSE, view_matrix);
glUniformMatrix4fv(projection_matrix_loc, 1, GL_FALSE, 						projection_matrix);
//渲染INSTANCE_COUNT个模型
glDrawArraysInstanced(GL_TRIANGLES, 0, object_size, 						INSTANCE_COUNT);
```

