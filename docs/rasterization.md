在经过 MVP(Model + View + Projection) 变换后，所有的物体将被归约至 $[-1,1]^3$ 的 Canonical Cube 中。我们接下来的任务便是通过 *光栅化(Rasterization)* 将其绘制到屏幕上。

> 在图形学中，我们认为 screen 就是一个由 pixel 构成的二维数组。

??? info "Screen Space"
	> 本课程中的定义和 Tiger Book 中稍微有一些出入
	
	- 屏幕坐标系
		- 我们以 <u>左下角</u> 为屏幕坐标系的原点
		- $+X$ 方向水平向右，$+Y$ 方向竖直向上
	- 像素坐标
		- 我们使用 $\{(x,y) | x,y \in \mathbf{N}, x \lt width, y \lt height\}$ 来描述像素坐标
		- 每个像素（方格）的中心位于 $(x+0.5, y+0.5)$
	  
	<div style="width:100%;display:flex;justify-content:center;">
		<img src="https://img1.imgtp.com/2023/10/22/RN1updWB.png" style="width:300px;">
	</div>

- 从 Canonical Cube 到 Screen

	由于这个过程与 $Z$轴 坐标无关，所以我们要做的实际上是将 $XY$平面 $[-1,1]^2$ 拉伸到屏幕平面 $[0,width]\times[0,height]$ 
	
	=> 我们通过一个 $M_{viewport}$ 变换来实现
	> 先<u>放缩</u>到对应大小，然后将左下角<u>平移</u>至 $(0,0,0)$
	
	$$
	M_{viewport} = \left(
		\begin{array}{4}
			\frac{width}{2} & 0 & 0 & \frac{width}{2}\\
			0 & \frac{height}{2} & 0 & \frac{height}{2}\\
			0 & 0 & 1 & 0\\
			0 & 0 & 0 & 1
		\end{array}
	\right)
	$$

## 1 Triangles 三角形光栅化

!!! question "为什么选用三角形面片？"

- Triangle as Fundamental Shape Primitives
	- 三角形是最简单的多边形 —— 我们可以把任意多边形拆分为若干三角形
	- 三角形 *一定是平面的*
	- 三角形的 内外 划分十分清晰（不存在凹凸多边形问题）
	- 定义三个顶点的属性后，非常容易在其内部做过渡插值
		
		Well-defined method for interpolating values at vertices over triangle(barycentric interpolation)

### Sampling 采样

!!! question "Pixel Approximate：如何用像素描述一个三角形？"

- Sampling
	- 给出一个 *连续函数*，求其在特定位置的值。
	- We can *discretize* a function by sampling.

	```c title="f(x) 是一个连续函数，下面是一个采样过程"
	for (int x = 0 ; i < max_x ; i++) {
		output[x] = f(x);
	}
	```

#### Point-in-Triangle Test

对于给定的三角形，我们可以对 <u>Pixel Center is inside Triangle</u> 的像素点进行采样：

<div style="width:100%;display:flex;justify-content:center;">
	<img src="https://img1.imgtp.com/2023/10/22/kLrR5V7O.png" style="width:400px;">
</div>

可以使用函数 `inside(tri, x, y)` 进行描述：

