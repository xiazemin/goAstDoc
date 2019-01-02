开始构造AST之前，先要准备好用于构造AST的语法解析器。说白了，就是要利用语法解析器把“`x+y`”这样的输入（由词法分析器返回的三个语元）分解成由下列代码生成的AST：

```
ExprAST
*
X
=
new
VariableExprAST
(
"x"
);
ExprAST
*
Y
=
new
VariableExprAST
(
"y"
);
ExprAST
*
Result
=
new
BinaryExprAST
(
'+'
,
X
,
Y
);
```

为此，我们先定义几个辅助函数：

```
/// CurTok/getNextToken - Provide a simple token buffer.  CurTok is the current
/// token the parser is looking at.  getNextToken reads another token from the
/// lexer and updates CurTok with its results.
static
int
CurTok
;
static
int
getNextToken
()
{
return
CurTok
=
gettok
();
}
```

这段代码以词法分析器为中心，实现了一个简易的语元缓冲，让我们能够预先读取词法分析器将要返回的下一个语元。在我们的语法解析器中，所有函数都将`CurTok`视作当前待解析的语元。

```
/// Error* - These are little helper functions for error handling.
ExprAST
*
Error
(
const
char
*
Str
)
{
fprintf
(
stderr
,
"Error: %s
\n
"
,
Str
);
return
0
;}
PrototypeAST
*
ErrorP
(
const
char
*
Str
)
{
Error
(
Str
);
return
0
;
}
FunctionAST
*
ErrorF
(
const
char
*
Str
)
{
Error
(
Str
);
return
0
;
}
```

这三个用于报错的辅助函数也很简单，我们的语法解析器将用它们来处理解析过程中发生的错误。这里采用的错误恢复策略并不妥当，对用户也不怎么友好，但对于教程而言也就够用了。示例代码中各个函数的返回值类型各不相同，有了这几个函数，错误处理就简单了：它们的返回值统统是`NULL`。

准备好这几个辅助函数之后，我们就开始实现第一条语法规则：数值常量。

