---
title: "R语言笔记"
author: "云腾足下"
date: "2020年5月"
site: bookdown::bookdown_site
description: ""
documentclass: book
biblio-style: apalike
bibliography: [mybib.bib]
link-citations: yes
---



# 序 {-}

不知道说啥，还是留首诗吧。

>赵客缦胡缨，吴钩霜雪明。
>
>银鞍照白马，飒沓如流星。
>
>十步杀一人，千里不留行。
>
>事了拂衣去，深藏身与名。
>
>闲过信陵饮，脱剑膝前横。
>
>将炙啖朱亥，持觞劝侯嬴。
>
>三杯吐然诺，五岳倒为轻。
>
>眼花耳热后，意气素霓生。
>
>救赵挥金锤，邯郸先震惊。
>
>千秋二壮士，烜赫大梁城。
>
>纵死侠骨香，不惭世上英。
>
>谁能书阁下，白首太玄经。








<!--chapter:end:index.Rmd-->




# 面板数据 {#PanelData}
## 一句话Tips
- `PSTR`:面板平滑转移模型。
- `MSCMT`:多个结果变量的合成控制方法的包。
- 检查面板数据是否平衡：使用`table(PanelData[,1:2])`或者`is.banance`。
- ` phtt`包，交互效应的面板模型，用的Bai (2009)的估计方法。It offers the possibility of analyzing panel data with large dimensions n and T and can be considered when the unobserved heterogeneity effects are time-varying.

## `plm`包
- 包中的`vcovG`函数可以计算聚类标准误。一般这么用:

```r
summary(plm, vcov = vcovG(plm, cluster = 'group', inner = 'cluster'))
```

- 包中的`fixef`函数可以返回个体截距项(`type = level`)。
- `updata(object, formula)`函数可以更新公式重新估计。

## 动态面板阈值估计：R语言中有一个包`dtp`，其估计函数为：

```r
data(Mena)
reg<-dtp(GDPPC ~ FDI+OPEN|INF|INF,Mena,index=c("pays","ann"),4,2,0.95,0.8,1,graph = TRUE)
summary(reg)
```
注意：
- 第一根`|`前的变量是不依赖区制的变量，中间由`|`夹住的变量是阈值变量，最后一个`|`后面的变量是依赖区制的变量（好遗憾，貌似只允许一个这样的变量）。
- `initnum`参数指的是模型中的内生变量。在动态面板中，一般是因变量的滞后值，因此在数据框中滞后因变量，然后选好该滞后因变量所在列的数字即可。注意，这个数字是在剔除了id和year标识之后的列的序号。
- 数据上千以后，估计过程有点慢，耐心等待。
- 输出中包含一个`gamma`参数，我揣摩是截距项。

<!--chapter:end:001_paneldata.Rmd-->


# 普通回归和时间序列序列 {#TS}
## 一句话Tips
- `gdpc`计算广义动态主成分。
- `POET::POETKhat`提供了计算Bai and NG (2002)因子数目的函数。
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
m <- seas(AirPassengers) # x13 处理, AirPassengers是一个ts对象
final(m) # 最终调整序列
plot(m) # 绘制调整和未调整序列
```


<!--chapter:end:002_OLSandTS.Rmd-->


# 数据处理 {#DataProcess}
## 一句话Tips
- 因子操作

```r
# 使用字符串有两个缺陷：第一，不在因子水平范围内的不会转化成NA
# 第二，仅按字母排序。
# 因此，通过设定因子水平，可以解决上述两个问题。注意水平和字符串是一样的，
# 只是相当于设定了范围和排序。
	factor(c('Dec','Apr','Jam','Mar'), levels = ('Jan','Feb','Mar','Apr','May'))
# 因子重编码, 把1改成unmarried等
farcats::fct_recode(rawdata$marrige,
           'unmarried'='1','married'='2','cohabitation'='3','divore'='4','wid'='5')
```

- `dbplyr`可以连接到几乎任何数据库。
- `wbstats`下载世界银行数据，很牛逼。Stata里面的`wbopendata`包更牛逼。
- `stationaRy`:一个从NOAA上下载气象数据，如气温，风向等的包。该包就三个函数，一个用来得到站点id，一个用这个id下载数据，还有一个是如果你想得到其他额外的气象数据时可能有用。
- **当你发现你用`save`命令保存一个数据长达数分钟时**，建议你迅速调用`qs`包，可能一分钟不到就帮你快速读入和保存了。但这个包一次只能保存一个变量。
- `tor`: 提供允许用户同时导入多个文件的功能.
- 读入excel中的sheet名：`openxlsx::getSheetNames(file)`
- `XLConect`处理excel最强大的包。但需要JRE（java run enviornment）。

```r
# 可以不改变原有数据，然后把一个数据框精准地写入某个地方
writeWorksheetToFile("XLConnectExample2.xlsx", data = ChickWeight,
 sheet = "chickSheet", startRow = 3, startCol = 4,header = FALSE, clearSheets = FALSE)
