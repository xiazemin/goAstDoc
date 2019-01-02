本教程的目标是循序渐进地描绘一门语言，并详述它的开发过程。在这一过程中，我们将针对语言设计以及LLVM的用法等问题展开讨论。与此同时，我们会给出代码并进行讲解，你免各种细节把你弄得晕头转向。

需要提前说明的是，这一教程讲授的是编译器技术和LLVM，**不是**四平八稳的现代化软件工程准则。换言之，方便起见，我们会采用一些不太正规的手法。譬如对内存泄漏孰视无睹、滥用全局变量、无视[visitor](http://en.wikipedia.org/wiki/Visitor_pattern)等成熟设计模式等等……一切从简。如果你有意深究，并打算把这些代码用作今后项目的基础，这些毛病也不难修。

我试着编排了本教程的各个章节，碰到熟悉的或是不感兴趣的章节，你大可直接跳过。本教程的结构如下：

* [第一章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#)：**Kaleidoscope语言及其词法解析器简介**

  阐明我们的目的，并明确新语言应该具备哪些基本功能。为了让这份教程尽量易于理解、易于入手，我们决定不采用任何词法分析器和语法分析器的生成器[\[1\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id17)，而是直接用C++实现所有功能。当然，LLVM与这些工具配合使用也完全没问题，如果愿意，你大可放手一试。

* [第二章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-2.html)：**实现语法分析器和AST**

  完成词法分析器之后，我们就开始讨论语法解析技术，进而开始构造基本的AST。这份教程共介绍了递归下降解析和运算符优先级解析两种解析技术。[第一章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#)和[第二章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-2.html)的内容与LLVM完全无关，至此，我们的代码甚至都不需要链接LLVM库。 :-\)

* [第三章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-3.html)：**LLVM IR代码生成**

  搞定AST之后，我们忍不住要炫耀一番，让你看看LLVM IR的生成过程是多么的简单。

* [第四章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-4.html)：**添加JIT和优化支持**

  很多人都有意将LLVM用作JIT，有鉴于此，我们打算在此深入一下，让你见识一下区区三行代码搞定JIT支持。尽管LLVM在其他方面也应用广泛，但说到炫技，还是这三行代码最简单、最“惊艳”。

* [第五章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-5.html)：**语言扩展：流程控制**

  我们的语言已经可以运行了，现在来看看如何为它添加流程控制能力（即`if`/`then`/`else`和“`for`”循环）。在此我们还将借机介绍一下简单的[SSA](http://en.wikipedia.org/wiki/Static_single_assignment_form)构造和流程控制。

* [第六章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-6.html)：**语言扩展：用户自定义运算符**

  这一章用处不大，但却很有意思。我们再次对语言进行了扩展，赋予了用户在程序中自行定义任意一元和二元运算符的能力（还可以设置优先级！）。这一机制使得我们得以将该“语言”的很大一部分实现成库函数。

* [第七章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-7.html)：**语言扩展：可变变量**

  本章介绍的是用户自定义局部变量和赋值运算符的实现。最有意思的地方在于，在LLVM中构造SSA form简单至极：不，实际上LLVM压根儿就**不**要求你在前端[\[2\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id18)构造SSA form！

* [第八章](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-8.html)：**结论以及其他和LLVM有关的内容**

  本章对整个系列教程作出了总结，不仅讨论了多种潜在的语言扩展方向，还针对一系列“专题”给出了参考资料，例如垃圾回收支持、异常、调试、“[麻花栈](http://en.wikipedia.org/wiki/Spaghetti_stack)[\[3\]](https://llvm-tutorial-cn.readthedocs.io/en/latest/chapter-1.html#id19)”支持等，此外，还提到了很多其他的窍门和技巧。

截至本教程末尾为止，不算注释和空行，我们总共只需编写不到700行代码。面对这样一门相对复杂的语言，只用了这么点儿代码就我们实现了一款颇为像样的编译器，其中包括手工打造的词法分析器、语法分析器、AST，还实现了带JIT编译器的代码生成。我想，相对于其他系统给出的那些“hello world”级别的教程，这篇教程的广度足以诠释LLVM的强大；如果你对语言和编译器设计感兴趣，它应该能够说服你认真考虑LLVM。

最后说一句：我们希望你能够进一步扩展这一语言，好好玩味一番。拿着代码疯狂地hack去吧，编译器并非令人恐惧的怪兽——把玩程序语言，奇乐无穷！

