---
title: Markdown 语法
copyright: true
date: 2024-08-15 00:27:01
updated: 2024-08-15 00:27:01
tags:
    - Markdown
categories:
    - 杂项
excerpt: Markdown 语法简介以及常用的数学符号写法
---

>本文档中的内容都已在vscode的插件 Office Viewer(Markdown Editor) 中实现成功。

## 基本格式

### 标题
`#`、`##`、`###`、`####` 等(最多6个`#` 号)，分别表示一级标题、二级标题、三级标题、四级标题等，字号大小依次降低，而且显示在大纲中。

### 代码
行内代码用 \`这是行内代码\` 括起来，例如: `print("hello");`
代码段使用 \```只是一个代码段\``` 括起来，例如:
```
#include <iostream>
int main(){
    std::cout << "hello" << std::endl;
}
```
代码中可以出现一些markdown语法，但不按照 markdown 语法渲染，而是当作普通文本渲染，例如下面是行内代码中的一个二级标题和一个行内公式:
`## head2`，`$i=0$`

### 表格
表格由表格头、表格分隔线、表格内容三部分组成。分割线其中的 `-` 可以是任意个，每一行最后面的 `|` 可以省略(为了)，**分割线的表格数必须和表头的表格数一致**。

**表格内换行**使用html语法 `<br />` 

```
| 姓名 | 年龄 | 工作         |
| ---- | ---- | ------------ |
| 张三 | 34   | 遛<br />狗   |
| 李四 | 67   | 跟着张三遛狗 |
| 王五 | 23   |
| 赵六 |
| 没有 |
```
渲染后:

| 姓名 | 年龄 | 工作         |
| ---- | ---- | ------------ |
| 张三 | 34   | 遛<br />狗   |
| 李四 | 67   | 跟着张三遛狗 |
| 王五 | 23   |
| 赵六 |
| 没有 |

### 引用
使用 `>` 后面跟引用的内容。如果不空行那么引用将会一直持续下去

>这是一个引用，而这个引用非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长非常长

>这是另一个引用，与上一个引用空了一行
这里只是换了一行，但引用没有停

### 公式

`$行内公式$`与`$$换行公式$$`，在换行公式中换行使用`\\`。
例:
```
$y=x+1$ $$y=x+1\\=2$$
```
渲染后:

$y=x+1$

$$y=x+1\\=2$$

## latex语法

>**以下内容必须包含在行内公式或换行公式中。**
>**公式中使用{}来表示一个整体**

### 希腊字符

| 语法    | 效果        |  语法    | 效果        |
| ------- | ---------- |----------|-------------|
| `\alpha`   | $\alpha$    | `\beta`    | $\beta$     |
| `\gamma`   | $\gamma$    | `\delta`   | $\delta$    |
| `\epsilon` | $\epsilon$  | `\zeta`    | $\zeta$     |
| `\eta`     | $\eta$      | `\theta`   | $\theta$    |
| `\iota`    | $\iota$     | `\kappa`   | $\kappa$    |
| `\lambda`  | $\lambda$   | `\mu`      | $\mu$       |
| `\nu`      | $\nu$       | `\xi`      | $\xi$       |
| `\omicro`n | $\omicron$  | `\pi`      | $\pi$       |
| `\rho`     | $\rho$      | `\sigma`   | $\sigma$    |
| `\tau`     | $\tau$      | `\upsilon` | $\upsilon$  |
| `\phi`     | $\phi$      | `\chi`     | $\chi$      |
| `\psi`     | $\psi$      | `\omega`   | $\omega$    |

将首字母大写可切换到对应的希腊字符大写, 例如 `$\Theta$` 会被渲染为 $\Theta$

### 基本运算符

| 语法          | 效果          |     | 语法       | 效果       |
| ------------- | ------------- | --- | ---------- | ---------- |
| `\pm`          | $\pm$         |     | `\mp`       | $\mp$      |
| `\times`       | $\times$      |     | `\div`      | $\div$     |
| `\cdot`        | $\cdot$       |     | `\sqrt{x}` | $\sqrt{x}$ |
| `\sqrt[3]{x}` | $\sqrt[3]{x}$ |

注: pm 分别代表加(plus)和减(minus)

注: cdot 的 c 代表中间(center)的意思