```


- 使用`as.Date`来生成日期，必须带有年月日三个要素，使用`format`来输出日期格式，此时可以只输出年和月。如`as.Date('2010/05/01') %>% format(.,format = '%Y%m')`
- `seq.Date()`生成日期序列，包括日、星期、月、年。
- `readstata13`包可以读入更高版本的stata数据格式。
- `zoo::rollapply(x, 30, mean)`就是30天的移动平均求值。

- `select`是一个很牛逼的函数

```r
select(regdata,id, year) # 选择regdata数据框的id和year两列
select(regdata,starts_with('abc')) # 匹配以'abc'开头的列
select(regdata,ends_with('abc')) # 匹配以'abc'结尾的列
select(regdata,contains('abc')) # 匹配包含'abc'的列
select(regdata,matches('abc')) # 正则表达匹配
select(regdata,num_range('x',1:3)) # 匹配x1, x2,x3的列
```
- R语言给数组各维数命名

```r
# Create two vectors of different lengths.
vector1 <- c(5,9,3)
vector2 <- c(10,11,12,13,14,15)
column.names <- c("COL1","COL2","COL3")
row.names <- c("ROW1","ROW2","ROW3")
matrix.names <- c("Matrix1","Matrix2")

# Take these vectors as input to the array.
result <- array(c(vector1,vector2),dim = c(3,3,2),dimnames = list(row.names,column.names,
                                                                  matrix.names))
print(result)
```

-  `pdftools`包的函数可以读PDF文件：

```r
pdf_info(pdf, opw = "", upw = "")

pdf_text(pdf, opw = "", upw = "")

pdf_data(pdf, opw = "", upw = "")

pdf_fonts(pdf, opw = "", upw = "")

pdf_attachments(pdf, opw = "", upw = "")

pdf_toc(pdf, opw = "", upw = "")

