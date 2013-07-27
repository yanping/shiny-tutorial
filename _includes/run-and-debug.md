

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

When you make changes to your underlying user-interface definition or server script you don't need to stop and restart your application to see the changes. Simply save your changes and then reload the browser to see the updated application in action.

One qualification to this: when a browser reload occurs Shiny explicitly checks the timestamps of the ui.R and server.R files to see if they need to be re-sourced. If you have other scripts or data files that change Shiny isn't aware of those, so a full stop and restart of the application is necessary to see those changes reflected.

### 调试技巧

#### 打印 
There are several techniques available for debugging Shiny applications. The first is to add calls to the [cat](http://stat.ethz.ch/R-manual/R-devel/library/base/html/cat.html) function which print diagnostics where appropriate. For example, these two calls to cat print diagnostics to standard output and standard error respectively:

{% highlight r %}
cat("foo\n")
cat("bar\n", file=stderr())
{% endhighlight %}

#### 使用调试浏览器
The second technique is to add explicit calls to the [browser](http://stat.ethz.ch/R-manual/R-devel/library/base/html/browser.html) function to interrupt execution and inspect the environment where browser was called from. Note that using browser requires that you start the application from an interactive session (as opposed to using R -e as described above).

For example, to unconditionally stop execution at a certain point in the code:

{% highlight r %}
# Always stop execution here
browser() 
{% endhighlight %}

You can also use this technique to stop only on certain conditions. For example, to stop the MPG application only when the user selects "Transmission" as the variable:

{% highlight r %}
# Stop execution when the user selects "am"
browser(expr = identical(input$variable, "am"))
{% endhighlight %}

#### 建立一个自定义错误处理器
You can also set the R &quot;error&quot; option to automatically enter the browser when an error occurs:

{% highlight r %}
# Immediately enter the browser when an error occurs
options(error = browser)
{% endhighlight %}

Alternatively, you can specify the [recover](http://stat.ethz.ch/R-manual/R-devel/library/utils/html/recover.html) function as your error handler, which will print a list of the call stack and allow you to browse at any point in he stack:

{% highlight r %}
# Call the recover function when an error occurs
options(error = recover)
{% endhighlight %}

If you want to set the error option automatically for every R session, you can do this in your .Rprofile file as described in this article on [R Startup](http://stat.ethz.ch/R-manual/R-patched/library/base/html/Startup.html).




