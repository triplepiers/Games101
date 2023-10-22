
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
旋转 $-\theta$ (逆运算) 的情况如下：

$$
R_{-\theta} = \left[
	\begin{array}{2}
		cos\theta & sin\theta \\
		-sin\theta & cos\theta
	\end{array}
\right] = R_{\theta}^T = R_{\theta}^{-1}
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

### 2.1 3D Affine

#### Scale

$$
S(s_x,s_y,s_z) = \left(
	\begin{array}{4}
		s_x & 0 & 0 & 0 \\
		0 & s_y & 0 & 0 \\
		0 & 0 & s_z & 0 \\
		0 & 0 & 0 & 1
	\end{array}
\right)
$$

#### Translation

$$
T(t_x,t_y,t_z) = \left(
	\begin{array}{4}
		0 & 0 & 0 & t_x \\
		0 & 0 & 0 & t_y \\
		0 & 0 & 0& t_z \\
		0 & 0 & 0 & 1
	\end{array}
\right)
$$

#### Rotation

分别考虑物体绕 x, y, z 轴旋转的情况（对应坐标不变）：
> 由于 $y = z \times x = - (x \times z)$ ，所以绕 y 轴旋转的矩阵略有不同

$$
\begin{align*}
	R_x(\alpha) &=  \left(
		\begin{array}{4}
			1 & 0 & 0 & 0 \\
			0 & cos\alpha & -sin\alpha & 0 \\
			0 & sin\alpha & cos\alpha & 0 \\
			0 & 0 & 0 & 1
		\end{array}
	\right) \\
	R_y(\alpha) &=  \left(
		\begin{array}{4}
			cos\alpha & 0 & sin\alpha & 0 \\
			0 & 1 & 0 & 0 \\
			-sin\alpha & 0 & cos\alpha & 0 \\
			0 & 0 & 0 & 1
		\end{array}
	\right) \\
	R_z(\alpha) &=  \left(
		\begin{array}{4}
			cos\alpha & -sin\alpha & 0 & 0 \\
			sin\alpha & cos\alpha& 0 & 0 \\
			0 & 0 & 1 & 0 \\
			0 & 0 & 0 & 1
		\end{array}
	\right)
\end{align*}
$$

如果我们需要物体沿多个轴旋转 => 那么转三次就行了 hhhh
> 这玩意儿也被称作 *Euler Angles*，等价于位姿描述中的 roll, pitch & yaw

$$
R_{xyz}(\alpha,\beta,\gamma) = R_x(\alpha) \cdot R_y(\beta) \cdot R_z(\gamma)
$$

!!! info "Rodrigues' Rotation Formula"
	围绕轴 $\vec{n}$，旋转角 $\alpha$ 的结果可以写为如下形式：
	> 此处的 $\vec{n}$ 表示任意方向（可以被分解到 $x,y,z$ 方向）
	> 
	> 我们默认旋转轴 $\vec{n}$ *经过原点*，即旋转操作 <u>绕原点进行</u>
	
	$$
	R(\vec{n},\alpha) = cos\alpha \cdot \vec{I} + (1-cos\alpha)\vec{n}\vec{n}^T +sin\alpha \cdot \left(
		\begin{array}{3}
			0 & -n_z & n_y \\
			n_z & 0 & -n_x \\
			-n_y & n_x & 0
		\end{array}
	\right)
	$$


### 2.2 View / Camera 视图

- 定义相机位姿的三要素
  
	1. Position $\vec{e}$ ：描述相机位于三维空间中的哪一点
	2. Look-at Direction $\hat{g}$：描述相机在 360 度中的朝向
	3. Up Direction $\hat{t}$：描述相机的仰角

#### $M_{view}$

如果我们沿着相同的向量移动物体和相机，将得到和原先一致的观测结果

!!! tip "为什么不让相机保持在 $(0,0,0)$ + look at $-Z$ 方向 + up at $+Y$ 方向，然后去挪动所有的物体呢？"

我们将描述相机从原位姿变换为标准位姿的过程记为 $M_{view}$，它包含以下几步：

1. 将相机 <u>平移</u> 至原点
2. 将 $\hat{g}$ 旋转至 $-Z$ 方向
3. 将 $\hat{t}$ 旋转至 $+Y$ 方向
4. 将 $(\hat{g} \times \hat{t})$ 旋转至 $+X$ 方向

将 Step 1 所用的平移矩阵记为 $T_{view}$，Step 2-4 所有的(合成)旋转矩阵记为 $R_{view}$，不难得出 $M_{view} = R_{view} \cdot T_{view}$：

- 我们可以方便的写出平移矩阵：

	$$
	T_{view} = \left[
		\begin{array}{4}
			1 & 0 & 0 & -x_e \\
			0 & 1 & 0 & -y_e \\
			0 & 0 & 1 & -z_e \\
			0 & 0 & 0 & 1
		\end{array}
	\right]
	$$

- 旋转部分，我们先考虑 $X \rightarrow (g \times t),\ Y \rightarrow t,\ Z \rightarrow -g$，然后取逆矩阵（刚好是转置）

	$$
	R_{view}^{-1} = \left[
		\begin{array}{4}
			x_{g \times t} & x_t & x_{-g} & 0 \\
			y_{g \times t} & y_t & y_{-g} & 0 \\
			z_{g \times t} & z_t & z_{-g} & 0 \\
			0 & 0 & 0 & 1
		\end{array}
	\right]
	$$
	$$
	\Rightarrow R_{view} = (R_{view}^{-1})^{-1} = (R_{view}^{-1})^T =\ ...
	$$

