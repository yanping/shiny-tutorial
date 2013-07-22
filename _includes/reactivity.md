

![Reactivity Screenshot](screenshots/reactivity.png)

Reactivity的示例程序与Hello Text很相似，但是用到了反应式编程里更多细节的概念，要运行该例子，请键入： 

{% highlight console %}
> library(shiny)
> runExample("03_reactivity")
{% endhighlight %}

前面几个例子给你了个初步印象——Shiny应用程序的代码长成什么样子。前面解释了反应式编程的一点概念，不过略过了大部分细节。 在本节，我们会更进一步讲解这些细节。如果你想更深入学习这些细节，请看《理解反应式设计》这节，从[反应式设计概述](#reactivity-overview)开始。

### 什么是反应式设计

Shiny的web框架从本质上说是使从页面中获取*输入值*并传递给R变得更容易，然后把R代码的结果以*输出值*的形式返回给页面。

    input values => R code => output values

因为Shiny程序是交互式的，输入值可以随时改变，输出值也应该立即更新，以反映输入输入值的改变。

Shiny中有**反应式编程**的库，你可以用它来定义你的应用程序逻辑。使用这个库，改变输入值会自动引发R代码中相应的部分重新执行，反过来会更新输出结果。

### 反应式编程基础

反应式编程是种编程风格，这种风格以**反应值**开始，**反应值**是随时间变化的值，或者由用户输入的值，在反应值之上绑定有**反应表达式**（reactive expression），反应表达式会接收到反应值并执行其他反应表达式。

反应表达式有趣的地方在于，当它执行的时候，会自动跟踪读取到的反应值以及调用的其他反应表达式。如果反应表达式所依赖的反应值和反应表达式发生了改变，那么该反应表达式的返回值也应该变化（原文是If those “dependencies” become out of date, then they know that their own return value has also become out of date）。因为有这种跟踪机制，所以改变一个反应值会自动引发依赖于它的反应表达式重新执行。

在shiny中使用反应值时，最常见的方式是使用`input`对象。`input`对象会被传递给`shinyServer`函数中，让你可以用类似列表的语法来访问网页上的输入值。从代码上看，你好像是从列表或者数据框里读取了值，但实际上你读取的是反应值。你不必写监测输入值变化的代码，只需要写反应表达式来读取所需的反应值，Shiny会处理好什么时候调用它们。

创建反应表达式很简单，只需要把一个正常的表达式传递给`reactive`函数就行。在本节的示例程序中，下面这个简单的反应表达式的功能是，基于用户在表单中选择的选项来返回R数据框。 

{% highlight r %}
datasetInput <- reactive({
   switch(input$dataset,
          "rock" = rock,
          "pressure" = pressure,
          "cars" = cars)
})
{% endhighlight %}


为了将反应值转化为可以在网页上呈现的输出，我们要将它们赋值给`output`对象(同样传递给`shinyServer`函数)。下面是个赋值给输出值的例子，输出值依赖于我们刚才定义的反应表达式`datasetInput`，以及`input$obs`：

{% highlight r %}
output$view <- renderTable({
   head(datasetInput(), n = input$obs)
})
{% endhighlight %}

不管是 `datasetInput` 还是`input$obs`，一旦它们的值发生改变，上面这个表达式将会重新执行（它的输出也会在浏览器里重新渲染）。

### 回到代码上

现在我们已经对一些核心概念有了更多了解，我们再来看看源代码，并尝试更深层次理解。用户接口的定义中，增加了来用定义说明文字（caption）的文本输入框，尽管它与前一个例子还是很相似。

#### ui.R

{% highlight r %}
library(shiny)

# Define UI for dataset viewer application
shinyUI(pageWithSidebar(

  # Application title
  headerPanel("Reactivity"),

  # Sidebar with controls to provide a caption, select a dataset, and 
  # specify the number of observations to view. Note that changes made
  # to the caption in the textInput control are updated in the output
  # area immediately as you type
  sidebarPanel(
    textInput("caption", "Caption:", "Data Summary"),

    selectInput("dataset", "Choose a dataset:", 
                choices = c("rock", "pressure", "cars")),

    numericInput("obs", "Number of observations to view:", 10)
  ),


  # Show the caption, a summary of the dataset and an HTML table with
  # the requested number of observations
  mainPanel(
    h3(textOutput("caption")), 

    verbatimTextOutput("summary"), 

    tableOutput("view")
  )
))
{% endhighlight %}


### 服务端脚本

服务端脚本声明了反应表达式`datasetInput`和三个反应输出值。下面有每个定义的详细注释描述了在反应式系统中是如何运作的：

#### server.R

{% highlight r %}
library(shiny)
library(datasets)

# Define server logic required to summarize and view the selected dataset
shinyServer(function(input, output) {

  # By declaring datasetInput as a reactive expression we ensure that:
  #
  #  1) It is only called when the inputs it depends on changes
  #  2) The computation and result are shared by all the callers (it 
  #     only executes a single time)
  #  3) When the inputs change and the expression is re-executed, the
  #     new result is compared to the previous result; if the two are
  #     identical, then the callers are not notified
  #
  datasetInput <- reactive({
    switch(input$dataset,
           "rock" = rock,
           "pressure" = pressure,
           "cars" = cars)
  })

  # The output$caption is computed based on a reactive expression that
  # returns input$caption. When the user changes the "caption" field:
  #
  #  1) This expression is automatically called to recompute the output 
  #  2) The new caption is pushed back to the browser for re-display
  # 
  # Note that because the data-oriented reactive expression below don't 
  # depend on input$caption, those expression are NOT called when 
  # input$caption changes.
  output$caption <- renderText({
    input$caption
  })

  # The output$summary depends on the datasetInput reactive expression, 
  # so will be re-executed whenever datasetInput is re-executed 
  # (i.e. whenever the input$dataset changes)
  output$summary <- renderPrint({
    dataset <- datasetInput()
    summary(dataset)
  })

  # The output$view depends on both the databaseInput reactive expression
  # and input$obs, so will be re-executed whenever input$dataset or 
  # input$obs is changed. 
  output$view <- renderTable({
    head(datasetInput(), n = input$obs)
  })
})
{% endhighlight %}

在前三个例子中，我们查看了一些代码，也解释了不少概念。下一节将重点讲解从头开始构建shiny程序的机制，也会讲解如何运行和调试shiny程序。