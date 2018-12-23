```
// parser 结构体是解析器的核心
  type parser struct {
  	file    *token.File
  	errors  scanner.ErrorList
  	scanner scanner.Scanner 

  	// 追踪和问题排查
  	mode   Mode // 解析模式
  	trace  bool // == (mode & Trace != 0)
  	indent int  // 跟踪输出的缩进

  	// Comments
  	comments    []*ast.CommentGroup
  	leadComment *ast.CommentGroup 
  	lineComment *ast.CommentGroup 

  	// 定义下一个标识
  	pos token.Pos   
  	tok token.Token 
  	lit string     

  	// 错误恢复
  	syncPos token.Pos
  	syncCnt int       

  	// 非句法解析器控制
  	exprLev int  
  	inRhs   bool 

  	// 普通标识符范围
  	pkgScope   *ast.Scope       
  	topScope   *ast.Scope       
  	unresolved []*ast.Ident     
  	imports    []*ast.ImportSpec 

  	// 标签范围
  	labelScope  *ast.Scope    
  	targetStack [][]*ast.Ident 
  }
```



