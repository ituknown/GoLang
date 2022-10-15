# 数组

数组是任何一门编程语言都绕不过去的一种数据类型，Go 也不例外。之所以特别强调一下数组，是因为在 Go 中，数组是切片的基础，如果想要理解切片的原理，那么数组是绕不过去的一道坎。所以我觉得还是应该简单的梳理下数组的基本原理及内存布局。

数组这种数据结构特点特别鲜明：

- “固定大小且不可变”
- “是内存中连续的一块区域”

在 Go 中声明一个数组变量与其他编程语言如出一辙。比如声明一个 int 类型，长度为 4 的数组：

```go
var i1 [4]int
var i2 [4]int = [4]int{}

fmt.Printf("i1: %v\n", i1)
fmt.Printf("i2: %v\n", i2)
```

这两种声明方式在使用时是等效的，当我们打印时会看到数组中每个下标位置的值都是 0，这是因为 int 类型的默认值就是 0（如果是 string 类型那么默认值就是 nil）：

```log
i1: [0 0 0 0]
i2: [0 0 0 0]
```

除了这两种声明方式外，Go 还提供了另外两种声明方式，即赋初始值：

```go
var i3 [4]int = [4]int{1, 1}
var i4 [4]int = [4]int{2: 1}

fmt.Printf("i3: %v\n", i3)
fmt.Printf("i4: %v\n", i4)
```

如果有其他编程语言基础，i3 的声明方式就很好理解：在声明时就在下标为 0 和 1 的位置设置初始值。等同于：

```go
var i3 [4]int
i3[0] = 1
i3[1] = 1
```

但是 i4 就不那么好理解了，因为这种声明方式在其他编程语言中很难见到。他的含义与 i3 一样，都是赋初始值。不同点是：指定下标赋初始值。这里就是指定给下标为2的空间赋初始值。因此，我们可以换一种方式声明 i3：

```go
var i3 [4]int = [4]int{0: 1, 1: 1}

// 隐藏的含义是:
var i3 [4]int = [4]int{0: 1, 1: 1, 2: 0, 3: 0}
```

现在，我们再来打印下 i3 和 i4 的值就很容易理解了：

```log
i3: [1 1 0 0]
i4: [0 0 1 0]
```

# 数组的内存布局

数组的最大特点是：是内存中连续的一块空间。该怎么体现呢？来看下下面的代码，打印每个下标数据的内存地址：

```go
var i5 [4]int8 = [4]int8{1, 2, 3, 4}

fmt.Printf("i5: %v\n", i5)
fmt.Printf("i5    addr: %p\n", &i5)
fmt.Printf("i5[0] addr: %p\n", &i5[0])
fmt.Printf("i5[1] addr: %p\n", &i5[1])
fmt.Printf("i5[2] addr: %p\n", &i5[2])
fmt.Printf("i5[3] addr: %p\n", &i5[3])
```

为了便于理解，这次我将数组的类型换成 int8（因为 int8 在内存中占 1 字节，如果数组是内存中连续的一块空间，那么数组中每个元素的内存地址就应该是递增），现在来看下输出结果：

```go
i5: [1 2 3 4]
i5    addr: 0xc00012e020
i5[0] addr: 0xc00012e020
i5[1] addr: 0xc00012e021
i5[2] addr: 0xc00012e022
i5[3] addr: 0xc00012e023
```

注意看，i5[0] 的内存地址是 `0xc00012e020`，而 i5[1]、i5[2] 和 i5[3] 的内存地址也是以次递增，也印证了数组是内存中连续的一块空间。

再来看下 i5 和 i5[0] 的内存地址，会发现他们是相等的，这也说明了数组的另一个特点：数组的内存地址就是首元素的内存地址。

现在我们可以重新理解数组：**内存中连续的一块区域，数组的地址就是首元素的内存地址。** 而知道这点我们就能够理解根据下标获取值得原理了。

想一下，我们是如何获取数组下标的元素值得？是不是指定下标就好了（比如 `i5[1]`），感觉好简单是不是？

那为什么根据下标就能获取元素值了呢？这还是要归结于数组是内存中连续空间的内存结构，所以在底层上它还是根据内存的加减来计算下标值。

还是以 i5 为例，他的第一个元素内存地址为 `0xc00012e020`，第二个元素内存地址为 `0xc00012e021`。以此类推，很明显就能看出来内存地址在 “增加”。

而每次应该 “增加” 多少呢？这个又与数据类型的宽度有关了。在计算机中，所有的数据虽然都是以二进制存储，但是最小的存储单位却是 “字节”，而一字节就是8位。

