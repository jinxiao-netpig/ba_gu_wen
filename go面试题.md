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

`bmap` 就是我们常说的“桶”，一个桶里面会最多装 8 个 key，这些 key 之所以会落入同一个桶，是因为它们经过**哈希计算**后，**哈希结果的低B位是相同的**，关于 key 的定位我们在 map 的查询中详细说明。在桶内，又会根据 key 计算出来的 hash 值的高 8 位来决定 key 到底落入桶内的哪个位置（一个桶内最多有8个位置)。**（低B位决定落入哪个桶，高8位决定落入桶内哪个槽）**

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

当 map 的 key 和 value 都不是指针类型时候，bmap 将完全不包含指针，那么 gc 时候就不用扫描 bmap。**bmap 指向溢出桶的字段 overflow 是 uintptr 类型**，为了防止这些 overflow桶被 gc 掉，所以需要 mapextra.overflow 将它保存起来。如果 bmap 的 overflow 是 *bmap 类型，那么 gc 扫描的是一个个拉链表，效率明显不如直接扫描一段内存(hmap.mapextra.overflow)

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

# 16、Go map遍历为什么是无序的

Go 语言的设计者们**有意为之**，旨在提示开发者们，Go 底层实现并不保证 map 遍历顺序稳定，请大家不要依赖 range 遍历结果顺序

**主要原因**：

- map 在遍历时，并不是从固定的0号 bucket 开始遍历的，每次遍历，都会从一个**随机值序号的 bucket **，再从其中**随机的 cell **开始遍历
- map 遍历时，是按序遍历 bucket，同时按需遍历bucket 中和其 overflow bucket 中的 cell。但是 map 在扩容后，会发生 key 的搬迁，这造成原来落在一个 bucket 中的 key，搬迁后，有可能会落到其他 bucket 中了，从这个角度看，遍历 map 的结果就不可能是按照原来的顺序了

如果想顺序遍历 map，需要对 map key 先排序，再按照 key 的顺序遍历 map。

# 17、Go map为什么是非线程安全的

map 默认是并发不安全的，同时对 map 进行并发读写时，程序会 panic，原因如下：

Go 官方在经过了长时间的讨论后，认为 Go map 更应适配典型使用场景（不需要从多个 goroutine 中进行安全访问），而不是为了小部分情况（并发访问），导致大部分程序付出加锁代价（性能），决定了不支持。

**实现 map 线程安全，有两种方式**：

1. 使用读写锁 `map` + `sync.RWMutex`
2. 使用 Go 提供的 `sync.Map`

# 18、Go map如何查找？

Go 语言中读取 map 有两种语法：带 comma 和 不带 comma。当要查询的 key 不在 map 里，带 comma 的用法会返回一个 bool 型变量提示 key 是否在 map 中；而不带 comma 的语句则会返回一个 value 类型的零值。如果 value 是 int 型就会返回 0，如果 value 是 string 类型，就会返回空字符串。

**查找流程**：

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202408180940055.png)

**1、写保护监测**

函数首先会检查 map 的标志位 flags。如果 flags 的写标志位此时被置 1 了，说明有其他协程在执行“写”操作，进而导致程序 panic，这也说明了 map 不是线程安全的

```go
if h.flags&hashWriting != 0 {
	throw("concurrent map read and map write")
}
```

**2、计算 hash 值**

```go
hash := t.hasher(key, uintptr(h.hash0))
```

key 经过哈希函数计算后，得到的哈希值如下（主流64位机下共 64 个 bit 位）， 不同类型的 key 会有不同的 hash 函数

```
10010111 | 000011110110110010001111001010100010010110010101010 │ 01010
```

**3、找到 hash 对应的 bucket**

bucket 定位：**哈希值的低B个bit 位**，用来定位 key 所存放的 bucket

如果当前正在扩容中，并且定位到的旧 bucket 数据还未完成迁移，则使用旧的 bucket（扩容前的 bucket）

