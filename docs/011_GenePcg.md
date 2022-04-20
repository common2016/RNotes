
# 创建包的一些建议
## 一句话Tips

- 在R包结构下做分析，要注意，在函数中调用其他包的函数时，要记得先用`use_package()`函数来添`Imports`域，否则会有一些意想不到的问题。比如在使用`data.table`时就会出现类似`Error in `[.data.frame`(x, i, j) : undefined columns selected`这样的问题。
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
#
# 你可以在此写下需要用到的代码，比如用你在包中编制的函数跑一个结果出来
#
text_that('',{
    expect_equal(.,.) # 比较前后两个数值是否一致
    expect_equal(.,.)
    expect_ture(object) # 这个函数有着更加广泛的用途，比如可以检验a < b
})
```
当你完成这些工作以后，你可以简单的输入`devtools::test()`来一次性检查测试是否成功。

## 泛型函数
当你拓展一个泛型函数的时候，需要注意：

1. 如果当前函数处理不了这个对象，可在函数末端返回`NextMethod('your_method')`。`your_method`方法的输入参数与当前方法相同，该对象最终由`your_method`来处理。

```r
t.data.frame <- function(x)
{
    x <- as.matrix(x)
    NextMethod("t")
}
```
2. 方法要包含原始泛型的所有参数，顺序也要一致。比如`summary`这个内置的泛型函数的参数为`object`和`...`，那么当你拓展该函数的时候，这两个参数的名称是不能改的。
3. 缺省设置也要和原始泛型一致。

## 其他

- 如果在例子或vignette中使用了某个包，但在函数中并未使用它，那就在`Description`文件的`Suggests`域添加该包。不添加该包,`check()`会报警告，在`Imports`域添加，`rhub::check_on_linux()`会报Note。可用如下函数添加，也可以直接修改`Description`文件。

```r
usethis::use_package(your_pac, type = 'Suggests')
```

- `spell_check()`检查拼写错误。
- 可以使用`devtools::use_readme_rmd()`来创建README.Rmd并将其添加到.Rbuildignore，这将可以在readme中生成R代码。

- 使用`devtools::use_travis()`建立一个基本的`.travis.yml`配置文件，以便以后每次推送都自动检查是否符合包的规范。当然，这需要在traivis网站中打开这个项目的开关。
- 包中调用`ggplot2::aes()`时，通常会报错没有显式绑定全局变量（`no visible binding for global variable ‘.’`），因此最好使用`ggplot2::aes_string()`。但作者明确表示`ggplot2::aes_string()`将来会折旧。因此新的解决办法是，

```r
#' @importFrom rlang .data
my_func <- function(data) {
    ggplot(data, aes(x = .data$x, y = .data$y))
}
```
实际上这种问题还会出现在`tidyverse`中管道传输时的报错，那么通过在前面引入`#' @importFrom rlang .data`语句，同时把后面的占位符改成`.data`是解决这个全局绑定的好办法。

- `@example /data/stage/chenpu/a.R`是把单独文件中的例子添加到函数文档中，而`@examples`则是直接添加。



