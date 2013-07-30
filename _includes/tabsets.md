

![Tabsets Screenshot](screenshots/tabsets.png)

示例程序Tabsets展示的是如何用选项卡（tabs）来组织输出。要运行这个例子，就执行下面的命令： 

{% highlight console %}
> library(shiny)
> runExample("06_tabsets")
{% endhighlight %}


### 选项卡面板（Tab Panels）

选项卡（tabsets）是由调用`tabsetPanel`函数创建的，在这函数中，又需要用`tabPanel`函数创建选项（tab）列表。每一个选项卡面板是由输出元素组成的，这些元素在选项卡中垂直排列。

在这个例子中，我们修改了原来的Hello Shiny程序，增加了一个摘要和数据表，两者分别渲染到它们各自的选项卡中。下面就是用户接口的代码：

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for random distribution application 
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Tabsets"),

  # Sidebar with controls to select the random distribution type
  # and number of observations to generate. Note the use of the br()
  # element to introduce extra vertical spacing
  sidebarPanel(
    radioButtons("dist", "Distribution type:",
                 list("Normal" = "norm",
                      "Uniform" = "unif",
                      "Log-normal" = "lnorm",
                      "Exponential" = "exp")),
    br(),

    sliderInput("n", 
                "Number of observations:", 
                 value = 500,
                 min = 1, 
                 max = 1000)
  ),

  # Show a tabset that includes a plot, summary, and table view
  # of the generated distribution
  mainPanel(
    tabsetPanel(
      tabPanel("Plot", plotOutput("plot")), 
      tabPanel("Summary", verbatimTextOutput("summary")), 
      tabPanel("Table", tableOutput("table"))
    )
  )
))
{% endhighlight %}


### 选项卡和反应式数据（Reactive Data）

将选项卡引入用户接口的时候，应该强调为共享数据创建反应表达式的重要性。在这个例子中，每个选项卡都提供了对数据集的查看方式。如果对数据集的处理比较费时，那么用户接口的定义可能变得很慢。下面的服务端脚本展示的是如何用反应表达式一次性计算数据，其结果被三个选项卡所共享。

#### server.R

{% highlight r %}
library(shiny)

# Define server logic for random distribution application
shinyServer(function(input, output) {

  # Reactive expression to generate the requested distribution. This is 
  # called whenever the inputs change. The renderers defined 
  # below then all use the value computed from this expression
  data <- reactive({  
    dist <- switch(input$dist,
                   norm = rnorm,
                   unif = runif,
                   lnorm = rlnorm,
                   exp = rexp,
                   rnorm)

    dist(input$n)
  })

  # Generate a plot of the data. Also uses the inputs to build the 
  # plot label. Note that the dependencies on both the inputs and
  # the 'data' reactive expression are both tracked, and all expressions 
  # are called in the sequence implied by the dependency graph
  output$plot <- renderPlot({
    dist <- input$dist
    n <- input$n

    hist(data(), 
         main=paste('r', dist, '(', n, ')', sep=''))
  })

  # Generate a summary of the data
  output$summary <- renderPrint({
    summary(data())
  })

  # Generate an HTML table view of the data
  output$table <- renderTable({
    data.frame(x=data())
  })
})
{% endhighlight %}