在内存中也是如此，每次增加1个地址空间，其实就是增加8位（即1比特）。比如由 `0xc00012e020` 增加到 `0xc00012e021`，其实就是增加8位（1字节）。

而 int8 这种数据类型就是字面上的意思，占用8位空间。因此你会看到在内存地址中每次是增加1个字节单位。如果你将 int8 换成 int32，那么他的每个元素在内存地址上每次就是增加4个字节单位，输出结果就变成下面的样子了：

```log
i5[0] addr: 0xc00012e020 +0
i5[1] addr: 0xc00012e024 +4
i5[2] addr: 0xc00012e028 +8
i5[3] addr: 0xc00012e02c +12
```

但是在内存计算中我们不可能真的去 +4、+8 这么计算，因为数据所占用的空间大小是不确定的。最典型的就是 int 这个数据类型，它在32位计算机上大小是32位（3字节），但是在64位计算机上却是64位（8字节）。它不是唯一不变的，因此在实际使用时还是建议使用 int32、int64 这样具体的数据类型。

不过，也不用担心数据大小的问题。因此 Go 提供了 unsafe 标准库，其中 Sizeof 方法就是用于计算某个数据类型所占用的空间大小，示例：

```go
var i1 int8
var i2 int16
var i3 int32
var i4 int64
var i5 int

fmt.Printf("sizeof(int8): %d\n", unsafe.Sizeof(i1))
fmt.Printf("sizeof(int16): %d\n", unsafe.Sizeof(i2))
fmt.Printf("sizeof(int32): %d\n", unsafe.Sizeof(i3))
fmt.Printf("sizeof(int64): %d\n", unsafe.Sizeof(i4))
fmt.Printf("sizeof(int): %d\n", unsafe.Sizeof(i5))
```

输出结果如下：

```log
sizeof(int8): 1
sizeof(int16): 2
sizeof(int32): 4
sizeof(int64): 8
sizeof(int): 8
```

现在我们知道了数组的原理也知道如何计算数据类型占用空间大小之后，我们就可以基于内存的方式来获取数组的元素值了。

比如根据内存加减运算获取 i5[1] 的值：

```go
// 得到数组的地址(首元素的地址)
ptr := unsafe.Pointer(&i5[0])

// 转换成 uintptr 进行指针加减运算
ptr2 := uintptr(ptr)

// 获取数组数据类型的占用空间大小
// 加上数据类型大小后就是下个元素的内存地址了
size := unsafe.Sizeof(i5[0])
ptr2 += size

// 将 uintptr 再次转换为 unsafe.Pointer, 用于获取指针值
ptr3 := unsafe.Pointer(ptr2)
var i5_1 *int8 = (*int8)(ptr3)

fmt.Printf("i5[1]: %v, i5[1] == i5_1: %t\n", i5[1], i5[1] == *i5_1)
```

当执行这段代码后你会发现输出结果如下：

```log
i5[1]: 2, i5[1] == i5_1: true
```

是不是很有意思？这就是数组根据下标获取值的原理。

当我们想要获取数组中某个下标的值得时，他底层的原理就是根据数组的首地址，进行加减运算得到某个下标的内存地址。得到下标的内存地址之后再转换成目标数据类型就是我们要得到的结果。

因此当我们思考数组的内存布局时我们可以将它想象中如下数据结构：

```go
type ArrayLayout struct {
	// 数组的首地址
	Date uintptr

	// 数组元素的宽度(unsafe.Sizeof)
	Weight uintptr

	// 数组的元素个数
	Len int
}
```

如果以这种结构体来理解数组是不是就突然觉得好简单了？接下面我们就以该结构体为例，做适当地修改，来实现基于内存的操作获取下标值，直接上代码：

```go
type ArrayLayout[T interface{}] struct {
	// 数组的首地址
	Date uintptr

	// 数组元素的宽度(unsafe.Sizeof)
	Weight uintptr

	// 数组的元素个数
	Len int
}

func (r *ArrayLayout[T]) Get(index int) T {
	if index < 0 || index >= r.Len {
		panic("index out of array")
	}

  // 数据类型的宽度 * 下标所在的位置 就是该数据在内存中的相对位置
	w := r.Weight * uintptr(index+1)
	adr := r.Date + w

	ptr := unsafe.Pointer(adr)
	return *(*T)(ptr)
}
```

之后我们来做下数据测试：

```go
func main() {

	var i5 [4]int = [4]int{1, 2, 3, 4}

	var layout = &ArrayLayout[int]{
		Date:   uintptr(unsafe.Pointer(&i5)),
		Weight: unsafe.Sizeof(i5[0]),
		Len:    len(i5),
	}

	v := layout.Get(2)
	fmt.Println(v)
}
```

