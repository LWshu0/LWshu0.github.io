---
title: OpenGL渲染管线
copyright: true
date: 2023-08-29 21:58:41
updated: 2023-08-29 21:58:41
tags:
    - OpenGL
    - 课程笔记
categories:
    - 计算机图形学
    - OpenGL
excerpt: 记录OpenGL中关于渲染管线的内容, 主要包括 VBO, VAO, EBO/IBO
index_img: img/OpenGL渲染管线/1693317800616.png
banner_img:
---
# 图形渲染管线

图形渲染管线主要包含如下几个阶段, 通过渲染管线可以把三维顶点数据渲染到2D屏幕上.

![1693317800616](/img/OpenGL渲染管线/1693317800616.png)

我们可以自定义的着色器是图中的三个蓝色部分, 我们 **必须** 自定义至少一个顶点着色器和一个片段着色器, 而几何着色器在GPU中有默认的. **顶点着色器** 把一个单独的顶点作为输入, 同时可以对顶点的属性进行一些基本的处理. **片段着色器** 负责计算一个像素最终的颜色, 这是所有OpenGL高级效果产生的地方(光照, 阴影等).

# 渲染一个简单三角形的流程

这是我们最终希望的效果:

![1693319036346](/img/OpenGL渲染管线/1693319036346.png)

## 准备顶点数据

### 顶点定义

**OpenGL**中使用的都是三维的顶点(x, y, z), 它们的范围为 [-1, 1] (超出范围的点不显示). 也就是说把屏幕标准化了, 所以同一个三角形在不同分辨率的屏幕上显示大小会不同.

**标准化设备坐标** (同我们以前习惯的坐标系一样)

![1693318755780](/img/OpenGL渲染管线/1693318755780.png)

定义三角形的三个顶点:

```c++
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```

### VBO(顶点缓冲对象)

> OpenGL中的各种对象都有一个独一无二的ID, 访问对象时也是用这个ID(`unsigned int`类型)

VBO管理在GPU上存放的顶点数据, 使用 `glGenBuffers(1, &VBO);` 生成1个缓冲对象, 并把其ID存放在VBO中

> OpenGL有很多缓冲对象类型, 我们可以分别为它们绑定缓冲对象

VBO 的缓冲对象类型为 `GL_ARRAY_BUFFER`, 我们就用 `glBindBuffer(GL_ARRAY_BUFFER, VBO);`把刚才创建的顶点缓冲对象绑定到 `GL_ARRAY_BUFFER`, 同时使用 `glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);`把之前定义的顶点数据复制到GPU的缓冲中.

```
glBufferData(目标缓冲对象类型, 传输的数据字节, 要发送的数据指针, 显卡管理数据的方式);
```

- GL_STATIC_DRAW ：数据不会或几乎不会改变。
- GL_DYNAMIC_DRAW：数据会被改变很多。
- GL_STREAM_DRAW ：数据每次绘制时都会改变。

## 顶点着色器(Vertex Shader)

> 使用着色器语言GLSL(OpenGL Shading Language)编写着色器

每个着色器都起始于一个版本声明。OpenGL 3.3以及和更高版本中，GLSL版本号和OpenGL的版本是匹配的（比如说GLSL 420版本对应于OpenGL 4.2）. 我们同样明确表示我们会使用核心模式.

**此部分后续文章详细记录.**

### 定义着色器

```c
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

### 创建着色器

我们把要创建的着色器类型传入函数中, 返回着色器的ID. 这里我们要创建一个顶点着色器, 所以传入 `GL_VERTEX_SHADER`

```c++
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```

### 编译着色器

定义好着色器并且创建好着色器后就要编译着色器以待GPU执行.

```c++

const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "void main()\n"
    "{\n"
    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
    "}\0";
//******************************************
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

其中使用 `glShaderSource`把着色器的源码附着到着色器对象(用ID指代).

```c++
glShaderSource(着色器ID, 源码字符串数量, 源码数据, NULL)
```

### 检查是否编译成功

