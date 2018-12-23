```
// 所有的AST树的节点都需要实现Node接口
  type Node interface {
      Pos() token.Pos // position of first character belonging to the node
      End() token.Pos // position of first character immediately after the node
  }

  // 所有的表达式都需要实现Expr接口
  type Expr interface {
      Node
      exprNode()
  }

  // 所有的语句都需要实现Stmt接口
  type Stmt interface {
      Node
      stmtNode()
  }

  // 所有的声明都需要实现Decl接口
  type Decl interface {
      Node
      declNode()
  }
```

上面就是语法的三个主体,表达式\(expression\),语句\(statement\)和声明\(declaration\),Node是基类接口,任何类型的主体都是Node，用于标记该节点位置的开始和结束.

不过三个主体的函数没有实际意义,只是用三个interface来区分不同的语法单位,如果某个语法是Stmt的话,就实现一个空的stmtNode函数即可.

这样的好处是可以对语法单元进行comma,ok来判断类型,并且保证只有这些变量可以赋值给对应的interface.但是实际上这个划分不是很严格

