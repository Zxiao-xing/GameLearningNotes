```c++
vmath::mat4 vmath::translate(float x, float y, float z);
```

**描述：**

返回平移(x, y, z)的平移矩阵

```c++
vmath::mat4 vmath::scale(float s)
```

**描述：**

返回一个缩放倍数为s的变换矩阵

```c++
vmath::mat4 vmath::rotate(float x, float y, float z)
```

**描述：**

返回一个变换矩阵，绕x轴转x度，y轴转y度，z轴转z度

```c++
vmath::mat4 vmath::frusturm(float left, float right, float bottom, float top, float near, float far);
```

**描述：**

 根据给定的视椎体设置返回一个透视投影矩阵，近平面矩形通过left、right、bottom、top定义，近平面和远平面的距离通过near和far定义

```c++
vmath::mat4 vmath::lookAt(vmath::vec3 eye, vamth::vec3 center, vmath:vec3 up);
```

**描述：**

根据eye朝向enter的视线，根据up定义的上方向，返回一个透视投影矩阵

```c++
vmath::mat4 vmath::ortho(vmath::vec3 eye, vmath::vec3 center, vmath::vec3 up);
```

**描述：**

从观察点eye朝向center，根据up定义的正方向返回一个正交投影变换矩阵。

注：这个函数原型可能是有问题的，应该与frusturm的参数保持一致