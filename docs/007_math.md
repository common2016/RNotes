
# 数学计算 {#math}
## 一句话Tips
- `matlab::meshgrid`可能有助于你画3D图。
- `crossprod(x,y)`的意思是$x'y$，`tcrossprod(x,y)`的意思是$xy'$。在OLS估计时还蛮省事。
- `matlab`包模拟了matlab软件中的许多矩阵函数。
- 克罗内克积使用`%x%`或者`kronecker`。
- `numDeriv`里面的`hession`计算海塞矩阵，它的逆的负数，就是极大似然估计的标准差。里面的`grad(fun, par)` 计算梯度，也就是一阶导。
- `Matrix::bdiag(A,A)`生成以两个A为对角元素的分块对角矩阵。
- `Mod`计算复数的模，不同于`mod`用来整除。
-  `caracas`: 通过提供对Python SymPy库的访问来实现计算机代数，从而使以符号方式解方程、寻找符号积分、符号和和其他重要量成为可能。
-  `calculus`: 针对数值和符号演算提供了C++优化函数，包括符号算术、张量演算、泰勒级数展开、多元埃尔米特多项式等.
-  `rootSolve`提供了非线性方程(组)的解，以及微分方程的稳态解的形式。

```r
fun <- function (x) cos(2*x)^3
curve(fun, 0, 8) # 先画个图看看解大约在哪里
abline(h = 0, lty = 3) # 把0轴搞出来
uni <- uniroot(fun, c(0, 8))$root # 此时求解
```
方程组的根求解用`multiroot`。

## 数值优化
- 在Linux系统下，安装`nloptr`包，需要先在linux系统中安装`nlopt`包。
```
yum install nlopt nlopt-devel
```

- **无约束优化**。`optimx`一个优化包，经常用它的`optimx`函数。但是它是无约束优化，尽管可以包含上下界的约束(盒子约束)。

```r
para <- list(R1 = 0.1, phi1 = 0.8, H1 = 0.3, A1 = 0.8, intcp = 1) %>% as.numeric()
# par就是要优化的参数的初值
# fn就是要优化的函数，譬如似然函数，这个函数可以包含多个参数，
#    如这里的Y,X,r等。然后那个没写进来的参数就是要优化的参数。
#    lnlik <- function(Y = Y, X = X,para = para, r = 1){...}
# gr和hess如果必要，可以包含进来。
# lower,upper就是搜寻的上下界。
a <- optimx(par = para, fn = lnlik, Y = Y, X = X, r = 1, 
            lower = c(0.01,0.01,0.01,0.01,-Inf), 
            upper = c(Inf, 0.99,Inf,Inf,Inf),control = list(all.methods = T))
gHgen(par, fn) # 创造得分矩阵，海塞矩阵。
```
- **有约束优化**。`stats::constrOptim`和`alabama::constrOptim.nl`都可进行有约束的优化，后者是对前者的一个强化，不仅在算法上更牛逼，也可以放入非线性约束。后者是用函数如`hin(x)<=0`的形式来表达约束，需要理解的是，约束函数中的参数一定要与目标函数的参数一致，即便约束函数没有用到目标函数的参数。

```r
# 比如我的目标函数如下
fn <- function (x,a,b){
    ...
}
# 约束函数如下。一定要把a和b写进去，即便函数中未用到它
hin <- function(x, a, b){
    ...
}
# 优化函数如下
constrOptim.nl(par, fn, gr = NULL, 
hin = NULL, hin.jac = NULL, heq = NULL, heq.jac = NULL, 
control.outer=list(), control.optim = list(), ...)
```
一旦得到优化函数，就可以调用`numDeriv::hessian(fn, par)`来计算海塞矩阵，从而得到标准误之类的。
