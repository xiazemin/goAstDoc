下面来解析函数原型。在Kaleidoscope语言中，有两处会用到函数原型：一是“`extern`”函数声明，二是函数定义。相关代码很简单，没太大意思（相对于解析表达式的代码而言）：

```
/// prototype
///   ::= id '(' id* ')'
static
PrototypeAST
*
ParsePrototype
()
{
if
(
CurTok
!=
tok_identifier
)
return
ErrorP
(
"Expected function name in prototype"
);
std
::
string
FnName
=
IdentifierStr
;
getNextToken
();
if
(
CurTok
!=
'('
)
return
ErrorP
(
"Expected '(' in prototype"
);
std
::
vector
<
std
::
string
>
ArgNames
;
while
(
getNextToken
()
==
tok_identifier
)
ArgNames
.
push_back
(
IdentifierStr
);
if
(
CurTok
!=
')'
)
return
ErrorP
(
"Expected ')' in prototype"
);
// success.
getNextToken
();
// eat ')'.
return
new
PrototypeAST
(
FnName
,
ArgNames
);
}
```

在此基础之上，函数定义就很简单了，说白了就是一个函数原型再加一个用作函数体的表达式：

```
/// definition ::= 'def' prototype expression
static
FunctionAST
*
ParseDefinition
()
{
getNextToken
();
// eat def.
PrototypeAST
*
Proto
=
ParsePrototype
();
if
(
Proto
==
0
)
return
0
;
if
(
ExprAST
*
E
=
ParseExpression
())
return
new
FunctionAST
(
Proto
,
E
);
return
0
;
}
```

除了用于用户自定义函数的前置声明，“`extern`”语句还可以用来声明“`sin`”、“`cos`”等（C标准库）函数。这些“`extern`”语句不过就是些不带函数体的函数原型罢了：

```
/// external ::= 'extern' prototype
static
PrototypeAST
*
ParseExtern
()
{
getNextToken
();
// eat extern.
return
ParsePrototype
();
}
```

最后，我们还允许用户随时在顶层输入任意表达式并求值。这一特性是通过一个特殊的匿名零元函数（没有任何参数的函数）实现的，所有顶层表达式都定义在这个函数之内：

```
/// toplevelexpr ::= expression
static
FunctionAST
*
ParseTopLevelExpr
()
{
if
(
ExprAST
*
E
=
ParseExpression
())
{
// Make an anonymous proto.
PrototypeAST
*
Proto
=
new
PrototypeAST
(
""
,
std
::
vector
<
std
::
string
>
());
return
new
FunctionAST
(
Proto
,
E
);
}
return
0
;
}
```

现在所有零部件都准备完毕了，只需再编写一小段引导代码就可以跑起来了！