pdf_pagesize(pdf, opw = "", upw = "")
```


同时，利用`qpdf`包的`pdf_subset,pdf_combine,pdf_split`可以提取PDF的部分内容，合并PDF文件，把每一页分成一个PDF文件。

## `RJSDMX`包下载世界各大数据库数据

一般工作流：

```r
library(RJSDMX)
# 查看有哪些库可以用
getProviders()
# 库中有哪些子库可以用
getFlows('WITS')
# 该子库调取数据需要哪几个字段
getDimensions('WITS','WBG_WITS,DF_WITS_TradeStats_Tariff,1.0')
# 查看这个指标有几个选项 
getCodes('WITS','WBG_WITS,DF_WITS_TradeStats_Tariff,1.0','INDICATOR')
# 查好了就可以下载
ans <- getTimeSeries('WITS', 'DF_WITS_TradeStats_Tariff/A.CHN.WLD.01-05_Animal.MFN-WGHTD-AVRG')
# 你也可以调用图形窗口查阅命令
sdmxHelp()
```

`IMF2`里面的`IFS`数据库里面有很多季度的宏观数据，如GDP，固定资本形成等

<!--chapter:end:003_DataProecess.Rmd-->



# 统计
## 一句话Tips
- `cmna::mcint `可以进行蒙特卡洛积分。
- 数值积分：`pracma::integral` 
- 多元正态分布随机抽样：`SimDesign::rmvnorm`，还有`mvnfast`
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

<!--chapter:end:004_stat.Rmd-->



# 原生的R {#rawR}
## 一句话Tips
- ``+`(3,1)`与`3+1`的作用是一样的。这可以推广到其他中缀函数，如`%*%`等。
- 写函数时，可以使用`...`参数，为捕获这个(可能是多个)参数的值，可以用`list(...)`这个办法。
- 只显示3位小数：

```r
round(0.123456,3)
```

```
## [1] 0.123
```
- 属性赋值：

```r
y <- c(1,2,5,8)
attr(y,'my_attribute') <- 'This is a vector'
attr(y,'my_attribute')
```

```
## [1] "This is a vector"
```
- `remove.packages('dplyr')`，卸载已安装的包。
- `system`或`shell`运行Shell命令。
- 更新所有的包`update.packages(checkBuilt=TRUE, ask=FALSE)`
- `pkgsearch`包的`ps`函数提供CRAN的关键词搜寻。
- `detach(package:dplyr)`可以去掉加载的包。
- `foreach`包提供循环的平行计算
- 在jupyter里面安装R，只需在anaconda里面的命令行中（anaconda prompt）输入，
```
conda install -c r r-essentials
```

- 工作目录下所有文件名`dir()`
- `file.copy, file.create, file.remove, file.rename, dir.creat, file.exists, file.info`
- `file.rename`批量修改文件名

```r
fr = paste('./加工贸易HS/2016/',dir('./加工贸易HS/2016'),sep = '')
to = paste('./加工贸易HS/2016/hp',dir('./加工贸易HS/2016'),sep = '')
file.rename(from = fr,to = to)
```
- `down.file`只要给出第一个参数：网址（包括ftp的）和第二个参数，下载的文件要保存的文件名，就可以直接在网上下载文件。如果中国乱码，记得使用fileEncoding = 'UTF-8'来修正。
- `getAnywhere(predict.Arima)`查看源代码
- `.rs.restartR()`重启一个新的R会话
- 平行计算。光使用`foreach`包是不够的，还需要注册一个平行背景注册，否则`foreach`包在运算完以后会返回警告：

> Warning message:
>
> executing %dopar% sequentially: no parallel backend registered 

A: 如何注册呢？调用`doParallel`包，代码如下：

```r
library(doParallel)
cl <- makeCluster(2)
registerDoParallel(cl)
foreach(i=1:3, .pacakages = 'tidyverse') %dopar% sqrt(i)
stopCluster(cl)
```

- 如何安装已经过期的包？
   1. 点[这里](https://cran.r-project.org/src/contrib/Archive/)找到过期的包，然后下载下来。
   2. 用这个命令安装本地的包：`install.packages('D:/MSBVAR_0.9-3.tar.gz',repos = NULL, type = 'source')`

## 类和方法
### S3类

```r
# 查看属于一个泛型函数的所有方法：
methods('mean')
```

```
## [1] mean.Date        mean.default     mean.difftime    mean.POSIXct    
## [5] mean.POSIXlt     mean.quosure*    mean.vctrs_vctr*
## see '?methods' for accessing help and source code
```

```r
# 反过来，查看一个类，都有何关联的泛型函数
methods(class = 'ts')
```

```
##  [1] [             [<-           aggregate     as.data.frame as_tibble    
##  [6] cbind         coerce        cycle         diff          diffinv      
## [11] filter        initialize    kernapply     lines         Math         
## [16] Math2         monthplot     na.omit       Ops           plot         
## [21] print         show          slotsFromS3   t             time         
## [26] window        window<-     
## see '?methods' for accessing help and source code
```

创建一个类，很多时候只需在最后返回一个这样的，就可以了，

```r
class(foo) <- 'myclass'   
```
然后为这个类创建一个泛型函数，只需要两步：

```r
# 创建一个类
a <- list()
class(a) <- 'a'
# 第一步：增加一个新的泛型函数。记住，没有搭配该泛型函数的方法，泛型函数是没有用的。
f <- function(x) UseMethod('f') 
# 第二步，为此泛型函数添加方法。关键在于命名规则，属于该泛型函数的方法一定具有类似f.a格式的命名。
# f是泛型函数, a是类,它们用点连起来。
f.a <- function(x) 'class a'
mean.a <- function(x) 'a' # 为已有的泛型函数增加方法
```


## 打印到文件
- `sink`函数：在代码开始前加一行：`sink(“output.txt”)`，就会自动把结果全部输出到工作文件夹下的output.txt文本文档。这时在R控制台的输出窗口中是看不到输出结果的。代码结束时用`sink()`切换回来。 示例：

```r
sink("a.txt") 
x<-rnorm(100,0,1) 
mean(x) 
sink()
```

- `cat`函数：`cat('abc','OK!',file = 'a.txt',sep = '\n',append = T)`
- `stargazer`函数：

```r
stargazer(fit1, fit2, title = "results", align = F, type = "text", no.space = TRUE, out = "fit.html")
```

<!--chapter:end:005_rawR.Rmd-->


# 经济学中的各种专业计算 {#Eco}
## 一句话Tips
- `lpirfs`包局部线性投影脉冲响应函数。
- `library(productivity)`计算满奎斯特效率指数，注意日期参数`time.var`以及个体`id.var`参数都要是整数。

```r
tfp <- malm(regdata, id.var = 'alphabets',time.var = 'yr', x.vars = c('wage','K'), y.vars = 'gdp')
Changes(tfp) # 获得malmquist指数及其成份
```

- 产品编码之间的转换包：`concordance::concord`：
It supports concordance between HS (Combined), ISIC Rev. 2,3, and SITC1,2,3,4 product classification codes, as well as BEC, NAICS, and SIC classifications. It also provides code nomenclature / descriptions look-up, Rauch classification look-up (via concordance to SITC2) and trade elasticity look-up (via concordance to SITC2/3 or HS3.ss).
- `ioanalysis`提供投入产出表的分析功能
- 出口增加值分解的包由`decompr`，这个包有Wang et at. (2013)的分解以及经典的里昂惕夫分解。以及`gvc`包。
- `hhi`包可以算赫芬达尔指数。

## wwz的贸易增加值分解
分解的主要函数是`decomp`。但是有一个`load_table_vectors`函数可以生成一个decompr class，这个类有很多我们想要的东西，如投入产出系数A，里昂惕夫矩阵B以及其他的一些数据。

```r
# load example data
data(leather)

