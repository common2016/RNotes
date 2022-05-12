
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


## 递交R Journal的笔记


第一，这个网址https://journal.r-project.org/dev/submissions.html 给出了整体递交的流程，新的递交都推荐这种方式，因为它包含了PDF和html两种格式。

第二，这个网址 https://rjournal.github.io/rjtools/articles/format-details.html 给出了利用RMarkdwon制作图、表、公式在R Journal中操作的很多细节。

第三，author-guide.pdf文件给出了R Journal特有的latex命令，如`\pkg,\CRARpkg,\code,\samp`等。

第四，书写完成，使用`rjtools::initial_check_article()`检查递交文件是否合乎规范。

其他注意点：

- 在重现论文结果时，不要使用`rm(list=ls())`和`setwd()`这两种语句，会被编辑退回。


### `rjtools`包提供的Rmd文件编写

- 参考文献都放在bib文件中，正文引用时如果要显示小括号，则写为`[@bernake2005;@stock2005]`，无小括号则写为`@bernake2005`。注意，bib,Rmd,tex三个文件必须具有同样的名字。比如`chen-chen-ouyang.bib,chen-chen-ouyang.Rmd,chen-chen-ouyang.tex`.
- 增加简单的markdown表格要在YAML中添加`preamble: \usepackage{longtable}`。
- 在正文对表或图的引用`\@ref(fig:plot-1)`或`\@ref(tab:t-1)`，注意表或图标签用`-`而不是下划线。
- 该包对图和表的嵌入是分latex和html两个部分来实现，你如果想在网页和PDF中都体现表格或者图，意味着你要写两个语句块。语句块的写法以及引用方式可以参加该包的模板。
- `knitr::kable()`函数生成表格是非常灵活的。这里我们给一个例子。可以看到矩阵的宽度安排是灵活的。另外，如果你的表格包含`$`等符号，可以设置`escape = FALSE`进行逃逸。

```r
# ```{r, eval = knitr::is_latex_output()}
# knitr::kable(tb1lt, format = "latex", caption = "FAVAR Function Description", align = c('l','p{34em}'), escape = FALSE)
# ```
```


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
