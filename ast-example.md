```
// example.go
```

```
package main

var a = 1 + 2

// parse.go
package main

import (
	"go/ast"
	"go/parser"
	"go/token"
)

func main() {
	fset := token.NewFileSet()
	// if the src parameter is nil, then will auto read the second filepath file
	f, _ := parser.ParseFile(fset, "./example.go", nil, parser.Mode(0))

	for _, d := range f.Decls {
		ast.Print(fset, d)
	}
}


>
>
>
 go run parse.go

```

会有下面这样的输出：

```
     0  *ast.GenDecl {
     1  .  TokPos: ./example.go:3:1
     2  .  Tok: var
     3  .  Lparen: -
     4  .  Specs: []ast.Spec (len = 1) {
     5  .  .  0: *ast.ValueSpec {
     6  .  .  .  Names: []*ast.Ident (len = 1) {
     7  .  .  .  .  0: *ast.Ident {
     8  .  .  .  .  .  NamePos: ./example.go:3:5
     9  .  .  .  .  .  Name: "a"
    10  .  .  .  .  .  Obj: *ast.Object {
    11  .  .  .  .  .  .  Kind: var
    12  .  .  .  .  .  .  Name: "a"
    13  .  .  .  .  .  .  Decl: *(obj @ 5)
    14  .  .  .  .  .  .  Data: 0
    15  .  .  .  .  .  }
    16  .  .  .  .  }
    17  .  .  .  }
    18  .  .  .  Values: []ast.Expr (len = 1) {
    19  .  .  .  .  0: *ast.BinaryExpr {
    20  .  .  .  .  .  X: *ast.BasicLit {
    21  .  .  .  .  .  .  ValuePos: ./example.go:3:9
    22  .  .  .  .  .  .  Kind: INT
    23  .  .  .  .  .  .  Value: "1"
    24  .  .  .  .  .  }
    25  .  .  .  .  .  OpPos: ./example.go:3:11
    26  .  .  .  .  .  Op: +
    27  .  .  .  .  .  Y: *ast.BasicLit {
    28  .  .  .  .  .  .  ValuePos: ./example.go:3:13
    29  .  .  .  .  .  .  Kind: INT
    30  .  .  .  .  .  .  Value: "2"
    31  .  .  .  .  .  }
    32  .  .  .  .  }
    33  .  .  .  }
    34  .  .  }
    35  .  }
    36  .  Rparen: -
    37  }

```

在视觉上不太像一棵树啊，我们把它画出来~  
![](http://xargin.com/content/images/2017/05/ast.png)

这下就像是一棵真正的树了\(虽然实际上更严格意义上就是一个node\)。

如果表达式再复杂一些，例如：

```
var a = 1 + 2 + 3

```

ast.BinaryExpr 这一部分会产生下面这样的变化：

```
    19  .  .  .  .  0: *ast.BinaryExpr {
    20  .  .  .  .  .  X: *ast.BinaryExpr {
    21  .  .  .  .  .  .  X: *ast.BasicLit {
    22  .  .  .  .  .  .  .  ValuePos: ./example.go:3:9
    23  .  .  .  .  .  .  .  Kind: INT
    24  .  .  .  .  .  .  .  Value: "1"
    25  .  .  .  .  .  .  }
    26  .  .  .  .  .  .  OpPos: ./example.go:3:11
    27  .  .  .  .  .  .  Op: +
    28  .  .  .  .  .  .  Y: *ast.BasicLit {
    29  .  .  .  .  .  .  .  ValuePos: ./example.go:3:13
    30  .  .  .  .  .  .  .  Kind: INT
    31  .  .  .  .  .  .  .  Value: "2"
    32  .  .  .  .  .  .  }
    33  .  .  .  .  .  }
    34  .  .  .  .  .  OpPos: ./example.go:3:15
    35  .  .  .  .  .  Op: +
    36  .  .  .  .  .  Y: *ast.BasicLit {
    37  .  .  .  .  .  .  ValuePos: ./example.go:3:17
    38  .  .  .  .  .  .  Kind: INT
    39  .  .  .  .  .  .  Value: "3"
    40  .  .  .  .  .  }
    41  .  .  .  .  }

```

显然外层的 X 结点现在变成了存储 1 + 2，+3 则存储在了外层的 Y 。实际上就是在语法分析的时候，按照优先级提前帮你把 1 + 2 进行结合并存入一个 ast 的 node 了。

这里提出一个思考题，如果让你对这个带有嵌套的 ast.BinaryExpr 进行求值，你觉得应该怎么做呢？\(答案在文末\)

了解了基本的二元表达式解析之后感觉自己已经无敌了，马上来个更复杂的实际代码的例子：

    // parsefile.go
    package main

    import (
        "fmt"
        "go/ast"
        "go/parser"
        "go/token"
    )

    func main() {
        fset := token.NewFileSet()
        // if the src parameter is nil, then will auto read the second filepath file
        f, _ := parser.ParseFile(fset, "example.go", src, parser.Mode(0))

        for _, d := range f.Decls {
            ast.Print(fset, d)
            fmt.Println()
        }

        for _, d := range f.Imports {
            ast.Print(fset, d)
            fmt.Println()
        }

    }

    var src = `package pppppp
    import _ "log"
    func add(n, m int) {}
    `


  


