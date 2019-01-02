改变结构体标签的各个方面。

![](https://user-gold-cdn.xitu.io/2017/10/27/db44065fe774e078b06c2bb6a25b68ea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

该包名为 **structtag**，可以从 [github.com/fatih/struc…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffatih%2Fstructtag) 获取。 这个包允许我们以简洁的方式解析和修改标签。以下是一个完整的示例，您可以复制/粘贴并自行尝试：

    package
     main


    import
     (

    "fmt"
    "github.com/fatih/structtag"

    )


    func
    main
    ()
     {
        tag := 
    `json:"foo,omitempty,string" xml:"foo"`
    // parse the tag

        tags, err := structtag.Parse(
    string
    (tag))

    if
     err != 
    nil
     {

    panic
    (err)
        }


    // iterate over all tags
    for
     _, t := 
    range
     tags.Tags() {
            fmt.Printf(
    "tag: %+v\n"
    , t)
        }


    // get a single tag

        jsonTag, err := tags.Get(
    "json"
    )

    if
     err != 
    nil
     {

    panic
    (err)
        }


    // change existing tag

        jsonTag.Name = 
    "foo_bar"

        jsonTag.Options = 
    nil

        tags.Set(jsonTag)


    // add new tag

        tags.Set(
    &
    structtag.Tag{
            Key:     
    "hcl"
    ,
            Name:    
    "foo"
    ,
            Options: []
    string
    {
    "squash"
    },
        })


    // print the tags

        fmt.Println(tags) 
    // Output: json:"foo_bar" xml:"foo" hcl:"foo,squash"

    }
    复制代码

现在我们了解了如何解析、修改或创建结构体标签，是时候尝试修改一个 Go 源文件了。在上面的示例中，标签已经存在，但是如何从现有的 Go 结构体中获取标签呢？

