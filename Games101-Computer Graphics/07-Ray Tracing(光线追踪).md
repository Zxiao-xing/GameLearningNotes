- 光栅化和光线追踪的对比：

  光栅化很难表现出全局的效果，如阴影、间接光照、，对Glossy reflection(拥有反射效果和一些高光但表面仍不是非常光滑)、间接光照(一个光通过不止一次镜面反射或者漫反射最后进入视野中)的效果也很难达到高效果，他本质上只能对这些效果进行近似而不能准确的表达出这些效果

  光线追踪很慢，经常用于做离线的东西(如电影)，其质量非常高

- 光线：直线传播、光线间不发生碰撞、从光源出发通过不断的反射追踪进入人的视野、具有可逆性(可以认为从眼睛发出感知光线通过不断反射打到光源上，将光线经过的路径称为光路)

  光线追踪实际上就是利用光路可逆性，从眼睛出发，最终到达光源上

- 光线追踪假设人眼是一个位置而不具有大小，光源也假设是点光源，光源打到物体也会发生完美的反射或折射。

- 光线着色：每个像素投出去一根光线和场景相交求最近交点，然后交点和光源连线判定是否对光源可见，然后着色写入像素值

**Whitted-Style Ray Tracing**

- 在任意一个点可以继续传播光线，只需要算出折射和反射方向，在后在每一个弹射点都会计算着色的值，然后将每个点的值都加回发射的像素点

- 任意一个光线：其中o是光源，t是时间(可以为负，表示反向的光线)，d是发射方向向量
  $$
  r(t)=o+td
  $$

- 求光线和隐式几何的交点，只需要找到满足两个式子的点，然后找距离最近的那个点即可

- 求光线和显示几何的交点：简单方式，所有一个三角形都与光线求交点，要么0个要么1个交点，转换成求平面和光线相交，再判断交点是否在三角形内，平面可以用法线和平面上的点定义平面的公式，然后再用隐式的方式来求

- 使用包围盒来控制三角形的数量，用一个简单的物体将模型框起来，如果光线连包围盒都碰不到，则一定不能碰到物体。包围盒最好使用和轴对齐(就是和分别和坐标轴平行垂直)的几对面，这样可以减少运算，这种包围盒叫轴对齐包围盒(Axis-Aligned Bounding Box, AABB)

  当光线进入所有对面的中间则光线进入盒子，当光线离开一对面的中间    则光线离开盒子，所以对每一对面计算进入t，当进入所有对面后求最大的时间即是进入t，对出去盒子的t则用离开盒子的t的最小值即可，如果进入时间小于离开时间，则光线肯定是和盒子有交点

  若光线离开盒子的t小于0，则盒子在光线的背后，所以是不可能有交点的
  
  当离开t是非负的，进入t是负的，则光线的起点就在盒子里，所以肯定是有交点的
  
  综上所述，若要光线和盒子有交点，当且仅当进入t小于离开t，且离开t非负
  
  使用中，将包围盒又分成若干个小的形状，然后进行预处理，标记有包括有物体表面的小形状，然后对光线经过的每个小形状进行检查，若发现该形状包含有物体的表面，再判断物体表面是否相交。具体划分有多少个小形状则需取一个折中的值
  
  当一根光线打在物体上，如果有奇数个交点，则光线一定处于物体内，偶数个则处于物体外
  
- Spatial Partition(空间划分)

  Oct-Tree(八叉树)：将空间中的包围盒沿着x，y，z的中点切成8分，然后对每一个子节点再切成8分，一直切到达到定义的规则为止。但是其在二维空间中是使用4叉树，在更改维度要使用2^n叉树，这是一个随维度变换的结构

  KD-Tree(kd树)：和8叉树很相似，但其在找到一个格子时，会沿着某个轴划分，划分是交替划分，例如在二维图中，先进行水平划分后再对两个划分的块进行垂直划分，最后的结果类似于二叉树的结构。在查找结果时，相当于二分查找，当与某个块有交点时，就和块中的每个物体判断是否有交点。存储方式是中间节点存包围盒，一直到叶子及节点存物体信息

  BSP-Tree：每次选一个方向划分一次，相比于kd不再是沿着某个轴划分，不满足AABB，所以其比不上kd树

  空间划分存在着一个问题，就是判断包围盒与空间中三角形相交是一个非常难的问题，且在划分时，一个物体和多个划分的包围盒有交点时，需要物体存储在多个包围盒中

