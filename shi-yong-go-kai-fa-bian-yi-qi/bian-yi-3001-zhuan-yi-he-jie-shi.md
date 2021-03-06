编译

编译是直将编写的代码从一个语言翻译为另一个更低层级语言的过程。一个 C 编译器其实并不会直接输出机器码。而是将 C 代码翻译为汇编语言。汇编编译器获取这些内容编译为机器码。C\# 和 Java 会翻译为字节码。字节码在虚拟机运行的时候才会被转换为机器码。



理解这其中的差异非常重要。



编译经常会伴随中间代码（IR）或中间语言的使用。汇编是一个很常见的中间语言。LLVM 的 IR 通常叫做 LLVM IR。C 也会作为中间语言出现。



转译

对照来说，转译是将代码从一个语言翻译到另一个同样层级的语言。例如将 Go 翻译为 Javascript。



不要它与类似 Scala 或 Clojure 这样的语言的行为混淆了。这些语言都是直接编译为 Java 字节码，然后使用 JVM 来执行它们。由于 Java 字节码是一个更低层次的抽象，所以它们不被认为是转译。在编译到字节码之前首先翻译为 Java，这种可以认为是使用转译的语言。



将 Go 或 Lisp 翻译到 C 也不是真正意义上的转译，尽管界限不是很清晰。C 虽然相对于汇编来说是层级要高，但是它仍然是一个低层级语言。



解释

解释器通常与脚本语言相关。通常直接进行解释，而不是将一个语言翻译为另一种语言或 IR。



在某些情况下，这意味着字面直译，扫描代码然后解释执行，直到出错为止。另一些情况下，意味着扫描整个代码，校验并将全部信息保存到一个树状数据结构，以便执行。



某种意义上来说，这种类型的解释器在运行时，每次都必须“编译”脚本或源代码，这使得它变慢。这是因为每次执行，它都需要完成编译过程的全部步骤，而不是只运行一次这个过程。



然而现代解释器通常不这么做。如果你对细节感兴趣，你可能需要研究以下 JIT（Just In Time） 编译。简单来说，脚本仅会被解释一次，就像编译器那样，然后存储在某些中间表中，这样可以更快的进行加载。这一过程同样允许设计者进行优化来给代码进一步提速。



二进制编译

编译器的目标通常就是创建一个可执行的二进制文件，或者某些及其可以读取和执行的对象。



但这不仅仅是创建机器码这么简单。编译实际上只是第一步。



这里有一个 GCC 和 Go 编译其代码的基本步骤：



使用 GCC 的 C：



通过 GNU C 编译器（gcc）将 C 翻译为汇编；

通过 GNU 汇编器（gas）将汇编翻译为机器码；

通过 GNU 链接器\(ld\)将机器码和标准库进行链接，生成特定架构下的二进制文件。

在 x86-32 上 Go 使用 Go 编译器：



通过 Go 编译器（8g）将 Go 翻译为汇编；

通过 Plan 9 汇编器（8a）将汇编翻译为机器码；

通过 Plan 9 链接器\(8l\)将机器码和标准库进行链接，生成特定架构下的二进制文件。

如你所见，编译器只是整个过程的第一步。在这一系列文章中，我们只会处理第一步。汇编编译和链接已经超出了这些文章的范畴。



别着急！我们的编译器会生成可执行的二进制文件！

