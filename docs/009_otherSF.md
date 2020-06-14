
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

- R语言调用Matlab

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
use_condaenv("D:/Anaconda3")
 
py_config()#安装的python版本环境查看，显示anaconda和numpy的详细信息。
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
