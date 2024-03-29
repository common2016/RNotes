
# 其他
## 一句话Tips
- `officer`是一个强大的R与Word，PPT等交互的包。
- `geosphere::distm`根据经纬度算距离。
- `stats19`一个下载英国交通事故记录的数据库。
- `pacman`包管理包，如`p_load()`。
- `vitae`: 提供多个模板和功能以简化简历的制作和维护.
- `DiagrammeR`：R语言画流程图，网络图，太棒了。教程网址：http://rich-iannone.github.io/DiagrammeR/
- `progress`包可以显示进度条。更具体的例子见收藏或者帮助。

```r
library(progress)
pb <- progress_bar$new(total = 500) # 循环中的次数
for (i in 1:500){
    pb$tick()
}
```

- `swirl`可以编写交互式教程。
- `RSelenium`包终于在201901月出来了。

## R语言动态报告
用R语言写一个latex，简单步骤如下：

1. 新建一个RSweave文档，后缀名是`.Rnw`
2. 把以前的tex文档格式直接往里面copy就行了。
3. 只是涉及到R语言时，使用如下格式：
```r
<<echo = FALSE>>=
x <- 1
print (x)
@
```
4. 一些快捷键：
- ctrl  + alt + I : 插入代码块；
- ctrl + shift + enter: 运行当前代码块；
- ctrl + shift + K: 预览当前文档；
5. 中文状态下，要更改渲染引擎为`Tools-Global Options-Sweave-XeLaTex`
6. 如果你感兴趣于修改R代码的输出格式，你可以使用`knitr`包的修改方法，但此时，你要更改RStudio全局设定中`Sweave`标签中的渲染引擎为`knitr`，而不是`sweave`，同时，你要把代码` \SweaveOpts{concordance=TRUE}`注释掉，然后你要在文本的第一个代码块中进行设置，比如我的一个经常性的设置为，
```r
<<setup, echo = FALSE, include=FALSE>>=
library(knitr)
render_sweave() # 基准设置是sweave的设置
opts_chunk$set(message = FALSE, warning = FALSE) # 全局抑制警告和信息输出
knitr::knit_hooks$set(source = function(x, options) {
  x <- paste('> ',x,sep = '') # 代码输入部分添加 > 符号
    paste("\\begin{lstlisting}[language=R,breaklines,aboveskip=0em,belowskip =0em,commentstyle=\\color{gray},basicstyle = {\\ttfamily\\color{RoyalBlue}},keywordstyle = \\color{RoyalBlue},stepnumber=2]\n", x,
        "\\end{lstlisting}\n", sep = "") # lstlisting格式的更改
})
@
```