### 2.3 Projection 投影

#### 1 Orthographic 正交
> 三步走

1. 把相机放到 $(0,0,0)$， look at $-Z$, up at $+Y$（很熟悉）
2. *丢掉 Z 轴坐标*（暂时不考虑前后遮盖问题）
3. 将结果等比放缩至 $[-1,1]^2$ 围成的正方形范围中

---
下面是一个更正式的做法：

- （用 $x,y,z$ 轴坐标范围）定义空间中的一个 长方体(cuboid) $[l,r] \times [b,t] \times [f,n]$
- 将立方体的 *中心* <u>平移</u> 至 $(0,0,0)$
- 将长方体 <u>放缩</u> 至 标准立方体(Canonical Cube) $[-1,1]^3$

> 由于相机 look at $-Z$ 方向，所以 *近处* 的物体将具有 *更大的 $Z$ 坐标*（使用左手系时结果相反）


我们可以写出上述过程对应的变换矩阵 $M_{ortho} = R \cdot T$：

1. 平移 $\frac{top+bot}{2}$ 的距离
2. 边长放缩至 2（ $-1 \to 1$ 的距离）

$$
M_{ortho} = \left[
	\begin{array}{4}
		\frac{2}{r-l} & 0 & 0 & 0 \\
		0 & \frac{2}{t-b} & 0 & 0 \\
		0 & 0 & \frac{2}{n-f} & 0 \\
		0 & 0 & 0 & 1
	\end{array}
\right] \left[
	\begin{array}{4}
		1 & 0 & 0 & -\frac{r+l}{2}\\
		0 & 1 & 0 & -\frac{t+b}{2}\\
		0 & 0 & 1 & -\frac{n+f}{2}\\
		0 & 0 & 0 & 1
	\end{array}
\right]
$$
#### 2 Perspective 透视
> 近大远小，（几何）平行线不再平行（延长后相较于消失点）

![截屏2023-10-22 00.23.27.png](https://img1.imgtp.com/2023/10/22/03Whmlvf.png)

透视投影的可视空间是一个 视锥（Frustum），我们可以分两步走：

1. Squish the *frustum* into a *cuboid*（转换为正交投影）   
2. 进行一次正交投影

!!! question "如何描述一个 Frustum ？"
	Vertical **field-of-view** (fovY) + **aspect ratio**（定义了底面和顶角）
	> 其中 fovY 为 相机 与 窗口上下底边中点 连线所形成的夹角
	
	<div style="width:100%; display:flex; justify-content:center;">
		<img src="https://img1.imgtp.com/2023/10/22/X4bk6Fxb.png" style="width:350px;">
	</div>
#### Squish

![截屏2023-10-22 00.29.35.png](https://img1.imgtp.com/2023/10/22/asDdrxdB.png)

??? info "使用 fovY + aspect ratio 描述的情况"
	![截屏2023-10-22 14.09.09.png](https://img1.imgtp.com/2023/10/22/GFIlQivz.png)

设挤压过程为 $(x,y,z) \rightarrow (x',y',n)$，有：
> 其中 $n$ 为未知数

$$
\left\{
	\begin{align*}
		y' &= \frac{n}{z}y \\
		x' &= \frac{n}{z}x
	\end{align*}
\right.
$$

将变换前后的坐标写成齐次形式，有：

$$
\left(
	\begin{array}{1}
		x \\ y \\ z \\ 1
	\end{array}
\right) \Rightarrow \left(
	\begin{array}{1}
		\frac{nx}{z} \\ \frac{ny}{z} \\ \text{unknown} \\ 1
	\end{array}
\right) = \left(
	\begin{array}{1}
		nx \\ ny \\ \text{unknown} \\ z
	\end{array}
\right)
$$
我们可以填出 $M_{persp \rightarrow ortho}$ 中的部分值：

$$
M_{persp \rightarrow ortho} = \left[
	\begin{array}{4}
		n & 0 & 0 & 0 \\
		0 & n & 0 & 0 \\
		? & ? & ? & ? \\
		0 & 0 & 1 & 0
	\end{array}
\right]
$$

又因为：

- 最近平面上的所有点 (所有)坐标 在挤压前后 *不变*

	$$
\left(
	\begin{array}{1}
		x \\ y \\ n \\ 1
	\end{array}
\right) = \left(
	\begin{array}{1}
		nx \\ ny \\ n^2 \\ n
	\end{array}
\right) \Rightarrow [?,?,?,?] = [0,0,A,B] \ and \ An + B = n^2
$$

- 最远平面上的所有点 $Z$坐标 在挤压前后 *不变*

	我们取特例 —— 中心点 $(0,0,f)$ => 挤压后还是 $(0,0,f)$
	
	$$
\left(
	\begin{array}{1}
		0 \\ 0 \\ f \\ 1
	\end{array}
\right) = \left(
	\begin{array}{1}
		0 \\ 0 \\ f^2 \\ f
	\end{array}
\right) \Rightarrow Af + B = f^2
$$


可以解得 $A,B$ 并补全 $M_{persp \rightarrow ortho}$