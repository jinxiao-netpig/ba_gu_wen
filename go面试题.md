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

# 32、Go 互斥锁允许自旋的条件

**线程没有获取到锁时常见有2种处理方式：**

1. 一种是没有获取到锁的线程就一直循环等待判断该资源是否已经释放锁，这种锁也叫做**自旋锁**，它不用将线程阻塞起来， 适用于**并发低且程序执行时间短**的场景，缺点是 **cpu 占用较高**
2. 另外一种处理方式就是把自己阻塞起来，会**释放 CPU 给其他线程**，内核会将线程置为「睡眠」状态，等到锁被释放后，内核会在合适的时机唤醒该线程，适用于高并发场景，缺点是有线程上下文切换的开销

Go语言中的 Mutex 实现了自旋与阻塞两种场景，当满足不了自旋条件时，就会进入阻塞

**允许自旋的条件：**

1. 锁已被占用，并且锁不处于饥饿模式
2. 积累的自旋次数小于最大自旋次数（active_spin=4）
3. cpu 核数大于1
4. 有空闲的 P
5. 当前 goroutine 所挂载的 P 下，本地待运行队列为空

```go
if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {  
    ...
    runtime_doSpin()   
    continue  
}


func sync_runtime_canSpin(i int) bool {  
    if i >= active_spin 
    || ncpu <= 1 
    || gomaxprocs <= int32(sched.npidle+sched.nmspinning)+1 {  
    	return false  
    }  
    if p := getg().m.p.ptr(); !runqempty(p) {  
    	return false  
    }  
    return true  
}
```

**自旋：**

```go
func sync_runtime_doSpin() {
	procyield(active_spin_cnt)
}   
```

如果可以进入自旋状态之后就会调用 `runtime_doSpin` 方法进入自旋， `doSpin` 方法会调用 `procyield(30)` 执行30次 `PAUSE` 指令，什么都不做，但是会消耗 CPU 时间

# 33、Go 读写锁的实现原理

**概念：**

读写互斥锁 RWMutex，是对 Mutex 的一个扩展，当一个 goroutine 获得了读锁后，其他 goroutine 可以获取读锁，但不能获取写锁；当一个 goroutine 获得了写锁后，其他 goroutine 既不能获取读锁也不能获取写锁（只能存在一个写者或多个读者，可以同时读）

**使用场景：**

**读**多于**写**的情况（既保证线程安全，又保证性能不太差）

**底层实现结构：**

互斥锁对应的是底层结构是 sync.RWMutex 结构体

```go
type RWMutex struct {
    w           Mutex  // 复用互斥锁
    writerSem   uint32 // 信号量，用于写等待读
    readerSem   uint32 // 信号量，用于读等待写
    readerCount int32  // 当前执行读的 goroutine 数量
    readerWait  int32  // 被阻塞的准备读的 goroutine 的数量
}
```

**操作：**

读锁的加锁与释放

```go
func (rw *RWMutex) RLock() // 加读锁
func (rw *RWMutex) RUnlock() // 释放读锁
```

**加读锁**

```go
func (rw *RWMutex) RLock() {
    // 为什么readerCount会小于0呢？往下看发现writer的Lock()会对readerCount做减法操作（原子操作）
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
    	// A writer is pending, wait for it.
    	runtime_Semacquire(&rw.readerSem)
    }
}
```

`atomic.AddInt32(&rw.readerCount, 1)` 调用这个原子方法，对当前在读的数量加1，如果返回负数，那么说明当前有其他写锁，这时候就调用 `runtime_SemacquireMutex` 休眠当前 goroutine 等待被唤醒

**释放读锁**

解锁的时候对正在读的操作减1，如果返回值小于 0 那么说明当前有在写的操作，这个时候调用 `rUnlockSlow` 进入慢速通道

```go
func (rw *RWMutex) RUnlock() {
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
    	rw.rUnlockSlow(r)
    }
}
```

被阻塞的准备读的 goroutine 的数量减1，readerWait 为 0，就表示当前没有正在准备读的 goroutine 这时候调用 `runtime_Semrelease` 唤醒写操作

```go
func (rw *RWMutex) rUnlockSlow(r int32) {
    // A writer is pending.
    if atomic.AddInt32(&rw.readerWait, -1) == 0 {
        // The last reader unblocks the writer.
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```

写锁的加锁与释放

```go
func (rw *RWMutex) Lock() // 加写锁
func (rw *RWMutex) Unlock() // 释放写锁
```

**加写锁**

```go
const rwmutexMaxReaders = 1 << 30

func (rw *RWMutex) Lock() {
    // First, resolve competition with other writers.
    rw.w.Lock()
    // Announce to readers there is a pending writer.
    r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
    // Wait for active readers.
    if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
    	runtime_Semacquire(&rw.writerSem)
    }
}
```

首先调用互斥锁的 lock，获取到互斥锁之后，如果计算之后当前仍然有其他 goroutine 持有读锁，那么就调用 `runtime_SemacquireMutex` 休眠当前的 goroutine 等待所有的读操作完成

这里 readerCount 原子性加上一个很大的负数，是防止后面的协程能拿到读锁，阻塞读

**释放写锁**

```go
func (rw *RWMutex) Unlock() {
    // Announce to readers there is no active writer.
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
    // Unblock blocked readers, if any.
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem, false)
    }
    // Allow other writers to proceed.
    rw.w.Unlock()
}
```

解锁的操作，会先调用 `atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)` 将恢复之前写入的负数，然后根据当前有多少个读操作在等待，循环唤醒

**注意：**

1. 读锁或写锁在 Lock() 之前使用 Unlock() 会导致 panic 异常
2. 使用 Lock() 加锁后，再次 Lock() 会导致死锁（不支持重入），需Unlock()解锁后才能再加锁
3. 锁定状态与 goroutine 没有关联，一个 goroutine 可以 RLock（Lock），另一个 goroutine 可以 RUnlock（Unlock）

**互斥锁和读写锁的区别：**

- 读写锁区分读者和写者，而互斥锁不区分
- 互斥锁同一时间只允许一个线程访问该对象，无论读写；读写锁同一时间内只允许一个写者，但是允许多个读者同时读对象。

# 34、Go 可重入锁如何实现

**概念：**

可重入锁又称为**递归锁**，是指在同一个线程在外层方法获取锁的时候，在进入该线程的内层方法时会自动获取锁，不会因为之前已经获取过还没释放再次加锁导致死锁

**为什么 Go 语言中没有可重入锁？**

Mutex 不是可重入的锁。Mutex 的实现中没有记录哪个 goroutine 拥有这把锁。理论上，任何 goroutine 都可以随意地 Unlock 这把锁，**所以没办法计算重入条件**，并且Mutex 重复Lock会导致死锁。

**如何实现可重入锁？**

实现一个可重入锁需要以下两点：

1. 记住持有锁的线程
2. 统计重入的次数

