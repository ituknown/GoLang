
**空切片**

```go
var s1 []int
var s2 = make([]int, 0)
var s3 = []int{}

fmt.Printf("s1 is nil: %t, len: %d, cap: %d\n", s1 == nil, len(s1), cap(s1))
fmt.Printf("s2 is nil: %t, len: %d, cap: %d\n", s2 == nil, len(s2), cap(s2))
fmt.Printf("s3 is nil: %t, len: %d, cap: %d\n", s3 == nil, len(s3), cap(s3))
```

**切片扩容**

```go
var i1 = make([]int32, 0, 0)
fmt.Printf("addr: %p, len: %2d, cap: %2d, v: %v\n", &i1, len(i1), cap(i1), i1)

var i int32
for i = 0; i < 10; i++ {
	i1 = append(i1, i)
	fmt.Printf("addr: %p, len: %2d, cap: %2d, v0: %d, v: %v\n", &i1[0], len(i1), cap(i1), i1[0], i1)
}
```

![slice-len&cap-grow.png](http://go-media.knowledge.ituknown.cn/slice/slice-len&cap-grow.png)