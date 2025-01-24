## 使用 Go `net/http` 包开发 HTTP 应用程序

`net/http` 是 Go 的核心库之一，它提供了轻量级的工具来处理 HTTP 请求和响应，适用于构建 HTTP 服务器和客户端。以下是它的主要功能和使用场景。

---

### 1. 构建 HTTP 服务器

Go 的 `net/http` 包使得创建 HTTP 服务器变得非常简单。通过将路径与处理函数绑定，可以快速实现功能。

#### 示例代码：基本 HTTP 服务器

```go
package main

import (
	"fmt"
	"net/http"
)

// 处理请求的函数
func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	// 路由处理
	http.HandleFunc("/", helloHandler)

	// 启动服务器
	fmt.Println("Server started on :8080")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

**优化点**：
- 使用 `http.Handle` 注册静态文件服务，如用于托管 HTML 文件。

---

### 2. 实现 SSE（服务器发送事件）

SSE（Server-Sent Events）是一种轻量级的服务端推送技术，适用于实时更新需求。

#### 示例代码：SSE 实现

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func handleSSE(w http.ResponseWriter, r *http.Request) {
	// 设置 HTTP 头
	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")

	for {
		// 模拟实时数据推送
		fmt.Fprintf(w, "data: %s\n\n", time.Now().Format(time.RFC3339))
		w.(http.Flusher).Flush() // 立即刷新输出缓冲
		time.Sleep(5 * time.Second)
	}
}

func main() {
	http.HandleFunc("/events", handleSSE)
	fmt.Println("SSE Server started on :8080")
	http.ListenAndServe(":8080", nil)
}
```

**注意**：SSE 适合简单实时推送场景，但对于高并发或复杂场景，可以考虑 WebSocket。

---

### 3. 构建 HTTP 客户端

`net/http` 同样支持 HTTP 客户端，允许发送请求和解析响应。

#### 示例代码：发送 GET 请求

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("https://api.github.com")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error reading response:", err)
		return
	}

	fmt.Println("Status:", resp.Status)
	fmt.Println("Body:", string(body))
}
```

**扩展内容**：
- 自定义超时设置：通过 `http.Client` 控制超时。
- 添加请求头：通过 `http.NewRequest` 设置自定义头部。

---

### 4. 处理 POST 请求

在开发中，发送 JSON 或表单数据的 POST 请求很常见。

#### 示例代码：发送 JSON 数据

```go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	jsonData := []byte(`{"name":"Alice","age":25}`)

	resp, err := http.Post("https://httpbin.org/post", "application/json", bytes.NewBuffer(jsonData))
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error reading response:", err)
		return
	}

	fmt.Println("Status:", resp.Status)
	fmt.Println("Body:", string(body))
}
```

**补充**：
- 对于复杂的请求，可以使用 `http.NewRequest` 和 `http.Client.Do`。

---

### 5. 路由和中间件

默认的 `net/http` 包提供的路由功能较为基础，可以借助第三方库（如 `gorilla/mux` 或 `chi`）实现更强大的功能。

#### 示例代码：使用 `gorilla/mux`

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	name := params["name"]
	fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/hello/{name}", helloHandler)

	fmt.Println("Server started on :8080")
	http.ListenAndServe(":8080", r)
}
```

**扩展**：
- 使用中间件处理日志、认证和压缩。
- 集成 `net/http` 的 `Handler` 接口实现高级功能。

---

### 6. 请求参数处理

解析 URL 查询参数或表单数据是常见需求。

#### 示例代码：获取查询参数

```go
package main

import (
	"fmt"
	"net/http"
)

func queryHandler(w http.ResponseWriter, r *http.Request) {
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}
	fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
	http.HandleFunc("/greet", queryHandler)

	fmt.Println("Server started on :8080")
	http.ListenAndServe(":8080", nil)
}
```

**补充**：
- 使用 `r.FormValue` 处理 POST 表单数据。

---

### 7. 高级特性

#### 自定义超时设置

通过 `http.Server` 和 `http.Client` 配置超时可提高应用的稳定性。

```go
// 自定义 HTTP 服务器超时
server := &http.Server{
	Addr:         ":8080",
	ReadTimeout:  5 * time.Second,
	WriteTimeout: 10 * time.Second,
}
server.ListenAndServe()
```

#### 请求取消与超时

支持使用 `context.Context` 实现请求超时和取消。

---

### 总结

`net/http` 是一个强大且高效的 HTTP 工具包，能够满足大多数 Web 开发需求。通过配合第三方库和自定义配置，可以实现高效、灵活的 Web 应用开发。

**推荐阅读**：
1. 官方文档：[https://pkg.go.dev/net/http](https://pkg.go.dev/net/http)
2. 第三方库：
    - [gorilla/mux](https://github.com/gorilla/mux)
    - [go-chi/chi](https://github.com/go-chi/chi)