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

### 变换法线向量

法线是与一个表面的切线垂直的向量。

当一个表面应用了矩阵M变换之后，其切线在应用M变换之后还是该表面的切线，但是法线在应用M变换之后就不一定是该表面的法线了。

由于法线和切线是垂直的，可以得到：

$$
n^Tt=0
$$

将变换后的向量表示为$t_M=Mt$和$n_N=Nn$，我们的目标是找到N使$n_N^Tt_M=0$，这通过一些代数的技巧可以做到。

首先将单位矩阵引入到点乘中，由于$M^{-1}M=I$：

$$
n^Tt=n^TIt=n^TM^{-1}Mt=0.
$$

因为$Mt=t_M$，可以推出：

$$n_N^T=N^TM^{-1}$$

然后对等式两边做一个转置就得到：

$$
n_N=(M^{-1})^Tn
$$

因此$N=(M^{-1})^T$，由于该矩阵可以会改变$n$的长度但并不影响其方向，我们不必关注法线向量的长度，所以可以忽略计算M的行列式：

$$
N=(\frac{adj(M)}{det(M)})^T=(adj(M))^T=cof(M)
$$

## 平移和仿射变换

之前提到的矩阵变换都属于线性变换，在二维中它们都是这样的形式：

$$
\begin{align}
x'=m_{11}x+m_{12}y, \\
y'=m_{21}x+m_{22}y.
\end{align}
$$

这样的变换无法用于移动对象，因为在线性变换中原点$(0,0)$是固定的。为了实现移动或平移，即让对象上所有的点都移动相同的距离，我们需要一个如下形式的变换：

$$
\begin{align}
x'=x+x_t, \\
y'=y+y_t.
\end{align}
$$


让$(x,y)$乘以一个2x2的矩阵是不可能实现的，有一种方法是添加一个额外的平移向量到每一个变换矩阵上，这完全可行，但是当组合两个变换时就没有单纯的线性变换那么简洁了。

相反地，利用一个简单的技巧可能让我们只用一个矩阵同时表示线性变换和平移变换，即将点$(x,y)$表示为一个三维向量$[\begin{array}{c} x & y & 1\end{array}]^T$，用3x3矩阵的形式表示整个变换：

$$
\begin{bmatrix}
m_{11} & m_{12} & x_t \\
m_{21} & m_{22} & y_t \\
0 & 0 & 1 
\end{bmatrix}
$$

利用单个矩阵实现附带平移的线性变换，称之为**仿射变换**。这种通过增加一个额外维度来实现仿射变换的方法叫做**齐次坐标**，齐次坐标不仅使变换变得简洁，还使仿射变换的组合变得简单。

当我们需要变换一个表示方向或偏移的向量（不是位置）时，平移不应该影响该向量的方向，可以把第三个坐标置为零。

当利用第三个坐标对位置向量（1）和方向向量（0）做区分时，变换后的位置仍是位置，变换后的方向仍是方向。

在图形系统中，几乎到处都用齐次坐标表示变换，齐次坐标是图形硬件中实现渲染器设计和操作的基础。

齐次坐标可以被认为仅仅是一种处理平移的聪明做法，但其实还有一种不同的几何解释。关键在于当我们基于z坐标做三维的剪切时，有如下变换：

$$
\begin{bmatrix}
1 & 0 & x_t \\
0 & 1 & y_t \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ z
\end{bmatrix}=
\begin{bmatrix}
x+x_tz \\ y+y_tz \\ z
\end{bmatrix}
$$

可以发现，它已经具备了在x和y上的2D平移形式，但是z在2D当中没有意义。

然后我们做一个关键的决定：对所有的2D位置添加一个值为1的z坐标，现在就可以将平移放到矩阵当中了，例如首先对$(x_t,y_t)$平移再旋转$\phi$的变换矩阵为：

$$
M=\begin{bmatrix}
\cos\phi & -\sin\phi & 0 \\
\sin\phi & \cos\phi & 0 \\
0 & 0 & 1 
\end{bmatrix}
\begin{bmatrix}
1 & 0 & x_t \\
0 & 1 & y_t \\
0 & 0 & 1 
\end{bmatrix}
$$

注意到，二维的旋转矩阵现在变成了“平移列”附带零的3x3矩阵。利用沿z=1剪切来实现平移的形式，我们可以只用一个三维矩阵表示任意的二维剪切、旋转和平移的组合。这种矩阵的最后一行将永远是$(0,0,1)$，因此不必存储它，只需要在相乘矩阵时记得有它就行了。

同样，对于方向向量来说，额外增加的坐标应为零，以保证其不受平移影响。

## 变换矩阵的逆

尽管以代数方式计算逆矩阵总是可行的，但如果我们已经知道变换做的是什么了，那从几何角度分析会更简单。

例如缩放矩阵$scale(s_x,s_y,s_z)$的逆是$scale(1/s_x,1/s_y,1/s_z)$，旋转矩阵的逆就是以相同角度、相反符号的旋转，而平移矩阵的逆就是以相同位移、相反方向的平移。

