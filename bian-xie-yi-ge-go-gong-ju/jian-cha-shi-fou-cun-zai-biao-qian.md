    接下来，我们检查是否存在标签。如果标签字段为空（也就是 nil），则**初始化**标签字段。这在有助于后面的 `cfg.process()` 函数避免 panic：


    ```
    if
     f.Tag == 
    nil
     {
        f.Tag = 
    &
    ast.BasicLit{}
    }
    复制代码
    ```

    现在让我先解释一下一个**有趣**的地方，然后再继续。**gomodifytags** 尝试获取字段的字段名称并处理它。然而，当它是一个匿名字段呢？：

    ```
    type
     Bar 
    string
    type
     Foo 
    struct
     {
        Bar 
    //this is an anonymous field

    }
    复制代码
    ```

    在这种情况下，因为没有字段名称，我们尝试从类型名称中获取**字段名称**：

    ```
    // if there is a field name use it

    fieldName := 
    ""
    if
    len
    (f.Names) != 
    0
     {
        fieldName = f.Names[
    0
    ].Name
    }


    // if there is no field name, get it from type's name
    if
     f.Names == 
    nil
     {
        ident, ok := f.Type.(*ast.Ident)

    if
     !ok {

    continue

        }

        fieldName = ident.Name
    }
    复制代码
    ```

    一旦我们获得了字段名称和标签值，就可以开始处理该字段。`cfg.process()` 函数负责处理有字段名称和标签值（如果有的话）的字段。在它返回处理结果后（在我们的例子中是 **struct tag** 格式），我们使用它来覆盖现有的标签值：

    ```
    res, err := c.process(fieldName, f.Tag.Value)

    if
     err != 
    nil
     {
        errs.Append(fmt.Errorf(
    "%s:%d:%d:%s"
    ,
            c.fset.Position(f.Pos()).Filename,
            c.fset.Position(f.Pos()).Line,
            c.fset.Position(f.Pos()).Column,
            err))

    continue

    }


    // rewrite the field with the new result,i.e: json:"foo"

    f.Tag.Value = res
    复制代码
    ```

    实际上，如果你记得 **structtag**，它返回标签实例的 **String\(\)** 表述。在我们返回标签的最终表述之前，我们根据需要使用 **structtag** 包的各种方法修改结构体。以下是一个简单的说明图示：



    ![](https://user-gold-cdn.xitu.io/2017/10/27/a3b3b6a13122bd9bf2bfcacb5dfc49a6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "用 structtag 包修改每个字段")

    用 structtag 包修改每个字段



    例如，我们要扩展 **process\(\)** 中的 `removeTags()` 函数。此功能使用以下配置来创建要删除的标签数组（键名称）：

    ```
    flagRemoveTags = flag.String(
    "remove-tags"
    , 
    ""
    , 
    "Remove tags for the comma separated list of keys"
    )


    if
     *flagRemoveTags != 
    ""
     {
        cfg.remove = strings.Split(*flagRemoveTags, 
    ","
    )
    }
    复制代码
    ```

    在 `removeTags()` 中，我们检查是否使用了 `--remove-tags`。如果有，我们将使用 structtag 的 [tags.Delete\(\)](https://link.juejin.im?target=https%3A%2F%2Fgodoc.org%2Fgithub.com%2Ffatih%2Fstructtag%23Tags.Delete) 方法来删除标签：

    ```
    func
    (c *config)
    removeTags
    (tags *structtag.Tags)
     *
    structtag
    .
    Tags
     {

    if
     c.remove == 
    nil
     || 
    len
    (c.remove) == 
    0
     {

    return
     tags
        }

        tags.Delete(c.remove...)

    return
     tags
    }
    复制代码
    ```

    此逻辑同样适用于 `cfg.Process()` 中的所有函数。

    ---

    我们已经有了一个重写的节点，让我们来讨论最后一个话题。输出和格式化结果：



    ![](https://user-gold-cdn.xitu.io/2017/10/27/595019173fc933643f3132110f86a9cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



    在 main 函数中，我们将使用上一步重写的节点来调用 `cfg.format()` 函数：

    ```
    func
    main
    ()
     {

    // ... rewrite the node


        out, err := cfg.format(rewrittenNode, errs)

    if
     err != 
    nil
     {

    return
     err
        }

        fmt.Println(out)
    }
    复制代码
    ```

    您需要注意的一件事是，我们输出到 **stdout**。这佯做有许多优点。**首先**，您只需运行工具就能查看到结果， 它不会改变任何东西，只是为了让工具用户立即看到结果。**其次**，stdout 是可组合的，可以重定向到任何地方，甚至可以用来覆盖原来的工具。

    现在我们来看看 `format()` 函数：

    ```
    func
    (c *config)
    format
    (file ast.Node, rwErrs error)
    (
    string
    , error)
     {

    switch
     c.output {

    case
    "source"
    :

    // return Go source code
    case
    "json"
    :

    // return a custom JSON output
    default
    :

    return
    ""
    , fmt.Errorf(
    "unknown output mode: %s"
    , c.output)
        }
    }
    复制代码
    ```

    我们有**两种输出模式**。

    **第一个**（**source**）以 Go 格式打印 `ast.Node`。这是默认选项，如果您在命令行使用它或只想看到文件中的更改，那么这非常适合您。

    **第二个**选项（**json**）更为先进，其专为其他环境而设计（特别是编辑器）。它根据以下结构体对输出进行编码：

        type
         output 
        struct
         {
            Start  
        int
        `json:"start"`

            End    
        int
        `json:"end"`

            Lines  []
        string
        `json:"lines"`

            Errors []
        string
        `json:"errors,omitempty"`

        }
        复制代码

    对工具进行输入和最终结果输出（没有任何错误）大概示意图如下：



    ![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)



    回到 `format()` 函数。如之前所述，有两种模式。source 模式使用 **go/format** 包将 AST 格式化为 Go 源码。该软件包也被许多其他官方工具（如 **gofmt**）使用。以下是 **source** 模式的实现方式：

    ```
    var
     buf bytes.Buffer
    err := format.Node(
    &
    buf, c.fset, file)

    if
     err != 
    nil
     {

    return
    ""
    , err
    }


    if
     c.write {
        err = ioutil.WriteFile(c.file, buf.Bytes(), 
    0
    )

    if
     err != 
    nil
     {

    return
    ""
    , err
        }
    }


    return
     buf.String(), 
    nil
    复制代码
    ```

    格式包接受 `io.Writer` 并对其进行格式化。这就是为什么我们创建一个中间缓冲区（`var buf bytes.Buffer`）的原因，当用户传入一个 `-write` 标志时，我们可以使用它来覆盖文件。格式化后，我们返回缓冲区的字符串表示形式，其中包含格式化后的 Go 源代码。

    **json** 模式更有趣。因为我们返回的是一段源代码，因此我们需要准确地呈现它原本的格式，这也意味着要把注释包含进去。问题在于，当使用 `format.Node()` 打印单个结构体时，如果它们是有损的，则无法打印出 Go 注释。

    什么是有损注释（lossy comment）？看看这个例子：

    ```
    type
     example 
    struct
     {
        foo 
    int
    // this is a lossy comment


        bar 
    int

    }
    复制代码
    ```

    每个字段都是 `*ast.Field` 类型。此结构体有一个 `*ast.Field.Comment` 字段，其包含某字段的注释。

    但是，在上面的例子中，它属于谁？属于 **foo** 还是 **bar**？

    因为**不可能**确定，这些注释被称为有损注释。如果现在使用 `format.Node()` 函数打印上面的结构体，就会出现问题。 当你打印它时，你可能会得到（[play.golang.org/p/peHsswF4J…](https://link.juejin.im?target=https%3A%2F%2Fplay.golang.org%2Fp%2FpeHsswF4JQ)）：

    ```
    type
     example 
    struct
     {
        foo 
    int


        bar 
    int

    }
    复制代码
    ```

    问题在于有损注释是 `*ast.File` 的**一部分**，**它与树分开**。只有打印整个文件时才能打印出来。 所以解决方法是打印整个文件，然后删除掉我们要在 JSON 输出中返回的指定行：

    ```
    var
     buf bytes.Buffer
    err := format.Node(
    &
    buf, c.fset, file)

    if
     err != 
    nil
     {

    return
    ""
    , err
    }


    var
     lines []
    string

    scanner := bufio.NewScanner(bytes.NewBufferString(buf.String()))

    for
     scanner.Scan() {
        lines = 
    append
    (lines, scanner.Text())
    }


    if
     c.start 
    >
    len
    (lines) {

    return
    ""
    , errors.New(
    "line selection is invalid"
    )
    }

    out := 
    &
    output{
        Start: c.start,
        End:   c.end,
        Lines: lines[c.start
    -1
     : c.end], 
    // cut out lines

    }

    o, err := json.MarshalIndent(out, 
    ""
    , 
    "  "
    )

    if
     err != 
    nil
     {

    return
    ""
    , err
    }


    return
    string
    (o), 
    nil
    复制代码
    ```

    这样做确保我们可以打印所有注释。

    ---

    这就是全部内容！

    我们成功完成了我们的工具，以下是我们在整个指南中实施的完整步骤图：



    ![](https://user-gold-cdn.xitu.io/2017/10/27/24654db094acec4dc0cbba1609ddafa4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "gomodifytags的概述")

    gomodifytags的概述



    回顾一下我们做了什么：

    * 我们通过 CLI 标志
      **检索**
      配置
    * 我们通过 
      `go/parser`
       包解析文件来获取一个 
      `ast.Node`
      。
    * 在解析文件之后，我们
      **搜索**
       获取相应的结构体来获取开始位置和结束位置，这样我们可以知道需要修改哪些字段
    * 一旦我们有了开始位置和结束位置，我们再次遍历 
      `ast.Node`
      ，重写开始位置和结束位置之间的每个字段（通过使用 
      `structtag`
       包）
    * 之后，我们将格式化重写的节点，为编辑器输出 Go 源代码或自定义的 JSON

    在创建此工具后，我收到了很多友好的评论，评论者们提到了这个工具如何简化他们的日常工作。正如您所看到，尽管看起来它很容易制作，但在整个指南中，我们已经针对许多特殊的情况做了特别处理。

    **gomodifytags** 成功应用于以下编辑器和插件已经有几个月了，使得数以千计的开发人员提升了工作效率：

    * vim-go
    * atom
    * vscode
    * acme

    如果您对原始源代码感兴趣，可以在这里找到：

    * [github.com/fatih/gomod…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffatih%2Fgomodifytags)

    我还在 **Gophercon 2017** 上发表了一个演讲，如果您感兴趣，可点击下面的 youtube 地址观看：

    [www.youtube.com/embed/T4AIQ…](https://link.juejin.im?target=https%3A%2F%2Fwww.youtube.com%2Fembed%2FT4AIQ4RHp-c%3Fversion%3D3%26rel%3D1%26fs%3D1%26autohide%3D2%26showsearch%3D0%26showinfo%3D1%26iv_load_policy%3D1%26wmode%3Dtransparent)






