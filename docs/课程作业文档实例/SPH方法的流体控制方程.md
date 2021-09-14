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

# SPH方法的流体控制方程

## 密度的粒子近似法（连续性方程）

粒子法天然满足连续性方程。

### 密度求和法

适用于广义流体力学问题，体现了SPH近似法的本质。

$$\rho_i = \sum_{j=1}^N{m_jW_{ij}}$$

修正方案（对等式右端正则化）：

$$\rho_i = \frac{\sum_{j=1}^N{m_jW_{ij}}}{\sum_{j=1}^N{(\frac{m_j}{\rho_j})W_{ij}}}$$

### 连续性密度法

适用于强间断问题的模拟（如爆炸、高速冲击等）。

$$ \frac{d \rho_i}{d t} = \sum_{j=1}^N{m_jv_{ij}^\beta\cdot\frac{\partial W_{ij}}{\partial x_i^\beta}} $$

## 动量方程的粒子近似

$$ \frac{dv_i^\alpha}{dt} = -\sum_{j=1}^N m_j \frac{p_i+p_j}{\rho_i\rho_j}\frac{\partial W_{ij}}{\partial x_i^\alpha} + \sum_{j=1}^N m_j\frac{\mu_i\epsilon_i^{\alpha\beta} + \mu_j\epsilon_j^{\alpha\beta}}{\rho_i\rho_j}\frac{\partial W_{ij}}{\partial x_i^\beta} $$

其中，

$$ \epsilon_i^{\alpha\beta} = \sum_{j=1}^N\frac{m_j}{\rho_j}v_{ij}^\beta\frac{\partial W_{ij}}{\partial x_i^\alpha} +
\sum_{j=1}^N\frac{m_j}{\rho_j}v_{ij}^\alpha\frac{\partial W_{ij}}{\partial x_i^\beta} -
(\frac{2}{3}\sum_{j=1}^N\frac{m_j}{\rho_j}\cdot\nabla_iW_{ij})\delta^{\alpha\beta}$$

该方法属于直接法，可模拟不同粘度系数的流体。

