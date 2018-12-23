fset := token.NewFileSet\(\)

新建一个AST文件集合，

```
 // A FileSet represents a set of source files.
  // Methods of file sets are synchronized; multiple goroutines
  // may invoke them concurrently.
  //
  type FileSet struct {
      mutex sync.RWMutex // 加锁保护
      base  int          // 基准偏移
      files []*File      // 按顺序被加入的文件集合
      last  *File        // 最后一个文件缓存
  }
```

然后，解析该集合，f, err := parser.ParseFile\(fset, "", src, 0\)

ParseFile会解析单个Go源文件的源代码并返回相应的ast.File节点。源代码可以通过传入源文件的文件名，或src参数提供。如果src！= nil，则ParseFile将从src中解析源代码，文件名为仅在记录位置信息时使用。

