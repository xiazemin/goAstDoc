```
  // 解析左列表 一般是 l := r 或者 l1,l2 = r1,r2 或者 l <- r 或者 l++
        x := p.parseLhsList()
        switch p.tok {
        case
                token.DEFINE, token.ASSIGN, token.ADD_ASSIGN,
                token.SUB_ASSIGN, token.MUL_ASSIGN, token.QUO_ASSIGN,
                token.REM_ASSIGN, token.AND_ASSIGN, token.OR_ASSIGN,
                token.XOR_ASSIGN, token.SHL_ASSIGN, token.SHR_ASSIGN, token.AND_NOT_ASSIGN:
        // 如果看到range,range作为一种运算符按照range rhs来解析
        // 如果没看到就按正常赋值语句解析 lhs op rhs 来解析op可以是上面那些token中的一种.
                pos, tok := p.pos, p.tok
                p.next()
                var y []ast.Expr
                isRange := false
                if mode == rangeOk && p.tok == token.RANGE && (tok == token.DEFINE || tok == token.ASSIGN) {
                        pos := p.pos
                        p.next()
                        y = []ast.Expr{&ast.UnaryExpr{OpPos: pos, Op: token.RANGE, X: p.parseRhs()}}
                        isRange = true
                } else {
                        y = p.parseRhsList()
                }
                as := &ast.AssignStmt{Lhs: x, TokPos: pos, Tok: tok, Rhs: y}

    // 碰到":"找一个ident, 构成 goto: indent 之类的语句.
    case token.COLON:
                colon := p.pos
                p.next()
                if label, isIdent := x[0].(*ast.Ident); mode == labelOk && isIdent {
                        // Go spec: The scope of a label is the body of the function
                        // in which it is declared and excludes the body of any nested
                        // function.
                        stmt := &ast.LabeledStmt{Label: label, Colon: colon, Stmt: p.parseStmt()}
                        p.declare(stmt, nil, p.labelScope, ast.Lbl, label)
                        return stmt, false
                }
    // 碰到"<-",就构成 <- rhs 这样的语句.
        case token.ARROW:
                // send statement
                arrow := p.pos
                p.next()
                y := p.parseRhs()
                return &ast.SendStmt{Chan: x[0], Arrow: arrow, Value: y}, false

    // 碰到"++"或者"--"就构成一个单独的自增语句.
        case token.INC, token.DEC:
                // increment or decrement
                s := &ast.IncDecStmt{X: x[0], TokPos: p.pos, Tok: p.tok}
                p.next()
                return s, false
        }
```