```c++
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

## 片段着色器(Fragment Shader)

### 定义着色器

输出颜色, 这里简单的只输出一种颜色

```c
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```

### 创建着色器

与顶点着色器类似, 此处着色器类型为 `GL_FRAGMENT_SHADER`

```c++
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
```

### 编译/检查着色器

与顶点着色器相同

## 着色器程序对象(Shader Program Object)

将上述定义的一系列着色器链接为一个**着色器程序**, 在渲染对象时要激活这个着色器程序, 激活后就可以发送渲染调用渲染图形.

### 创建程序对象

```c++
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
```

### 附加并链接着色器

将我们定义的着色器附加到着色器对象上:

```c++
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
```

将着色器链接为着色器程序:

```c++
glLinkProgram(shaderProgram);
```

> 链接为着色器对象后之前定义的着色器对象就没有用了, 可以使用 `glDeleteShader`删除着色器对象

### 检查链接

```c++
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if(!success) {
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    ...
}
```

### 激活程序对象

```c++
glUseProgram(shaderProgram);
```

## 顶点属性

一个顶点可以有很多属性(至少有16个4分量的顶点属性), 向下面的两个VBO例子, 其中都定义了三个点, 但是他们的属性数量不同.

位置属性:

![1693321975378](/img/OpenGL渲染管线/1693321975378.png)

位置属性+颜色属性:

![1693321965242](/img/OpenGL渲染管线/1693321965242.png)

记得顶点着色器中的 `location = 0`吗? 它指明了位置属性在VBO中的位置值(换句话说就是第几个属性, 标号从0开始).

### 设置顶点属性指针

当我们把顶点数据复制到VBO中后, OpenGL并不知道该怎么去解释这些数据(当然我们自己是知道的), 哪些是位置, 哪些是颜色需要我们告知OpenGL.

```c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

```c++
glVertexAttribPointer(属性的位置值, 属性的大小, 数据类型, 是否标准化([0, 1]for unsigned 或者[-1, 1] for signed), 顶点属性之间的步长(相邻顶点相同属性之间的间隔), 数据在缓冲中起始偏移量);
glEnableVertexAttribArray(属性的位置值); //启用顶点属性 默认是禁用的
```

> 每个顶点属性从一个VBO管理的内存中获得它的数据，而具体是从哪个VBO（程序中可以有多个VBO）获取则是通过在调用glVertexAttribPointer时绑定到GL_ARRAY_BUFFER的VBO决定的。

# 顶点数组对象(VAO)

绑定一个VAO后, 任何顶点属性调用都会存储在当前绑定的VAO中:

- glEnableVertexAttribArray和glDisableVertexAttribArray的调用。
- 通过glVertexAttribPointer设置的顶点属性配置。
- 通过glVertexAttribPointer调用与顶点属性关联的顶点缓冲对象。

如图, VAO中一共有16个顶点属性指针, 但VAO1只使用了第一个(也就是0号), 而VAO2使用了两个, 因为VBO2有两个属性

![1693323124547](/img/OpenGL渲染管线/1693323124547.png)

当下一次想要渲染绘制物体时, 只要绑定对应的VAO即可, 不需要再进行设置顶点属性指针等操作.

## 创建VAO

与创建VBO类似:

```c++
unsigned int VAO;
glGenVertexArrays(1, &VAO);
```

## 绑定VAO

```c++
glBindVertexArray(VAO);
```

## 绘图流程

```c++
// ..:: 初始化代码（只运行一次 (除非你的物体频繁改变)） :: ..
// 1. 绑定VAO
glBindVertexArray(VAO);
// 2. 把顶点数组复制到缓冲中供OpenGL使用
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. 设置顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

[...]

// ..:: 绘制代码（渲染循环中） :: ..
// 4. 绘制物体
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
someOpenGLFunctionThatDrawsOurTriangle();
```

# 绘制三角形

```c++
glDrawArrays(GL_TRIANGLES, 0, 3);
```

参数列表:

1. `GL_TRIANGLES`是绘制的图元类型, 这里要绘制三角形
2. 顶点数组的起始索引
3. 绘制多少个顶点(三角形就填3)

# 元素缓冲对象(EBO/IBO)

为了减少额外的内存开销, 对于同一个顶点我们在VBO中只存储一次, 而EBO中存储的是顶点在VBO中的索引.

## 创建EBO

也就是创建一个缓冲区

```c++
unsigned int EBO;
glGenBuffers(1, &EBO);
```

## 绑定EBO并传输数据

```c++
unsigned int indices[] = {
    // 注意索引从0开始! 
    // 此例的索引(0,1,2,3)就是顶点数组vertices的下标，
    // 这样可以由下标代表顶点组合成矩形

    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
};
//***********************************************
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

## 绘制

```c++
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

参数列表:

1. 图元类型
2. 要绘制的顶点个数(这里绘制两个三角形所以是6)
3. 索引的类型
4. 指定EBO中的偏移量（或者传递一个索引数组，但是这是当你不在使用索引缓冲对象的时候）

## 绑定到VAO

> 在绑定VAO时，绑定的最后一个元素缓冲区对象存储为VAO的元素缓冲区对象。然后，绑定到VAO也会自动绑定该EBO。

![1693324412564](/img/OpenGL渲染管线/1693324412564.png)

> 当目标是GL_ELEMENT_ARRAY_BUFFER的时候，VAO会储存glBindBuffer的函数调用。这也意味着它也会储存解绑调用，所以确保你没有在解绑VAO之前解绑索引数组缓冲，否则它就没有这个EBO配置了。
