---
title: OpenGL变换
copyright: true
date: 2023-09-02 00:23:19
updated: 2023-09-02 00:23:19
tags:
    - OpenGL
    - 课程笔记
categories:
    - 计算机图形学
    - OpenGL
excerpt: OpenGL中使用GLM库进行图像的变换
index_img: https://learnopengl-cn.github.io/img/01/07/transformations.png # 文章在首页的封面图, 默认由 default_index_img 配置
# banner_img: 文章页顶部大图 
# sticky: 100 # 文章在首页排序权重, 越大越靠前
# archive: true # 首页隐藏文章, 归档页显示文章
# hide: true # 不在首页和其他归档分类页里展示, 隐藏后依然可以通过文章链接访问
---

# 数学基础

向量, 矩阵相关的计算, 包括向量点乘, 向量的叉乘, 矩阵乘法, 向量与矩阵的乘法等. 本标题下的内容不明白也无妨, GLM库已经帮我们都实现, 我们只需要简单直观的调用一个函数, 就能得到我们所希望的矩阵.

## 缩放矩阵

$$
\begin{bmatrix} \color{red}{S_1} & \color{red}0 & \color{red}0 & \color{red}0 \\ \color{green}0 & \color{green}{S_2} & \color{green}0 & \color{green}0 \\ \color{blue}0 & \color{blue}0 & \color{blue}{S_3} & \color{blue}0 \\ \color{purple}0 & \color{purple}0 & \color{purple}0 & \color{purple}1 \end{bmatrix} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} \color{red}{S_1} \cdot x \\ \color{green}{S_2} \cdot y \\ \color{blue}{S_3} \cdot z \\ 1 \end{pmatrix}
$$

## 位移矩阵

$$
\begin{bmatrix}  \color{red}1 & \color{red}0 & \color{red}0 & \color{red}{T_x} \\ \color{green}0 & \color{green}1 & \color{green}0 & \color{green}{T_y} \\ \color{blue}0 & \color{blue}0 & \color{blue}1 & \color{blue}{T_z} \\ \color{purple}0 & \color{purple}0 & \color{purple}0 & \color{purple}1 \end{bmatrix} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} x + \color{red}{T_x} \\ y + \color{green}{T_y} \\ z + \color{blue}{T_z} \\ 1 \end{pmatrix}
$$

## 旋转矩阵

### 沿X轴旋转

$$
\begin{bmatrix} \color{red}1 & \color{red}0 & \color{red}0 & \color{red}0 \\ \color{green}0 & \color{green}{\cos \theta} & - \color{green}{\sin \theta} & \color{green}0 \\ \color{blue}0 & \color{blue}{\sin \theta} & \color{blue}{\cos \theta} & \color{blue}0 \\ \color{purple}0 & \color{purple}0 & \color{purple}0 & \color{purple}1 \end{bmatrix} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} x \\ \color{green}{\cos \theta} \cdot y - \color{green}{\sin \theta} \cdot z \\ \color{blue}{\sin \theta} \cdot y + \color{blue}{\cos \theta} \cdot z \\ 1 \end{pmatrix}
$$

### 沿Y轴旋转

$$
\begin{bmatrix} \color{red}{\cos \theta} & \color{red}0 & \color{red}{\sin \theta} & \color{red}0 \\ \color{green}0 & \color{green}1 & \color{green}0 & \color{green}0 \\ - \color{blue}{\sin \theta} & \color{blue}0 & \color{blue}{\cos \theta} & \color{blue}0 \\ \color{purple}0 & \color{purple}0 & \color{purple}0 & \color{purple}1 \end{bmatrix} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} \color{red}{\cos \theta} \cdot x + \color{red}{\sin \theta} \cdot z \\ y \\ - \color{blue}{\sin \theta} \cdot x + \color{blue}{\cos \theta} \cdot z \\ 1 \end{pmatrix}
$$

### 沿Z轴旋转