当你运行该代码时你就会得到预想中的结果~

# 切片 slice

之所以在上面使用长篇大论来介绍数组的根本原因是 slice 底层就是基于数组实现的。在实际使用中，用的最多的就是 slice，而数组却很少使用。原因是 slice 对数组做了改进，不再有大小的限制，可以随意 “扩容”，使用起来特别灵活。

切片其实是一种结构体，他的结构体定义在 `runtime/slice.go` 文件中，结构体如下：

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

切片相比较数组而言，它多了一个 cap（即容量）。这个是切片的主要特点，与扩容有关。而 slice 的本质就是持有一个指向特定数据类型的数组的地址（array）。

我们可以使用下面这个图理解 slice：

![slice-struct.png](http://go-media.knowledge.ituknown.cn/slice/slice-struct.png)

如果你仔细观察这个 slice 结构体的话你可能会想到，如果 array 是 nil 怎么办？即：

![slice-array-nil%20.png](http://go-media.knowledge.ituknown.cn/slice/slice-array-nil%20.png)

其实，这个确实是存在的，这个就与 slice 的声明有关了。声明 slice 主要有如下四种形式（Type 是数据类型，如 int）：

```go
var s1 []Type
var s2 = []Type{}
var s3 = make([]Type, len)
var s4 = make([]Type, len, cap)
```

s1 和 s2 含义相同，都表示创建一个 nil 切片，所谓的 nil 切片表示的就是初始化时不创建底层数组。

s3 和 s4 表示指定底层数组的长度创建切片，不过如果指定的 len 和 cap 都是 0 时其实与 s1、s2 是等效的。

看如下示例：

```go
var s1 []int
var s2 = []int{}
var s3 = make([]int, 0)
var s4 = make([]int, 0, 0)

fmt.Printf("s1 is nil: %t, len: %d, cap: %d\n", s1 == nil, len(s1), cap(s1))
fmt.Printf("s2 is nil: %t, len: %d, cap: %d\n", s2 == nil, len(s2), cap(s2))
fmt.Printf("s3 is nil: %t, len: %d, cap: %d\n", s3 == nil, len(s3), cap(s3))
fmt.Printf("s4 is nil: %t, len: %d, cap: %d\n", s4 == nil, len(s4), cap(s4))
```

输出结果为：

```log
s1 is nil: true, len: 0, cap: 0
s2 is nil: false, len: 0, cap: 0
s3 is nil: false, len: 0, cap: 0
s4 is nil: false, len: 0, cap: 0
```

这三种声明方式对应的就是没有底层数组的形式（ptr 为 nil）：

![slice-array-nil%20.png](http://go-media.knowledge.ituknown.cn/slice/slice-array-nil%20.png)

如果我们指定创建的切片的底层数组长度大于 0 时，就会得到不一样的输出结果：

```go
var s5 = make([]int32, 2, 4)

fmt.Printf("s5 is nil: %t, len: %d, cap: %d, v: %v\n", s5 == nil, len(s5), cap(s5), s5)
```

输出如下：

```log
s5 is nil: false, len: 2, cap: 4, v: [0 0]
```

s5 与前面几个切片最大的切片时创建了一个底层数组，对应的内存布局如下：

![slice-make-len&cap-indiff.png](http://go-media.knowledge.ituknown.cn/slice/slice-make-len&cap-indiff.png)

到这里，相信对切片已经有了基本的认识。下面再从结构体角度来理解使用 make 关键字创建切片的含义：

- `make([]Type, 10)` 等价于：

```go
var s = slice{
	array: ptr
	len: 10
	cap: 10
}
```

- `make([]Type, 4, 10)` 等价于：

```go
var s = slice{
	array: ptr
	len: 4
	cap: 10
}
```

- `make([]Type, 0)`、`[]Type`、`[]Type{}` 等价于：

```go
var s = slice{
	array: nil
	len: 0
	cap: 0
}
```

## 切片扩容

切片的灵活之处在于它支持 “扩容”，当然你可能注意到了。它底层使用的数组，数组是不支持修改的，它是如果实现扩容的？

嗯... 其实就是创建新的数组 😅

也就是说切片的扩容其实就是初始化新的 slice 结构体，并将老结构体中的数据拷贝过去。对，仅仅如此~

而切片扩容的临界点就是 `len == cap`。在实际工作中你可能经常注意到下面这种写法：

```go
var i1 []int
i1 = append(i1, 10)
```

即 i1 的明明没有底层数组为什么能够使用 append 将数据 10 添加到切片中？原因就是因为触发了扩容！

i1 是没有底层数组的，它的 len 和 cap 都是 0，满足扩容的临界点。所以当我们使用 append 添加数据之前，他会先触发一次扩容，当扩容完成之后再将数据 10 添加到新的底层数组中。

我们可以通过打印切片的地址来做验证：

```go
var i1 []int
fmt.Printf("i1 addr: %p, len: %d, cap: %d, v: %v\n", i1, len(i1), cap(i1), i1)

i1 = append(i1, 10)
fmt.Printf("i1 addr: %p, len: %d, cap: %d, v: %v\n", i1, len(i1), cap(i1), i1)
```

输出结果为：

```log
i1 addr: 0x0, len: 0, cap: 0, v: []
i1 addr: 0xc0000220b8, len: 1, cap: 1, v: [10]
```

|**Note**|
|:-------|
|这里要特别注意下，初始时他的内存地址是 0x0，跟 `make([]Type, 0)` 是有区别的，下面再会特别说明。|

当 append 一个数据之后，内存地址变成 0xc0000220b8 了，原因就是因为触发了扩容。

另外要特别说一下，当切片的容量在 1024 之下时，每次扩容时容量扩大 2 倍（注意不是 2 的幂次方！），当容量超过 1024 之后，接下来的扩容通常在 1.5 ~ 3 倍之间。

现在我们来使用下面这段代码来验证下切片的扩容机制：

```go
var i1 = make([]int32, 0)
fmt.Printf("addr: %p, len: %2d, cap: %2d, v: %v\n", i1, len(i1), cap(i1), i1)

var i int32
for i = 0; i < 10; i++ {
	i1 = append(i1, i)
	fmt.Printf("addr: %p, len: %2d, cap: %2d, v0: %d, v: %v\n", i1, len(i1), cap(i1), i1[0], i1)
}
```

我使用了颜色标记了每次扩容的实际以及数据范围，如下图：

![slice-len&cap-grow.png](http://go-media.knowledge.ituknown.cn/slice/slice-len&cap-grow.png)

其他就不多说了，基本上看到这个图就什么都明白了~

## nil 切片 vs 空切片

还记得前面在前面[切片扩容](#切片扩容)中声明的一个切片，当输出它的地址时发现是 0x0 吗？

下面，这里就来探讨这个问题。首先声明三个切片，如下：

```go
var i1 []int
var i2 []int = []int{}
var i3 = make([]int, 0)

fmt.Printf("i1 is nil: %t, len: %d, cap: %d\n", i1 == nil, len(i1), cap(i1))
fmt.Printf("i2 is nil: %t, len: %d, cap: %d\n", i2 == nil, len(i2), cap(i2))
fmt.Printf("i3 is nil: %t, len: %d, cap: %d\n", i3 == nil, len(i3), cap(i3))
```

当我们执行时会发现输出如下结果：

```log
i1 is nil: true, adr: 0x0, len: 0, cap: 0
i2 is nil: false, adr: 0x119f310, len: 0, cap: 0
i3 is nil: false, adr: 0x119f310, len: 0, cap: 0
```

注意看，`i1` 是一个 nil 切片，并且内存地址是零值（0x0）。而 i2 和 i3 虽然 cap 都是 0，但是却有具体的内存地址。实际上，按照 i1 方式声明切片的就是 nil 切片，而 i2 和 i3 则是空切片，。

我们可以使用下面的代码来理解 i1、i2 和 i3：

```go
i1: = struct {}

i2 := slice {
	array: nil,
	len: 0,
	cap: 0,
}

i3 := slice {
	array: nil,
	len: 0,
	cap: 0,
}
```

i1 是一个空结构体（没有任何名字），这种空结构体在 go 中是不需要分配内存的，可以理解为是变相的 nil。而 i2、i3 则是有具体的结构体，只不过没有指向具体地址的数组空间。

不管是 nil 切片还是空切片在实际使用时效果是一致的，比如第一次 append 时一定会触发扩容，看着似乎没有什么区别。

不过，如果你使用 `json.Marshal` 进行序列化 nil 或者空切片时会得到不同的结果：

```go
m, _ := json.Marshal(i1)
fmt.Println("marshal i1:", string(m))

m, _ = json.Marshal(i2)
fmt.Println("marshal i2:", string(m))

m, _ = json.Marshal(i3)
fmt.Println("marshal i2:", string(m))
```

输出结果：

```log
marshal i1: null
marshal i2: []
marshal i2: []
```

所以，对于有序列化的需求要注意了~

--

完结，撒花🎉🎉