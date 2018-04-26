# Go 语言基础

### 工程结构

* GOPATH 要指向project
* project
	* src
	* bin
	* pkg
* 工具
	* `goimports` 可以把不需要的引用包删除
	* `gofmt` 格式化代码结构
	* `go get`拉包下来到GOPATH
		* `go get -v -u` -u 更新包和依赖
		* 不加`-u`跳过更新
	* `gopm get -v -g` -g 下载到GOPATH目录

### 语言特点
* Go语言只有值传递  
* Go语言没有全局变量，只有包变量  

### 变量定义和声明
* 变量类型
	* 基本数据类型
		* bool, string
		* (u)int, (u)int8, (u)int16, (u)int32, (u)int64, uintptr
		* byte, rune
		* float32, float64, complex64, complex128
	* 容器类型
		* map
		* slice
		* 数组
	* 字符串
		* byte
		* rune

* 变量声明
	* var 变量名 类型 = 初始值 [var 变量名 = 初始值]
	* 变量名 := 初始值
	* 大堆变量定义简写   
	
	```
	var (  
		A := 初始值  
		B := 初始值  
		C := 初始值  
	)  
	```
	
	* 数组是值传递
	* 引用类型初始化
		* new
			* var *变量名 = new(类型)  [注: new分配只能给指针]
		* make
		
		```
			* var 变量名 = make([]int, length, capacity)
			* var 变量名 = make(chan int)
			* var 变量名 = make(map[string]int)
		```
		
		* 直接指向变量
			* var 变量名 = 类型{初始值}
	* 数组和切片的不同
		* 数组 arr := [...]int{2, 3, 4}
		* 切片 slice := []int{2, 3, 4}

* 字符串
	* 国际化字符串

		```
		[]rune(带汉字的字符串)
		```
* 常量
	* 常量声明
	
	```
	const filename = "abc.txt"
	const a, b = 3, 4
	const a, b int = 3, 4
	```
	
	* 命名规则
		* 想普通变量一样小写就可以
	* 枚举类型 
		* 普通枚举类型   
		
		```
		const(  
			python = 1  
			java = 2  
			golang = 3  
		)  
		```
		* 自增值

		```
		const(  
			python = iota  
			_  
			java 
			golang  
		)  
		```
			
		* 另一种自增值可以参与运算 

		```  
		const(  
			b = 1 << (10 * iota)  
			kb   
			gb   
		)  	 
		```

### 程序结构
* 分支
	* 优先使用这种形式
	
	```
	if content, err := ioutil.ReadFile(filename); err != nil {
		...
	} else {
		...
	}
	```
	
	* switch自动添加了break
	
	```
	switch op {
		case "+":
			...
		case "-"
			...
		default:
			painc(...)
	}
	```
	
	* swith后面不带表达式，判断case后面条件即可
	
	```
	op = 12
	switch {
		case op > 0:
			...
		case op >23 || op < 56
			...
		default:
			painc(...)
	}
	```
* 循环
	* for

* 面向对象
	* Go语言没有继承和多态
	* 封装
		* 靠包变量的大小写区分私有和公有变量

	* 结构
		* 声明
		
		```
			type TreeNode struct {
				value int
				left, right *treeNode
			}
		```   
		* 构造  
		
		```
			var root = TreeNode{value: 3}
		```
		* Go语言局部变量也可以给别人用，垃圾会负责这个
	* 方法
	
	```
	func (node *TreeNode) Print() {
		fmt.Println(node.value)
	}
	
	func (node *TreeNode) SetValue(int i) {
		node.value = i
	}
	```
