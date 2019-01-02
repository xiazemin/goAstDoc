## 定义变量和调用变量 {#定义变量和调用变量}

定义变量以及调用变量，本质可以理解为一个用一种数据结构保存数据，并在需要的时候取数据， 所以用map比较合适，可以直接用变量名作为key，值作为value。

稍微麻烦一点的是考虑作用域\(scope\)，全局的作用域，函数内的作用域、闭包等等， 可以通过类似于一个stack的层次结构实现 \(因为后进的变量，作用域更靠内，需要优先被查找\)， 先在本作用域内找变量，找不到再到外层作用域找，直到全局内找到变量，或者没找到返回错误。

关于作用域，lexical scoping和dynamic scoping以及闭包的概念，可以参见：

[http://www.yinwang.org/blog-cn/2012/08/01/interpreter](http://www.yinwang.org/blog-cn/2012/08/01/interpreter)

需要先了解一下Scheme或者Racket的语法。

## 定义变量与调用变量的实现 {#定义变量与调用变量的实现}

先不考虑作用域，将所有变量视为全局变量， 设计一个结构，存储这样的map，并提供取变量和存储变量的函数。

```
type
Environment
struct
{
envName
string
store
map
[
string
]
NodeValue
}
func
NewEnvironment
(
name
string
)
*
Environment
{
return
&
Environment
{
envName
:
name
,
store
:
make
(
map
[
string
]
NodeValue
),
}
}
func
(
e
*
Environment
)
SaveVariable
(
varName
string
,
value
NodeValue
)
{
e
.
store
[
varName
]
=
value
}
func
(
e
*
Environment
)
GetVariable
(
varName
string
)
NodeValue
{
value
,
exists
:=
e
.
store
[
varName
]
if
!
exists
{
panic
(
"variable not found"
)
}
return
value
}
```

### 声明变量与调用变量 {#声明变量与调用变量}

```
//声明变量
type
DeclarationNode
struct
{
varName
string
val
Node
env
*
Environment
}
func
NewDeclarationNode
(
environment
*
Environment
,
name
string
,
n
Node
)
*
DeclarationNode
{
return
&
DeclarationNode
{
varName
:
name
,
val
:
n
,
env
:
environment
,
}
}
func
(
n
*
DeclarationNode
)
Eval
()
NodeValue
{
value
:=
n
.
val
.
Eval
()
n
.
env
.
SaveVariable
(
n
.
varName
,
value
)
return
value
}
//获取变量
type
VariableNode
struct
{
varName
string
env
*
Environment
}
func
NewVariableNode
(
environment
*
Environment
,
name
string
)
*
VariableNode
{
return
&
VariableNode
{
varName
:
name
,
env
:
environment
,
}
}
func
(
n
*
VariableNode
)
Eval
()
NodeValue
{
return
n
.
env
.
GetVariable
(
n
.
varName
)
}
```


