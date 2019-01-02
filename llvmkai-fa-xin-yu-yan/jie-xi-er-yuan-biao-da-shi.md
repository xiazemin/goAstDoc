二元表达式的解析难度要大得多，因为它们往往具有二义性。例如，给定字符串“`x+y*z`”，语法解析器既可以将之解析为“`(x+y)*z`”，也可以将之解析为“`x+(y*z)`”。按照通常的数学定义，我们期望解析成后者，因为“`*`”（乘法）的优先级要高于“`+`”（加法）。

这个问题的解法很多，其中属[运算符优先级解析](http://en.wikipedia.org/wiki/Operator-precedence_parser)最为优雅和高效。这是一种利用二元运算符的优先级来引导递归调用走向的解析技术。首先，我们需要制定一张优先级表：

```
/// BinopPrecedence - This holds the precedence for each binary operator that is
/// defined.
static
std
::
map
<
char
,
int
>
BinopPrecedence
;
/// GetTokPrecedence - Get the precedence of the pending binary operator token.
static
int
GetTokPrecedence
()
{
if
(
!
isascii
(
CurTok
))
return
-
1
;
// Make sure it's a declared binop.
int
TokPrec
=
BinopPrecedence
[
CurTok
];
if
(
TokPrec
<
=
0
)
return
-
1
;
return
TokPrec
;
}
int
main
()
{
// Install standard binary operators.
// 1 is lowest precedence.
BinopPrecedence
[
'
<
'
]
=
10
;
BinopPrecedence
[
'+'
]
=
20
;
BinopPrecedence
[
'-'
]
=
20
;
BinopPrecedence
[
'*'
]
=
40
;
// highest.
...
}
```

最基本的Kaleidoscope语言仅支持4种二元运算符（对于我们英勇无畏的读者来说，再加几个运算符自然是小菜一碟）。函数`GetTokPrecedence`用于查询当前语元的优先级，如果当前语元不是二元运算符则返回`-1`。这里的`map`简化了新运算符的添加，同时也可以证明我们的算法与具体的运算符无关。当然，要想去掉`map`直接在`GetTokPrecedence`中比较优先级也很简单。（甚至可以直接使用定长数组）。

有了上面的函数作为辅助，我们就可以开始解析二元表达式了。运算符优先级解析的基本思想就是通过拆解含有二元运算符的表达式来解决可能的二义性问题。以表达式“`a+b+(c+d)*e*f+g`”为例，在进行运算符优先级解析时，它将被视作一串按二元运算符分隔的主表达式。按照这个思路，解析出来的第一个主表达式应该是“`a`”，紧跟着是若干个有序对，即：`[+,b]`、`[+,(c+d)]`、`[*,e]`、`[*,f]`和`[+,g]`。注意，括号表达式也是主表达式，所以在解析二元表达式时无须特殊照顾`(c+d)`这样的嵌套表达式。

一开始，每个表达式都由一个主表达式打头阵，身后可能还跟着一串由有序对构成的列表，其中有序对的格式为`[binop,primaryexpr]`：

```
/// expression
///   ::= primary binoprhs
///
static
ExprAST
*
ParseExpression
()
{
ExprAST
*
LHS
=
ParsePrimary
();
if
(
!
LHS
)
return
0
;
return
ParseBinOpRHS
(
0
,
LHS
);
}
```

函数`ParseBinOpRHS`用于解析有序对列表（其中`RHS`是Right Hand Side的缩写，表示“右侧”；与此相对应，`LHS`表示“左侧”——译者注）。它的参数包括一个整数和一个指针，其中整数代表运算符优先级，指针则指向当前已解析出来的那部分表达式。注意，单独一个“`x`”也是合法的表达式：也就是说`binoprhs`有可能为空；碰到这种情况时，函数将直接返回作为参数传入的表达式。在上面的例子中，传入`ParseBinOpRHS`的表达式是“`a`”，当前语元是“`+`”。

传入`ParseBinOpRHS`的优先级表示的是该函数所能处理的**最低运算符优先级**。假设语元流中的下一对是“`[+,x]`”，且传入`ParseBinOpRHS`的优先级是`40`，那么该函数将直接返回（因为“`+`”的优先级是`20`）。搞清楚这一点之后，我们再来看`ParseBinOpRHS`的定义，函数的开头是这样的：

```
/// binoprhs
///   ::= ('+' primary)*
static
ExprAST
*
ParseBinOpRHS
(
int
ExprPrec
,
ExprAST
*
LHS
)
{
// If this is a binop, find its precedence.
while
(
1
)
{
int
TokPrec
=
GetTokPrecedence
();
// If this is a binop that binds at least as tightly as the current binop,
// consume it, otherwise we are done.
if
(
TokPrec
<
ExprPrec
)
return
LHS
;
```

这段代码首先检查当前语元的优先级，如果优先级过低就直接返回。由于无效语元（这里指不是二元运算符的语元——译者注）的优先级都被判作`-1`，因此当语元流中的所有二元运算符都被处理完毕时，该检查自然不会通过。如果检查通过，则可知当前语元一定是二元运算符，应该被纳入当前表达式：

```
// Okay, we know this is a binop.
int
BinOp
=
CurTok
;
getNextToken
();
// eat binop
// Parse the primary expression after the binary operator.
ExprAST
*
RHS
=
ParsePrimary
();
if
(
!
RHS
)
return
0
;
```

就这样，二元运算符处理完毕（并保存妥当）之后，紧跟其后的主表达式也随之解析完毕。至此，本例中的第一对有序对`[+,b]`就构造完了。

现在表达式的左侧和`RHS`序列中第一对都已经解析完毕，该考虑表达式的结合次序了。路有两条，要么选择“`(a+b)binopunparsed`”，要么选择“`a+(bbinopunparsed)`”。为了搞清楚到底该怎么走，我们先预读出“`binop`”，查出它的优先级，再将之与`BinOp`（本例中是“`+`”）的优先级相比较：

```
// If BinOp binds less tightly with RHS than the operator after RHS, let
// the pending operator take RHS as its LHS.
int
NextPrec
=
GetTokPrecedence
();
if
(
TokPrec
<
NextPrec
)
{
```

`binop`位于“`RHS`”的右侧，如果`binop`的优先级低于或等于当前运算符的优先级，则可知括号应该加在前面，即按“`(a+b)binop...`”处理。在本例中，当前运算符是“`+`”，下一个运算符也是“`+`”，二者的优先级相同。既然如此，理应按照“`a+b`”来构造AST节点，然后我们继续解析：

```
// ... if body omitted ...
}
// Merge LHS/RHS.
LHS
=
new
BinaryExprAST
(
BinOp
,
LHS
,
RHS
);
}
// loop around to the top of the while loop.
}
```

接着上面的例子，“`a+b+`”的前半段被解析成了“`(a+b)`”，于是“`+`”成为当前语元，进入下一轮迭代。上述代码进而将“`(c+d)`”识别为主表达式，并构造出相应的有序对`[+,(c+d)]`。现在，主表达式右侧的`binop`是“`*`”，由于“`*`”的优先级高于“`+`”，负责检查运算符优先级的`if`判断通过，执行流程得以进入`if`语句的内部。

现在关键问题来了：`if`语句内的代码怎样才能完整解析出表达式的右半部分呢？尤其是，为了构造出正确的AST，变量`RHS`必须完整表达“`(c+d)*e*f`”。出人意料的是，写成代码之后，这个问题出奇地简单：

```
// If BinOp binds less tightly with RHS than the operator after RHS, let
// the pending operator take RHS as its LHS.
int
NextPrec
=
GetTokPrecedence
();
if
(
TokPrec
<
NextPrec
)
{
RHS
=
ParseBinOpRHS
(
TokPrec
+
1
,
RHS
);
if
(
RHS
==
0
)
return
0
;
}
// Merge LHS/RHS.
LHS
=
new
BinaryExprAST
(
BinOp
,
LHS
,
RHS
);
}
}
```

看一下主表达式右侧的二元运算符，我们发现它的优先级比当前正在解析的`binop`的优先级要高。由此可知，如果自`binop`以右的若干个连续有序对都含有优先级高于“`+`”的运算符，那么就应该把它们全部解析出来，拼成“`RHS`”后返回。为此，我们将最低优先级设为“`TokPrec+1`”，递归调用函数“`ParseBinOpRHS`”。该调用会完整解析出上述示例中的“`(c+d)*e*f`”，并返回构造出的AST节点，这个节点就是“`+`”表达式右侧的`RHS`。

最后，`while`循环的下一轮迭代将会解析出剩下的“`+g`”并将之纳入AST。仅需区区14行代码，我们就完整而优雅地实现了通用的二元表达式解析算法。上述讲解比较简略，这段代码还是颇有些难度的。我建议你找些棘手的例子多跑跑看，好彻底搞明白这段代码背后的原理。

表达式的解析就此告一段落。现在，我们可以将任意语元流喂入语法解析器并逐步从中构造出表达式，直到解析至不属于表达式的语元为止。接下来，我们来处理函数定义等其他结构。

