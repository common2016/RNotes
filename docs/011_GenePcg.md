
# 创建包的一些建议
## 一句话Tips
- 要输出函数，记得在注释中书写`@export`
- 从RStudio中创建一个包项目
- 不再使用`library()`或者`require()`，因为这会改变搜索路径。而是在控制台输入`usethis::use_package('magrittr',type = 'Imports',min_version = NULL)`执行一下。该命令在DESCRIPTION文件中添加如下语句：
```
Imports:
    ggplot2,
    tidyverse
```

**这个语句的作用只是保证了在安装包时，这些包必须被安装**。因此，你在写函数时，要调用这个包，需要使用`pdg::fun()`。或者在函数文档中添加注释如`@import magrittr`（不建议该操作，可能会改变全局环境，该操作实际上修改了NAMESPACE文件）。我经常使用这个函数`#' @importFrom magrittr '%>%' `。
- 执行`devtools::document()`以产生函数的说明文档。
- 执行`devtools::load_all()`将source`R/`目录下所有文件
- `check()`在本地整个地检验包是否可行，而`devtools::check_win_*()`族函数将其提交至CRAN服务器进行检查，并在10-20分钟后通过邮件返回你一个结果。


## 使用自己的数据集
- 如果你想使用一个脚本来创造一个数据集，然后这个数据集在你发布的包中使用。那么首先执行命令`usethis::use_data_raw('MySet')`，它有三个功能：
    1. 生成`data-raw`子目录。
    2. 在`data-raw`目录下生成脚本`MySet.R`，该脚本最后一行是`usethis::use_data(MySet)`,该语句是将`MySet.R`脚本中创造的数据，以`rda`的格式保存到`data`目录中。
    3. 在`.Rbuildignore`中添加语句使得在建立包时，`data-raw`里面的东西不会被包含进将来要发布的包中。
- 建立数据文档和建立函数文档一样，你也只需要在`R/`下面建立一个R文件，该文件只包含一个写着数据名字的字符串，然后前面是描述。一般数据集文档包含两个标签`@format`和`@source`。
- 非R语言格式的数据文件放到`inst/`下面。

如果你有一个数据集，只是在你的函数内部调用，你可以用`use_data(myvar,overwrite = T,internal = T)`命令，它在`R/`下生成数据文件`sysdata.rdata`，它包含了变量`myvar`。然后你在函数中直接调用该变量。

## 测试
要提交包到CRAN必须有测试。执行`devtools::testhat()`做了三件事：
- 创建`tests/testthat`目录；
- 在Description的Suggests域增加testthat;
- 创建一个文件`tests/testthat.R`。这个文件不要动。通常是在`tests/testthat/`目录下面建立类似这样的测试文件(文件名总以`test`开始)，

```r
context("string length")
text_that('',{
    expect_equal(.,.)
    expect_equal(.,.)
})
```
当你完成这些工作以后，你可以简单的输入`devtools::test()`来一次性检查测试是否成功。

## 泛型函数
当你拓展一个泛型函数的时候，需要注意：

1. 函数末端返回`NextMethod()`。

```r
t.data.frame <- function(x)
{
    x <- as.matrix(x)
    NextMethod("t")
}
```
2. 方法要包含原始泛型的所有参数，顺序也要一致。
3. 缺省设置也要和原始泛型一致。

## 其他
- 可以使用`devtools::use_readme_rmd()`来创建README.Rmd并将其添加到.Rbuildignore，这将可以在readme中生成R代码。

- 使用`devtools::use_travis()`建立一个基本的`.travis.yml`配置文件，以便以后每次推送都自动检查是否符合包的规范。当然，这需要在traivis网站中打开这个项目的开关。
- 包中调用`ggplot2::aes()`时，通常会报错没有显式绑定全局变量，因此最好使用`ggplot2::aes_string()`。
- `@example /data/stage/chenpu/a.R`是把单独文件中的例子添加到函数文档中，而`@examples`则是直接添加。



