本章将介绍如何把第二章中构造的[抽象语法树](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-2.html)转换成LLVM IR。从这一章起，我们就要正式接触LLVM了，你将亲眼见证LLVM的简单便捷：跟词法分析器和语法解析器比起来，LLVM IR代码生成部分的开发工作量根本就不值一提。 :-\)

**请注意**：本章及后续章节的代码必须用2.2以上版本的LLVM才能编译。LLVM 2.1及更早的版本都不行。另外也请注意选用与你所用的LLVM版本相配套的教程：如果你采用的是LLVM官方发行的版本，请参考版本自带的文档，在[llvm.org的历史版本汇总页](http://llvm.org/docs/tutorial/LangImpl3.html)上可以找到各个版本的文档。

## 代码生成的准备工作

在开始生成LLVM IR之前，还有一些准备工作要做。首先，给每个AST类添加一个虚函数`Codegen`（code generation），用于实现代码生成：

```
/// ExprAST - Base class for all expression nodes.
class
ExprAST
{
public
:
virtual
~
ExprAST
()
{}
virtual
Value
*
Codegen
()
=
0
;
};
/// NumberExprAST - Expression class for numeric literals like "1.0".
class
NumberExprAST
:
public
ExprAST
{
double
Val
;
public
:
NumberExprAST
(
double
val
)
:
Val
(
val
)
{}
virtual
Value
*
Codegen
();
};
...
```

每种AST节点的`Codegen()`方法负责生成该类型AST节点的IR代码及其他必要信息，生成的内容以LLVM`Value`对象的形式返回。LLVM用“`Value`”类表示“[静态一次性赋值（SSA，Static Single Assignment）](http://en.wikipedia.org/wiki/Static_single_assignment_form)寄存器”或“SSA值”。SSA值最为突出的特点就在于“固定不变”：SSA值经由对应指令运算得出后便固定下来，直到该指令再次执行之前都不可修改。详情请参考[Static Single Assignment](http://en.wikipedia.org/wiki/Static_single_assignment_form)——这个概念并不难，习惯了就好。

除了在`ExprAST`类体系中添加虚方法以外，还可以利用[visitor模式](http://en.wikipedia.org/wiki/Visitor_pattern)等其他方法来实现代码生成。再次强调，本教程不拘泥于软件工程实践层面的优劣：就当前需求而言，添加虚函数是最简单的方案。

其次，我们还需要一个“`Error`”方法，该方法与语法解析器里用到的报错函数类似，用于报告代码生成过程中发生的错误（例如引用了未经声明的参数）：

```
Value
*
ErrorV
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
static
Module
*
TheModule
;
static
IRBuilder
<
>
Builder
(
getGlobalContext
());
static
std
::
map
<
std
::
string
,
Value
*
>
NamedValues
;
```

上述几个静态变量都是用于完成代码生成的。其中`TheModule`是LLVM中用于存放代码段中所有函数和全局变量的结构。从某种意义上讲，可以把它当作LLVM IR代码的顶层容器。

`Builder`是用于简化LLVM指令生成的辅助对象。[`IRBuilder`](http://llvm.org/doxygen/IRBuilder_8h-source.html)类模板的实例可用于跟踪当前插入指令的位置，同时还带有用于生成新指令的方法。

`NamedValues`映射表用于记录定义于当前作用域内的变量及与之相对应的LLVM表示（换言之，也就是代码的符号表）。在这一版的Kaleidoscope中，可引用的变量只有函数的参数。因此，在生成函数体的代码时，函数的参数就存放在这张表中。

有了这些，就可以开始进行表达式的代码生成工作了。注意，在生成代码之前必须先设置好`Builder`对象，指明写入代码的位置。现在，我们姑且假设已经万事俱备，专心生成代码即可。

