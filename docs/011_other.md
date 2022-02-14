
# 其他
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
