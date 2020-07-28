---
title: 坐标系和变换
tags: 
- graphics
- vulkan
---

# View Matrix

将世界坐标转换到相机的观测坐标系，如下图所示：

$$ \mathbf{p'} = \mathbf{T}\mathbf{p} $$

![image.png]({{ site.url }}/assets/images/view_transform.png)

坐标系\\( x,y,z \\) 表示世界坐标系，而右下角的\\( x',y',z' \\)则表示相机的坐标系，相机坐标系xyz分别对应相机的右、上、后等方向。

观测矩阵就是将世界坐标系中的点\\( p(a,b,c,d) \\)变换到观测坐标系中的\\( p'(a',b',c',d') \\)。

令\\( \mathbf{f},\mathbf{s},\mathbf{u} \\)分别表示相机在世界坐标系下的前、右和上三个方向的单位向量，\\( \mathbf{L} \\)表示相机在世界坐标系的位置， \\( <\mathbf{s},\mathbf{u},-\mathbf{f}> \\)则组成了我们想要的坐标系，表示成矩阵则是：

$$ \begin{bmatrix}
\mathbf{s}_{x} & \mathbf{u}_{x} & -\mathbf{f}_{x} & 0 \\\\
\mathbf{s}_{y} & \mathbf{u}_{y} & -\mathbf{f}_{y} & 0 \\\\
\mathbf{s}_{z} & \mathbf{u}_{z} & -\mathbf{f}_{z} & 0 \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

再加上相机的位置所需要的偏移，则最终的变换矩阵就是：

$$ \begin{bmatrix}
\mathbf{s}_{x} & \mathbf{u}_{x} & -\mathbf{f}_{x} & -\mathbf{L} \cdot \mathbf{s} \\\\
\mathbf{s}_{y} & \mathbf{u}_{y} & -\mathbf{f}_{y} & -\mathbf{L} \cdot \mathbf{u} \\\\
\mathbf{s}_{z} & \mathbf{u}_{z} & -\mathbf{f}_{z} & \mathbf{L} \cdot \mathbf{f} \\\\
0 & 0 & 0 & 1
\end{bmatrix} $$

其中\\( (-\mathbf{L} \cdot \mathbf{s},-\mathbf{L} \cdot \mathbf{u},-\mathbf{L} \cdot \mathbf{-f}) \\)是相机位置\\( \mathbf{L} \\)在相机坐标系内表示后的反方向。

