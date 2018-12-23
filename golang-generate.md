利用go generate自动生成Go代码，今日在查看Go源代码时发现有大量使用此命令已生成各类代码。

命令诉求



通用计算有一特性——图灵完备。是一个计算机程序能编写一个计算机程序。既能写程序的程序。按规则定义描述内容，则可以根据描述生成程序代码。10年时刚做项目便以增删改查为主，代码生成器生成代码那是杠杠的。



通过定义便可高效生成代码，无需手工编码。如当定义一个枚举后，为了打印友好内容，我们经常手工定义String方法。



typeStatus int



const\( Offline Status = iotaOnline Disable Deleted\)



varstatusText = \[\] string{ "Offline", "Online", "Desable", "Deleted"}



func\(s Status\) String\(\) string{ v :=int\(s\)



ifv &lt; 0\|\|v &gt; len\(statusText\) {



returnfmt.Sprintf\( "Status\(%d\)", s\) }



returnstatusText\[v\]}



当遇到枚举调整时，则必须要再同步修改statusText，而此事常容被忽视。



Generate命令说明



早在Go1.4版本实现，所以你现在可以看到Go源码中大量含有的该命令使用。



如：在unicode包中生产Unicode表，为encoding/gob创建有效的编解码方法，在time包中创建时区数据等等



go generate用于一键式批量执行任何命令，创建或更新Go文件或者输出结果。



Generate 命令和其他go build、go get、go test等没半毛钱关系。需特定执行，命令如下：



go generate \[-run regexp \]\[-n \]\[-v \]\[-x \]\[build flags \]\[file.go... \| packages \]

