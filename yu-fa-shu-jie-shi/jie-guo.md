```
 0  *ast.File {
     1  .  Package: 2:1                        |PACKAGE token
     2  .  Name: *ast.Ident {                    |IDENT token
     3  .  .  NamePos: 2:9                    |
     4  .  .  Name: "main"                    |
     5  .  }                            |整个构成了顶部的 package main
     6  .  Decls: []ast.Decl (len = 1) {            |最上层的申明列表
     7  .  .  0: *ast.FuncDecl {                |func main的函数申明
     8  .  .  .  Name: *ast.Ident {                |IDENT token
     9  .  .  .  .  NamePos: 3:6                |
    10  .  .  .  .  Name: "main"                |
    11  .  .  .  .  Obj: *ast.Object {                |Objec是一个用于表达语法对象的结构
    12  .  .  .  .  .  Kind: func                |表示之前存在过,Decl指向了7,也就是第7行的FuncDecl.
    13  .  .  .  .  .  Name: "main"                |
    14  .  .  .  .  .  Decl: *(obj @ 7)                |
    15  .  .  .  .  }                        |
    16  .  .  .  }                        |
    17  .  .  .  Type: *ast.FuncType {                |函数类型,也就是函数签名
    18  .  .  .  .  Func: 3:1                    |参数和返回值都是空的
    19  .  .  .  .  Params: *ast.FieldList {            |
    20  .  .  .  .  .  Opening: 3:10
    21  .  .  .  .  .  Closing: 3:11
    22  .  .  .  .  }
    23  .  .  .  }
    24  .  .  .  Body: *ast.BlockStmt {                |块语句,也就是main的body
    25  .  .  .  .  Lbrace: 3:12
    26  .  .  .  .  List: []ast.Stmt (len = 2) {        |语句列表
    27  .  .  .  .  .  0: *ast.AssignStmt {            |赋值语句
    28  .  .  .  .  .  .  Lhs: []ast.Expr (len = 1) {        |左值是x
    29  .  .  .  .  .  .  .  0: *ast.Ident {
    30  .  .  .  .  .  .  .  .  NamePos: 5:2            |
    31  .  .  .  .  .  .  .  .  Name: "x"
    32  .  .  .  .  .  .  .  .  Obj: *ast.Object {        |
    33  .  .  .  .  .  .  .  .  .  Kind: var
    34  .  .  .  .  .  .  .  .  .  Name: "x"            |
    35  .  .  .  .  .  .  .  .  .  Decl: *(obj @ 27)
    36  .  .  .  .  .  .  .  .  }
    37  .  .  .  .  .  .  .  }                    |
    38  .  .  .  .  .  .  }
    39  .  .  .  .  .  .  TokPos: 5:3                |:=和它的位置
    40  .  .  .  .  .  .  Tok: :=
    41  .  .  .  .  .  .  Rhs: []ast.Expr (len = 1) {        |右边是一个数字类型的token
    42  .  .  .  .  .  .  .  0: *ast.BasicLit {
    43  .  .  .  .  .  .  .  .  ValuePos: 5:5
    44  .  .  .  .  .  .  .  .  Kind: INT
    45  .  .  .  .  .  .  .  .  Value: "1"
    46  .  .  .  .  .  .  .  }
    47  .  .  .  .  .  .  }
    48  .  .  .  .  .  }
    49  .  .  .  .  .  1: *ast.GoStmt {                |接下来是go语句
    50  .  .  .  .  .  .  Go: 6:2
    51  .  .  .  .  .  .  Call: *ast.CallExpr {            |一个调用表达式
    52  .  .  .  .  .  .  .  Fun: *ast.Ident {            |IDENT token是println
    53  .  .  .  .  .  .  .  .  NamePos: 6:5
    54  .  .  .  .  .  .  .  .  Name: "println"
    55  .  .  .  .  .  .  .  }
    56  .  .  .  .  .  .  .  Lparen: 6:12            |左括号的位置
    57  .  .  .  .  .  .  .  Args: []ast.Expr (len = 1) {    |参数列表
    58  .  .  .  .  .  .  .  .  0: *ast.Ident {            |是一个符号INDENT,并且指向的是32行的x
    59  .  .  .  .  .  .  .  .  .  NamePos: 6:13
    60  .  .  .  .  .  .  .  .  .  Name: "x"
    61  .  .  .  .  .  .  .  .  .  Obj: *(obj @ 32)
    62  .  .  .  .  .  .  .  .  }
    63  .  .  .  .  .  .  .  }
    64  .  .  .  .  .  .  .  Ellipsis: -
    65  .  .  .  .  .  .  .  Rparen: 6:14            |右括号的位置
    66  .  .  .  .  .  .  }
    67  .  .  .  .  .  }
    68  .  .  .  .  }
    69  .  .  .  .  Rbrace: 8:1
    70  .  .  .  }
    71  .  .  }
    72  .  }
    73  .  Scope: *ast.Scope {                    |最顶级的作用域
    74  .  .  Objects: map[string]*ast.Object (len = 1) {
    75  .  .  .  "main": *(obj @ 11)
    76  .  .  }
    77  .  }
    78  .  Unresolved: []*ast.Ident (len = 1) {            |这里有个没有定义的符号println,是因为是内置符号,会另外处理
    79  .  .  0: *(obj @ 52)                    |从源文件上是表现不出来的.
    80  .  }
    81  .  Comments: []*ast.CommentGroup (len = 1) {        |评论列表,以及位置和内容.
    82  .  .  0: *ast.CommentGroup {
    83  .  .  .  List: []*ast.Comment (len = 1) {
    84  .  .  .  .  0: *ast.Comment {
    85  .  .  .  .  .  Slash: 4:2
    86  .  .  .  .  .  Text: "// comments"
    87  .  .  .  .  }
    88  .  .  .  }
    89  .  .  }
    90  .  }
    91  }
```



