**向量**

- 带箭头的字母或者粗体字母表示

- 或用从一个点起始到一个终止的方式表示：
  $$
  \vec{AB}=B-A
  $$
  
- 拥有方向和长度

- 没有绝对的开始和终止位置，因为它可以随意移动

- 单位向量运算，常用于表示方向
  $$
  \hat{a}=\vec{a}\ /\ ||\vec{a}||
  $$

- 数学表示法
  $$
  A=\begin{bmatrix}x \\ y\end{bmatrix}
  $$

  $$
  A^T=\begin{bmatrix}x&y\end{bmatrix}
  $$

  $$
  ||A||=\sqrt{x^2+y^2}
  $$

  图形学中，默认使用列向量表示

- 点乘

  
  $$
  \vec{a}\cdot{}\vec{b}=||\vec{a}||\ ||\vec{b}||\ cos\theta
  $$

  $$
  cos\theta=\vec{a}\cdot{}\vec{b}\ /\ ||\vec{a}||\ ||\vec{b}||
  $$

  $$
  cos\theta=\hat{a}\cdot\hat{b}
  $$

  $$
  \vec{a}\cdot{}\vec{b}=
  \begin{bmatrix}x_a \\ y_a\end{bmatrix}
  \cdot{}
  \begin{bmatrix}x_b \\ y_b\end{bmatrix}
  = x_ax_b+y_ay_b
  $$

  运算法则
  $$
  交换律：\vec{a}\cdot{}\vec{b}=\vec{b}\cdot{}\vec{a}
  $$

  $$
  分配律：\vec{a}\cdot{}(\vec{b}+\vec{c})=\vec{a}\cdot{}\vec{b}+\vec{a}\cdot{}\vec{c}
  $$

  $$
  结合律：(k\vec{a})\cdot\vec{b}=\vec{a}\cdot(k\vec{b})=k(\vec{a}\cdot{}\vec{b})
  $$

  b向量在a向量上的投影
  $$
  ||\vec{b}||\ cos\theta\vec{a} = \frac{\vec{a}\cdot{}\vec{b}}{||\vec{a}||}\vec{a}
  $$
  水平垂直分解向量：分解为投影向量和b向量减去投影向量

  判断两向量是同向还是反向：点乘结果>0同向，点乘结果<0反向

  判断向量的接近程度：越接近点乘结果越趋近于1，越远离越趋近于-1。这两个判断都是基于cos的运算

- 叉乘
  $$
  \vec{a}*\vec{b}=-\vec{b}*\vec{a}
  $$

  $$
  ||\vec{a}*\vec{b}||=||\vec{a}||\ ||\vec{b}||sin\theta
  $$

  $$
  \vec{a}*\vec{a}=0
  $$

  $$
  \vec{a}*(\vec{b}+\vec{c})=\vec{a}*\vec{b}+\vec{a}*\vec{c}
  $$

  $$
  \vec{a}*(k\vec{b})=k(\vec{a}*\vec{b})
  $$

  $$
  \vec{a}*\vec{a}=
  \begin{bmatrix}y_az_b-y_bz_a \\ z_ax_b-x_az_b \\ x_ay_b-y_ax_b\end{bmatrix}
  $$

  判定左右和内外：若向量a叉乘向量b大于0则b在a左侧，若小于0则在右侧。若三角形ABC和一点P，分别用AB，BC，CA叉乘AP，BP，CP，若都大于0或都小于0（顺时针和逆时针情况不同），则处于三角形内，若有一者不满足则处于三角形外

**矩阵**

- 矩阵相乘要前者的列数等于后者的行数才能相乘
  $$
  结合律：(AB)C=A(BC)
  $$

  $$
  分配律：A(B+C)=AB+AC
  $$

  $$
  (A+B)C=AC+BC
  $$

  $$
  (AB)^T=B^TA^T
  $$

  $$
  AA^{-1}=E
  $$

- 对角阵，除了对角线上全是0

- 正交阵，矩阵的转置等于矩阵的逆

- 单位阵，对角阵，且对角线上全为1