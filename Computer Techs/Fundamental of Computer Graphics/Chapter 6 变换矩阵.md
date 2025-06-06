线性代数的机制能够用于解释在3D场景下组织物体、通过摄像机观察它们以及将它们放到屏幕上需要的一系列操作。

旋转、平移、缩放和投影这样的几何变换可以通过矩阵乘法来完成。

## 2D线性变换

可以使用一个2x2的矩阵来变换一个2D向量：

$$
\left[
\begin{matrix}
a_{11}&a_{12}\\
a_{21}&a_{22}
\end{matrix}
\right]
\left[
\begin{array}{c}
x \\ y
\end{array}
\right]
=
\left[
\begin{array}{c}
a_{11}x+a_{12}y\\
a_{21}x+a_{22}y
\end{array}
\right]
$$

像这样通过一个简单的矩阵乘法将一个2D向量转变成另一个2D向量的操作，称为**线性变换**。

### 缩放

最基础的变换就是沿着坐标轴进行缩放，它可以改变长度，也可能改变方向。

$$
scale(s_x,s_y)=\left[\begin{matrix}s_x&0\\0&s_y\end{matrix}\right]
$$

其中$s_x$和$s_y$分别代表沿x和y轴的缩放因子。

### 剪切

剪切是将某些东西推向一侧的感觉，就像手里拿着一叠卡片，然后用手推动，越上方的卡片移动的距离越多。

水平和垂直的剪切矩阵如下：

$$
shear-x(s)=\left[\begin{matrix}1&s\\0&1\end{matrix}\right],\quad
shear-y(s)=\left[\begin{matrix}1&0\\s&1\end{matrix}\right].
$$

另一种思考剪切的方式是将它看作对x或y轴的旋转。

例如将垂直轴沿顺时针倾斜某个角度的矩阵为：

$$
\left[\begin{matrix}1&\tan\phi\\0&1\end{matrix}\right]
$$

将水平轴沿逆时针倾斜的矩阵为：

$$
\left[\begin{matrix}1&0\\\tan\phi&1\end{matrix}\right]
$$

### 旋转

假设我们想让向量a逆时针旋转$\phi$得到向量b，a与x轴的夹角为$\alpha$，其长度为$r$，则：

$$
\begin{align}x_a=r\cos\alpha,\\ y_a=r\sin\alpha.\end{align}
$$

向量b的长度也是r，与x轴夹角为$\alpha+\phi$，通过三角形加法恒等式可以得到：

$$
\begin{align}
x_b=r\cos(\alpha+\phi)=r\cos\alpha\cos\phi-r\sin\alpha\sin\phi,\\
y_b=r\sin(\alpha+\phi)=r\sin\alpha\cos\phi+r\cos\alpha\sin\phi.
\end{align}
$$

即：

$$
\begin{align}
x_b=x_a\cos\phi-y_a\sin\phi,\\
y_b=y_a\cos\phi+x_a\sin\phi.
\end{align}
$$

将a变换为b的矩阵形式为：

$$
scale(s_x,s_y)=\left[
\begin{matrix}
\cos\phi & -\sin\phi \\ \sin\phi & \cos\phi
\end{matrix}
\right]
$$

由于旋转矩阵的每一行的范数（向量模长）都为1，并且每一行正交（向量点乘结果为0），所以旋转矩阵是正交矩阵。

### 反射

通过附带一个负的缩放因子的缩放矩阵，我们可以将一个向量沿着某条坐标轴倒影。

$$
reflect-y=\left[\begin{matrix}-1&0\\0&1\end{matrix}\right],\quad
reflect-x=\left[\begin{matrix}1&0\\0&-1\end{matrix}\right].
$$

如果两个缩放因子都为-1，那么它实际上是一个旋转180°的旋转矩阵。

### 变换的组合和分解

对于图形程序来说，对一个物体应用多个变换是常见的，例如我们想先应用一个缩放矩阵，在应用一个旋转矩阵，这可以写成：

