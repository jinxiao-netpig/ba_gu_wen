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

