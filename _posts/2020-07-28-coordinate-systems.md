---
title: 坐标系和变换矩阵
tags: 
- graphics
- vulkan
---

# View Matrix

将世界坐标转换到相机的观测坐标系，如下图所示：

![image.png]({{ site.url }}/assets/images/view_transform.png)

<!--more-->
坐标系\\( x,y,z \\) 表示世界坐标系，而右下角的\\( x',y',z' \\)则表示相机的坐标系，相机坐标系xyz分别对应相机的右、上、后等方向。

观测矩阵就是将世界坐标系中的点\\( p(a,b,c,d) \\)变换到观测坐标系中的\\( p'(a',b',c',d') \\)。

$$ \mathbf{p'} = \mathbf{T}\mathbf{p} $$

令\\( \mathbf{f},\mathbf{s},\mathbf{u} \\)分别表示相机在世界坐标系下的前、右和上三个方向的单位向量，\\( \mathbf{L} \\)表示相机在世界坐标系的位置， \\( <\mathbf{s},\mathbf{u},-\mathbf{f}> \\)则组成了我们想要的坐标系，表示成矩阵就是：

$$ \mathbf{T} = \begin{bmatrix}
\mathbf{s}_{x} & \mathbf{u}_{x} & -\mathbf{f}_{x} & 0 \\\\
\mathbf{s}_{y} & \mathbf{u}_{y} & -\mathbf{f}_{y} & 0 \\\\
\mathbf{s}_{z} & \mathbf{u}_{z} & -\mathbf{f}_{z} & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

再加上相机的位置所需要的偏移（变换矩阵的最后一列表示的是位移），则最终的变换矩阵是：

$$ \mathbf{T} = \begin{bmatrix}
\mathbf{s}_{x} & \mathbf{u}_{x} & -\mathbf{f}_{x} & -\mathbf{L} \cdot \mathbf{s} \\\\
\mathbf{s}_{y} & \mathbf{u}_{y} & -\mathbf{f}_{y} & -\mathbf{L} \cdot \mathbf{u} \\\\
\mathbf{s}_{z} & \mathbf{u}_{z} & -\mathbf{f}_{z} & -\mathbf{L} \cdot (-\mathbf{f}) \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

其中\\( \mathbf{L}'=(\mathbf{L} \cdot \mathbf{s},\mathbf{L} \cdot \mathbf{u},\mathbf{L} \cdot (\mathbf{-f})) \\)是相机位置\\( \mathbf{L} \\)在相机坐标系内的坐标，因为原世界坐标系原点在相机的观测坐标系内的位置就是沿着\\( -\mathbf{L} \\)方向的，所以最终矩阵中的位移为\\( -\mathbf{L}' \\)。

# Projection Matrix

投影矩阵将相机观测坐标系下的点映射到裁剪坐标系下的点，如下图所示：

![image.png]({{ site.url }}/assets/images/proj_transform.png)

坐标系\\( x,y,z \\) 表示相机观测坐标系，而左下角的\\( x',y',z' \\)则表示裁剪空间的坐标系，不同的图形API使用的裁剪空间可能不同，如Vulkan使用的就是图中的右手坐标系，OpenGL则是与图中的Y轴和Z轴相反的右手坐标系，DirectX是与图中的Z轴相反的左手坐标系。之后以Vulkan的裁剪空间坐标系为准。

裁剪空间内点坐标取值在\\( [-1,1] \times [-1,1] \times [0,1] \\)范围内的将输出到屏幕（这里依旧是Vulkan的标准，DirectX与此相同，OpenGL则不相同）。

有两种常用的投影矩阵：正射投影和透视投影。

## 正射投影

正射投影将裁剪坐标系下在\\( [l,r] \times [b,t] \times [n,f] \\)范围内的点映射到\\( [-1,1] \times [-1,1] \times [0,1] \\)，其中\\( l \\)表示裁剪盒最左边的x坐标，\\( r \\)表示裁剪盒最右边的x坐标，\\( b \\)表示裁剪盒最底部的y坐标，\\( t \\)表示裁剪盒最上边的y坐标，\\( n \\)表示裁剪盒最前边的z坐标，\\( f \\)表示裁剪盒最后边的z坐标。

首先考虑裁剪盒以z轴为中心的正射投影，如下图所示为Y-Z平面图：

![image.png]({{ site.url }}/assets/images/ortho_proj.png)

其中\\( w \\)为裁剪盒的宽度，\\( h \\)为裁剪盒的高度，容易计算得到投影矩阵为：

$$ \mathbf{T} = \begin{bmatrix}
\frac{2}{w} & 0  & 0  & 0 \\\\
0 & \frac{2}{h} & 0 & 0 \\\\
0 & 0 & \frac{1}{f-n} & \frac{-n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

而考虑到观测坐标系到裁剪坐标系的变换矩阵为:

$$ \mathbf{S} =  \begin{bmatrix}
1 & 0  & 0  & 0 \\\\
0 & -1 & 0  & 0 \\\\
0 & 0  & -1 & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

则最终的投影矩阵为：

$$ \mathbf{T'} = \mathbf{T}\mathbf{S} 
= \begin{bmatrix}
\frac{2}{w} & 0  & 0  & 0 \\\\
0 & \frac{2}{h} & 0 & 0 \\\\
0 & 0 & \frac{1}{f-n} & \frac{-n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} \begin{bmatrix}
1 & 0  & 0  & 0 \\\\
0 & -1 & 0  & 0 \\\\
0 & 0  & -1 & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} 
= \begin{bmatrix}
\frac{2}{w} & 0  & 0  & 0 \\\\
0 & -\frac{2}{h} & 0 & 0 \\\\
0 & 0 & -\frac{1}{f-n} & -\frac{-n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

再考虑更一般的正射投影，只需增加一个平移项将裁剪盒移到Z轴中心即可：

$$ \mathbf{T'} = \mathbf{T}\mathbf{S} 
= \begin{bmatrix}
\frac{2}{r-l} & 0  & 0  & 0 \\\\
0 & \frac{2}{t-b} & 0 & 0 \\\\
0 & 0 & \frac{1}{f-n} & \frac{-n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} \begin{bmatrix}
1 & 0  & 0  & -\frac{l+r}{2} \\\\
0 & -1 & 0  & -\frac{b+t}{2} \\\\
0 & 0  & -1 & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} 
= \begin{bmatrix}
\frac{2}{w} & 0  & 0  & -\frac{l+r}{r-l} \\\\
0 & -\frac{2}{h} & 0 & -\frac{b+t}{t-b} \\\\
0 & 0 & -\frac{1}{f-n} & -\frac{-n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

## 透视投影

透视投影则以参数fov、宽高比、近点距离和远点距离定义的平截头体来进行投影，其投影矩阵的求解与正射投影类似，如下图所示：

![image.png]({{ site.url }}/assets/images/perspective_proj.png)

$$ \mathbf{T} = \begin{bmatrix}
\frac{1}{ar \cdot \tan(\frac{\alpha}{2})} & 0  & 0  & 0 \\\\
0 & \frac{1}{\tan(\frac{\alpha}{2})} & 0 & 0 \\\\
0 & 0 & \frac{f}{f-n} & -\frac{f \cdot n}{f-n} \\\\
0 & 0 & 1 & 0
\end{bmatrix} $$

第三行由下面这个方程计算得到

$$ \g(z)= \mathbf{A} + \frac{\mathbf{B}}{z} $$

$$ \g(n)= 0 $$

$$ \g(f)= 1 $$

透视投影矩阵第四行第三列为1的原因是投影矩阵的计算需要除以z坐标值，无法直接用矩阵相乘表示，只能够通过将z坐标值保存在w坐标处，最后输出时除以w即可。

而考虑到观测坐标系到裁剪坐标系的变换矩阵为:

$$ \mathbf{S} =  \begin{bmatrix}
1 & 0  & 0  & 0 \\\\
0 & -1 & 0  & 0 \\\\
0 & 0  & -1 & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

则最终的投影矩阵为：

$$ \mathbf{T'} = \mathbf{T}\mathbf{S} 
= \begin{bmatrix}
\frac{1}{ar \cdot \tan(\frac{\alpha}{2})} & 0  & 0  & 0 \\\\
0 & \frac{1}{\tan(\frac{\alpha}{2})} & 0 & 0 \\\\
0 & 0 & \frac{f}{f-n} & -\frac{f \cdot n}{f-n} \\\\
0 & 0 & 0 & 1
\end{bmatrix} \begin{bmatrix}
1 & 0  & 0  & 0 \\\\
0 & -1 & 0  & 0 \\\\
0 & 0  & -1 & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} 
= \begin{bmatrix}
\frac{1}{ar \cdot \tan(\frac{\alpha}{2})} & 0  & 0  & 0 \\\\
0 & -\frac{1}{\tan(\frac{\alpha}{2})} & 0 & 0 \\\\
0 & 0 & -\frac{f}{f-n} & -\frac{f \cdot n}{f-n} \\\\
0 & 0 & -1 & 0
\end{bmatrix} $$