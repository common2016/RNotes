
# 数据处理 {#DataProcess}
## 一句话Tips

- 整体上概览一个数据框，有一个很好的函数，

```r
skimr::skim(iris)
```


Table: (\#tab:unnamed-chunk-1)Data summary

|                         |     |
|:------------------------|:----|
|Name                     |iris |
|Number of rows           |150  |
|Number of columns        |5    |
|_______________________  |     |
|Column type frequency:   |     |
|factor                   |1    |
|numeric                  |4    |
|________________________ |     |
|Group variables          |None |


**Variable type: factor**

|skim_variable | n_missing| complete_rate|ordered | n_unique|top_counts                |
|:-------------|---------:|-------------:|:-------|--------:|:-------------------------|
|Species       |         0|             1|FALSE   |        3|set: 50, ver: 50, vir: 50 |


**Variable type: numeric**

|skim_variable | n_missing| complete_rate| mean|   sd|  p0| p25|  p50| p75| p100|hist  |
|:-------------|---------:|-------------:|----:|----:|---:|---:|----:|---:|----:|:-----|
|Sepal.Length  |         0|             1| 5.84| 0.83| 4.3| 5.1| 5.80| 6.4|  7.9|▆▇▇▅▂ |
|Sepal.Width   |         0|             1| 3.06| 0.44| 2.0| 2.8| 3.00| 3.3|  4.4|▁▆▇▂▁ |
|Petal.Length  |         0|             1| 3.76| 1.77| 1.0| 1.6| 4.35| 5.1|  6.9|▇▁▆▇▂ |
|Petal.Width   |         0|             1| 1.20| 0.76| 0.1| 0.3| 1.30| 1.8|  2.5|▇▁▇▅▃ |
- 也可以形成一个数据摘要

```r
modelsummary::datasummary_skim(mtcars)
```
关键是这个数据摘要可以各种形式输出，包括html,tex,md,txt,png等。

- `feather`包可以读写`feather`格式的软件，优点在于该格式不仅速度快，压缩打，而且与Jialia, python可以交互。
- `expand.grid(x = seq(-3,3.1,0.5), y = seq(-3,3.1,0.5))`会生成关于x和y的格点，这些格点可以用来画向量场，三维图之类的很有用。
- `ISOcodes`包提供了各种ISO标准，包括国家的2位码，3位码等。

```r
# ISO_3166_1 is a character frame with variables Alpha_2, Alpha_3, and Numeric (giving the two-letter, three-letter and three-digit numeric country codes) and Name, Official_name, and Common_name (giving the respective names).
ISO_3166_1
```

- 因子操作

```r
# 使用字符串有两个缺陷：第一，不在因子水平范围内的不会转化成NA
# 第二，仅按字母排序。
# 因此，通过设定因子水平，可以解决上述两个问题。注意水平和字符串是一样的，
# 只是相当于设定了范围和排序。
	factor(c('Dec','Apr','Jam','Mar'), levels = c('Jan','Feb','Mar','Apr','May'))
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
- `XLConnect`处理excel最强大的包。但需要JRE（java run enviornment）。

```r
# 可以不改变原有数据，然后把一个数据框精准地写入某个地方
writeWorksheetToFile("XLConnectExample2.xlsx", data = ChickWeight,
 sheet = "chickSheet", startRow = 3, startCol = 4,header = FALSE, clearSheets = FALSE)
```


- 使用`as.Date`来生成日期，必须带有年月日三个要素，使用`format`来输出日期格式，此时可以只输出年和月。如`as.Date('2010/05/01') %>% format(.,format = '%Y%m')`
- `seq.Date()`生成日期序列，包括日、星期、月、年。
- `readstata13`包可以读入更高版本的stata数据格式。
- `zoo::rollapply(x, 30, mean)`就是30天的移动平均求值。

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

## `tidyverse`包的一些函数列示

- `filter, arrange, mutate, relocate`
- `distinct(flights, origin, dest)`，flights数据框中`origin,dest`两列中唯一的数据行挑出来。
- `count`与`distinct`类似，但多了个对重复元素计数的列`n`。
- `rename(a = b)`把`b`重命名为`a`。
- `slice_head(df, n = 1),slice_taile(df, n= 1),slice_min(df,x,n=1),slice_max(df,x,n=1)`抽取第1个或倒数第一个元素,抽取`x`列最小或最大的第一个元素。
- `pivot_longer, pivot_wider`长宽改变。
- `select`是一个很牛逼的函数

```r
select(regdata,id, year) # 选择regdata数据框的id和year两列
select(regdata,starts_with('abc')) # 匹配以'abc'开头的列
select(regdata,ends_with('abc')) # 匹配以'abc'结尾的列
select(regdata,contains('abc')) # 匹配包含'abc'的列
select(regdata,matches('abc')) # 正则表达匹配
select(regdata,num_range('x',1:3)) # 匹配x1, x2,x3的列
```

## R与网络交互

- 很多网站提供了API以供调用，可以利用`RCurl::getForm()`函数调用API。比如百度翻译的API调用如下：

```r
feed_url <- "http://api.fanyi.baidu.com/api/trans/vip/translate"
    rlt <- RCurl::getForm(feed_url, .params = list(q = 'apple', 
        from = 'en', to = 'zh', appid = appid, salt = rmdnum, sign = sn))
```
第一个参数书调用的网址，第二个参数是GET请求的各项参数。
- `RJSONIO::fromJSON()`可以对网站返回的JSON格式的数据变成列表以便调用。


## `data.table`包

这个包操作几十万的数据，简直就是“卧槽”，几乎感觉不到延迟，数据量大，强烈推荐该包。

### 基本操作

- 该包基本语法为`dt[i,j,by]`，即行，列操作(包括选列，函数运算等)和分组。
- 提供单个参数，如`dt[1:3]`，即为选择行。
- `.N`是非常有用的符号，分组时，它表示计数，单独用`dt[.N]`则选择了最后一行。`.I`则表示`1:.N`。
- 筛选行时，可以直接使用列名，如`dt[province == '北京市']`
- 选择列时，也是直接用列名如`city`，而不是加引号的`'city'`。比如`dt[,city]`，如果你一定喜欢用引号，可以使用`dt[,'city',with = FALSE]`。
- 选择多个列，既可以用`dt[,c('id','year'),with = FALSE]`也可以用`dt[,list(id,year)]`，而且还可以这样创造新的列`dt[,list(id,year, density = size/weight)]`。而且`list()`还有个缩写`.()`。
- 原地赋值`:=`。如`dt[,density:=size/weight]`，原来的数据框就已经改变了。注意这实际上是“引用语义”，有个时候在把数据传递进函数时，你并不想使用引用语义，你可以在函数前面增加一行代码`DT <- copy(DT)`，以便实现深复制。
- 该包的`set`系列函数因为避免了不必要的复制，展现出惊人的性能。`setDF(dt)`把data.table变成data.frame，`setDT(dt)`是反向操作。`setnames(dt,'name_pre','name_now')`修改列名。`setcolorder(dt,c('id','year'))`给列重新排序。

### 分组操作


## 与SQL数据库交互

SQL数据库可以百度SQLStudio软件作为图形化界面打开，查阅等。在R语言中，有`RSQLite`包。不过该数据框最多容纳2000列。工作流如下

```r
library(RSQLite)
# 创建连接
con <- dbConnect(SQLite(), dbname = 'data-raw/mydatabase.sqlite')
# 写入数据库，name 是数据库的表名，value 是R中的数据框
dbWriteTable(con, name = 'GQ2008', value = GQ2008, overwrite = T)
# 查数据库有啥表
dbListTables(con)
# 读数据库
dbReadTable(con, 'GQ2008')
# 读部分数据库
dbGetQuery(con, 'select var1, var2, [var(3)] from GQ2008') # 字段中包含特殊字符时要用中括号括起来
# 断开链接
dbDisconnect(con)
```


## 连接各种数据库的R包
### `RJSDMX`包下载世界各大数据库数据

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

### `imfr`包下载IMF数据


```r
# Download Real Effective Exchange Rate (CPI base) for the UK and China
# at an annual frequency
real_ex <- imf_data(database_id = 'IFS', indicator = 'EREER_IX',
               country = c('CN', 'GB'), freq = 'A')
```

### `OECD`包下载OECD的数据

它的小短文非常有用。

### `WDI`包

该包提供了对世界银行世界发展指标的一个下载接口，非常好用。同时它的`WDI_data`包含了一些有用的数据集，包括各国2位码和3位码标识。

## 前向、后向、线性和样条插值
- `zoo`包
  - `zoo::na.locf`缺省设置可以前向插，即缺失值等于前面的值。当将该函数的`fromLast`参数设为真时，即为后向插。
  - `zoo:na.approx`可以线性插值但不能外推；`na.spline`可以样条插值；

- `imputeTS`包,`imputeTS::na.locf`也可以，不过它只能对数值。它也有后向插值选项。

### `signal`包
它有一个插值函数`interp1`函数，比较好用：

```r
interp1(x, y, xi, method = c("linear", "nearest", "pchip", "cubic", "spline"), 
        extrap = NA, ...)
```
它的参数说明如下

- x,y：vectors giving the coordinates of the points to be interpolated. x is assumed to be strictly monotonic.
- xi：points at which to interpolate.

method	：
one of "linear", "nearest", "pchip", "cubic", "spline".

- 'nearest': return nearest neighbour
- 'linear': linear interpolation from nearest neighbours
- 'pchip': piecewise cubic hermite interpolating polynomial
- 'cubic': cubic interpolation from four nearest neighbours
- 'spline': cubic spline interpolation–smooth first and second derivatives throughout the curve. for method='spline', additional arguments passed to splinefun.
Details

- extrap：
if TRUE or 'extrap', then extrapolate values beyond the endpoints. If extrap is a number, replace values beyond the endpoints with that number (defaults to NA).
