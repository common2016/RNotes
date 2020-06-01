

# 原生的R {#rawR}
## 一句话Tips
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
