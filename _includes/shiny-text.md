
![Tabsets Screenshot](screenshots/shiny-text.png)

Shiny Text这个应用程序展示的是直接打印R对象，以及用HTML表格展示数据框。要运行例子程序，只需键入： 

{% highlight console %}
> library(shiny)
> runExample("02_text")
{% endhighlight %}

前面那个例子里用一个滑动条来输入数值，并且输出图形。而这个例子更进了一步：有两个输入，以及两种类型的文本输出。

如果你改变观测个数， 将会发现Shiny应用程序的一大特性：输入和输出是结合在一起的，并且“实时”更新运算结果（就像Excel一样）。 在这个例子中，当观测个数发生变化时，只有表格更新，而不需要重新加载整个页面。

下面是用户界面定义的代码。请注意，`sidebarPanel`和`mainPanel`的函数调用中有两个参数（对应于两个输入和两个输出）

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for dataset viewer application
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Shiny Text"),

  # Sidebar with controls to select a dataset and specify the number
  # of observations to view
  sidebarPanel(
    selectInput("dataset", "Choose a dataset:", 
                choices = c("rock", "pressure", "cars")),

    numericInput("obs", "Number of observations to view:", 10)
  ),

  # Show a summary of the dataset and an HTML table with the requested
  # number of observations
  mainPanel(
    verbatimTextOutput("summary"),

    tableOutput("view")
  )
))
{% endhighlight %}

服务端的程序要稍微复杂一点。现在，我们创建：

*  一个反应性表达式来返回用户选择的相应数据集。
*  还有两个渲染表达式（rendering expressions，分别是`renderPrint` 和`renderTable`），以返回 `output$summary` 的 `output$view` 的值。 

这些表达式和第一个例子中的 `renderPlot` 运作方式类似：通过声明渲染表达式，你也就告诉了shiny，一旦渲染表达式所依赖的值（在这里例子中是两个用户输入值的任意一个：`input$dataset` 或 `input$n`）发生改变，表达式就会执行。。

#### server.R

{% highlight r %}
library(shiny)
library(datasets)

# Define server logic required to summarize and view the selected dataset
shinyServer(function(input, output) {

  # Return the requested dataset
  datasetInput <- reactive({
    switch(input$dataset,
           "rock" = rock,
           "pressure" = pressure,
           "cars" = cars)
  })

  # Generate a summary of the dataset
  output$summary <- renderPrint({
    dataset <- datasetInput()
    summary(dataset)
  })

  # Show the first "n" observations
  output$view <- renderTable({
    head(datasetInput(), n = input$obs)
  })
})
{% endhighlight %}


我们已经介绍了一些反应性表达式的用法，但是并没有真正解释它们是如何运作的。下一个例子会以此为基础进一步讲解Shiny中反应性表达式的用法。