```go
package main

import (
    "bytes"
    "fmt"
    "runtime"
    "strconv"
    "sync"
    "sync/atomic"
)

type ReentrantLock struct {
	sync.Mutex
	recursion int32 // 这个goroutine 重入的次数
	owner     int64 // 当前持有锁的goroutine id
}

// Get returns the id of the current goroutine.
func GetGoroutineID() int64 {
    var buf [64]byte
    var s = buf[:runtime.Stack(buf[:], false)]
    s = s[len("goroutine "):]
    s = s[:bytes.IndexByte(s, )]
    gid, _ := strconv.ParseInt(string(s), 10, 64)
    return gid
}

func NewReentrantLock() sync.Locker {
    res := &ReentrantLock{
        Mutex:     sync.Mutex{},
        recursion: 0,
        owner:     0,
    }
    return res
}

// ReentrantMutex 包装一个Mutex,实现可重入
type ReentrantMutex struct {
    sync.Mutex
    owner     int64 // 当前持有锁的goroutine id
    recursion int32 // 这个goroutine 重入的次数
}

func (m *ReentrantMutex) Lock() {
    gid := GetGoroutineID()
    // 如果当前持有锁的goroutine就是这次调用的goroutine,说明是重入
    if atomic.LoadInt64(&m.owner) == gid {
        m.recursion++
        return
    }
    m.Mutex.Lock()
    // 获得锁的goroutine第一次调用，记录下它的goroutine id,调用次数加1
    atomic.StoreInt64(&m.owner, gid)
    m.recursion = 1
}

func (m *ReentrantMutex) Unlock() {
    gid := GetGoroutineID()
    // 非持有锁的goroutine尝试释放锁，错误的使用
    if atomic.LoadInt64(&m.owner) != gid {
        panic(fmt.Sprintf("wrong the owner(%d): %d!", m.owner, gid))
    }
    // 调用次数减1
    m.recursion--
    if m.recursion != 0 { // 如果这个goroutine还没有完全释放，则直接返回
    	return
    }
    // 此goroutine最后一次调用，需要释放锁
    atomic.StoreInt64(&m.owner, -1)
    m.Mutex.Unlock()
}

func main() {
    var mutex = &ReentrantMutex{}
    mutex.Lock()
    mutex.Lock()
    fmt.Println(111)
    mutex.Unlock()
    mutex.Unlock()
}
```

# 35、Go 原子操作有哪些

Go atomic包是**最轻量级的锁**（也称无锁结构），可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作，不过这个包只支持**int32/int64/uint32/uint64/uintptr**这几种数据类型的一些基础操作（**增减**、**交换**、**载入**、**存储**等）

**概念：**

**原子操作仅会由一个独立的CPU指令代表和完成**。原子操作是**无锁的**，常常直接通过CPU指令直接实现。 事实上，其它同步技术的实现常常依赖于原子操作。

**使用场景：**

当我们想要对**某个变量**并发安全的修改，除了使用官方提供的 `mutex`，还可以使用 sync/atomic 包的原子操作，它能够保证对变量的读取或修改期间不被其他的协程所影响。

atomic 包提供的原子操作能够确保任一时刻只有一个 goroutine 对变量进行操作，善用 atomic 能够避免程序中出现大量的锁操作。

**常见操作：**

- 增减 Add
- 载入 Load
- 比较并交换 CompareAndSwap
- 交换 Swap
- 存储 Store

> atomic 操作的对象是一个**地址**，你需要把可寻址的变量的地址作为参数传递给方法，而不是把变量的值传递给方法

**增减操作**

此类操作的前缀为 `Add`

```go
func AddInt32(addr *int32, delta int32) (new int32)

func AddInt64(addr *int64, delta int64) (new int64)

func AddUint32(addr *uint32, delta uint32) (new uint32)

func AddUint64(addr *uint64, delta uint64) (new uint64)

func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

需要注意的是，第一个参数必须是指针类型的值，通过指针变量可以获取被操作数在内存中的地址，从而施加特殊的 CPU 指令，确保同一时间只有一个 goroutine 能够进行操作。

**载入操作**

此类操作的前缀为 `Load`

```go
func LoadInt32(addr *int32) (val int32)

func LoadInt64(addr *int64) (val int64)

func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

func LoadUint32(addr *uint32) (val uint32)

func LoadUint64(addr *uint64) (val uint64)

func LoadUintptr(addr *uintptr) (val uintptr)

// 特殊类型： Value类型，常用于配置变更
func (v *Value) Load() (x interface{}) {}
```

载入操作能够保证原子的读变量的值，当读取的时候，任何其他 CPU 操作都无法对该变量进行读写，其实现机制受到**底层硬件的支持**。

**比较并交换**

此类操作的前缀为 `CompareAndSwap`, 该操作简称 CAS，可以用来实现**乐观锁**

```go
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)

func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)

func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)

func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)

func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)

func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
```

该操作在进行交换前首先确保变量的值未被更改，即仍然保持参数 `old` 所记录的值，满足此前提下才进行交换操作。CAS 的做法类似操作数据库时常见的乐观锁机制。

需要注意的是，当有大量的 goroutine 对变量进行读写操作时，**可能导致 CAS 操作无法成功**，这时可以利用 for 循环多次尝试。

**交换**

此类操作的前缀为 `Swap`

```go
func SwapInt32(addr *int32, new int32) (old int32)

func SwapInt64(addr *int64, new int64) (old int64)

func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

func SwapUint32(addr *uint32, new uint32) (old uint32)

func SwapUint64(addr *uint64, new uint64) (old uint64)

func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
```

相对于 CAS，明显此类操作更为暴力直接，并**不管变量的旧值是否被改变**，直接赋予新值然后返回背替换的值。

**存储**

此类操作的前缀为 `Store`

```go
func StoreInt32(addr *int32, val int32)

func StoreInt64(addr *int64, val int64)

func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

func StoreUint32(addr *uint32, val uint32)

func StoreUint64(addr *uint64, val uint64)

func StoreUintptr(addr *uintptr, val uintptr)

// 特殊类型： Value类型，常用于配置变更
func (v *Value) Store(x interface{})
```

此类操作确保了写变量的原子性，避免其他操作读到了修改变量过程中的脏数据。

# 36、Go 原子操作和锁的区别

- 原子操作由底层硬件支持，而锁是基于原子操作+信号量完成的。若实现相同的功能，前者通常会更有效率
- 原子操作是单个指令的互斥操作；互斥锁/读写锁是一种数据结构，可以完成临界区（多个指令）的互斥操作，扩大原子操作的范围
- 原子操作是无锁操作，属于乐观锁；说起锁的时候，一般属于悲观锁
- 原子操作存在于各个指令/语言层级，比如“机器指令层级的原子操作”，“汇编指令层级的原子操作”，“Go语言层级的原子操作”等
- 锁也存在于各个指令/语言层级中，比如“机器指令层级的锁”，“汇编指令层级的锁”，“Go语言层级的锁”等

# 37、Go goroutine 的底层实现原理

**概念：**

Goroutine 可以理解为一种 Go 语言的协程（轻量级线程），是 Go 支持高并发的基础，属于**用户态的线程**，由Go runtime 管理而不是操作系统。

**底层数据结构**

```go
type g struct {
    goid    int64 // 唯一的goroutine的ID
    sched gobuf // goroutine切换时，用于保存g的上下文
    stack stack // 栈
    gopc        // pc of go statement that created this goroutine
    startpc    uintptr // pc of goroutine function
    ...
}

type gobuf struct {
    sp   uintptr // 栈指针位置
    pc   uintptr // 运行到的程序位置
    g    guintptr // 指向 goroutine
    ret  uintptr  // 保存系统调用的返回值
    ...
}

