

# 原生的R {#rawR}
## 一句话Tips
- 从本地安装一个源码包：

```r
install.packages('E:\\17_HuaDong\\NEDatabase\\MyRef.Attachments\\RForEconometrics/phtt_3.1.2.tar.gz',type = 'source',repos = NULL)
```
- `suppressMessages()`可以抑制代码抛出的信息等。
- `+(3,1)`与`3+1`的作用是一样的。这可以推广到其他中缀函数，如`%*%`等。
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
- 如何安装已经过期的包？
   1. 点[这里](https://cran.r-project.org/src/contrib/Archive/)找到过期的包，然后下载下来。
   2. 用这个命令安装本地的包：`install.packages('D:/MSBVAR_0.9-3.tar.gz',repos = NULL, type = 'source')`

## 平行计算
平行计算中，光使用`foreach`包是不够的，还需要注册一个平行背景注册，否则`foreach`包在运算完以后会返回警告：

```r
# Warning message:
#
# executing %dopar% sequentially: no parallel backend registered 
```

平行计算一般的工作流如下：

```r
library(doParallel)
cl <- parallel::makeCluster(2)
doParallel::registerDoParallel(cl)
foreach(i=1:3, .pacakages = 'tidyverse') %dopar% sqrt(i)
parallel::stopCluster(cl)
```

要注意，平行计算中，在`foreach`后的语句中，相当于在每个进程中，重启了一个新环境。因此，如果你要用到`foreach`外面的变量，则需要把变量、包等都传进去。同时，这些变量如果是向量或者`list`则不需要特别地指定迭代变量是哪个，程序会自动将它们处理成迭代变量。如果这些变量的长度不一，则迭代时以最少长度的变量为准。一个简单的例子(`VARrf::IRFrf_gen`)如下：

```r
# 传进去了5个变量,nhist, itevar,pmax,s, shockvar，这些变量的长度都是一样的。
picdata <- foreach::foreach(i = 1:nhist,itevar = itevar,
                              pmax = pmax_para,s = s, shockvar = shockvar,
                              .packages = 'tidyverse') %dopar% {
    devtools::load_all()
    IRFrf(data = itevar, pmax = pmax, s = s, shockvar = shockvar,histime = i)
  }
```


## 类和方法
### S3类

```r
# 查看属于一个泛型函数的所有方法：
methods('mean')
```

```
## [1] mean.Date     mean.default  mean.difftime mean.POSIXct  mean.POSIXlt 
## [6] mean.quosure*
## see '?methods' for accessing help and source code
```

```r
# 反过来，查看一个类，都有何关联的泛型函数
methods(class = 'ts')
```

```
##  [1] [             [<-           aggregate     as.data.frame cbind        
##  [6] coerce        cycle         diff          diffinv       initialize   
## [11] kernapply     lines         Math          Math2         monthplot    
## [16] na.omit       Ops           plot          print         show         
## [21] slotsFromS3   t             time          window        window<-     
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

## `apply`函数族
- `lapply(vector,fun)`，可以将函数作用于向量`vector`的每一个元素上，然后返回一个`list`。
- `sapply(vector,fun)`的好处在于不是返回一个list，而是返回一个向量或者矩阵。如果`fun`返回的是一个元素，那么`sapply`就返回一个向量，如果`fun`返回的是一个向量，则`sapply`按列将结果拼接成一个矩阵。

```r
sapply(1:10, function(i) i^2)
```

```
##  [1]   1   4   9  16  25  36  49  64  81 100
```

```r
sapply(1:10, function(i) c(i^2,i))
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    1    4    9   16   25   36   49   64   81   100
## [2,]    1    2    3    4    5    6    7    8    9    10
```
- `apply`是我用得最熟的，它是将函数应用到矩阵或数组的行或列上。
- `mapply`是`lapply`或`sapply`的多元版本，即它可以同时输入多个向量，如

```r
mapply(function(x,y,z) x*y + y*z + x*z, x = c(1,2,3),y = c(1,2,3), z = c(-1,-2,-3))
```

```
## [1] -1 -4 -9
```

## Linux中的R
很多时候，对服务器我们没有权限，因此我们只能下载包的源码，然后安装。工作流如下：
- 在自己有权限的目录下新建一个目录，比如`/data/stage/chenpu/RLib`作为包的安装目录，命令为
```
mkdir /data/stage/chenpu/RLib
```
- 将包的源码下载下来，然后命令行安装
```
R CMD INSTALL /.../mypackage.tar.gz --library=/data/stage/chenpu/RLib
```
- 在R跑程序，加载的时候，要注意`lib.loc`参数，

```r
library(zoo, lib.loc = '/data/stage/chenpu/RLib')
```
或者修改启动文件`.Rprofile`，这就不会每次启动R都要重新设置了。

### 如何修改`.Rprofile`？
进入R以后，

```r
R.home() # 确定R安装在了哪里
file.edit(file.path('~','.Rprofile')) # 编辑Rprofile
# 此时进入vi界面， 在插入模式下，键入
.libPaths('/data/stage/chenpu/RLib')
# 再按Esc退出插入模式进入命令行模式，输入`:wq`保存退出。
```

## 如何完美复制项目的结果
在写一个项目时，往往对应着当时你调用的各种R包的版本。一年以后，许多包已经更新，可能你当时的代码结果已经无法复制。我们需要管理不同版本的R包。

`renv`包就是干这个的。工作流如下：

1. `renv::init()`初始化这个项目，会生产对应该项目的包库文件目录。此时安装包，都是在这个私有库下进行，不影响全局的包库。
2. `renv::snapshot()`记下此时包的各个版本。
3. `renv::restore()`恢复上一个快照时包库的状态。
