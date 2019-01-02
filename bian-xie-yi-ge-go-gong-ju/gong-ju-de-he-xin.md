在 `main` 函数中，我们将使用从上一步解析的节点来调用 `cfg.rewrite()` 函数：

```
func
main
()
 {
    
// ... find start and end position of the struct to be modified



    rewrittenNode, errs := cfg.rewrite(node, start, end)
    
if
 errs != 
nil
 {
        
if
 _, ok := errs.(*rewriteErrors); !ok {
            
return
 errs
        }
    \}


    
// continue outputting the rewritten node

}

```

这是该工具的核心。在 `rewrite` 函数中，我们将重写开始位置和结束位置之间的所有结构体字段。 在深入了解之前，我们可以看一下该函数的大概内容：

```
// rewrite rewrites the node for structs between the start and end
// positions and returns the rewritten node
func
(c *config)
rewrite
(node ast.Node, start, end 
int
)
(ast.Node, error)
 {
    errs := 
&
rewriteErrors{errs: 
make
([]error, 
0
)}

    rewriteFunc := 
func
(n ast.Node)
bool
 {
        
// rewrite the node ...

    }

    
if
len
(errs.errs) == 
0
 {
        
return
 node, 
nil

    }

    ast.Inspect(node, rewriteFunc)
    
return
 node, errs
}
复制代码
```

正如你所看到的，我们再次使用 `ast.Inspect()` 来逐步处理给定节点的树。我们重写 `rewriteFunc` 函数中的每个字段的标签（更多内容在后面）。

因为传递给 `ast.Inspect()` 的函数不会返回错误，因此我们将创建一个错误映射（使用 `errs` 变量定义），之后在我们逐步遍历树并处理每个单独的字段时收集错误。现在让我们来谈谈 `rewriteFunc` 的内部原理：

```
rewriteFunc := 
func
(n ast.Node)
bool
 {
    x, ok := n.(*ast.StructType)
    
if
 !ok {
        
return
true

    }

    
for
 _, f := 
range
 x.Fields.List {
        line := c.fset.Position(f.Pos()).Line

        
if
 !(start 
<
= line 
&
&
 line 
<
= end) {
            
continue

        }

        
if
 f.Tag == 
nil
 {
            f.Tag = 
&
ast.BasicLit{}
        }

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

        
// anonymous field
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

        f.Tag.Value = res
    }

    
return
true

}
复制代码
```

记住，AST 树中的**每一个节点**都会调用这个函数。因此，我们只寻找类型为 `*ast.StructType` 的节点。一旦我们拥有，就可以开始迭代结构体字段。

这里我们使用 `start` 和 `end` 变量。这定义了我们是否要修改该字段。如果字段位置位于 start-end 之间，我们将继续，否则我们将忽略：

```
if
 !(start 
<
= line 
&
&
 line 
<
= end) {
    
continue
// skip processing the field

}
复制代码
```




