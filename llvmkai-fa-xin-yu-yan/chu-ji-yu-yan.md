## 初级语言

本教程介绍了一门玩具语言，名叫“[Kaleidoscope](http://en.wikipedia.org/wiki/Kaleidoscope)”（取“美仑美奂、形态万千、多姿多彩”之意）[\[4\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id20)。Kaleidoscope是一个过程式语言，用户可以定义函数、使用条件语句、执行数学运算等等。随着教程的深入，我们将逐步扩展Kaleidoscope，为它增加`if`/`then`/`else`结构、`for`循环、用户自定义运算符，以及带有简单命令行界面的JIT编译器等等。

简单起见，Kaleidoscope只支持一种数据类型，即64位浮点数（也就是C中的“double”）。这样一来，所有的值都是双精度浮点数，类型申明也省了，语言的语法简洁明快。以下是一个用于计算[斐波那契数](http://en.wikipedia.org/wiki/Fibonacci_number)的简单示例：

```
# Compute the x'th fibonacci number.
def
fib
(
x
)
if
x
<
3
then
1
else
fib
(
x
-
1
)
+
fib
(
x
-
2
)
# This expression will compute the 40th number.
fib
(
40
)
```

Kaleidoscope还能调用标准库函数（有了LLVM JIT，实现这一点简单至极）。不过，在调用函数之前需要先用“`extern`”关键字对函数进行申明[\[5\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id21)（碰到相互递归调用的函数[\[6\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id22)时也需要这么做）。例如：

```
extern
sin
(
arg
);
extern
cos
(
arg
);
extern
atan2
(
arg1
arg2
);
atan2
(
sin
(
.
4
),
cos
(
42
))
```

[第六章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-6.html)的例子更有意思：我们用Kaleidoscope编写了一个小应用，可以按不同尺度[显示Mandelbrot集合](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-6.html#kicking-the-tires)。

让我们来细细品味一下这门语言的实现过程吧！

## 