$$
v'=(RS)v
$$

这也就意味着，我们可以利用一个单独的相同规模的矩阵来表示对一个向量应用多种变换效果的多个矩阵序列。

另外需要切记，由于矩阵乘法没有交换性，相乘的顺序非常重要，最先被应用的变换矩阵是放在最右边的。

有时撤销一个组合变换是必要的，即把一个变换拆分成多个小的变换。

> 特征值分解

对称矩阵总是能通过特征值分解这样的形式：

$$
A=RSR^T
$$

R是一个正交矩阵，其列向量是特征向量，记为$v_1$和$v_2$。S是一个对角矩阵，其元素为特征值，记为$\lambda_1$和$\lambda_2$。

它其实就是一个多步的几何变换：

1. 将v1和v2旋转到x和y轴（$R^T$）
2. 以$\lambda_1$和$\lambda_2$在x和y上缩放（$S$）
3. 将x和y轴旋转回v1和v2（$R$）

![[Pasted image 20250606152345.png]]

这三个矩阵的效果就是沿着一对坐标轴进行非统一缩放，和轴对齐缩放一样，这对轴是相互垂直的，但它们并非坐标轴，而是矩阵A的特征向量。

这告诉我们对称矩阵其实意味着缩放操作，虽然可能是非统一和非轴对齐的那一种。

> 奇异值分解

对非对称矩阵也可以类似的方式分解，即奇异值分解（SVD）。

$$
A=USV^T
$$

矩阵U的列为左奇异向量，称为$u_i$，矩阵V的列为右奇异向量，称为$v_i$，对角矩阵S的元素为奇异值。

其几何解释和特征值分解非常相似：

1. 将v1和v2旋转到x和y轴（$V^T$）
2. 以$\sigma_1$和$\sigma_2$在x和y上缩放（$S$）
3. 将x和y轴旋转到u1和u2（$U$）

![[Pasted image 20250606154831.png]]

由于SVD在两侧有不同的奇异值，而负的奇异值是不需要的，因为我们可以反转其中一个奇异值的符号，逆转与之关联的奇异向量的方向，然后再次获得相同的变换。

出于这个原因，SVD总是会产生一个所有元素均为正的对角矩阵，但是矩阵U和V并不能保证是纯旋转矩阵，它们可能包含了反射。

不过可以通过检查行列式来区分旋转和反射，当行列式为1时为旋转，为-1时为反射。

总的来说，任何矩阵都能通过SVD分解成一个旋转乘以一个缩放乘以另一个旋转矩阵。只有对称矩阵能通过特征值分解成一个旋转乘以一个缩放乘以旋转的转置，这样的矩阵其实是在任意方向的缩放变换。通过SVD分解对称矩阵将会产生相同的三个矩阵乘积，只不过在代数操作上更为复杂。

> Paeth 旋转矩阵分解

这种方式利用剪切来表示非零旋转：

$$
\left[\begin{matrix}
\cos\phi & -\sin\phi \\ \sin\phi & \cos\phi 
\end{matrix}\right]=
\left[\begin{matrix}
1 & \frac{\cos\phi-1}{\sin\phi} \\ 0 & 1
\end{matrix}\right]
\left[\begin{matrix}
1 & 0 \\ \sin\phi & 1
\end{matrix}\right]
\left[\begin{matrix}
1 & \frac{\cos\phi-1}{\sin\phi} \\ 0 & 1
\end{matrix}\right]
$$

这个特殊的变换对于光栅旋转很有用，因为剪切是一种图像上非常高效的光栅操作，虽然会引入一些锯齿，但不会留下小孔。

如果我们对光栅位置$(i,j)$应用一个水平剪切，得到：

$$
\left[\begin{matrix}1 & s \\ 0 & 1\end{matrix}\right]
\left[\begin{matrix}i \\ j\end{matrix}\right]=
\left[\begin{matrix}i+sj \\ j\end{matrix}\right]
$$

