之所以先从数值常量下手，是因为它最简单。Kaleidoscope语法中的每一条生成规则（production），都需要一个对应的解析函数。对于数值常量，就是：

```
/// numberexpr ::= number
static
ExprAST
*
ParseNumberExpr
()
{
ExprAST
*
Result
=
new
NumberExprAST
(
NumVal
);
getNextToken
();
// consume the number
return
Result
;
}
```

这个函数很简单：调用它的时候，当前待解析语元只能是`tok_number`。该函数用刚解析出的数值构造出了一个`NumberExprAST`节点，然后令词法分析器继续读取下一个语元，最后返回构造的AST节点。

这里有几处很有意思，其中最显著的便是该函数的行为，它不仅消化了所有与当前生成规则相关的所有语元，还把下一个待解析的语元放进了词法分析器的语元缓冲（该语元与当前的生成规则无关）。这是非常标准的递归下降解析器的行为。下面这个括号运算符的例子更能说明问题：

```
/// parenexpr ::= '(' expression ')'
static
ExprAST
*
ParseParenExpr
()
{
getNextToken
();
// eat (.
ExprAST
*
V
=
ParseExpression
();
if
(
!
V
)
return
0
;
if
(
CurTok
!=
')'
)
return
Error
(
"expected ')'"
);
getNextToken
();
// eat ).
return
V
;
}
```

该函数体现了这个语法解析器的几个特点：

1. 它展示了
   `Error`
   函数的用法。调用该函数时，待解析的语元只能是“
   `(`
   ”，然而解析完子表达式之后，紧跟着的语元却不一定是“
   `)`
   ”。比如，要是用户输入的是“
   `(4`
   `x`
   ”而不是“
   `(4)`
   ”，语法解析器就应该报错。既然错误时有发生，语法解析器就必须提供一条报告错误的途径：就这个语法解析器而言，应对之道就是返回
   `NULL`
   。
2. 该函数的另一特点在于递归调用了
   `ParseExpression`
   （很快我们就会看到
   `ParseExpression`
   还会反过来调用
   `ParseParenExpr`
   ）。这种手法简化了递归语法的处理，每一条生成规则的实现都得以变得非常简洁。需要注意的是，我们没有必要为括号构造AST节点。虽然这么做也没错，但括号的作用主要还是对表达式进行分组进而引导语法解析过程。当语法解析器构造完AST之后，括号就没用了。

下一条生成规则也很简单，它负责处理变量引用和函数调用：

```
/// identifierexpr
///   ::= identifier
///   ::= identifier '(' expression* ')'
static
ExprAST
*
ParseIdentifierExpr
()
{
std
::
string
IdName
=
IdentifierStr
;
getNextToken
();
// eat identifier.
if
(
CurTok
!=
'('
)
// Simple variable ref.
return
new
VariableExprAST
(
IdName
);
// Call.
getNextToken
();
// eat (
std
::
vector
<
ExprAST
*
>
Args
;
if
(
CurTok
!=
')'
)
{
while
(
1
)
{
ExprAST
*
Arg
=
ParseExpression
();
if
(
!
Arg
)
return
0
;
Args
.
push_back
(
Arg
);
if
(
CurTok
==
')'
)
break
;
if
(
CurTok
!=
','
)
return
Error
(
"Expected ')' or ',' in argument list"
);
getNextToken
();
}
}
```

该函数与其它函数的风格别无二致。（调用该函数时当前语元必须是`tok_identifier`。）前文提到的有关递归和错误处理的特点它统统具备。有意思的是这里采用了**预读**（lookahead）的手段来试探当前标识符的类型，判断它究竟是个独立的变量引用还是个函数调用。只要检查紧跟标识符之后的语元是不是“`(`”，它就能知道到底应该构造`VariableExprAST`节点还是`CallExprAST`节点。

现在，解析各种表达式的代码都已经完成，不妨再添加一个辅助函数，为它们梳理一个统一的入口。我们将上述表达式称为**主**表达式（primary expression），具体原因参见本教程的[后续章节](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-6.html#user-defined-unary-operators)。在解析各种主表达式时，我们首先要判定待解析表达式的类别：

```
/// primary
///   ::= identifierexpr
///   ::= numberexpr
///   ::= parenexpr
static
ExprAST
*
ParsePrimary
()
{
switch
(
CurTok
)
{
default
:
return
Error
(
"unknown token when expecting an expression"
);
case
tok_identifier
:
return
ParseIdentifierExpr
();
case
tok_number
:
return
ParseNumberExpr
();
case
'('
:
return
ParseParenExpr
();
}
}
```

看完这个函数的定义，你就能明白为什么先前的各个函数能够放心大胆地对`CurTok`的取值作出假设了。这里预读了下一个语元，预先对待解析表达式的类型作出了判断，然后才调用相应的函数进行解析。

基本表达式全都搞定了，下面开始开始着手解析更为复杂的二元表达式。