```go
hash := t.hasher(key, uintptr(h.hash0))
// 桶的个数m-1，即 1<<B-1,B=5时，则有0~31号桶
m := bucketMask(h.B)
// 计算哈希值对应的bucket
// t.bucketsize为一个bmap的大小，通过对哈希值和桶个数取模得到桶编号，通过对桶编号和buckets起始地址进行运算，获取哈希值对应的bucket
b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
// 是否在扩容
if c := h.oldbuckets; c != nil {
    // 桶个数已经发生增长一倍，则旧bucket的桶个数为当前桶个数的一半
    if !h.sameSizeGrow() {
        // There used to be half as many buckets; mask down one more power of two.
        m >>= 1
    }
    // 计算哈希值对应的旧bucket
    oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
    // 如果旧bucket的数据没有完成迁移，则使用旧bucket查找
    if !evacuated(oldb) {
        b = oldb
    }
}
```

**4、遍历 bucket 查找**

tophash 值定位：**哈希值的高8个bit 位**，用来快速判断 key 是否已在当前 bucket 中（如果不在的话，需要去 bucket 的 overflow 中查找）

用步骤2中的 hash 值，得到高8个bit位，也就是`10010111`，转化为十进制，也就是**151**

```go
top := tophash(hash)
func tophash(hash uintptr) uint8 {
    top := uint8(hash >> (goarch.PtrSize*8 - 8))
    if top < minTopHash {
    	top += minTopHash
    }
    return top
}
```

上面函数中hash是64位的，sys.PtrSize 值是8，所以`top := uint8(hash >> (sys.PtrSize*8 - 8))`等效`top = uint8(hash >> 56)`，最后top取出来的值就是 hash 的高8位值

在 bucket 及 bucket 的 overflow 中寻找**tophash 值（HOB hash）为 151\* 的 槽位**，即为 key 所在位置，找到了空槽位或者 2 号槽位，这样整个查找过程就结束了，其中找到空槽位代表没找到。

```go
for ; b != nil; b = b.overflow(t) {
    for i := uintptr(0); i < bucketCnt; i++ {
        if b.tophash[i] != top {
            // 未被使用的槽位，插入
            if b.tophash[i] == emptyRest {
                break bucketloop
            }
            continue
        }
        // 找到tophash值对应的的key
        k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
        if t.key.equal(key, k) {
            e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
            return e
        }
    }
}
```

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202408181003940.png)

**5、返回 key 对应的指针**

如果通过上面的步骤找到了 key 对应的槽位下标 i，我们再详细分析下 key/value 值是如何获取的：

```go
// keys的偏移量
dataOffset = unsafe.Offsetof(struct{
    b bmap
    v int64
}{}.v)

// 一个bucket的元素个数
bucketCnt = 8

// key 定位公式
k :=add(unsafe.Pointer(b),dataOffset+i*uintptr(t.keysize))

// value 定位公式
v:= add(unsafe.Pointer(b),dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
```

bucket 里 keys 的起始地址就是 unsafe.Pointer(b)+dataOffset

第 i 个下标 key 的地址就要在此基础上跨过 i 个 key 的大小；

而我们又知道，**value 的地址是在所有 key 之后**，因此第 i 个下标 value 的地址还需要加上所有 key 的偏移。

# 19、Go map冲突的解决方式

**两种解决方案比较**

对于链地址法，基于数组 + 链表进行存储，链表节点可以在需要时再创建，不必像开放寻址法那样事先申请好足够内存，因此链地址法对于内存的利用率会比开方寻址法高。**链地址法对装载因子的容忍度会更高，并且适合存储大对象、大数据量的哈希表**。而且相较于开放寻址法，它更加灵活，支持更多的优化策略，比如可采用红黑树代替链表。但是**链地址法需要额外的空间来存储指针**。

对于开放寻址法，它只有数组一种数据结构就可完成存储，继承了数组的优点，对CPU缓存友好，易于序列化操作。但是它对内存的利用率不如链地址法，且发生冲突时代价更高。**当数据量明确、装载因子小，适合采用开放寻址法。**