type stack struct {
    lo uintptr // 栈的下界内存地址
    hi uintptr // 栈的上界内存地址
}
```

最终有一个 runtime.g 对象放入调度队列

**状态流转**

| 状态                | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| 空闲中_Gidle        | G刚刚新建, 仍未初始化                                        |
| 待运行_Grunnable    | 就绪状态，G在运行队列中, 等待M取出并运行                     |
| 运行中_Grunning     | M正在运行这个G, 这时候M会拥有一个P                           |
| 系统调用中_Gsyscall | M正在运行这个G发起的系统调用, 这时候M并不拥有P               |
| 等待中_Gwaiting     | G在等待某些条件完成, 这时候G不在运行也不在运行队列中(可能在channel的等待队列中) |
| 已中止_Gdead        | G未被使用, 可能已执行完毕                                    |
| 栈复制中_Gcopystack | G正在获取一个新的栈空间并把原来的内容复制过去(**用于防止GC扫描**) |

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191749497.jpeg)

**创建**

通过`go`关键字调用底层函数`runtime.newproc()`创建一个`goroutine`

当调用该函数之后，goroutine会被设置成`runnable`状态

创建好的这个 goroutine 会新建一个自己的栈空间，同时在 G(也就是这个 goroutine) 的 sched 中维护栈地址与程序计数器这些信息

每个 G 在被创建之后，都会被优先放入到本地队列中，如果本地队列已经满了，就会被放入到全局队列中

**运行**

G 本身只是一个数据结构，真正让 G 运行起来的是**调度器**。Go 实现了一个**用户态的调度器**（GMP模型），这个调度器充分利用现代计算机的多核特性，同时让多个 goroutine(G) 运行，同时 goroutine 设计的很轻量级，调度和上下文切换的代价都比较小。

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191805026.png)

**调度时机**

- 新起一个协程和协程执行完毕
- 会阻塞的系统调用，比如文件io、网络io
- channel、mutex 等阻塞操作
- time.sleep
- 垃圾回收之后
- 主动调用 runtime.Goshed()
- 运行过久或系统调用过久等等

每个 M 开始执行 P 的本地队列中的 G 时，这个 G 会被设置成`running`状态

如果某个 M 把本地队列中的 G 都执行完成之后，然后就会去全局队列中拿 G，这里需要注意，每次去全局队列拿 G 的时候，都需要**上锁**，避免同样的任务被多次拿。

如果全局队列都被拿完了，而当前 M 也没有更多的 G 可以执行的时候，它就会去其他 P 的本地队列中拿任务，这个机制被称之为 **work stealing 机制**，每次会拿走**一半**的任务，向下取整，比如另一个 P 中有 3 个任务，拿一半就是一个任务。

当全局队列为空，M 也没办法从其他的 P 中拿任务的时候，就会让自身进入**自旋状态**，等待有新的 G 进来。最多只会有 GOMAXPROCS 个 M 在自旋状态，过多 M 的自旋会浪费 CPU 资源。

**阻塞**

channel 的读写操作、等待锁、等待网络数据、系统调用等都有可能发生阻塞，会调用底层函数`runtime.gopark()`，会让出CPU时间片，让调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行。

当调用该函数之后，goroutine 会被设置成`waiting`状态

**唤醒**

处于 waiting 状态的 goroutine，在调用`runtime.goready()`函数之后会被唤醒，唤醒的 goroutine 会被重新放到 M 对应的上下文 P 对应的runqueue 中，等待被调度。

当调用该函数之后，goroutine 会被设置成`runnable`状态

**退出**

当 goroutine 执行完成后，会调用底层函数`runtime.Goexit()`

当调用该函数之后，goroutine 会被设置成`dead`状态

# 38、Go goroutine 和线程的区别

|            | goroutine                                                    | 线程                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 内存占用   | 创建一个 goroutine 的栈内存消耗为 **2 KB**，实际运行过程中，如果栈空间不够用，会自动进行扩容 | 创建一个 线程 的栈内存消耗为 1 MB                            |
| 创建和销毀 | goroutine 因为是由 Go runtime 负责管理的，创建和销毁的消耗非常小，是**用户级**。 | 线程的创建和销毀都会有巨大的消耗，因为要和操作系统打交道，是**内核级**的，通常解决的办法就是线程池 |
| 切换       | goroutines 切换只需保存三个寄存器：PC、SP、BP goroutine 的切换约为 200 ns，相当于 2400-3600 条指令。 | 当线程切换时，需要保存各种寄存器，以便恢复现场。 线程切换会消耗 1000-1500 ns，相当于 12000-18000 条指令。 |

# 39、Go goroutine 泄露的场景

**泄露原因：**

- Goroutine 内进行 channel/mutex 等读写操作被一直阻塞
- Goroutine 内的业务逻辑进入死循环，资源一直无法释放
- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待

**泄露场景：**

1. 如果输出的 goroutines 数量是在不断增加的，就说明存在泄漏
2. **nil channel：** channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞
3. **发送不接收：**channel 发送数量 超过 channel 接收数量，就会造成阻塞
4. **接收不发送：**channel 接收数量 超过 channel 发送数量，也会造成阻塞
5. **http request body未关闭：**`resp.Body.Close()` 未被调用时，goroutine不会退出
6. **互斥锁忘记解锁：**第一个协程获取 `sync.Mutex` 加锁了，但是他可能在处理业务逻辑，又或是忘记 `Unlock` 了。因此导致后面的协程想加锁，却因锁未释放被阻塞了
7. **sync.WaitGroup使用不当：**由于 `wg.Add` 的数量与 `wg.Done` 数量并不匹配，因此在调用 `wg.Wait` 方法后一直阻塞等待

**排查方法：**

1. 单个函数：调用 `runtime.NumGoroutine` 方法来打印 执行代码前后 Goroutine 的运行数量，进行前后比较，就能知道有没有泄露了。
2. 生产/测试环境：使用`PProf`实时监测 Goroutine 的数量

# 40、Go 如何查看正在执行的 goroutine 数量

### 程序中引入pprof pakage

```go
import _ "net/http/pprof"
```

程序中开启HTTP监听服务：

```go
package main

import (
"net/http"
_ "net/http/pprof"
)

