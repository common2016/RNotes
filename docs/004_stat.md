

# 统计
## 一句话Tips
- `cmna::mcint `可以进行蒙特卡洛积分。
- 数值积分：`pracma::integral` 
- 多元正态分布随机抽样：`SimDesign::rmvnorm`，还有`mvnfast`。`mvtnorm`包是多变量正态分布的包，很全。
- `KSgeneral`包执行KS检验，比较一个分布是否来自某个理论分布。`stats`包的`ks.test`和`dgof`包的`ks.test`也可以,并且可以比较双样本是否来自同一个分布。
- `choose(n,k)`：组合公式，n个里面选k个，有多少种组合方式。`utils::combn(n,k)`也可以。 `e1071::permutations`实现排列。
- `qrandom`: 利用量子波动产生真随机数.
- 主成分分析可以调用`psych`包两个步骤实现：

```r
# 画个图选特征值数目：
# 1. 特征值在1以上的才行； 2. 特征值大于模拟的平均特征才可行； 3. 碎石图
library(psych)
fa.parallel(regdata, fa = 'pc')
# 计算2个主成分。如果想要主成分载荷更有经济意义，注意设置旋转参数
 principal(regdata,nfactors = 2,rotate = 'none')
```

## MCMC算法
### 吉布斯抽样原理
如果联合分布不好求，但条件分布好求，可以用这个算法。

### 一些共轭先验分布的结论
理解这些结论，对于后续使用吉布斯抽样、MH算法非常有用。

**结论1** 若$x_1,\cdots,x_n$是从均值为$\mu$(**未知**)，方差为$\sigma^2$(**已知**且为正)中正态分布中所抽取的一个随机样本，同时假定$\mu\sim \mathcal{N}(\mu_0,\sigma_0^2)$，则给定数据和先验分布，$\mu$的后验分布也是一个正态分布，其后验均值和方差为，
$$\mu_* = \frac{\sigma^2\mu_0+n\sigma_0^2\overline x}{\sigma^2+n\sigma_0^2},\hspace{2em}\sigma_*=\frac{\sigma^2\sigma^2_0}{\sigma^2+n\sigma^2_0},\;\;\;\text{其中},\overline x= \sum_i^n x_i/n$$

推广到多变量，则可以写为，
$${\mu}_*=\Sigma_*(\Sigma_0^{-1}{\mu}_0+\Sigma^{-1}\overline{\bf{x}}), \hspace{2em}\Sigma_*^{-1} = \Sigma_0^{-1}+n\Sigma^{-1}$$

**结论2**  若$e_1,\cdots,e_n$是从均值为0，方差为$\sigma^2$的正态分布中抽取的随机样本，同时假定$\sigma^2$的先验分布是自由度为$\nu$的逆$\chi^2$分布，即$\frac{\nu\lambda}{\sigma^2}\sim \chi^2_\nu,\lambda>0$，则$\sigma^2$的后验分布也是逆$\chi^2$分布，自由度为$\nu+n$，
$$\frac{\nu\lambda+\sum_i^ne_i^2}{\sigma^2}\sim \chi^2_{\nu+n}$$

### 一个吉布斯抽样的典型案例
一个带自相关的回归模型可以写为，
\begin{align}
y_t&=\beta_0+\beta_1x_{1t}+\cdots+\beta_kx_{kt}+z_t\\
z_t&=\phi z_{t-1}+e_t
\end{align}

该模型需要估计的参数有三个，即$\theta = (\beta',\phi,\sigma^2)$。该参数的联合分布并不好求，但是条件分布则好求得多。


### Metropolis 和 M-H算法
如果后验分布除了那个归一化的常数不知道，但分子是知道的，那可以用这个算法。这个场景是不是在贝叶斯估计中很熟悉？

`MCMCpack::MCMCmetrop1R`中有个例子提供了Metropolis算法，感觉还是很清晰。里面提到的'The proposal distribution'其实就是跳跃分布，即给定上一次抽样的参数，从这个跳跃分布中抽下一个参数。

### 一些带贝叶斯估计的R包使用报告
- `MTS::BVAR`：这个包可以在一个一般的先验设定上估计VAR，先验可以是乏信息先验，也可以是明尼苏达先验，但问题是该包仅返回估计系数的均值和标准误，不返回抽样。
- `bvartools`：在很大程度上可以定制BVAR的mcmc抽样，见它的一个优秀的引言。我用这个,自己写了乏信息先验的BVAR估计包。下次我再把明尼苏达先验添进去。
- `MCMCpack::MCMCregress`:单方程的贝叶斯估计，它提供了$\beta$是多元正态先验，方程误差项的方差协方差是逆伽玛的先验估计。
- `bayesm::runireg`：单方程的贝叶斯估计，它提供了$\beta$是多元正态先验，方程误差项的方差协方差是卡方分布的先验估计。
