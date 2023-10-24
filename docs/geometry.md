

## 1 Representations

### Implicit 隐式

(三维)点坐标满足的关系： $f(x,y,z) = 0$

> 比如球体方程 $x^2 + y ^2 + z^2 = 1$
	
我们很难看着方程说他 *是什么*，但很容易判断点 *在不在/在内外* 函数规定的几何体上

- Algebra Surface：参数方程（不直观）

- Constructive Solid Geometry
	
	对基本几何体进行 Union / Intersection / Difference 操作
		
- Signed Distance Functions (SDF)

	描述空间中任何一点到给定几何体的最小距离（可以 $\lt 0$）

	- 通过对两个几何体的 SDF 进行 blend（类似于场叠加）操作，就可以得到平滑的融合过程
	- 找到所有 $dist = 0$ 的点就可以恢复出立体了 => 所有 $dist$ 相等的点属于同一层平面

- Fractals 分型

### Explicit 显式

给出所有满足条件的(三维)点坐标 / 参数映射(Parameter Mapping)
> 后者给你满足条件的 $(u,v)$ 集合，并定义 $(u,v) \rightarrow (x,y,z)$ 的映射关系

难以判断给定点在几何体 *内外*

- Point Cloud 点云：list of points $(x,y,z)$

	理论上可以表示任何几何体，但需要巨大的 dataset（$\gt$ 1 point / pixel）

	一般会把点变成 polygon mesh

- Polygon Mesh 多边形面片：三角形和四边形用的比较多

	- `.obj` 文件格式：是一个文本文件
	  
	    分别记录了 vertices, normals, texture coordinates 以及 数据之间的关系（哪三个点构成三角形，用什么材质 ...）

- Subdivision, NURBS

## 2 Bezier Curves 贝赛尔曲线

下面是贝塞尔曲线具备的一些性质：

- 固定的起止位置：$b(0) = b_0,\ b(1)=b_n$
- 固定的起止方向（系数针对 4 point的情况）：$b'(0) = 3(b_1-b_0),\ b'(1)=3(b_3-b_2)$
- 对整个曲线仿射变换 <=> 对控制点仿射变换后重新绘制
- Convex Hull Property：最终得到的贝塞尔曲线一定在控制点形成的 *凸包内*
#### de Castelijau Algorithm

考虑三个控制点的情况（生成的是 二次 quadratic 贝塞尔曲线）

- 曲线开始于 $b_0$，结束于 $b_2$

- 假设曲线是在 $t \in [0,1]$ 在空间中自 $b_0$ 漫游至 $b_2$ 的过程，对于时刻 $t$：

	- 在两个线段 $b_0b_1,\ b_1b_2$ 上同时画出 $t$ 等分点 $b_0^1,\ b_1^1$
	
	> 	满足 $\frac{b_0b_0^1}{b_0^1b_1} = \frac{b_1b_1^1}{b_1^1b_2} = \frac{t}{1-t}$
		
	- 寻找线段 $b_0^1b_1^1$ 的 $t$ 等分点 $b_0^2$ => 即位曲线在 $t$ 时刻所处的位置

