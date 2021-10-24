<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# 光滑、粒子、NS方程（流体动力学）

## 光滑(理解为，近似，但它仍应该叫做光滑)

对于网格法，在空间布有网格，即可以照顾到所有所需空间。

而对离散的粒子而言，SPH不是MD，从实际效率上SPH也不能在所需空间内完全布满与真实世界一样密度的真实粒子，所以需要光滑（近似手段），使用伪粒子模拟实际粒子流场特性。

反过来不难理解，尽管我们将流体视为一个个分散的粒子，但流体毕竟是连续充满整个空间的，流体中每个位置参与运算的值都是由周围一组粒子累加起来的。这就是光滑的来由。

----

以上是从文字描述上来看待问题，以下给出数学定义：

迪利克雷函数 -> 光滑（核）函数。

$$\delta(r) = \left\{
\begin{array}{ll}
\infty, & if \ r = 0; \\
0, & otherwise.
\end{array}
\right.$$

满足，$\int{\delta(r)dv}=1$。

任意函数，满足

$$ f(r) =\int_\Omega f(r^{'})\delta(r - r^{'})dr^{'} $$

引入光滑函数，来近似，显然需要满足一些条件让光滑函数保留迪利克雷函数的特质。(SPH习惯用法，核近似用角括弧`<>`标记)

$$ f(r) \approx <f(r)> =\int_\Omega f(r^{'})W(r - r^{'}, h)dr^{'} $$

不得不提，光滑函数$W$常选用偶函数，需要满足：

1. 正则化条件；
2. 当光滑长度趋于0时，具有迪利克雷函数性质；
3. 紧支性；
4. 偶函数的核函数，具有$h^2$精度或二阶精度。

## 粒子（亦称为，伪粒子）

从网格法到粒子法，事情并没有那么美好。

与SPH核近似法相关的连续积分表示式可转化为支持域内所有粒子叠加求和的离散化形式。

$$ f(r) \approx <f(r)> =\int_\Omega f(r^{'})W(r - r^{'}, h)dr^{'} \\
\approx \sum_{j=1}^N f(r_j)W(r-r_j,h)\Delta V_j \\
=\sum_{j=1}^N\frac{m_j}{\rho_j}f(r_j)W(r-r^{'},h) $$

注意到，这里有**2处近似**。  
则粒子$i$处的函数的粒子近似式：

$$ <f(r_i)> \approx \sum_{j=1}^N\frac{m_j}{\rho_j}f(r_j)W_{ij} $$

其中，$W_{ij} = W(r_i-r_j, h)$。

粒子法作为一种无网格法，它也需要面对很多实际问题做出相应的理论和实践解释，这并不比网格法简单。

## N-S方程（流体动力学）

3大控制方程 + 补充方程（本构方程，或状态方程），能体会出来，粒子法天然性地满足质量守恒定律（粒子不应该凭空产生，也不该凭空消失）。
所以问题就变成了求解N-S方程。

### 粒子受力分析$^{[1]}$

SPH算法的基本设想，就是将连续的流体想象成一个个相互作用的微粒，这些例子相互影响，共同形成了复杂的流体运动，对于每个单独的流体微粒，依旧遵循最基本的牛顿第二定律。

$$\overrightarrow{F}=m\overrightarrow{a}$$

这是我们分析的基础，在SPH算法里，流体的质量是由流体单元的密度决定的，所以一般用密度代替质量

$$\overrightarrow{F}=\rho\overrightarrow{a}$$

这里$F$的量纲发生了变化，需要注意！  
易知：

$$\overrightarrow{F} = \overrightarrow{F}^{external} + \overrightarrow{F}^{pressue} + \overrightarrow{F}^{viscosity}$$

#### 重力

外部力一般是重力。

$$\overrightarrow{F}^{external} = \rho \overrightarrow{g}$$

#### 压差作用力

由流体内部的压力差产生的作用力。

$$\overrightarrow{F}^{pressue} = -\nabla p$$

#### 粘性力

由粒子之间的速度差引起的。

$$\overrightarrow{F}^{viscosity} = \mu\nabla^2\overrightarrow{u}$$

#### NS方程

联立上式，得

$$\rho \overrightarrow{a} = \rho \overrightarrow{g} -\nabla p + \mu \nabla^2\overrightarrow{u}$$

加速度形式：

$$\overrightarrow{a} = \overrightarrow{g} - \frac{\nabla p}{\rho} + \frac{\mu \nabla^2\overrightarrow{u}}{\rho}$$

实际运算过程中，有时还需要考虑表面张力得影响。

对应SPH中的NS方程形式为：

$$\overrightarrow{a}(r_i) = \overrightarrow{g} - \frac{\nabla p(r_i)}{\rho(r_i)} + \frac{\mu \nabla^2\overrightarrow{u}(r_i)}{\rho(r_i)}$$

### SPH推导过程

对于SPH算法来说，基本流程就是这样，根据光滑核函数逐个推出流体中某点的密度，压力，速度相关的累加函数，进而推导出此处的加速度，从而模拟流体的运动趋势。

#### 插曲（标注）：函数空间导数的粒子近似式

$$ \nabla f(r_i) \approx <\nabla f(r_i)> \approx \sum_{j=1}^N\frac{m_j}{\rho_j}f(r_j)\nabla W_{ij} $$

$$ \nabla^2 f(r_i) \approx <\nabla^2 f(r_i)> \approx \sum_{j=1}^N\frac{m_j}{\rho_j}f(r_j)\nabla^2 W_{ij} $$

思考：$\nabla 1 = 0, \nabla^2 1 = 0$。

#### 密度

$$ <\rho(r_i)> \approx \sum_{j=1}^N\frac{m_j}{\rho_j}\rho(r_j)W_{ij} \\
= \sum_{j=1}^N m_j W_{ij} $$

#### 压力

$$ <\nabla p(r_i)> \approx \sum_{j=1}^N\frac{m_j}{\rho_j}p(r_j)\nabla W_{ij} $$

#### 粘度

$$ <\mu \nabla^2 \overrightarrow{u}> \approx \mu\sum_{j=1}^N\frac{m_j}{\rho_j}\overrightarrow{u}(r_j)\nabla^2W_{ij} $$

## 算法实现

SPH算法的基本流程：

1. 初始化粒子，为每个粒子赋上初始位置
2. 计算每个粒子的密度
3. 计算每个粒子的压强
4. 计算每个粒子的加速度
5. 根据临界条件调整加速度
6. 根据加速度计算每个粒子的速度变化
7. 根据速度计算粒子位置的变化
8. 绘制粒子
9. 回到步骤2

## 参考文献

[1] [SPH算法简介: 对我的启蒙](https://blog.csdn.net/liuyunduo/article/details/84098884)[EB/OL].(2018-11-15). 
(说明：作者原文链接失效，所以仅采用有效参考链接。)