func main() {

    for i := 0; i < 100; i++ {
        go func() {
            select {}
        }()
    }

    go func() {
    	http.ListenAndServe("localhost:6060", nil)
    }()

    select {}
}
```

### 分析goroutine文件

在命令行下执行：

```shell
go tool pprof -http=:1248 http://127.0.0.1:6060/debug/pprof/goroutine
```

会自动打开浏览器页面如下图所示

![none](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202409191830256.png)

在图中可以清晰的看到 goroutine 的数量以及调用关系，可以看到有103个 goroutine

# 41、Go 如何控制并发的 goroutine 数量

**为什么要控制 goroutine 并发的数量？**

在开发过程中，如果不对 goroutine 加以控制而进行滥用的话，可能会导致服务整体崩溃。比如耗尽系统资源导致程序崩溃，或者CPU使用率过高导致系统忙不过来。

**用什么方法控制 goroutine 并发的数量？**

- **有缓冲 channel：**利用缓冲满时发送阻塞的特性
- **无缓冲 channel：**任务发送和执行分离，指定消费者并发协程数

# 42、Go 线程实现模型

Go 实现的是两级线程模型（M:N），准确地说是 GMP 模型，是对两级线程模型的改进实现，使它能够更加灵活地进行线程之间的调度。

## 背景

|                 | 含义                                                         | 缺点                                                    |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| 单进程时代      | 每个程序就是一个进程，直到一个程序运行完，才能进行下一个进程 | 1. 无法并发，只能串行  2. 进程阻塞所带来的 CPU 时间浪费 |
| 多进程/线程时代 | 一个线程阻塞， cpu 可以立刻切换到其他线程中去执行            | 1. 进程/线程占用内存高 2. 进程/线程上下文切换成本高     |
| 协程时代        | 协程（用户态线程）绑定线程（内核态线程），cpu调度线程执行    | 1. 实现起来较复杂，协程和线程的绑定依赖调度器算法       |

线程 -> CPU 由 **操作系统** 调度，协程 -> 线程 由 **Go调度器** 来调度，协程与线程的映射关系有三种线程模型



## 三种线程模型

线程实现模型主要分为：`内核级线程模型`、`用户级线程模型`、`两级线程模型`，他们的区别在于用户线程与内核线程之间的对应关系。

### 内核级线程模型（1：1）

1个用户线程对应1个内核线程，这种最容易实现，协程的调度都有 CPU 完成了

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410060853480.jpeg)

**优点：**

- 实现起来最简单
- 能够利用多核
- 如果进程中的一个线程被阻塞，不会阻塞其他线程，是能够切换同一进程内的其他线程继续执行

**缺点：**

- 上下文切换成本高，创建、删除和切换都由 CPU 完成

### 用户级线程模型（N：1）

1个进程中的所有线程对应1个内核线程

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410060856303.jpeg)

**优点：**

- 上下文切换成本低，在用户态即可完成协程切换

**缺点：**

- 无法利用多核
- 一旦协程阻塞，造成线程阻塞，本线程的其它协程无法执行

### 两级线程模型（M：N）

M个线程对应N个内核线程

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410060859914.jpeg)

**优点：**

- 能够利用多核
- 上下文切换成本低
- 如果进程中的一个线程被阻塞，不会阻塞其他线程，是能够切换同一进程内的其他线程继续执行

**缺点：**

- 实现起来最复杂

# 43、Go GMP 和 GM 模型

什么才是一个好的调度器？

能在适当的时机将合适的协程分配到合适的位置，保证公平和效率。

Go采用了GMP模型（对两级线程模型的改进实现），使它能够更加灵活地进行线程之间的调度。

## GMP 模型

GMP 是 Go 运行时调度层面的实现，包含4个重要结构，分别是 G、M、P、Sched

 ![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410060904313.png)

**G（Goroutine）**：代表 Go 协程Goroutine，存储了 Goroutine 的执行栈信息、Goroutine 状态以及 Goroutine 的任务函数等。**G的数量无限制，理论上只受内存的影响**，创建一个 G 的初始栈大小为2-4K，配置一般的机器也能简简单单开启数十万个 Goroutine ，而且 **Go 语言在 G 退出的时候还会把 G 清理之后放到 P 本地或者全局的闲置列表 gFree 中以便复用**。

**M（Machine）**： Go 对操作系统线程（OS thread）的封装，可以看作操作系统内核线程，想要在 CPU 上执行代码必须有线程，通过系统调用 clone 创建。M在绑定有效的 P 后，进入一个调度循环，而调度循环的机制大致是从 P 的本地运行队列以及全局队列中获取 G，切换到 G 的执行栈上并执行 G 的函数，调用 goexit 做清理工作并回到 M，如此反复。M 并不保留 G 状态，这是 G 可以跨 M 调度的基础。**M的数量有限制，默认数量限制是 10000**，可以通过 debug.SetMaxThreads() 方法进行设置，如果有M空闲，那么就会回收或者睡眠。

**P（Processor）**：虚拟处理器，M 执行 G 所需要的资源和上下文，**只有将 P 和 M 绑定，才能让 P 的 runq 中的 G 真正运行起来**。P 的数量决定了系统内最大可并行的 G 的数量，P 的数量受本机的 CPU 核数影响，可通过环境变量 $GOMAXPROCS 或在 runtime.GOMAXPROCS() 来设置，默认为 CPU 核心数。

**Sched：调度器结构**，它维护有存储 M 和 G 的全局队列，以及调度器的一些状态信息

|          | G                      | M                                                            | P                                                            |
| -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数量限制 | 无限制，受机器内存影响 | 有限制，默认最多10000                                        | 有限制，最多 GOMAXPROCS 个                                   |
| 创建时机 | go func                | 当没有足够的 M 来关联 P 并运行其中的可运行的 G 时会请求创建新的M | 在确定了 P 的最大数量 n 后，运行时系统会根据这个数量创建个 P |

### 核心数据结构

```go
//src/runtime/runtime2.go
type g struct {
    goid    int64 // 唯一的goroutine的ID
    sched gobuf // goroutine切换时，用于保存g的上下文
    stack stack // 栈
    gopc        // pc of go statement that created this goroutine
    startpc    uintptr // pc of goroutine function
    ...
}

type p struct {
    lock mutex
    id          int32
    status      uint32 // one of pidle/prunning/...

    // Queue of runnable goroutines. Accessed without lock.
    runqhead uint32 // 本地队列队头
    runqtail uint32 // 本地队列队尾
    runq     [256]guintptr // 本地队列，大小256的数组，数组往往会被都读入到缓存中，对缓存友好，效率较高
    runnext guintptr // 下一个优先执行的goroutine（一定是最后生产出来的)，为了实现局部性原理，runnext中的G永远会被最先调度执行
    ... 
}

type m struct {
    g0            *g     
    // 每个M都有一个自己的G0，不指向任何可执行的函数，在调度或系统调用时，M会切换到G0，使用G0的栈空间来调度
    curg          *g    
    // 当前正在执行的G
    ... 
}

