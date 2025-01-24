## `fmt` 包的使用详解

`fmt` 包是 Go 语言中的核心库之一，用于格式化输入和输出操作。无论是调试还是日志记录，都离不开它的支持。以下将详细介绍其常用功能及占位符说明。

---

### 1. **Println**

`Println` 是最常用的输出函数之一，用于按行打印内容，并在输出末尾自动追加一个换行符。

#### 示例代码

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
	fmt.Println(42, true, 3.14)
}
```

**特点**：

- 自动以空格分隔多个参数。
- 自动在末尾添加换行符。
- 不支持格式化，占位符在此函数中无效。

---

### 2. **Printf**

`Printf` 提供强大的格式化功能，可以通过占位符来指定输出格式。

#### 示例代码

```go
package main

import "fmt"

func main() {
	name := "Alice"
	age := 25
	fmt.Printf("Name: %s, Age: %d\n", name, age)
}
```

**特点**：

- 使用占位符控制输出格式。
- 不会自动添加换行符（需要手动添加 `\n`）。

---

### 3. **支持的数据类型及占位符**

`Printf` 使用占位符表示不同类型的数据。以下是常用占位符及其适用场景：

#### 数值相关

| 占位符  | 描述            | 示例值         | 输出             |
|------|---------------|-------------|----------------|
| `%d` | 十进制整数         | `123`       | `123`          |
| `%b` | 二进制整数         | `123`       | `1111011`      |
| `%o` | 八进制整数         | `123`       | `173`          |
| `%x` | 十六进制（小写）      | `123`       | `7b`           |
| `%X` | 十六进制（大写）      | `123`       | `7B`           |
| `%f` | 浮点数（小数形式）     | `3.14159`   | `3.141590`     |
| `%e` | 浮点数（科学计数法，小写） | `123456.78` | `1.234568e+05` |
| `%E` | 浮点数（科学计数法，大写） | `123456.78` | `1.234568E+05` |

#### 字符串和字符

| 占位符  | 描述       | 示例值       | 输出        |
|------|----------|-----------|-----------|
| `%s` | 字符串      | `"hello"` | `hello`   |
| `%q` | 带双引号的字符串 | `"hello"` | `"hello"` |
| `%c` | 单个字符     | `'A'`     | `A`       |

#### 布尔值和指针

| 占位符  | 描述   | 示例值    | 输出             |
|------|------|--------|----------------|
| `%t` | 布尔值  | `true` | `true`         |
| `%p` | 指针地址 | `&x`   | `0xc0000123a0` |

#### 通用占位符

| 占位符   | 描述                  | 示例值  | 输出                              |
|-------|---------------------|------|---------------------------------|
| `%v`  | 默认格式                | 任意类型 | 根据值的类型决定输出                      |
| `%+v` | 类似 `%v`，但对结构体会添加字段名 | 结构体  | `{field1:value1 field2:value2}` |
| `%#v` | 值的 Go 语法表示          | 任意类型 | 相应值的Go语法表示                      |
| `%T`  | 打印值的类型              | 任意类型 | 类型名称                            |

#### 示例使用

假设我们有一个Go语言的结构体实例，我们可以使用 `%v`, `%+v`, 和 `%#v` 来展示这个结构体的不同视图：

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func main() {
	p := Person{Name: "John Doe", Age: 30}

	fmt.Printf("%v\n", p)  // 输出: {John Doe 30}
	fmt.Printf("%+v\n", p) // 输出: {Name:John Doe Age:30}
	fmt.Printf("%#v\n", p) // 输出: main.Person{Name:"John Doe", Age:30}
	fmt.Printf("%T\n", p)  // 输出: main.Person
}

```

---

### 4. **补充占位符功能**

#### 宽度和精度控制

可以使用 `宽度.精度` 来控制输出的格式。

| 格式      | 描述           | 示例值       | 输出       |
|---------|--------------|-----------|----------|
| `%6d`   | 至少6个字符宽度，右对齐 | `42`      | `    42` |
| `%-6d`  | 至少6个字符宽度，左对齐 | `42`      | `42    ` |
| `%6.2f` | 宽度6，保留2位小数   | `3.14159` | `  3.14` |
| `%06d`  | 宽度6，前面补0     | `42`      | `000042` |

#### 示例代码

```go
package main

import "fmt"

