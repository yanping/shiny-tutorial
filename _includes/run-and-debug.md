

## 运行和调试

在整个教程中，你会发现我们都是调用`runApp`来运行例子程序。
这个函数会启动shiny程序并打开浏览器以便查看程序。在shiny程序运行的过程中，R控制台的交互将被阻断，也就是说，不能在控制台运行命令。

要停止shiny程序，你只需中断R，有两种方式：（1）在R的任何前端里按下Escape键；（2）点击R环境里提供的停止按钮。

### 在独立进程里运行

如果你不想在运行shiny程序的时候阻断访问控制台，则可以在单独的进程中运行shiny程序。你可以打开终端窗口或者控制台窗口，然后执行下面的命令：

{% highlight console %}
R -e "shiny::runApp('~/shinyapp')"
{% endhighlight %}

默认情况下， `runApp` 会打开8100端口。如果你用的是默认设置，你可以在浏览器里通过地址[http://localhost:8100](http://localhost:8100) 来连接应用程序。

下面我们会讨论调试shiny程序的技术，包括停止执行程序和查看当前环境变量的能力。为了把这些技术和在独立的终端会话相结合，你必须用交互方式运行R（也就是，首先输入“R”来启动R会话然后在会话中执行`runApp`）。

### 实时重载

当你修改用户接口定义或者服务端脚本的时候，你不必关闭并重启应用程序以查看更改后的效果。只需保存更改，重新加载浏览器就可以查看更新后的程序。

有个条件是这样的：当浏览器重新加载，会引发shiny检查ui.R和server.R的时间戳，以判断这两个文件是否需要重新加载。如果其他脚本或数据文件发生改变，shiny是不会知道的，所以这时要关闭并重启应用程序来查看相应的更改。


### 调试技巧

#### 打印 

有好几种技巧可以用来调试shiny程序。第一种是增加[cat](http://stat.ethz.ch/R-manual/R-devel/library/base/html/cat.html)函数的调用，这样可以在适当的地方打印诊断信息。例如，下面两条调用就是用来打印标准输出和标准错误的信息：

{% highlight r %}
cat("foo\n")
cat("bar\n", file=stderr())
{% endhighlight %}

#### 使用调试浏览器

第二种方式是增加[browser](http://stat.ethz.ch/R-manual/R-devel/library/base/html/browser.html)函数的显式调用，来中断程序执行，并查看调用browser时所处的环境。注意，使用browser需要你从交互式会话中启动应用程序（这与上面提到的用R -e的方式相反）。

例如，在代码的某个地方无条件地停止执行：

{% highlight r %}
# Always stop execution here
browser() 
{% endhighlight %}

你也可以用这种方法在特定条件下停止执行代码。例如，当用户选择"Transmission"作为变量的时候停止MPG程序：

{% highlight r %}
# Stop execution when the user selects "am"
browser(expr = identical(input$variable, "am"))
{% endhighlight %}

#### 建立一个自定义错误处理器
你可以设置R的 &quot;error&quot; 选项，使得当错误发生的时候，自动进入调试浏览器：

{% highlight r %}
# Immediately enter the browser when an error occurs
options(error = browser)
{% endhighlight %}

另一种方法，你可以设置[recover](http://stat.ethz.ch/R-manual/R-devel/library/utils/html/recover.html)函数做为错误处理器，它可以打印一个调用列表，并允许你在堆栈的任何位置查看：

{% highlight r %}
# Call the recover function when an error occurs
options(error = recover)
{% endhighlight %}

如果你想自动地对每个会话设置error选项，你可以用[R Startup](http://stat.ethz.ch/R-manual/R-patched/library/base/html/Startup.html)这篇文章描述的方法来修改.Rprofile文件。




