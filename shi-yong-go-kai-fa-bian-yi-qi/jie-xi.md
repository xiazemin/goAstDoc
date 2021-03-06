长征已经走了很远。我们概览了扫描和抽象语法树的基本概念。现在终于可以向着解析前进。

如果你已经开始与概念点不停的斗争，那么我需要警告你，从现在开始会变得越来越难。解析可能是你脑袋里已有的概念中最难的部分。我们将处理扫描器发现的词素，给它们提供一个含义，并且在 AST 中保存结果对象。

在继续前行前确保你已经理解了前面的资料。

## 解析对象

我们这个小语言的[解析器](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go)可以说是相当的简单。与扫描器一样，我们有一个指向正在扫描的文件相关信息的指针。我们通过某种途径跟踪错误和扫描器对象。

剩下的三个字段直接从扫描器返回的值映射过来：位置、标识符和词法串。

## 入口

[ParseFile](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L18)是魔法开始的地方。初始化解析器并且开始扫描。如果发生一个或多个错误，就停下来并且输出错误信息。否则，返回作为程序的入口的文件对象。

与扫描器类似，[init](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L56)进行启动准备。任何对解析器函数的调用都会让解析器和扫描器向前移动，并且无法后退。燃料就是那一行行代码，那么就让我们发动引擎吧！

[parseFile](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L126)是其导可出版本的伙伴。这个函数创建了 AST 的第一个对象。再次回顾[第五部分](http://mikespook.com/2014/05/%e7%bf%bb%e8%af%91%e7%bc%96%e8%af%91%e5%99%a85-%e8%af%ad%e8%a8%80%e8%a7%84%e6%a0%bc%e8%af%b4%e6%98%8e%e4%b9%a6/)（看到语法规则有多么重要了吗？）我们可以认为这个文件对象是所有东西的根。这就是我们已经完成的，File 对象就是第一个对象。

主意，我们并未在处理寻找第一个表达式之前进行任何的检查。这是因为我们的语法规则告诉我们应当如此。否则就是错的。我们预期找到某种形式的表达式，如果没有找到的话会是件另人烦躁的事情，不论如何，一无反顾的向前吧！

最后，我们需要确保在文件中的最后一个标识符是文件结束（EOF）。语法规则标识，在得到根表达式后，不应该还有其他任何内容。如果在期后还发现有其他任何东西的话，就报告一个错误。向每个人宣布，召集大家来看，并且一起嘲讽！

## 表达式

我们已经多次讨论了表达式。现在，任务是用[parseGenExpr](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L94)找到一个表达式，只是一开始我们并不知道它是什么类型。不过，第一个标识符会告诉我们所有需要的信息。如果我们找到了一个左括号，那么就是一个二值表达式。如果找到了一个整数，那么就是一个整数。否则就生成一个错误，然后继续。

整数是最容易解析的元素。它没有什么需要关注的细节。[代码](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L66)自己已经很能说明问题。

然而，二值表达式会更棘手一点。Calc 1 只有二值表达式，不过不久的将来，我们会添加更多类型的表达式。与 Expression 对象类似，我们需要一个更加通用的方式在特殊处理每一个之前，来进行通用的处理。[parseExpr](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L112)就是用于这个目的的。

首先，期望找到一个左括号。如果没有找到，就是错的。接着，需要明确这是哪种类型的表达式。我们知道，当前仅有的表达式类型就是二值表达式，因此接下来确定下一个标识符到底是什么运算符。如果得到的不是预期的内容，需要报告一个错误。

[BinaryExpr](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L70)展示了我们解析器不断循环调用的过程。我们已经找到了左括号和运算符，因此接下来需要寻找运算对象。这一过程是由递归的调用 parseGenExpr 来完成的。不断的一层一层的构建整个树，直到结束。

当我们找到了全部运算对象后，我们预期有一个右括号来结束表达式。最后返回作为结果的 BinaryExpr 对象，并插入到树中去。

## Expect、Next 和 AddError

[Expect](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L47)是一个超棒的小工具函数。我们告诉它预期得到什么标识符。如果它在那，那很好。如果不是的话，就报告一个错误。不论怎样，都会返回元素的位置。

[Next](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L62)实际上不需要怎么解释。它只是获取扫描器找到的内容，然后将其存储在解析器的对象中。

[AddError](https://github.com/rthornton128/calc/blob/calc1/parse/parse.go#L39)向 ErrorList 增加一个错误。如果发现了超过十个错误，那就没有必要继续了。它会打印错误，并强制解析器用一个错误码退出。

## 语法分析

我们的解析步骤在其工作时进行语法分析，这保证了解析的源代码符合语法规则。

缺少任何东西都是无法接受的。真的！

## 完成

这就是这个步骤的全部内容。如果这部分你领会得不错，那么最后一步应该很容易。当我们的语法设计越来越深入的时候，解析器会变得越来越复杂。对此，Calc 2 应该是个好例子。

