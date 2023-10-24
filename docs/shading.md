着色 (Shading) is the process of  <u>applying a material</u> to an object.

#### Shading is Local
!!! warning "Shading $\neq$ Shadow（着色不会生成阴影）"

我们以物体表面上的一个 Shading Point 为考虑着色结果的基本单位：

- （物体表面可以是曲面）在极小范围内，我们认为 Shading Point 位于 *平面* 上
- 三个(单位)向量：

	1. 平面法线 Surface Normal $\vec{n}$：垂直于平面向上
	2. 观测方向 Viewer Direction $\vec{v}$：shading point 与 相机 的连线方向
	3. 光照方向 Light Direction $\vec{l}$：（每个）点光源 与 shading point 的连线方向

- 此外，我们还会定义一些列的 surface params：color, shininess, ..
## 1 Illumination & Shading

### 1.1 Blinn-Phong Reflectance Model

单一光源对物体表面照明的效果可以分为三个部分：

- 高光 Specular Highlights
- 漫反射 Diffuse Reflection
- 环境光照 Ambient Lighting

将以上三项相加，即可得到 Blinn-Phong 反射模型的结果，即：

$$
\begin{align*}
L &= L_a + L_d + L_s \\
&= k_aI_a + k_d\frac{I}{r^2}\max(0,\vec{n}\cdot\vec{l})+k_s\frac{I}{r^2}\max(0,\vec{n}\cdot\vec{h})^p
\end{align*}
$$
#### 1.1.1 Specular Term(Bliin-Phong)

- 比较 *光滑* 的物品才具有高光 => 接近于 *镜面反射*

- 当 观察角度 接近于 (镜面)反射角度 时，才能观测到高光 <=> 法线近似于角平分线

	记 *半程向量(角平分线)* 方向为 $\vec{h}$，我们可以列出：
	
	$$
	\vec{h} = \text{bisector}(\vec{v},\vec{l}) = \frac{\vec{v}+\vec{l}}{\|\vec{v}+\vec{l}\|}
	$$
	
	> 当 $\vec{h} \rightarrow \vec{n} \Rightarrow \vec{v} \rightarrow \vec{R}$，即角平分线接近法线时，观测方向也接近出射方向 $\vec{R}$

记 $\vec{n}$ 与 $\vec{h}$ 之间的夹角大小为 $\alpha$ ，我们可以列出 specularly reflected light $L_s$ 的公式：

$$
L_s = k_s \left(\frac{I}{r^2}\right)\max\left(0,cos\alpha\right)^p 
= k_s \left(\frac{I}{r^2}\right)\max\left(0,\vec{n} \cdot \vec{h}\right)^p
$$
其中：

- $k_s$ 为 specular coefficient
- 由于 $cos$ 函数的变化过于平缓，我们使用 $p$ 次方实现骤降以贴合实际（一般取 $[100,200]$）

> Blinn-Phong 模型简化了镜面反射中被吸收的部分能量
#### 1.1.2 Diffuse Reflection
> 光线被 *均匀* 的反射至各个方向
##### Lambert's cosine law

!!! question "不同的入射角会导致不同的结构，how much light is received ?"

- 我们认为 <u>单位面积</u> 的入射光具有相同的能量，记为 $P$
- 另外，记 入射方向 $\vec{l}$ 与 法线方向 $\vec{n}$ 的夹角为 $\theta = \vec{l} \cdot \vec{n}$ 

那么，实际有效的接受面积为 $S' = S \cdot cos\theta$（投影至垂直入射方向的面积），接受的总能量就是 $PS'=PS \cdot cos\theta$

