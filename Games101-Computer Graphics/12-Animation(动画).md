- 动画：在图形学中是对于建模和几何的扩展，在不同的帧有不同的图，电影要求24fps，fps要求30fps，VR要求90fps
- 关键帧动画：在关键的地方画出图像，中间的地方通过其他方式生成(插值、人力)，例如漫画动画化
- 质点弹簧系统：一系列相互连接的质点和弹簧
- 粒子系统：
- 骨骼系统：
- Rigging：对形状的控制，类似于木偶
- 动作捕捉(Motion Capture)：通过在实际物体上感知对应的控制点来捕捉动画
- 前项/显示欧拉方法：用上一帧的速度去估计下一帧的速度，就相当于用匀速的方程去算。该方法会有误差且会迅速变的不稳定
- 中点法：取欧拉方法得到的值和上一个方法得到的值的中点然后用中点再用次欧拉方法得到结果点
- 后项/隐式欧拉方法：是稳定的
- Runge-Kutta方法：

数值分析课上讲这些

- 非基于物理的方法：不会保证很多基于物理的性质，但是计算非常快。例如Position-Based方法，