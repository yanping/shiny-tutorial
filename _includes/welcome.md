

# Shiny简介

Shiny是RStudio公司开发的新包，有了它，可以用R语言轻松开发交互式web应用。

想查看更详细的介绍和实例，请访问[Shiny的官方主页](http://www.rstudio.com/shiny)。

### 特性

* 只用几行代码就可以构建有用的web应用程序&mdash;不需要用JavaScript。
* Shiny应用程序会自动刷新计算结果，这与电子表格实时计算的效果类似。 当用户修改输入时，输出值自动更新，而不需要在浏览器中手动刷新。
* Shiny用户界面可以用纯R语言构建，如果想更灵活，可以直接用HTML、CSS和JavaScript来写。
* 可以在任何R环境中运行（R命令行、Windows或Mac中的Rgui、ESS、StatET、RStudio等）
* 基于[Twitter Bootstrap](http://twitter.github.com/bootstrap)的默认UI主题很吸引人。
* 高度定制化的滑动条小工具（slider widget），内置了对动画的支持。
* 预先构建有输出小工具，用来展示图形、表格以及打印输出R对象。
* 采用[websockets](http://illposed.net/websockets.html)包，做到浏览器和R之间快速双向通信。
* 采用[被动式（reactive）](http://en.wikipedia.org/wiki/Reactive_programming)编程模型，摒弃了繁杂的 事件处理代码，这样你可以集中精力于真正关心的代码上。
* 开发和分发你自己的Shiny小工具，其他开发者也可以非常容易地加到自己的应用中（即将面市！）

### 安装

Shiny可以从CRAN获取， 所以你可以用通常的方式来安装，在R的命令行里输入：

```r
install.packages("shiny")
```

### Let's Go!

本教程涵盖了Shiny的基础知识，提供了详尽的案例来展示它的各种特性。点击Next按钮对Shiny说hello吧！
