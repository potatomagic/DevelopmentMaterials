# Hello Go ！
```
package main

import "fmt"

func main() {
  /* helloworld */
   fmt.Println("Hello, World!")
}
```

- package main 定义了包名，必须在源文件中非注释的第一行指明这个 go 文件属于哪个包，每一个 Go 应用程序都包含一个名为 main 的包。
- import "fmt" 表示导入 fmt 包，fmt 包实现了格式化 IO 的函数。
- func main() 是程序开始执行的函数。和其他语言不一样的是如果有 init() 函数则会先执行 init() 函数。
-  /\* helloworld \*/ 是注释， 可以是单行，也可以是多行，不可嵌套，也可以直接使用 // ... 来注释
- fmt.Println("Hello, World!") 是往控制台输出 "Hello, World!" 并且换行

值得注意的是：
- 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如 Datas ，那么这个标识符被外部包所使用，这被称为导出，等同于 public
- 标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的，等同于 protected
- { 不能单独放在一行，这样运行会报错

# Go 语言环境下载安装
- https://golang.google.cn/dl/
- https://golang.org/dl/

像Mac、Linux等系统，可以通过命令行安装，如：
```
brew install go
brew install golang
apt install golang-go
yum install golang
```

# Go 语言的执行
1.命令行中直接使用 “go run helloworld.go”，即可执行 helloworld.go 程序

2.命令行使用“go build helloworld.go”，可以生成名为 helloworld 的二进制可执行文件

# Go 语言数据类型
- 布尔型 ： 布尔型的值只可以是常量 true 或者 false ，默认值为 false
- 数字类型 ： 整型 和 浮点型； 位运算采用补码
- 字符串类型 ： UTF-8 编码标识 Unicode 文本
- 派生类型 ： 指针类型（Pointer）、数组类型、结构化类型(struct) 、Channel 类型、函数类型、切片类型、接口类型（interface）、Map 类型

## 数字类型
|数字类型|说明|
|--|--|
|uint|32 或 64 位，和系统有关，如果是32位系统则表示uint32，如果是64位系统则表示uint64|
|uint8|无符号 8 位整型 (0 到 255)|
|uint16|无符号 16 位整型 (0 到 65535)|
|uint32|无符号 32 位整型 (0 到 4294967295)|
|uint64|无符号 64 位整型 (0 到 18446744073709551615)|
|int|32 位或 64 位，和系统有关，如果是32位系统则表示int32，如果是64位系统则表示int64|
|int8|有符号 8 位整型 (-128 到 127)|
|int16|有符号 16 位整型 (-32768 到 32767)|
|int32|有符号 32 位整型 (-2147483648 到 2147483647)|
|int64|有符号 64 位整型 (-9223372036854775808 到 9223372036854775807)|
|float32|IEEE-754 32 位浮点型数|
|float64|IEEE-754 64 位浮点型数|
|complex64|32 位实数和虚数|
|complex128|64 位实数和虚数|
|byte|类似 uint8，通常用来保存单个ASCII字符（等同于char）|
|rune|类似 int32，通常用来保存单个Unicode字符|
|uintptr|无符号整型，用于存放一个指针，没有指定具体的bit大小但是足以容纳指针。通常在 Go 语言和 C 语言函数库或操作系统接口相交互的地方使用|

字符只能被单引号包裹，字符直接输出会输出该字符对应的码值，正是因为字符等同于码值，所以字符类型是可以运算的。如果输出对应字符，则需要使用格式化输出。

## 字符串类型
字符串有两种标识，双引号和反引号
- 双引号： var str =  " hello\nworld "  输出的时候会换行
- 反引号： var str = \` hello\nworld \` 输出的时候原样输出，不会转义

多行字符串用“+”拼接的时候，需要把“+”放在上一行的末尾，告诉 Go 后面还有，不然会报错
```
//正确写法
var str := "hello" +
	" world!"

//错误写法
var str := "hello "
	+ "world!"
```

## 基本数据类型相互转换
### 其他基本类型 转 string类型
### string类型 转 其他基本类型
