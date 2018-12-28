## 词法分析

那么，从哪里开始呢？

这是最难的一部分，对我来说，扫描看起来应该挺简单的，但是很快我就迷失在细节里。有许多种实现扫描器的方法，我只会向你展示其中的一种。这里是 Rob Pike 在一次演讲中的演示文稿，是关于另外一种很酷的方法：[在 Go 中的词法扫描](https://www.youtube.com/watch?v=HxaD_trXwRE)。

扫描器的基本原理就是从顶到底、从左到右、直到源代码的结尾进行检索。每次，发现所需要的元素，就报告词法串被找到，标识符会告诉解析器它是什么，以及找到它的位置。

## 有限状态机

我现在不会真正深入到[有限状态机（](http://en.wikipedia.org/wiki/Finite-state_machine)有限自动机）或其他任何相关内容的细节中去。你应当自己去发掘相关内容。[Coursera](https://www.coursera.org/course/compilers)有一个关于编译器设计的课程可能会有所帮助，同时它也包含了相关的话题。思想非常重要，不过也不是一定有必要了解每一个细节（不过我还是鼓励你去了解一下它们）。

基本思路是我们的扫描器会返回有限个数字状态。这些状态是由标识符标识的，同时只可以返回已经定义的有限的标识符。可以认为我们的扫描器具有有限的状态。也就是有限状态机。理解自动机对于理解正则表达式以及扫描时是否应当接受或拒绝一个独立的字符会有帮助。

很快这些就会变得清晰。

## 哎呀

我想要对我第一次尝试编写扫描器的时候犯的错误做一些澄清。在没有对语言做任何可靠的定义之前编写编译器的任何一个部分都是非常糟糕的想法。如果语言的核心设计一直在变化中，那么你将会需要不断的重写编译器。就是这样，不断的。我不得不将我的第一个解释器重写了数遍，每次我修订语言的时候，都是各种蛋疼。完全是浪费时间（译注：我的 Yin 语言 Group 的申请被驳回了，多半是因为申请的时候我写了：Looking for the specification。本文作者和王大师对语言和编译器的设计和实现思路完全不同。不过我觉得，应当没有对错，只有选择）。

这个过程最终让我意识到这是一个非常糟糕的决定。那些一开始看起来是很棒的想法最终会变成一个怀主意，但是完全不做决定最终将会是灾难。许多次，我都对着语言的设计问自己，“你到底是见了什么鬼做了这些？真蠢！”我是百分之百的事后诸葛亮。

## 扫描器

[扫描器](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go)相当简单。我们从一个可以跟踪例某些事情的简单对象开始。例如跟踪当前扫描的字符、从文件开头算起的偏移量（读取偏移量）、已经扫描的代码和指向文件本身的细节的指针等。

扫描的第一步是利用[Init](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L24)初始化扫描器。这里没什么特别的，外层会继续调用下一个方法，我管这个叫做“填装弹药”。

[next](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L72)方法比是个较有趣的函数。它首先将当前的字符设为零，以便确定文件的结尾。如果读取的偏移量（roffset）比文件的长度小，那么偏移量（offset）就修改为读取偏移量（roffset）。如果发现了一个新行，就在 file 对象中记录它的位置。主意，虽然我们丢弃了新行，但是仍然记录了它的位置。最后，更新当前字符并增加读取偏移量。

## 读取偏移量和 Unicode

要如何处理增加读取偏移量呢？特别是 Unicode 来说。一个字符可能占用一个或多个字节，因此每次对偏移量加一是不行的。UTF8 包的[DecodeRune](http://golang.org/pkg/unicode/utf8/#DecodeRune)函数返回了字符的字节数。在这种情况下，读取偏移量就可以确认下一个需要读取的字符的开始位置。

当前这个扫描器并不是 Unicode 友好的，不过还是可以整合这些函数，这样可以最终在添加 Unicode 支持的时候少做点工作。还会用到 unicode 包中的 IsDigit 和 IsSpace 函数。

## 扫描

这个是扫描器的基本功能。[Scan](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L32)方法首先会跳过空白字符。[skipWhitespace](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L103)只是让扫描器一次一个字符的向前移动，一直到第一个非空白字符。我利用 unicode.IsSpace 来达到这个目的。

接下来，看一下多字符元素吧。在这个例子中，只是寻找数字。然后寻找单字符元素，最终将所有内容打包到一起，汇报错误的字符或结束文件处理。

每个部分结束，都需要通过调用 next 来增加扫描器的位置，并且返回扫描的结果。

我们还要有一个语言规格说明书在手边。它告诉我们到底需要做什么。如果你需要的话，可以跳回到[第五部分](http://mikespook.com/2014/05/%e7%bf%bb%e8%af%91%e7%bc%96%e8%af%91%e5%99%a85-%e8%af%ad%e8%a8%80%e8%a7%84%e6%a0%bc%e8%af%b4%e6%98%8e%e4%b9%a6/)找到它。

## 整数

如果我们遇到的是数字，我们就用 scanNumber 来扫描一个更长的串。

我选择使用 unicode.IsDigit 函数，不过也可以编写一个简单的自己的实现。例如：`return s.ch >= '0' && s.ch <= '9'`这样简单的代码就可以满足。scanNumber 会持续的向前移动扫描器，直到遇到第一个非数字。在循环后的 if 语句处理数字出现在文件结尾的情况。

在之后版本的 Calc 里，或者你自己的版本里，我会扩展这个函数来包含其他格式的数字，比如浮点数或十六进制。

## 更多的扫描

如果没有找到数字，那么就继续检查[单个字符](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L39)。除了分号之外的其他内容都容易理解。

文件中分号开始到行未的内容是注释。在当前的扫描器实现中，注释被直接[丢弃](https://github.com/rthornton128/calc/blob/calc1/scan/scan.go#L97)，不过要保留它们也很容易。例如，Go 的扫描器将它发现的任何注释都传递给解析器，这样解析器就可以创建超好的、人人都爱的 Go 文档了！

## 总结

这就是扫描器的全部。相当直白。
