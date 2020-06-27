
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
Matlab$startServer() # 启动matlab服务器，可能会较慢
# 创造与matlab交互的客户端对象，并看它的运行状态
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