* 面向接口
	* 对比其他语言
		* 传统语言由实现者定义
		* Go由使用者定义
	* 接口对值接受和指针接收有严格的规定
	
	```
	//实现部分
	type Mock strucr {
		content string
	}
	
	func (m Mock) Get(url string) string {
		return m.content
	}
	```
	
	```
	//使用部分
	type retriever interface {
		Get(url string)string
	} 
	
	//值接收
	var r retriever
	r = mock.Mock{"This is a mock"}
	r.Get("www.baidu.com")
	//指针接受
	r = &mock.Mock("This is a mock")
	// 错误，因为mock中的方法是值接受的
	```
	* 任意类型
		* `interface{}` 因为没有实现任何方法，所以任何类型都可以匹配
		* 判断类型使用类型断言
	* 判断接口类型
	
	```
	//只能使用switch
	switch r.(type) {
	case real.Retriver:
		fmt.Println("real ")
	case *real.Retriver:
		fmt.Println("real point")

	}
	```
	* 接口组合
	
	```
	type Mock strucr {
		content string
	}
	
	func (m Mock) Get(url string) string {
		return m.content
	}
	fuc (m Mock) Post() {
		fmt.Println(m.content)
	}
	
	//组合接口
	type web interface {
		retriever
		Post()
	}
	//使用部分
	type retriever interface {
		Get(url string)string
	} 
	
	func session(w web) {
		w.Post()
	}
	//值接收
	var r retriever
	s := mock.Mock{"This is a mock"}
	r = s
	r.Get("www.baidu.com")
	session(s)
	session(r)//错误，retriever没有Post
	```
	
	* 三个通用接口
		* Stringer接口
		
		```
		// 实现Stringer接口，打印对象时候会调用这个方法
		type retriever interface {
			Get(url string)string
			//添加接口
			fmt.Stringer
		} 
		
		type web interface {
			retriever
			Post()
			//添加接口对应的方法
			String()string
		}
		
		```
		
		* Reader接口
		
		```
		// 实现Reader接口
		type retriever interface {
			Get(url string)string
			//添加接口
			io.Reader
		} 
		
		type web interface {
			retriever
			Post()
			//添加接口对应的方法
			Read(p []byte) (n int, err error)
		}
		
		```
	* 错误处理
		* 自定义一个Error
		```
		error := errors.New("This is a costomize error")
		```
		* `panic` 处理原理
			* panic会停止当前函数的运行，一层一层往上跑调用defer函数
			* 在defer函数中遇到recover可以做处理
		* `recover`机制
			* recover只会处理panic的错误
			* recover只能放在外层defer函数中
			
			```
			defer func() {
				r := recover()
				if r != nil {

					logrus.Warning(r)
					http.Error(
						writer,
						http.StatusText(http.StatusInternalServerError),
						http.StatusInternalServerError)
				}
			}()
			```
			
		* 统一错误处理
			* 返回给用户的错误，不能暴露内部的信息，要考虑抽象层次
			* 好处：可以跑到外部统一做处理，让内部代码更干净
			* 代码区分Warnnig，Error
				* Warnning: 可以恢复的错误，不至于让这个进程终止
				* Error：可以让整个程序无法继续工作的错误
			
	* 代码测试
		* 表驱动测试法
			* 数据和代码分离
				* 在同一个包里面放置`xxx_test.go`的文件
				
				```
				import (
				"testing" //添加测试头文件
		
				"golang.org/x/tools/container/intsets"
				)
		
				func TestAdd(t *testing.T) {
					tests := []struct{ a, b, c int }{
						{1, 2, 3},
						{1, 5, 6},
						{2, 6, 8},
						{4, 2, 6},
						{9, 6, 15},
						{intsets.MaxInt, 1, intsets.MinInt},
					}
					for _, v := range tests {
						if actual := Add(v.a, v.b); actual != v.c {
							t.Errorf(
								"Add %d + %d = %d, %d expected",
								v.a,
								v.b,
								actual,
								v.c)
						}
					}
				}
				```
				* 在包里面执行`go test .`
	* 代码覆盖
		* `go test -coverproifle=c.out`生成覆盖文件
		* `go tool cover -html=c.out`在浏览器中打开覆盖文件
	* 性能测试
		* 在xxx_test.go中添加Benchmark测试函数
		
		```
		func BenchmarkAdd(b *testing.B) {
			v := struct{ a, b, c int }{intsets.MaxInt, 1, intsets.MinInt}
			//b.N测试系统决定的，这个数自动给选取合适的测试数量
			for i := 0; i < b.N; i++ {
				if actual := Add(v.a, v.b); actual != v.c {
					b.Errorf(
						"Add %d + %d = %d, %d expected",
						v.a,
						v.b,
						actual,
						v.c)
				}
			}
		}
		```
		* 命令行 `go test -bench .`
	* 性能分析
		* `go test -bench . -cpuprofile=cpu.out`   
		* `go tool pprof cpu.out`
		* 交互命令输入help，也可以直接输入web，框越大使用的时间越大
		
		```
		func BenchmarkAdd(b *testing.B) {
			xxxx//准备输入数据的时间会被忽略
			b.ResetTimer()
			v := struct{ a, b, c int }{intsets.MaxInt, 1, intsets.MinInt}
			for i := 0; i < b.N; i++ {
				if actual := Add(v.a, v.b); actual != v.c {
					b.Errorf(
						"Add %d + %d = %d, %d expected",
						v.a,
						v.b,
						actual,
						v.c)
				}
			}
		}
		
		```
		
	* http服务器测试
		* 伪造request， response
	
		```
		func TestWrapper(t *testing.T) {
		mockHander := []struct {
			h       appHander
			code    int
			message string
		}{
			{errPanic, 500, ""},
		}
	
		for _, mock := range mockHander {
			hander := wrappHander(mock.h)
			request := httptest.NewRequest(
				http.MethodGet,
				"http://www.baidu.com/lists/",
				nil)
			response := httptest.NewRecorder()
			hander(response, request)
			b, _ := ioutil.ReadAll(response.Body)
			if response.Code != mock.code ||
				mock.message != string(b) {
				t.Errorf("expected(%d, %s); (%d, %s)",
					mock.code, mock.message,
					response.Code, string(b))
			}
		}
		```
	
	* 查看竞争数据和代码
		* `go run -race`
	
	* Channel
		* 只读channel
			`<- chan int`
		* 只写channel
			* `chan <- int`
		* channel只能由发送方close，接收方可以使用ok或者range判断channel是否关闭

	* select

