
![Hello Shiny Screenshot](screenshots/hello-shiny.png)

Hello Shiny是个简单的应用程序， 这个程序可以生成正态分布的随机数，随机数个数可以由用户定义，并且绘制这些随机数的直方图. 要运行这个例子，只需键入： 

{% highlight r %}
library(shiny)
runExample("01_hello")
{% endhighlight %}

Shiny应用程序分为两个部分：用户界面定义和服务端脚本。这两部分的源代码将在下面列出。

在教程的后续章节，我们将解释代码的细节并讲解如何用“被动式”表达式来生成输出。现在，就尝试运行一下例子程序，浏览一下源代码，以获得对shiny的初始印象。也请认真阅读注释。

用户界面是在源文件ui.R中定义的：

#### ui.R


{% highlight r %}
library(shiny)

# Define UI for application that plots random distributions 
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Hello Shiny!"),

  # Sidebar with a slider input for number of observations
  sidebarPanel(
    sliderInput("obs", 
                "Number of observations:", 
                min = 0, 
                max = 1000, 
                value = 500)
  ),

  # Show a plot of the generated distribution
  mainPanel(
    plotOutput("distPlot")
  )
))
{% endhighlight %}

下面列出了服务端的代码。从某种程度上说，它很简单——生成给定个数的随机变量， 然后将直方图画出来。不过，你也注意到了，返回图形的函数被 `renderPlot`包裹着。函数上面的注释对此做出了一些解释，不过如果你觉得还是搞不明白，不用担心——后面我们将更进一步解释这个概念。

#### server.R

{% highlight r %}
library(shiny)

# Define server logic required to generate and plot a random distribution
shinyServer(function(input, output) {

  # Expression that generates a plot of the distribution. The expression
  # is wrapped in a call to renderPlot to indicate that:
  #
  #  1) It is "reactive" and therefore should be automatically 
  #     re-executed when inputs change
  #  2) Its output type is a plot 
  #
  output$distPlot <- renderPlot({

    # generate an rnorm distribution and plot it
    dist <- rnorm(input$obs)
    hist(dist)
  })
})
{% endhighlight %}

下一个例子将展示其他输入控件的用法，以及生成文本输出的被动式函数的用法。
