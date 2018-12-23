AST 树有我们想知道的所有信息，但是如何才能找出我们想要的信息呢？这时 go/ast 包就派上了用场。

我们使用 ast.Walk。这个函数接受 2 个参数。第二个参数是一个 ast.Node，AST 中所有节点都实现了的接口。第一个参数是 ast.Visitor 接口。

这个接口有一个方法：

type Visitor interface {

```
Visit\(node Node\) \(w Visitor\)
```

}

现在我们已经有了一个节点，是 parser.ParseFile 返回的 ast.File。但是我们需要创建一个自己的 ast.Visitor。

我们实现了一个打印节点类型并返回自己的 ast.Visitor。

```
package main

import (
    "fmt"
    "go/ast"
    "go/parser"
    "go/token"
    "log"
)

func main() {
    fs := token.NewFileSet()
    f, err := parser.ParseFile(fs, "", "package main; var a = 3", parser.AllErrors)
    if err != nil {
        log.Fatal(err)
    }
    var v visitor
    ast.Walk(v, f)
}

type visitor struct{}

func (v visitor) Visit(n ast.Node) ast.Visitor {
    fmt.Printf("%T\n", n)
    return v
}
```

运行这个程序我们会得到没有树结构的节点序列。那些 nil 节点是什么？在 ast.Walk 的文档中可以了解我们返回 visitor 的时候会继续找他的下级节点，如果没有下级节点将会返回 nil。

知道这个特性后我们就可以像树那样打印这个结果。

```
type visitor int

func (v visitor) Visit(n ast.Node) ast.Visitor {
    if n == nil {
        return nil
    }
    fmt.Printf("%s%T\n", strings.Repeat("\t", int(v)), n)
    return v + 1
}
```



