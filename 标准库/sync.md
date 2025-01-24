## sync篇

在 Go 语言中，`sync` 包提供了基本的同步原语，用于在并发程序中协调多个 goroutine 的执行。这个包包含了多种类型的同步工具，最常用的包括
`Mutex`、`RWMutex`、`WaitGroup`、`Once` 和 `Cond`。下面是对这些工具的详细介绍和示例。

### 1. `Mutex` 和 `RWMutex`

#### `Mutex`

`Mutex` 是一个互斥锁，用于保护共享资源，确保同一时间只有一个 goroutine 可以访问该资源。

**示例代码：**

```go
package main

import (
	"fmt"
	"sync"
)

var (
	mu      sync.Mutex
	counter int
)

func increment(wg *sync.WaitGroup) {
	defer wg.Done()
	mu.Lock()   // 加锁
	counter++   // 访问共享资源
	mu.Unlock() // 解锁
}

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go increment(&wg)
	}
	wg.Wait()
	fmt.Println("Final counter value:", counter)
}
```

#### `RWMutex`

`RWMutex` 是读写互斥锁，允许多个读操作并发进行，但写操作是独占的。

**示例代码：**

```go
package main

import (
	"fmt"
	"sync"
)

var (
	rwMu    sync.RWMutex
	counter int
)

func read(wg *sync.WaitGroup) {
	defer wg.Done()
	rwMu.RLock() // 加读锁
	fmt.Println("Current counter value:", counter)
	rwMu.RUnlock() // 解读锁
}

func write(wg *sync.WaitGroup) {
	defer wg.Done()
	rwMu.Lock() // 加写锁
	counter++
	rwMu.Unlock() // 解写锁
}

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go write(&wg)
	}
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go read(&wg)
	}
	wg.Wait()
	fmt.Println("Final counter value:", counter)
}
```

### 2. `WaitGroup`

`WaitGroup` 用于等待一组 goroutine 完成。你可以使用 `Add` 方法增加计数，使用 `Done` 方法减少计数，使用 `Wait` 方法阻塞直到计数为零。

**示例代码：**

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("Worker %d is working\n", id)
}

func main() {
	var wg sync.WaitGroup
	for i := 1; i <= 5; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}
	wg.Wait() // 等待所有 goroutine 完成
	fmt.Println("All workers completed.")
}
```

### 3. `Once`

`Once` 用于确保某个操作只执行一次，通常用于初始化操作。

**示例代码：**

```go
package main

import (
	"fmt"
	"sync"
)

var once sync.Once

func initialize() {
	fmt.Println("Initialization done.")
}

func worker() {
	once.Do(initialize) // 确保 initialize 只执行一次
	fmt.Println("Worker is working.")
}

func main() {
	for i := 0; i < 5; i++ {
		go worker()
	}
	// 等待一段时间以确保所有 goroutine 完成
	var input string
	fmt.Scanln(&input)
}
```

### 4. `Cond`

`Cond` 是条件变量，用于在 goroutine 之间进行通知和等待。它通常与 `Mutex` 一起使用。

**示例代码：**

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	mu    sync.Mutex
	cond  = sync.NewCond(&mu)
	ready = false
)

func worker() {
	mu.Lock()
	for !ready {
		cond.Wait() // 等待条件
	}
	fmt.Println("Worker is working.")
	mu.Unlock()
}

func main() {
	go worker()

	time.Sleep(1 * time.Second) // 模拟一些工作
	mu.Lock()
	ready = true
	cond.Signal() // 通知等待的 goroutine
	mu.Unlock()

	// 等待一段时间以确保所有 goroutine 完成
	var input string
	fmt.Scanln(&input)
}
```

### 总结

`sync` 包提供了多种工具来处理并发编程中的同步问题。通过使用 `Mutex`、`RWMutex`、`WaitGroup`、`Once` 和 `Cond`，你可以有效地管理
goroutine 之间的共享资源和协调它们的执行。选择合适的同步工具可以帮助你编写更安全和高效的并发程序。
