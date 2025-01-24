## strings库篇

在Go语言中，标准库 `strings` 包提供了许多常用的字符串处理函数。这些函数用于操作和处理字符串，例如查找、替换、分割和连接等。以下是一些常用的字符串处理函数及其作用和示例：

### 总结

`strings` 包提供了强大且多样的字符串处理功能，能有效地满足各种字符串操作需求。常用的字符串处理函数包括：

- `strings.Contains`：检查子字符串是否包含在另一个字符串中。
- `strings.Count`：返回子字符串在字符串中出现的次数。
- `strings.HasPrefix` 和 `strings.HasSuffix`：检查字符串是否以指定前缀或后缀开头或结尾。
- `strings.Index` 和 `strings.LastIndex`：返回子字符串在字符串中首次或最后一次出现的位置索引。
- `strings.Replace`：在字符串中将所有或指定次数的子字符串替换为另一个子字符串。
- `strings.Split`：将字符串按照指定的分隔符分割为切片。
- `strings.Join`：将一个字符串切片的所有元素连接为一个字符串，元素之间用指定的分隔符分隔。
- `strings.Trim` 和 `strings.TrimSpace`：删除字符串前后的指定字符或空白字符。
- `strings.ToUpper` 和 `strings.ToLower`：将字符串转换为全大写或全小写。
- `strings.Repeat`：返回一个重复 `count` 次的字符串。
- `strings.TrimPrefix` 和 `strings.TrimSuffix`：删除字符串前后的指定前缀或后缀。

### strings.Contains

#### 作用

检查子字符串是否包含在另一个字符串中。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello, world!"
	fmt.Println(strings.Contains(str, "world")) // 输出: true
	fmt.Println(strings.Contains(str, "World")) // 输出: false
}
```

### strings.Count

#### 作用

返回子字符串在字符串中出现的次数。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "banana"
	fmt.Println(strings.Count(str, "a")) // 输出: 3
	fmt.Println(strings.Count(str, "n")) // 输出: 2
}
```

### strings.HasPrefix和strings.HasSuffix

#### 作用

检查字符串是否以指定前缀或后缀开头或结尾。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "filename.txt"
	fmt.Println(strings.HasPrefix(str, "file")) // 输出: true
	fmt.Println(strings.HasSuffix(str, ".txt")) // 输出: true
}
```

### strings.Index和strings.LastIndex

#### 作用

返回子字符串在字符串中首次或最后一次出现的位置索引。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello, world!"
	fmt.Println(strings.Index(str, "o"))     // 输出: 4
	fmt.Println(strings.LastIndex(str, "o")) // 输出: 8
}
```

### strings.Replace

#### 作用

在字符串中将所有或指定次数的子字符串替换为另一个子字符串。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello, world! Hello, Go!"
	fmt.Println(strings.Replace(str, "Hello", "Hi", -1)) // 输出: Hi, world! Hi, Go!
	fmt.Println(strings.Replace(str, "Hello", "Hi", 1))  // 输出: Hi, world! Hello, Go!
}
```

### strings.Split

#### 作用

将字符串按照指定的分隔符分割为切片。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "a,b,c,d"
	parts := strings.Split(str, ",")
	fmt.Println(parts) // 输出: [a b c d]
}
```

### strings.Join

#### 作用

将一个字符串切片的所有元素连接为一个字符串，元素之间用指定的分隔符分隔。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	parts := []string{"a", "b", "c", "d"}
	str := strings.Join(parts, ",")
	fmt.Println(str) // 输出: a,b,c,d
}
```

### strings.Trim和strings.TrimSpace

#### 作用

删除字符串前后的指定字符或空白字符。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "  Hello, world!  "
	fmt.Println(strings.TrimSpace(str))   // 输出: Hello, world!
	fmt.Println(strings.Trim(str, " "))   // 输出: Hello, world!
	fmt.Println(strings.Trim(str, " H!")) // 输出: ello, world
}
```

### strings.ToUpper和strings.ToLower

#### 作用

将字符串转换为全大写或全小写。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello, World!"
	fmt.Println(strings.ToUpper(str)) // 输出: HELLO, WORLD!
	fmt.Println(strings.ToLower(str)) // 输出: hello, world!
}
```

### strings.Repeat

#### 作用

返回一个重复 `count` 次的字符串。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "abc"
	fmt.Println(strings.Repeat(str, 3)) // 输出: abcabcabc
}
```

### strings.TrimPrefix和strings.TrimSuffix

#### 作用

删除字符串前后的指定前缀或后缀。

#### 示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello, World!"
	fmt.Println(strings.TrimPrefix(str, "Hello"))  // 输出: , World!
	fmt.Println(strings.TrimSuffix(str, "World!")) // 输出: Hello, 
}
```

### 补充

strings库没有可以判断`一个字符串是否匹配一个正则表达式`的函数，在这个可以使用 `regexp` 包来判断一个字符串是否匹配一个正则表达式。

#### 总结

- `regexp.MatchString`：直接检查字符串是否匹配给定的正则表达式。
- `regexp.Compile` 和 `regexp.MustCompile`：编译正则表达式，返回一个可复用的 `Regexp` 对象。
- `Regexp.Match`：检查字节切片是否匹配编译好的正则表达式。

#### regexp.MatchString

##### 作用

检查字符串是否匹配给定的正则表达式。

##### 示例

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	pattern := `^Hello,.*!$`
	str := "Hello, World!"
	matched, err := regexp.MatchString(pattern, str)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Matched:", matched) // 输出: Matched: true
	}
}
```

#### regexp.Compile和regexp.MustCompile

##### 作用

编译一个正则表达式，返回一个 `Regexp` 对象，可以复用多次。

##### 示例

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	pattern := `^Hello,.*!$`
	re := regexp.MustCompile(pattern)

	str1 := "Hello, World!"
	str2 := "Goodbye, World!"

	fmt.Println("Matched str1:", re.MatchString(str1)) // 输出: Matched str1: true
	fmt.Println("Matched str2:", re.MatchString(str2)) // 输出: Matched str2: false
}
```

#### Regexp.Match

##### 作用

检查字节切片是否匹配编译好的正则表达式。

##### 示例

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	pattern := `^Hello,.*!$`
	re := regexp.MustCompile(pattern)

	str := []byte("Hello, World!")
	matched := re.Match(str)
	fmt.Println("Matched:", matched) // 输出: Matched: true
}
```