**Go map采用链地址法**解决冲突，具体就是**插入key到map中时**，当key定位的桶**填满8个元素后**（这里的单元就是桶，不是元素），将会创建一个溢出桶，并且将溢出桶插入当前桶所在链表尾部。

```go
if inserti == nil {
    // all current buckets are full, allocate a new one.
    newb := h.newoverflow(t, b)
    // 创建一个新的溢出桶
    inserti = &newb.tophash[0]
    insertk = add(unsafe.Pointer(newb), dataOffset)
    elem = add(insertk, bucketCnt*uintptr(t.keysize))
}
```

# 20、Go map 的负载因子为什么是 6.5

**什么是负载因子**：

**负载因子（load factor），用于衡量当前哈希表中空间占用率的核心指标**，也就是每个 bucket 桶存储的平均元素个数。主要目的是**为了平衡 buckets 的存储空间大小和查找元素时的性能高低**。

在 Go 语言中，**当 map存储的元素个数大于或等于 6.5 \* 桶个数 时，就会触发扩容行为**。

把 Go 中的 map 的负载因子硬编码为 6.5

官方测试出来的结果

# 21、Go map如何扩容

**扩容时机：**

在**向 map 插入新 key** 的时候，会进行条件检测，符合下面这 2 个条件，就会触发扩容

```go
if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
    hashGrow(t, h)
    goto again // Growing the table invalidates everything, so try again
}

// 判断是否在扩容
func (h *hmap) growing() bool {
	return h.oldbuckets != nil
}
```

**扩容条件：**

**条件1：超过负载**

map 元素个数 > 6.5 * 桶个数

```go
func overLoadFactor(count int, B uint8) bool {
	return count > bucketCnt && uintptr(count) > loadFactor*bucketShift(B)
}

其中 

bucketCnt = 8，一个桶可以装的最大元素个数
loadFactor = 6.5，负载因子，平均每个桶的元素个数
bucketShift(B): 桶的个数
```

**条件2：溢出桶太多（hash 冲突太多）**

当桶总数 < 2 ^ 15 时，如果溢出桶总数 >= 桶总数，则认为溢出桶过多。

当桶总数 >= 2 ^ 15 时，直接与 2 ^ 15 比较，当溢出桶总数 >= 2 ^ 15 时，即认为溢出桶太多了。

```go
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
    // If the threshold is too low, we do extraneous work.
    // If the threshold is too high, maps that grow and shrink can hold on to lots of unused memory.
    // "too many" means (approximately) as many overflow buckets as regular buckets.
    // See incrnoverflow for more details.
    if B > 15 {
    	B = 15
    }
    // The compiler does not see here that B < 16; mask B to generate shorter shift code.
    return noverflow >= uint16(1)<<(B&15)
}
```

对于条件2，其实算是对条件1的补充。因为在负载因子比较小的情况下，有可能 map 的查找和插入效率也很低，而第 1 点识别不出来这种情况。

表面现象就是负载因子比较小比较小，即 map 里元素总数少，但是桶数量多（真实分配的桶数量多，包括大量的溢出桶）。比如不断的增删，这样会造成 overflow 的 bucket 数量增多，但负载因子又不高，达不到第 1 点的临界值，就不能触发扩容来缓解这种情况。这样会造成桶的使用率不高，值存储得比较稀疏，查找插入效率会变得非常低，因此有了第 2 扩容条件。



**扩容机制：**

**双倍扩容**：针对**条件1**，新建一个 buckets 数组，新的 buckets 大小是原来的2倍，然后旧 buckets 数据搬迁到新的 buckets。该方法我们称之为**双倍扩容**。

**等量扩容**：针对**条件2**，并不扩大容量，buckets 数量维持不变，重新做一遍类似双倍扩容的搬迁动作，把松散的键值对重新排列一次，使得同一个 bucket 中的 key 排列地更紧密，节省空间，提高 bucket 利用率，进而保证更快的存取。该方法我们称之为**等量扩容**。



**扩容函数：**

