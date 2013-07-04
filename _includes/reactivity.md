

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

The most common way you'll encounter reactive values in Shiny is using the `input` object. The `input` object, which is passed to your `shinyServer` function, lets you access the web page's user input fields using a list-like syntax. Code-wise, it looks like you're grabbing a value from a list or data frame, but you're actually reading a reactive value. No need to write code to monitor when inputs change--just write reactive expression that read the inputs they need, and let Shiny take care of knowing when to call them.

It's simple to create reactive expression: just pass a normal expression into `reactive`. In this application, an example of that is the expression that returns an R data frame based on the selection the user made in the input form:

{% highlight r %}
datasetInput <- reactive({
   switch(input$dataset,
          "rock" = rock,
          "pressure" = pressure,
          "cars" = cars)
})
{% endhighlight %}


To turn reactive values into outputs that can viewed on the web page, we assigned them to the `output` object (also passed to the `shinyServer` function). Here is an example of an assignment to an output that depends on both the `datasetInput` reactive expression we just defined, as well as `input$obs`:

{% highlight r %}
output$view <- renderTable({
   head(datasetInput(), n = input$obs)
})
{% endhighlight %}

This expression will be re-executed (and its output re-rendered in the browser) whenever either the `datasetInput` or `input$obs` value changes.

### 回到代码上

Now that we've taken a deeper loop at some of the core concepts, let's revisit the source code and try to understand what's going on in more depth. The user interface definition has been updated to include a text-input field that defines a caption. Other than that it's very similar to the previous example:

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

The server script declares the `datasetInput` reactive expression as well as three reactive output values. There are detailed comments for each definition that describe how it works within the reactive system:

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

We've reviewed a lot code and covered a lot of conceptual ground in the first three examples. The next section focuses on the mechanics of building a Shiny application from the ground up and also covers tips on how to run and debug Shiny applications.
