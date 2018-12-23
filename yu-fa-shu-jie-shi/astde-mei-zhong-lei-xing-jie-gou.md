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



