//go:generate  myenumstr -type Status

自动生成如下代码：

package user

import "fmt"

func \(c Status\) String\(\) string {

```
switch c {

case Offline:

    return "Offline"

case Online:

    return "Online"

case Disable:

    return "Disable"

case Deleted:

    return "Deleted"

}

return fmt.Sprintf\("Status\(%d\)", c\)
```

}业务逻辑

该如何实现呢？实际我们需要获得三项:

包名：user，该文件将存放在当前目录，需要知晓包名称

类型名：Status，参数传递

所有同类型var: Offline，Online，Disable，Deleted

再生成代码后将其保存到当前目录，同时进行gofmt。

具体实现

1.通过环境变量获取包名称

pkgName = os.Getenv\("GOPACKAGE"\)

pkgName = os.Getenv\("GOPACKAGE"\)

2.获取当前目录包信息 这里利用Go内置的库go/build解析目录，则可以获得该文件夹下包信息。

var err error

pkgInfo, err = build.ImportDir\(".", 0\)

if err != nil {

```
log.Fatal\(err\)
```

}

var err error

pkgInfo, err = build.ImportDir\(".", 0\)

if err != nil {

```
log.Fatal\(err\)
```

}

至此便能获得目录下所有 Go文件 pkgInfo.GoFiles，用于语法树解析。

3.解析 Go 文件语法树，提取Status相关信息。 需要注意的是，我们约定所定义的枚举信息实际应该全部是 Const。需从语法树中 提取出所有的 Const，并判断类型是否符合条件。

这里利用的是 Go 的语法树库 go/ast\(abstract syntax tree\) 和解析库 go/parser，语法树是按语句块 \(\) 形成树结构。从中过滤 const 语句块

