要实现一门语言，第一要务就是要能够处理文本文件，搞明白其中究竟写了些什么。传统上，我们会先利用“[词法分析器](#)”（也称为“扫描器”）将输入切成“语元（token）”，然后再做处理。词法分析器返回的每个语元都带有一个语元编号，此外可能还会附带一些元数据（比如某个数值）。首先，我们把所有可能出现的语元都定义出来：

```
//===----------------------------------------------------------------------===//
// Lexer
//===----------------------------------------------------------------------===//
// The lexer returns tokens [0-255] if it is an unknown character, otherwise one
// of these for known things.
enum
Token
{
tok_eof
=
-
1
,
// commands
tok_def
=
-
2
,
tok_extern
=
-
3
,
// primary
tok_identifier
=
-
4
,
tok_number
=
-
5
};
static
std
::
string
IdentifierStr
;
// Filled in if tok_identifier
static
double
NumVal
;
// Filled in if tok_number
```

我们的词法分析器返回的语元，要么是上述若干个语元枚举值之一，要么是诸如“`+`”这样的“未知”字符。对于后一种情况，词法分析器返回的是这些字符的ASCII值。如果当前语元是标识符，其名称将被存入全局变量`IdentifierStr`。如果当前语元是数值常量（比如`1.0`），其值将被存入`NumVal`。注意，简单起见，我们动用了全局变量，在真正的语言实现中这可不是最佳选择 :-\) 。

Kaleidoscope的词法分析器由一个名为`gettok`的函数实现。调用该函数，就可以得到标准输入中的下一个语元。它的开头是这样的：

```
/// gettok - Return the next token from standard input.
static
int
gettok
()
{
static
int
LastChar
=
' '
;
// Skip any whitespace.
while
(
isspace
(
LastChar
))
LastChar
=
getchar
();
```

`gettok`通过C标准库的`getchar()`函数从标准输入中逐个读入字符。它一边识别读取的字符，一边将最后读入的字符存入`LastChar`，留待后续处理。这个函数干的第一件事就是利用上面的循环剔除语元之间的空白符。

接下来，`gettok`开始识别标识符和“`def`”、“`extern`”等关键字。这个任务由下面的循环负责，很简单：

```
if
(
isalpha
(
LastChar
))
{
// identifier: [a-zA-Z][a-zA-Z0-9]*
IdentifierStr
=
LastChar
;
while
(
isalnum
((
LastChar
=
getchar
())))
IdentifierStr
+=
LastChar
;
if
(
IdentifierStr
==
"def"
)
return
tok_def
;
if
(
IdentifierStr
==
"extern"
)
return
tok_extern
;
return
tok_identifier
;
}
```

注意，标识符一被识别出来就被存入全局变量`IdentifierStr`。此外，语言中的关键字也由这个循环负责识别，在此处一并处理。数值的识别过程与此类似：

```
if
(
isdigit
(
LastChar
)
||
LastChar
==
'.'
)
{
// Number: [0-9.]+
std
::
string
NumStr
;
do
{
NumStr
+=
LastChar
;
LastChar
=
getchar
();
}
while
(
isdigit
(
LastChar
)
||
LastChar
==
'.'
);
NumVal
=
strtod
(
NumStr
.
c_str
(),
0
);
return
tok_number
;
}
```

处理输入字符的代码简单明了。只要碰到代表数值的字符串，就用C标准库中的`strtod`函数将之转换为数值并存入`NumVal`。注意，这里的错误检测并不完备：这段代码会将“1.23.45.67”错误地识别成“1.23”。改还是不改，那就随你的便了 :-\) 。下面我们来处理注释：

```
if
(
LastChar
==
'#'
)
{
// Comment until end of line.
do
LastChar
=
getchar
();
while
(
LastChar
!=
EOF
&
&
LastChar
!=
'\n'
&
&
LastChar
!=
'\r'
);
if
(
LastChar
!=
EOF
)
return
gettok
();
}
```

注释的处理很简单：直接跳过注释所在的那一行，然后返回下一个语元即可。最后，如果碰到上述情况都处理不了的字符，那么只有两种可能：要么碰到了表示运算符的字符（比如“`+`”），要么就是已经读到了文件末尾。这两种情况由以下代码负责处理：

```
// Check for end of file.  Don't eat the EOF.
if
(
LastChar
==
EOF
)
return
tok_eof
;
// Otherwise, just return the character as its ascii value.
int
ThisChar
=
LastChar
;
LastChar
=
getchar
();
return
ThisChar
;
}
```

至此，完整的Kaleidoscope词法分析器就完成了（[完整源码](#)参见本教程的[下一章](#)）。接下来，我们将[编写一个简单的语法分析器并利用它来构建抽象语法树](#)。届时，我们还会再加上一段引导代码，让词法分析器和语法分析器珠联璧合、天衣无缝。

脚注

| [\[1\]](#) | 例如lex/yacc或flex/bison——译者注。 |
| :--- | :--- |


| [\[2\]](#) | 这里指的是编译器或解释器的前端——译者注。 |
| :--- | :--- |


| [\[3\]](#) | Spaghetti Stack，spaghetti的本意是意大利面——译者注。 |
| :--- | :--- |


| [\[4\]](#) | Kaleidoscope意即“万花筒”——译者注。 |
| :--- | :--- |


| [\[5\]](#) | 原文中用的是define；但从上下文来看此处应该是函数原型声明，而非函数定义，故改译作“声明”——译者注。 |
| :--- | :--- |


| [\[6\]](#) | 指函数A调用函数B，函数B又反过来调用函数A，从而形成递归调用的情况——译者注。 |
| :--- | :--- |




