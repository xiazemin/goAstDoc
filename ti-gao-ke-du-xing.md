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

&{&lt;nil&gt; 1 main \[0xc420054100\] scope 0xc42000e210 {

        var a

}

 \[\] \[\] \[\]}

将 Println 换成 fmt.Printf\("%\#v",f\) 并重试：



go run main.go

&ast.File{Doc:\(\*ast.CommentGroup\)\(nil\), Package:1, Name:\(\*ast.Ident\)\(0xc42000a060\), Decls:\[\]ast.Decl{\(\*ast.GenDecl\)\(0xc420054100\)}, Scope:\(\*ast.Scope\)\(0xc42000e210\), Imports:\[\]\*ast.ImportSpec\(nil\), Unresolved:\[\]\*ast.Ident\(nil\), Comments:\[\]\*ast.CommentGroup\(nil\)}

