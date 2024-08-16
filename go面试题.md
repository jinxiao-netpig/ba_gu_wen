# 1、Go 程序的基本结构

- 关键字（25）
- 包声明
- 注释
- 函数
- 变量和常量声明
- 类型
- 语句
  - 赋值语句
  - 条件语句
  - 循环语句
  - 跳转语句
- 运算符
  - 算术运算符
  - 逻辑运算符
  - 位运算符

# 2、Go 有哪些关键字

- package：包声明
- import：引入包
- func：定义函数和方法
- return：从函数返回
- defer：在函数退出之前执行
- var：变量声明
- const：常量声明
- interface：声明接口类型
- struct：声明结构体类型
- chan：声明 channel 类型
- map：声明 map 类型
- type：声明自定义类型
- break、case、continue、for、fallthrough、else、if、switch、goto、default：流程控制
- range：读取 slice、map、channel 数据
- go：创建 goroutine
- select：选择不同类型的通讯

# 3、Go 有哪些数据类型

- 布尔型：bool
- 数字类型：uint、int、float32、float64、byte、rune
- 字符串类型：string
- 复合类型：数组类型（array）、切片类型（slice）、字典类型（map）、管道类型（channel）、结构体类型（struct）
- 指针类型：pointer
- 接口类型：interface
- 函数类型：func
- 方法类型：method

# 4、Go 方法与函数的区别

方法有接收者（必须是结构体），函数没有接收者

# 5、Go 方法值接收者与指针接收者的区别

- 接收者是指针类型，无论调用者是对象还是对象指针，修改的都是对象本身，会影响调用者；
- 接收者是值类型，无论调用者是对象还是对象指针，修改的都是对象的副本，不影响调用者

# 6、Go 函数返回局部变量的指针是否是安全的

**安全**

因为 Go 编译器会对每个局部变量进行逃逸分析，如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上，因为它们不在栈区，所以即使释放函数，其内容也不会受影响

# 7、Go 函数参数传递到底是值传递还是引用传递

Go **只有值传递**

# 8、Go defer 关键字的实现原理

**定义**：defer 能够让我们推迟执行某些函数调用，推迟到当前函数返回前才实际执行，defer 与 panic 和 recover 结合，形成了 Go 语言风格的异常与捕获机制。

**适用场景**：defer 语句常被用于处理成对的操作，如文件句柄关闭、连接关闭、释放锁

**优点**：方便开发者使用

**缺点**：有性能损耗

**实现原理**：Go1.14 中编译器会将 defer 函数直接插入到函数的尾部，无需链表和栈上参数拷贝，性能大幅提升，把 defer 函数在当前函数内展开并直接调用，这种方式被称为 **open coded defer**

# 9、Go 内置函数 make 和 new 的区别

变量初始化，一般包含两步：

1. 变量声明
2. 变量内存分配

var 关键字就是用来声明变量的，new 和 make 函数主要是用来分配内存的

var 声明**值类型**的变量时，系统会默认为它分配内存空间

**使用场景区别**：

- make 只能用来分配及初始化类型为 slice、map、chan 的数据
- new 可以分配任意类型的数据，并且置零

make 返回类型本身，new 返回类型指针

# 10、Go slice 的底层实现原理

切片是基于数组实现的，它的底层是数组，可以理解为对底层数组的抽象

**slice 数据结构**：

```go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```

slice 占用24个字节

**array**：指向底层数组的指针，占用8个字节

**len**：切片的长度，占用8个字节

**cap**：切片的容量，cap 总是大于等于 len 的，占用8个字节

初始化 slice 调用的是 runtime.makeslice，makeslice 函数的工作主要就是计算 slice 所需内存大小，然后调用 mallocgc 进行内存的分配

**所需内存大小** = 切片中元素大小 * 切片的容量