若将$sj$四舍五入到最近的整数，相当于让图像中的每一行向一边移动相同的距离，这使我们能够得到没有缝隙的图像。

## 3D线性变换

3D线性变换是对2D变换的扩展，例如沿笛卡尔坐标轴的旋转矩阵为：

$$
scale(s_x,s_y,s_z)=\left[\begin{matrix}
s_x & 0 & 0 \\
0 & s_y & 0 \\
0 & 0 & s_z
\end{matrix}
\right]
$$

3D中的旋转相对来说更复杂一些：

$$
\begin{align}
rotate-z(\phi)=\left[\begin{matrix}
\cos\phi & -sin\phi & 0 \\
\sin\phi & \cos\phi & 0 \\
0 & 0 & 1
\end{matrix}\right]\\
rotate-x(\phi)=\left[\begin{matrix}
1 & 0 & 0 \\
0 & \cos\phi & -sin\phi \\
0 & \sin\phi & \cos\phi
\end{matrix}\right] \\
rotate-y(\phi)=\left[\begin{matrix}
\cos\phi & 0 & sin\phi \\
0 & 1 & 0 \\
-\sin\phi & 0 & \cos\phi
\end{matrix}\right]
\end{align}
$$

沿x轴剪切的变换矩阵为：

$$
shear-x(d_y,d_z)=\left[\begin{matrix}
1 & d_y & d_z \\
0 & 1 & 0 \\
0 & 0 & 1
\end{matrix}\right]
$$

### 任意3D旋转

3D旋转矩阵是正交的，在几何上这意味着矩阵的3行（列）向量是相互正交的单位向量，这样的旋转矩阵有无数多个，统一表示为：

$$
R_{uvw}=
\left[\begin{matrix}
x_u & y_u & z_u \\
x_v & y_v & z_v \\
x_w & y_w & z_w
\end{matrix}\right]
$$

通过将该矩阵分别应用到u，v和w上，我们可以推断出旋转矩阵的一些性质。

$$
R_{uvw}u=
\left[\begin{matrix}
x_u & y_u & z_u \\
x_v & y_v & z_v \\
x_w & y_w & z_w
\end{matrix}\right]
\left[\begin{matrix}
x_u \\ y_u \\ z_u
\end{matrix}\right]=
\left[\begin{matrix}
x_u x_u + y_u y_u + z_u z_u \\
x_v x_u + y_v y_u + z_v z_u \\
x_w x_u + y_w y_u + z_w z_u
\end{matrix}\right]=
\left[\begin{matrix}
u\cdot u \\ v\cdot u \\ w\cdot u
\end{matrix}\right]=
\left[\begin{matrix}
1 \\ 0 \\ 0
\end{matrix}\right]=x
$$

类似地，$R_{uvw}v=y$，$R_{uvw}w=z$。如果$R_{uvw}$是一个列正交的矩阵，那么$R_{uvw}^T$则是一个行正交的矩阵。

对于变换矩阵来说，代数上的逆也是几何上的逆，因此：

$$
R_{uvw}^Ty=
\left[\begin{matrix}
x_u & x_v & x_w \\
y_u & y_v & y_w \\
z_u & z_v & z_w
\end{matrix}\right]
\left[\begin{matrix}
0 \\ 1 \\ 0
\end{matrix}\right]=
\left[\begin{matrix}
x_v \\ y_v \\ z_v
\end{matrix}\right]=v
$$

于是，如果我们想绕任意轴a旋转，可以用$w=a$构建一个正交基，将其旋转到标准正交基$xyz$，绕z轴旋转，然后再从标准正交基旋转会$uvw$基。用矩阵形式表示为：

$$
R_{uvw}^TR_x(\phi)R_{uvw}
$$

如果已知一个旋转矩阵，可以计算它的特征向量和特征值得到旋转轴和旋转角。

