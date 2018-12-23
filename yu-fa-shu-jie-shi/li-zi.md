    package main

    import (
            "go/ast"
            "go/parser"
            "go/token"
    )

    func main() {
            fset := token.NewFileSet()
            f, err := parser.ParseFile(fset, "", ` 
    package main
    func main(){
            // comments
            x:=1
            go println(x)

    }
            `, parser.ParseComments)
            if err != nil {
                    panic(err)
            }
            ast.Print(fset, f)
    }