上面说的 `hashGrow()` 函数实际上并没有真正地“搬迁”，它只是分配好了新的 buckets，并将老的 buckets 挂到了 oldbuckets 字段上。真正搬迁 buckets 的动作在 `growWork()` 函数中，而调用 `growWork()` 函数的动作是在 mapassign 和 mapdelete 函数中。也就是**插入或修改、删除 key 的时候，都会尝试进行搬迁 buckets 的工作**。先检查 oldbuckets 是否搬迁完毕，具体来说就是检查 oldbuckets 是否为 nil。

```go
1  func hashGrow(t *maptype, h *hmap) {
2  // 如果达到条件 1，那么将B值加1，相当于是原来的2倍
3  // 否则对应条件 2，进行等量扩容，所以 B 不变
4    bigger := uint8(1)
5    if !overLoadFactor(h.count+1, h.B) {
6        bigger = 0
7        h.flags |= sameSizeGrow
8    }
9  // 记录老的buckets
10    oldbuckets := h.buckets
11  // 申请新的buckets空间
12    newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)
13  // 注意&^ 运算符，这块代码的逻辑是转移标志位
14    flags := h.flags &^ (iterator | oldIterator)
15    if h.flags&iterator != 0 {
16        flags |= oldIterator
17    }
18    // 提交grow (atomic wrt gc)
19    h.B += bigger
20    h.flags = flags
21    h.oldbuckets = oldbuckets
22    h.buckets = newbuckets
23  // 搬迁进度为0
24    h.nevacuate = 0
25  // overflow buckets 数为0
26    h.noverflow = 0
27
28  // 如果发现hmap是通过extra字段 来存储 overflow buckets时
29    if h.extra != nil && h.extra.overflow != nil {
30        if h.extra.oldoverflow != nil {
31            throw("oldoverflow is not nil")
32        }
33        h.extra.oldoverflow = h.extra.overflow
34        h.extra.overflow = nil
35    }
36    if nextOverflow != nil {
37        if h.extra == nil {
38            h.extra = new(mapextra)
39        }
40        h.extra.nextOverflow = nextOverflow
41    }
42}
```

由于 map 扩容需要将原有的 key/value 重新搬迁到新的内存地址，如果map存储了数以亿计的key-value，一次性搬迁将会造成比较大的延时，因此 Go map 的扩容采取了一种称为**“渐进式”**的方式，原有的 key 并不会一次性搬迁完毕，**每次最多只会搬迁 2 个 bucket**。

```go
func growWork(t *maptype, h *hmap, bucket uintptr) {
    // 为了确认搬迁的 bucket 是我们正在使用的 bucket
    // 即如果当前key映射到老的bucket1，那么就搬迁该bucket1。
    evacuate(t, h, bucket&h.oldbucketmask())
    // 如果还未完成扩容工作，则再搬迁一个bucket。
    if h.growing() {
    	evacuate(t, h, h.nevacuate)
    }
}
```

# 22、Go map 和 sync.Map 谁的性能好，为什么？

Go 语言的 `sync.Map` 支持并发读写，采取了 “空间换时间” 的机制，**冗余了两个数据结构**，分别是：read 和 dirty

```go
type Map struct {
    mu Mutex
    read atomic.Value // readOnly
    dirty map[interface{}]*entry
    misses int
}
```

**对比原始map：**

和原始 map+RWLock 的实现并发的方式相比，减少了加锁对性能的影响。它做了一些优化：可以无锁访问 read map，而且会优先操作read map，倘若只操作 read map 就可以满足要求，那就不用去操作 write map(dirty)，所以在某些特定场景中它发生锁竞争的频率会远远小于 map+RWLock 的实现方式

**优点：**

适合读多写少的场景

**缺点：**

写多的场景，会导致 read map 缓存失效，需要加锁，冲突变多，性能急剧下降

# 23、Go channel 有什么特点

channel有2种类型：无缓冲、有缓冲

channel有3种模式：写操作模式（单向通道）、读操作模式（单向通道）、读写操作模式（双向通道）

|      | 写操作模式       | 读操作模式       | 读写操作模式   |
| ---- | ---------------- | ---------------- | -------------- |
| 创建 | make(chan<- int) | make(<-chan int) | make(chan int) |