type schedt struct {
    ...
    runq     gQueue // 全局队列，链表（长度无限制）
    runqsize int32  // 全局队列长度
    ...
}
```

GMP 模型的实现算是 Go 调度器的一大进步，但调度器仍然有一个令人头疼的问题，那就是不支持抢占式调度，这导致一旦某个 G 中出现死循环的代码逻辑，那么 G 将永久占用分配给它的 P 和 M，而位于同一个 P 中的其他 G 将得不到调度，出现“饿死”的情况。

当只有一个 P（GOMAXPROCS=1）时，整个 Go 程序中的其他 G 都将“饿死”。于是在 Go 1.2 版本中实现了基于协作的“抢占式”调度，在Go 1.14 版本中实现了基于信号的“抢占式”调度。

## GM 模型

Go早期是GM模型，没有P组件

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410060915046.png)

**GM调度存在的问题：**

1. **全局队列的锁竞争**，当 M 从全局队列中添加或者获取 G 的时候，都需要获取队列锁，导致激烈的锁竞争
2. **M 转移 G 增加额外开销**，当 M1 在执行 G1 的时候， M1 创建了 G2，为了继续执行 G1，需要把 G2 保存到全局队列中，无法保证G2是被M1处理。因为 M1 原本就保存了 G2 的信息，所以 G2 **最好**是在 M1 上执行，这样的话也不需要转移G到全局队列和线程上下文切换
3. **线程使用效率不能最大化**，没有 **work-stealing** 和 **hand-off** 机制

# 44、Go 调度原理

goroutine 调度的本质就是**将 Goroutine (G）按照一定算法放到 CPU 上去执行**。

CPU感知不到 Goroutine，只知道内核线程，所以需要**Go 调度器**将协程调度到内核线程上面去，然后**操作系统调度器**将内核线程放到 CPU 上去执行。

M是对内核级线程的封装，**所以Go调度器的工作就是将G分配到M**。

## 1、设计思想

- 线程复用（**work stealing 机制**和**hand off 机制**）
- 利用并行（利用多核CPU）
- 抢占调度（解决公平性问题）

## 2、调度对象

Go 调度器

> Go 调度器是属于 Go runtime 中的一部分，Go runtime 负责实现 Go 的**并发调度**、**垃圾回收**、**内存堆栈管理**等关键功能

## 3、被调度对象

**G 的来源**

- P的 runnext（只有1个G，局部性原理，永远会被最先调度执行）
- P的本地队列（数组，最多256个G）
- 全局G队列（链表，无限制）
- 网络轮询器*network poller*（存放网络调用被阻塞的G）

**P 的来源**

- 全局 P 队列（数组，GOMAXPROCS个P）

**M 的来源**

- 休眠线程队列（未绑定P，长时间休眠会等待GC回收销毁）
- 运行线程（绑定P，指向P中的G）
- 自旋线程（绑定P，指向M的G0）

其中运行线程数 + 自旋线程数 <= P的数量（GOMAXPROCS），M个数 >= P个数

## 4、调度流程

协程的调度采用了**生产者-消费者**模型，实现了用户任务与调度器的解耦

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410061635437.png)

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410061636827.jpeg)

生产端我们开启的每个协程都是一个计算任务，这些任务会被提交给 go 的 runtime。如果计算任务非常多，有成千上万个，那么这些任务是不可能同时被立刻执行的，所以这个计算任务一定会被先暂存起来，一般的做法是放到内存的队列中等待被执行。

G的生命周期：G 从创建、保存、被获取、调度和执行、阻塞、销毁，步骤如下：

1. **创建 G**，关键字 `go func()` 创建 G
2. **保存 G**，创建的 G 优先保存到本地队列 P，如果 P 满了，则会平衡部分 P 中的 G 到全局队列中
3. **唤醒或者新建M**执行任务，进入调度循环（步骤4,5,6)
4. **M 获取 G**，M首先从P的本地队列获取 G，如果 P为空，则从全局队列获取 G，如果全局队列也为空，则从另一个本地队列偷取一半数量的 G（负载均衡），这种从其它P偷的方式称之为 **work stealing**
5. **M 调度和执行 G**，M调用 `G.func()` 函数执行 G
   - 如果 M在执行 G 的过程发生**系统调用阻塞**（同步），会阻塞G和M（操作系统限制），此时P会和当前M解绑，并寻找新的M，如果没有空闲的M就会新建一个M ，接管正在阻塞G所属的P，**接着继续执行 P 中其余的G**，这种阻塞后释放P的方式称之为**hand off**。当**系统调用结束**后，这个G会尝试获取一个空闲的P执行，优先获取之前绑定的P，并放入到这个P的本地队列，如果获取不到P，那么这个线程M变成休眠状态，加入到空闲线程中，然后这个G会被放入到全局队列中。
   - 如果M在执行G的过程发生网络IO等操作阻塞时（异步），阻塞G，**不会阻塞M**。M会寻找P中其它可执行的G继续执行，**G会被网络轮询器 network poller 接手**，当阻塞的G恢复后，G1从network poller 被移回到P的 LRQ 中，重新进入可执行状态。异步情况下，通过调度，Go scheduler 成功地将 I/O 的任务转变成了 CPU 任务，或者说将内核级别的线程切换转变成了用户级别的 goroutine 切换，大大提高了效率。
6. **M执行完G后清理现场**，重新进入调度循环（将M上运⾏的goroutine切换为G0，G0负责调度时协程的切换）

**其中步骤2中保存 G的详细流程如下：**

- 执行 go func 的时候，主线程 M0 会调用 newproc()生成一个 G 结构体，这里会**先选定当前 M0 上的 P 结构**
- 每个协程 G 都会被尝试先放到 P 中的 runnext，若 runnext 为空则放到 runnext 中，生产结束
- 若 runnext 满，则将原来 runnext 中的 G 踢到本地队列中，将当前 G 放到 runnext 中，生产结束
- 若本地队列也满了，则将本地队列中的 G 拿出一半，放到全局队列中，生产结束。

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/33653a35dca9472fd1e487d3a94557c6.writebug)

## 5、调度时机

**在以下情形下，会切换正在执行的 goroutine：**

- 抢占式调度
- sysmon 检测到协程运行过久（比如sleep，死循环）
- 切换到g0，进入调度循环
- 主动调度
- 新起一个协程和协程执行完毕
- 触发调度循环
- 主动调用runtime.Gosched()
- 切换到g0，进入调度循环
- 垃圾回收之后
- stw之后，会重新选择g开始执行
- 被动调度
- 系统调用（比如文件IO）阻塞（同步）
- 阻塞G和M，P与M分离，将P交给其它M绑定，其它M执行P的剩余G
- 网络IO调用阻塞（异步）
- 阻塞G，G移动到NetPoller，M执行P的剩余G
- atomic/mutex/channel等阻塞（异步）
- 阻塞G，G移动到channel的等待队列中，M执行P的剩余G

## 6、调度策略

**使用什么策略来挑选下一个goroutine执行？**

由于 P 中的 G 分布在 runnext、本地队列、全局队列、网络轮询器中，则需要挨个判断是否有可执行的 G，大体逻辑如下：

- 每执行61次调度循环，从全局队列获取G，若有则直接返回
- 从P 上的 runnext 看一下是否有 G，若有则直接返回
- 从P 上的 本地队列 看一下是否有 G，若有则直接返回
- 上面都没查找到时，则去全局队列、网络轮询器查找或者从其他 P 中窃取，**一直阻塞**直到获取到一个可用的 G 为止

```go
func schedule() {
    _g_ := getg()
    var gp *g
    var inheritTime bool
    ...
    if gp == nil {
        // 每执行61次调度循环会看一下全局队列。为了保证公平，避免全局队列一直无法得到执行的情况，当全局运行队列中有待执行的G时，通过schedtick保证有一定几率会从全局的运行队列中查找对应的Goroutine；
        if _g_.m.p.ptr().schedtick%61 == 0 && sched.runqsize > 0 {
        lock(&sched.lock)
        gp = globrunqget(_g_.m.p.ptr(), 1)
        unlock(&sched.lock)
        }
    }
    if gp == nil {
        // 先尝试从P的runnext和本地队列查找G
        gp, inheritTime = runqget(_g_.m.p.ptr())
    }
    if gp == nil {
        // 仍找不到，去全局队列中查找。还找不到，要去网络轮询器中查找是否有G等待运行；仍找不到，则尝试从其他P中窃取G来执行。
        gp, inheritTime = findrunnable() // blocks until work is available
        // 这个函数是阻塞的，执行到这里一定会获取到一个可执行的G
    }
    ...
    // 调用execute，继续调度循环
    execute(gp, inheritTime)
}
```

从全局队列查找时，如果要所有 P 平分全局队列中的 G，每个 P 要分得多少个，这里假设会分得 n 个。然后把这 n 个 G，转移到当前 G 所在 P 的本地队列中去。**但是最多不能超过 P 本地队列长度的一半（即 128）**。这样做的目的是，如果下次调度循环到来的时候，就不必去加锁到全局队列中再获取一次 G 了，性能得到了很好的保障。源码如下：

```go
func globrunqget(_p_ *p, max int32) *g {
    ...
    // gomaxprocs = p的数量
    // sched.runqsize是全局队列长度
    // 这里n = 全局队列的G平分到每个P本地队列上的数量 + 1
    n := sched.runqsize/gomaxprocs + 1
    if n > sched.runqsize {
    	n = sched.runqsize
    }
    if max > 0 && n > max {
    	n = max
    }
    // 平分后的数量n不能超过本地队列长度的一半，也就是128
    if n > int32(len(_p_.runq))/2 {
    	n = int32(len(_p_.runq)) / 2
    }

    // 执行将G从全局队列中取n个分到当前P本地队列的操作
    sched.runqsize -= n

    gp := sched.runq.pop()
    n--
    for ; n > 0; n-- {
        gp1 := sched.runq.pop()
        runqput(_p_, gp1, false)
    }
    return gp
}
```

从其它P查找时，会偷一半的G过来放到当前P的本地队列

# 45、Go work stealing 机制

**概念：**

当线程M⽆可运⾏的G时，尝试从其他M绑定的P偷取G，减少空转，提高了线程利用率（避免闲着不干活）。

当从本线程绑定 P 本地 队列、全局G队列、netpoller都找不到可执行的 g，会从别的 P 里窃取G**并放到当前P上面**。

从 *netpoller* 中拿到的G是 _Gwaiting 状态（ 存放的是因为网络IO被阻塞的G），从其它地方拿到的G是  Grunnable状态

从全局队列取的G数量：N = min(len(GRQ)/GOMAXPROCS + 1, len(GRQ/2)) （根据 GOMAXPROCS 负载均衡）

从其它P本地队列**窃取**的G数量：N = len(LRQ)/2（平分）

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410061705058.png)

## 1、窃取流程

源码见runtime/proc.go stealWork函数，窃取流程如下，如果经过多次努力一直找不到需要运行的goroutine则调用stopm进入睡眠状态，等待被其它工作线程唤醒。

1. 选择要窃取的P
2. 从P中偷走一半G

**选择要窃取的 P**

窃取的实质就是遍历 allp 中的所有 p，查看其运行队列是否有 goroutine，如果有，则取其一半到当前工作线程的运行队列

为了保证公平性，遍历 allp 时**并不是固定的从allp[0]即第一个p开始**，而是从随机位置上的p开始，而且遍历的顺序也随机化了，并不是现在访问了第i个p下一次就访问第i+1个p，而是使用了一种伪随机的方式遍历 allp 中的每个p，防止每次遍历时使用同样的顺序访问 allp 中的元素

```go
offset := uint32(random()) % nprocs
coprime := 随机选取一个小于nprocs且与nprocs互质的数
const stealTries = 4 // 最多重试4次
for i := 0; i < stealTries; i++ {
    for i := 0; i < nprocs; i++ {
        p := allp[offset]
        从p的运行队列偷取goroutine
        if 偷取成功 {
            break
        }
        offset += coprime
        offset = offset % nprocs
    }
}
```

可以看到只要随机数不一样，偷取p的顺序也不一样，但可以保证经过 nprocs 次循环，每个p都会被访问到。

**从 P 中偷走一半 G**

源码见runtime/proc.go runqsteal函数：

挑选出盗取的对象p之后，则调用 runqsteal 盗取 p 的运行队列中的 goroutine，runqsteal 函数再调用 runqgrap 从 p 的**本地队列尾部**批量偷走一半的 G

为啥是偷一半的g，可以理解为负载均衡

```go
func runqgrab(_p_ *p, batch *[256]guintptr, batchHead uint32, stealRunNextG bool) uint32 {
    for {
        h := atomic.LoadAcq(&_p_.runqhead) // load-acquire, synchronize with other consumers
        t := atomic.LoadAcq(&_p_.runqtail) // load-acquire, synchronize with the producer
        n := t - h        //计算队列中有多少个goroutine
        n = n - n/2     //取队列中goroutine个数的一半
        if n == 0 {
            ......
            return ......
        }
        return n
    }
}
```

# 46、Go hand off 机制

## 1、概念

也称为**P分离机制**，当本线程 M 因为 G 进行的系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的 M 执行，也提高了线程利用率（避免站着茅坑不拉shi）。

## 2、分离流程

当前线程M阻塞时，释放P，给其它空闲的M处理

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071452345.png)

# 47、Go 抢占式调度

在1.2版本之前，Go的调度器仍然不支持抢占式调度，程序只能依靠Goroutine主动让出CPU资源才能触发调度，这会引发一些问题，比如：

- 某些 Goroutine 可以长时间占用线程，造成其它 Goroutine 的饥饿
- 垃圾回收器是需要**stop the world（STW）**的，如果垃圾回收器想要运行了，那么它必须先通知其它的goroutine停下来，这会造成较长时间的等待时间

为解决这个问题：

- Go 1.2 中实现了基于协作的“抢占式”调度
- Go 1.14 中实现了基于信号的“抢占式”调度

## 1、基于协作的抢占式调度

协作式：大家都按事先定义好的规则来，比如：一个 goroutine 执行完后，退出，让出 p，然后下一个 goroutine 被调度到 p 上运行。这样做的缺点就在于是否让出 p 的决定权在 groutine 自身。一旦某个 g 不主动让出 p 或执行时间较长，那么后面的 goroutine 只能等着，没有方法让前者让出 p，导致延迟甚至饿死。

非协作式: 就是由 runtime 来决定一个 goroutine 运行多长时间，如果你不主动让出，对不起，我有手段可以抢占你，把你踢出去，让后面的 goroutine 进来运行。

**基于协作的抢占式调度流程：**

- 编译器会在调用函数前插入 runtime.morestack，让运行时有机会在这段代码中检查是否需要执行抢占调度
- Go语言运行时会在垃圾回收暂停程序、系统监控发现 Goroutine 运行超过 10ms，那么会在这个协程设置一个抢占标记
- 当发生函数调用时，可能会执行编译器插入的  runtime.morestack，它调用的  runtime.newstack 会检查抢占标记，如果有抢占标记就会触发抢占让出 cpu，切到调度主协程里

这种解决方案只能说局部解决了“饿死”问题，只在有函数调用的地方才能插入“抢占”代码（埋点），对于没有函数调用而是纯算法循环计算的 G，Go 调度器依然无法抢占。

为了解决这些问题，**Go 在 1.14 版本中增加了对非协作的抢占式调度的支持**，这种**抢占式调度是基于系统信号的，也就是通过向线程发送信号的方式来抢占正在运行的 Goroutine**。

## 2、基于信号的抢占式调度

真正的抢占式调度是基于信号完成的，所以也称为“**异步抢占**”。不管协程有没有意愿主动让出 cpu 运行权，**只要某个协程执行时间过长，就会发送信号强行夺取 cpu 运行权**。

- M 注册一个 SIGURG 信号的处理函数：sighandler
- sysmon 启动后会间隔性的进行监控，最长间隔10ms，最短间隔20us。如果发现某协程独占 P 超过10ms，会给 M 发送抢占信号
- M 收到信号后，内核执行 sighandler 函数把当前协程的状态从 _Grunning正在执行改成  _Grunnable可执行，**把抢占的协程放到全局队列里**，M继续寻找其他 goroutine 来运行
- 被抢占的 G 再次调度过来执行时，会继续原来的执行流

抢占分为`_Prunning`和`_Psyscall`，`_Psyscall`抢占通常是由于阻塞性系统调用引起的，比如磁盘io、cgo。`_Prunning`抢占通常是由于一些类似死循环的计算逻辑引起的。

# 48、Go 如何查看运行时调度信息

有 2 种方式可以查看一个程序的调度GMP信息，分别是go tool trace和GODEBUG

trace.go

```
package main

