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
	* `gopm get -v -g` 

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