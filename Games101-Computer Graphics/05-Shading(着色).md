- 词典的定义：绘画中对物体颜色和明暗的处理

  课里的定义：对不同物体应用不同的材质

- 漫反射：当光照射到粗糙的平面上会向四面八方发散

- 着色具有局部性，它不考虑其他物体的存在，只针对当前着色点，即使一个物体将光源挡住

- 摄像机所能接受到的一个点光能量(L漫反射光，k漫反射系数为0时所有光线都被吸收，I/(r^2)光到达着色点的强度，n是法线，l是点到光的向量，nl向量都是单位向量)：该公式基于一个简单的模型，实际上的更复杂
  $$
  L_d=k_d(I/r^2)max(0,\vec{n}\vec{l})
  $$
  可以将k定义成RGB的向量用来表示什么光被吸收了多少从而显示一个点的颜色。而且可以看出漫反射和观察点的方向是没有关系的，只与点所在的平面和光源的位置有关
  
- 高光：当光照射到光滑的平面上会向固定方向及其周围较小范围反射。即只有在观察的方向非常接近镜面反射的方向时会出现高光。

- 半程向量h(观察向量和光发射向量夹角中间的向量)(v是观察向量、l是光发射向量)：
  $$
  h=\frac{v+l}{||v+l||}
  $$
  当观察方向与镜面反射方向接近即代表半程向量和法向量n接近，判断接近用点乘。
  $$
  L_s=k_s(I/r^2)max(0,\vec{n}\vec{h})^p
  $$
  如果通过观察方向和镜面反射方向接近也可以计算，但计算量相比半程向量来说要大许多。
  
  光使用nh的点乘仍然不行，因为cos函数在其角度已经离得较远时，值仍然偏大(距离45度仍有sqrt(2)/2)，随着指数增大可以让角度稍远时，值变换的更接近0，其可以控制高光的范围，越大高光范围越小。一般指数会用100-200的数
  
- 环境光：简化认为任何一个点接受环境光永远都是相同的

  k是环境光系数，I是环境光强
  $$
  L_a=k_aI_a
  $$
  可以发现环境光和光源方向没有关系，和观测方向也没有关系，和法线也没有关系，它就是一个常数。作用是让所有地方都不是黑的

- Binn-Phong反射模型
  $$
  L=L_a+L_d+L_s=k_aI_a+k_d(I/r^2)max(0,\vec{n}\vec{l})+k_s(I/r^2)max(0,\vec{n}\vec{h})^p
  $$
  
- 着色频率：把着色应用到哪些点上

  Flat shading：对三角形的每个顶点进行一个着色，不够平滑

  Gouraud shading：对每个顶点进行着色对内部进行插值方法求值

  Phong shading：对每个像素进行一次着色

  具体使用哪种着色频率更好取决于面的多少，当面足够多的情况下，Flat shading已经足够

- 简单求一个顶点法线的方法：一个顶点被周围几个三角形共用，直接对共用的三角形的法线求加权平均即可，权值取决于三角形的大小

- 求中间像素的法线：知道两个顶点的法线，对两个顶点连线插值

- 图形管线(古老说法，大佬更愿意叫实时渲染管线)：表示经过一系列操作将模型变换成图形的过程。

  OpenGL中图形渲染管线的概念：

  输入一些空间中的点

  将三维空间中的点投影到平面上(Vertex Processing)

  将顶点连成多个三角形(Triangle Processing)

  将三角形光栅化，离散成不同像素(Rasterization)

  对片段进行着色(Fragment Processing)

  进行后处理输出到屏幕上(Framebuffer Operations)

- Shader(着色器)：控制顶点或像素如何着色，着色频率不同使得可以在Vertex Processing阶段和Fragment Processing阶段进行着色，但其他阶段没有。实际上随着图形学的发展，已经不止这两种着色器

- 任何一个三维物体的表面都是二维的

- 纹理映射：定义任何一个点的基本属性

- 纹理坐标(u横向v纵向)：纹理中每个三角形对应的坐标，不管表面展开成什么样子，都约定u和v在0-1范围内

- 纹理：表面上是一张图，但本质上是一块可以进行范围查询的数据，该数据可以是颜色，可以是相对高度等等

- 重心坐标：任意一个点都可以是三个顶点的坐标线性组合而成
  $$
  (x,y)=\alpha A+ \beta B + \gamma C
  $$
  该顶点和三角形在同一平面内则
  $$
  \alpha+\beta+\gamma=1
  $$
  若这个点在三角形内则
  $$
  \alpha>=0,\beta>=0,\gamma>=0
  $$
  ![image-20201117143717660](C:\Users\A\AppData\Roaming\Typora\typora-user-images\image-20201117143717660.png)

  可以使用面积比来求相应值
  $$
  \alpha=\frac{S_a}{S_a+S_b+S_c}，\beta=\frac{S_b}{S_a+S_b+S_c}，\gamma=\frac{S_c}{S_a+S_b+S_c}
  $$

