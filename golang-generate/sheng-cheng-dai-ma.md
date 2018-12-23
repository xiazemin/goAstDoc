通用计算的一个特性--图灵完备--是一个计算机程序可以编写一个计算机程序。这是一个强大的想法，尽管经常出现，但还不足够完美。例如，它是编译器定义的重要组成部分。它也是go test命令的工作原理：它扫描要测试的软件包，写出一个包含为包定制的测试工具的Go程序，然后编译并运行。现代电脑快到可以在几分之一秒完成这个看似昂贵的序列。

还有很多程序编写程序的其他例子。例如，yacc读入一个语法描述，并写出一个程序来解析该语法。Protocol buffer“编译器”读取接口描述输出结构定义，方法和其他支持代码。各种配置工具也是这样工作的，检查元数据或环境，输出自定义的本地配置。

因此，编写程序的程序是软件工程的重要组成部分，但是像yacc这些可以生成源代码的程序需要基础到构建过程中，以便可以编译它们的输出。当使用像Make这样的外部构建工具时，这通常可以很容易做到。但是在Go中，Go的工具从Go源中获取所有必要的构建信息，这有一个问题。没有机制可以单独地从go tool中运行yacc。

直到现在，就是这样。

最新的Go发布版，1.4，包含一个新命令，可以更轻松地运行这些工具。它叫做go generate,它可以通过扫描Go源码中的特殊注释来识别要运行的常规命令。了解go generate不是go build的一部分很重要。它不包含依赖关系分析，必须在运行go build之前显式运行。它旨在由Go package的作者使用，而不是其客户端

Go generate命令很容易使用。作为一个预热，下面展示如何使用它来生成yacc语法。假设你有一个名为gopher.y的YACC输入文件，它定义了一种新语言的语法。要生成实现语法的Go源码文件，通常会调用Yacc的标准Go版本：

```
go tool yacc -o gopher.go -p parser gopher.y
```

-o选项命令输出文件，-p选项指定包名。

要使go generate驱动这个过程，在同一目录中的任何一个普通（非生成）.go文件中，将该注释添加的文件中的任何位置:

```
//go:generate go tool yacc -o gopher.go -p parser gopher.y
```

这个文本就是上面的命令，前面加上一个由go generate识别的特殊注释。注释必须从行的开始处开始，并在在//和go:generate之间没有空格。在该标记之后，该行的其余部分指定go generate运行的命令。

现在运行它。切换到源目录，运行go generate，然后go build等等。

```
$ cd $GOPATH/myrepo/gopher
```

```
$ go generate
```

```
$ go build
```

```
$ go test
```

   假设没有错误，go generate命令将调用yacc来创建gopher.go文件，此时目录包含完整的go源文件，因此我们可以正常构建，测试和正常工作。每次gopher.y被修改，只需要重新运行go generate来重新生成解析器。

   有关go generate如何工作的更多详细信息，包括选项，环境变量等，可以参阅设计文档。

Go generate不会影响到make或其他一些编译机制，但它依附go tool，不需要额外安装，而且很适合Go生态系统。请记住，它是为package作者，而不是客户端，只是因为它调用的程序在目标机器上可能不可用。另外，如果包含的包是通过go get导入的，一旦文件被生成（并且被测试），他就必须被检入到源码库以供客户端使用。
