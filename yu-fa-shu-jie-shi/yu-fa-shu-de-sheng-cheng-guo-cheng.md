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

```
func ParseFile(fset *token.FileSet, filename string, src interface{}, mode Mode) (f *ast.File, err error) {
  	if fset == nil {
  		panic("parser.ParseFile: no token.FileSet provided (fset == nil)")
  	}

  	// 读取源文件，返回[]byte类型数据
  	text, err := readSource(filename, src)
  	if err != nil {
  		return nil, err
  	}

  	var p parser
  	defer func() {
  		if e := recover(); e != nil {
  			// resume same panic if it's not a bailout
  			if _, ok := e.(bailout); !ok {
  				panic(e)
  			}
  		}

  		// set result values
  		if f == nil {
  			// source is not a valid Go source file - satisfy
  			// ParseFile API and return a valid (but) empty
  			// *ast.File
  			f = &ast.File{
  				Name:  new(ast.Ident),
  				Scope: ast.NewScope(nil),
  			}
  		}

  		p.errors.Sort()
  		err = p.errors.Err()
  	}()

  	// 解析文件
    // 初始化解析器
  	p.init(fset, filename, text, mode)
    // 解析器解析文件
  	f = p.parseFile()

  	return
  }
```