# create intermediate object (class decompr)
decompr_object <- load_tables_vectors(inter,
                                      final,
                                      countries,
                                      industries,
                                      out        )
```
这个类包含了如下内容，我挑一些我确切知道是啥的:

- B: 里昂惕夫矩阵，即$(I-A)^{-1}$
- Vc: 增加值系数矩阵
- ESR: 总出口，包括中间品出口`Eint`和最终品`Efd`出口.
- L: 单个国家的里昂惕夫逆矩阵
- Y: 最终需求。几个国家就几列。
- Yd: 自己对自己的需求。也就是在这个矩阵里面，自己对别人的需求都是0。
- Ym: 自己对别人的需求。自己对自己的需求都是0。
- Bm, Bd: 含义比照Ym, Yd.

<!--chapter:end:006_Econ.Rmd-->


# 数学计算 {#math}
## 一句话Tips
- `crossprod(x,y)`的意思是$x'y$，`tcrossprod(x,y)`的意思是$xy'$。在OLS估计时还蛮省事。
- `matlab`包模拟了matlab软件中的许多矩阵函数。
- 克罗内克积使用`%x%`或者`kronecker`。
- `numDeriv`里面的`hession`计算海塞矩阵，它的逆的负数，就是极大似然估计的标准差。 
- `Matrix::bdiag(A,A)`生成以两个A为对角元素的分块对角矩阵。
- `Mod`计算复数的模，不同于`mod`用来整除。
-  `caracas`: 通过提供对Python SymPy库的访问来实现计算机代数，从而使以符号方式解方程、寻找符号积分、符号和和其他重要量成为可能。
-  `calculus`: 针对数值和符号演算提供了C++优化函数，包括符号算术、张量演算、泰勒级数展开、多元埃尔米特多项式等.
-  `rootSolve`提供了非线性方程(组)的解，以及微分方程的稳态解的形式。

```r
fun <- function (x) cos(2*x)^3
curve(fun, 0, 8) # 先画个图看看解大约在哪里
abline(h = 0, lty = 3) # 把0轴搞出来
uni <- uniroot(fun, c(0, 8))$root # 此时求解
```
方程组的根求解用`multiroot`。

## 数值优化
- **无约束优化**。`optimx`一个优化包，经常用它的`optimx`函数。但是它是无约束优化，尽管可以包含上下界的约束(盒子约束)。

```r
para <- list(R1 = 0.1, phi1 = 0.8, H1 = 0.3, A1 = 0.8, intcp = 1) %>% as.numeric()
# par就是要优化的参数的初值
# fn就是要优化的函数，譬如似然函数，这个函数可以包含多个参数，
#    如这里的Y,X,r等。然后那个没写进来的参数就是要优化的参数。
#    lnlik <- function(Y = Y, X = X,para = para, r = 1){...}
# gr和hess如果必要，可以包含进来。
# lower,upper就是搜寻的上下界。
a <- optimx(par = para, fn = lnlik, Y = Y, X = X, r = 1, 
            lower = c(0.01,0.01,0.01,0.01,-Inf), 
            upper = c(Inf, 0.99,Inf,Inf,Inf),control = list(all.methods = T))
