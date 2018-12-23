Package: 2:1代表Go解析出package这个词在第二行的第一个

main是一个ast.Ident标识符，它的位置在第二行的第9个

此处func main被解析成ast.FuncDecl（function declaration）,而函数的参数（Params）和函数体（Body）自然也在这个FuncDecl中。Params对应的是\*ast.FieldList，顾名思义就是项列表；而由大括号“｛｝”组成的函数体对应的是ast.BlockStmt（block statement）

而对于main函数的函数体中，我们可以看到调用了println函数，在ast中对应的是ExprStmt（Express Statement），调用函数的表达式对应的是CallExpr\(Call Expression\)，调用的参数自然不能错过，因为参数只有字符串，所以go把它归为ast.BasicLis \(a literal of basic type\)。

最后，我们可以看出ast还解析出了函数的作用域，以及作用域对应的对象。

Go将所有可以识别的token抽象成Node，通过interface方式组织在一起。

在这里说到token我们需要说一下词法分析，token是词法分析的结果，即将字符序列转换为标记\(token\)的过程，这个操作由词法分析器完成。这里的标记是一个字符串，是构成源代码的最小单位。