$$
\begin{bmatrix} \color{red}{\cos \theta} & - \color{red}{\sin \theta} & \color{red}0 & \color{red}0 \\ \color{green}{\sin \theta} & \color{green}{\cos \theta} & \color{green}0 & \color{green}0 \\ \color{blue}0 & \color{blue}0 & \color{blue}1 & \color{blue}0 \\ \color{purple}0 & \color{purple}0 & \color{purple}0 & \color{purple}1 \end{bmatrix} \cdot \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix} = \begin{pmatrix} \color{red}{\cos \theta} \cdot x - \color{red}{\sin \theta} \cdot y  \\ \color{green}{\sin \theta} \cdot x + \color{green}{\cos \theta} \cdot y \\ z \\ 1 \end{pmatrix}
$$

### 沿任意轴

$$
\begin{bmatrix} \cos \theta + \color{red}{R_x}^2(1 - \cos \theta) & \color{red}{R_x}\color{green}{R_y}(1 - \cos \theta) - \color{blue}{R_z} \sin \theta & \color{red}{R_x}\color{blue}{R_z}(1 - \cos \theta) + \color{green}{R_y} \sin \theta & 0 \\ \color{green}{R_y}\color{red}{R_x} (1 - \cos \theta) + \color{blue}{R_z} \sin \theta & \cos \theta + \color{green}{R_y}^2(1 - \cos \theta) & \color{green}{R_y}\color{blue}{R_z}(1 - \cos \theta) - \color{red}{R_x} \sin \theta & 0 \\ \color{blue}{R_z}\color{red}{R_x}(1 - \cos \theta) - \color{green}{R_y} \sin \theta & \color{blue}{R_z}\color{green}{R_y}(1 - \cos \theta) + \color{red}{R_x} \sin \theta & \cos \theta + \color{blue}{R_z}^2(1 - \cos \theta) & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}
$$

# GLM库

>使用GLM库只是为了得到变换矩阵, 调用下面的函数都会在原有的矩阵(我称之为基础矩阵)上右乘一个变换矩阵(由GLM内部隐式生成)得到新的变换矩阵. 因为是右乘, 所以我们使用代码声明组合矩阵的顺序和实际矩阵结合的顺序相反

## 缩放

```c++
glm::mat4 trans(1.0f);
trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5));
```
1. 基础矩阵
2. 每个轴上缩放的倍数

## 位移

```c++
glm::mat4 trans(1.0f);
trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
```
1. 基础矩阵
2. 位移的方向向量

## 旋转

```c++
glm::mat4 trans(1.0f);
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
```
1. 基础矩阵
2. 逆时针旋转的弧度, 使用`glm::radians`可以把角度转为弧度
3. 绕哪个向量旋转

## 组合矩阵示例

先旋转再平移, (方式二是使用矩阵相乘的方式显式组合, 一般不用吧):
```c++
// 方式一
glm::mat4 trans(1.0f);
trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));

// 方式二
glm::mat4 mat_rotate(1.0f);
glm::mat4 trans(1.0f);
trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
mat_rotate = glm::rotate(mat_rotate, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
trans = trans * mat_rotate;
```

# 传递矩阵到着色器

使用`uniform`向着色器传递变换矩阵, 着色器中将向量左乘变换矩阵完成坐标变换

```c
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 transform;

void main()
{
    gl_Position = transform * vec4(aPos, 1.0f);
    TexCoord = vec2(aTexCoord.x, 1.0 - aTexCoord.y); // 如果载入图片的时候反转过y轴, 这里就不用 1-y
}
```

**uniform传递矩阵到着色器**

```c++
unsigned int transformLoc = glGetUniformLocation(my_shader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```
与其他的传递方式一致, 首先查询`uniform`的位置值, 然后使用`glUniformMatrix4fv`传递矩阵, 其参数的意义:
1. 着色器中`uniform`矩阵的位置值
2. 要传递的矩阵数量
3. 是否要对矩阵进行转置(OpenGL中和GLM中的矩阵存储都采用列主序, 所以不用转置)
4. 真正的矩阵数据, 需要GLM中的函数返回存储在其中的数据