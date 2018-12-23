来看看我们如何解析表达式 var a = 3 并且获得他的 AST。

```
package main

import (
    "fmt"
    "go/parser"
    "go/token"
    "log"
)

func main() {
    fs := token.NewFileSet()
    f, err := parser.ParseFile(fs, "", "var a = 3", parser.AllErrors)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(f)
}
```

这段代码可以编译通过但是在运行时会报错：



1:1: expected 'package', found 'var' \(and 1 more errors\)

为了解析这个我们叫做 ParseFile 的声明，我们需要给出一个完整的 go 源文件格式（以 package 作为源文件开头）。





