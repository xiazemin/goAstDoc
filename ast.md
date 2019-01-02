大多数编译型的语言都逃不开词法分析，语法分析\(语义分析\)、编译链接几个阶段。学生时代如果学习过编译原理，啃过龙书，接触过 lex 或者 yacc 的话，一定还对当初要求一周做出一个编译器这种任性大作业的噩梦记忆犹新。编译原理这个领域本身有一定的门槛，随便给你一段程序，你也不太好说就能很快地给出一个可用的词法分析器语法分析器\(里面还有相当的体力活成分\)。

幸运的是，我们抱到了亲爹 google 的大粗腿。在 golang 里官方已经提供了一套非常友好的词法分析和语法分析工具链。可以很方便地帮助你得到你想要的语法分析的结果：ast。有了 ast 我们就可以上天了\(笑。正经点说，是我们可以基于 ast 做很多静态分析、自动化和代码生成的事情。

这篇文章会简单介绍 golang 的 ast，并在此基础上教会你完成一个从 request struct 生成 api 项目中的 controller 层的主要代码所需要的手段。

从一个简单的例子开始，

```
package main

import (
    "fmt"
    "go/parser"
)

func main() {
    expr, _ := parser.ParseExpr("a * -1")
    fmt.Printf("%#v\n", expr)
}

```

这个例子会输出：

```
&
ast.BinaryExpr{X:(*ast.Ident)(0xc42000a3e0), OpPos:3, Op:14, Y:(*ast.UnaryExpr)(0xc42000a420)}

```

什么鬼啊，赶紧看看官方文档：

```
type BinaryExpr

A BinaryExpr node represents a binary expression.

type BinaryExpr struct {
        X     Expr        // left operand
        OpPos token.Pos   // position of Op
        Op    token.Token // operator
        Y     Expr        // right operand
}

```

理解了 binary expression 就豁然开朗了，实际上就是你的`a * -1`表达式，Op 代表二元表达式的操作符，而 OpPos 代表操作符在表达式中的偏移。这里的 Op 类型是token.Token，实际上是个枚举值：

```
const (
        // Special tokens
        ILLEGAL Token = iota
        EOF
        COMMENT

        // Identifiers and basic type literals
        // (these tokens stand for classes of literals)
        IDENT  // main
        .....

        // Operators and delimiters
        ADD // +
        SUB // -
        MUL // *
        QUO // /
        REM // %

        AND     // 
&

        .....

        ADD_ASSIGN // +=
        .....

        AND_ASSIGN     // 
&
=
        .....

        LAND  // 
&
&

        .....

        EQL    // ==
        .....

        NEQ      // !=
        .....

        LPAREN // (
        .....

        RPAREN    // )
        .....

        // Keywords
        BREAK
        ...略...
)


```

很好理解，就是一些特殊符号，比较符号，关键字，和理论上的 token 是一回事。X 和 Y 就更好理解了，左操作数，右操作数。

