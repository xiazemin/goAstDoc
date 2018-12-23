AST 树有我们想知道的所有信息，但是如何才能找出我们想要的信息呢？这时 go/ast 包就派上了用场。



我们使用 ast.Walk。这个函数接受 2 个参数。第二个参数是一个 ast.Node，AST 中所有节点都实现了的接口。第一个参数是 ast.Visitor 接口。



这个接口有一个方法：



type Visitor interface {

    Visit\(node Node\) \(w Visitor\)

}

现在我们已经有了一个节点，是 parser.ParseFile 返回的 ast.File。但是我们需要创建一个自己的 ast.Visitor。



我们实现了一个打印节点类型并返回自己的 ast.Visitor。

