


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
- `update(object, formula)`函数可以更新公式重新估计。

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
