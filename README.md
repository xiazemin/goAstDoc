# Introduction

词法分析，然后才有语法分析。Go的parser接受的输入是源文件,内嵌了一个scanner,最后把scanner生成的token变成一颗抽象语法树\(AST\)。

    package main

    import (
    	"go/ast"
    	"go/parser"
    	"go/token"
    )

    func main() {
    	// src is the input for which we want to print the AST.
    	src := `
    package main
    func main() {
    	println("Hello, World!")
    }
    `

    	// Create the AST by parsing src.
    	fset := token.NewFileSet() // positions are relative to fset
    	f, err := parser.ParseFile(fset, "", src, 0)
    	if err != nil {
    		panic(err)
    	}

    	// Print the AST.
    	ast.Print(fset, f)

    }