- Object Partition(物体划分)：解决了空间划分的两个问题。

  Bounding Volume Hierarchy(BVH)：将物体划分为两个部分，然后对两部分分别求包围盒，将两包围盒存在二叉树中，然后再对两部分继续划分，将结果存在子树中，一直划分到指定规则为止。但这样划分出来的包围盒是存在重叠的，所以要在划分时使重叠部分最小。衍生出来很多的做法，比如为了让树的两边平衡，将较长的轴分成两半，并从中位三角形进行划分。

  存储方式是中间节点存包围盒，叶子节点存物体信息

**Basic radiometry(辐射度量学)**

- 是一个精准的实际物理量的光计算方法。它表示如何去描述光照，定义了一系列方法和单位，给光以各种属性(空间中的，不包含时间属性)，仍然基于几何光学认为光线是直线传播不具有波动性。属性包括辐射通量(Radiant flux)、强度(intensity)，照度(irradiance)，亮度(radiance)

- Radiant Energy and Flux(Power)：能量和单位时间的能量，图形学中不会使用实际的能量，只会使用单位的能量

- Radiant Intensity：一个单位立体角的能量。用能量除以4pi就可以得到单位立体角的能量

  立体角：二维中的弧度的衍生，从一个球的中心出发，打到球表面形成一个锥，打到球表面面积为A，半径为r
  $$
  \Omega=\frac{A}{r^2}
  $$
  假设一个xyz轴的右手坐标系，向量向y轴偏移为theta角，向x轴偏移为phi角，那么向量偏移极小的theta和phi角就是微分立体角
  $$
  d\omega=\frac{dA}{r^2}=sin\theta d\theta d\Phi
  $$
  
- Irradiance：一个物体表面接受到多少能量，即光在投影面上的能量
  $$
  E(x)=\frac{d\Phi(x)}{dA}
  $$
  
- Radiance：度量光线在传播中的能量，单位立体角上的单位面积中的能量为多少
  $$
  L(p,\omega)=\frac{d^2\Phi(p,\omega)}{d\omega dA cos\theta}
  $$

- Irradiace和Radiance在图形学中用的多，两者关系为后者是前者在特定立体角收到的能量，前者是后者所有方向收到的能量。对于点p的能量
  $$
  E(p)=\int_{H^2}L_{i}(p,\omega)cos\theta d\omega
  $$

- 双向反射分布函数(BRDF)：定义一个点如何向各个方向分配吸收到的能量，它定义了 不同的材质
  $$
  f_{r}(\omega_i\to\omega_r)=\frac{dL_r(\omega_r)}{dE_i(\omega_i)}=
  \frac{dL_r(\omega_r)}{L_i(\omega_i)cos\theta_id\omega_i}[\frac{1}{sr}]
  $$

- 渲染方程：一个点出射的光由自己产生的光加上别人反射过来的
  $$
  L_o(p,\omega_o)=L_e(p,\omega_o)+\int_{\Omega+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\omega_i)d\omega_i
  $$
  简化：
  $$
  I(u)=e(u)+\int I(v)K(u,v)dv
  $$
  再简化：
  $$
  L=E+KL
  $$

  $$
  L=(I-K)^{-1}L
  $$

  泰勒展开：
  $$
  L=E+KE+K^2E+K^3E+...
  $$
  该式可以理解为一个点的Radiance等于自身的光照加上一次反射的光照加上二次反射的光照一直累加到n次

  通过能量守恒或者实际可以发现，光在弹射到无穷次只会收敛于一种光照

- 全局光照：所有光线的弹射次数加起来就是全局光照，即直接加间接

