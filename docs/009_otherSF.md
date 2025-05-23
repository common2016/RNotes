
# 与其他软件的交互 {#otherSF}
## 一句话Tips

- 在Mac里面安装万得的R接口，老报错。等报错以后，你看下你存放R包的目录里面有个文件链接`00LOCK-WindR/00new/WindR`，同时目录下直接还有个`WindR`。把前者`WindR`下的全部内容复制到后者目录下即可。
- Mac中Jupyter配置R核


```r
# R控制台运行
install.packages('IRkernel')
# 终端运行
IRkernel::installspec()
# 再次打开Jupyter就有R内核了
```

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

- 在windows下，`options(RStata.StataPath = "\"D:\\Program Files (x86)\\Stata14\\StataMP-64\"")`
- 在Mac下，`options(RStata.StataPath = "\"/Applications/Stata/StataMP.app/Contents/MacOS/stata-mp\"")`

或许你还想增加stata的版本设置`options("RStata.StataVersion" = 17)`。

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

## R语言调用Matlab
### 强大的`R.matlab`包
一般工作流如下：

```r
library(R.matlab)
Matlab$startServer(matlab = "/Applications/MATLAB_R2021b.app/bin/matlab") # 启动matlab服务器，可能会较慢
# 创造与matlab交互的客户端对象，并看它的运行状态,此时是关闭的
matlab <- Matlab()
print(matlab)
# 打开运行
isOpen <- open(matlab)
print(matlab)
# matlab中运行脚本
evaluate(matlab, "A = 1+2;", "B = ones(2, 20);")
# 打印A的值
evaluate(matlab, "A")
# 将matlab中的值传到R中
data <- getVariable(matlab, c("A", "B"))
# 将R中的值传到matlab中
ABCD <- matrix(rnorm(10000), ncol = 100)
setVariable(matlab, ABCD = ABCD)
# 关闭与matlab间的连接
close(matlab)
```

### 其他的一些小办法
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

## R与Python的无缝对接
- 第一步，首先配置好环境

```r
library(reticulate)
# use_condaenv("D:/Anaconda3")
# 这是从官网下载python并安装的默认位置，我在spyder中也设定了调用这个python
# 我把这个命令也放在了.Rprofile文件中，免得每次调python都跑一下
use_python('/usr/local/bin/python3') 

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
py_config() # 可以查你有几个版本的python

# 想使用哪个版本的python
use_python('C:/Users/sheng/AppData/Local/Continuum/anaconda3/python.exe')
# 或者直接在.Rprofile文件中添加一行如：Sys.setenv(RETICULATE_PYTHON = "D:/usr/Anaconda3")
# 以配置路径

# 检查python可不可用
py_available()
# 检查模块可不可用
py_module_available('tushare')
```

### 如何在R中让python找到另外一个安装python包的位置

问题：在R中调用Python时，某个包不存在。于是你用`py_install()`函数进行了包的安装，默认情况下安装在`/Users/yangnay/.virtualenvs/r-reticulate/lib/python3.12/site-packages`中，但`import()`该包时怎么也无法导入。

你首先应该查看一下，当前使用的python是在哪些位置找python包，命令为`import sys; sys.path`，比如我就发现我的python在如下位置找包：
```python
[1] ""                                                                               
[2] "/Library/Frameworks/Python.framework/Versions/3.12/bin"                         
[3] "/Users/yangnay/elements2/RLibrary/reticulate/config"                            
[4] "/Library/Frameworks/Python.framework/Versions/3.12/lib/python312.zip"           
[5] "/Library/Frameworks/Python.framework/Versions/3.12/lib/python3.12"              
[6] "/Library/Frameworks/Python.framework/Versions/3.12/lib/python3.12/lib-dynload"  
[7] "/Library/Frameworks/Python.framework/Versions/3.12/lib/python3.12/site-packages"
[8] "/Users/yangnay/elements2/RLibrary/reticulate/python"        
```

这当然找不到刚才默认位置安装的包了。你要把你刚才默认安装包的那个位置告诉python，让它去那里找。方法就是：

