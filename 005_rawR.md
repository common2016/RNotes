

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