gHgen(par, fn) # 创造得分矩阵，海塞矩阵。
```
- **有约束优化**。`stats::constrOptim`和`alabama::constrOptim.nl`都可进行有约束的优化，后者是对前者的一个强化，不仅在算法上更牛逼，也可以放入非线性约束。后者是用函数如`hin(x)<=0`的形式来表达约束，需要理解的是，约束函数中的参数一定要与目标函数的参数一致，即便约束函数没有用到目标函数的参数。

```r
# 比如我的目标函数如下
fn <- function (x,a,b){
    ...
}
# 约束函数如下。一定要把a和b写进去，即便函数中未用到它
hin <- function(x, a, b){
    ...
}
# 优化函数如下
constrOptim.nl(par, fn, gr = NULL, 
hin = NULL, hin.jac = NULL, heq = NULL, heq.jac = NULL, 
control.outer=list(), control.optim = list(), ...)
```
一旦得到优化函数，就可以调用`numDeriv::hessian(fn, par)`来计算海塞矩阵，从而得到标准误之类的。

<!--chapter:end:007_math.Rmd-->


# 机器学习和微观计量 {#ML}
## 一句话Tips
- `gfoRmula`：一个处理时变处理干扰的R包。
- `oem`包可以执行各类lasso，group lasso等算法。
- `gsynth`广义合成控制包。考虑了交互固定效应。
- `ArCo`: 一个人工反事实的包
- `ForecastComb`一个集成预测的包，包括stacking 方法。
- `htree`基于历史回归树，所以可以用于面板数据的随机森林模型。
- `tsensembler`一个对机器学习多种预测方法进行集成的包，包括随机森林，装袋，支持向量回归等。
- `knerlab`支持向量机和支持向量回归，他的vig写得好。
- `grf`，一个因果树的包。
- `spikeslab`，一个选择变量的贝叶斯方法的包。
-  `orf`: 实现Lechner和Okasa（2019）中开发的有序森林估计量，以估计具有有序分类结果的模型（有序选择模型）的条件概率.
-  `MatchIt`做匹配感觉很好。主要函数是`matchit`，针对这个函数得到的类，使用`summay`可以得到匹配前后均值变化，经验分位变化等。这个函数返回的类包含有一个元素`match.matrix`，里面有被匹配的控制组信息。

## ROC曲线绘制
ROC曲线可以同时展现所有可能阈值出现的两类错误。其横轴为1-特异度(1-specificity)，纵轴为灵敏度（sensitivity）.

调用`pROC`包即可计算。该包的参数如下：

```r
roc(response, predictor, controls, cases,
density.controls, density.cases,
levels=base::levels(as.factor(response)), percent=FALSE, na.rm=TRUE,
direction=c("auto", "<", ">"), algorithm = 5, quiet = TRUE, 
smooth=FALSE, auc=TRUE, ci=FALSE, plot=FALSE, smooth.method="binormal",
ci.method=NULL, density=NULL, ...)
```
- `response`: 原始的$y$;
- `pridictor`: 估计的$\hat y$;
- `smooth`:是否平滑ROC曲线；
- `percent`:是否百分比的形式显示相关信息，如AUC；

## `randomForestSRC`包使用报告
这个包有几大特征，我就说我用过的几个：

- 可以因变量多变量建模
- 重要性抽样有置信区间
- 偏效应计算接口更友好

<!--chapter:end:008_ML.Rmd-->


# 与其他软件的交互 {#otherSF}
## 一句话Tips
- `stargazer`的一个模版调用：

```r
stargazer(regression,type = 'text',out = "../PicTab/cmp.html",no.space = T,report = c('vcp'))
```
`report`意味着报告变量、系数与p值。

- `officer`: 与微软软件互动的一个包
- `readstata13`包读Stata13以后的数据格式。
- R语言调用stata

用RStata包可以从R里面调用stata，不过要先用`chooseBinStata()`先设置stata的安装路径。 也可以在R的启动环境中进行配置。
注意在启动环境中（即Rprofile.site文件中）配置时，应增加如下一行，
`options(RStata.StataPath = "\"D:\\Program Files (x86)\\Stata14\\StataMP-64\"")`

## R语言调用Matlab

Matlab里面的三维画图比R要省事很多。这里探讨一下如何从R调用Matlab的一般步骤。

    - 安装R.matlab包。使用`writemat(filename,A=A,B=B)`把R里面的数据写进Matlab并保存成`.mat`格式。
    - 安装matlabr包。使用`run_matlab_script`命令来执行一个`.m`脚本。或者使用`R.matlab`包里面的`evaluate`来一个一个地执行matlab命令。或者类似于调用stata：
    

```r
    library(matlabr)
    MatlabCode <- '
    a = 3;
    b = a+1;
    '
    run_matlab_code(MatlabCode)