import (
"fmt"
"os"
"runtime/trace"
"time"
)

func main() {

//创建trace文件
f, err := os.Create("trace.out")
if err != nil {
panic(err)
}

defer f.Close()

//启动trace goroutine
err = trace.Start(f)
if err != nil {
panic(err)
}
defer trace.Stop()

//main
for i := 0; i < 5; i++ {
time.Sleep(time.Second)
fmt.Println("Hello World")
}
}
```

## go tool trace

启动可视化界面：

```
go run trace.go
go tool trace trace.out
2022/04/22 10:44:11 Parsing trace...
2022/04/22 10:44:11 Splitting trace...
2022/04/22 10:44:11 Opening browser. Trace viewer is listening on http://127.0.0.1:35488
```

打开 `http://127.0.0.1:35488` 查看可视化界面：

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071504325.png)

点击 `view trace` 能够看见可视化的调度流程：

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220422213604926.png)

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220422210738598.png)

一共有2个G在程序中，一个是特殊的G0，是每个M必须有的一个初始化的G，另外一个是G1 main goroutine (执行 main 函数的协程)，在一段时间内处于可运行和运行的状态。

**2. 点击 Threads 那一行可视化的数据条，我们会看到M详细的信息**

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071504270.png)

一共有2个 M 在程序中，一个是特殊的 M0，用于初始化使用，另外一个是用于执行G1的M1

**3. 点击 Proc 那一行可视化的数据条，我们会看到P上正在运行goroutine详细的信息**

一共有3个 P 在程序中，分别是P0、P1、P2

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220422212719433.png)

点击具体的 Goroutine 行为后可以看到其相关联的详细信息:

```
Start：开始时间
Wall Duration：持续时间
Self Time：执行时间
Start Stack Trace：开始时的堆栈信息
End Stack Trace：结束时的堆栈信息
Incoming flow：输入流
Outgoing flow：输出流
Preceding events：之前的事件
Following events：之后的事件
All connected：所有连接的事件
```

## GODEBUG

GODEBUG 变量可以控制运行时内的调试变量。查看调度器信息，将会使用如下两个参数：

- schedtrace：设置 `schedtrace=X` 参数可以使运行时在每 X 毫秒发出一行调度器的摘要信息到标准 err 输出中。
- scheddetail：设置 `schedtrace=X` 和 `scheddetail=1` 可以使运行时在每 X 毫秒发出一次详细的多行信息，信息内容主要包括调度程序、处理器、OS 线程 和 Goroutine 的状态。

**查看基本信息**