![截屏2023-10-23 00.45.02.png](https://img1.imgtp.com/2023/10/23/wQunzd2X.png)

##### Light Falloff

!!! question "点光源在距其特定距离处提供了多少能量？"

- 我们假设以点光源为球心的 *球壳* 表面均具有等量的能量，且半径为 1 的球壳上的能量密度为 $I$
- 在半径为 $r$ 处的球壳面积将是为 1 处的 $r^2$ 倍（球表面积为 $4\pi r^2$）
- 由于两个球壳携带的能量总量相等，我们可以列出方程：

	$$
	E_1 = 4 \pi 1^2 \cdot I = E_r = 4 \pi r^2 \cdot I' \Rightarrow I' = \frac{I}{r^2}
	 $$

##### Lambertian (Diffuse) Shading

综合上面两个结论，我们可以得出 diffusely reflected light $L_d$ 的公式：

$$
L_d = k_d\left(\frac{I}{r^2}\right)\max\left(0,\vec{n}\cdot\vec{l}\right)
$$

其中：

- $k_d$ 为 diffuse coefficient，间接反映了物体的 color 
  
    => shading point 的颜色决定了表面会 *吸收* 多少能量
  
- $r$ 为点光源到 shading point 的距离 
  
    => $\frac{I}{r^2}$ 为该光源到达 shading point 处的能量总量
  
- $\max\left(0,\vec{n}\cdot\vec{l}\right) = \max\left(0,cos\theta\right)$ 表示 shading point 实际接受能量占比
  
	=> 加一个 $max$ 是因为负的夹角没有意义（光源在物体内部）

!!! tip "由于漫反射向各个方向散布的能量大小是一致的，所以观测结果与 *观测方向无关*"

#### 1.1.3 Ambient Term

我们认为 **所有 shading point 接受的环境光照都是<u>相同的</u>**，记为 $I_a$。

对于不同颜色的 shading point，我们可以列出 reflected ambient light $L_a$ 的方程：

$$
L_a = k_a \cdot I_a,\ k_a\text{ is ambient coefficient}
$$
### 1.2 Shading Frequency 着色频率

![截屏2023-10-23 20.42.16.png](https://img1.imgtp.com/2023/10/23/RxJSosKX.png)

上图中自左向右分别为：

- Flat Shading：以 *面* 为单位进行 shading
- Gouraud Shading：对于 *面的每个顶点* 进行 shading，三角形内部进行插值
- Phong Shading：以 *像素* 为单位进行 shading

但 Flat Shading 的效果不一定很差 => 当面片数量较多时，我们运用这种简单的方法也能取得较好的结果

> 但是 Phong Shading 的效果一定有保证 hhh

![截屏2023-10-23 20.50.18.png](https://img1.imgtp.com/2023/10/23/qxx8XoTz.png)

#### Per-Vertex Normal Vectors
> 顶点法线方向计算

- Best to get vertex normals from the *underlying geometry*

	当我们用三角面片去拟合一个球体时，我们可以忽略三角形直接看球体方程

- Otherwise has to *infer* vertex normals from triangle faces

	*Average surrounding* face normals：相邻面片法线的平均值

	$$
	N_v =\frac{\sum_i N_i}{\|\sum_iN_i\|}
	$$

	优化：考虑面片面积的贡献 => 按面积加权平均

#### Per-Pixel Normal Vectors
> 在已知 vertex 法线的前提下，得到三角形内部平滑过渡的逐像素法线

使用 重心坐标 barycentric interpolation 对 vertex normals 进行插值

## 2 Graphics Pipeline
> 描述如何处理场景并形成渲染图的过程（一系列操作）

![截屏2023-10-23 21.51.59.png](https://img1.imgtp.com/2023/10/23/n7dOodCG.png)

### Shader Programs

我们可以自定义 Shader 规定 GPU 在 Vertex Processing / Fragment Processing 部分的着色行为（逐 顶点 / 像素 着色）

- Shader 会根据定义在 vertex / pixel 上执行 *一次*，是一个通用程序

```c++ title="GLSL Shader - 如何计算像素的颜色"
uniform sampler2D myTexture;
uniform vec3 lightDir;
varying vec2 uv;      // 由 rasterizer 生成：per fragment
varying vec3 norm;    // 由 rasterizer 生成：per fragment

void diffuseShader() {
	vec3 kd;
	kd = texture2d(myTexture, uv);
	kd *= clamp(dot(-lightDir, norm), 0.0, 1.0); // Lambertian Model
	gl_FragColor = vec4(kd, 1.0);                // 返回 color
}
```

## 3 Texture Mapping 纹理映射
> 纹理：定义任何一个点具备的不同属性

!!! tip "Surfaces are 2D -> 把三维物体的表皮展开为平面"

- （贴图）我们将纹理作为一张平面图，只要描述 3D Model 表面 与 2D Texture 的映射关系即可


- 我们使用坐标 $(u,v)$ 来描述每一个三角形顶点在 texture 上的坐标
> 	整个 U-V 坐标系一般处于 $[0,1]^2$ 范围内

	![截屏2023-10-23 22.22.00.png](https://img1.imgtp.com/2023/10/23/6tMpQy10.png)

!!! tip "（贴瓷砖）Textures can be used MULTIPLE times ! "

### 3.1 Barycentric Coordinates 重心坐标
> 如何在三角形内部对 *任意属性* 进行平滑插值？


- 坐标系定义

	平面 $ABC$ 内 *任意一点* 均可表示为三角形顶点 $A,B,C$ 坐标的 <u>线性组合</u> $(\alpha,\beta,\gamma)$
	
	![截屏2023-10-23 22.31.45.png](https://img1.imgtp.com/2023/10/23/IAtXnfOF.png)
> 	因为 $\alpha + \beta + \gamma = 1$ ，所以实际上只有两个未知数（对应二维平面）

- 通过面积比求解 $(\alpha,\beta,\gamma)$

	![截屏2023-10-23 22.37.37.png](https://img1.imgtp.com/2023/10/23/ABZCe2cM.png)
	显然，三角形重心 $O$ 的重心坐标为 $(\frac{1}{3},\frac{1}{3},\frac{1}{3})$

- 一般表达式

	$$
	\begin{align*}
	\alpha &= \frac{-(x-x_B)(y_C-y_B) + (y-y_B)(x_C-x_B)}{-(x_A-x_B)(y_C-y_B) + (y_A-y_B)(x_C-x_B)} \\
	\beta &= \frac{-(x-x_C)(y_A-y_C) + (y-y_C)(x_A-x_C)}{-(x_B-x_C)(y_A-y_C) + (y_B-y_C)(x_A-x_C)}\\
	\gamma & = 1 - \alpha - \beta
	\end{align*}
	$$

!!! tip "使用重心坐标对三个顶点的属性进行线性组合，即可得到平滑的插值"

!!! danger "Projection 可能导致重心坐标变化"
### 3.2 Texture Magnification

!!! question "What if the texture is too SMALL ?"

这回导致我们返回 texture 中查询的坐标 *不是整数*，尝试通过一些简单的方式解决：

- Nearest：简单的四舍五入
- Bilinear 双线性插值：考虑边上的 4 个 texel，在 水平/竖直 方向做两轮插值

	![截屏2023-10-23 22.56.07.png](https://img1.imgtp.com/2023/10/23/VeCseWYe.png)
- Bicubic 双三次插值：考虑边上的 16 个点

### 3.3 Texture Queries

!!! question "What if the texture is too LARGE ?"
> 事实上纹理大了的效果会更抽象（近处锯齿 + 远处摩尔纹）

- *较近* 的像素在 texture space 中对应了 *较小* 的一块区域，此时使用区域 *中心* 对色彩进行近似没有太大的问题
- 但对于 *较远* 像素来说，使用局部概括整体显然是不够妥当的

> => Supersampling 可以解决走样问题，但是开销太大了 orz

!!! tip "采样引发了走样，要不就 *不采样* 了？"

不如从 Point Query 转变为 Range(avg) Query 吧！
#### Mipmap
> Range Query 应该支持对 *任意大小* 区域的快速（近似）查询

Mipmap 仅支持对*方形区域*进行查询：

- 预先算好一串的 Range Query 结果（每次分辨率折半）
- 只比原图增加了 $\frac{1}{3}$ 的存储开销

![截屏2023-10-23 23.14.14.png](https://img1.imgtp.com/2023/10/23/9OrfaZWQ.png)

- Range Size 的计算：

	1. 把目标像素周围的几个像素一起映射到 texture space
	2. 根据 右侧 & 上侧 投影结果的 *最大距离* 作为方形区域边长 $L$，并决定 range size
	3. 查询第 $log(L)$ 张 mipmap => 方形区域在这一层上对应 1 pixel
	
	![截屏2023-10-23 23.19.19.png](https://img1.imgtp.com/2023/10/23/c7IsjVbk.png)

- Trilinear Interpolation 三线性插值
  
    >  使用插值解决对非整数层的查询)

	当我们尝试查询第 $N.x$ 层时：
	
	1. 查询第 $N,\ N+1$ 层，并分别对结果进行双线性插值
	2. 对两层的插值结果再做一次插值

- Anisotropic Filtering 各向异性过滤 

	使用 Mipmap 会在 *远处* 的像素会产生 Overblur

	- Mipmap 忽视了长宽比并非 1:1 的压缩情况，只考虑了 squre
	- 然而 1 pixel 映射得到的区域往往不是方正的，甚至可能是一个倾斜的矩形

	使用 各项异性过滤 可以部分解决 Mipmap 的部分问题 => 允许查询矩形区域（但不能查询倾斜区域）

	开销是原图的 $3$ 倍

- EWA Filtering

	将区域拆分成若干个椭圆，使得查询区域更好的覆盖映射区域

	但多次查询会导致更大的开销 orz

## 4 Applications

In modern GPUs, `texture` = Memory + Range Query(Filtering)
> 纹理是内存上的一块数据，而不局限于一张“图片”

我们可以用它来表示更多的东西 ...

### Environment Map 环境光照

也可以叫"环境贴图"，用于描述来自各个方向的光线信息

- 我们假设环境光均来自于 *无限远处* => 只要记录方向就 ok 啦

- Spherical Map：信息可以记录在 *球面* 上
	- 拉开成一张矩形图 => 扭曲的一张图
	- Cube Map：拉开成一个(外接) ，对应点与球心连线和方盒相交处 => 不那么扭的六张图，但需要判断点在哪一个面上

### Bump Mapping 法线 贴图

我们可以用纹理定义点与平面的相对高度（like 橘子皮）

在原本的几何体上增加很多假的法线，从而影响着色结果
> 冬枣不会变成核桃，只是 “看上去” 皱巴巴

- 考虑 平面(flatland) 的情况：

	- 原始的表面法线 $n(p) = (0,1)$ ，垂直表面向上
	- 假设贴图函数为 $h(x)$，我们认为此处的差分为 $dp = c \cdot [h(p+1) - h(p)]$ => 该点的 *切线* 方向为 $(1, dp)$
	- 新的法线于切线垂直，为 $\text{normalized}(-dp, 1)$

- 考虑 3D 的情况：

	- 原始法线为 $(0,0,1)$
	- 纹理有 $(u,v)$ 两个维度，此处也对应了两个差分

		$$
		\begin{align*}
			\frac{dp}{du} &= c_1 \cdot [h(u+1) - h(u)] \\
			\frac{dp}{dv} &= c_2 \cdot [h(v+1) - h(v)]
		\end{align*}
		$$
		
	- 新的法线为 $\text{normalized}(-\frac{dp}{du},-\frac{dp}{dv})$

> 实际上原始的表面法线并不一定是 $(0,0,1)$，但我们可以通过定义 local coordinate 来解决
> 
> 虽然需要进行坐标变换 orz

!!! info "Bump Mapping 并没有真正移动顶点（只是计算了假的法线）</br> Displacement Mapping 将真正挪动顶点并改变几何体（更加真实）"

## 5 Shadow Mapping

Shadow Mapping 被应用在 Image Space（生成阴影不需要知道场景的几何信息）

- 缺陷
	- 只能用于之所 Hard Shadows
	- Shadow Map 的分辨率问题可能导致走样
	- 在比较深度时，浮点数的精度问题可能导致误差

!!! info "Hard Shadow & Soft Shadow"

	- Hard Shadow：非黑即白
	- Soft Shadow：具有透明度
		- Umbra 本影区域全黑
		- Penumbra 半影区域具有不同的透明度（部分可见）

- 生成点光源下的硬阴影
> 	事实上只有 *拥有一定体积的光源* 才能产生软阴影，点光源只能产生硬阴影

	!!! tip "*不在阴影* 中的点必须同时被 light & camera 可视"
	
	1. Render from Light：点是否被光源可视？
	
		使用 Depth Image 记录从光源出发所有 *可视* 点的最大深度（不进行着色）
	
	2. Render from Camera：从相机出发进行观测
	
		从相机出发，将可视点投影至 *光源平面* 并与 Depth Image 记录结果进行比较
	
		- 深度一致：该点可见
		- 深度更大：该点不 (被光源) 可见