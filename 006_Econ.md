
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
