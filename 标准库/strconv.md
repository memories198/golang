
## strconv篇

在 Go 语言中，`strconv` 包提供了一组函数，用于在字符串和基本数据类型之间进行转换。这个包非常有用，尤其是在处理输入输出、数据解析和格式化时。以下是
`strconv` 包的一些常用功能和示例。

### 1. 常用函数

以下是 `strconv` 包中一些常用的函数：

- **字符串到整数的转换**
    - `strconv.Atoi(s string) (int, error)`: 将字符串转换为整数。
    - `strconv.ParseInt(s string, base int, bitSize int) (i int64, err error)`: 将字符串转换为指定进制和位数的整数。

- **字符串到浮点数的转换**
    - `strconv.ParseFloat(s string, bitSize int) (f float64, err error)`: 将字符串转换为浮点数。

- **整数到字符串的转换**
    - `strconv.Itoa(i int) string`: 将整数转换为字符串。
    - `strconv.FormatInt(i int64, base int) string`: 将整数转换为指定进制的字符串。

- **浮点数到字符串的转换**
    - `strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string`: 将浮点数转换为字符串。

- **布尔值到字符串的转换**
    - `strconv.FormatBool(b bool) string`: 将布尔值转换为字符串。

### 2. 示例代码

以下是一些示例，展示如何使用 `strconv` 包进行各种类型的转换。

#### 字符串到整数

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	str := "123"
	num, err := strconv.Atoi(str)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Converted number:", num)
	}
}
```

#### 字符串到浮点数

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	str := "3.14"
	f, err := strconv.ParseFloat(str, 64)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Converted float:", f)
	}
}
```

#### 整数到字符串

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	num := 456
	str := strconv.Itoa(num)
	fmt.Println("Converted string:", str)
}
```

#### 浮点数到字符串

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	f := 3.14159
	str := strconv.FormatFloat(f, 'f', 2, 64) // 'f' 表示格式，2 表示小数位数
	fmt.Println("Converted float to string:", str)
}
```

#### 布尔值到字符串

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	b := true
	str := strconv.FormatBool(b)
	fmt.Println("Converted bool to string:", str)
}
```

### 3. 进制转换示例

你还可以使用 `strconv` 包进行进制转换，例如将整数转换为二进制、八进制或十六进制字符串。

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	num := 255
	binaryStr := strconv.FormatInt(int64(num), 2) // 二进制
	octalStr := strconv.FormatInt(int64(num), 8)  // 八进制
	hexStr := strconv.FormatInt(int64(num), 16)   // 十六进制

	fmt.Println("Binary:", binaryStr)
	fmt.Println("Octal:", octalStr)
	fmt.Println("Hexadecimal:", hexStr)
}
```

### 4. 错误处理

在进行字符串转换时，务必检查返回的错误，以确保转换成功。例如，`strconv.Atoi` 和 `strconv.ParseFloat`
都会返回一个错误，如果输入字符串无法转换为目标类型，则会返回一个非 `nil` 的错误。

### 总结

`strconv` 包是 Go 语言中处理字符串与基本数据类型之间转换的重要工具。通过使用这个包，你可以方便地进行各种类型的转换，确保你的程序能够正确处理输入和输出。
