函数原型和函数的代码生成比较繁琐，相关代码不及表达式的代码生成来得优雅，不过却刚好可以用于演示一些重要概念。首先，我们来看看函数原型的代码生成过程：函数定义和外部函数声明都依赖于它。这部分代码一开始是这样的：

```
Function
*
PrototypeAST
::
Codegen
()
{
// Make the function type:  double(double,double) etc.
std
::
vector
<
Type
*
>
Doubles
(
Args
.
size
(),
Type
::
getDoubleTy
(
getGlobalContext
()));
FunctionType
*
FT
=
FunctionType
::
get
(
Type
::
getDoubleTy
(
getGlobalContext
()),
Doubles
,
false
);
Function
*
F
=
Function
::
Create
(
FT
,
Function
::
ExternalLinkage
,
Name
,
TheModule
);
```

短短几行暗藏玄机。首先需要注意的是该函数的返回值类型是“`Function*`”而不是“`Value*`”。“函数原型”描述的是函数的对外接口（而不是某表达式计算出的值），返回代码生成过程中与之相对应的LLVM`Function`自然也合情合理。

`FunctionType::get`调用用于为给定的函数原型创建对应的`FunctionType`对象。在Kaleidoscope中，函数的参数全部都是`double`，因此第一行创建了一个包含“N”个LLVM`double`的`vector`。随后，`FunctionType::get`方法以这“N”个`double`为参数类型、以单个`double`为返回值类型，创建出一个参数个数不可变（最后一个参数`false`就是这个意思）的函数类型。注意，和常数一样，LLVM中的类型对象也是单例，应该用“`get`”而不是“`new`”来获取。

最后一行实际上创建的是与该函数原型相对应的函数。其中包含了类型、链接方式和函数名等信息，还指定了该函数待插入的模块。“[`ExternalLinkage`](http://llvm.org/docs/LangRef.html#linkage)”表示该函数可能定义于当前模块之外，且/或可以被当前模块之外的函数调用。`Name`是用户指定的函数名：如上述代码中的调用所示，既然将函数定义在“`TheModule`”内，函数名自然也注册在“`TheModule`”的符号表内。

```
// If F conflicted, there was already something named 'Name'.  If it has a
// body, don't allow redefinition or reextern.
if
(
F
-
>
getName
()
!=
Name
)
{
// Delete the one we just made and get the existing one.
F
-
>
eraseFromParent
();
F
=
TheModule
-
>
getFunction
(
Name
);
```

在处理名称冲突时，`Module`的符号表与`Function`的符号表类似：在模块中添加新函数时，如果发现函数名与符号表中现有的名称重复，新函数会被默默地重命名。上述代码用于检测函数有否被定义过。

对于Kaleidoscope，在两种情况下允许重定义函数：第一，允许对同一个函数进行多次`extern`声明，前提是所有声明中的函数原型保持一致（由于只有一种参数类型，我们只需要检查参数的个数是否匹配即可）。第二，允许先对函数进行`extern`声明，再定义函数体。这样一来，才能定义出相互递归调用的函数。

为了实现这些功能，上述代码首先检查是否存在函数名冲突。如果存在，（调用`eraseFunctionParent`）将刚刚创建的函数对象删除，然后调用`getFunction`获取与函数名相对应的函数对象。请注意，LLVM中有很多`erase`形式和`remove`形式的API。`remove`形式的API只会将对象从父对象处摘除并返回。`erase`形式的API不仅会摘除对象，还会将之删除。

```
// If F already has a body, reject this.
if
(
!
F
-
>
empty
())
{
ErrorF
(
"redefinition of function"
);
return
0
;
}
// If F took a different number of args, reject.
if
(
F
-
>
arg_size
()
!=
Args
.
size
())
{
ErrorF
(
"redefinition of function with different # args"
);
return
0
;
}
```

为了在上述代码的基础上进一步进行校验，我们来看看之前定义的函数对象是否为“空”。换言之，也就是看看该函数有没有定义基本块。没有基本块就意味着该函数尚未定义函数体，只是一个前导声明。如果已经定义了函数体，就不能继续下去了，抛出错误予以拒绝。如果之前的函数对象只是个“`extern`”声明，则检查该函数的参数个数是否与当前的参数个数相符。如果不符，抛出错误。

```
// Set names for all arguments.
unsigned
Idx
=
0
;
for
(
Function
::
arg_iterator
AI
=
F
-
>
arg_begin
();
Idx
!=
Args
.
size
();
++
AI
,
++
Idx
)
{
AI
-
>
setName
(
Args
[
Idx
]);
// Add arguments to variable symbol table.
NamedValues
[
Args
[
Idx
]]
=
AI
;
}
```

最后，遍历函数原型的所有参数，为这些LLVM`Argument`对象逐一设置参数名，并将这些参数注册倒`NamedValues`映射表内，以备AST节点类`VariableExprAST`稍后使用。完事之后，将`Function`对象返回。注意，此处并不检查参数名冲突与否（说的是“`externfoo(aba`”这样的情况）。按照之前的讲解，要加上这一重检查易如反掌。

```
Function
*
FunctionAST
::
Codegen
()
{
NamedValues
.
clear
();
Function
*
TheFunction
=
Proto
-
>
Codegen
();
if
(
TheFunction
==
0
)
return
0
;
```

下面是函数定义的代码生成过程，开场白很简单：生成函数原型（`Proto`）的代码并进行校验。与此同时，需要清空`NamedValues`映射表，确保其中不会残留之前代码生成过程中的产生的内容。函数原型的代码生成完毕后，一个现成的LLVM`Function`对象就到手了。

```
// Create a new basic block to start insertion into.
BasicBlock
*
BB
=
BasicBlock
::
Create
(
getGlobalContext
(),
"entry"
,
TheFunction
);
Builder
.
SetInsertPoint
(
BB
);
if
(
Value
*
RetVal
=
Body
-
>
Codegen
())
{
```

现在该开始设置`Builder`对象了。第一行新建了一个名为“entry”的[基本块](http://en.wikipedia.org/wiki/Basic_block)对象，稍后该对象将被插入`TheFunction`。第二行告诉`Builder`，后续的新指令应该插至刚刚新建的基本块的末尾处。LLVM基本块是用于定义[控制流图（Control Flow Graph）](http://en.wikipedia.org/wiki/Control_flow_graph)的重要部件。当前我们还不涉及到控制流，所以所有的函数都只有一个基本块。这个问题我们留到[第五章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-5.html#chapter-5)再改 :-\)

```
if
(
Value
*
RetVal
=
Body
-
>
Codegen
())
{
// Finish off the function.
Builder
.
CreateRet
(
RetVal
);
// Validate the generated code, checking for consistency.
verifyFunction
(
*
TheFunction
);
return
TheFunction
;
}
```

选好插入点后，调用函数主表达式的`CodeGen()`方法。不出意外的话，

