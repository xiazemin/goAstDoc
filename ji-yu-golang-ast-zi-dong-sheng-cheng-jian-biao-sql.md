生成服务器API基础代码以及Swagger文档注释 \(只支持gin框架\)

生成服务器API客户端代码

go struct 批量添加 tag

生成 gorm model struct

model struct 生成 sql

因为这些功能跟我们内部的公共库有一定耦合，因此整个工具可能无法开源出来。这里，我们以model struct 生成 sql功能为例，聊聊我们在做这个工具的思路和使用到的工具。



任务

这里以我们在项目中使用的jinzhu同学的gorm作为orm库。如果你在使用golang的其他orm lib，实现方式应该大同小异。



我们的任务是从下面的这个model struct定义：







生成 mysql 建表语句（文件）：







思路

从model struct 生成 sql是一个将语言A翻译为语言B的问题。而这个过程跟我们平时将源代码编译为二进制可执行程序从原理上说是没有区别的。因此，这个问题本质上是一个编译问题。一个完整的编译包含以下步骤：







对于本文要完成的任务来说，主要完成词法分析、语法分析、目标代码生成即可。



工具

要完成词法分析和语法分析，我们有上古神器 Lex 和 Yacc, Yet Another Compiler-Compiler. 而我们只是想完成一个建表文件的生成任务而已，使用者两个工具有时候要自定义语法，又是要自己写lex和yacc文件，累觉不爱……



Golang 有很多其他语言羡慕不来的工具，例如 go pprof, go list, go vet 等。在语言元编程方面，go 1.4实现了自举；而编译时候涉及到的词法分析和语法分析很早前就放在了标准库 go/ast 中。AST是abstract syntax tree的缩写，直译过来是抽象语法树。通过AST，我们可以编写一个go程序解析go源代码。具体到本文要完成的任务，要编写一个这样的程序解析定义数据表的model struct, 然后生成sql建表语句。



实现

具体到我们的任务实现，可以拆分为如下几个步骤：



加载源代码，生成 AST Tree

获取和解析 model struct AST

根据struct field name/tag 生成create\_definition, table\_options





完整代码实现，可以移步github gorm2sql.



实现效果：



user\_email.go:



type UserBase struct {

    UserId string \`sql:"index:idx\_ub"\`

    Ip     string \`sql:"unique\_index:uniq\_ip"\`

}



type UserEmail struct {

    Id       int64    \`gorm:"primary\_key"\`

    UserBase

    Email      string

    Sex        bool

    Age        int

    Score      float64

    UpdateTime time.Time \`sql:"default:CURRENT\_TIMESTAMP ON UPDATE CURRENT\_TIMESTAMP"\`

    CreateTime time.Time \`sql:"default:CURRENT\_TIMESTAMP"\`

}

gorm2sql sql -f user\_email.go -s UserEmail -o db.sql

Result:



CREATE TABLE \`user\_email\`

\(

  \`id\` bigint AUTO\_INCREMENT NOT NULL ,

  \`user\_id\` varchar\(128\) NOT NULL ,

  \`ip\` varchar\(128\) NOT NULL ,

  \`email\` varchar\(128\) NOT NULL ,

  \`sex\` boolean NOT NULL ,

  \`age\` int NOT NULL ,

  \`score\` double NOT NULL ,

  \`update\_time\` datetime NOT NULL  DEFAULT CURRENT\_TIMESTAMP ON UPDATE CURRENT\_TIMESTAMP,

  \`create\_time\` datetime NOT NULL  DEFAULT CURRENT\_TIMESTAMP,

  INDEX idx\_ub \(\`user\_id\`\),

  UNIQUE INDEX uniq\_ip \(\`ip\`\),

  PRIMARY KEY \(\`id\`\)

\) engine=innodb DEFAULT charset=utf8mb4;



