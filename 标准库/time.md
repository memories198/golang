## time篇

### time.Now

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now()
	fmt.Printf("type: %T, value: %s", now, now)
}
```

### time.Since

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now()
	fmt.Printf("type: %T, value: %v\n", now, now)
	time.Sleep(5 * time.Second)

	duration := time.Since(now)
	fmt.Printf("type: %T, value: %v\n", duration, duration)
}
```

### time.Date

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	date := time.Date(2006, 1, 2, 15, 4, 5, 0, time.UTC)
	fmt.Printf("type: %T, value: %v", date, date)
}
```

### time.After

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	date := time.Date(2006, 1, 2, 15, 4, 5, 0, time.UTC)
	fmt.Printf("type: %T, value: %v\n", date, date)

	now := time.Now()
	fmt.Printf("now.After: %v\n", now.After(date))  // 判断now是否在date之后
	fmt.Printf("date.After: %v\n", date.After(now)) // 判断date是否在now之后
} 
```

### time.Before

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	date := time.Date(2006, 1, 2, 15, 4, 5, 0, time.UTC)
	fmt.Printf("type: %T, value: %v\n", date, date)

	now := time.Now()
	fmt.Printf("now.Before: %v\n", now.Before(date))  // 判断now是否在date之前
	fmt.Printf("date.Before: %v\n", date.Before(now)) // 判断date是否在now之前
}
```

### time.Add

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	date := time.Date(2006, 1, 2, 15, 4, 5, 0, time.UTC)
	fmt.Printf("type: %T, value: %v\n", date, date)

	newDate := date.Add(time.Hour)
	fmt.Printf("type: %T, value: %v\n", newDate, newDate)
}
```

### time.AddDate

```go

package main

import (
	"fmt"
	"time"
)

func main() {
	date := time.Date(2006, 1, 2, 15, 4, 5, 0, time.UTC)
	fmt.Printf("type: %T, value: %v\n", date, date)

	newDate := date.AddDate(10, 12, 30) // 会自动进位
	fmt.Printf("type: %T, value: %v\n", newDate, newDate)
}
```

### time.Layout

```go
package main

func main() {
	const (
		Layout = "01/02 03:04:05PM '06 -0700" // The reference time, in numerical order.

		ANSIC       = "Mon Jan _2 15:04:05 2006"
		UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
		RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
		RFC822      = "02 Jan 06 15:04 MST"
		RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
		RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
		RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
		RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
		RFC3339     = "2006-01-02T15:04:05Z07:00"
		RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
		Kitchen     = "3:04PM"
		// Handy time stamps.
		Stamp      = "Jan _2 15:04:05"
		StampMilli = "Jan _2 15:04:05.000"
		StampMicro = "Jan _2 15:04:05.000000"
		StampNano  = "Jan _2 15:04:05.000000000"
		DateTime   = "2006-01-02 15:04:05"
		DateOnly   = "2006-01-02"
		TimeOnly   = "15:04:05"
	)
}

```

在 Go 语言中，时间格式字符串用于定义时间的表示方式。每个格式字符串中的部分都有特定的含义，以下是你提供的时间格式的详细解释：

#### 1. `Layout = "01/02 03:04:05PM '06 -0700"`

- **`01`**: 月份（01-12）
- **`02`**: 日期（01-31）
- **`03`**: 小时（12小时制，01-12）
- **`04`**: 分钟（00-59）
- **`05`**: 秒（00-59）
- **`PM`**: 表示上午或下午（AM/PM）
- **`'06`**: 年的最后两位（06表示2006年）
- **`-0700`**: 时区偏移量（相对于 UTC 的偏移，例如 -0700 表示比 UTC 慢 7 小时）

#### 2. `ANSIC = "Mon Jan _2 15:04:05 2006"`

- **`Mon`**: 星期几（缩写，例：Mon, Tue）
- **`Jan`**: 月份（缩写，例：Jan, Feb）
- **`_2`**: 日期（2位数，前面有空格填充，例： 2 -> " 2"）
- **`15`**: 小时（24小时制，00-23）
- **`04`**: 分钟（00-59）
- **`05`**: 秒（00-59）
- **`2006`**: 完整的年份

#### 3. `UnixDate = "Mon Jan _2 15:04:05 MST 2006"`

- 与 `ANSIC` 类似，但最后的 `MST` 表示时区（例如：MST 表示山地标准时间）。

