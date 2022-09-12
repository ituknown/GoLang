
# 整数与数组

```go
package main

import "fmt"

func myFunc(num int, arr [2]int) {
	fmt.Printf("%5s myFunc - num=(%d, %p) arr=(%v, %p)\n", "in", num, &num, arr, &arr)
}

func main() {

	num := 30
	arr := [2]int{66, 77}

	fmt.Printf("%5s myFunc - num=(%d, %p) arr=(%v, %p)\n", "after", num, &num, arr, &arr)
	myFunc(num, arr)
	fmt.Printf("%5s myFunc - num=(%d, %p) arr=(%v, %p)\n", "after", num, &num, arr, &arr)
}
```


