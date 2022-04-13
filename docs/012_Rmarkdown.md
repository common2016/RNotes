
# RMarkdown

## Rmarkdown的语法格式

- 写脚标
```
这是一个脚标[^1]

% 文中任意一处，一般放在文后
[^1]:我是一个脚标。
```
- 公式编号和引用。代码如下，
```
$$x = 1+2\tag{1}\label{eq1}$$

根据公式$\eqref{eq1}$式……
```
格式表现如下，
$$
x=1+2\tag{1}\label{eq1}
$$
根据公式$\eqref{eq1}$式……

### `rjtools`包提供的Rmd文件编写

- 参考文献都放在bib文件中，正文引用时如果要显示小括号，则写为`[@bernake2005;@stock2005]`，无小括号则写为`@bernake2005`。
- 增加简单的markdown表格要在YAML中添加`preamble: \usepackage{longtable}`。
- 在正文对表或图的引用`\@ref(fig:plot-1)`或`\@ref(tab:t-1)`，注意表或图标签用`-`而不是下划线。

## R语言写latex

简单步骤如下：

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
