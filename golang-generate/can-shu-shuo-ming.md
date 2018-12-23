-run 正则表达式匹配命令行，仅执行匹配的命令



-v 打印已被检索处理的文件。



-n 打印出将被执行的命令，此时将不真实执行命令



-x 打印已执行的命令



执行举例：



\# 打印当前目录下所有文件，将被执行的命令go generate -n ./... \# 对包下所有Go文件进行处理go generate github.com/ysqi/repo \# 打印包下所有文件，将被执行的命令go generate -n runtime 如何使用Generate命令



需在的代码中配置generate标记，则在执行go generate时可被检测到。go generate执行时，实际在扫描如下内容：



//go:generate command argument...



generate命令不是解析文件，而是逐行匹配以//go:generate开头的行\(前面不要有空格\)。故命令可以写在任何位置，也可存在多个命令行。



//go:generate后跟随具体的命令。命令为可执行程序，形同在Shell下执行。所以命令是在环境变量中存在，也可是完整路径。如：



packagemain import"fmt"//go:generate echo hello//go:generate go run main.go//go:generate echo file=$GOFILE pkg=$GOPACKAGEfuncmain\(\) { fmt.Println\( "main func"\)}



执行：



$ go generatehelloman func file=main.go pkg=main



在执行go generate时将会加入些信息到环境变量，可在命令程序中使用。



$GOARCH 架构 \(arm, amd64, etc.\)$GOOS OS \(linux, windows, etc.\)$GOFILE 当前处理中的文件名$GOLINE 当前命令在文件中的行号$GOPACKAGE 当前处理文件的包名$DOLLAR 固定的"$",不清楚用途



同时该命令是在所处理文件的目录下执行，故利用上述信息和当前目录，边可获得丰富的DIY功能。