### 比较符号

| 语法  | 效果   |     | 语法      | 效果      |
| ----- | ------ | --- | --------- | --------- |
| `gt`  | $\gt$  |     | `lt`      | $\lt$     |
| `geq` | $\geq$ |     | `leq`     | $\leq$    |
| `neq` | $\neq$ |     | `\approx` | $\approx$ |

注: g - greater, l - less, eq - equal, n - not, t - than

### 极限

`\lim_{i \to 0}`中`\lim`是极限符号，`\to`是箭头。渲染为 $\lim_{i \to 0}$

### 求和符号

| 语法                     | 效果                     |     | 语法             | 效果              |
| ------------------------ | ------------------------ | --- | ---------------- | ----------------- |
| `\sum`                   | $\sum$                   |     |
| `\sum\limits_{i=0}`      | $\sum\limits_{i=0}$      |     | `\sum_{i=0}`     | $\sum_{i=0}$      |
| `\sum\limits_{i=0}^{10}` | $\sum\limits_{i=0}^{10}$ |     | `\sum{i=0}^{10}` | $\sum_{i=0}^{10}$ |

### 求积符号

| 语法                      | 效果                      |     | 语法              | 效果               |
| ------------------------- | ------------------------- | --- | ----------------- | ------------------ |
| `\prod`                   | $\prod$                   |     |
| `\prod\limits_{i=0}`      | $\prod\limits_{i=0}$      |     | `\prod_{i=0}`     | $\prod_{i=0}$      |
| `\prod\limits_{i=0}^{10}` | $\prod\limits_{i=0}^{10}$ |     | `\prod{i=0}^{10}` | $\prod_{i=0}^{10}$ |



### 积分

积分有三种，一重积分`int`，二重积分`iint`，三重积分`iiint`，都可以类似下表使用:

| 语法                     | 效果                     |     | 语法             | 效果              |
| ------------------------ | ------------------------ | --- | ---------------- | ----------------- |
| `\int`                   | $\int$                   |     |
| `\int\limits_{i=0}`      | $\int\limits_{i=0}$      |     | `\int_{i=0}`     | $\int_{i=0}$      |
| `\int\limits_{i=0}^{10}` | $\int\limits_{i=0}^{10}$ |     | `\int{i=0}^{10}` | $\int_{i=0}^{10}$ |

### 分式

`$\frac{Numerator}{Denominator}$` 渲染为 $\frac{Numerator}{Denominator}$

例如 $y=\frac{2}{x+1}$

### 括号

在使用换行公式时(也就是`$$`)，有如下两种括号样式，第二种会适应公式大小。这是latex中的语法。
```
$$(\frac{1}{x+1})$$

$$\left(\frac{1}{x+1}\right)$$
```
$$(\frac{1}{x+1})$$

$$\left(\frac{1}{x+1}\right)$$

### 偏微分
`\partial` : $\partial$

需要注意如果在 `\partial` 后面跟其他公式，用{}括起来 `y=\frac{\partial{f(x)}}{\partial{x}}`: $y=\frac{\partial{f(x)}}{\partial{x}}$

条件偏导 `\left. \frac{\partial{f(x)}}{\partial{x}} \right|_{x=0}`:
$\left. \frac{\partial{f(x)}}{\partial{x}} \right|_{x=0}$

这里的 `\left.` 匹配了 `\right|`，完成了闭合，类似的用法还有 `\left\{ \right.`，其可以仅渲染左花括号(我的实测仅在换行公式中成功)，效果如下： 

