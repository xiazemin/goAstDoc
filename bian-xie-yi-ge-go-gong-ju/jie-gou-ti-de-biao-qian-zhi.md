结构体的标签值（内容，如 `json: "foo"`）**不是官方规范的一部分**，但是 `reflect` 包定义了一个非官方规范的格式标准，这个格式同样被 `stdlib` 包（如 `encoding/json`）所使用。它通过 [reflect.StructTag](https://link.juejin.im?target=https%3A%2F%2Fgolang.org%2Fpkg%2Freflect%2F%23StructTag) 类型定义：



![](https://user-gold-cdn.xitu.io/2017/10/27/1e4ae2ae020e5ccf540168ac3a00f32f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这个定义有点长，不是很容易让人理解。我们尝试分解一下它：

* 一个结构体标签是一个字符串文字（因为它有字符串类型）
* 键（key）部分是一个
  **无引号**
  的字符串文字
* 值（value）部分是
  **带引号**
  的字符串文字
* 键和值由冒号（:\)分隔。键与值且由冒号分隔组成的值称为
  **键值对**
* 结构体标签可以
  **包含多个键值对**
  （可选）。键值对
  **由空格分隔**
  。
* 不是定义的部分是选项设置。像 
  `encoding/json`
   这样的包在读取值时当作一个由逗号分隔列表。 第一个逗号后的内容都是选项部分，比如 
  `foo,omitempty,string`
  。其有一个名为 
  `foo`
   的值和 \[
  `omitempty`
  , 
  `string`
  \] 选项
* 因为结构体标签是字符串文字，所以需要使用双引号或反引号包围。因为值必须使用引号，因此我们总是使用反引号对整个标签做处理。

总的来说：



![](https://user-gold-cdn.xitu.io/2017/10/27/b3f44b23f5c05d3b361351e45ceb6e19?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "结构体标签定义有许多隐藏的细节")

结构体标签定义有许多隐藏的细节



我们已经了解了什么是结构体标签，我们可以根据需要轻松地修改它。 现在的问题是，我们如何解析它才能使我们能够轻松进行修改？幸运的是，`reflect.StructTag` 包含一个方法，它允许我们进行解析并返回指定键的值。以下是一个示例：

    package
     main


    import
     (

    "fmt"
    "reflect"

    )


    func
    main
    ()
     {
        tag := reflect.StructTag(
    `species:"gopher" color:"blue"`
    )
        fmt.Println(tag.Get(
    "color"
    ), tag.Get(
    "species"
    ))
    }
    复制代码

结果：

```
blue gopher
复制代码
```

如果键不存在，则返回一个空字符串。

这是非常有用，**但**也有一些不足使得它并不适合我们，因为我们需要更多的灵活性：

* 它无法检测到标签是否
  **格式错误**
  （如：键部分用引号包裹，值部分没有使用引号等）。
* 它无法得知选项的
  **语义**
  。
* 它没有
  **办法迭代现有的标签**
  或返回它们。我们必须要知道要修改哪些标签。如果不知道名字怎么办？
* 修改现有标签是不可能的。
* 我们
  **不能**
  从头开始
  **构建新的结构体标签**
  。

  


作者：Oopsguy

  


链接：https://juejin.im/post/59f29894f265da43333da41e

  


来源：掘金

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

