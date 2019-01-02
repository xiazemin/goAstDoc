我们有了这些，现在可以通过使用这两个知识点开始构建我们的工具（命名为 **gomodifytags**）。该工具应按顺序执行以下操作

* 获取配置，用于告诉我们要修改哪个结构体
* 根据配置查找和修改结构体
* 输出结果

由于 **gomodifytags** 将主要应用于编辑器，我们将通过 CLI 标志传入配置。第二步包含多个步骤，如解析文件，找到正确的结构体，然后修改结构体（通过修改 AST）。最后，我们将结果输出，无论结果的格式是原始的 Go 源文件还是某种自定义协议（如 JSON，稍后再说）。

以下是简化版 gomodifytags 的主要功能：



![](https://user-gold-cdn.xitu.io/2017/10/27/ebae981c6c4eb217c733f1c6943f58f6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



让我们更详细地解释每一个步骤。为了简单起见，我将尝试以概括的形式来解释重要部分。 原理都一样，一旦你读完这篇博文，你将能够在没有任何指导情况下阅整个源码（指南末尾附带了所有资源）

让我们从第一步开始，了解如何**获取配置**。以下是我们的配置，包含所有必要的信息

```
type
 config 
struct
 {
    
// first section - input 
&
 output

    file     
string

    modified io.Reader
    output   
string

    write    
bool
// second section - struct selection

    offset     
int

    structName 
string

    line       
string

    start, end 
int
// third section - struct modification

    remove    []
string

    add       []
string

    override  
bool

    transform 
string

    sort      
bool

    clear     
bool

    addOpts    []
string

    removeOpts []
string

    clearOpt   
bool

}
复制代码
```

它分为**三个**主要部分：

**第一部分**包含有关如何读取和读取哪个文件的设置。这可以是本地文件系统的文件名，也可以直接来自 stdin（主要用在编辑器中）。 它还设置如何输出结果（go 源文件或 JSON），以及是否应该覆盖文件而不是输出到 stdout。

**第二部分**定义了如何选择一个结构体及其字段。有多种方法可以做到这一点。 我们可以通过它的偏移（光标位置）、结构体名称、一行单行（仅选择字段）或一系列行来定义它。最后，我们无论如何都得到开始行/结束行。例如在下面的例子中，您可以看到，我们使用它的名字来选择结构体，然后提取开始行和结束行以选择正确的字段：



![](https://user-gold-cdn.xitu.io/2017/10/27/4fed8938ce30f090f51fbdc353ae4b50?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果是用于编辑器，则最好使用**字节偏移量**。例如下面你可以发现我们的光标刚好在 `port` 字段名称后面，从那里我们可以很容易地得到开始行/结束行：



![](https://user-gold-cdn.xitu.io/2017/10/27/6db20620ccc70ed3f6906a0efaad2716?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



配置中的**第三个部分**实际上是一个映射到 `structtag` 包的一对一映射。它基本上允许我们在读取字段后将配置传给 `structtag` 包。 如你所知，`structtag` 包允许我们解析一个结构体标签并对各个部分进行修改。但它不会覆盖或更新结构体字段。

我们如何获得配置？我们只需使用 `flag` 包，然后为配置中的每个字段创建一个标志，然后分配它们。举个例子：

```
flagFile := flag.String(
"file"
, 
""
, 
"Filename to be parsed"
)
cfg := 
&
config{
    file: *flagFile,
}
复制代码
```

我们**对配置中的每个字段**执行相同操作。有关完整内容，请查看 gomodifytag 当前 master 分支的[标志定义](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffatih%2Fgomodifytags%2Fblob%2F480c94615eadfaf99e261571ae37bd53559beabb%2Fmain.go%23L76)

一旦我们有了配置，就可以做些基本的验证：

```
func
main
()
 {
    cfg := config{ ... }

    err := cfg.validate()
    
if
 err != 
nil
 {
        log.Fatalln(err)
    }

    
// continue parsing

}


// validate validates whether the config is valid or not
func
(c *config)
validate
()
error
 {
    
if
 c.file == 
""
 {
        
return
 errors.New(
"no file is passed"
)
    }

    
if
 c.line == 
""
&
&
 c.offset == 
0
&
&
 c.structName == 
""
 {
        
return
 errors.New(
"-line, -offset or -struct is not passed"
)
    }

    
if
 c.line != 
""
&
&
 c.offset != 
0
 ||
        c.line != 
""
&
&
 c.structName != 
""
 ||
        c.offset != 
0
&
&
 c.structName != 
""
 {
        
return
 errors.New(
"-line, -offset or -struct cannot be used together. pick one"
)
    }

    
if
 (c.add == 
nil
 || 
len
(c.add) == 
0
) 
&
&

        (c.addOptions == 
nil
 || 
len
(c.addOptions) == 
0
) 
&
&

        !c.clear 
&
&

        !c.clearOption 
&
&

        (c.removeOptions == 
nil
 || 
len
(c.removeOptions) == 
0
) 
&
&

        (c.remove == 
nil
 || 
len
(c.remove) == 
0
) {
        
return
 errors.New(
"one of "
 +
            
"[-add-tags, -add-options, -remove-tags, -remove-options, -clear-tags, -clear-options]"
 +
            
" should be defined"
)
    }

    
return
nil

}
复制代码
```

将验证部分放置在一个单独的函数中，以便测试。  
现在我们了解了如何获取配置并进行验证，我们继续解析文件：



![](https://user-gold-cdn.xitu.io/2017/10/27/ebf5ba95148a8b957897ab3b6251862b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



我们已经开始讨论如何解析文件了。这里的解析是 `config` 结构体的一个方法。实际上，所有的方法都是 `config` 结构体的一部分：

```
func
main
()
 {
    cfg := config{}

    node, err := cfg.parse()
    
if
 err != 
nil
 {
        
return
 err
    }

    
// continue find struct selection ...

}


func
(c *config)
parse
()
(ast.Node, error)
 {
    c.fset = token.NewFileSet()
    
var
 contents 
interface
{}
    
if
 c.modified != 
nil
 {
        archive, err := buildutil.ParseOverlayArchive(c.modified)
        
if
 err != 
nil
 {
            
return
nil
, fmt.Errorf(
"failed to parse -modified archive: %v"
, err)
        }
        fc, ok := archive[c.file]
        
if
 !ok {
            
return
nil
, fmt.Errorf(
"couldn't find %s in archive"
, c.file)
        }
        contents = fc
    }

    
return
 parser.ParseFile(c.fset, c.file, contents, parser.ParseComments)
}
复制代码
```

**parse** 函数只做一件事：解析源代码并返回一个 `ast.Node`。如果我们传入的是文件，那就非常简单了，在这种情况下，我们使用 parser.ParseFile\(\) 函数。需要注意的是 `token.NewFileSet()`，它创建一个 `*token.FileSet` 类型。我们将它存储在 `c.fset` 中，同时也传给了 `parser.ParseFile()` 函数。为什么呢？

因为 **fileset** 用于为每个文件**独立**存储每个节点的位置信息。这在以后非常有用，可以用于获得 `ast.Node` 的确切位置（请注意，`ast.Node` 使用了一个压缩了的位置信息 `token.Pos`。要获取更多的信息，它需要通过 `token.FileSet.Position()` 函数来获取一个 `token.Position`，其包含更多的信息）

让我们继续。如果通过 stdin 传递源文件，那么这更加有趣。`config.modified` 字段是一个易于测试的 `io.Reader`，但实际上我们传递的是 stdin。我们如何检测是否需要从 stdin 读取呢？

我们询问用户是否想通过 stdin 传递内容。这种情况下，工具用户需要传递 `--modified` 标志（这是一个**布尔**标志）。如果用户了传递它，我们只需将 stdin 分配给 `c.modified`：

```
flagModified = flag.Bool(
"modified"
, 
false
,
    
"read an archive of modified files from standard input"
)


if
 *flagModified {
    cfg.modified = os.Stdin
}
复制代码
```

如果再次检查上面的 `config.parse()` 函数，您将发现我们检查是否已分配了 `.modified` 字段。因为 stdin 是一个任意的数据流，我们需要能够根据给定的协议进行解析。在这种情况下，我们假定存档包含以下内容：

* 文件名，后接一行新行
* 文件大小（十进制），后接一行新行
* 文件的内容

因为我们知道文件大小，可以无障碍地解析文件内容。任何超出给定文件大小的部分，我们仅仅停止解析。

此**方法**也被其他几个工具所使用（如 **guru**、**gogetdoc** 等），对编辑器来说非常有用。 因为这样可以让编辑器传递修改后的文件内容，而**不会保存到文件系统中**。因此命名为 **modified**。

现在我们有了自己的节点，让我们继续 “搜索结构体” 这一步：



![](https://user-gold-cdn.xitu.io/2017/10/27/13777f6297d6c072757a95e4499d8d11?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在 main 函数中，我们将使用从上一步解析得到的 `ast.Node` 调用 `findSelection()` 函数：

```
func
main
()
 {
    
// ... parse file and get ast.Node


    start, end, err := cfg.findSelection(node)
    
if
 err != 
nil
 {
        
return
 err
    }

    
// continue rewriting the node with the start
&
end position

}
复制代码
```

`cfg.findSelection()` 函数根据配置返回结构体的开始位置和结束位置以告知我们如何选择一个结构体。它迭代给定节点，然后返回开始位置/结束位置（如上配置部分中所述）：



![](https://user-gold-cdn.xitu.io/2017/10/27/207547adc73265493f627f291b1e9ebe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 "查找步骤遍历所有节点，直到找到一个 \*ast.StructType，并返回该文件的开始位置和结束位置")

查找步骤遍历所有节点，直到找到一个 \*ast.StructType，并返回该文件的开始位置和结束位置



但是怎么做呢？记住有三种模式。分别是**行**选择、**偏移量**和**结构体名称**：

```
// findSelection returns the start and end position of the fields that are
// suspect to change. It depends on the line, struct or offset selection.
func
(c *config)
findSelection
(node ast.Node)
(
int
, 
int
, error)
 {
    
if
 c.line != 
""
 {
        
return
 c.lineSelection(node)
    } 
else
if
 c.offset != 
0
 {
        
return
 c.offsetSelection(node)
    } 
else
if
 c.structName != 
""
 {
        
return
 c.structSelection(node)
    } 
else
 {
        
return
0
, 
0
, errors.New(
"-line, -offset or -struct is not passed"
)
    }
}
复制代码
```

**行**选择是最简单的部分。这里我们只返回标志值本身。因此如果用户传入 `--line 3,50` 标志，函数将返回`(3, 50, nil)`。 它所做的就是拆分标志值并将其转换为整数（同样执行验证）：

```
func
(c *config)
lineSelection
(file ast.Node)
(
int
, 
int
, error)
 {
    
var
 err error
    splitted := strings.Split(c.line, 
","
)

    start, err := strconv.Atoi(splitted[
0
])
    
if
 err != 
nil
 {
        
return
0
, 
0
, err
    }

    end := start
    
if
len
(splitted) == 
2
 {
        end, err = strconv.Atoi(splitted[
1
])
        
if
 err != 
nil
 {
            
return
0
, 
0
, err
        }
    }

    
if
 start 
>
 end {
        
return
0
, 
0
, errors.New(
"wrong range. start line cannot be larger than end line"
)
    }

    
return
 start, end, 
nil

}
复制代码
```

当您选中一组行并高亮它们时，编辑器将使用此模式。

  


作者：Oopsguy

  


链接：https://juejin.im/post/59f29894f265da43333da41e

  


来源：掘金

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