```
go build trace.go
GODEBUG=schedtrace=1000 ./trace
SCHED 0ms: gomaxprocs=8 idleprocs=6 threads=4 spinningthreads=1 idlethreads=0 runqueue=0 [1 0 0 0 0 0 0 0]
Hello World
SCHED 1010ms: gomaxprocs=8 idleprocs=8 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0 0 0 0 0 0 0]
Hello World
SCHED 2014ms: gomaxprocs=8 idleprocs=8 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0 0 0 0 0 0 0]
Hello World
SCHED 3024ms: gomaxprocs=8 idleprocs=8 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0 0 0 0 0 0 0]
Hello World
SCHED 4027ms: gomaxprocs=8 idleprocs=8 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0 0 0 0 0 0 0]
Hello World
SCHED 5029ms: gomaxprocs=8 idleprocs=7 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0 0 0 0 0 0 0]
```

sched：每一行都代表调度器的调试信息，后面提示的毫秒数表示启动到现在的运行时间，输出的时间间隔受 `schedtrace` 的值影响。

gomaxprocs：当前的 CPU 核心数（GOMAXPROCS 的当前值）。

idleprocs：空闲的处理器数量，后面的数字表示当前的空闲数量。

threads：OS 线程数量，后面的数字表示当前正在运行的线程数量。

spinningthreads：自旋状态的 OS 线程数量。

idlethreads：空闲的线程数量。

runqueue：全局队列中中的 Goroutine 数量，而后面的[0 0 0 0 0 0 0 0] 则分别代表这 8 个 P 的本地队列正在运行的 Goroutine 数量。

**查看详细信息**

```
go build trace.go
GODEBUG=scheddetail=1,schedtrace=1000 ./trace
SCHED 0ms: gomaxprocs=8 idleprocs=6 threads=4 spinningthreads=1 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
P0: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=1 gfreecnt=0 timerslen=0
P1: status=1 schedtick=0 syscalltick=0 m=2 runqsize=0 gfreecnt=0 timerslen=0
P2: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
P4: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
P5: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
P6: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
P7: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
M3: p=0 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
M2: p=1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
M0: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=1
G1: status=1(chan receive) m=-1 lockedm=0
G2: status=1() m=-1 lockedm=-1
G3: status=1() m=-1 lockedm=-1
G4: status=4(GC scavenge wait) m=-1 lockedm=-1
```

G

```
status：G 的运行状态。
m：隶属哪一个 M。
lockedm：是否有锁定 M。
```

G 的运行状态共涉及如下 9 种状态：

| 状态              | 值   | 含义                                                         |
| ----------------- | ---- | ------------------------------------------------------------ |
| _Gidle            | 0    | 刚刚被分配，还没有进行初始化。                               |
| _Grunnable        | 1    | 已经在运行队列中，还没有执行用户代码。                       |
| _Grunning         | 2    | 不在运行队列里中，已经可以执行用户代码，此时已经分配了 M 和 P。 |
| _Gsyscall         | 3    | 正在执行系统调用，此时分配了 M。                             |
| _Gwaiting         | 4    | 在运行时被阻止，没有执行用户代码，也不在运行队列中，此时它正在某处阻塞等待中。 |
| _Gmoribund_unused | 5    | 尚未使用，但是在 gdb 中进行了硬编码。                        |
| _Gdead            | 6    | 尚未使用，这个状态可能是刚退出或是刚被初始化，此时它并没有执行用户代码，有可能有也有可能没有分配堆栈。 |
| _Genqueue_unused  | 7    | 尚未使用。                                                   |
| _Gcopystack       | 8    | 正在复制堆栈，并没有执行用户代码，也不在运行队列中。         |

M

```
p：隶属哪一个 P。
curg：当前正在使用哪个 G。
runqsize：运行队列中的 G 数量。
gfreecnt：可用的G（状态为 Gdead）。
mallocing：是否正在分配内存。
throwing：是否抛出异常。
preemptoff：不等于空字符串的话，保持 curg 在这个 m 上运行。
```

P

```
status：P 的运行状态。
schedtick：P 的调度次数。
syscalltick：P 的系统调用次数。
m：隶属哪一个 M。
runqsize：运行队列中的 G 数量。
gfreecnt：可用的G（状态为 Gdead）
```

| 状态      | 值   | 含义                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| _Pidle    | 0    | 刚刚被分配，还没有进行进行初始化。                           |
| _Prunning | 1    | 当 M 与 P 绑定调用 acquirep 时，P 的状态会改变为 _Prunning。 |
| _Psyscall | 2    | 正在执行系统调用。                                           |
| _Pgcstop  | 3    | 暂停运行，此时系统正在进行 GC，直至 GC 结束后才会转变到下一个状态阶段。 |
| _Pdead    | 4    | 废弃，不再使用。                                             |

# 49、Go 内存分配机制

Go语言内置运行时（就是runtime），抛弃了传统的内存分配方式，改为**自主管理**。这样可以自主地实现更好的内存使用模式，比如内存池、预分配等等。这样，不会每次内存分配都需要进行系统调用。

## 1、设计思想

- 内存分配算法采用Google的`TCMalloc算法`，**每个线程都会自行维护一个独立的内存池**，进行内存分配时优先从该内存池中分配，当内存池不足时才会向加锁向全局内存池申请，**减少系统调用并且避免不同线程对全局内存池的锁竞争**
- 把内存切分的非常的细小，分为多级管理，以降低锁的粒度
- 回收对象内存时，并没有将其真正释放掉，只是放回预先分配的大块内存中，**以便复用**。只有内存闲置过多的时候，才会尝试归还部分内存给操作系统，降低整体开销

## 2、分配组件

Go的内存管理组件主要有：`mspan`、`mcache`、`mcentral`和`mheap`

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071509945.png)

**内存管理单元：mspan**

`mspan`是 内存**管理**的基本单元，该结构体中包含 `next` 和 `prev` 两个字段，它们分别指向了前一个和后一个mspan，每个`mspan` 都管理 `npages` 个大小为 8KB 的页，一个span 是由多个 page 组成的，**这里的页不是操作系统中的内存页，它们是操作系统内存页的整数倍**。

`page`是内存**存储**的基本单元，“对象”放到`page`中

```go
type mspan struct {
    next *mspan // 后指针
    prev *mspan // 前指针
    startAddr uintptr // 管理页的起始地址，指向page
    npages    uintptr // 页数
    spanclass   spanClass // 规格
    ...
}

type spanClass uint8
```

Go有68种不同大小的 spanClass，用于小对象的分配

```go
const _NumSizeClasses = 68
var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536,1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
```

如果按照序号为1的spanClass（对象规格为8B）分配，每个span占用堆的字节数：8k，mspan可以保存1024个对象

如果按照序号为2的spanClass（对象规格为16B）分配，每个span占用堆的字节数：8k，mspan可以保存512个对象

…以此类推

如果按照序号为67的 spanClass（对象规格为32K）分配，每个span占用堆的字节数：32k，mspan 可以保存1个对象

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071515788.png)

字段含义：

- **class**： class ID，每个span结构中都有一个class ID, 表示该span可处理的对象类型
- **bytes/obj**：该class代表对象的字节数
- **bytes/span**：每个span占用堆的字节数，也即页数*页大小
- **objects**: 每个span可分配的对象个数，也即（bytes/spans）/（bytes/obj）
- **waste bytes**: 每个span产生的内存碎片，也即（bytes/spans）%（bytes/obj）

大于 32k 的对象出现时，会直接从 heap 分配一个特殊的 span，这个特殊的 span 的类型(class)是**0**, 只包含了一个大对象

**线程缓存：mcache**

mcache 管理线程在本地缓存的 mspan，每个 goroutine 绑定的P都有一个`mcache`字段

```go
type mcache struct {
	alloc [numSpanClasses]*mspan
}

_NumSizeClasses = 68
numSpanClasses = _NumSizeClasses << 1
```

`mcache`用`Span Classes`作为索引管理多个用于分配的`mspan`，它包含所有规格的`mspan`。它是`_NumSizeClasses`的2倍，也就是`68*2=136`，其中*2是将 spanClass 分成了有指针和没有指针两种,方便与垃圾回收。对于每种规格，有2个 mspan，一个 mspan 不包含指针，另一个 mspan 则包含指针。对于无指针对象的`mspan`在进行垃圾回收的时候无需进一步扫描它是否引用了其他活跃的对象。

