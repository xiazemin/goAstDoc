parseStmt最后会进入到语句的解析,然后根据不同的token选择进入不同的解析流程,比如看到var,type,const就是申明,碰到标识符和数字等等可能就是单独的表达式,

如果碰到go,就知道是一个go语句,如果看到defer和return都能判断出相应的语句并按规则解析,看到break等条件关键字就解析条件语句,看到{就解析块语句.都是可以递归去解析的.

```
func (p *parser) parseStmt() (s ast.Stmt) {
        if p.trace {
                defer un(trace(p, "Statement"))
        }

        switch p.tok {
        case token.CONST, token.TYPE, token.VAR:
                s = &ast.DeclStmt{Decl: p.parseDecl(syncStmt)}
        case
                // tokens that may start an expression
                token.IDENT, token.INT, token.FLOAT, token.IMAG, token.CHAR, token.STRING, token.FUNC, token.LPAREN, // operands
                token.LBRACK, token.STRUCT, token.MAP, token.CHAN, token.INTERFACE, // composite types
                token.ADD, token.SUB, token.MUL, token.AND, token.XOR, token.ARROW, token.NOT: // unary operators
                s, _ = p.parseSimpleStmt(labelOk)
                // because of the required look-ahead, labeled statements are
                // parsed by parseSimpleStmt - don't expect a semicolon after
                // them
                if _, isLabeledStmt := s.(*ast.LabeledStmt); !isLabeledStmt {
                        p.expectSemi()
                }
        case token.GO:
                s = p.parseGoStmt()
        case token.DEFER:
                s = p.parseDeferStmt()
        case token.RETURN:
                s = p.parseReturnStmt()
        case token.BREAK, token.CONTINUE, token.GOTO, token.FALLTHROUGH:
                s = p.parseBranchStmt(p.tok)
        case token.LBRACE:
                s = p.parseBlockStmt()
    ...省略
```



