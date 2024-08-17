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

# 11、Go slice 和 array 的区别

1. **数组长度不同**
   - 数组初始化必须指定长度，并且长度就是固定的
   - 切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大
2. **函数传参不同**
   - 数组是**值类型**，将一个数组赋值给另一个数组时，传递的是一份深拷贝，函数传参操作都会复制整个数组数据，会占用额外的内存，函数内对数组元素值的**修改**，不会修改原数组内容
   - 切片是**引用类型**，将一个切片赋值给另一个切片时，传递的是一份浅拷贝，函数传参操作不会拷贝整个切片，只会复制 len 和 cap，底层共用同一个数组，不会占用额外的内存，函数内对数组元素值的**修改**，会修改原数组内容
3. **计算数组长度方式不同**
   - 数组需要遍历计算数组长度，时间复杂度为O(n)
   - 切片底层包含 len 字段，可以通过 len() 计算切片长度，时间复杂度为O(1)

# 12、Go slice 深拷贝和浅拷贝

**深拷贝**：拷贝的是**数据本身**，创造一个新对象，新创建的对象与原对象**不共享内存**，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值

**实现深拷贝的方式**：

1. copy(slice2, slice1)
2. 遍历 append 赋值

```go
func main() {
    slice1 := []int{1, 2, 3, 4, 5}
    slice2 := make([]int, 5, 5)
    
    fmt.Printf("slice1: %v, %p", slice1, slice1)
    copy(slice2, slice1)
    fmt.Printf("slice2: %v, %p", slice2, slice2)
    slice3 := make([]int, 0, 5)
    for _, v := range slice1 {
    	slice3 = append(slice3, v)
    }
    fmt.Printf("slice3: %v, %p", slice3, slice3)
}

slice1: [1 2 3 4 5], 0xc0000b0030
slice2: [1 2 3 4 5], 0xc0000b0060
slice3: [1 2 3 4 5], 0xc0000b0090
```

**浅拷贝**：拷贝的是**数据地址**，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化

**实现浅拷贝的方式**：

1. 引用类型的变量，默认赋值操作就是浅拷贝，slice2 := slice1

```go
func main() {
    slice1 := []int{1, 2, 3, 4, 5}
    
    fmt.Printf("slice1: %v, %p", slice1, slice1)
    slice2 := slice1
    fmt.Printf("slice2: %v, %p", slice2, slice2)
}

slice1: [1 2 3 4 5], 0xc00001a120
slice2: [1 2 3 4 5], 0xc00001a120
```

# 13、Go slice 扩容机制

扩容会发生在 slice **append **的时候，当 slice 的 cap 不足以容纳新元素，就会进行扩容，扩容规则如下：

- 如果新申请容量比两倍原有容量大，那么扩容后容量大小为新申请容量
- 如果原有 slice 长度**小于 1024**， 那么每次就扩容为原来的 2 倍
- 如果原 slice 长度**大于等于 1024**， 那么每次扩容就扩为原来的 1.25 倍

# 14、Go slice 为什么不是线程安全的

**线程安全的定义**：

多个线程访问同一个对象时，调用这个对象的行为都可以获得正确的结果，那么这个对象就是线程安全的。

若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。

**Go 实现线程安全常用的几种方式**：

1. 互斥锁
2. 读写锁
3. 原子操作
4. sync.once
5. sync.atomic
6. channel

slice 底层结构并没有使用加锁等方式，不支持并发读写，所以并不是线程安全的，使用多个 goroutine 对类型为 slice 的变量进行操作，每次输出的值大概率都不会一样，与预期值不一致; slice `在并发执行中不会报错，但是数据会丢失`

# 15、Go map 的底层实现原理

Go 中的 map 是一个指针，占用8个字节，指向 hmap 结构体

hmap 包含若干个结构为 bmap 的数组，每个 bmap 底层都采用**链表结构**，bmap 通常叫其 bucket

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202408171703366.png)

**hmap 结构体**

