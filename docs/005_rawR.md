

# 原生的R {#rawR}
## 一句话Tips


- `cmd+alt+B`运行到当前行，`cmd+shift+0`重启R会话。
- 比如你自己写了个包，你可能想查看它被安装下载了多少次，代码如下：

```r
dlstats::cran_stats('CHNCapitalStock') # 给出每个月的下载量
# 或者使用另一个包给出每天的下载量
# devtools::install_github("metacran/cranlogs")
cran_downloads('CHNCapitalStock',from = '2020-06-01', to = '2021-01-14')
```

- `.libPaths()`查看R的安装目录
- `object.size()`可以查看某个对象占了多少内存，但一般不准，返回结果偏大。可以使用`lobstr::obj_size()`来看变量的大小。
- `do.call(str_fun, args, quote, envir)`函数感觉很好用, 第一个参数是字符串函数名，第二个参数是一个list, 它里面元素的名字就是你调用的函数的参数的名字。在某种程度上，它可以替代`eval(parse(text = str))`的作用。
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
但通常给数据框的列赋予一个`label`的属性，在RStudio中显示数据时，会同时把这个标签显示出来，非常方便。

```r
dt <- data.frame(x = 1:3, y = 6:8)
attr(dt$x, 'label') <- 'time'
```


- `remove.packages('dplyr')`，卸载已安装的包。
- `system`或`shell`运行Shell命令。

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

## 容错机制

R中使用`tryCatch`进行容错。语法如下：


```r
tryCatch(
  expr, 
  error = function(e) { ... },
  warning = function(w) { ... },
  finally = { ... }
)
```

- expr：需要尝试执行的代码。
- error：一个函数，当expr中出现错误时会被调用，e是错误对象。
- warning：一个函数，当expr中出现警告时会被调用，w是警告对象。
- finally：一个表达式，无论是否出现错误或警告，都会在tryCatch执行完成后运行。

一个例子如下，

```r
result <- tryCatch(
  {
    # 尝试执行的代码
    seq.Date(as.Date('2025-01-10'), as.Date('2025-01-09'))
  },
  error = function(e) {
    # 错误处理代码
    message("发生错误：", e$message)
    return(NA)  # 返回一个默认值
  }
)
print(result)
```

在这个例子中，10 / 0会导致错误，tryCatch会捕获这个错误，并执行error函数中的代码，返回NA。

## 整洁运算

在R语言中存在大量整洁运算，比如`dplyr::rename(dt, newname = oldname)`中`oldname`就不需要引号括起来。那么如何在自己撰写的函数中进行整洁运算，即如何让函数的参数能够不先求值而直接传递到函数中去。一般有两种做法：一是使用`enquo`和`!!`对，二是使用两对花括号`{{}}`。如下：

```r
# enquo, !!的用法
myfunction <- function(data, x, y){
  x <- enquo(x)
  y <- enquo(y)
  ggplot(data, aes(!!x, !!y)) + geom_points()
}

# {{}}的用法
myfunction <- function(data, x, y){
  ggplot(data, aes({{x}}, {{y}})) + geom_points()
}

# 数据框中列名用字符串的方法
my_var <- "disp"
mtcars %>% summarise(mean = mean(.data[[my_var]]))
```

另外就是`:=`符号可以让左边的变量名也能用上整洁运算，一个例子如下：

