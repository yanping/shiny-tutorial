

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
* Accessing input using slots on the `input` object and generating output by assigning to slots on the `output` object.
* Initializing data at startup that can be accessed throughout the lifetime of the application.
* Using a reactive expression to compute a value shared by more than one output.

The basic task of a Shiny server script is to define the relationship between inputs and outputs. Our script does this by accessing inputs to perform computations and by assigning reactive expressions to output slots. 

Here is the source code for the full server script (the inline comments explain the implementation technqiues in more detail):

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

The use of `renderText` and `renderPlot` to generate output (rather than just assigning values directly) is what makes the application reactive. These reactive wrappers return special expressions that are only re-executed when their dependencies change. This behavior is what enables Shiny to automatically update output whenever input changes.


### 展示输出

The server script assigned two output values: `output$caption` and `output$mpgPlot`. To update our user interface to display the output we need to add some elements to the main UI panel. 

In the updated user-interface definition below you can see that we've added the caption as an h3 element and filled in it's value using the `textOutput` function, and also rendered the plot by calling the `plotOutput` function:

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


Running the application now shows it in its final form including inputs and dynamically updating outputs:

![MPG Screenshot](screenshots/mpg-with-outputs.png)

现在我们完成了一个简单的程序，接下来可能想做一些修改。下一个主题将介绍编辑、运行、调试shiny程序的流程。

