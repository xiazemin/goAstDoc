```
// 解析一个func.
        pos := p.expect(token.FUNC)
    // 开一个新的作用域,topScope作为父Scope.
        scope := ast.NewScope(p.topScope) // function scope
    // 解析一个ident作为函数名
    ident := p.parseIdent()    
    // 解析函数签名,也就是参数和返回值
    params, results := p.parseSignature(scope)
    // 再解析body
    body = p.parseBody(scope)
    // 最后返回函数申明.
        decl := &ast.FuncDecl{
                Doc:  doc,
                Recv: recv,
                Name: ident,
                Type: &ast.FuncType{
                        Func:    pos,
                        Params:  params,
                        Results: results,
                },
                Body: body,
        }
```

解析参数和返回值就是解析\(filed,filed\)这样的格式,每个filed是indent type的token,最后构造成函数签名.然后来到parseBody,这个函数其实就是解析了左右花括号,然后向下开始解析Statement列表,类似于body -&gt; { stmt\_list },然后进入stmt\_list的解析,不断地解析statement.

