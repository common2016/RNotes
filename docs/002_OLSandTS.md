
# 普通回归和时间序列序列 {#TS}
## 一句话Tips
- `confint`函数返回系数的置信区间
- `bssm`拟合非线性卡尔曼滤波的包。`pomp`，`KFAS`也是。`pomp`好像接口更简单些，第四节有一个非线性的例子。
- `NlinTS`一个利用神经网络的格兰杰因果非线性检验。
-  `slider`: 在任何R数据类型上提供类型稳定的滚动窗口函数，并支持累积窗口和扩展窗口。
-  `testcorr`: 提供计算单变量时间序列中自相关显著性、双变量时间序列中互相关显著性、多变量序列中皮尔逊相关显著性和单变量序列i.i.d.特性的测试统计量的功能。  
- `apt`一个阈值协整包。
- `fDMA`动态平均模型。卡尔曼滤波的贝叶斯模型平均。
- `MuMIn`利用信息准则进行模型平均的包。
- `MSBVAR`提供了贝叶斯框架下的马尔科夫转移VAR。`MSwM `是一个单方程（非单变量）的马尔科夫转移模型估计。
- 因子变虚拟变量：`model.matrix`可以生成回归所需要的矩阵，可以把因子变量变成虚拟变量。
- `mfGARCH`包估计混频GARCH。
- `TED::ur.za.fast`和`urca::ur.za`未知断点的单位根检验。 
- `mFilter`包有各种经济和金融常用的滤波，如HP，BK等滤波（好像没有更新了，官网包的镜像没有找到）。但是可以使用`FRAPO`包的`trdhp`函数来计算HP滤波 。
- `svars`是一个数据驱动的结构VAR包。`vars`是一个VAR各种估计和诊断的标准包。`tsDyn`也有线性VAR和VECM的估计，其中它还允许包含外生变量。
- `lmtest`有`grangertest()`做双变量格兰杰因果检验。`MTS::GrangerTest(regdata[,-c(1,2)],2,locInput = 1)`也可以，而且可以做多个变量是不是某个变量的格兰杰原因。`locInput`表示因变量是第几列。
- `stats4`包提供了函数`mle`可以进行极大似然估计，还可以固定部分参数，优化其他参数，这其实是集中似然的思想。关键是它还返回方差协方差矩阵。语法如下，

```r
mle(minuslogl, start = formals(minuslogl), method = "BFGS",
    fixed = list(), nobs, ...) # 注意它的初值是一个list
```

- `dynlm::dynlm`包一个比`lm`更强大线性回归结构，优点有三：
    - 可以使用差分、滞后等表述，如`d(y)~L(y,2)`，可以直接添加趋势项`trend(y)`将使用`$(1:n)/Freq$`作为回归元。
    - 可以进行工具变量估计。
但要注意，他的数据不是数据框，而是一个`ts`对象。 

- `nardl`估计非线性协整分布滞后模型。
- `rugarch`：单变量garch建模。一个比`forcast`更好用的时序建模包。可以用`show`函数来返回一个丰富的结果，包括一些检验结果。
- `rmgarch`：多变量garch建模。包括dcc,adcc,gdcc等。
- `stats`包中的`ARMAtoMA`函数可以计算AR变成MA。`vars`包的`Phi`返回VAR的移动平均系数。
- `vars`包里面的`Phi`函数可以把VAR变成VMA。使用`summary`函数来摘要var的估计结果，会给粗特征根，残差相关矩阵等。
- `tsDyn`包的`VECM`函数比较好用，可以包括外生变量，可以选择OLS或Joson方法。这个包也是可以估计线性VAR的，主要是他的`lineVar`函数。`egcm`包是恩格尔格兰杰协整检验，这个检验在`urca`包里业可行。
- `TSA::periodogram`可以做谱分解。
- `bvarsv`时变参数var建模
- `nls`非线性最小二乘法函数
- `highfrequance`里面有不少意思的函数，包括`HAR`。
## 回归中关于公式的理解和构造

```r
# 构造公式, 只要包含波浪线就意味着这是一个公式。
F1 <- dist ~ speed - 1
# 获得公式中所有的变量
mf1 <- model.frame(F1,data = cars)
# 抽取因变量
model.response(mf1)
# 抽取自变量
model.matrix(F1, data = cars)
```
公式的高级应用还有一个包`Formula`，其说明文件很到位。主要阐述了`|`的使用方式。

## GMM估计
$$
i _t =\beta_0 + \beta_1pi_t + \beta_2GDP_t + \beta_3hp_t + \beta_4i_{t-1} + \varepsilon_t
$$

因包含因变量的滞后项从而有内生性，欲使用$i_{t-2},i_{t-3},i_{t-4}$作为工具变量，从而做一个GMM估计，亦即整个方程的矩条件为，


\begin{align}
E(pi_t\varepsilon_t) = 0\\

E(GDP_t\varepsilon_t) = 0\\

E(hp_t\varepsilon_t) = 0\\

E(i_{t-2}\varepsilon_t) = 0\\

E(i_{t-3}\varepsilon_t) = 0\\

E(i_{t-4}\varepsilon_t) = 0
\end{align}


利用这些矩条件的GMM估计在`gmm`包中的写法为，

```r
gmmrlt <- gmm(g = it ~ pi + gdp + hp + it1, x = ~ pi + gdp + hp + it2 + it3 + it4,data = dwg0)
summary(gmmrlt)
```
其中`g`可直接写成公式，`x`即为工具变量集。

## 季节调整
### R中有`x12`包可以做季节处理
注意：
- 要先下载美国统计局的x12程序包，并在调用函数时，记得写上所以存储的路径。
- 仅可处理R中内置的时间序列对象`ts`。
- 示例代码：

```r
library(x12)
data(AirPassengers)
x12out <- x12work(AirPassengers,
             x12path = 'C:\\ado\\plus\\WinX12\\x12a\\x12a.exe',keep_x12out = FALSE)
x12out$d11 #此即为调整后的时间序列
```
其中，`keep_x12out`参数表示是否要保留计算后的文件。

### `seasonal`包有x13处理，更加便捷

```r
library(seasonal)
m <- seas(AirPassengers) # x13 处理
final(m) # 最终调整序列
plot(m) # 绘制调整和未调整序列
```

