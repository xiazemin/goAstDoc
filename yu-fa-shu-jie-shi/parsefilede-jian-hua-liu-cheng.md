```
 // package clause
        // 获取源文件开头的doc注释,从这里递归向下的解析开始了
        doc := p.leadComment
        // expect 从scanner获取一个token,并且返回位置pos.
        pos := p.expect(token.PACKAGE)
        // parseIdent 获取一个token并且转化为indent,如果不是报错.
        ident := p.parseIdent()
        if ident.Name == "_" && p.mode&DeclarationErrors != 0 {
                p.error(p.pos, "invalid package name _")
        }
        // 作用域开始,标记解释器当前开始一个新的作用域
        p.openScope()
        // pkgScope 就是现在进入的作用域
        p.pkgScope = p.topScope 
        // 解析 import 申明
        for p.tok == token.IMPORT {
        // parseGenDecl解析的是 
        // import (
        // )
        // 这样的结构,如果有括号就用parseImportSpec解析列表
        // 没有就单独解析.
        // 而parseImportSpec解析的是 一个可选的indent token和一个字符串token.
        // 并且加入到imports列表中.
                decls = append(decls, p.parseGenDecl(token.IMPORT, p.parseImportSpec))
        }
    // 解析全局的申明,包括函数申明
        if p.mode&ImportsOnly == 0 {
                // rest of package body
                for p.tok != token.EOF {
                        decls = append(decls, p.parseDecl(syncDecl))
                }
        }
    // 标记从当前作用域离开.
    p.closeScope()
    // 最后返回ast.File文件对象.
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
```



