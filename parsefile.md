ParseFile 的原型：

```
func ParseFile(fset *token.FileSet, filename string, src interface{}, mode Mode) (f *ast.File, err error)

```

会返回一个 ast.File 的结构体：

```
type File struct {
        Doc        *CommentGroup   // associated documentation; or nil
        Package    token.Pos       // position of "package" keyword
        Name       *Ident          // package name
        Decls      []Decl          // top-level declarations; or nil
        Scope      *Scope          // package scope (this file only)
        Imports    []*ImportSpec   // imports in this file
        Unresolved []*Ident        // unresolved identifiers in this file
        Comments   []*CommentGroup // list of all comments in the source file
}

```

如果看到这里还没有弃坑的话，那你已经赢了一半了，我们来看看parsefile.go的输出结果：

```
     0  *ast.GenDecl {
     1  .  TokPos: example.go:2:1
     2  .  Tok: import
     3  .  Lparen: -
     4  .  Specs: []ast.Spec (len = 1) {
     5  .  .  0: *ast.ImportSpec {
     6  .  .  .  Name: *ast.Ident {
     7  .  .  .  .  NamePos: example.go:2:8
     8  .  .  .  .  Name: "_"
     9  .  .  .  }
    10  .  .  .  Path: *ast.BasicLit {
    11  .  .  .  .  ValuePos: example.go:2:10
    12  .  .  .  .  Kind: STRING
    13  .  .  .  .  Value: "\"log\""
    14  .  .  .  }
    15  .  .  .  EndPos: -
    16  .  .  }
    17  .  }
    18  .  Rparen: -
    19  }

     0  *ast.FuncDecl {
     1  .  Name: *ast.Ident {
     2  .  .  NamePos: example.go:4:6
     3  .  .  Name: "Add"
     4  .  .  Obj: *ast.Object {
     5  .  .  .  Kind: func
     6  .  .  .  Name: "Add"
     7  .  .  .  Decl: *(obj @ 0)
     8  .  .  }
     9  .  }
    10  .  Type: *ast.FuncType {
    11  .  .  Func: example.go:4:1
    12  .  .  Params: *ast.FieldList {
    13  .  .  .  Opening: example.go:4:9
    14  .  .  .  List: []*ast.Field (len = 1) {
    15  .  .  .  .  0: *ast.Field {
    16  .  .  .  .  .  Names: []*ast.Ident (len = 2) {
    17  .  .  .  .  .  .  0: *ast.Ident {
    18  .  .  .  .  .  .  .  NamePos: example.go:4:10
    19  .  .  .  .  .  .  .  Name: "n"
    20  .  .  .  .  .  .  .  Obj: *ast.Object {
    21  .  .  .  .  .  .  .  .  Kind: var
    22  .  .  .  .  .  .  .  .  Name: "n"
    23  .  .  .  .  .  .  .  .  Decl: *(obj @ 15)
    24  .  .  .  .  .  .  .  }
    25  .  .  .  .  .  .  }
    26  .  .  .  .  .  .  1: *ast.Ident {
    27  .  .  .  .  .  .  .  NamePos: example.go:4:13
    28  .  .  .  .  .  .  .  Name: "m"
    29  .  .  .  .  .  .  .  Obj: *ast.Object {
    30  .  .  .  .  .  .  .  .  Kind: var
    31  .  .  .  .  .  .  .  .  Name: "m"
    32  .  .  .  .  .  .  .  .  Decl: *(obj @ 15)
    33  .  .  .  .  .  .  .  }
    34  .  .  .  .  .  .  }
    35  .  .  .  .  .  }
    36  .  .  .  .  .  Type: *ast.Ident {
    37  .  .  .  .  .  .  NamePos: example.go:4:15
    38  .  .  .  .  .  .  Name: "int"
    39  .  .  .  .  .  }
    40  .  .  .  .  }
    41  .  .  .  }
    42  .  .  .  Closing: example.go:4:18
    43  .  .  }
    44  .  }
    45  .  Body: *ast.BlockStmt {
    46  .  .  Lbrace: example.go:4:20
    47  .  .  Rbrace: example.go:4:21
    48  .  }
    49  }

     0  *ast.ImportSpec {
     1  .  Name: *ast.Ident {
     2  .  .  NamePos: example.go:2:8
     3  .  .  Name: "_"
     4  .  }
     5  .  Path: *ast.BasicLit {
     6  .  .  ValuePos: example.go:2:10
     7  .  .  Kind: STRING
     8  .  .  Value: "\"log\""
     9  .  }
    10  .  EndPos: -
    11  }



```

ast.GenDecl 和 ast.FuncDecl 都实现了 ast.Decl 这个 interface，所以可以统一地存在 ast.Decl 的数组中\(golang的特性\)。看看这个 interface 长啥样：

```
type Decl interface {
        Node
        // contains filtered or unexported methods
}

type Node interface {
        Pos() token.Pos // position of first character belonging to the node
        End() token.Pos // position of first character immediately after the node
}

```

可以看到，任意类型只要实现了这里的 Pos 和 End 方法，就可以被当作一个 ast.Decl 的 node 来操作\(golang 特性\)。证据在此：  
![](http://xargin.com/content/images/2017/05/funcdecl.png)

![](http://xargin.com/content/images/2017/05/gendecl.png)

另外一点值得注意的是，我们这个例子中的函数声明的参数列表使用了：

```
func add(n, m int) {}

```

`n, m int`这种偷懒形式\(其实从规范的角度来讲，这样写不太好，变量和类型的视觉距离变远了\)。在被 go 的 parser 解析过之后被合并到了同一个 ast.Field 里，用不同的 name 来表示不同的参数，感兴趣的话你可以试试用`n int, m int`来修改这个程序，看看输出会有什么变化~

这个例子里的函数体里没有逻辑，所以 body 部分基本是空的，只有左括号和右括号。

不过到了这一步，我们已经拿到了一个程序的梗概。我们可以从 golang 的源码中提取出这个文件中的 import 信息，doc 信息，comment 信息，声明\(类型、函数\)信息，也可以通过遍历这些 AST 来获取到 struct 内的字段名、字段类型、tag 值，函数的 body，函数内的语句块。

