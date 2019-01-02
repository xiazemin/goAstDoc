**偏移量**和**结构体名称**选择需要做更多的工作。 对于这些，我们首先需要收集所有给定的结构体，以便可以计算偏移位置或搜索结构体名称。为此，我们首先要有一个收集所有结构体的函数：

```
// collectStructs collects and maps structType nodes to their positions
func
collectStructs
(node ast.Node)
map
[
token
.
Pos
]*
structType
 {
    structs := 
make
(
map
[token.Pos]*structType, 
0
)
    collectStructs := 
func
(n ast.Node)
bool
 {
        t, ok := n.(*ast.TypeSpec)
        
if
 !ok {
            
return
true

        }

        
if
 t.Type == 
nil
 {
            
return
true

        }

        structName := t.Name.Name

        x, ok := t.Type.(*ast.StructType)
        
if
 !ok {
            
return
true

        }

        structs[x.Pos()] = 
&
structType{
            name: structName,
            node: x,
        }
        
return
true

    }
    ast.Inspect(node, collectStructs)
    
return
 structs
}
复制代码
```

我们使用 `ast.Inspect()` 函数逐步遍历 AST 并搜索结构体。  
我们首先搜索 `*ast.TypeSpec`，以便我们可以获得结构体名称。搜索 `*ast.StructType` 时给定的是结构体本身，而不是它的名字。 这就是为什么我们有一个自定义的 `structType` 类型，它保存了名称和结构体节点本身。这样在各个地方都很方便。 因为每个结构体的位置都是唯一的，并且在同一位置上不可能存在两个不同的结构体，因此我们使用位置作为 map 的键。

现在我们拥有了所有结构体，在最后可以返回一个结构体的起始位置和结束位置的偏移量和结构体名称模式。 对于偏移位置，我们检查偏移是否在给定的结构体之间：

```
func
(c *config)
offsetSelection
(file ast.Node)
(
int
, 
int
, error)
 {
    structs := collectStructs(file)

    
var
 encStruct *ast.StructType
    
for
 _, st := 
range
 structs {
        structBegin := c.fset.Position(st.node.Pos()).Offset
        structEnd := c.fset.Position(st.node.End()).Offset

        
if
 structBegin 
<
= c.offset 
&
&
 c.offset 
<
= structEnd {
            encStruct = st.node
            
break

        }
    }

    
if
 encStruct == 
nil
 {
        
return
0
, 
0
, errors.New(
"offset is not inside a struct"
)
    }

    
// offset mode selects all fields

    start := c.fset.Position(encStruct.Pos()).Line
    end := c.fset.Position(encStruct.End()).Line

    
return
 start, end, 
nil

}
复制代码
```

我们使用 `collectStructs()` 来收集所有结构体，之后在这里迭代。还得记得我们存储了用于解析文件的初始 `token.FileSet` 么？

现在可以用它来获取每个结构体节点的偏移信息（我们将其解码为一个 `token.Position`，它为我们提供了 `.Offset` 字段）。 我们所做的只是一个简单的检查和迭代，直到我们找到结构体（这里命名为 `encStruct`）：

```
for
 _, st := 
range
 structs {
    structBegin := c.fset.Position(st.node.Pos()).Offset
    structEnd := c.fset.Position(st.node.End()).Offset

    
if
 structBegin 
<
= c.offset 
&
&
 c.offset 
<
= structEnd {
        encStruct = st.node
        
break

    }
}
复制代码
```

有了这些信息，我们可以提取找到的结构体的开始位置和结束位置：

```
start := c.fset.Position(encStruct.Pos()).Line
end := c.fset.Position(encStruct.End()).Line
复制代码
```

该逻辑同样适用于结构体名称选择。 我们所做的只是尝试**检查结构体名称**，直到找到与给定名称一致的结构体，而不是检查偏移量是否在给定的结构体范围内：

```
func
(c *config)
structSelection
(file ast.Node)
(
int
, 
int
, error)
 {
    
// ...
for
 _, st := 
range
 structs {
        
if
 st.name == c.structName {
            encStruct = st.node
        }
    }

    
// ...

}
复制代码
```

现在我们有了开始位置和结束位置，我们终于可以进行第三步了：修改结构体字段。



![](https://user-gold-cdn.xitu.io/2017/10/27/90d0642cb03bd1e922edfabd0371f957?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





  


作者：Oopsguy

  


链接：https://juejin.im/post/59f29894f265da43333da41e

  


来源：掘金

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