$$ \left\{ \right. $$

### 矩阵

矩阵开始于`\begin{matrix}`，结束于`\end{matrix}`，每一行中的元素用 `&` 分隔，行后用 `\\` 换行。
```
$A=\left[ \begin{matrix} 1402 & 191  \\ 1371 & 821  \\ 949 & 1437  \\ 147 & 1448  \\\end{matrix} \right]$
```
$A=\left[ \begin{matrix} 1402 & 191  \\ 1371 & 821  \\ 949 & 1437  \\ 147 & 1448  \\\end{matrix} \right]$

### 绝对值/行列式

与括号类似可以使用`\left \right`来适应公式大小。
```
$A = \left| \begin{matrix} 1 & 2 \\ 3 & 4 \\ \end{matrix}\right|$
$\lvert x \rvert$
```
$A = \left| \begin{matrix} 1 & 2 \\ 3 & 4 \\ \end{matrix}\right|$
$\lvert x \rvert$

### 范数

```
$\|x\|$
```
$\|x\|$

### 集合符号

| 语法        | 效果         |     | 语法        | 效果         |
| ----------- | ------------ | --- | ----------- | ------------ |
| `\in`        | $\in$        |     | `\notin`     | $\notin$     |
| `\subset`    | $\subset$    |     | `\supset`    | $\supset$    |
| `\subseteq`  | $\subseteq$  |     | `\supseteq`  | $\supseteq$  |
| `\nsubseteq` | $\nsubseteq$ |     | `\nsupseteq` | $\nsupseteq$ |
| `\cup`       | $\cup$       |     | `\cap`       | $\cap$       |

### 广义交/并符号

| 语法                        | 效果                        |     | 语法                | 效果                 |
| --------------------------- | --------------------------- | --- | ------------------- | -------------------- |
| `\bigcup`                   | $\bigcup$                   |     |
| `\bigcup\limits_{i=0}`      | $\bigcup\limits_{i=0}$      |     | `\bigcup_{i=0}`     | $\bigcup_{i=0}$      |
| `\bigcup\limits_{i=0}^{10}` | $\bigcup\limits_{i=0}^{10}$ |     | `\bigcup{i=0}^{10}` | $\bigcup_{i=0}^{10}$ |

| 语法                        | 效果                        |     | 语法                | 效果                 |
| --------------------------- | --------------------------- | --- | ------------------- | -------------------- |
| `\bigcap`                   | $\bigcap$                   |     |
| `\bigcap\limits_{i=0}`      | $\bigcap\limits_{i=0}$      |     | `\bigcap_{i=0}`     | $\bigcap_{i=0}$      |
| `\bigcap\limits_{i=0}^{10}` | $\bigcap\limits_{i=0}^{10}$ |     | `\bigcap{i=0}^{10}` | $\bigcap_{i=0}^{10}$ |


### 独立同分布

```
\stackrel{\text{i.i.d}}{\sim}
```
$\stackrel{\text{i.i.d}}{\sim}$

### 其他符号

| 语法      | 效果      |     | 语法        | 效果        |
| --------- | --------- | --- | ----------- | ----------- |
| `\bar{x}` | $\bar{x}$ |     | `\hat{x}`   | $\hat{x}$   |
| `\dot{x}` | $\dot{x}$ |     | `\tilde{x}` | $\tilde{x}$ |
| `\vec{x}` | $\vec{x}$ |     | `\cdotp`    | $\cdotp$    |
| `\cdots`  | $\cdots$  |     | `\ddots`    | $\ddots$    |
| `\dots`   | $\dots$   |     | `\vdots`    | $\vdots$    |
| `\infty`  | $\infty$  |
| `\bot`    | $\bot$    |     | `propto`    | $\propto$   |
| `circ`    | $\circ$   |     | `bullet`    | $\bullet$   |

### 方程

其中cases可以显示为大括号
```
\begin{equation}
a(t)=
\begin{cases}
x=Q(x)=q_{1}& \text{ $ x \in [minimum,C] $ } \\
x=Q(x)=q_{2}& \text{ $ x \in [C,maximum] $ }
\end{cases}
\end{equation}
```
$$
\begin{equation}
a(t)=
\begin{cases}
x=Q(x)=q_{1}& \text{ $ x \in [minimum,C] $ } \\
x=Q(x)=q_{2}& \text{ $ x \in [C,maximum] $ }
\end{cases}
\end{equation}
$$

只写一个 cases 也是可以的，这样子就不会有方程编号了
```
a(t)=
\begin{cases}
x=Q(x)=q_{1}& \text{ $ x \in [minimum,C] $ } \\
x=Q(x)=q_{2}& \text{ $ x \in [C,maximum] $ }
\end{cases}
```
$$
a(t)=
\begin{cases}
x=Q(x)=q_{1}& \text{ $ x \in [minimum,C] $ } \\
x=Q(x)=q_{2}& \text{ $ x \in [C,maximum] $ }
\end{cases}
$$