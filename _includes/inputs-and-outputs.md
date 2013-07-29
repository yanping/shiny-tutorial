

## 输入和输出

### 在Sidebar上添加输入

我们要使用R内置的datasets包中的mtcars数据构建程序， 允许用户查看箱线图来研究英里每加仑（miles-per-gallon，简称MPG） 和其他三个变量（气缸，变速器，齿轮）之间的关系。

我们想提供一种方式来选择绘制MPG与哪个变量的图形，也提供了个选项，可选择绘图时包含或剔除异常值。为了完成这个目标，我们要往sidebar上加两个元素，一个是`selectInput`，用来指定变量，另一个是`checkboxInput`，用来控制是否显示异常值。添加这些元素后的用户接口定义如下所示：

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for miles per gallon application
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Miles Per Gallon"),

  # Sidebar with controls to select the variable to plot against mpg
  # and to specify whether outliers should be included
  sidebarPanel(
    selectInput("variable", "Variable:",
                list("Cylinders" = "cyl", 
                     "Transmission" = "am", 
                     "Gears" = "gear")),

    checkboxInput("outliers", "Show outliers", FALSE)
  ),

  mainPanel()
))
{% endhighlight %}

如果在做了这些修改之后你再运行该程序，你会在sidebar看到两个用户输入：

![MPG Screenshot](screenshots/mpg-with-inputs.png)


### 创建服务端脚本

我们需要定义程序的服务端脚本，用来接收输入，并计算输出。文件server.R如下所示，它说明了下面几个重要的概念：

* 使用`input`对象的组件来访问输入，并通过向`output`对象的组件赋值来生成输出。
* 在启动的时候初始化的数据可以在应用程序的整个生命周期中被访问
* 使用反应表达式来计算被多个输出共享的值

shiny服务端的脚本的基本任务是定义输入和输出之间的关系。脚本访问输入值，然后计算，接着向输出的组件赋以反应表达式。

下面是全部服务端脚本的代码（注释更详尽地讲解了实现技巧）：

#### server.R

{% highlight r %}
library(shiny)
library(datasets)

# We tweak the "am" field to have nicer factor labels. Since this doesn't
# rely on any user inputs we can do this once at startup and then use the
# value throughout the lifetime of the application
mpgData <- mtcars
mpgData$am <- factor(mpgData$am, labels = c("Automatic", "Manual"))

# Define server logic required to plot various variables against mpg
shinyServer(function(input, output) {

  # Compute the forumla text in a reactive expression since it is 
  # shared by the output$caption and output$mpgPlot expressions
  formulaText <- reactive({
    paste("mpg ~", input$variable)
  })

  # Return the formula text for printing as a caption
  output$caption <- renderText({
    formulaText()
  })

  # Generate a plot of the requested variable against mpg and only 
  # include outliers if requested
  output$mpgPlot <- renderPlot({
    boxplot(as.formula(formulaText()), 
            data = mpgData,
            outline = input$outliers)
  })
})
{% endhighlight %}

shiny用`renderText`和`renderPlot`生成输出（而不是直接赋值），这样做让程序成为反应式的。这一层封装返回特殊的表达式，只有当其所依赖的值改变的时候才会重新执行。这就使shiny在输入值发生改变时自动更新输出。

### 展示输出

服务端脚本给两个输出赋值：`output$caption`和`output$mpgPlot`。为了让用户接口能显示输出，我们需要在主UI面板上添加一些元素。

在下面修改后的用户接口定义中，你可以看到，我们用h3元素添加了说明文字，并用`textOutput`函数添加了其中的文字，还调用了`plotOutput`函数渲染了图形。

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for miles per gallon application
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Miles Per Gallon"),

  # Sidebar with controls to select the variable to plot against mpg
  # and to specify whether outliers should be included
  sidebarPanel(
    selectInput("variable", "Variable:",
                list("Cylinders" = "cyl", 
                     "Transmission" = "am", 
                     "Gears" = "gear")),

    checkboxInput("outliers", "Show outliers", FALSE)
  ),

  # Show the caption and plot of the requested variable against mpg
  mainPanel(
    h3(textOutput("caption")),

    plotOutput("mpgPlot")
  )
))
{% endhighlight %}

运行应用程序，就可以显示它的最终形式，包括输入和动态更新的输出：

![MPG Screenshot](screenshots/mpg-with-outputs.png)

现在我们完成了一个简单的程序，接下来可能想做一些修改。下一个主题将介绍编辑、运行、调试shiny程序的流程。