### http库
*  `http.Get`使用
	
	```
	func main() {
		repon, err := http.Get("http://www.baidu.com")
		if err != nil {
			panic(err)
		}
		defer repon.Body.Close()
		body, err := httputil.DumpResponse(repon, true)
		if err != nil {
			panic(err)
		}
	
		fmt.Printf("%s\n", body)
	}
	```
* 更灵活的Get

```
	func main() {
		request, err := http.NewRequest(http.MethodGet, "http://www.baidu.com", nil)
		if err != nil {
			panic(err)
		}
	
		request.Header.Add("User-Agent", "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Mobile Safari/537.36")
		resp, err := http.DefaultClient.Do(request)
		if err != nil {
			panic(err)
		}
	
		body, err := httputil.DumpResponse(resp, true)
		if err != nil {
			panic(err)
		}
		fmt.Printf("%s\n", body)
	}
```

* 检查重定向(重定向钩子)

```
	func main() {
		request, err := http.NewRequest(http.MethodGet, "http://www.baidu.com", nil)
		if err != nil {
			panic(err)
		}
	
		request.Header.Add("User-Agent", "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Mobile Safari/537.36")
		client := http.Client{CheckRedirect: func(req *http.Request, via []*http.Request) error {
			fmt.Println("Get Rediected ", req)
			return nil
		}}
		resp, err := client.Do(request)
		if err != nil {
			panic(err)
		}
		body, err := httputil.DumpResponse(resp, true)
		if err != nil {
			panic(err)
		}
		fmt.Printf("%s\n", body)
	}
```
	
* http服务器性能分析
	* 添加`_ net/http/pprof`, 需要用到包中的一些函数
	* 在url中输入localhost:/debug/pprof，可以看到性能数据
	* `go test pprof http://localhost:8888/pprof/profile` 可以查看30s cpu profile.




#### Golang项目配置选择
* 命令行配置
	* flag
* 文件配置
	* JSON
		* Why
			* 在标准库中
			* no human-readable config
			* 不能注释
			 
	* YAML
	 	* Why
	 		* 对程序员更友好
	 		* 对用户太复杂
	* Goconf
	* XML
		* Why
			* 更简洁，整齐
	* INI
		* Why
			* 可以注释