```go
// A header for a Go map.
type hmap struct {
    count     int 
    // 代表哈希表中的元素个数，调用len(map)时，返回的就是该字段值。
    flags     uint8 
    // 状态标志（是否处于正在写入的状态等）
    B         uint8  
    // buckets（桶）的对数
    // 如果B=5，则buckets数组的长度 = 2^B=32，意味着有32个桶
    noverflow uint16 
    // 溢出桶的数量
    hash0     uint32 
    // 生成hash的随机数种子
    buckets    unsafe.Pointer 
    // 指向buckets数组的指针，数组大小为2^B，如果元素个数为0，它为nil。
    oldbuckets unsafe.Pointer 
    // 如果发生扩容，oldbuckets是指向老的buckets数组的指针，老的buckets数组大小是新的buckets的1/2;非扩容状态下，它为nil。
    nevacuate  uintptr        
    // 表示扩容进度，小于此地址的buckets代表已搬迁完成。
    extra *mapextra 
    // 存储溢出桶，这个字段是为了优化GC扫描而设计的，下面详细介绍
}
```

**bmap 结构体**

`bmap` 就是我们常说的“桶”，一个桶里面会最多装 8 个 key，这些 key 之所以会落入同一个桶，是因为它们经过**哈希计算**后，**哈希结果的低8位是相同的**，关于 key 的定位我们在 map 的查询中详细说明。在桶内，又会根据 key 计算出来的 hash 值的高 8 位来决定 key 到底落入桶内的哪个位置（一个桶内最多有8个位置)。**（低8位决定落入哪个桶，高8位决定落入桶内哪个槽）**

```go
// A bucket for a Go map.
type bmap struct {
    tophash [bucketCnt]uint8        
    // len为8的数组
    // 用来快速定位key是否在这个bmap中
    // 一个桶最多8个槽位，如果key所在的tophash值在tophash中，则代表该key在这个桶中
}
```

上面 bmap 结构是静态结构，在编译过程中`runtime.bmap`会拓展（因为一开始还不知道键值的类型）成以下结构体：

```go
type bmap struct{
    tophash [8]uint8
    keys [8]keytype 
    // keytype 由编译器编译时候确定
    values [8]elemtype 
    // elemtype 由编译器编译时候确定
    overflow uintptr 
    // overflow指向下一个bmap，overflow是uintptr而不是*bmap类型，保证bmap完全不含指针，是为了减少gc，溢出桶存储到extra字段中
}
```

tophash 就是用于实现**快速定位 key 的位置**，在实现过程中会使用 key 的 hash 值的高8位作为 tophash 值，存放在 bmap 的 tophash 字段中

tophash 字段不仅存储 key 哈希值的高8位，还会存储一些状态值，用来表明**当前桶单元状态**，这些状态值都是小于 minTopHash 的

为了避免key哈希值的高8位值和这些状态值相等，产生混淆情况，所以当key哈希值高8位若小于 minTopHash 时候，自动将其值加上 minTopHash 作为该 key 的 tophash。桶单元的状态值如下：

```go
emptyRest      = 0 // 表明此桶单元为空，且更高索引的单元也是空
emptyOne       = 1 // 表明此桶单元为空
evacuatedX     = 2 // 用于表示扩容迁移到新桶前半段区间
evacuatedY     = 3 // 用于表示扩容迁移到新桶后半段区间
evacuatedEmpty = 4 // 用于表示此单元已迁移
minTopHash     = 5 // key的tophash值与桶状态值分割线值，小于此值的一定代表着桶单元的状态，大于此值的一定是key对应的tophash值

func tophash(hash uintptr) uint8 {
    top := uint8(hash >> (goarch.PtrSize*8 - 8))
    if top < minTopHash {
    	top += minTopHash
    }
    return top
}
```

**mapextra 结构体**

当 map 的 key 和 value 都不是指针类型时候，bmap 将完全不包含指针，那么 gc 时候就不用扫描 bmap。bmap 指向溢出桶的字段 overflow 是 uintptr 类型，为了防止这些 overflow桶被 gc 掉，所以需要 mapextra.overflow 将它保存起来。如果 bmap 的 overflow 是 *bmap 类型，那么 gc 扫描的是一个个拉链表，效率明显不如直接扫描一段内存(hmap.mapextra.overflow)

```go
type mapextra struct {
    overflow    *[]*bmap
    // overflow 包含的是 hmap.buckets 的 overflow 的 buckets
    oldoverflow *[]*bma
    // oldoverflow 包含扩容时 hmap.oldbuckets 的 overflow 的 bucket
    nextOverflow *bmap 
    // 指向空闲的 overflow bucket 的指针
}
```

**bmap 的 key 和 value 是各自放在一起的**，使用 key/value 这种形式可能会因为**内存对齐**导致内存空间浪费，所以 Go 采用 key 和 value 分开存储的设计，更节省内存空间

















