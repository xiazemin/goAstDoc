我们已经能够解析代码并访问 AST 节点从而导出我们想要的信息：哪个变量名是包中最常用的。



代码和以前很像都是使用 go/scanner 从命令行读取文件列表。



package main



import \(

    "fmt"

    "go/ast"

    "go/parser"

    "go/token"

    "log"

    "os"

    "strings"

\)



func main\(\) {

    if len\(os.Args\) &lt; 2 {

        fmt.Fprintf\(os.Stderr, "usage:\n\t%s \[files\]\n", os.Args\[0\]\)

        os.Exit\(1\)

    }

    fs := token.NewFileSet\(\)

    var v visitor

    for \_, arg := range os.Args\[1:\] {

        f, err := parser.ParseFile\(fs, arg, nil, parser.AllErrors\)

        if err != nil {

            log.Printf\("could not parse %s: %v", arg, err\)

            continue

        }

        ast.Walk\(v, f\)

    }

}



type visitor int



func \(v visitor\) Visit\(n ast.Node\) ast.Visitor {

    if n == nil {

        return nil

    }

    fmt.Printf\("%s%T\n", strings.Repeat\("\t", int\(v\)\), n\)

    return v + 1

}

执行这段代码我们将会得到所有来自命令行参数的文件的 AST。我们可以试试传入刚刚写的 main.go 文件。



$ go build -o parser main.go  && parser main.go

\# output removed for brevity

改变 visitor 来跟踪每种标识符都被不同的变量声明形式使用了多少次。



首先我们来跟踪短变量声明。因为我们知道它一般都是一个局部变量。



type visitor struct {

    locals map\[string\]int

}



func \(v visitor\) Visit\(n ast.Node\) ast.Visitor {

    if n == nil {

        return nil

    }

    switch d := n.\(type\) {

    case \*ast.AssignStmt:

        for \_, name := range d.Lhs {

            if ident, ok := name.\(\*ast.Ident\); ok {

                if ident.Name == "\_" {

                    continue

                }

                if ident.Obj != nil && ident.Obj.Pos\(\) == ident.Pos\(\) {

                    v.locals\[ident.Name\]++

                }

            }

        }

    }

    return v

}

检查每个赋值语句的名字是不是需要被忽略的 \_ ，这时我们需要 Obj 字段跟踪声明的上下文。



如果 Obj 的字段是 nil，说明这个变量不在本文件中定义，所以它不是一个局部变量声明我们可以忽略它。



如果我们对标准库执行这段代码将会得到：



7761 err

6310 x

5446 got

4702 i

3821 c

有趣的是为什么 v 不在，我们漏掉了什么局部变量的声明的方式么？



考虑参数和 range 中的变量

我们漏掉了一对节点类型其实它们也是一种局部变量。



函数参数，接收者，返回值名称

range 语句

因为会大部分沿用之前的代码，所以我们特意为其定义了一个方法。



func \(v visitor\) local\(n ast.Node\) {

    ident, ok := n.\(\*ast.Ident\)

    if !ok {

        return

    }

    if ident.Name == "\_" \|\| ident.Name == "" {

        return

    }

    if ident.Obj != nil && ident.Obj.Pos\(\) == ident.Pos\(\) {

        v.locals\[ident.Name\]++

    }

}

对于参数、返回值和方法接收者，我们都会获取到一个长度为一的标识符列表。再定义一个方法来处理这个标识符列表：



func \(v visitor\) localList\(fs \[\]\*ast.Field\) {

    for \_, f := range fs {

        for \_, name := range f.Names {

            v.local\(name\)

        }

    }

}

这样我们就可以处理所有声明局部变量的类型：



case \*ast.AssignStmt:

    if d.Tok != token.DEFINE {

        return v

    }

    for \_, name := range d.Lhs {

        v.local\(name\)

    }

case \*ast.RangeStmt:

    v.local\(d.Key\)

    v.local\(d.Value\)

case \*ast.FuncDecl:

    v.localList\(d.Recv.List\)

    v.localList\(d.Type.Params.List\)

    if d.Type.Results != nil {

        v.localList\(d.Type.Results.List\)

    }

现在让我们运行这段代码：



$ ./parser ~/go/src/\*\*/\*.go

most common local variable names

  12264 err

  9395 t

  9163 x

  7442 i

  6127 c

处理 var 声明

现在我们需要进一步处理 var 声明，它有可能是全局变量也有可能是局部变量，并且只有判断其是否为 ast.File 级来判断它是不是全局变量。



为了达到这个目的我们为每个新的文件创建 visitor 用来跟踪文件中的全局变量，这样我们就可以正确的计算出标识符的数量。



我们会在结构体中增加一个 pkgDecls 类型为 map\[\*ast.GenDecl\]bool。在我们的 visitor 中，我们会使用 newVisitor 函数创建一个新的 visitor 并进行初始化工作，而且还会添加 globals 字段来跟踪全局变量标识符被声明的次数。



type visitor struct {

    pkgDecls map\[\*ast.GenDecl\]bool

    globals  map\[string\]int

    locals   map\[string\]int

}



func newVisitor\(f \*ast.File\) visitor {

    decls := make\(map\[\*ast.GenDecl\]bool\)

    for \_, decl := range f.Decls {

        if d, ok := decl.\(\*ast.GenDecl\); ok {

            decls\[d\] = true

        }

    }

    return visitor{

        decls,

        make\(map\[string\]int\),

        make\(map\[string\]int\),

    }

}

我们的 main 函数将会需要为每个文件创建一个新的 visitor 去跟踪汇总结果：



locals, globals := make\(map\[string\]int\), make\(map\[string\]int\)



for \_, arg := range os.Args\[1:\] {

    f, err := parser.ParseFile\(fs, arg, nil, parser.AllErrors\)

    if err != nil {

        og.Printf\("could not parse %s: %v", arg, err\)

        continue

    }

    v := newVisitor\(f\)

    ast.Walk\(v, f\)

    for k, v := range v.locals {

        locals\[k\] += v

    }

    for k, v := range v.globals {

        globals\[k\] += v

    }

}

还有最后一个部分需要完成就是需要跟踪 \*ast.GenDecl 节点并找到在变量中的所有声明：



case \*ast.GenDecl:

    if d.Tok != token.VAR {

        return v

    }

    for \_, spec := range d.Specs {

        if value, ok := spec.\(\*ast.ValueSpec\); ok {

            for \_, name := range value.Names {

                if name.Name == "\_" {

                    continue

                }

                if v.pkgDecls\[d\] {

                    v.globals\[name.Name\]++

                } else {

                    v.locals\[name.Name\]++

                }

            }

        }

    }

在每个声明中我们都只计算以 token.VAR 开头的声明。因此常量、类型和其他形式的标识符都会被忽略。在每个声明中我们还要判断它是全局变量还是局部变量，并相应的记录出现次数并忽略 \_。



程序的完全版在 这里，



执行程序我们会得到：



$ ./parser ~/go/src/\*\*/\*.go

most common local variable names

  12565 err

  9876 x

  9464 t

  7554 i

  6226 b

most common global variable names

    29 errors

    28 signals

    23 failed

    15 tests

    12 debug

至此，我们得出结论，最常用的局部变量就是 err。最常用的包名是 errors。