并且，特定类型的变换矩阵的逆会比较简单。首先是缩放矩阵，它是对角矩阵；其次是旋转矩阵，它是正交矩阵，其转置就是逆。另外，最后一行为$\begin{bmatrix}0&0&0&1\end{bmatrix}$的矩阵的逆的最后一行还是$\begin{bmatrix}0&0&0&1\end{bmatrix}$。

更有趣的是，还可以用SVD来计算逆，因为我们已经知道任何矩阵都可以被分解为旋转、缩放和旋转的点乘，例如：

$$
M=R_1scale(\sigma_1,\sigma_2,\sigma_3)R_2
$$

根据上面推断出的规律可以知道：

$$
M^{-1}=R_2^Tscale(1/\sigma_1,1/\sigma_2,1/\sigma_3)R_1^T
$$

## 坐标变换

之前的所有讨论都是就使用变换矩阵来移动点而言的，我们还可以换一种方式思考矩阵变换，即改变点所在的坐标系统。

改变坐标系统的思想就和编程当中类型转换的思想非常像，当我们想对一个整数加一个浮点数时，要么将整数转换为浮点数再和浮点数相加，要么将浮点数转换为整数和整数相加，以确保类型匹配。

在几何学上，一个坐标系统或坐标系由原点和基（3个向量的集合）组成，在一个原点为$\mathbf{p}$，基为$\{\mathbf{u},\mathbf{v},\mathbf{w}\}$的坐标系中，坐标$(u,v,w)$表示这样一个点：

$$
\mathbf{p}+u\mathbf{u}+v\mathbf{v}+w\mathbf{w}
$$

当我们在计算机中存储这些向量时，它们需要以某个坐标系为基准来表示。因此，我们通常会定义一个“全局”或“世界”坐标，用来描述所有其他的系统。

在2D中我们习惯用$\mathbf{o}$表示原点，用$\mathbf{x}$和$\mathbf{y}$表示右手标准正交基向量，这两个数据永远不会被显式存储，它们是其他所有坐标系统的参考系。

另一个坐标系统可能以$\mathbf{e}$为原点，以$\mathbf{u}$和$\mathbf{v}$为右手标准正交基向量。

![[Pasted image 20250609163515.png]]

对于p点，通常可以写成这样的有序对：

$$
\mathbf{p}=(x_p,y_p)\equiv\mathbf{o}+x_p\mathbf{x}+y_p\mathbf{y}.
$$

类似地，p点也可以表示为：

$$
\mathbf{p}=(u_p,v_p)\equiv\mathbf{e}+u_p\mathbf{u}+v_p\mathbf{v}.
$$

可以利用矩阵机制表达这种相同的关系：

$$
\begin{bmatrix}
x_p \\ y_p \\ 1
\end{bmatrix}
=
\begin{bmatrix}
1 & 0 & x_e \\
0 & 1 & y_e \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x_u & x_v & 0 \\
x_v & y_v & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
u_p \\ v_p \\ 1
\end{bmatrix}
=
\begin{bmatrix}
x_u & x_v & x_e \\
y_u & y_v & y_e \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
u_p \\ v_p \\ 1
\end{bmatrix}
$$

这个变换矩阵其实只是包含了旋转和平移，而是非常容易写下来，只需要把u、v和e放到矩阵的列上，最后一行为$\begin{bmatrix}0&0&1\end{bmatrix}$：

$$
\mathbf{p}_{xy}=
\begin{bmatrix}
u&v&e\\
0&0&1
\end{bmatrix}
\mathbf{p}_{uv}
$$

我们将该矩阵称为$(\mathbf{u},\mathbf{v})$的参考系到规范系变换矩阵。

对于相反方向有：

$$
\begin{bmatrix}
u_p \\ v_p \\ 1
\end{bmatrix}
=
\begin{bmatrix}
x_u & y_u & 0 \\
x_v & y_v & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & -x_e \\
0 & 1 & -y_e \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x_p \\ y_p \\ 1
\end{bmatrix}
$$

将两个矩阵相乘后得到的是参考系到规范系变换矩阵的逆，叫做规范系到参考系变换矩阵：

$$
\mathbf{p}_{uv}=
\begin{bmatrix}
u&v&e\\
0&0&1
\end{bmatrix}^{-1}
\mathbf{p}_{xy}
$$

记住，所有坐标系统都是等效的，这种非对称表象只是源于我们以x和y坐标存储向量的人为约定。

基于(u,v)的规范系到参考系变换矩阵可以用o、x和y表示为：

$$
\mathbf{p}_{uv}=
\begin{bmatrix}
x_{uv}&y_{uv}&o_{uv}\\
0&0&1
\end{bmatrix}^{-1}
\mathbf{p}_{xy}
$$

所有的这些思想在三维中都是完全相似的。



