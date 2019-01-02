我可以按照自己制定的字段命名规范去检查出所有不符合规范的变量、函数名。但是我们发现，这件事情似乎已经有人帮我们做了。。。golint。

那么还有 Doc 和 Comment，如果我们按照规范来组织这些信息，可以直接给我们的 golang 程序生成文档！然后发现这件事情官方已经做了。。。godoc

哦，还可以检查文件里有哪些没有 import 没有被用到\(pia，好好好不逗比，说正经的。

先说最简单的，如果我提供一个只有形如

    type Person struct {
        age int `json:"age"`
    }


的定义的文件，那么我们就可以通过 ParseFile 轻而易举地拿到这个 struct 的全部信息，对于我们来说，这个 struct 的字段的名字、类型、tag 就都是可见的了。

看到这里可能你还是懵逼的，我再改一下这个 struct 的定义：

    // for api @post @/user/create
    type createAccountRequest struct {
        Age int `form:"age"` validation:"gte=18"`
        Name string `form:"name" validation:"length
    >
    0"`
    }


是不是感觉恍然大悟了？

在每一个 web 开发的入口层 api，一般会做统一的参数绑定和校验，实际上这些工作大多数情况下都是让人比较烦躁的重复劳动。

比如你的代码可能是这样的：

```
func createAccount(w http.ResponseWriter, req *http.Request) {
    req.ParseForm()
    var createRequest =  createAccountRequest{}
    if createRequest.Age, ok = req.Form["age"];!ok {
        return someErrorResponse
    }
    if createRequest.Name, ok = req.Form["name"]; !ok {
        return someErrorResponse
    }
    ......
}

```

生活真是不美好，如果这个 Request 有 50 个字段，我真的有点想跳楼。

我们怎么用现在刚学到的黑科技偷懒呢？

很简单，我们只要先写一个带有 tag 的 request 的 struct 定义的 go 文件，然后对它进行 ParseFile。拿到所有顶级的GenDecl，再根据`Tok == type`? ，简单判断一下是否是我们想要的类型的文件。

然后通过遍历这些顶级的 request struct 定义，我们可以得到每个 request 对应每个 field 的 tag 下的对应的 value。有了这些 value 之后，结合 Names 和 Type 字段根据 value 就可以生成我们想要的东西了。比如这个例子里的 form 字段，我们完全可以自动生成 ParseForm，req.Age = Form\["age"\]代码。还可以根据他来生成某个 api 的 swagger 文档的 spec。至于 tag 里的 validation 字段，则要麻烦一些，我们虽然可以根据它来生成 struct 的 validation 函数\(有反射洁癖或者确实有“高性能”要求的人大概喜欢这么干，实际上 validation 有现成的基于反射的轮子\)，但 validate 的轮子造起来比较体力活。。。建议直接使用开源工具。

完成了参数的绑定和 validation，对于通常的 Web 开发  
controller 层就没有什么复杂的任务了。在生成的代码基础上，我们还可以补充一些自己的 validate 逻辑。举例来说，XXXID 可能需要去外部系统验证合法性，只要在生成的函数中补上这一步缺失逻辑即可。

这里不得不提到 golang 的类型默认值导致的麻烦问题，例如下面的 struct：

```
type req struct {
    Age int
}

```

如果你声明了这样的 struct，并把 http 的请求数据绑定到了这个 struct 的 Age 字段。

那么当 Age == 0 的时候，你就再也没有办法判断这个 0 到底是 go 的类型默认值还是调用方压根儿没有传这个参数了。解决方法倒也是有，你可以把 Age 变成`*int`，然后在每次取 req.Age 的时候都去解引用。估且算是可以解决。这个场景在一些从 0 开始计的 Type/Status 会存在这样的问题。所以请务必留意。

有点跑题。高谈阔论了一堆，我们以一个实例结尾吧：

    // example.go
    package lp

    type CollectRequest struct {
    	Star int `form:"star" validation:"gte=1,lte=5" doc:"formData"`
    }

    // parse.go
    package main

    import (
    	"fmt"
    	"go/ast"
    	"go/parser"
    	"go/token"
    	"strings"
    )

    var codeTemplate = `
    func getReqStruct(r *http.Request) (*{{requestStructName}}, error) {
    	r.ParseForm()
    	var reqStruct = 
    &
    {{requestStructName}}{}

    	// bind data
    	{{bindData}}j

    	// bind partial example
    	// reqStruct.{{fieldName}} =
    	// {{transToFieldType}}(r.Form['{{fieldTagFormName}}'])


    	if bindErr != nil {
    		return nil, err
    	}

    	// validate data
    	{{validateData}}

    	// validate partial example
    	// validateErr = validate(reqStruct.{{fieldName}}, validateStr)
    	// if validateErr != nil
    	// return nil, err


    	return reqStruct, nil
    }
    `

    func getTag(input string) []structTag {
    	var out []structTag
    	var tagStr = input
    	tagStr = strings.Replace(tagStr, "`", "", -1)
    	tagStr = strings.Replace(tagStr, "\"", "", -1)
    	tagList := strings.Split(tagStr, " ")
    	for _, val := range tagList {
    		tmpArr := strings.Split(val, ":")
    		st := structTag{}
    		st.key = tmpArr[0]
    		st.values = strings.Split(tmpArr[1], ",")
    		out = append(out, st)
    	}

    	return out
    }

    type structTag struct {
    	key    string
    	values []string
    }

    func main() {
    	fset := token.NewFileSet()
    	// if the src parameter is nil, then will auto read the second filepath file
    	f, _ := parser.ParseFile(fset, "./example.go", nil, parser.Mode(0))
    	//	ast.Print(fset, f.Decls[0])

    	tagList := getTag(f.Decls[0].(*ast.GenDecl).Specs[0].(*ast.TypeSpec).Type.(*ast.StructType).Fields.List[0].Tag.Value)
    	fieldName := f.Decls[0].(*ast.GenDecl).Specs[0].(*ast.TypeSpec).Type.(*ast.StructType).Fields.List[0].Names[0].Name
    	fieldType := f.Decls[0].(*ast.GenDecl).Specs[0].(*ast.TypeSpec).Type.(*ast.StructType).Fields.List[0].Type.(*ast.Ident).Name
    	requestStructName := f.Decls[0].(*ast.GenDecl).Specs[0].(*ast.TypeSpec).Name.Name
    	fmt.Println(tagList)
    	fmt.Println(fieldName)
    	fmt.Println(fieldType)
    	fmt.Println(requestStructName)
    }