channel有3种状态：未初始化、正常、关闭

|      | 未初始化         | 关闭                               | 正常             |
| ---- | ---------------- | ---------------------------------- | ---------------- |
| 关闭 | panic            | panic                              | 正常关闭         |
| 发送 | 永远阻塞导致死锁 | panic                              | 阻塞或者成功发送 |
| 接收 | 永远阻塞导致死锁 | 缓冲区为空则为零值, 否则可以继续读 | 阻塞或者成功接收 |

**注意点**：

1. 一个 channel不能多次关闭，会导致painc
2. 如果多个 goroutine 都监听同一个 channel，那么 channel 上的数据都**可能随机被某一个 goroutine 取走进行消费**
3. 如果多个 goroutine 监听同一个 channel，如果这个 channel 被关闭，则所有 goroutine **都能收到退出信号**

# 24、Go channel 的底层实现原理

**channel 概念：**

Go中的 channel 是一个**队列**，遵循先进先出的原则，负责**协程之间的通信**（Go 语言提倡不要通过共享内存来通信，而要通过通信来实现内存共享，CSP(Communicating Sequential Process)并发模型，就是通过 goroutine 和 channel 来实现的）

**使用场景**：

- 停止信号监听
- 定时任务
- 生产方和消费方解耦
- 控制并发数

**底层数据结构**：

通过 var 声明或者 make 函数创建的 channel 变量是一个存储在函数栈帧上的指针，占用8个字节，指向堆上的 hchan 结构体

源码包中`src/runtime/chan.go`定义了hchan的数据结构：

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202408182049130.png)

**hchan 结构体：**

```go
type hchan struct {
    closed   uint32   // channel是否关闭的标志
    elemtype *_type   // channel中的元素类型

    // channel分为无缓冲和有缓冲两种。
    // 对于有缓冲的channel存储数据，使用了 ring buffer（环形缓冲区) 来缓存写入的数据，本质是循环数组
    // 为啥是循环数组？普通数组不行吗，普通数组容量固定更适合指定的空间，弹出元素时，普通数组需要全部都前移
    // 当下标超过数组容量后会回到第一个位置，所以需要有两个字段记录当前读和写的下标位置
    buf      unsafe.Pointer // 指向底层循环数组的指针（环形缓冲区）
    qcount   uint           // 循环数组中的元素数量
    dataqsiz uint           // 循环数组的长度
    elemsize uint16                 // 元素的大小
    sendx    uint           // 下一次写下标的位置
    recvx    uint           // 下一次读下标的位置

    // 尝试读取channel或向channel写入数据而被阻塞的goroutine
    recvq    waitq  // 读等待队列
    sendq    waitq  // 写等待队列

    lock mutex //互斥锁，保证读写channel时不存在并发竞争问题
}
```

**等待队列：**

双向链表，包含一个头结点和一个尾结点

每个节点是一个 sudog 结构体变量，记录哪个协程在等待，等待的是哪个 channel，等待发送/接收的数据在哪里

```go
type waitq struct {
    first *sudog
    last  *sudog
}

type sudog struct {
    g *g
    next *sudog
    prev *sudog
    elem unsafe.Pointer 
    c        *hchan 
    ...
}
```

**操作：**

**1、创建**

使用 `make(chan T, cap)` 来创建 channel，make 语法会在编译时，转换为 `makechan64` 和 `makechan`

```go
func makechan64(t *chantype, size int64) *hchan {
    if int64(int(size)) != size {
    panic(plainError("makechan: size out of range"))
    }

    return makechan(t, int(size))
}
```

创建 channel 有两种，一种是带缓冲的 channel，一种是不带缓冲的 channel

**创建时会做一些检查:**

- 元素大小不能超过 64K
- 元素的对齐大小不能超过 maxAlign 也就是 8 字节
- 计算出来的内存是否超过限制

**创建时的策略:**