#### 4. `RubyDate = "Mon Jan 02 15:04:05 -0700 2006"`

- **`Mon`**: 星期几（缩写）
- **`Jan`**: 月份（缩写）
- **`02`**: 日期（2位数）
- **`15`**: 小时（24小时制）
- **`04`**: 分钟
- **`05`**: 秒
- **`-0700`**: 时区偏移量
- **`2006`**: 完整的年份

#### 5. `RFC822 = "02 Jan 06 15:04 MST"`

- **`02`**: 日期（2位数）
- **`Jan`**: 月份（缩写）
- **`06`**: 年的最后两位
- **`15`**: 小时（24小时制）
- **`04`**: 分钟
- **`MST`**: 时区（缩写）

#### 6. `RFC822Z = "02 Jan 06 15:04 -0700"`

- 与 `RFC822` 类似，但使用时区偏移量（例如：-0700）。

#### 7. `RFC850 = "Monday, 02-Jan-06 15:04:05 MST"`

- **`Monday`**: 星期几（完整名称）
- **`02-Jan-06`**: 日期（2位数，月份缩写，年最后两位）
- **`15:04:05`**: 时间（24小时制）
- **`MST`**: 时区（缩写）

#### 8. `RFC1123 = "Mon, 02 Jan 2006 15:04:05 MST"`

- **`Mon`**: 星期几（缩写）
- **`02`**: 日期（2位数）
- **`Jan`**: 月份（缩写）
- **`2006`**: 完整的年份
- **`15:04:05`**: 时间（24小时制）
- **`MST`**: 时区（缩写）

#### 9. `RFC1123Z = "Mon, 02 Jan 2006 15:04:05 -0700"`

- 与 `RFC1123` 类似，但使用时区偏移量（例如：-0700）。

#### 10. `RFC3339 = "2006-01-02T15:04:05Z07:00"`

- **`2006-01-02`**: 日期（完整年份-月份-日期）
- **`T`**: 日期和时间的分隔符
- **`15:04:05`**: 时间（24小时制）
- **`Z`**: 表示 UTC 时间（可选）
- **`07:00`**: 时区偏移量

#### 11. `RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"`

- 与 `RFC3339` 类似，但包含纳秒部分（999999999）。

#### 12. `Kitchen = "3:04PM"`

- **`3`**: 小时（12小时制）
- **`04`**: 分钟
- **`PM`**: 表示上午或下午（AM/PM）

#### 13. `Stamp = "Jan _2 15:04:05"`

- **`Jan`**: 月份（缩写）
- **`_2`**: 日期（2位数，前面有空格填充）
- **`15:04:05`**: 时间（24小时制）

#### 14. `StampMilli = "Jan _2 15:04:05.000"`

- 与 `Stamp` 类似，但包含毫秒部分（.000）。

#### 15. `StampMicro = "Jan _2 15:04:05.000000"`

- 与 `Stamp` 类似，但包含微秒部分（.000000）。

#### 16. `StampNano = "Jan _2 15:04:05.000000000"`

- 与 `Stamp` 类似，但包含纳秒部分（.000000000）。

#### 17. `DateTime = "2006-01-02 15:04:05"`

- **`2006-01-02`**: 日期（完整年份-月份-日期）
- **`15:04:05`**: 时间（24小时制）

#### 18. `DateOnly = "2006-01-02"`

- **`2006-01-02`**: 仅日期（完整年份-月份-日期）

#### 19. `TimeOnly = "15:04:05"`

- **`15:04:05`**: 仅时间（24小时制）

### time.Parse

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	const (
		Layout   = "01/02 03:04:05PM '06 -0700" // The reference time, in numerical order.
		UnixDate = "Mon Jan _2 15:04:05 MST 2006"
		DateOnly = "2006-01-02"
	)

	parse, err := time.Parse(Layout, "01/02 03:04:05PM '06 -0700")
	if err != nil {
		panic(err)
	}
	fmt.Printf("type: %T, value: %v\n", parse, parse.Format(DateOnly))

	parse, err = time.Parse(UnixDate, "Mon Jan 2 15:04:05 MST 2006")
	if err != nil {
		panic(err)
	}
	fmt.Printf("type: %T, value: %v\n", parse, parse.Format(DateOnly))
}
```

#### 练习题

设计一个函数输入一个int(0 <= minute < 60)类型，返回一个time.Durtion类型，功能是根据现在的时间计算出最快还需要多久可以到达指定分钟minute。
