```
func (p *parser) parseFile() *ast.File {
    // 追踪模式
  	if p.trace {
  		defer un(trace(p, "File"))
  	}

    // 如果我们扫描第一个令牌时出现错误，不影响其他的解析
  	if p.errors.Len() != 0 {
  		return nil
  	}

  	// package的条款
  	doc := p.leadComment
  	pos := p.expect(token.PACKAGE)

  	ident := p.parseIdent()
  	if ident.Name == "_" && p.mode&DeclarationErrors != 0 {
  		p.error(p.pos, "invalid package name _")
  	}
  	p.expectSemi()


  	if p.errors.Len() != 0 {
  		return nil
  	}

  	p.openScope()
    // 作用域
  	p.pkgScope = p.topScope
    // 声明的node集合
  	var decls []ast.Decl
  	if p.mode&PackageClauseOnly == 0 {
  		// import decls
  		for p.tok == token.IMPORT {
  			decls = append(decls, p.parseGenDecl(token.IMPORT, p.parseImportSpec))
  		}

  		if p.mode&ImportsOnly == 0 {
  			// 其余的包装体
  			for p.tok != token.EOF {
  				decls = append(decls, p.parseDecl(syncDecl))
  			}
  		}
  	}
  	p.closeScope()
  	assert(p.topScope == nil, "unbalanced scopes")
  	assert(p.labelScope == nil, "unbalanced label scopes")

  	// 解析同一文件中的全局标识符
  	i := 0
  	for _, ident := range p.unresolved {
  		assert(ident.Obj == unresolved, "object already resolved")
  		ident.Obj = p.pkgScope.Lookup(ident.Name) // also removes unresolved sentinel
  		if ident.Obj == nil {
  			p.unresolved[i] = ident
  			i++
  		}
  	}
    // 构造AST文件
  	return &ast.File{
  		Doc:        doc,
  		Package:    pos,
  		Name:       ident,
  		Decls:      decls,
  		Scope:      p.pkgScope,
  		Imports:    p.imports,
  		Unresolved: p.unresolved[0:i],
  		Comments:   p.comments,
  	}
  }
```