$$
inside(tri,x,y) = \left\{
	\begin{align*}
	1&, (x,y) \ \text{in triangle tri} \\
	0&, \text{otherwise}
	\end{align*}
\right.
$$

如果采用 binary image，我们可以使用如下函数获得光栅化结果：

```c
int image[X_MAX][Y_MAX];

for (int i = 0 ; i < X_MAX ; i++) {
	for (int j = 0 ; j < Y_MAX ; j++) {
		image[i][j] = inside(tri, x+0.5, y+0.5); // 判断像素中心是否处于界内
	}
}
```

??? info "使用 Cross Product 判断点是否处于三角形内"
	<div style="width:100%;display:flex;justify-content:center;">
		<img src="https://img1.imgtp.com/2023/10/22/HlPl9av0.png" style="width:400px;">
	</div>

	通过 $\vec{P_0P_1} \times \vec{P_0Q},\vec{P_1P_2} \times \vec{P_1Q},\vec{P_2P_0} \times \vec{P_2Q}$ 三次叉乘，我们可以判断 $Q$ 处于 7分 空间的那一份中
	
	当切仅当<u>三次叉乘结果符号相同</u>时，$Q$ 处于三角形内

!!! info "Edge Class"
	当像素中心 *刚好处于图形边缘线* 上时，应该做统一处理。
	> 本课程中统一忽略这些点
	
!!! info "Bounding Box"
	我们显然不需要判断 *屏幕上所有的 pixel* 是否落在三角形内（位置超过上下界的像素点显然没有判断的意义）
	
	=> 只需要判断落在 (Axis-Aligned) Bounding Box 内的 pixel 即可：

	- `X_MIN = min(P0_X, P1_X, P2_X), X_MAX = max(P0_X, P1_X, P2_X);`
	- `Y_MIN, Y_MAX` 同理
	
	<div style="width:100%;display:flex;justify-content:center;">
		<img src="https://img1.imgtp.com/2023/10/22/8xlMhy7L.png" style="width:350px;">
	</div>

	对于 瘦长/旋转后 的三角形，我们可以对 *每一行* 取 MIN-MAX 进行判断：

	<div style="width:100%;display:flex;justify-content:center;">
		<img src="https://img1.imgtp.com/2023/10/22/cap3fayA.png" style="width:350px;">
	</div>

!!! warning "Sampling 会导致 aliasing => 具体表现为 *锯齿 jaggies*（我们会在下面进行讨论）"

## 2 Antialiasing 反走样

### 2.1 Sampling Theory

#### Fourier Transform 傅立叶级数展开

任何一个周期函数都可以写成 *一系列* $cos,\ sin$  的线性组合（当然还有常数项）

=> Fourier Transform 将信号分为 frequencies（将信号从时域变换至频域）

![截屏2023-10-22 16.16.27.png](https://img1.imgtp.com/2023/10/22/spdPF66N.png)

---

!!! info "Higher Frequencies need Faster Sampling"

![截屏2023-10-22 16.19.20.png](https://img1.imgtp.com/2023/10/22/Yt94WyKN.png)

我们可以对分解得到的正余弦波进行采样，然后通过连线去近似：

- 在频率较低的正弦波中，这一方法具有较好的结果
- 但在频率较高的正弦波中（如 $f_5(x)$），效果十分糟糕

	事实上，这可能导致 *高频信号* 与 *低频信号* 呈现相同的采样结果（下图中的 蓝/黑 信号在该采样频率下的表现完全一致）
	![截屏2023-10-22 16.24.00.png](https://img1.imgtp.com/2023/10/22/jyAk8CVf.png)

#### Filtering 滤波
!!! notes "角度A：<u>Get rid of</u> certain frequency contents."

事实上，大多数有意义的信息都集中在 *低频* 区域

1. Filter out *low frequency*（高通滤波）

	得到的结果是 Edges => 边界是信号发生 *突变* 的区域

2. Filter out *high frequency* （低通滤波）

	使图像模糊 => 因为边界被丢掉了

3. Filter out *low & high frequency* 

	提取不太明显的边界特征 => 最明显的边界 & 大规模相似的填充被丢掉了

!!! notes "角度B：Convolution (卷积) / Averaging"

- Convolution Theorem

	时域上的卷积结果 = 频域上的乘积

所以当我们希望对一个信号进行卷积操作时，有两种方案：

1. 直接对 Spatial Domain (原信号) 进行卷积操作

2. 从 "频域上的乘积" 角度出发：

	1. 使用 Fourier Transform 将原信号转换至 Frequency Domain
	2. 将结果与 convolution kernel 的 Fourier Transform 结果相乘
	3. 使用 inverse Fourier 将乘积转换会 Spatial Domain

!!! tip "Blur (Box Filter) 操作的两种实现"
![截屏2023-10-22 16.43.59.png](https://img1.imgtp.com/2023/10/22/G22oaX6m.png)

??? info "Box Filter (Low Pass)"
	上图中采用的卷积核也被称为 Box Filter

	- 开头的 $\frac{1}{9}$ 是为了对结果进行归一化，避免亮度上的问题
	- 模糊：其 Fourier Transform 的结果集中在 *低频* 区域 => 相乘后保留低频部分
	- *wider kernel* (3 x 3) 在频域上对应了 *lower frequency* => 更加模糊

#### Sampling 采样
> Repeating Frequency Contents

- 在时域上

	Sampling = $f(x) \times \text{冲击函数}$

- 在频域上

	Sampling =  $f(x) \text{和冲击函数的卷积}$
	
	=> 以冲击函数频率为间隔 *复制粘贴* $f(x)$ 的频谱
	
	!!! danger "采样频率低时，频谱之间的 *交叠* 导致了 走样 (Aliasing)"

![截屏2023-10-22 16.57.52.png](https://img1.imgtp.com/2023/10/22/npCrWqZH.png)

### 2.2 *Sampling* Artifacts in CG

!!! info "其根本原因是 信号变化频率 > 采样频率"

- 锯齿 Jaggies (Staircase Pattern)
  
- 摩尔纹 Moire Patterns

	在忽略原图的 odd rows & columns 后产生

- Wagon Wheel Illusion (False Motion)

	由于人眼采样频率低于运动速度而产生

### 2.3 Blurring (Pre-Filtering) *before* sampling

以三角形光栅化为例：

- 对 binary image 采样会导致锯齿：像素非黑即白
- 但对原图进行 blur 后进行采样，使得像素内容可以取一些 intermediate value 就可以一定程度上实现抗锯齿

![截屏2023-10-22 16.06.25.png](https://img1.imgtp.com/2023/10/22/z5ioM04t.png)

!!! danger "Sample *then filter* (Blurred Aliasing) 是不可行的！"
### 2.2 实践

#### 1 Increase sampling rate
> 天下武功唯快不破

- 提高采样频率增加了频域中 replicas 之间的距离 => 避免了交叠，解决了走样
- 但事实上屏幕分辨率是固定的（我们不可能通过迫害使一块 1080p 的屏幕直接飞升到 4k）

#### 2 Antialiasing
> 一个典型实现是 Filter out *HIGH* frequencies before filtering

本质上使得 Fourier Contents <u>在 repeat 前变得更窄</u> => 避免了交叠
> 是 chop，不是放缩 orz

在实践中，我们采用 1-pixel-width Box Filter 进行模糊操作：
> 把部分覆盖的颜色平均到整个像素中

![截屏2023-10-22 17.11.22.png](https://img1.imgtp.com/2023/10/22/PL2PUTuF.png)

#### 3 Supersampling (MSAA)
> 只是对模糊操作的一个近似，将 单个像素 划分为 $n \times n$ 的 $n^2$ 宫格

在采样时：

1. 单独考虑 $n^2$ 个格子是否处于三角形内
2. 使用 $\frac{N_{in}}{n^2}$ 作为 整个像素点 的 blur 结果

!!! danger "但是 MSAA 增大了计算量 => 翻了 $n^2$ 倍（不太妙）"

---
其他的抗锯齿方案还有：

- FXAA (Fast Approximate AA)：通过图像匹配后期将锯齿边界替换为曲线
- TAA (Temporal AA)：复用上一帧的感知结构

## 3 Visibility / Occlusion 可见性与遮挡

### 3.1 Painter's Algorithm

!!! tip "Overwrite"
	先画远处的物品，再画近处的物品（然后就被正确覆盖了）

存在的问题：

- 需要对各个面片的 depth 进行排序，复杂度为 $O(n \log n)$

- !!! danger "Can have UNresolvable depth order"
	- 面片之间相互交叠
		<div style="width:100%;display:flex;justify-content:center;">
				<img src="https://img1.imgtp.com/2023/10/22/GDE8hQtC.png" style="width:350px;">
			</div>
	- 其他的例子还有立方体的侧面：左右两面和上下两面谁更 “近” 呢？

### 3.2 Z-Buffering 深度缓存

!!! tip "既然我们没法决定 *triangle* 的远近顺序，那么为什么不以 *pixel* 为基本单位进行考虑呢？"

该算法需要用一个额外（与 image 等大）的 buffer 存储每个 pixel 的 current MIN z-value 值，并最终产出：

- frame buffer 中存储的，具有正确覆盖关系的 color value
- depth buffer 中存储的，每个 pixel 的 depth value

> 为了处理的方便性，我们假设 $depth \gt 0$，并且 $\text{smaller depth} \rightarrow \text{closer}$

```c title="Z-Buffer Algorithm"
// During rasterization
for each triangle T:
	for each pixel (x,y,z) in T:
		if z < zbuffer[x,y]:        // 截至目前 最近 的 pixel
			framebuffer[x,y] = rgb; // 更新该点的 color
			zbuffer[x,y] = z;       // 更新该点的 min-depth
```

时间复杂度：$O(n)$ => 对于 n 个 triangle 而言（不需要排序）