`mcache`在初始化的时候是没有任何`mspan`资源的，在使用过程中会动态地从`mcentral`申请，之后会缓存下来。当对象小于等于32KB大小时，使用`mcache`的相应规格的`mspan`进行分配。

**中心缓存：mcentral**

mcentral 管理全局的 mspan 供所有线程使用，全局mheap 变量包含 central 字段，每个 mcentral 结构都维护在**mheap**结构内

```go
type mcentral struct {
    spanclass spanClass // 指当前规格大小

    partial [2]spanSet // 有空闲object的mspan列表
    full    [2]spanSet // 没有空闲object的mspan列表
}
```

每个 mcentral 管理一种 spanClass 的 mspan，并将有空闲空间和没有空闲空间的 mspan 分开管理。partial 和 full`的数据类型为`spanSet，表示 `mspans `集，可以通过pop、push来获得 mspans

```go
type spanSet struct {
    spineLock mutex
    spine     unsafe.Pointer // 指向[]span的指针
    spineLen  uintptr        // Spine array length, accessed atomically
    spineCap  uintptr        // Spine array cap, accessed under lock

    index headTailIndex  // 前32位是头指针，后32位是尾指针
}
```

简单说下`mcache`从`mcentral`获取和归还`mspan`的流程：

- 获取； 加锁，从`partial`链表找到一个可用的`mspan`；并将其从`partial`链表删除；将取出的`mspan`加入到`full`链表；将`mspan`返回给工作线程，解锁。
- 归还； 加锁，将`mspan`从`full`链表删除；将`mspan`加入到`partial`链表，解锁。

**页堆：mheap**

mheap 管理 Go 的所有动态分配内存，可以认为是 Go 程序持有的**整个堆空间**，**全局唯一**

```go
var mheap_ mheap
type mheap struct {
    lock      mutex    // 全局锁
    pages     pageAlloc // 页面分配的数据结构
    allspans []*mspan // 所有通过 mheap_ 申请的mspans
    // 堆
    arenas [1 << arenaL1Bits]*[1 << arenaL2Bits]*heapArena

    // 所有中心缓存mcentral
    central [numSpanClasses]struct {
        mcentral mcentral
        pad      [cpu.CacheLinePadSize - unsafe.Sizeof(mcentral{})%cpu.CacheLinePadSize]byte
    }
    ...
}
```

所有`mcentral`的集合则是存放于`mheap`中的。`mheap`里的`arena` 区域是堆内存的抽象，运行时会将 `8KB` 看做一页，这些内存页中存储了所有在堆上初始化的对象。运行时使用二维的 runtime.heapArena 数组管理所有的内存，每个 runtime.heapArena 都会管理 64MB 的内存。

当申请内存时，依次经过 `mcache` 和 `mcentral` 都没有可用合适规格的大小内存，这时候会向 `mheap` 申请一块内存。然后按指定规格划分为一些列表，并将其添加到相同规格大小的 `mcentral` 的 `非空闲列表` 后面

## 3、分配对象

- 微对象 (0, 16B)：先使用线程缓存上的微型分配器，再依次尝试线程缓存、中心缓存、堆 分配内存；
- 小对象 [16B, 32KB]：依次尝试线程缓存、中心缓存、堆 分配内存；
- 大对象 (32KB, +∞)：直接尝试堆分配内存；

## 4、分配流程

- 首先通过计算使用的大小规格
- 然后使用`mcache`中对应大小规格的块分配。
- 如果`mcentral`中没有可用的块，则向`mheap`申请，并根据算法找到最合适的`mspan`。
- 如果申请到的`mspan` 超出申请大小，将会根据需求进行切分，以返回用户所需的页数。剩余的页构成一个新的 mspan 放回 mheap 的空闲列表。
- 如果 mheap 中没有可用 span，则向操作系统申请一系列新的页（最小 1MB）

![img](https://raw.githubusercontent.com/jinxiao-netpig/user-image/master/imgs/202410071533201.png)

# 50、Go 内存逃逸机制

## 1、概念

在一段程序中，每一个函数都会有自己的内存区域存放自己的局部变量、返回地址等，这些内存会由编译器在栈中进行分配，每一个函数都会分配一个栈桢，在函数运行结束后进行销毁，但是有些变量我们想在函数运行结束后仍然使用它，那么就需要把这个变量在堆上分配，这种**从"栈"上逃逸到"堆"上的现象就成为内存逃逸**。

在栈上分配的地址，一般由系统申请和释放，不会有额外性能的开销，比如函数的入参、局部变量、返回值等。在堆上分配的内存，如果要回收掉，需要进行 GC，那么 GC 一定会带来额外的性能开销。编程语言不断优化 GC 算法，主要目的都是为了减少 GC 带来的额外性能开销，变量一旦逃逸会导致性能开销变大。

## 2、逃逸机制

编译器会根据变量是否被外部引用来决定是否逃逸：

1. 如果函数外部没有引用，则优先放到栈中；
2. 如果函数外部存在引用，则必定放到堆中;
3. 如果栈上放不下，则必定放到堆上;

逃逸分析也就是由编译器决定哪些变量放在栈，哪些放在堆中，通过编译参数`-gcflag=-m`可以查看编译过程中的逃逸分析，发生逃逸的几种场景如下：

### 指针逃逸

```go
package main

func escape1() *int {
    var a int = 1
    return &a
}

func main() {
	escape1()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:6: moved to heap: a
```

函数返回值为局部变量的指针，函数虽然退出了，但是**因为指针的存在，指向的内存不能随着函数结束而回收，因此只能分配在堆上**。

### 栈空间不足

```go
package main

func escape2() {
    s := make([]int, 0, 10000)
    for index, _ := range s {
    	s[index] = index
    }
}

func main() {
	escape2()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:11: make([]int, 10000, 10000) escapes to heap
```

当栈空间足够时，不会发生逃逸，但是当变量过大时，已经完全超过栈空间的大小时，将会发生逃逸到堆上分配内存。局部变量s占用内存过大，编译器会将其分配到堆上

### 变量大小不确定

```go
package main

func escape3() {
    number := 10
    s := make([]int, number) // 编译期间无法确定slice的长度
    for i := 0; i < len(s); i++ {
    	s[i] = i
    }
}

func main() {
	escape3()
}
```

编译期间无法确定slice的长度，这种情况为了保证内存的安全，编译器也会触发逃逸，在堆上进行分配内存。直接`s := make([]int, 10)`不会发生逃逸

### 动态类型

动态类型就是编译期间不确定参数的类型、参数的长度也不确定的情况下就会发生逃逸

空接口 interface{} 可以表示任意的类型，如果函数参数为 interface{}，编译期间很难确定其参数的具体类型，也会发生逃逸。

```go
package main

import "fmt"

func escape4() {
	fmt.Println(1111)
}

func main() {
	escape4()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:6:14: 1111 escapes to heap
```

fmt.Println(a …interface{}) 函数参数为 interface，编译器不确定参数的类型，会将变量分配到堆上

### 闭包引用对象

```go
package main

func escape5() func() int {
    var i int = 1
    return func() int {
        i++
        return i
    }
}

func main() {
	escape5()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:6: moved to heap: i
```

闭包函数中局部变量 i 在后续函数是继续使用的，编译器将其分配到堆上

## 总结

1. 栈上分配内存比在堆中分配内存效率更高
2. 栈上分配的内存不需要 GC 处理，而堆需要
3. 逃逸分析目的是决定内分配地址是栈还是堆
4. 逃逸分析在编译阶段完成

因为无论变量的大小，只要是指针变量都会在堆上分配，所以对于小变量我们还是使用传值效率（而不是传指针）更高一点。

# 51、Go 内存对齐机制



























