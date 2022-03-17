# 3D Reconstruction of Cameras and Structure

本章主要描述了如何利用2张图片来恢复相机的参数以及物体在三维空间中的形状。

## 问题 
第一个问题：三维空间的点 $X_i$,这是未知量。已知量是：$x_i$位于第一幅图像, $x^{'}_i$位于第一幅图像。而且已知$x_i$与$x^{'}_i$是相互对应的：$x_i \leftrightarrow x^{'}_i$，他们都是$X_i$分别在第一第二幅图像上的投影点，用公式表示为：$x_i = PX_i$, $x^{'}_i = P^{'}X_i$
三维重建就是要找到$P, P^{'}$ 并且重建是有歧义的，歧义具体是指什么，如何消除这种歧义。这是第二个问题。

## 10.1 三维重建的具体过程
1. 根据$x_i \leftrightarrow x^{'}_i$计算基本矩阵$F$, 其具体过程在11章叙述
2. 根据F求出摄像机外参$R,T$，具体过程参见结论 9.14
3. 根据$R,T$，$x_i \leftrightarrow x^{'}_i$ 求解$X_i$，这个过程叫三角化（Triangulation），具体步骤在12章

**要点**：**三角化唯一不能确定的点就是基线上的点。因为从两个光心出发的射线互相重合了**

## 10.2 重建中的歧义

如果我们仅仅知道若干图像，是不可能恢复出三维空间点的绝对位置。
原因如下：
我们定义相似变换$H_S$

$$
 \left[
 \begin{matrix}
   R & t \\
   0^T & \lambda \\
  \end{matrix}
  \right] 
$$

我们有$PX_i = (PH^{-1}_s) (H_sX_i)$,因为$\lambda$是任意的，所以有很多的$H_s$可以满足前式。几何解释参见书中265页图10.2.

如果摄像机的内参也不知道，那么$H$矩阵就是不相似变换了，而是投影变换，投影变换只能保持直线还是直线，但是直线之间的角度就无法保持了。

以下介绍几种不同的重建类型。

## 10.3 projective reconstruction

projective reconstruction的特点是相机没有标定。

**结论 10.1**：如果两幅图像中若干对应点已知，具体表示为$x_i \leftrightarrow x^{'}_i$，那么我们可以求出基本矩阵$F$, 只需要$F$就可以重建出三维空间中的点。而且任意更换相机投影矩阵，对重建没有影响，因为不同重建之间是等价的（比如第一次重建用的是$P_1,P^{'}_1$,第二次重建用的是$P_2,P^{'}_2$，但是点不能换，不管第一次重建还是第二次重建，都是用$x_i \leftrightarrow x^{'}_i$）具体见书266页

## 10.4   Stratified reconstruction

 Stratified reconstruction指的是先有一个projective reconstruction， 然后再把它优化到 affine reconstruction 或者metric reconstruction。当然，需要注意的是affine 和 metric reconstruction都需要额外知道一些关于重建场景本身的信息，或者相机要被标定过。

 ### 10.4.1 affine reconstruction的具体过程。
 我们现有一个project reconstruction的结果，记为$(P,P^{'},\{X_i\})$。 现在我们需要找出一个平面$\pi$, 使其成为无穷远平面$(0,0,0,1)$在图像上的投影。则必然存在一个$H$, 满足 $H^{-1}\pi = (0,0,0,1)$。$H$可以写成如下形式：

 $$
 \left[
 \begin{matrix}
   I | 0 \\
   \pi^T  \\
  \end{matrix}
  \right] 
 $$

 找到了$H$以后，把$H$作用在所有重建得到的点上，就完成了affine reconstruction。affine的意思就是说把我们得到的某平面投影到无穷远处。**那么如何找出$(0,0,0,1)$到底映射到已知图像上的什么地方？以下给出几个例子**

 #### 摄像机纯平移
 简单来说就是摄像机从不同位置拍两幅图，但是摄像机本身只能有平移，不能有旋转。那么这两幅图中有一些点是没有移动的，比如月亮，比如一条延伸到无穷远处的公路。这样的话，月亮这一点的三维坐标写成$X_i$, 在两幅图像中的坐标写成$x_i,x^{'}_i$。这样三个点就确定了一个平面。该平面就是$(0,0,0,1)$投影到拍摄图像上的结果。这两图拍摄图像对应的基本矩阵$F$是一个斜对称（skew-symmetric）矩阵。

 #### 对场景增加约束
 场景约束主要目的就是为了找三个在无穷远平面上的点。比方说两个平行线为一组，可以确定无穷远平面上的一个点，这样找三组就可以了。具体步骤参见12章，13章。
 另外一个需要注意的是，在一副图像中找出无穷远点以后，可以利用基本矩阵找出第二幅图像中的对应点，不用重新算一遍。

 第二种方法是用相交直线之间的比例关系，具体过程参见书47页

 #### The infinite homography
 当我们找出了无穷远平面$(0,0,0,1)$在图像中的投影，我们实质上确定了一个映射$H_{\infty}$ ，$H_{\infty}$把图像$P$中的点映射到$P^{'}$。具体可以表示为$x^{'} = H_{\infty}x_i$。

 怎样求出这个$H_{\infty}$? 假设我们现在有一个affine reconstruction，两个摄像机的外参表示为$P=[M|m]$, $P^{'}=[M^{'}|m^{'}]$, $H_{\infty} = M^{'}M^{-1}$

 $H_{\infty}$还可以通过基本矩阵$F$和三对无穷远点来计算，参见13章

 #### 两个摄像机中有一个是affine摄像机

 在这种情况下如何求出affine reconstruction。我们有以下结论：

  **结论 10.4**

 $(P,P^{'},\{X_i\})$ 是一个projective reconstruction。 $P=[I|0]$。所以$P$是一个affine 摄像机，那么affine reconstruction可以这样获得：交换$P,P^{'}$的最后两列，交换$\{X_i\}$ 的最后两个坐标。

 ### 10.4.2 metric reconstruction的步骤

 metric reconstruction的要点是找到absolute conic。
 比较实际的做法是在图像中找到absolute conic，该conic反投影回无穷远处的平面，就变成了cone，那么这个cone就定义了无穷远处的absolute conic。

 **结论 10.5**

 假设图像中的absolute conic已知，记为$\omega$, affine reconstruction的相机外参已知，记为$P=[M|m]$, 那么affine reconstruction就可以利用一个矩阵$H$变成metric reconstruction. $H$如下所示

 $$
 \left[
 \begin{matrix}
   A^{-1} & 0 \\
   0      & 1  \\
  \end{matrix}
  \right] 
 $$

 其中$A$ 满足 $AA^{-1} = (M^{T}\omega M)^{-1}$

 我们可以把上式右边用Cholesky factorization处理，就可以得到$A$

 那么接下来的问题就是如何找到图像中的absolute conic？ (用$\omega$来表示该conic)。
 我们可以对该conic施加一些约束，然后再求解它。有这么几个约束：

 1. 被重建场景中的正交性
   消失点$v_1,v_2$分别来自两个正交的直线，那么他们满足 $v_1^{T} \omega v_2 = 0$ 确定一个conic需要五个参数，那么找三对$v_1,v_2$就可以解出这个方程。或者 $v_1,v_2$分别来自一条直线和一个平面（直线与平面正交）。 则他们满足 $l=\omega v$
 
 2. 相机内参
   
    因为$\omega = K^{-T}K^{-1}$

 3. 根据上一条约束，我们知道$\omega$只和相机内参有关系，跟外参没关系。那么我们可以用同一个相机在两个不同位置拍摄。这个过程可以表示为 $\omega^{'} = H^{-T}_{\infty}\omega H^{-1}_{\infty}$. 找出足够多的$H_{\infty}$也可以解出这个方程

 ### 10.4.3 直接利用$\omega$重建

 我们有一个projective reconstruction，我们还知道$\omega$, 把$\omega$在无穷远平面上的位置记为$\Omega_{\infty}$, 然后$\omega$所在的平面记为$\pi_{\infty}$。那么从$\omega$到$\Omega_{\infty}$的矩阵就可以求解出来，参见书342页练习题(x),知道这个矩阵，把它作用在projective的重建结果上，就得到了metric 重建的结果。

 ## 10.5 利用ground truth重建
 假设我们知道一些三维点的gt，记为$X_{Ei}$, 重建出来的点记为$X_i$,那么他们满足$X_{Ei} = HX_i$，因为前文说过重建时有歧义的。我们把$X_i$替换为图像中的点$x_i$, 那么就有$x_i = PH^{-1}X_{Ei}$，找出足够多的点，解这个方程就可以了。当我们知道了$H$,就可以$H$乘到相机矩阵$P,P^{'}$上，这样projective重建就变成了真实的三维坐标。

 ## 总结
 参见书277页




