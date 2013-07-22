
## UI & Server

我们来把构建简单的Shiny应用程序的流程走一遍。shiny程序是个简单的目录，里面包括用户接口的定义、服务端脚本以及起支持作用的数据、脚本和其他资源。

构建应用程序之初，先建一个空目录，在这个目录里创建空文件`ui.R` 和 `server.R`。为了便于解释，我们假定你选择在~/shinyapp创建程序：

<pre><code>~/shinyapp
|-- ui.R
|-- server.R
</code></pre>

现在我们将在每个源文件中添加所需的最少代码。先定义用户接口，调用函数`pageWithSidebar`并传递它的结果到`shinyUI`函数：

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for miles per gallon application
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Miles Per Gallon"),

  sidebarPanel(),

  mainPanel()
))
{% endhighlight %}


三个函数 `headerPanel`、`sidebarPanel`和 `mainPanel` 定义了用户接口的不同区域。 程序将会叫做 "Miles Per Gallon"，所以在创建header panel的时候我们把它设置为标题。其他panel到目前为止还是空的。

我们来定义一个简单的服务端实现。我们调用`shinyServer`并传递给它一个函数，用来接收两个参数：`input`和`output`

#### server.R

{% highlight r %}
library(shiny)

# Define server logic required to plot various variables against mpg
shinyServer(function(input, output) {

})
{% endhighlight %}

服务端程序现在还是空的，不过之后我们会用它来定义输入和输出的关系。

我们来创建一个最小的Shiny应用程序。你可以调用`runApp`函数来运行这个程序：

{% highlight console %}
> library(shiny)
> runApp("~/shinyapp")
{% endhighlight %}

如果一切正常，你会在浏览器里看到如下图所示的应用程序：

![MPG Screenshot](screenshots/mpg-empty.png)

我们创建了一个可运行的shiny程序，尽管它还做不了什么。
下一节，我们会完善用户接口并实现服务端脚本，来完成这个应用程序。