1. 在Python能够找到的位置的site-packages目录中放置一个`.pth`文件，该文件包含你想要添加到Python路径的目录。
例如，创建一个名为mypackage.pth的文件，内容为"/Library/Frameworks/Python.framework/Versions/3.12/lib/python3.12/site-packages"
2. 然后将其放置在你的Python环境的site-packages目录中。

### `py_install()`安装了包，但无法调用

现在我比较喜欢去python的官网安装最新的python，然后去spyder的官网安装最新的spyder，然后开展数据分析。同时也经常从R语言中用`reticulate`调python。

通常python会在每个环境下，建立自己的包文件目录，从而避免了环境之间的包的影响。比如：

1. python本身有自己的包文件，往往在`C:\Users\admin\AppData\Local\Programs\Python\Python313\Lib\site-packages`目录中。

2. Spyder也有自己的包文件，往往在`C:\Users\admin\AppData\Local\spyder-6\Lib\site-packages`中。

3. 当你在R包`reticulate`中使用`py_install()`安装包时，它又会创造一个新的包文件，往往在`c:\users\admin\docume~1\virtua~1\r-reti~1\lib\site-packages`中。

如果你在R中调用python时，你要用`import sys; sys.path`看它在哪里找包，很可能不会在`c:\users\admin\docume~1\virtua~1\r-reti~1\lib\site-packages`里面找，这样，你当然无法在R中调用刚刚安装的包了。此时，你可以用前面的方法在R中调用python可以找到包的地方创造一个`.pth`文件以包含`c:\users\admin\docume~1\virtua~1\r-reti~1\lib\site-packages`的内容，这就搞定了。

**或者你直接修改R语言调用的python解释器**，比如
```
use_python('C:/Users/sheng/AppData/Local/Spyder-6/envs/spyder-runtime/python.exe')
# 或者直接在.Rprofile文件中添加一行如：Sys.setenv(RETICULATE_PYTHON = "C:/Users/sheng/AppData/Local/Spyder-6/envs/spyder-runtime")
```
那么，只要这个包在spyder里面能用，那么在R里面也就能用了。

### r与python的交互控制

```r
library(reticulate)
mydata = head(cars, n=15)

# 去往python控制台
repl_python()
import pandas as pd
# 载入数据集
travel = pd.read_excel(“AIR.xlsx”)
r.mydata.describe() # 利用r对象调用前面R中生成的变量
# 回到R控制台
exit

# 利用py对象调用python中生成的变量
py$travel
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

## 与C++的对接：`Rcpp`包
### `Rcpp`引入的C++中的标量、向量和矩阵类

- 在写C++时任何变量通常需要首先声明类型，标量类型包括`double, int, String, bool`。
- 向量类型包括`NumericVector, IntegerVector, CharacterVector, LogicalVector`。这些类型按我的理解，应该是类，对于类就有对应的方法，比如`.size()`方法就可以计算这个向量的长度。选取向量元素用`[]`。
- 此外也有对应矩阵类型(`NumericMatrix, IntegerMatrix, CharacterMatrix, LogicalMatrix`)。
  - 选取矩阵元素用`()`而不是`[]`
  - `.nrow(), .ncol()`计算矩阵维度

### 使用`sourceCpp`函数
- 单独的C++文件头应包含：
```c++
#include <Rcpp.h>
using namespace Rcpp;
// [[Rcpp::export]]
/*** R
# 这里可以跑R代码，从而可以简单测试上述C++语言
*/
```
- 如何从R语言的`list`类中提取数据？如果`mod`是从`lm`返回的`list`，提取它的残差同时转化为C++的向量，可用`as<NumericVector>(mod["residuals"])`。
### C++代码中调用R函数
这给我们提供了极大的方便。调用方式如下：
```c++
RObject callWithOne(Function f){
  return f(1);
}
```
  - 用`Function`声明R函数；
  - 用`RObject`表示R函数返回的类型；
  - 现在R中的函数`f`就可以在C++中用了；
