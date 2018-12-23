`parser`结构体的定义,parser是以file为单位的.

```
// The parser structure holds the parser's internal state.
type parser struct {
        file    *token.File
        errors  scanner.ErrorList // 解析过程中遇到的错误列表
        scanner scanner.Scanner // 词法分析器.

        // Tracing/debugging
        mode   Mode // parsing mode // 解析模式
        trace  bool // == (mode & Trace != 0)
        indent int  // indentation used for tracing output

        // Comments 列表
        comments    []*ast.CommentGroup
        leadComment *ast.CommentGroup // last lead comment
        lineComment *ast.CommentGroup // last line comment

        // Next token
        pos token.Pos   // token position
        tok token.Token // one token look-ahead
        lit string      // token literal

        // Error recovery
        // (used to limit the number of calls to syncXXX functions
        // w/o making scanning progress - avoids potential endless
        // loops across multiple parser functions during error recovery)
        syncPos token.Pos // last synchronization position 解析错误的同步点.
        syncCnt int       // number of calls to syncXXX without progress

        // Non-syntactic parser control
        // 非语法性的控制
        // <0 在控制语句中, >= 在表达式中.
        exprLev int  // < 0: in control clause, >= 0: in expression
        // 正在解析右值表达式
        inRhs   bool // if set, the parser is parsing a rhs expression

        // Ordinary identifier scopes
        pkgScope   *ast.Scope        // pkgScope.Outer == nil
        topScope   *ast.Scope        // top-most scope; may be pkgScope
        unresolved []*ast.Ident      // unresolved identifiers
        imports    []*ast.ImportSpec // list of imports

        // Label scopes
        // (maintained by open/close LabelScope)
        labelScope  *ast.Scope     // label scope for current function
        targetStack [][]*ast.Ident // stack of unresolved labels
}
```

解析的入口是ParseFile,首先调用init,再调用parseFile进行解析.

整个解析是一个递归向下的过程也就是最low但是最实用的手写实现的方式.像yacc\[4\]生成的是我们编译里学的LALR\[5\]文法,牛逼的一逼,但是

gcc和Go都没用自动生成的解析器,也就是手写个几千行代码的事,所以为了更好的掌握编译器的细节,都选择了手写最简单的递归向下的方式.

通过init初始化scanner等.

```
func (p *parser) init(fset *token.FileSet, filename string, src []byte, mode Mode) {
        p.file = fset.AddFile(filename, -1, len(src))
        var m scanner.Mode
        if mode&ParseComments != 0 {
                m = scanner.ScanComments
        }
        // 错误处理函数是在错误列表中添加错误.
        eh := func(pos token.Position, msg string) { p.errors.Add(pos, msg) }
        p.scanner.Init(p.file, src, eh, m)

        p.mode = mode
        p.trace = mode&Trace != 0 // for convenience (p.trace is used frequently)

        p.next()
}
```



