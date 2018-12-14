---
title: "非线性最小二乘(18-11-02更新)"
tags: ["VIO","SLAM" ]
date: "2018-06-20"
---
高博的SLAM十四讲写的很好，但有些地方可能大家会有点疑问，或者说是不够详尽。
这里介绍细节的非线性最小二乘的方法。
<!--more-->

## 1. 最小二乘问题
最简单的最小二乘问题：  
$$ \underset{x}{min}\frac{1}{2}||f(x)||_{2}^{2} \tag{1}$$
自变量 $x\epsilon\mathbb{R}^{n}$ ，f是任意非线性函数，设它为m维:$f(x)\epsilon\mathbb{R}^{m}$。  
如果f的形式很简单，问题可能可以用解析形式求解，即令目标函数的导数为零，进而求得x的最优值。但实际应用中并不方便使用这种方法，更多的是使用迭代逼近的方法取找到导数为0的点。

### 1.1 梯度下降法
将目标函数 $||f(x)||^{2}$在x处进行taylor展开：
$$||f(x+\Delta{x})||^{2} \thickapprox ||f(x)||^{2} + J(x)\Delta{x} + \frac{1}{2}\Delta{x}^{T}H\Delta{x} \tag{2}$$  
J是目标函数$||f(x)||^{2}$关于x的导数，即jacobian矩阵，H为Hssian矩阵。  
如果仅保留一阶项，整理有：
$$||f(x)||^{2} - ||f(x+\Delta{x})||^{2} =  -J(x)\Delta{x} \tag{3}$$  
在$\Delta{x}$是下降的方向的前提下，即$J(x)\Delta{x}<0$，就可以把右边看作是目标函数从x到$x+\Delta{x}$下降的量。

进一步，$\Delta{x}$取多少才能使下降量尽可能大？
首先引入一个步长参数$\lambda$来控制学习率，而限定$\Delta{x}$为单位长度，则显然，当取$\lambda>0$，$\Delta{x}$与-J(x)同向时，右式最大。

即结论是函数沿负梯度方向下降最快，即: $\Delta{x^{*}} = -J^{T}(x)$ ，而具体下降的多少，则有步长$\lambda$来确定。

步长的确定:
    通常full gradient的方法会使用 Line Search的方式来确定步长， SGD则使用自适应的learning rate, 如AdaDelt，Adam 等等

Armijo + Line Search的基本思想: 每次试一个步长，如果使用该步长，函数值会不会比当前点下降一定的程度，如果没有，就按比例减小步长，再试，直到满足条件(Arimigo-Goldstein condition)

---

### 1.2 牛顿法

2018-11-2日更新，阔别依旧呀~ 诸君~长出来了呀~

上面我们介绍说确定方向即可，只使用了一阶梯度的信息，如果我说，我想直接求出 $\Delta x$ 呢？  
Easy~ 同样的，我们把(2)式子调整一下，保留二阶信息:
$$ ||f(x)||^{2} - ||f(x+\Delta{x})||^{2} = -J(x)\Delta{x} - \frac{1}{2}\Delta{x}^{T}H\Delta{x} \tag{4}$$  
我们希望取得的 $\Delta x$ 使得左边的下降量最大, 即求等号左边$ y(\Delta x) $ 的极大值， 对$\Delta x$求偏导，有下式:
$$ H\Delta{x} = -J^T \tag{5} $$

牛顿法存在的问题就是需要计算H矩阵，这在问题规模比较大的时候非常困难，所以一般用拟牛顿法替代。

--- 

### 1.3 高斯-牛顿法
高斯-牛顿法则是不再直接对 $f(x)^2$ 展开，而是针对 $f(x)$ 进行展开:
$$ f(x +\Delta{x}) \thickapprox f(x) + J(x)\Delta{x} \tag{6}$$
不去寻找x使得目标函数最小，转而去寻找 $\Delta{x}$ 使得下降量最大; 先展开:
$$ \frac{1}{2}||f(x) + J(x)\Delta{x}||^2 = \frac{1}{2}(||f(x)||_2^2 +2f(x)^T J(x)\Delta{x} + \Delta{x}^T J(x)^T J(x)\Delta{x}) \tag{7}$$  
对上式求$\Delta{x}$偏导，使其等于0，最后得到下式:
$$ J(x)^T J(x)\Delta{x} = -J(x)^T f(x) \tag{8}$$  
最终也就变成了线性的标准求解形式 $ Ax = b $,就可以使用标准的求解方式计算(SVD分解、Choleskey分解等方法). 

---

### 1.4 列文伯格-马夸尔特法
上述方法，存在的最大问题，就是当约束不充足，$J^T J$不满秩