func main() {
	fmt.Printf("%6d\n", 42)        // 输出：    42
	fmt.Printf("%-6d\n", 42)       // 输出：42    
	fmt.Printf("%6.2f\n", 3.14159) // 输出：  3.14
	fmt.Printf("%06d\n", 42)       // 输出：000042
}
```

---

### 5.**转义字符列表**

| 转义字符         | 描述                                | 示例代码                                    | 输出                     |
|--------------|-----------------------------------|-----------------------------------------|------------------------|
| `\n`         | 换行符，表示换到下一行                       | `fmt.Printf("Hello\nWorld")`            | Hello<br>World         |
| `\t`         | 制表符，插入水平制表（Tab）                   | `fmt.Printf("A\tB\tC")`                 | A    B    C            |
| `\\`         | 反斜杠（`\`）本身                        | `fmt.Printf("Path: C:\\Program Files")` | Path: C:\Program Files |
| `\"`         | 双引号                               | `fmt.Printf("Quote: \"Hello World\"")`  | Quote: "Hello World"   |
| `\'`         | 单引号                               | `fmt.Printf("Char: \'A\'")`             | Char: 'A'              |
| `\r`         | 回车符                               | `fmt.Printf("Line1\rLine2")`            | Line2                  |
| `\f`         | 换页符                               | `fmt.Printf("Page1\fPage2")`            | Page1<br>Page2         |
| `\b`         | 退格符（删除前一个字符）                      | `fmt.Printf("AB\bC")`                   | AC                     |
| `\a`         | 响铃符（触发系统响铃或警报音，部分终端支持）            | `fmt.Printf("Alert\a")`                 | （响铃）                   |
| `\v`         | 垂直制表符                             | `fmt.Printf("A\vB\vC")`                 | A<br> B<br> C          |
| `\xhh`       | 以十六进制表示字符（`hh`为两位十六进制数）           | `fmt.Printf("\x48\x65\x6C\x6C\x6F")`    | Hello                  |
| `\uhhhh`     | 以 Unicode 表示字符（`hhhh`为四位十六进制）     | `fmt.Printf("\u4F60\u597D")`            | 你好                     |
| `\Uhhhhhhhh` | 以 Unicode 表示字符（`hhhhhhhh`为八位十六进制） | `fmt.Printf("\U0001F600")`              | 😀                     |

---

#### 示例代码

##### 使用转义字符换行和制表

```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello\nWorld\tGo\n") // 换行 + 制表符
}
```

**输出**：

```
Hello
World   Go
```

---

#### 打印路径

```go
package main

import "fmt"

func main() {
	fmt.Printf("Windows Path: C:\\Program Files\\Go\\bin\n")
}
```

**输出**：

```
Windows Path: C:\Program Files\Go\bin
```

---

#### Unicode 输出

```go
package main

import "fmt"

func main() {
	fmt.Printf("Unicode: \u4F60\u597D\n") // 输出“你好”
	fmt.Printf("Emoji: \U0001F600\n")     // 输出😀
}
```

**输出**：

```
Unicode: 你好
Emoji: 😀
```

---

#### 注意事项

1. **与占位符结合使用**：转义字符可以与占位符混合使用，但需确保格式字符串的正确性。例如：
   ```go
   fmt.Printf("Name:\t%s\nAge:\t%d\n", "Alice", 25)
   ```
   **输出**：
   ```
   Name:   Alice
   Age:    25
   ```

2. **`%q` 和转义字符**：`%q` 格式化输出时会自动为字符串添加引号，并对特殊字符进行转义。例如：
   ```go
   fmt.Printf("%q\n", "Hello\nWorld")
   ```
   **输出**：
   ```
   "Hello\nWorld"
   ```

---

通过合理使用转义字符，可以进一步优化输出的格式和可读性，特别是在控制台输出表格、路径或特殊符号时效果显著。

### 6. **其它常用函数**

#### Sprintf

`Sprintf` 和 `Printf` 类似，但它不会直接输出结果，而是返回格式化后的字符串。

```go
package main

import "fmt"

func main() {
	name := "Alice"
	age := 25
	result := fmt.Sprintf("Name: %s, Age: %d", name, age)
	fmt.Println(result) // 输出：Name: Alice, Age: 25
}
```

#### Errorf

`Errorf` 用于创建格式化的错误信息。

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := fmt.Errorf("an error occurred: %s", "file not found")
	fmt.Println(errors.Unwrap(err)) // 输出：an error occurred: file not found
}
```

---

### 7. **调试利器 `%v` 和 `%+v`**

- `%v`：按默认格式输出值。
- `%+v`：在结构体中显示字段名和字段值。
- `%#v`：按 Go 语法格式输出值（更适合调试）。

#### 示例代码

```go
package main

import "fmt"

type User struct {
	Name string
	Age  int
}

func main() {
	user := User{Name: "Alice", Age: 25}
	fmt.Printf("%v\n", user)  // 输出：{Alice 25}
	fmt.Printf("%+v\n", user) // 输出：{Name:Alice Age:25}
	fmt.Printf("%#v\n", user) // 输出：main.User{Name:"Alice", Age:25}
}
```

---

### 8. **多参数类型匹配**

如果传入的参数与占位符的数量或类型不匹配，`fmt` 会报错。

#### 示例代码

```go
package main

import "fmt"

func main() {
	fmt.Printf("%d %s\n", 42) // 编译错误：占位符数量不匹配
}
```

**提示**：

- 在格式复杂时，可以通过运行时校验（如 `go vet`）避免错误。

---

### 总结

`fmt` 包功能强大，特别是在格式化输出、日志记录和调试时尤为有用。掌握占位符的使用和高级特性，可以显著提升代码质量和可读性。

以下是 `fmt` 包中常见转义字符的补充说明。转义字符在格式化字符串中用于表示特殊字符或控制格式输出的行为。

---
