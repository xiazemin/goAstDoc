Go语言（golang）第一个正式版Go1发布了，但是这个新兴的编程语言还是非常不完善。这不，我（Liigo）又发现它的编译器竟然不支持编译带BOM的UTF-8编码的.go源文件。这就很奇怪，该语言明明要求源代码文件.go必须是UTF-8编码，但又不允许带UTF-8 BOM。要知道，这个世界上带BOM的文件太多了，很多文本编辑器/代码编辑器/IDE支持生成、甚至默认生成带有BOM的UTF-8文件。如果仅仅因为源代码文件多了BOM，编译器就不能编译这个文件，那也太低能了。



Go语言编译器\(gc\)不支持带有BOM的UTF-8源文件：

Golang's compiler \(gc\) don't accept the .go files with UTF-8 BOM: 



E:\liigo\golang\src&gt;go run hello.go

package :

hello.go:1:1: illegal character U+FEFF



E:\liigo\golang\src&gt;go run hello.go

\# command-line-arguments

.\hello.go:9: illegal UTF-8 sequence

ce d2



　　好在Go语言是开源项目，我\(Liigo\)来贡献代码，让它支持编译带UTF-8 BOM的.go源代码文件。经过分析后发现，Go语言编译器（gc）源代码中有两处地方涉及从磁盘文件中读取.go文件：一个是C语言写的词法分析器（src/cmd/gc/lex.c），一个是Go语言写的.go文件解析器（src/pkg/go/parser/interface.go）。解决思路也很简单，就是从磁盘读取文件内容后，判断前三个字节，与UTF-8 BOM的三个字节（0xef, 0xbb, 0xbf）核对，如果一致则忽略这三个字节，从第四个字节算作文件的真正内容，然后再交给词法分析器和解析器处理，后面就一切正常了。

　　Go语言的词法分析器是用C语言手工编写的，其中用到了lib9/libbio库，是一个磁盘文件读写缓冲区，逐字节从该缓冲区读取数据时，它可以允许“反读取”最近的最多4个字节。就是说，它可以把吐出来的最后那个字节再吸回去。假设我刚刚读取了abcd是个字节，现在我“反读取”后两个字节，实际上就相当于我刚刚读取了ab还没有读取cd。利用它的这个“反读取”机制，恰恰可以很容易的忽略掉UTF-8文件最前面的BOM：首先读出前三个字节，如果这三个字节正好是UTF-8 BOM的三个字节（0xef, 0xbb, 0xbf），那么直接把刚刚读出的三个字节扔掉就完事了，后面词法分析器处理时正好从BOM后面的字节开始读取；如果已经读出的三个字节不是UTF-8 BOM呢，需要“反读取”，即把他们再放回去，就当作我没有读取过它们。修改后的代码如下：



// src/cmd/gc/lex.c : 

  319                 // Try to read and ignore UTF-8 BOM

  320                 c1 = Bgetc\(curio.bin\);

  321                 c2 = Bgetc\(curio.bin\);

  322                 c3 = Bgetc\(curio.bin\);

  323                 if\(c1 != 0xef \|\| c2 != 0xbb \|\| c3 != 0xbf\) {

  324                         // If not UTF-8 BOM, restore the bytes.

  325                         // Bungetsize &gt; 3, so we can safely call Bungetc\(\) 3 times.

  326                         Bungetc\(curio.bin\);

  327                         Bungetc\(curio.bin\);

  328                         Bungetc\(curio.bin\);

  329                 }



　　Go语言的源代码解析器\(pkg/go/parser\)是用Go语言自己编写的，其功能是解析.go源代码文件为语法树。Go语言官方提供的go build命令用pkg/go/parser分析处理编译前的库依赖项。go build命令（pkg/go/parser）是把.go文件整个读入内存后再解析的。我要做的工作就是，在pkg/go/parser开始正式解析前，把前面可能存在的UTF-8 BOM删除掉即可，这个工作仅仅涉及Go语言中byte slice的基本操作，是很轻量级的廉价操作。



// src/pkg/go/parser/interface.go : 

+// The data read from .go files maybe start with the UTF-8 BOM\(byte order mark\),

+// we ignore the bytes here to make sure that the parser parses properly.

+//

+func ignoreUTF8BOM\(data \[\]byte\) \[\]byte {

+	if data == nil {

+		return nil

+	}

+	if len\(data\) &gt;= 3 && data\[0\] == 0xef && data\[1\] == 0xbb && data\[2\] == 0xbf {

+		return data\[3:\]

+	}

+	return data

+}

+

 // If src != nil, readSource converts src to a \[\]byte if possible;

 // otherwise it returns an error. If src == nil, readSource returns

 // the result of reading the file specified by filename.

@@ -27,22 +40,23 @@

 		case string:

 			return \[\]byte\(s\), nil

 		case \[\]byte:

-			return s, nil

+			return ignoreUTF8BOM\(s\), nil

 		case \*bytes.Buffer:

 			// is io.Reader, but src is already available in \[\]byte form

 			if s != nil {

-				return s.Bytes\(\), nil

+				return ignoreUTF8BOM\(s.Bytes\(\)\), nil

 			}

 		case io.Reader:

 			var buf bytes.Buffer

 			if \_, err := io.Copy\(&buf, s\); err != nil {

 				return nil, err

 			}

-			return buf.Bytes\(\), nil

+			return ignoreUTF8BOM\(buf.Bytes\(\)\), nil

 		}

 		return nil, errors.New\("invalid source"\)

 	}

-	return ioutil.ReadFile\(filename\)

+	fileData, err := ioutil.ReadFile\(filename\)

+	return ignoreUTF8BOM\(fileData\), err

 }





　　我把上面修改的代码提交到Go语言官方源码库，代码审查页面地址是: http://codereview.appspot.com/6036054/, 或 http://codereview.appsp0t.com/6036054/。



　　但是Go语言的作者/官方开发者拒绝这一改进。Go开发组老大Rob Pike亲自回复给出拒绝的理由：



Strictly speaking, a BOM is legal in UTF-8 but only as a marker for

the type of the data stream, a magic number if you will. Since Go

source code is required to be UTF-8, a BOM is never necessary and

arguably erroneous. We've come this far without accepting BOMS and I'd

like to keep it that way.

　　在我看来，这理由非常勉强，在逻辑上甚至都不成立。哦，既然规定了.go必须使用UTF-8编码，所以就一定不能加UTF-8 BOM了？加了BOM就拒不认可了？前面已经说过了，几乎所有文本编辑器都支持UTF-8 BOM，怎么到你这里就不合法了。一个很现实的例子是，Windows XP / Windows 7系统内置的“记事本”程序（notepad.exe）在保存UTF-8文件时是一定会自动添加UTF-8 BOM的。也就是说，你想用记事本保存的.go源代码是一定不能编译通过的。但就是这么严重的问题，Go作者们就是不当一回事；这么容易就可以改进的问题，他们就是拒绝改进。生生的为用户使用go语言又凭空制造一道阻力。要说是面临技术方案的妥协选择折中还可以理解，偏偏要在无关紧要的地方坚持己见、宁死不去考虑方便用户。我只能说他们是死脑筋。类似的情况我遇到也不止一次了，总结下来就是：技术大牛挂帅做产品害人害己；闭门造车的代价是死都不知道咋死的。（这样的总结也为我们做易语言产品敲响了警钟：绝对不能单纯以技术人员的心态做产品。）

