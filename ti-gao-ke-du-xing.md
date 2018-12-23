```
func main() {
    fs := token.NewFileSet()
    f, err := parser.ParseFile(fs, "", "package main; var a = 3", parser.AllErrors)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(f)
}
```

运行后输出如下：

$ go run main.go

&{&lt;nil&gt; 1 main \[0xc420054100\] scope 0xc42000e210 {var a}\[\] \[\] \[\]}

将 Println 换成 fmt.Printf\("%\#v",f\) 并重试：

go run main.go

\*.File{Doc:\(\*ast.CommentGroup\)\(nil\), Package:1, Name:\(\*ast.Ident\)\(0xc42000a060\), Decls:\[\]ast.Decl{\(\*ast.GenDecl\)\(0xc420054100\)}, Scope:\(\*ast.Scope\)\(0xc42000e210\), Imports:\[\]\*ast.ImportSpec\(nil\), Unresolved:\[\]\*ast.Ident\(nil\), Comments:\[\]\*ast.CommentGroup\(nil\)}

看起来可以了但是不易读，可以使用 github.com/davecgh/go-spew/spew 来让输出更易读：https://github.com/davecgh/go-spew/

spew.Dump\(f\)

重新运行程序我们会得到更加易读的输出：

```
$ go run main.go
(*ast.File)(0xc42009c000)({
 Doc: (*ast.CommentGroup)(<nil>),
 Package: (token.Pos) 1,
 Name: (*ast.Ident)(0xc42000a120)(main),
 Decls: ([]ast.Decl) (len=1 cap=1) {
  (*ast.GenDecl)(0xc420054100)({
   Doc: (*ast.CommentGroup)(<nil>),
   TokPos: (token.Pos) 15,
   Tok: (token.Token) var,
   Lparen: (token.Pos) 0,
   Specs: ([]ast.Spec) (len=1 cap=1) {
    (*ast.ValueSpec)(0xc4200802d0)({
     Doc: (*ast.CommentGroup)(<nil>),
     Names: ([]*ast.Ident) (len=1 cap=1) {
      (*ast.Ident)(0xc42000a140)(a)
     },
     Type: (ast.Expr) <nil>,
     Values: ([]ast.Expr) (len=1 cap=1) {
      (*ast.BasicLit)(0xc42000a160)({
       ValuePos: (token.Pos) 23,
       Kind: (token.Token) INT,
       Value: (string) (len=1) "3"
      })
     },
     Comment: (*ast.CommentGroup)(<nil>)
    })
   },
   Rparen: (token.Pos) 0
  })
 },
 Scope: (*ast.Scope)(0xc42000e2b0)(scope 0xc42000e2b0 {
    var a
}
),
 Imports: ([]*ast.ImportSpec) <nil>,
 Unresolved: ([]*ast.Ident) <nil>,
 Comments: ([]*ast.CommentGroup) <nil>
})
```



