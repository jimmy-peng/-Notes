# Go 语言基础

* Go语言没有全局变量，只有包变量
* 变量声明
	* var 变量名 类型 = 初始值 [var 变量名 = 初始值]
	* 变量名 := 初始值
	* 大堆变量定义简写   
	var (  
		A := 初始值  
		B := 初始值  
		C := 初始值  
	)  

	* 引用类型初始化
		* new
			* var *变量名 = new(类型)  [注: new分配只能给指针]
		* make
			* var 变量名 = make([]int, length, capacity)
			* var 变量名 = make(chan int)
			* var 变量名 = make(map[string]int)
		* 直接指向变量
			* var 变量名 = 类型{初始值}
			
* 变量类型
	* bool, string
	* (u)int, (u)int8, (u)int16, (u)int32, (u)int64, uintptr
	* byte, rune
	* float32, float64, complex64, complex128

* 常量
	* const filename = "abc.txt"
	* const a, b = 3, 4[const a, b int = 3, 4]
	* 命名规则
		* 想普通变量一样小写就可以
	* 枚举类型 
		* 普通枚举类型   
		const(  
			python = 1  
			java = 2  
			golang = 3  
		)  
		* 自增值
		const(  
			python = iota  
			_  
			java 
			golang  
		)  	
		* 另一种自增值可以参与运算   
		const(  
			b = 1 << (10 * iota)  
			kb   
			gb   
		)  	 

---
### 程序结构

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