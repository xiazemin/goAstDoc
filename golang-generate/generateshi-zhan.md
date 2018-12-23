```
//go:generate myenumstr -type Status
```

自动生成如下代码：

```
packageuser import"fmt"func(c Status) String() string{ switchc { caseOffline: return"Offline"caseOnline: return"Online"caseDisable: return"Disable"caseDeleted: return"Deleted"} returnfmt.Sprintf( "Status(%d)", c)}
```

 业务逻辑

该如何实现呢？实际我们需要获得三项:

包名：user，该文件将存放在当前目录，需要知晓包名称

类型名：Status，参数传递

所有同类型var: Offline，Online，Disable，Deleted

再生成代码后将其保存到当前目录，同时进行gofmt。

具体实现

1.通过环境变量获取包名称

pkgName = os.Getenv\( "GOPACKAGE"\)

2.获取当前目录包信息 这里利用Go内置的库go/build解析目录，则可以获得该文件夹下包信息。

varerr errorpkgInfo, err = build.ImportDir\( ".", 0\) iferr !=nil{ log.Fatal\(err\)}

至此便能获得目录下所有Go文件pkgInfo.GoFiles，用于语法树解析。

3.解析Go文件语法树，提取Status相关信息。 需要注意的是，我们约定所定义的枚举信息实际应该全部是Const。需从语法树中 提取出所有的Const，并判断类型是否符合条件。

这里利用的是Go的语法树库go/ast\(abstract syntax tree\)和解析库go/parser，语法树是按语句块\(\)形成树结构。从中过滤const语句块

fset :=token.NewFileSet\(\) //解析go文件f, err :=parser.ParseFile\(fset, gofile, nil, 0\) iferr !=nil{ log.Fatal\(err\)} //遍历每个树节点ast.Inspect\(f, func\(n ast.Node\) bool{ decl, ok :=n.\( \*ast.GenDecl\) if!ok \|\|decl.Tok !=token.CONST { returntrue} //...}

再循环const语句块中最小部分：

for\_, spec :=rangedecl.Specs {}

对每小部分判断，并获得对应的Type，如果Type为空则同上一行的一致。

// Const 代码块const\( Offline Status = iotaOnline Disable Deleted\)

行内容

vspec :=spec.\( \*ast.ValueSpec\) ifvspec.Type ==nil&&len\(vspec.Values\) &gt; 0{ // 排除 v = 1 这种结构typ = ""continue} //如果Type不为空，则确认typifvspec.Type !=nil{ ident, ok :=vspec.Type.\( \*ast.Ident\) if!ok { continue} typ = ident.Name}

但我们只需要获得符合要求的Type，获得符合要求的Const信息：

consts, ok :=typesMap\[typ\] if!ok { continue} for\_, n :=rangevspec.Names { consts = append\(consts, n.Name\)}typesMap\[typ\] = consts

这里的typesMap是根据程序入参建立：

var\( typeNames = flag.String\( "type", "", ""\)\)types :=strings.Split\( \*typeNames, ","\)typesMap :=make\( map\[ string\]\[\] string, len\(types\)\) for\_, v :=rangetypes { typesMap\[strings.TrimSpace\(v\)\] = \[\] string{}}

4.根据收集的Const，生成String函数

使用模板生成内容，同时进行gofmt

5.保存代码到文件 将文件直接保存到当前目录下，文件名已”\_string”结尾

src :=genString\(consts\) outputName :=""ifoutputName ==""{ types :=strings.Split\( \*typeNames, ","\) baseName :=fmt.Sprintf\( "%s\_string.go", types\[ 0\]\) outputName = filepath.Join\( ".", strings.ToLower\(baseName\)\)}err :=ioutil.WriteFile\(outputName, src, 0644\) iferr !=nil{ log.Fatalf\(err\)}

6.在status.go源文件配置标记

//go:generate myenumstr -type Status

运行go generate，出现错误：

user/status.go:5: running "myenumstr": exec: "myenumstr": executable file not found in $PATH

此时需要将myenumstr.go install

go install github.com/ysqi/string/myenumstr.go

安装后，我们可以在status.go使用两种方式调用：

//go:generate myenumstr -type Status//go:generate go run github.com/ysqi/string/myenumstr.go -type Status

