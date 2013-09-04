![Downloading Data Screenshot](screenshots/downloads.png)

到目前为止的例子中，输出结果都是直接出现在页面上，比如绘图、表格、文本框。Shiny也提供了下载计算结果文件的特性，这能可以很容易地构建报告系统。

要运行下面的例子，键入：

{% highlight console %}
> library(shiny)
> runExample("10_download")
{% endhighlight %}

要定义文件下载的功能，在服务端用`downloadHandler`使用函数，在UI端使用`downloadButton`或 `downloadLink`。

#### ui.R

{% highlight r %}
shinyUI(pageWithSidebar(
  headerPanel('Download Example'),
  sidebarPanel(
    selectInput("dataset", "Choose a dataset:", 
                choices = c("rock", "pressure", "cars")),
    downloadButton('downloadData', 'Download')
  ),
  mainPanel(
    tableOutput('table')
  )
))
{% endhighlight %}

#### server.R

{% highlight r %}
shinyServer(function(input, output) {
  datasetInput <- reactive({
    switch(input$dataset,
           "rock" = rock,
           "pressure" = pressure,
           "cars" = cars)
  })
  
  output$table <- renderTable({
    datasetInput()
  })
  
  output$downloadData <- downloadHandler(
    filename = function() { paste(input$dataset, '.csv', sep='') },
    content = function(file) {
      write.csv(datasetInput(), file)
    }
  )
})
{% endhighlight %}

正如你看的那样，`downloadHandler`接收参数`filename`，这个参数告诉浏览器保存时默认的文件名，它可以是简单的字符串，也可以是返回字符串的函数（如上例）。

`content`参数必须是包含一个参数的函数，这个参数是临时文件的文件名。`content`函数的作用是往要下载的临时文件里写内容。

`filename`和 `content`参数可以使用反应值或表达式（尽管在上面例子中的`filename`，参数是实际的函数，`filename = paste(input$dataset, '.csv')`不会按你想的那样运行，因为当下载处理器定义时，它只计算一次）。

一般来说，只有两个参数你需要。有个可选参数`contentType`，如果它是`NA`或 `NULL`，shiny会基于文件名猜测一个合适的值。如果你想改变这一行为，可以提供一个类型的字符串（比如，`"text/plain"`）。