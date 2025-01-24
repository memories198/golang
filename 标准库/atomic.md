## atomic篇

在 Go 语言中，`sync/atomic` 包提供了一组原子操作的函数，用于在并发环境中安全地操作基本数据类型。原子操作是指在执行过程中不会被其他线程或协程中断的操作，这对于实现高效的并发程序非常重要。

### 1. 基本用法

`sync/atomic` 包主要提供了对整型和指针的原子操作。以下是一些常用的原子操作函数：

- `atomic.AddInt32`
- `atomic.AddInt64`
- `atomic.LoadInt32`
- `atomic.LoadInt64`
- `atomic.StoreInt32`
- `atomic.StoreInt64`
- `atomic.CompareAndSwapInt32`
- `atomic.CompareAndSwapInt64`

### 2. 示例代码

以下是一个使用 `sync/atomic` 包的示例，展示如何安全地在多个 goroutine 中递增一个计数器。

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {
	var counter int64 // 使用 int64 类型的计数器

	var wg sync.WaitGroup
	numGoroutines := 10 // 启动的 goroutine 数量

	// 启动多个 goroutine
	for i := 0; i < numGoroutines; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for j := 0; j < 1000; j++ {
				atomic.AddInt64(&counter, 1) // 原子递增计数器
			}
		}()
	}

	// 等待所有 goroutine 完成
	wg.Wait()

	// 输出最终计数器的值
	fmt.Printf("Final counter value: %d\n", counter)
}
```

### 3. 代码解析

- **计数器**: 我们使用 `int64` 类型的 `counter` 变量来存储计数器的值。
- **WaitGroup**: 使用 `sync.WaitGroup` 来等待所有 goroutine 完成。
- **原子递增**: 在每个 goroutine 中，我们使用 `atomic.AddInt64(&counter, 1)` 来安全地递增计数器的值。
- **输出结果**: 最后，我们输出计数器的最终值。

### 4. 其他原子操作示例

#### 原子加载和存储

```go
package main

import "sync/atomic"

func main() {
	// 原子存储
	atomic.StoreInt64(&counter, 100)

	// 原子加载
	value := atomic.LoadInt64(&counter)
	fmt.Printf("Loaded counter value: %d\n", value)
}
```

#### 原子比较和交换

```go
var value int64 = 10
success := atomic.CompareAndSwapInt64(&value, 10, 20) // 如果 value 是 10，则将其设置为 20
if success {
fmt.Println("Value was 10, now set to 20")
} else {
fmt.Println("Value was not 10")
}
```

### 5. 注意事项

- 原子操作只适用于基本数据类型（如 `int32`, `int64`, `uint32`, `uint64`, `uintptr`, `*T` 等）。
- 对于复杂的数据结构（如切片、映射、结构体等），需要使用其他同步机制（如 `sync.Mutex` 或 `sync.RWMutex`）来确保线程安全。
- 原子操作的性能通常优于使用互斥锁，但在某些情况下，使用互斥锁可能更易于理解和维护。

### 总结

`sync/atomic` 包提供了一种高效的方式来在并发程序中安全地操作基本数据类型。通过使用原子操作，可以避免使用互斥锁，从而提高程序的性能。在编写并发程序时，合理使用原子操作可以帮助你实现更高效和安全的代码。

