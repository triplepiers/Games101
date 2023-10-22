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

### 1 Blinn-Phong Reflectance Model

单一光源对物体表面照明的效果可以分为三个部分：

- 高光 Specular Highlights
- 漫反射 Diffuse Reflection
- 环境光照 Ambient Lighting

#### Diffuse Reflection
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

- $k_d$ 为 diffuse coefficient，表示 color 
  
    => shading point 的颜色决定了表面会 *吸收* 多少能量
  
- $r$ 为点光源到 shading point 的距离 
  
    => $\frac{I}{r^2}$ 为该光源到达 shading point 处的能量总量
  
- $\max\left(0,\vec{n}\cdot\vec{l}\right) = \max\left(0,cos\theta\right)$ 表示 shading point 实际接受能量占比
  
	=> 加一个 $max$ 是因为负的夹角没有意义（光源在物体内部）

!!! tip "由于漫反射向各个方向散布的能量大小是一致的，所以观测结果与 *观测方向无关*"

## 2 Graphics Pipeline