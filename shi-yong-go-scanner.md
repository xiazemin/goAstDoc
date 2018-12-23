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



