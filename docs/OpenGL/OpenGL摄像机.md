---
title: OpenGL摄像机
copyright: true
date: 2023-09-02 20:06:06
updated: 2023-09-02 20:06:06
tags:
    - OpenGL
    - 课程笔记
categories:
    - 计算机图形学
    - OpenGL
excerpt: OpenGL摄像机/观察空间记录, 编写一个摄像机类
index_img: https://learnopengl-cn.github.io/img/01/09/camera_axes.png
# banner_img:
---
# 摄像机/观察空间

在坐标系统一节中, 我们定义的摄像机仅仅是平移了一段距离, 实际上摄像机可以在世界坐标的任意位置, 并且朝向任意角度. 我们以摄像机为原点, 将所有物体在世界坐标系下的坐标, 转换到以摄像机为原点的坐标系下, 也就得到了我们在屏幕中看到的物体的样子, 下面的图4就是我们在摄像机上建立的坐标系. (红色x轴, 绿色y轴, 蓝色z轴)

![摄像机](https://learnopengl-cn.github.io/img/01/09/camera_axes.png)

摄像机有四个重要的属性:

1. **位置**: 就是摄像机在世界坐标系下的位置坐标, 上一节中我们的摄像机在(0, 0, 3)处.
2. **摄像机方向**: 摄像机朝向的反方向. 比如我们的摄像机位于 `cameraPos`处, 它朝向 `cameraTarget`处, 那么摄像机方向就是 `cameraPos - cameraTarget`
3. **右轴**: 摄像机坐标系的X轴. 要得到这个向量, 需要借助叉乘, 很容易知道这个右轴是同时正交于摄像机方向和世界坐标系的Y轴(称之为*上向量*, up), 所以右轴就是 `glm::normalize(glm::cross(up, cameraDirection));`
4. **上轴**: 摄像机坐标系的Y轴. 因为坐标系三轴正交, 所以由摄像机方向和右轴叉乘即可得到上轴.

至此, 我们已经建立了摄像机坐标系, 接下来只需要把世界中所有物体的坐标都转换到摄像机坐标系下.

# Look At 矩阵

$$
LookAt = \begin{bmatrix} \color{red}{R_x} & \color{red}{R_y} & \color{red}{R_z} & 0 \\ \color{green}{U_x} & \color{green}{U_y} & \color{green}{U_z} & 0 \\ \color{blue}{D_x} & \color{blue}{D_y} & \color{blue}{D_z} & 0 \\ 0 & 0 & 0  & 1 \end{bmatrix} * \begin{bmatrix} 1 & 0 & 0 & -\color{purple}{P_x} \\ 0 & 1 & 0 & -\color{purple}{P_y} \\ 0 & 0 & 1 & -\color{purple}{P_z} \\ 0 & 0 & 0  & 1 \end{bmatrix}
$$

其中$\color{red}{R}$是右向量, $\color{green}{U}$是上向量, $\color{blue}{D}$是方向向量, $\color{purple}{P}$是摄像机位置向量. **LookAt矩阵可以乘以任何向量来将向量变换到我们定义的摄像机坐标空间**.

GLM库提供了一个函数 `glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f));`来生成一个LookAt矩阵. 参数意义:

1. 摄像机位置
2. 摄像机朝向的目标
3. 上向量