- 可以用重心坐标对三角形进行插值。但是在投影变换中从一个三维空间的三角形变成二维空间的三角形时，重心坐标很可能会改变，所以在三维空间中应使用三维空间中的重心坐标的插值然后投影在二维空间中来

- 运用纹理：屏幕上任何一个采样点有一个位置，就知道其在该位置插值出来的uv在哪，然后在纹理上查询对应uv的点，然后将该uv上的颜色运用到采样点上

- 纹理放大：图片很大，但对应的纹理较小，图片上的 一些点对应的uv就不是整数

  取最近整数：就会对其取整，使一个范围的纹理一样

  双线性插值：根据一个采样点周围的四个像素的中点进行插值，其插值结果比取整好，但比更高级的方法差
  $$
  lerp(x,v_0,v_1)=v_0+x(v_1-v_0)
  $$

  $$
  u_0=lerp(s,u_{00},u_{10})
  $$

  $$
  u_1=lerp(s,u_{01},u_{11})
  $$

  $$
  f(x,y)=lerp(t,u_0,u_1)
  $$

  <img src="C:\Users\A\AppData\Roaming\Typora\typora-user-images\image-20201117162010465.png" alt="image-20201117162010465" style="zoom:80%;" />

- 当纹理过大时，每个采样点都会对应很大一片纹理，造成走样问题

  解决该问题可以使用之前的方法，多取一些采样点然后求均值

  mipmap：可以进行范围查询，查询速度快，但只能进行近似的正方形的范围查询。其是从一张图生成多张图，每次从上一层图的分辨率少一半，总共有log张图，所有图加起来相当于原图的4/3倍，即只比存储原图多了1/3。要运用mipmap要查询对应的图，可以看每个像素对应的第0层纹理的像素大小，将对应的uv近似成一个正方形，观察他包含了多少单位的uv，假设包含了4个，那就是取第二层的，即取log层。若在两层之间，则分别算出结果进行插值即可，称作三线性插值

- 各向异性过滤：可以分别对横向和纵向缩小1/2。比三线性插值更好，因为mipmap只能近似正方形，对一个长宽不同矩形的查询比不上各向异性过滤，但各向异性仍然无法处理斜着的或不规则的图像，且开销是本来的三倍

- EWA过滤：将一个不规则的图像分解成多个椭圆形。但查询和存储代价更大

- 环境光贴图：定义环境光起始于无限远的地方，没有深度的概念，如果将环境光贴图放在一个球上，会展开时会导致扭曲的现象，后就将环境光贴图放在一个由正方体包围的球上，球上的一个点由圆心连线延长至正方体的点上，这种方式很少出现扭曲的现象，但是在计算某个方向的环境光时要进行额外的运算

- 凹凸贴图，通过计算每个点凹凸的切线，通过切线逆时针旋转90度算出法向量，通过法向量计算颜色来欺骗人眼而不会改变模型本身

- 位移贴图，会对顶点本身进行位移，其显示的凹凸效果比凹凸贴图好，但其要求三角形要足够细比纹理的定义的还要高。在DirectX中，位移贴图是可以现在粗糙的三角形上，然后判断哪些三角形需要位移，然后把三角形拆分成小三角形

- ShadowMapping：如果一个点不在阴影里，则该点会被摄像机看到且能被光源直接照射到，而在阴影里说明摄像机能看到该点，但光源不能直接照射到该点。

  提出它为了解决在光栅化过程中无法处理阴影的问题，在使用该纹理的时候，是不需要知道场景中几何的信息，但缺点是容易产生走样现象

  - 经典的ShadowMapping：先从光源看向场景，得到光源能看到什么的图，记录能看到的点对应的深度。再从摄像机出发看向场景，比较摄像机能看到的点的深度和对应的光源图上的深度，当深度一致时说明该点能被光源照射到，当不一致则不能被光源照射到的，那么该点就会处在阴影中。

    其在实际应用中会有问题，第一深度一般是浮点数，而浮点数判断相等是一件很困难的事情。于是就出现了不判断相等，只要记录的值比摄像机看到的深度小，且能接受小距离的偏差，这些仍不能完全解决问题。第二，得到的shadowmapping也有分辨率，所以当分辨率比原图大或小时都会出现纹理过大过小的类似问题。但其在3D游戏中的应用仍然广泛

    只能处理点光源，因为点光源是没有大小的，它有明显的边界，物体要么能被光源照射到，要么不能，这种阴影叫做硬阴影。在物理中，有完全不能看到光源的地方叫本影，也有部分能看到光源的地方叫伴影，从本影过渡伴影的阴影叫做软阴影，软阴影要求光源有一定的大小