- 如果是无缓冲的 channel，会直接给 hchan 分配内存
- 如果是有缓冲的 channel，并且**元素不包含指针**，那么会为 hchan 和底层数组分配一段连续的地址
- 如果是有缓冲的 channel，并且**元素包含指针**，那么会为 hchan 和底层数组分别分配地址

**2、发送**

发送操作，编译时转换为`runtime.chansend`函数

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool 
```

**阻塞式：**

调用 chansend 函数，并且block=true

```go
ch <- 10
```

**非阻塞式：**

调用 chansend 函数，并且block=false

```go
select {
    case ch <- 10:
    ...

    default
}
```

向 channel 中发送数据时大概分为两大块：**检查**和**数据发送**，数据发送流程如下：

- 如果 channel 的读等待队列存在接收者 goroutine
- 将数据**直接发送**给第一个等待的 goroutine， **唤醒接收的 goroutine**
- 如果 channel 的读等待队列不存在接收者goroutine
- 如果循环数组 buf 未满，那么将会把数据发送到循环数组 buf 的队尾
- 如果循环数组 buf 已满，这个时候就会走阻塞发送的流程，将当前 goroutine 加入写等待队列，并**挂起等待唤醒**

**3、接收**

发送操作，编译时转换为`runtime.chanrecv`函数

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) 
```

**阻塞式：**

调用chanrecv函数，并且block=true

```go
<ch

v := <ch

v, ok := <ch

// 当channel关闭时，for循环会自动退出，无需主动监测channel是否关闭，可以防止读取已经关闭的channel,造成读到数据为通道所存储的数据类型的零值
for i := range ch {
	fmt.Println(i)
}
```

**非阻塞式：**

调用chanrecv函数，并且block=false

```go
select {
    case <-ch:
    ...

    default
}
```

向 channel 中接收数据时大概分为两大块，检查和数据发送，而数据接收流程如下：

- 如果 channel 的写等待队列存在发送者goroutine
  - 如果是无缓冲 channel，**直接**从第一个发送者 goroutine 那里把数据拷贝给接收变量，**唤醒发送的 goroutine**
  - 如果是有缓冲 channel（已满），将循环数组 buf 的队首元素拷贝给接收变量，将第一个发送者 goroutine 的数据拷贝到 buf 循环数组队尾，**唤醒发送的 goroutine**
- 如果 channel 的写等待队列不存在发送者goroutine
  - 如果循环数组 buf 非空，将循环数组 buf 的队首元素拷贝给接收变量
  - 如果循环数组buf为空，这个时候就会走阻塞接收的流程，将当前 goroutine 加入读等待队列，并**挂起等待唤醒**

**4、关闭**

关闭操作，调用close函数，编译时转换为`runtime.closechan`函数

```go
close(ch)
func closechan(c *hchan) 
```

**总结**，hchan 结构体的主要组成部分有四个：

- 用来保存 goroutine 之间传递数据的**循环数组**：buf
- 用来记录此循环数组当前发送或接收数据的**下标值**：sendx 和 recvx
- 用于保存向该 chan 发送和从该 chan 接收数据**被阻塞的 goroutine 队列**： sendq 和 recvq
- 保证 channel 写入和读取数据时**线程安全**的锁：lock

# 25、Go channel有无缓冲的区别

无缓冲：一个送信人去你家送信，你不在家他不走，你一定要接下信，他才会走。

有缓冲：一个送信人去你家送信，扔到你家的信箱转身就走，除非你的信箱满了，他必须等信箱有多余空间才会走。

|          | 无缓冲             | 有缓冲                |
| -------- | ------------------ | --------------------- |
| 创建方式 | make(chan TYPE)    | make(chan TYPE, SIZE) |
| 发送阻塞 | 数据接收前发送阻塞 | 缓冲满时发送阻塞      |
| 接收阻塞 | 数据发送前接收阻塞 | 缓冲空时接收阻塞      |

# 26、Go channel 为什么是线程安全的

**为什么设计成线程安全？**

不同协程通过 channel 进行通信，本身的使用场景就是多线程，为了保证数据的一致性，必须实现线程安全

**如何实现线程安全的？**