- MonteCarlo积分：求一段概率密度函数的定积分，在积分域内不断采样，然后采样结果相加，即一个积分可以近似为
  $$
  F_N=\frac{1}{N}\sum_{i=1}^N\frac{f(X_i)}{p(X_i)}
  $$
  当积分处于a-b之间，且和为1时
  $$
  \int f(x)dx=\frac{b-a}{N}\sum_{i=1}^N{f(X_i)}
  $$
  这种积分要求必须是对应函数轴的采样

- 通过MonteCarlo积分解渲染方程
  $$
  L_o(p,\omega_o)=\frac{1}{N}\sum_{i=1}^N\frac{L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\omega_i)}{p(\omega_i)}
  $$
  其中p(wi)=1/2pi

- 光线追踪不能很好体现出Clossy材质，而且光线追踪只能显示直接光照的结果，*不能体现出全局光照的结果*

- 路径追踪算法

  ```c++
  shade(p,wo)
      Randomly chosse N directions wi-pdf
      Lo = 0.0
      For each wi
      	Trace a ray r(p, wi)
      	If ray r hit the light		//直接光照
      		Lo += (1/n) * Li * fr * cosine/pdf(wi)
      	Else if ray r hit an object at q		//间接光照
              Lo += (1/N) * shade(q, -wi) * fr * cosine/pdf(wi)
      Return Lo
  ```

  1. 这种算法递归的光线数量会爆炸，这是一个指数级的算法，解决方法是让N=1，1的指数级永远是1，路径追踪算法中，N=1，分布式路径追踪算法中N!=1

     而光线放到了从像素点出发，从一个像素点发出N根光线，然后求平均得出该像素点的着色

  2. 该递归没有停止条件，在实际生活中光确实要弹射无数次，但计算机不能完成。俄罗斯轮盘赌(Russian Roulette，RR)就是解决该问题，俄罗斯轮盘赌是说的在能装n发子弹的枪中装m发子弹，然后开枪射自己，自己死的概率，通过这种思想，设置一个概率使光线有这个概率发出去，然后得到的结果再除以这个概率
     $$
     E=P*(L_o/P)+(1-P)*0=L_o
     $$
     可以看出期望仍是Lo

  解决之后的路径追踪算法：

  ```c++
  shade(p,wo)
      Manually specify a probability P_RR
      Ranndom select ksi in a uniform dist.in [0,1]
      if(ksi > P_RR) return 0.0;
  
      Randomly chosse ONE directions wi-pdf(w)
      Trace a ray r(p, wi)
      If ray r hit the light		//直接光照
      	Return  Li * fr * cosine/pdf(wi)/P_RR
      Else if ray r hit an object at q		//间接光照
          Return shade(q, -wi) *fr * cosine/pdf(wi)/P_RR
  ```

  该算法仍然不是特别高效，因为从像素出发最终到达光源完全看运气，导致很多达不到光源的光线是浪费掉的，所以不在像素点采样，从光源采样可以使光线全都不会浪费，但是渲染方程是基于物体的采样积分，所以要将积分改为在光源上采样的积分

  重写渲染方程：x是物体的点，x'是光源的点，n是物体的法线，n‘是光源的法线，theta是物体法线与光线的夹角，theta'是光源发现与光线的夹角
  $$
  L(x,\omega_o)=\int_AL_i(x,\omega_i)f_r(x,\omega_i,\omega_o)\frac{cos\theta cos\theta'}{||x'-x||^2}dA
  $$
  然后将一个点上的光分解成来自光源和来自其它物体两部分

  改写后的路径追踪算法：

  ```c++
  shade(p,wo)
  	Uniformly sample the light at x' (pdf_light = 1/A)
  	L_dir=Li*fr*costheta*costheta'/|x'-p|^2 / pdf_light
  	
  	L_indir = 0.0
  	Test Russian Roulette with probability P_RR
  	Uniformly sample th hemisphere toward wi (pdf_hemi=1/2pi)
  
      If ray r hit a non-emitting object at q		//间接光照
      	L_indir=shade(q,-wi)*fr*costheta/pdf_hemi/P_RR
     Return L_dir+L_indir
  ```

- 从前说光追就是指Whitted-Style光追，现在是指所有光线传播方法的集合

