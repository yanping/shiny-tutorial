

![More Widgets Screenshot](screenshots/more-widgets.png)

More widgets这个应用程序展示的是帮助文本和提交按钮，以及用嵌入HTML元素的方式来自定义格式。要运行例子，请键入：

{% highlight r %}
> library(shiny)
> runExample("07_widgets")
{% endhighlight %}

### UI增强

在这个例子里，我们更新了Shiny Text程序，增加了控件，调整了格式，特别地：

* 增加了`helpText`控件，用以在输入控件旁边添加解释文本。
* 增加了 `submitButton`控件，用来表示我们不想让输入和输出进行实时连接，而是想让用户点击按钮后才更新输出。这一特点在当计算过程非常复杂时是很有用的。
* 在输出面板上增加了 `h4` 元素（四级标题）。Shiny提供了大量的函数可直接往页面上添加HTML元素，包括标题、段落、链接等等。

下面更新后的用户接口界面的代码：

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for dataset viewer application
shinyUI(pageWithSidebar(

  # Application title.
  headerPanel("More Widgets"),

  # Sidebar with controls to select a dataset and specify the number
  # of observations to view. The helpText function is also used to 
  # include clarifying text. Most notably, the inclusion of a 
  # submitButton defers the rendering of output until the user 
  # explicitly clicks the button (rather than doing it immediately
  # when inputs change). This is useful if the computations required
  # to render output are inordinately time-consuming.
  sidebarPanel(
    selectInput("dataset", "Choose a dataset:", 
                choices = c("rock", "pressure", "cars")),

    numericInput("obs", "Number of observations to view:", 10),

    helpText("Note: while the data view will show only the specified",
             "number of observations, the summary will still be based",
             "on the full dataset."),

    submitButton("Update View")
  ),

  # Show a summary of the dataset and an HTML table with the requested
  # number of observations. Note the use of the h4 function to provide
  # an additional header above each output section.
  mainPanel(
    h4("Summary"),
    verbatimTextOutput("summary"),

    h4("Observations"),
    tableOutput("view")
  )
))
{% endhighlight %}


### 服务端脚本

和原来的Shiny Text的程序比，所有的变化都是在用户接口界面部分，服务端脚本保持不变。

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
