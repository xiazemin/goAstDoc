参考：

[https://arslan.io/2017/09/14/the-ultimate-guide-to-writing-a-go-tool/](https://arslan.io/2017/09/14/the-ultimate-guide-to-writing-a-go-tool/)

https://github.com/fatih/structtag



To do that we’re going to use the go/parser package to parse the file to obtain the AST \(of the whole file\) and then use the go/ast package to walk down the tree \(we could do it manually as well, but that’s the topic of another blog post\). Below you can see a fully working example:

    package main

    import (
        "fmt"
        "go/ast"
        "go/parser"
        "go/token"
    )

    func main() {
        src := `package main
            type Example struct {
        Foo string` + " `json:\"foo\"` }"

        fset := token.NewFileSet()
        file, err := parser.ParseFile(fset, "demo", src, parser.ParseComments)
        if err != nil {
            panic(err)
        }

        ast.Inspect(file, func(x ast.Node) bool {
            s, ok := x.(*ast.StructType)
            if !ok {
                return true
            }

            for _, field := range s.Fields.List {
                fmt.Printf("Field: %s\n", field.Names[0].Name)
                fmt.Printf("Tag:   %s\n", field.Tag.Value)
            }
            return false
        })
    }



