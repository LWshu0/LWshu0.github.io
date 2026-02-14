---
title: OpenGL着色器
copyright: true
date: 2023-08-30 16:48:07
updated: 2023-08-30 16:48:07
tags:
    - OpenGL
    - 课程笔记
categories:
    - 计算机图形学
    - OpenGL
excerpt: OpenGL着色器的简要介绍, 包括输入, 输出等基本概念, 也包括了如何使用多个属性
index_img: /img/OpenGL渲染管线/1693321965242.png
banner_img:
---
# GLSL

## 基本格式

OpenGL着色器语言是一种类C语言, 基本格式:

```c
// 版本
#version version_number
// 输入
in type in_variable_name;
in type in_variable_name;
// 输出
out type out_variable_name;
// uniform变量
uniform type uniform_name;

int main{
    //处理输入的一些图形操作
    //...

    //将结果输出
    out_variable_name = weird_stuff_we_processed;
}
```

每一个输入的变量都叫一个顶点属性, 由于硬件, 我们一般只能使用16个4分量的顶点属性, 这在前一节中也有提到(VAO中的顶点属性指针, 标号从0到15)

**查询方法**: `glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);`

## 数据类型

GLSL中包含C等其它语言大部分的默认基础数据类型: int、float、double、uint和bool. GLSL也有两种容器类型, 它们会在这个教程中使用很多, 分别是向量(Vector)和矩阵(Matrix)

**向量(vector)** 

| 类型  | 含义                          |
| ----- | ----------------------------- |
| vecn  | 包含n个float分量的默认向量    |
| bvecn | 包含n个bool分量的向量         |
| ivecn | 包含n个int分量的向量          |
| uvecn | 包含n个unsigned int分量的向量 |
| dvecn | 包含n个double分量的向量       |

*注: 其中n为 2, 3, 4中的一个*

向量中的第一个分量, 第二个分量, 第三个分量, 第四个分量分别使用 x, y, z, w 代表, 访问一个二维向量的 z 和 w 都是非法的.

GLSL也允许你对颜色使用rgba，或是对纹理坐标使用stpq访问相同的分量。

以下几种赋值方式也是允许的:

```c
// 重组
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
// 使用向量构造
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

## 输入与输出

**着色器之间的交流只能通过输入和输出**, 着色器使用 `in`和 `out`关键字来声明输入和输出. 名称相同的输入和输出会在着色器链接阶段进行链接, 从而达到传输数据的目的.

**顶点着色器**接受特殊的输入, 他的输入直接来自于顶点数据, 我们需要使用 `location`来指定顶点属性在顶点数据中的位置, 同时顶点着色器需要为输入提供一个 `layout`标识.

```c
// 顶点着色器的一个输入
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0
```

**片段着色器**需要输出一个 `vec4`的颜色输出, 如果没有在片段着色器中定义输出颜色, OpenGL默认渲染成黑色.

## Uniform

**从CPU向GPU中的着色器传输数据**

uniform是全局的, 它的名称在所有的着色器中是唯一的, 着色器可以在任意阶段访问uniform.

> 如果声明了一个uniform但是没有在GLSL中使用, 编译时会忽略该uniform

首先,着色器中声明一个uniform:

```c
#version 330 core
out vec4 FragColor;

uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量

void main()
{
    FragColor = ourColor;
}
```

然后找到uniform的location, 从而设置它的值:

```c
// 绿色随时间改变
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram); // 更新着色器之前必须激活对应的着色器程序
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

如果 `glGetUniformLocation`返回-1, 代表没有找到该uniform.

更新着色器(`glUniform4f`)之前必须先激活着色器程序, 因为 `glUniform4f`是设置当前激活的着色器程序.

| glUniform 后缀 | 含义                                 |
| -------------- | ------------------------------------ |
| f              | 函数需要一个float作为它的值          |
| i              | 函数需要一个int作为它的值            |
| ui             | 函数需要一个unsigned int作为它的值   |
| 3f             | 函数需要3个float作为它的值           |
| fv             | 函数需要一个float向量/数组作为它的值 |

# 更多属性

![1693321965242](/img/OpenGL渲染管线/1693321965242.png)

使用 `glVertexAttribPointer`配置位置值为1的属性, 并使用 `glEnableVertexAttribArray`激活该属性. 添加更多属性也使用类似的方法

```c++
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
glEnableVertexAttribArray(1);
```

使用这些属性的时候(一般是在顶点着色器中), 使用 `location`关键字指明属性所在的位置值

# 着色器类

# 从文件读取
