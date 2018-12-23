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



