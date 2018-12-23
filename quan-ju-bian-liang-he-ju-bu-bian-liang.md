如果我们想要知道最常用的局部变量名应该怎么做？如果想知道最常用的类型或函数呢？针对这些问题 go/scanner 并不能满足我们的需求，因为它缺少对上下文的支持。按前文的方法我们可以找到需要的 token（例：var a = 3），为了获取 token 所在的作用域（包级，函数级，代码块级）我们需要上下文的支持。

在一个包中可以有很多的声明，其中的一些可能是函数声明，而在函数声明中，又可能有局部变量、常量或函数声明。

但是我们如何在 token 序列中找到这种结构呢？每种编程语言都有从 token 序列到语法树结构的转换规则。就像下面这样：

VarDecl = "var" \( VarSpec \| "\(" { VarSpec ";" } "\)" \) .

VarSpec = IdentifierList \( Type \[ "=" ExpressionList \] \| "="

```
                    ExpressionList \) .
```

这个转换规则告诉我们一个 VarDecl（变量声明） 以一个 var token 开始，紧接着是一个 VarSpec（变量说明）或是一个被括号包围的以分号分隔的标识符列表。

注意：分号其实是 Go scanner 自动添加的，所以你不会在语法分析的时候看到他们。

以 var a = 3 为例，使用 go/scanner 我们会得到这样的 token：

\[VAR\],\[IDENT "a"\],\[ASSIGN\],\[INT "3"\],\[SEMICOLON\]

依据前文描述的规则，这是一个只有 VarSpec 的 VarDecl。紧接着我们分析出标识符列表（IdentifierList）里有一个标识符（Identifier） a，没有类型（Type），表达式列表（ExpressionList）有一个整数 3 的表达式（Expression）。

这个能使我们能从 token 序列解析树结构的规则叫做语法或句法，而解析出的树结构叫做抽象语法树，简称 AST。





