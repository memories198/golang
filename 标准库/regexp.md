## regexp篇

Go 语言的 `regexp` 包提供了对正则表达式的支持，可以用于字符串的匹配、搜索和替换等操作。以下是 `regexp` 包的基本使用方法和一些常见的示例。

### 编译正则表达式

#### regexp.Compile()

将正则表达式的字符串编译为对象，编译失败会返回一个err

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	regexpLazy, err := regexp.Compile("a([a-z|0-9]+?)b") // 尽可能少的匹配(懒惰匹配)
	if err != nil {
		fmt.Println("Error compiling regex:", err)
	}

	regexpGreed, err := regexp.Compile("a([a-z|0-9]+)b") // 尽可能多的匹配(贪婪匹配)
	if err != nil {
		fmt.Println("Error compiling regex:", err)
	}

	fmt.Printf("%v\n", regexpGreed.FindAllString("ab", -1)) // 输出：[] 没有满足的子串，返回nil
	fmt.Printf("%v\n", regexpLazy.FindAllString("ab", -1))  // 输出：[] 没有满足的子串，返回nil

	fmt.Printf("%v\n", regexpGreed.FindAllString("ababb", -1)) // 输出：[ababb]
	fmt.Printf("%v\n", regexpLazy.FindAllString("ababb", -1))  // 输出：[abab]

	fmt.Printf("%v\n", regexpGreed.FindAllString("a123b", -1)) // 输出：[]
	fmt.Printf("%v\n", regexpLazy.FindAllString("a123b", -1))  // 输出：[]

	fmt.Printf("%v\n", regexpGreed.FindAllString("a.bbatb", -1)) // 输出：[atb]
	fmt.Printf("%v\n", regexpLazy.FindAllString("a.bbatb", -1))  // 输出：[atb]
}
```

#### regexp.MustCompile()

在编译失败时会引发恐慌（panic），适合在程序启动时使用。

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	re := regexp.MustCompile(`(a|b`) // panic示例
	fmt.Println(re)
}
```

### 匹配字符串

#### re.MatchString()

#### re.FindString()

#### re.FindAllString()

#### re.FindStringSubmatch()

### 替换字符串

使用 `ReplaceAllString` 方法可以替换匹配的字符串。

```go
replaced := re.ReplaceAllString("ahello b aworld b", "X")
fmt.Println(replaced) // 输出: X X
```

### 使用正则表达式的标志

可以使用 `regexp.Compile` 的第二个参数来设置标志，例如 `(?i)` 表示不区分大小写。

```go
re := regexp.MustCompile("(?i)hello")
matched := re.MatchString("Hello")
fmt.Println(matched) // 输出: true
```

### 示例代码

以下是一个完整的示例，展示了如何使用 `regexp` 包：

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	// 编译正则表达式
	re := regexp.MustCompile(`a([a-z]+)b`)

	// 匹配字符串
	fmt.Println(re.MatchString("ahello b")) // true

	// 查找匹配
	fmt.Println(re.FindString("ahello b")) // ahello b

	// 查找所有匹配
	results := re.FindAllString("ahello b aworld b", -1)
	fmt.Println(results) // [ahello b aworld b]

	// 提取子匹配
	submatches := re.FindStringSubmatch("ahello b")
	fmt.Println(submatches) // [ahello b hello]

	// 替换字符串
	replaced := re.ReplaceAllString("ahello b aworld b", "X")
	fmt.Println(replaced) // X X
}
```