channel 的底层实现中，hchan 结构体中采用 Mutex 锁来保证数据读写安全。在对循环数组 buf 中的数据进行入队和出队操作时，**必须先获取互斥锁**，才能操作 channel 数据

# 27、Go channel 如何控制 goroutine 并发执行顺序

**多个 goroutine 并发执行时，每一个 goroutine 抢到处理器的时间点不一致，gorouine 的执行本身不能保证顺序。**即代码中先写的 gorouine 并不能保证先执行

思路：使用 channel 进行通信通知，用 channel 去传递信息，从而控制并发执行顺序

# 28、Go channel 共享内存有什么优劣势

**“不要通过共享内存来通信，我们应该使用通信来共享内存”** 这句话想必大家已经非常熟悉了，在官方的博客，初学时的教程，甚至是在 Go 的源码中都能看到

无论是通过共享内存来通信还是通过通信来共享内存，**最终我们应用程序都是读取的内存当中的数据**，只是前者是直接读取内存的数据，而后者是通过发送消息的方式来进行同步。而通过发送消息来同步的这种方式常见的就是 Go 采用的 CSP(Communication Sequential Process) 模型以及 Erlang 采用的 Actor 模型，这两种方式都是通过通信来共享内存。

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202408201427799.png)

大部分的语言采用的都是第一种方式直接去操作内存，然后通过互斥锁，CAS 等操作来保证并发安全。Go 引入了 Channel 和 Goroutine 实现 CSP 模型将生产者和消费者进行了解耦，Channel 其实和消息队列很相似。而 Actor 模型和 CSP 模型都是通过发送消息来共享内存，但是它们之间最大的区别就是 Actor 模型当中并没有一个独立的 Channel 组件，而是 Actor 与 Actor 之间直接进行消息的发送与接收，每个 Actor 都有一个本地的“信箱”消息都会先发送到这个“信箱当中”。

**优点**

- 使用 channel 可以帮助我们解耦生产者和消费者，可以降低并发当中的耦合

**缺点**

- 容易出现死锁的情况

# 29、Go channel 发送和接收什么情况下会死锁

**死锁：**

- 单个协程永久阻塞
- 两个或两个以上的协程的执行过程中，由于竞争资源或由于彼此通信而造成的一种阻塞的现象。

**channel死锁场景：**

- 非缓存channel只写不读
- 非缓存channel读在写后面（必须先读后写，先读会把 goroutine 挂在读接收队列）
- 缓存channel写入超过缓冲区数量
- 空读
- 多个协程互相等待

# 30、Go 互斥锁的实现原理

sync包提供了两种锁类型：互斥锁sync.Mutex 和 读写互斥锁sync.RWMutex，都属于**悲观锁**。

**概念：**

Mutex 是互斥锁，当一个 goroutine 获得了锁后，其他 goroutine 不能获取锁（只能存在一个写者或读者，不能同时读和写）

**适用场景：**

多个线程同时访问临界区，为保证数据的安全，锁住一些共享资源， 以防止并发访问这些共享数据时可能导致的数据不一致问题。

获取锁的线程可以正常访问临界区，未获取到锁的线程等待锁释放后可以尝试获取锁

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191240821.png)

**底层实现结构：**

互斥锁对应的是底层结构是 sync.Mutex 结构体

```go
type Mutex struct {
    state int32
    sema uint32
}
```

state 表示锁的状态，有**锁定**、**被唤醒**、**饥饿模式**等，并且是用 state 的**二进制位**来标识的，不同模式下会有不同的处理方式

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191251082.png)

sema 表示信号量，mutex 阻塞队列的定位是通过这个变量来实现的，从而实现 goroutine 的阻塞和唤醒

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191253668.png)