!!! info "四个控制点生成曲线的情况"
	- 不过是多做几次线性插值
	- 使用 $k$ 个控制点时，需要 $\sum_{i=1}^{k-1} i$ 次线性插值确定一个点
	
	![截屏2023-10-24 13.22.55.png](https://img1.imgtp.com/2023/10/24/cwkmUD9M.png)

#### Algebraic Formula

de Castelijau Algorithm 事实上生成了一系列 *金字塔状* 排布的 coefficient

![截屏2023-10-24 13.27.01.png](https://img1.imgtp.com/2023/10/24/UkGYSKl1.png)

考虑三个控制点的情况， $t$ 时刻位置 $b_0^2$ 可以表示为三个点的线性组合：

$$
\left\{
	\begin{align*}
		b_0^1(t) &= (1-t)b_0 + tb_1 \\
		b_1^1(t) &= (1-t)b_1 + tb_2 \\
		b_0^2(t) &= (1-t)b_0^1 + tb_1^1
	\end{align*}
\right. \Rightarrow b_0^2(t) = (1-t)^2b_0 + 2t(1-t)b_1 + t^2b_2
$$

考虑 General Case of order $n$（ $n+1$ 个控制点）：

$$
b^n(t)=b_0^n(t)= \sum_{j=0}^n B_j^n(t) \cdot b_j
$$
其中：

- $b_j$ 是 $n+1$ 个控制点
- $B_j^n(t)$ 是 Bernstein Polynomial（其实就是二项式定理）

	$$
	B_j^n(t) = \left(
		\begin{array}{1}
		n\\j
		\end{array}
	\right) t^j(1-t)^{n-j}
	$$

#### Piecewise 逐段
!!! question "Higher-Order Bezier Curve 的效果并不好"
	使用 100 个点定义一条曲线也没法让他变得特别扭曲

!!! tip "少量多次：将多条 Low-Order 贝塞尔曲线首位相连"
	Piecewise Bezier Curve 就是逐段定义低阶贝塞尔曲线并对其进行拼接的技术（钢笔工具）
	
	- 使用四个控制点定义一段曲线（两个短柄）
	- 保证光滑过渡（将两个短柄移动至 共线 · 等长 的状态 => 导数连续）

- 两段 Bezier Curve $[k,k+1]\ [k+1,+2]$ 之间的连续性

	假设两段曲线的控制点分别为 $[a_0,a_n],\ [b_0,b_n]$

	- $C^0$ Continuity：上一段的终点是下一段的起点 $a_n == b_0$

		连是连上了，但可能是折线
	
	- $C^1$ Continuity：切线(一阶导)连续 $a_n==b_0=\frac{1}{2}(a_{n-1}+b_1)$

## 3 Surfaces

### 3.1 Bezier Surface

使用了 4 x 4 个控制点，在 X,Y 方向上分别运用贝塞尔曲线确定坐标：

- 四组控制点各自形成了 4 条 贝塞尔曲线
- 在 t 时刻，我们四条曲线与平面 $x = t$ 存在四个交点，构成了另一条贝塞尔曲线

	=> $t\in[0,1]$ 上的各条曲线相连便得到了曲面

### 3.2 Mesh Operations 
> 三角面片涉及的几何处理

- Subdivision 细分：使用更多的面片可以得到更贴合的模型 (upsampling)
- Simplification 简化：更少的面可以节省存储空间 (downsampling)
- Regularization 正规化：把面片变换的接近正三角形 (modify distribution to improve quality)

#### Subdivision 细分

首先创造更多的三角形，然后改变他们的位置让模型更平滑

##### Loop Subdivision

- 连接三角形三边的中点，你就能得到四个小三角形
- 对新旧顶点应用不同的位置更新规则：
  
	  | Type | Rule | Pic |
	  | :--: | :---|:--:|
	  | New Vertex| 白色点调整至 $\frac{3}{8}(A+B) + \frac{1}{8}(C+D)$| <img src="https://img1.imgtp.com/2023/10/24/XmS5eESA.png" style="height:200px;">|
	  |Old Vertex |白色点调整至 $(1-n*u) \cdot original\_pos + u \cdot \sum(neighbor\_pos)$ </br> - $n$ 为 vertex degree </br> - 若 n = 3，则 u = $\frac{3}{16}$，否则 u = $\frac{3}{8n}$| <img src="https://img1.imgtp.com/2023/10/24/cVXLWn6c.png" style="height:200px;">|

##### Catmull-Clark Subdivision
> 适用于 General Mesh 的情况（面片并不全是三角形）

- 定义
	- Non-quad face 三角形面片
	- Extraordinary Vertex 奇异点：$\text{degree} \neq 4$ 的顶点

- 过程

	- 取每条边的中点 & 每个面的重心（中点）
	- 连接这些点

!!! question "经过一次划分后，现在有多少个奇异点？"

	$n' = n + n_{triangle}$ 
	
	- 原本的奇异点仍然是奇异点(degree = 5)
	- 三角心重心是新增的奇异点(degree = 3)

!!! info "此时已经 *没有* Non-quad Face 了"
	所有的三角形面片都被划分为了 3 * 四边形面片
	
	=> 之后的所有细分操作将 *不再增加* 奇异点的数量

- 顶点位置更新策略

	- Face Point：位于面片中间的点 $f$

		记面片的四个顶点为 $(v_1,v_4)$，我们令 $f = \frac{v_1+v_2+v_3+v_4}{4}$

	- Edge Point：面片边缘上的点 $e$

		考虑该边的两个端点 $v_1,\ v_2$ 以及相邻两个面片的 face points $f_1,\ f_2$

		我们令 $e = \frac{v_1+v_2+f_1+f_2}{4}$

	- (Old) Vertex Point
	  
	  ![截屏2023-10-24 15.23.50.png](https://img1.imgtp.com/2023/10/24/UOZiegBg.png)
#### Simplification

使用 边坍缩 Edge Collapsing 进行简化

![截屏2023-10-24 15.31.04.png](https://img1.imgtp.com/2023/10/24/KqDgMh9N.png)

##### Quadric Error Metrics 二次误差度量

![截屏2023-10-24 15.32.49.png](https://img1.imgtp.com/2023/10/24/FBBSKhgC.png)
> 我们尝试使用蓝色的三角形来取代原本的灰色多边形 => 顶点放在哪里？

- 左侧是将顶点置于上方三个顶点的 *local average* 的结果（太平啦）
- 右侧是将点置于二次误差最小化位置的结果

	到四条虚线距离平方和最小的位置

!!! question "如何选择需要坍缩的边？"
	Iteratively：每次坍缩二次误差最小的边
	
	- 坍缩一条边会引起边上其他边的二次误差变化 => 需要 Update

		使用 Heap / Priority Queue 作为存储结构（每次取最小 + 更新）
	
	- 这是一种贪心算法（不能确保全局最优）