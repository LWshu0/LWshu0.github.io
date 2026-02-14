---
title: OpenGL纹理
copyright: true
date: 2023-08-31 15:39:57
updated: 2023-08-31 15:39:57
tags:
    - OpenGL
    - 课程笔记
categories:
    - 计算机图形学
    - OpenGL
excerpt: OpenGL中有关纹理的创建与使用
index_img: https://learnopengl-cn.github.io/img/01/06/texture_wrapping.png
banner_img:
---
# 纹理

纹理是一个2D的图片, 通过指定**纹理坐标**(也就是在纹理上的点, 左下角是(0, 0), 右上角是(1, 1))来确定显示在图形上的纹理范围.

# 纹理环绕方式

| 环绕方式           | 描述                                                                                   |
| ------------------ | -------------------------------------------------------------------------------------- |
| GL_REPEAT          | 对纹理的默认行为。重复纹理图像。                                                       |
| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。                                          |
| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
| GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色。                                                       |

![纹理图像的例子](https://learnopengl-cn.github.io/img/01/06/texture_wrapping.png)

## 指定环绕方式

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
```

- 第一个参数是纹理的目标, 我们使用2D纹理就指定 `GL_TEXTURE_2D`
- 第二个参数是设置的选项和应用的纹理轴(有s, t, r 轴), 这里配置 `WRAP`选项, 应用到 `S`轴
- 第三个参数是环绕方式

## 配置边缘颜色

```c++
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

- 第一个参数指定纹理类型
- 第二个参数是配置选项
- 第三个参数是一个颜色

# 纹理过滤

> 纹理像素: 是纹理中的一个像素点, 只能是整型坐标
> 纹理坐标: 是浮点型的(较)准确的位置坐标, 限制在[0, 1], 超出部分将按照设置的环绕方式填充

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

- 第二个参数中 `MIN`是进行缩小操作时的过滤方法, `MAG`是放大操作时的过滤方法
- `GL_NEAREST`会根据当前纹理坐标去找一个最近的纹理像素并使用这个纹理像素的颜色, 该方法只会使用纹理上已经存在的颜色
- `GL_LINEAR`会根据坐标附近的纹理像素计算一个线性插值, 该插值可能是原纹理不存在的颜色

相比较两种过滤方法, 邻近过滤产生的图像会有颗粒状, 我们能够看出单个纹理像素, 而线性过滤则会模糊平滑, 看不出单个纹理像素
![纹理过滤效果](https://learnopengl-cn.github.io/img/01/06/texture_filtering.png)

# 多级渐远纹理

多级渐远纹理解决高分辨率纹理应用在小物体上产生的不真实感觉以及高分辨率纹理内存浪费的问题, 如下图是多级渐远纹理, OpenGL可以使用 `glGenerateMipmaps`函数创建一系列多级渐远纹理.

![多级渐远纹理](https://learnopengl-cn.github.io/img/01/06/mipmaps.png)

> 注意: 多级渐远纹理只用于缩小过滤, 用于放大会报错

| 过滤方式                  | 描述                                                                     |
| ------------------------- | ------------------------------------------------------------------------ |
| GL_NEAREST_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样       |
| GL_LINEAR_MIPMAP_NEAREST  | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样                     |
| GL_NEAREST_MIPMAP_LINEAR  | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
| GL_LINEAR_MIPMAP_LINEAR   | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样         |

例: `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);`

# 加载与创建纹理

使用 `stb_image.h`库加载图像, 它是一个单头文件图像加载库, 能够加载大部分流行的文件格式.

## 加载图片

```c++
int width, height, nrChannels;
stbi_set_flip_vertically_on_load(true); // 图片的y轴在最上方, 不加这一句图片会上下颠倒
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
```

函数参数为: 图像文件路径, 同时函数会根据图像填充图像的宽度, 高度, 颜色通道数

返回图像的数据, 这个数据将会被用来生成纹理

## 生成纹理

**创建纹理**

```c++
unsigned int texture;
glGenTextures(1, &texture);
```

**绑定纹理**

将创建的纹理绑定到 `GL_TEXTURE_2D`(因为我们将要使用2D纹理), 后续的操作都将使用 `GL_TEXTURE_2D`来代指我们刚刚创建的纹理(只要我们不再去绑定其他纹理到 `GL_TEXTURE_2D`上或者绑定0也就是解绑)

```c++
glBindTexture(GL_TEXTURE_2D, texture);
```

**生成纹理并使用多级渐进纹理**

这里就是使用我们刚刚绑定了纹理的纹理目标 `GL_TEXTURE_2D`, 为它添加我们先前载入图片时得到的数据 `data`. 生成完成纹理后, 我们又使用 `glGenerateMipmap`为纹理创建了多级渐远纹理.

```c++
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```

`glTexImage2D`函数参数:

1. 指定纹理目标, 这里指定2D纹理, 绑定到1D和3D的纹理将不受影响.
2. 多级渐远纹理的级别, 0就是基本级别
3. 将纹理存储为 `GL_RGB`格式, 因为这里的图像只有RGB值
4. 纹理的宽度(由载入图像函数得到)
5. 纹理的高度(由载入图像函数得到)
6. 历史遗留问题, 总为0即可
7. 源图像的格式, 这里只有RGB值
8. 源图像的数据类型, 我们载入的时候存储为了 `char(byte)`数组
9. 真正的图像数据, 由图像载入函数返回

**释放图像内存**

我们创建好纹理后, 先前载入的图像数据就没有用了, 所以我们可以释放载入的图像数据 `data`:

```c++
stbi_image_free(data);
```

## 应用纹理

### 添加纹理坐标属性

应用纹理需要我们为顶点提供新的属性---纹理坐标, 那么新的顶点格式如下:

![添加纹理坐标的顶点属性](https://learnopengl-cn.github.io/img/01/06/vertex_attribute_pointer_interleaved_textures.png)

使用 `glVertexAttribPointer`和 `glEnableVertexAttribArray`添加这个属性, 详细参数不再赘述

### 顶点着色器

此时顶点着色器需要接收三个输入: 顶点坐标(`vec3`), 顶点颜色(`vec3`), 纹理坐标(`vec2`). 同时有两个输出到片段着色器: 顶点颜色和纹理坐标, 直接赋值即可.

```c
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

### 片段着色器

片段着色器要接收从顶点着色器传来的两个参数(顶点颜色和纹理坐标), 如果只使用纹理渲染那么顶点颜色是用不到的. 同时还要有一个 `uniform`的采样器(sampler)输入, 采样器以纹理类型作为后缀, 如:`sampler1D`、`sampler2D`和 `sampler3D`, 我们会把纹理传给采样器(当我们只使用一个纹理时, 不必用 `glUniform`给它赋值, 这一点在纹理单元中会详细说明). GLSL中的 `texture`方法会根据我们的纹理采样器和纹理坐标去采集纹理的颜色.

```c
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}
```

### 绑定纹理

在调用绘制函数之前绑定纹理即可渲染出使用纹理的图像:

```c++
glBindTexture(GL_TEXTURE_2D, texture); // 绑定纹理
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

## 纹理单元

一个纹理的位置值通常称为一个**纹理单元(Texture Unit)**, 也就是说我们可以使用多个纹理(一般是16个, 纹理位置从 `GL_TEXTURE0`到 `GL_TEXTURE15`, 并且它们连续可加的, 如: `GL_TEXTURE0`+3 == `GL_TEXTURE3`). `GL_TEXTURE0`是默认激活的纹理单元, 所以在上面我们没有手动激活纹理单元就绑定了纹理.

当我们要使用多个纹理时, 应该先激活我们要绑定纹理的纹理单元, 然后在绑定纹理:

```c++
glActiveTexture(GL_TEXTURE0); // 在绑定纹理之前先激活纹理单元
glBindTexture(GL_TEXTURE_2D, texture); // texture纹理将会绑定到GL_TEXTURE0

glActiveTexture(GL_TEXTURE1); // 激活纹理单元GL_TEXTURE1
glBindTexture(GL_TEXTURE_2D, texture2); //将texture2纹理绑定到GL_TEXTURE1
```

当有多个纹理单元时, 我们就必须指定片段着色器中采样器的位置值:

```c
#version 330 core
// ...
uniform sampler2D texture1;
uniform sampler2D texture2;
// ...
```

```c++
ourShader.use(); // 不要忘记在设置uniform变量之前激活着色器程序！
glUniform1i(glGetUniformLocation(ourShader.ID, "texture1"), 0); // 手动设置采样器1
ourShader.setInt("texture2", 1); // 或者使用着色器类设置采样器2
```
