# 通用输出

```log
%v   仅输出变量的值(常用于基本数据类型, 如 int/bool/float)
%+v  输出变量的字段和值(常用于结构体和map)
%#v  在 `%+v` 基础上还输出结构体名字
%T   输出数据类型, 如果是结构体会输出结构体名称
%%   输出 `%` 号
```

示例：

```go
package main

type User struct {
  Name string
  Age  uint8
}

func main() {
  var user = User{"Bob", 18}
  var num  = 18

  fmt.Printf("%v\n", user)
  fmt.Printf("%+v\n", user)
  fmt.Printf("%#v\n", user)
  fmt.Printf("%T\n", user)

  fmt.Printf("%v\n", num)
  fmt.Printf("%+v\n", num)
  fmt.Printf("%#v\n", num)
  fmt.Printf("%T\n", num)
}
```

| **变量** |  **`%v`**  |      **`%+v`**      |             **`%#v`**             |  **`%T`**   |
| :------: | :--------: | :-----------------: | :-------------------------------: | :---------: |
|   user   | `{Bob 18}` | `{Name:Bob Age:18}` | `main.User{Name:"Bob", Age:0x12}` | `main.User` |
|   num    |     18     |         18          |                18                 |     int     |

# 整数类型

```log
%b   以二进制形式输出
%c   以 Unicode 码点所表述的字符输出
%d   以十进制形式输出
%o   以八进制形式输出
%x   以十六进制形式输出, 小写字母 a-f
%X   以十六进制形式输出, 大写字母 A-F
%U   以 Unicode 格式输出
%#   在输出的数据前面加上进制符号(可配合 %b、%o、%x、%X、%U 使用)
```

示例：

```go
package main


func main() {
  var num uint16 = 200

  fmt.Printf("%b\n", num)
  fmt.Printf("%#b\n", num)
  fmt.Printf("%c\n", num)
  fmt.Printf("%d\n", num)
  fmt.Printf("%o\n", num)
  fmt.Printf("%#o\n", num)
  fmt.Printf("%x\n", num)
  fmt.Printf("%#x\n", num)
  fmt.Printf("%X\n", num)
  fmt.Printf("%#X\n", num)
  fmt.Printf("%U\n", num)
  fmt.Printf("%#U\n", num)
}
```

| **变量** |  **`%b`**  |  **`%#b`**   | **`%c`** | **`%d`** | **`%o`** | **`%#o`** | **`%x`** | **`%#x`** | **`%X`** | **`%X`** |   **`%U`**   |  **`%#U`**   |
| :------: | :--------: | :----------: | :------: | :------: | :------: | :-------: | :------: | :-------: | :------: | :------: | :----------: | :----------: |
|   num    | `11001000` | `0b11001000` |   `È`    |  `200`   |  `310`   |  `0310`   |   `c8`   |  `0xc8`   |   `C8`   | `U+0XC8` | `U+U+U+00C8` | `U+00C8 'È'` |

# 浮点类型

```log
%b   以二进制指数的形式输出(不保留小数部分)

%e   科学计数法, 如 2.520000e+01
%E   同 %e, 如 2.520000E+01

%f   保留小数部分, 无指数部分(默认保留 6 位小数)
%F   同 %f

%g   追求简洁性, 根据实际情况自动判断采用 %e 或 %f(推荐)
%G   追求简洁性, 根据实际情况自动判断采用 %E 或 %F(推荐)
```

示例：

```go
package main


func main() {
  var num float64 = 25.2

  fmt.Printf("%b\n", num)
  fmt.Printf("%e\n", num)
  fmt.Printf("%E\n", num)
  fmt.Printf("%f\n", num)
  fmt.Printf("%F\n", num)
  fmt.Printf("%g\n", num)
  fmt.Printf("%G\n", num)
}
```