```go
addr = &sema
func semroot(addr *uint32) *semaRoot {  
    return &semtable[(uintptr(unsafe.Pointer(addr))>>3)%semTabSize].root  
}
root := semroot(addr)
root.queue(addr, s, lifo)
root.dequeue(addr)

var semtable [251]struct {  
    root semaRoot  
    ...
}

type semaRoot struct {  
    lock  mutex  
    treap *sudog // root of balanced tree of unique waiters.  
    nwait uint32 // Number of waiters. Read w/o the lock.  
}

type sudog struct {
    g *g  
    next *sudog  
    prev *sudog
    elem unsafe.Pointer // 指向sema变量
    waitlink *sudog // g.waiting list or semaRoot  
    waittail *sudog // semaRoot
    ...
}
```

**操作：**

锁的实现一般会依赖于原子操作、信号量，通过 atomic 包中的一些原子操作来实现锁的锁定，通过**信号量**来实现线程的阻塞与唤醒

**加锁：**

通过原子操作 CAS 加锁，如果加锁不成功，根据不同的场景选择自旋重试加锁或者阻塞等待被唤醒后加锁

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191259683.png)

```go
func (m *Mutex) Lock() {
    // Fast path: 幸运之路，一下就获取到了锁
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
    return
    }
    // Slow path：缓慢之路，尝试自旋或阻塞获取锁
    m.lockSlow()
}
```

**解锁：**

通过原子操作 add 解锁，如果仍有 goroutine 在等待，唤醒等待的 goroutine

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191300369.png)

**注意：**

- 在 Lock() 之前使用 Unlock() 会导致 panic 异常
- 使用 Lock() 加锁后，再次 Lock() 会导致死锁（不支持重入），需Unlock()解锁后才能再加锁
- 锁定状态与 goroutine 没有关联，一个 goroutine 可以 Lock，另一个 goroutine 可以 Unlock

# 31、Go 互斥锁正常模式和饥饿模式的区别

在 Go 中一共可以分为两种抢锁的模式，一种是**正常模式**，另外一种是**饥饿模式**。

**正常模式（非公平锁）**

在刚开始的时候，是处于正常模式（Barging），也就是，当一个 G1 持有着一个锁的时候，G2 会自旋的去尝试获取这个锁

当**自旋超过4次**还没有能获取到锁的时候，这个 G2 就会被加入到获取锁的等待队列里面，并阻塞等待唤醒

> 正常模式下，所有等待锁的 goroutine 按照 FIFO(先进先出)顺序等待。唤醒的 goroutine 不会直接拥有锁，而是会和新请求锁的 goroutine 竞争锁。新请求锁的 goroutine 具有优势：它正在 CPU 上执行，而且可能有好几个，所以刚刚唤醒的 goroutine 有很大可能在锁竞争中失败，长时间获取不到锁，就会切换到饥饿模式

**饥饿模式（公平锁）**

当一个 goroutine 等待锁时间超过 1 毫秒时，它可能会遇到饥饿问题。 在版本1.9中，这种场景下Go Mutex 切换到饥饿模式（handoff），解决饥饿问题。

```go
starving = runtime_nanotime()-waitStartTime > 1e6
```

> 饥饿模式下，直接把锁交给等待队列中排在第一位的 goroutine(队头)，同时饥饿模式下，新进来的 goroutine 不会参与抢锁也不会进入自旋状态，会直接进入等待队列的尾部,这样很好的解决了老的 goroutine 一直抢不到锁的场景。

那么也不可能说永远的保持一个饥饿的状态，总归会有吃饱的时候，也就是总有那么一刻 Mutex 会回归到正常模式，那么回归正常模式必须具备的条件有以下几种：

1. G 的执行时间小于1ms
2. 等待队列已经全部清空了

当满足上述两个条件的任意一个的时候，Mutex 会切换回正常模式，而 Go 的抢锁的过程，就是在这个正常模式和饥饿模式中来回切换进行的。

```go
delta := int32(mutexLocked - 1<<mutexWaiterShift)  
if !starving || old>>mutexWaiterShift == 1 {  
	delta -= mutexStarving
}
atomic.AddInt32(&m.state, delta)
```

**总结：**

对于两种模式，**正常模式下的性能是最好的**，goroutine 可以连续多次获取锁，**饥饿模式解决了取锁公平的问题，但是性能会下降**，其实是性能和公平的 一个平衡模式。





