```r
my_function <- function(data, var, suffix = "foo") {
  # Use `{{` to tunnel function arguments and the usual glue
  # operator `{` to interpolate plain strings.
  data %>%
    summarise("{{ var }}_mean_{suffix}" := mean({{ var }}))
}
```


## 更新R

1. 在Mac中更新R，你要去R的官网下载最新的R版本然后装上，它会自动覆盖原有的旧R。你可以进一步打开`/Library/Frameworks/R.frameworks/versions/`在里面删除旧R的残余文件夹。

2. 有个时候升级R以后，载入某些包时报错，你可以用如下命令更新所有的R包，基本都能解决。
```r
update.packages(checkBuilt=TRUE, ask=FALSE)
```

- `checkBuilt`参数是说如果是用以前的R安装的包是不是也算旧包需要更新？

## 大数据的存取

对于大的数据，载入内存运算往往给内存造成很大负担，16G内存的电脑，超过500M，就应该考虑使用一些专用的包来读取加速。

### `qs`包

`fst`包能高速读取数据框，`qs`包采用了这个技术，但它还能高速读取列表。用法如下


```r
library(qs)
df1 <- data.frame(x = rnorm(5e6), y = sample(5e6), z=sample(letters, 5e6, replace = T))
qsave(df1, "myfile.qs")
df2 <- qread("myfile.qs")
```

### `SOAR`

该包将数据存到硬盘上，然后利用`SOAR`包进行调用。通常可如下调用


```r
library(SOAR)
Store("some_data")
```

这会有如下动作：

- 在工作目录下创建一个子目录`.R_Cache`。这个目录是隐藏的，可以通过在RStudio右下角Plane中的Files标签中通过勾选显示隐藏而看到。
- 对象`some_data`以.Rdata的形式存储在该子目录中；
- 对象`some_data`在全局环境中被删掉；

此时，在该工作目录下，可以像调用全局环境中的变量一样调用`some_data`。同时还可以用`Remove`来去掉该变量，用`Ls()`来查看路径中有哪些变量。

## RStudio-server管理
```
rstudio-server start #启动
rstudio-server stop #停止
rstudio-server restart #重启

查看运行中R进程
rstudio-server active-sessions
指定PID，停止运行中的R进程
rstudio-server suspend-session <pid>
停止所有运行中的R进程
rstudio-server  suspend-all
强制停止运行中的R进程，优先级最高，立刻执行
rstudio-server force-suspend-session <pid>
rstudio-server force-suspend-all
RStudio Server临时下线，不允许web访问，并给用户友好提示
rstudio-server offline
RStudio Server临时上线
rstudio-server online
```

## 平行计算
### 循环中的平行运算
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

还要注意`foreach`中有一个参数`.export`,这个参数的含义很重要。因为在平行环境中是没有任何变量的，要把你当前环境中的变量或函数导入到平行运行的环境中，就要利用`.export`变量，它以字符串的形式把当前环境变量导入到平行环境中，如`.export = c('myvar','myfun')`。

### `apply`族的平行版本


```r
library(parallel)
# 检查系统有几个核，然后全部利用
cl <- makeCluster(detectCores())
# Windows并行时，每个进程中是没有变量的，所以要把函数或变量导入到每个进程中去
clusterExport(cl,c('fun1','var1'))
# 或者要把某个包导入到进程中去
clusterEvalQ(cl,{
    library(randomForestSRC)
    library(tidyverse)
  }) %>% invisible()
# 并行运算
parLapply(cl, c('a','b','c'), FUN)
stopCluster(cl)
```

## 类和方法
### S3类

```r
# 查看属于一个泛型函数的所有方法：
methods('mean')
```

```
##  [1] mean,ANY-method          mean,denseMatrix-method  mean,sparseMatrix-method
##  [4] mean,sparseVector-method mean.Date                mean.default            
##  [7] mean.difftime            mean.POSIXct             mean.POSIXlt            
## [10] mean.quosure*           
## see '?methods' for accessing help and source code
```

```r
# 反过来，查看一个类，都有何关联的泛型函数
methods(class = 'ts')
```

```
##  [1] [             [<-           aggregate     as.data.frame cbind        
##  [6] cbind2        coerce        cycle         diff          diffinv      
## [11] initialize    kernapply     kronecker     lines         Math         
## [16] Math2         monthplot     na.omit       Ops           plot         
## [21] print         rbind2        show          slotsFromS3   t            
## [26] time          window        window<-     
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
### Ubuntu下安装最新的R
1. 在` /etc/apt/sources.list`文件中增加软件的镜像源，`deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran40/`，也就是增加这样一行。注意后面这个`bionic-cran40`，它对应着Ubuntu系统下R4.0版本。
2. 更新源：
```
sudo apt-get update
```
更新的时候往往会报错，说是没有公钥。此时用这个命令增加公钥：
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6AF0E1940624A220 
```
注意，此处`6AF0E1940624A220`是你更新源时报错提示的key。
3. 安装：
```
sudo apt-get install r-base
```

## macOS与R
### 安装R

- 在官网下载对应的R，按说明装即可。要注意的是如果是macOS 11即Big Sur以后的版本，要下载对应的R包，否则可能在后期会出问题。比如我在Julia中安装RCall包以便在Julia中调用R，但该包怎么都安装不了，后来发现是R的版本安装错了。然后RStudio下载下来只要拖到Applications里面即可，比较简单。

- 如果要从源码编译R代码，还需要两个工具`Xcode`和`gfortran`。前者在App Store里面免费安装，后者参照R官网指导命令行安装即可(`https://mac.r-project.org/tools/`),要注意的是：
  - 解压命令`tar fxz gfortran-f51f1da0-darwin20.0-arm64.tar.gz -C /`意味着把压缩包解压到根目录下，而不是用户目录下。因此，前面加上`sudo`前缀，否则会解压到用户目录下。

### 配置R

在R启动前可以执行一些命令，这些命令可以写在`.Rprofile`文件中，该文件在主目录下。你可以使用`file.edit('~/.Rprofile')`命令打开，该文件若不存在，会自动创建一个。然后在该文件中写入如下代码以设置默认的R包的存储位置，

```r
.libPaths(c('~/elements/RLibrary',.libPaths()))
```
后面加了个`.libPaths()`是因为以前默认的R包位置已经安装了一些包，这些包也要在搜寻的路径上。

## 如何完美复制项目的结果
在写一个项目时，往往对应着当时你调用的各种R包的版本。一年以后，许多包已经更新，可能你当时的代码结果已经无法复制。我们需要管理不同版本的R包。

`renv`包就是干这个的。工作流如下：

1. `renv::init()`初始化这个项目，会生产对应该项目的包库文件目录。此时安装包，都是在这个私有库下进行，不影响全局的包库。
2. `renv::snapshot()`记下此时包的各个版本。
3. `renv::restore()`恢复上一个快照时包库的状态。
