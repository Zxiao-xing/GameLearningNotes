**二维变换：**

- 缩放(s是缩放比例)
  $$
  \begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}s_x & 0\\0 & s_y\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$

- 翻转
  $$
  沿y轴翻转：\begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}-1 & 0\\0 & 1\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$

  $$
  沿x轴翻转：\begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}1 & 0\\0 & -1\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$

- 切变（拖拽一条边）
  $$
  拖拽正方形上面一条边平移a：\begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}1 & a\\0 & 1\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$

- 旋转θ（当没明确指明方向默认逆时针旋转）
  $$
  \begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}cos\theta & -sin\theta\\sin\theta & cos\theta\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$
  ![image-20201113103810282](C:\Users\A\AppData\Roaming\Typora\typora-user-images\image-20201113103810282.png)

  旋转-θ
  $$
  \begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}cos\theta & sin\theta\\-sin\theta & cos\theta\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$
  可以看出对于旋转矩阵R，旋转矩阵是正交阵
  $$
  R_{-\theta}=R_\theta^T=R_\theta^{-1}
  $$
  
- 对于所有线性变换（上述都是线性变换）：
  $$
  \begin{bmatrix}x^{'}\\y^{'}\end{bmatrix}
  =
  \begin{bmatrix}a & b \\ c & d\end{bmatrix}
  \begin{bmatrix}x\\y\end{bmatrix}
  $$

  $$
  x^{'}=ax+by
  $$

  $$
  y^{'}=cx+dy
  $$

- 为了使平移变换也能用矩阵表示，引入了齐次坐标
  $$
  2维的点：(x,y,1)^T
  $$

  $$
  2维向量(向量具有平移不变性)：(x,y,0)^T
  $$

  $$
  平移：\begin{bmatrix}x^{'}\\y^{'}\\w^{'}\end{bmatrix}
  =
  \begin{bmatrix}1 & 0 & t_x\\ 0 & 1 &t_y \\ 0 & 0 & 1\end{bmatrix}
  \begin{bmatrix}x\\y\\1\end{bmatrix}
  =\begin{bmatrix}x+t_x\\y+t_y\\1\end{bmatrix}
  $$

  对于向量和点的齐次坐标表示仍满足运算：

  向量+向量=向量

  点-点=向量

  点+向量=点

  点+点=???

  扩充定义：对于一个(x，y，w)代表的是(x/w，y/w，1)的点

- 使用齐次坐标进行的变换称为仿射变换(实际是先应用旋转再用平移)：
  $$
  \begin{bmatrix}x^{'}\\y^{'}\\1\end{bmatrix}
  =
  \begin{bmatrix}a & b & t_x\\ c & d &t_y \\ 0 & 0 & 1\end{bmatrix}
  \begin{bmatrix}x\\y\\1\end{bmatrix}
  $$

- 逆变换（将进行的一次变换还原）：乘以变换矩阵的逆矩阵

- 对于列向量，变换顺序在乘积中由从右到左看的，对于行向量则从左到右

**三维变换：**

- 齐次坐标表示
  $$
  点：(x,y,z,1)^T
  $$

  $$
  向量：(x,y,z,0)^T
  $$

  扩充：点(x，y，z，w)代表点(x/w，y/w，z/w，1)

- 仿射变换
  $$
  \begin{bmatrix}x^{'}\\y^{'}\\z^{'}\\1\end{bmatrix}
  =
  \begin{bmatrix}a & b & c & t_x\\ d & e & f &t_y\\ g & h & i & t_z \\ 0 & 0 & 0 & 1\end{bmatrix}
  \begin{bmatrix}x\\y\\z\\1\end{bmatrix}
  $$
  
- 旋转
  $$
  R_x(\alpha)
  =
  \begin{bmatrix}1 & 0 & 0 & 0\\ 0 & cos\alpha & -sin\alpha & 0\\ 0 & sin\alpha & cos\alpha & 0 \\ 0 & 0 & 0 & 1\end{bmatrix}
  $$

  $$
  R_y(\alpha)
  =
  \begin{bmatrix} cos\alpha & 0 & sin\alpha & 0\\ 0 & 1 & 0 & 0\\ -sin\alpha & 0 & cos\alpha & 0 \\ 0 & 0 & 0 & 1\end{bmatrix}
  $$

  $$
  R_z(\alpha)
  =
  \begin{bmatrix} cos\alpha & -sin\alpha & 0 & 0\\ sin\alpha & cos\alpha & 0 & 0\\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1\end{bmatrix}
  $$

  绕Y轴旋转和其他两个不一样是因为对于XYZ三个轴，绕X轴时是Y * Z，绕Z轴是X * Y，而绕Y轴旋转是Z*X

- 欧拉角(alpha、beta、gamma)
  $$
  R_{xyz}(\alpha\beta\gamma)=R_x(\alpha)R_y(\beta)R_z(\gamma)
  $$
  俯仰角(yaw)、翻滚角(roll)、偏航角(pitch)

  罗德里德斯旋转公式(Rodrigus' Rotation Formula)以n轴(过原点)旋转alpha角度
  $$
  R(n,\alpha)=cos(\alpha)I+(1-cos(\alpha))nn^T+sin(\alpha)
  \begin{bmatrix}0 & -n_z & n_y \\ n_z & 0 & -n_x \\ -n_y & n_x & 0\end{bmatrix}
  $$

- 四元数（为了求旋转的插值值而设计的）

1. 找一个位置放置模型，模型变换(model transformation)
2. 找一个位置放置相机，视图变换(view transformation)
3. 在相机中对模型进行投影，投影变换(projection transformation)
4. 三个变换简称MVP变换

**视图变换(View/Camera Transformation)**

- 只要保证相机和模型是相对静止的，就可以投影出相同的图，所以一般约定将相机放在原点，且相机向-z方向看，向上是y轴

- 视图变换的本质就是将相机变换到约定位置。先进行平移，再进行旋转(将g旋转到-z，t旋转到y，此时g*t旋转到x)

  旋转的逆变换可以简单求出
  $$
  R_{view}^{-1}=
  \begin{bmatrix}x_{g*t}&x_t&x_{-g}&0 \\ y_{g*t}&y_t&y_{-g}&0 \\ 
  z_{g*t}&z_t&z_{-g}&0 \\ 0&0&0&1\end{bmatrix}
  $$

  $$
  M_{view}=R_{view}T_{view}=
  \begin{bmatrix}x_{g*t}&y_{g*t}&z_{g*t}&0 \\ x_t&y_t&z_{t}&0 \\ 
  x_{-g}&y_{-g}&z_{-g}&0 \\ 0&0&0&1\end{bmatrix}
  \begin{bmatrix}1&0&0&-x_e \\ 0&1&0&-y_e \\ 0&0&1&-z_e \\ 0&0&0&1\end{bmatrix}
  $$

**模型变换(Model Transformation)**

- 模型变换和视图变换经常连起来，称为模型视图变换
- 模型变换就是按着视图变换的两个矩阵乘一次保证相机和模型相对静止

**投影变换(Projection Transformation)**

- 透视投影，有近大远小的特性，使平行线不在平行，广泛使用该视图

  将远处的一个大平面按比例挤压到近处的一个小平面，规定远处平面的中心点和近处屏幕的中心点是相同的，且对于每一个点，根据距离的挤压过程中，z值没有发生变换，即不管在近的平面还是远的平面，z值都不会发生变换

  对于一个点仅需将(x,y,z)按z轴(距离)缩放为x' = nx/z y' = ny/z，在此情况下乘以一个z值点还是没变
  $$
  \begin{bmatrix}x\\y\\z\\1\end{bmatrix}=>
  \begin{bmatrix}nx/z\\ny/z\\n\\1\end{bmatrix}
  =
  \begin{bmatrix}nx\\ny\\n^2\\z\end{bmatrix}
  $$
  变换矩阵为(n代表近距离平面离相机的长度，f代表远平面离相机的长度)
  $$
  \begin{bmatrix}n&0&0&0\\0&n&0&0\\0&0&n+f&-nf\\0&0&1&0\end{bmatrix}
  $$

- 正交投影，假设相机无限远，使得远处和近处相同大小的物体大小差距不大，不会造成近大远小的现象，最多使用是用来工程制图

  简单做法：首先将相机摆放至一个位置，例如摆放至原点朝着-z方向看，这时只要将z轴给扔掉就可以得到一个物体的正交投影

  正式做法：用一个立方体（x轴分别对应r，l；y轴分别对应t，b；z轴分别对应n，f）将模型包裹起来，将立方体的中心点移动到原点，再将x，y，z的长度缩小到-1到1，之后在进行视口变换将模型的长宽比复原
  $$
  M=
  \begin{bmatrix} 2/(r-l)&0&0&0 \\ 0&2/(t-b)&0&0\\ 
  0&0&1/(f-n)&0 \\ 0&0&0&1 \end{bmatrix}
  \begin{bmatrix} 1&0&0&-(r+l)/2 \\ 0&1&0&-(t+b)/2\\ 
  0&0&1&-(n+f)/2 \\ 0&0&0&1 \end{bmatrix}
  $$

- 垂直可视角： 两根最短的线，线的夹角就是垂直可视角。

  水平可视角同理。

- 只要知道垂直可视角度距离平面的宽度即可求出平面的高

