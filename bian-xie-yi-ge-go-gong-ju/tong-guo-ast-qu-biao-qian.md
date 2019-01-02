这一点，我们将使用 [go/parser](https://link.juejin.im?target=https%3A%2F%2Fgolang.org%2Fpkg%2Fgo%2Fparser) 包来**解析**文件以获取 AST（整个文件），然后使用 [go/ast](https://link.juejin.im?target=https%3A%2F%2Fgolang.org%2Fpkg%2Fgo%2Fast) 包来处理整个树（我们可以手动做这个工作，但这是另一篇博文的主题）。 您在下面可以看到一个完整的例子：

    package
     main


    import
     (

    "fmt"
    "go/ast"
    "go/parser"
    "go/token"

    )


    func
    main
    ()
     {
        src := 
    `package main
            type Example struct {
        Foo string`
     + 
    " `json:\"foo\"` }"


        fset := token.NewFileSet()
        file, err := parser.ParseFile(fset, 
    "demo"
    , src, parser.ParseComments)

    if
     err != 
    nil
     {

    panic
    (err)
        }

        ast.Inspect(file, 
    func
    (x ast.Node)
    bool
     {
            s, ok := x.(*ast.StructType)

    if
     !ok {

    return
    true

            }


    for
     _, field := 
    range
     s.Fields.List {
                fmt.Printf(
    "Field: %s\n"
    , field.Names[
    0
    ].Name)
                fmt.Printf(
    "Tag:   %s\n"
    , field.Tag.Value)
            }

    return
    false

        })
    }
    复制代码

输出结果：

    Field: Foo
    Tag:   `json:
    "foo"
    `
    复制代码

代码执行以下操作：

* 我们使用一个单独的结构体定义了一个 Go 包示例
* 我们使用 
  **go/parser**
   包来解析这个字符串。
  `parser`
   包也可以从磁盘读取文件（或整个包）。
* 在解析后，我们处理了节点（分配给变量文件）并查找由 
  [ast.StructType](https://link.juejin.im?target=https%3A%2F%2Fgolang.org%2Fpkg%2Fgo%2Fast%2F%23StructType)
   定义的 AST 节点（参考 AST 图）。通过 
  `ast.Inspect()`
   函数完成树的处理。它会遍历所有节点，直到它收到 false 值。 这是非常方便的，因为它不需要知道每个节点。
* 我们打印了结构体的字段名称和结构体标签。

---

我们现在可以做**两件重要的事**，首先，我们知道了**如何解析一个 Go 源文件**并检索结构体标签（通过 `go/parser`）。其次，我们知道了如何**解析 Go 结构体标签**，并根据需要进行修改（通过 [github.com/fatih/struc…](https://link.juejin.im?target=github.com%2Ffatih%2Fstructtag)）。



