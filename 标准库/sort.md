## sort库篇

在Go语言中，标准库 `sort` 包提供了一系列用于排序的函数和方法，这些函数和方法可以对切片进行排序，包括整数、浮点数、字符串和自定义类型。以下是一些常用的排序函数及其作用和示例：

#### 总结

`sort` 包提供了多种用于排序的函数，能够对各种类型的切片进行排序，包括整数、浮点数、字符串和自定义类型。常用的排序函数包括：

- `sort.Ints`：对整数切片进行排序。
- `sort.Float64s`：对浮点数切片进行排序。
- `sort.Strings`：对字符串切片进行排序。
- `sort.IntsAreSorted`：检查整数切片是否已排序。
- `sort.Float64sAreSorted`：检查浮点数切片是否已排序。
- `sort.StringsAreSorted`：检查字符串切片是否已排序。
- `sort.Sort` 和 `sort.Reverse`：对实现了 `sort.Interface` 接口的自定义数据类型进行排序，`sort.Reverse` 用于将排序顺序反转。

#### sort.Ints

##### 作用

对整数切片进行排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	nums := []int{5, 2, 6, 3, 1, 4}
	sort.Ints(nums)
	fmt.Println(nums) // 输出: [1 2 3 4 5 6]
}
```

#### sort.Float64s

##### 作用

对浮点数切片进行排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	floats := []float64{5.1, 2.2, 6.3, 3.4, 1.5, 4.6}
	sort.Float64s(floats)
	fmt.Println(floats) // 输出: [1.5 2.2 3.4 4.6 5.1 6.3]
}
```

#### sort.Strings

##### 作用

对字符串切片进行排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	strs := []string{"banana", "apple", "cherry", "date"}
	sort.Strings(strs)
	fmt.Println(strs) // 输出: [apple banana cherry date]
}
```

#### sort.IntsAreSorted

##### 作用

检查整数切片是否已排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	nums := []int{1, 2, 3, 4, 5}
	fmt.Println(sort.IntsAreSorted(nums)) // 输出: true

	nums = []int{5, 4, 3, 2, 1}
	fmt.Println(sort.IntsAreSorted(nums)) // 输出: false
}
```

#### sort.Float64sAreSorted

##### 作用

检查浮点数切片是否已排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	floats := []float64{1.1, 2.2, 3.3, 4.4, 5.5}
	fmt.Println(sort.Float64sAreSorted(floats)) // 输出: true

	floats = []float64{5.5, 4.4, 3.3, 2.2, 1.1}
	fmt.Println(sort.Float64sAreSorted(floats)) // 输出: false
}
```

#### sort.StringsAreSorted

##### 作用

检查字符串切片是否已排序。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	strs := []string{"apple", "banana", "cherry", "date"}
	fmt.Println(sort.StringsAreSorted(strs)) // 输出: true

	strs = []string{"date", "cherry", "banana", "apple"}
	fmt.Println(sort.StringsAreSorted(strs)) // 输出: false
}
```

#### sort.Sort和sort.Reverse

##### 作用

对实现了`sort.Interface`接口的自定义数据类型进行排序，`sort.Reverse`用于将排序顺序反转。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

type Person struct {
	Name string
	Age  int
}

type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Alice", 30},
		{"Bob", 25},
		{"Charlie", 35},
	}

	// 按年龄排序
	sort.Sort(ByAge(people))
	fmt.Println("按年龄排序:", people)

	// 按年龄逆序排序
	sort.Sort(sort.Reverse(ByAge(people)))
	fmt.Println("按年龄逆序排序:", people)
}
```

#### sort.SearchInts

##### 作用

一个用于在有序（升序）整数切片中进行二分查找的函数。没有找到会返回给定整数值在该切片中的插入位置。

##### 示例

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	nums := []int{4, 3, 2, 1}
	sort.Ints(nums)
	i := sort.SearchInts(nums, 2)
	fmt.Println(i) // 1
	i = sort.SearchInts(nums, 12)
	fmt.Println(i) // 4
}
```

#### sort.Slice

```go
package main

import (
	"fmt"
	"sort"
)

// 定义一个结构体
type Person struct {
	Name string
	Age  int
}

func main() {
	// 创建一个 Person 结构体切片
	people := []Person{
		{"Alice", 30},
		{"Bob", 25},
		{"Charlie", 35},
	}

	// 按年龄升序排序
	sort.Slice(people, func(i, j int) bool {
		return people[i].Age < people[j].Age
	})

	fmt.Println("按年龄升序排序后的切片:")
	for _, person := range people {
		fmt.Printf("%s: %d\n", person.Name, person.Age)
	}

	// 按姓名降序排序
	sort.Slice(people, func(i, j int) bool {
		return people[i].Name > people[j].Name
	})

	fmt.Println("按姓名降序排序后的切片:")
	for _, person := range people {
		fmt.Printf("%s: %d\n", person.Name, person.Age)
	}
}
```
