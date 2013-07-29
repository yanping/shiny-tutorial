
![Sliders Screenshot](screenshots/sliders.png)

sliders这个例子展示的是滑动条（slider）控件的功能，包括产生动画效果。要运行这个例子，只需键入：

{% highlight console %}
> library(shiny)
> runExample("05_sliders")
{% endhighlight %}


### 定制化滑动条

slider控件功能非常强大，同时也可以定制化。它支持的特性有：

* 既可以输入单个的值，也可以输入一个范围。
* 自定义显示值的格式（比如，货币格式）
* 让滑动条在一定范围内自动滑动

滑动条控件是由调用`sliderInput`函数生成的。ui.R文件展示的是多种选项的滑动条。

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for slider demo application
shinyUI(pageWithSidebar(

  #  Application title
  headerPanel("Sliders"),

  # Sidebar with sliders that demonstrate various available options
  sidebarPanel(
    # Simple integer interval
    sliderInput("integer", "Integer:", 
                min=0, max=1000, value=500),

    # Decimal interval with step value
    sliderInput("decimal", "Decimal:", 
                min = 0, max = 1, value = 0.5, step= 0.1),

    # Specification of range within an interval
    sliderInput("range", "Range:",
                min = 1, max = 1000, value = c(200,500)),

    # Provide a custom currency format for value display, with basic animation
    sliderInput("format", "Custom Format:", 
                min = 0, max = 10000, value = 0, step = 2500,
                format="$#,##0", locale="us", animate=TRUE),

    # Animation with custom interval (in ms) to control speed, plus looping
    sliderInput("animation", "Looping Animation:", 1, 2000, 1, step = 10, 
                animate=animationOptions(interval=300, loop=T))
  ),

  # Show a table summarizing the values entered
  mainPanel(
    tableOutput("values")
  )
))
{% endhighlight %}


### 服务端脚本

slider应用程序的服务端是很简单的：它创建了个数据框，用来存放所有输入值，然后把它渲染到HTML表中：

#### server.R

{% highlight r %}
library(shiny)

# Define server logic for slider examples

shinyServer(function(input, output) {

  # Reactive expression to compose a data frame containing all of the values
  sliderValues <- reactive({

    # Compose data frame
    data.frame(
      Name = c("Integer", 
               "Decimal",
               "Range",
               "Custom Format",
               "Animation"),
      Value = as.character(c(input$integer, 
                             input$decimal,
                             paste(input$range, collapse=' '),
                             input$format,
                             input$animation)), 
      stringsAsFactors=FALSE)
  }) 

  # Show the values using an HTML table
  output$values <- renderTable({
    sliderValues()
  })
})
{% endhighlight %}
