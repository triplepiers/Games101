
变换主要包含两部分： Modeling (模型) 与 Viewing (视角)

## 2D Transformations

### 1.1 Linear
> 可以通过 (相同维度的) 矩阵相乘实现的变换

即满足：

$$
x' = M x \Rightarrow ß\left[
	\begin{array}{1}
	            x' \\
	            y'
	        \end{array}
	\right] = 
	    \left[
	        \begin{array}{2}
	            a & b \\
	            c & d
	        \end{array}
	    \right]
	    \left[
	        \begin{array}{1}
	            x \\
	            y
	        \end{array}
	    \right]
$$

> 其方程组形式为 $x,y = ax + by$，**没有常数项**

#### 1 Scale 缩放

若以原点 $(0,0)$ 对点 $(x,y)$ 作 $s$ 倍的放缩，有：

$$
\left\{
    \begin{align*}
	    x' &= s \cdot x \\
        y' &= s \cdot y
    \end{align*}
\right. \Rightarrow \left[
        \begin{array}{1}
            x' \\
            y'
        \end{array}
    \right] = 
    \left[
        \begin{array}{2}
            s & 0 \\
            0 & s
        \end{array}
    \right]
    \left[
        \begin{array}{1}
            x \\
            y
        \end{array}
    \right]
$$

若 $x, y$ 方向具有不同的缩放倍率，我们可以将上式修改为：

$$
\left[
        \begin{array}{1}
            x' \\
            y'
        \end{array}
\right] = 
    \left[
        \begin{array}{2}
            s_x & 0 \\
            0 & s_y
        \end{array}
    \right]
    \left[
        \begin{array}{1}
            x \\
            y
        \end{array}
    \right]
$$

#### 2 Reflection 对称

将图像沿 y 轴对称翻转，有：

$$
\left\{
	\begin{align*}
		x' &= -x \\
		y' &= y
	\end{align*}
\right.
$$

将其改写为矩阵形式，有：

$$
\left[
        \begin{array}{1}
            x' \\
            y'
        \end{array}
    \right] = 
    \left[
        \begin{array}{2}
            -1 & 0 \\
            0 & 1
        \end{array}
    \right]
    \left[
        \begin{array}{1}
            x \\
            y
        \end{array}
    \right]
$$

#### 3 Shear 切变

- 纵坐标 *不变*
- 横坐标变化 *正比于* 纵坐标 $\Delta x = a \cdot y$

$$
\left[
        \begin{array}{1}
            x' \\
            y'
        \end{array}
    \right] = 
    \left[
        \begin{array}{2}
            1 & a \\
            0 & 1
        \end{array}
    \right]
    \left[
        \begin{array}{1}
            x \\
            y
        \end{array}
    \right]
$$

#### 4 Rotate 旋转
> 默认绕 $(0,0)$ 沿 *逆时针* 方向旋转

![IMG_3381.png](https://img1.imgtp.com/2023/10/21/fE34B15K.png)

考虑边长为 1 的正方形绕逆时针旋转 $\theta$ 的情形，考虑两个特殊点：

$$
\left\{
	\begin{align*}
		[1,0] & \rightarrow [cos\theta, sin\theta] \\
		[0,1] & \rightarrow [-sin\theta, cos\theta]
	\end{align*}
\right.
$$

已知变换矩阵大小为 2 x 2，联立方程组即可解得：

$$
R_{\theta} = \left[
	\begin{array}{2}
		cos\theta & -sin\theta \\
		sin\theta & cos\theta
	\end{array}
\right]
$$

### 1.2 Homogeneous Coordinates
#### Translation 平移
对于平移变换 $T_{t_x, t_y}$ 我们可以直接写出其坐标变换方程：

$$
\left\{
	\begin{align*}
		x' &= x + t_x \\
		y' &= y + t_y
	\end{align*}
\right.
$$

但却 *无法* 将其写为 $x' = Mx$ 的形式（后面会多一个常数项）：

$$
\left[
	\begin{array}{1}
		x' \\ y'
	\end{array}
\right] = \left[
	\begin{array}{2}
		a & b\\ c & d
	\end{array}
\right] \left[
	\begin{array}{1}
		x \\ y
	\end{array}
\right] + \left[
	\begin{array}{1}
		t_x \\ t_y
	\end{array}
\right] 
$$

> 因此 *平移变换* 不是线性变换

!!! question "如何统一 平移变换 & 线性变换 的描述形式？"

#### 齐次坐标实现

我们需要增加第三个维度 w-coordinate：
> 因为向量具有 *平移不变性*，所以需要区别对待

- 对于 2D Point，表示为 $[x,y,1]^T$
- 对于 2D Vector，表示为 $[x,y,0]^T$

此时，点坐标 **平移** 的矩阵变换可以改写为：

$$
\left(
	\begin{array}{1}
		x' \\ y' \\ w'
	\end{array}
\right) = \left(
	\begin{array}{3}
		1 & 0 & t_x \\
		0 & 1 & t_y \\
		0 & 0 & 1
	\end{array}
\right) \cdot \left(
	\begin{array}{1}
		x \\ y \\ 1
	\end{array}
\right) = \left(
	\begin{array}{1}
		x + t_x \\ y + t_y \\ 1
	\end{array}
\right) 
$$

下面是一些 Point & Vector 混合运算的法则（看第三个维度）：

- Vector + Vector = Vector
- Point - Point = Vector
- Point + Vector = Point
- Point + Point = Point（齐次坐标下表示 *连线中点*）

> 对于点 $(x,y,w)$ ，若 $w \neq 0,\ 1$ 我们便将其归约为 $(\frac{x}{w}, \frac{y}{w}, 1)$ 的形式
> 
> 因此，Point + Point 时第三维 $=2$ 的情形可被归约为 $=1$（Point）的情形

#### Affine 仿射

Affine Map = Linear Map + Translation，即：

$$
\left(
	\begin{array}{1}
		x' \\ y'
	\end{array}
\right) = \left(
	\begin{array}{2}
		a & b \\ c & d
	\end{array}
\right) \cdot \left(
	\begin{array}{1}
		x \\ y
	\end{array}
\right) + \left(
	\begin{array}{1}
		t_x + t_y
	\end{array}
\right)
$$
在齐次坐标系下，仿射变换均可书写为：

$$
\left(
	\begin{array}{1}
		x' \\ y' \\ 1
	\end{array}
\right) = \left(
	\begin{array}{3}
		a & b & t_x \\
		c & d & t_y \\
		0 &0 & 1
	\end{array}
\right) \cdot \left(
	\begin{array}{1}
		x \\ y \\ 1
	\end{array}
\right) = \left(
	\begin{array}{1}
		ax + by + t_x \\ cx + dy + t_y \\ 1
	\end{array}
\right) 
$$

### 1.3 Inverse 逆变换

$M^{-1}$ is the inverse of transform $M$ in *both* <u>matrix & geometric</u> sense，即：

$x' \cdot M^{-1} = M \cdot x \cdot M^{-1} = x$ （恢复到初始状态）

### 1.4 Composite 变换组合

!!! warning "Transform ORDERing matters!"

由于坐标是 *基于原点* 变换的，不同的的变换组合顺序可能导致不同的变换结果：

- 上图：先平移后旋转
- 下图：先旋转后平移

> 从线代角度看，这是因为矩阵乘法 *不满足交换律*。

![截屏2023-10-21 15.30.45.png](https://img1.imgtp.com/2023/10/21/JzuQhZ5O.png)

当我们希望依次应用变换 $A_1, A_2, A_3, ... , A_n$ 时，等价于（从右向左）依次执行矩阵乘法，即：

$$
A_(...(A_2(A_1(x)))) = A_n \cdot ... \cdot A_2 \cdot A_1 \cdot \left(
	\begin{array}{1}
		x \\ y \\ 1
	\end{array}
\right)
$$

由于矩阵乘法具有 *结合律*，我们可以计算 $A = \prod_{i=1}^{n} A_i$ 并通过 *一次矩阵乘法* 得到等价的组合变换结果。

#### Decompose 变换分解

!!! question "如何使得图形基于 *给定点* 旋转？"

答案是三步走：

$$
M = T(c) \cdot R(\alpha) \cdot T(-c)
$$

1. 平移：将给定点移动到原点
2. 旋转：（绕原点）旋转指定角度
3. 平移：将图形平移回原位

## 3D Transformations

我们同样可以将齐次坐标应用至三维空间，此时需要增加第四维度。有：

$$
\left\{
	\begin{align*}
		& \text{3D Point }  [x,y,z,1]^T \\
		& \text{3D Vector } [x,y,z,0]^T 
	\end{align*}
\right.
$$

> 同样的，当 $w \neq 0, \ 1$ 时，我们需要将 3D Point 归约为 $(\frac{x}{w},\frac{y}{w},\frac{z}{w},1)$

此时，我们使用 $M_{4 \times 4}$ 来描述三维放射变换：

$$
\left(
	\begin{array}{1}
	x' \\ y' \\ z' \\1
	\end{array}
\right) = 
\left(
	\begin{array}{4}
		a & b & c & t_x \\
		d & e & f & t_y \\
		g & h & i & t_z \\
		0 & 0 & 0 & 1
	\end{array}
\right) \cdot \left(
	\begin{array}{1}
	x \\ y \\ z \\1
	\end{array}
\right)
$$
!!! info "应用顺序是 *先线性变换，后平移* （参考形式 $x' = ax + b$）"