| **变量** |        **`%b`**        |    **`%e`**    |    **`%E`**    |  **`%f`**   |  **`%F`**   | **`%g`** | **`%G`** |
| :------: | :--------------------: | :------------: | :------------: | :---------: | :---------: | :------: | :------: |
|   num    | `7093169413108531p-48` | `2.520000e+01` | `2.520000E+01` | `25.200000` | `25.200000` |  `25.2`  |  `25.2`  |

# 布尔类型

```log
%t   输出 true 或 false
```

# 字符串

```log
%s   字符串或切片的无解释字节输出(常用)
%q   使用双引号包裹的字符串, 有 Go 安全转义

%x   以小写字母的十六进制输出(每字节两字符)
%X   以大写字母的十六进制输出(每字节两字符)
```

示例：

```go
package main


func main() {
  var val string = `hello, "bob"`

  fmt.Printf("%s\n", val)
  fmt.Printf("%q\n", val)
  fmt.Printf("%x\n", val)
  fmt.Printf("%X\n", val)
}
```

| **变量** |    **`%s`**    |      **`%q`**      |          **`%x`**          |          **`%X`**          |
| :------: | :------------: | :----------------: | :------------------------: | :------------------------: |
|   num    | `hello, "bob"` | `"hello, \"bob\""` | `68656c6c6f2c2022626f6222` | `68656C6C6F2C2022626F6222` |

# 切片和指针

```log
%p   输出切片的地址或指针的值
```

示例：

```go
package main


func main() {

  var sli []byte = []byte("hello, world")

  var ptr = &sli[0]

  fmt.Printf("%p\n", sli)
  fmt.Printf("%p\n", ptr)
}
```

| **变量** |    **`%p`**    |
| :------: | :------------: |
|   sli    | `0xc0000140b4` |
|   ptr    | `0xc0000140b4` |

# 扩展

## 指定输出做小宽度

在进行格式化输出时还可以执行输出的字符宽度，以 float64 为例：

```log
%f     默认宽度及精度
%9f    默认精度, 但是宽度为9
%.2f   默认宽度, 但是精度为2(即保留两位小数)
%9.2f  指定数据输出最小宽度为 9, 精度为 2
%9.f   指定数据输出最小宽度为 9, 精度为 0
```

示例：

```go
package main


func main() {

  var val float64 = 65535.123001

  fmt.Printf("%f\n", val)
  fmt.Printf("%4f\n", val)
  fmt.Printf("%.3f\n", val)
  fmt.Printf("%4.3f\n", val)
  fmt.Printf("%4.f\n", val)
}

// 输出示例:

// 65535.123001
// 65535.123001
// 65535.123
// 65535.123
// 65535
```

## 指定对其方向

这个通常要配合输出宽度使用，数据在格式化是默认靠右对其，不过我们可以在数据宽度前加上 `-` 来指定靠右输出。还是以 float64 为例：

```go
package main


func main() {

  var val float64 = 65535.123001

  fmt.Printf("默认:\n")
  fmt.Printf("%12f\n", val)
  fmt.Printf("%12f\n", val)
  fmt.Printf("%12.3f\n", val)
  fmt.Printf("%12.3f\n", val)
  fmt.Printf("%12.f\n", val)

  fmt.Printf("默认:\n")
  fmt.Printf("%-12f\n", val)
  fmt.Printf("%-12f\n", val)
  fmt.Printf("%-12.3f\n", val)
  fmt.Printf("%-12.3f\n", val)
  fmt.Printf("%-12.f\n", val)
}

// 输出示例:

// 默认:
// 65535.123001
// 65535.123001
//    65535.123
//    65535.123
//        65535

// 左对其:
// 65535.123001
// 65535.123001
// 65535.123
// 65535.123
// 65535
```

## 输出数字符号

在格式化输出数字时，对于正数不会输出具体的符号，不过可以加上 `+` 来指定输出符号：

```go
package main


func main() {

  var val int32 = 10

  fmt.Printf("%d\n", val)
  fmt.Printf("%+d\n", val)
}

// 输出示例:

// 10
// +10
```