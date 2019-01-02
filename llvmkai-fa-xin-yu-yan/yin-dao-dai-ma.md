引导代码很简单，只需在最外层的循环中按当前语元的类型选定相应的解析函数就可以了。这段实在没什么可介绍的，我就单独把最外层循环贴出来好了。完整代码[参见下方](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-2.html#chapter-2-full-code)“Top-Level Parsing”那一段。

```
/// top ::= definition | external | expression | ';'
static
void
MainLoop
()
{
while
(
1
)
{
fprintf
(
stderr
,
"ready
>
 "
);
switch
(
CurTok
)
{
case
tok_eof
:
return
;
case
';'
:
getNextToken
();
break
;
// ignore top-level semicolons.
case
tok_def
:
HandleDefinition
();
break
;
case
tok_extern
:
HandleExtern
();
break
;
default
:
HandleTopLevelExpression
();
break
;
}
}
}
```

这段代码最有意思的地方在于我们忽略了顶层的分号。为什么呢？举个例子，当你在命令行中键入“`4+5`”后，语法解析器无法判断你键入的内容是否已经完结。如果下一行键入的是“`deffoo...`”，则可知顶层表达式就到`4+5`为止；但你也有可能会接着前面的表达式继续输入“`*6`”。有了顶层的分号，你就可以输入“`4+5;`”，于是语法解析器就能够辨别表达式在何处结束了。

## 总结

算上注释我们一共只编写了不到400行代码（去掉注释和空行后只有240行），就完整地实现了包括词法分析器、语法解析器及AST生成器在内的最基本的Kaleidoscope语言。由此编译出的可执行文件用于校验Kaleidoscope代码在语法方面的正确性。请看下面这个例子：

```
$ ./a.out
ready
>
 def foo(x y) x+foo(y, 4.0);
Parsed a function definition.
ready
>
 def foo(x y) x+y y;
Parsed a function definition.
Parsed a top-level expr
ready
>
 def foo(x y) x+y );
Parsed a function definition.
Error: unknown token when expecting an expression
ready
>
 extern sin(a);
ready
>
 Parsed an extern
ready
>
 ^D
$

```

可扩展的地方还有很多。通过定义新的AST节点，你可以按各种方式对语言进行扩展。在[下一章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-3.html)中，我们将介绍如何利通过AST生成LLVM中间语言（IR，Intermediate Representation）。

