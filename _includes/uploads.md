![Uploading Files Screenshot](screenshots/uploads.png)

有时你希望用户能上传数据到你的应用程序里。shiny使得用户可以很容易地用浏览器上传数据，然后服务端的逻辑可以访问这些数据。

**重要注解：**

* 这一特性不支持IE9或更早版本（shiny server也同样不支持它们）
* 默认情况下，shiny上传的每个文件最大不能超过5MB。你可以通过`shiny.maxRequestSize`选项来修改这个限制。例如，在`server.R`的最前面加上 `options(shiny.maxRequestSize=30*1024^2)`，可以把文件大小限制提高到30MB。

要运行这个例子，可键入：

{% highlight console %}
> library(shiny)
> runExample("09_upload")
{% endhighlight %}

文件上传控件是`ui.R`文件中的 `fileInput`来创建的，访问上传的数据也跟访问其他类型的输入相类似：用<code>input$<i>inputId</i></code>来引用。`fileInput`函数的`multiple`参数取`TRUE`可以允许用户选择多个文件，而`accept`参数可以提示用户应用程序希望接收什么样的文件类型。

#### ui.R

{% highlight r %}
shinyUI(pageWithSidebar(
  headerPanel("CSV Viewer"),
  sidebarPanel(
    fileInput('file1', 'Choose CSV File',
              accept=c('text/csv', 'text/comma-separated-values,text/plain')),
    tags$hr(),
    checkboxInput('header', 'Header', TRUE),
    radioButtons('sep', 'Separator',
                 c(Comma=',',
                   Semicolon=';',
                   Tab='\t'),
                 'Comma'),
    radioButtons('quote', 'Quote',
                 c(None='',
                   'Double Quote'='"',
                   'Single Quote'="'"),
                 'Double Quote')
  ),
  mainPanel(
    tableOutput('contents')
  )
))
{% endhighlight %}

#### server.R

{% highlight r %}
shinyServer(function(input, output) {
  output$contents <- renderTable({
    
    # input$file1 will be NULL initially. After the user selects and uploads a 
    # file, it will be a data frame with 'name', 'size', 'type', and 'datapath' 
    # columns. The 'datapath' column will contain the local filenames where the 
    # data can be found.

    inFile <- input$file1

    if (is.null(inFile))
      return(NULL)
    
    read.csv(inFile$datapath, header=input$header, sep=input$sep, quote=input$quote)
  })
})
{% endhighlight %}

上面这个接收了一个文件，然后用`read.csv`函数读取这个csv文件，最后用表格来展示所有的数据。正如`server.R`的注释展示的那样，`inFile`要么是`NULL`，要么是包含上传文件信息的数据框，每一行对应的一个文件。在本例中，`fileInput`没有`multiple`参数，这样它就只有一行。

文件内容可以通过`datapath`提供的文件名来访问。查看`?fileInput`获取更多帮助信息。