```
- R语言读取SPSS（中文字符）

```r
# 读英文字符
library(foreign)  
mydata=read.spss("data.sav")  
# 或者如下
library(Hmisc)  
data=spss.get("data.sav") 

# 读中文字符
library(memisc)
data1 = as.data.set(spss.system.file("data.sav"))
data = as.data.frame(data1)
```
## R与Python的无缝对接
- 第一步，首先配置好环境

```r
library(reticulate)
use_condaenv("D:/Anaconda3")

# 安装的python版本环境查看，显示anaconda和numpy的详细信息。放在
# use_condaenv()后，以使配置生效
py_config()

py_available()#[1] TRUE   #检查您的系统是否安装过Python
py_module_available("pandas")#检查“pandas”是否安装
```

- 第二步，调用有多种方法。我最喜欢这种，就是直接导入python模块，然后用R的风格来调用。此时R里面的美元符号$相当于python里面的“.”符号 ，如，


```r
os <- import("os")
os$getcwd()
os$listdir()#您可以使用os包中的listdir（）函数来查看工作目录中的所有文件
 
numpy <- import("numpy")
y <- array(1:4, c(2, 2))
y
x <- numpy$array(y)
x
numpy$transpose(x)#将数组进行转置
numpy$linalg$eig(x)#求特征根和特征向量
```
### 其他：
- 当你发现有些包没有，需要安装的时候，可以如下，

```r
library(reticulate)

# create a new environment 
conda_create("r-reticulate")

# install SciPy
conda_install("r-reticulate", "scipy")

# import SciPy (it will be automatically discovered in "r-reticulate")
scipy <- import("scipy")
```
- 这是调用时通常需要的代码：

```r
library(reticulate)
# 可以查你有几个版本的python
py_config()
# 想使用哪个版本的python
use_python('C:/Users/sheng/AppData/Local/Continuum/anaconda3/python.exe')
# 检查python可不可用
py_available()
# 检查模块可不可用
py_module_available('tushare')
```

## 与julia的对接
### `JuliaCall`包
感觉此包没有类似`reticulate`包调用python那么无缝。

- 在R中执行julia脚本

```r
library(JuliaCall)
# 设置存放julia二进制文件的文件夹
julia_setup(JULIA_HOME = 'D:/Program Files/Julia-1.4.2/bin')
# 几种主要的调用方式，我把我喜欢的写出来的
julia$command("a = sqrt(2);") # 在julia环境中产生了变量a
ans <- julia$eval("a") # 把变量a的值传给R环境中的ans变量
# 其他的调用方式
julia_command("a = sqrt(2);")
julia_eval("a")
#> [1] 1.414214
2 %>J% sqrt
#> [1] 1.414214
```
- R与julia互传变量：前面提到的`julia_eval`可以把julia中的变量传出来，使用`julia_assign`可以把R中的变量传到julia中去。

```r
julia_assign('a',1:5)
julia_command('a')
```

- julia控制台，而且只要你前期`julia_setup()`了，这个控制台里面就包含了前期运算时的变量

```r
julia_console()
# 输入exit 可以退出
# julia> exit
```

- 它的函数调用非常吸引人：你甚至可以用R对象作为julia函数的参数

```r
julia_install_package_if_needed("Optim")
  opt <- julia_pkg_import("Optim",
                           func_list = c("optimize", "BFGS"))
rosenbrock <- function(x) (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
result <- opt$optimize(rosenbrock, rep(0,2), opt$BFGS())
result
```



<!--chapter:end:009_otherSF.Rmd-->


# 其他
- `geosphere::distm`根据经纬度算距离。
- `stats19`一个下载英国交通事故记录的数据库。
- `pacman`包管理包，如`p_load()`。
- `vitae`: 提供多个模板和功能以简化简历的制作和维护.
- `DiagrammeR`：R语言画流程图，网络图，太棒了。教程网址：http://rich-iannone.github.io/DiagrammeR/
- `progress`包可以显示进度条。更具体的例子见收藏或者帮助。

```r
library(progress)
pb <- progress_bar$new(total = 500) # 循环中的次数
for (i in 1:500){
    pb$tick()
}
```

- `swirl`可以编写交互式教程。
- `RSelenium`包终于在201901月出来了。

<!--chapter:end:010_other.Rmd-->



<!--chapter:end:9990-references.Rmd-->

