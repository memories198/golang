## web socket编程

在 Go 语言中，WebSocket 编程通常使用第三方库来简化实现。最常用的库是 `gorilla/websocket`，它提供了一个简单的 API 来处理
WebSocket 连接。以下是一个基本的示例，演示如何使用 `gorilla/websocket` 创建一个 WebSocket 服务器和客户端。

### 安装 Gorilla WebSocket

首先，您需要安装 `gorilla/websocket` 包。可以使用以下命令进行安装：

```bash
go get -u github.com/gorilla/websocket
```

### 创建 WebSocket 服务器

以下是一个简单的 WebSocket 服务器示例：

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool {
		return true // 允许所有来源
	},
}

func handleConnection(w http.ResponseWriter, r *http.Request) {
	// 升级 HTTP 连接为 WebSocket 连接
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		fmt.Println("Error while upgrading connection:", err)
		return
	}
	defer conn.Close()

	fmt.Println("Client connected")

	for {
		// 读取消息
		messageType, msg, err := conn.ReadMessage()
		if err != nil {
			fmt.Println("Error while reading message:", err)
			break
		}
		fmt.Printf("Received: %s\n", msg)

		// 回送消息
		if err := conn.WriteMessage(messageType, msg); err != nil {
			fmt.Println("Error while writing message:", err)
			break
		}
	}
}

func main() {
	http.HandleFunc("/ws", handleConnection)

	fmt.Println("Starting WebSocket server on :8080...")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

### 代码解释

1. **导入包**：导入 `net/http` 和 `github.com/gorilla/websocket`。

2. **创建 Upgrader**：`websocket.Upgrader` 用于将 HTTP 连接升级为 WebSocket 连接。`CheckOrigin` 函数允许来自任何来源的连接。

3. **处理连接**：

- `handleConnection` 函数处理 WebSocket 连接。
- 使用 `upgrader.Upgrade` 将 HTTP 连接升级为 WebSocket 连接。
- 在一个循环中读取消息并回送相同的消息。

4. **启动服务器**：使用 `http.ListenAndServe` 启动服务器，监听在 8080 端口。

### 运行 WebSocket 服务器

将上述代码保存为 `main.go`，然后在终端中运行：

```bash
go run main.go
```

您应该会看到输出：

```
Starting WebSocket server on :8080...
```

### 创建 WebSocket 客户端

以下是一个简单的 WebSocket 客户端示例：

```go
package main

import (
	"fmt"
	"github.com/gorilla/websocket"
	"time"
)

func main() {
	// 连接到 WebSocket 服务器
	conn, _, err := websocket.DefaultDialer.Dial("ws://localhost:8080/ws", nil)
	if err != nil {
		fmt.Println("Error connecting to server:", err)
		return
	}
	defer conn.Close()

	// 发送消息
	for {
		message := "Hello, WebSocket!"
		err := conn.WriteMessage(websocket.TextMessage, []byte(message))
		if err != nil {
			fmt.Println("Error while sending message:", err)
			return
		}
		fmt.Printf("Sent: %s\n", message)

		// 等待一段时间
		time.Sleep(1 * time.Second)

		// 读取回送的消息
		_, msg, err := conn.ReadMessage()
		if err != nil {
			fmt.Println("Error while reading message:", err)
			return
		}
		fmt.Printf("Received: %s\n", msg)
	}
}
```

### 代码解释

1. **连接到服务器**：使用 `websocket.DefaultDialer.Dial` 连接到 WebSocket 服务器。

2. **发送和接收消息**：

- 在一个循环中发送消息并读取回送的消息。
- 使用 `conn.WriteMessage` 发送消息，使用 `conn.ReadMessage` 读取消息。

### 运行 WebSocket 客户端

将客户端代码保存为 `client.go`，然后在另一个终端中运行：

```bash
go run client.go
```

### 总结

通过以上示例，您可以看到如何使用 Go 语言和 `gorilla/websocket` 库创建 WebSocket
服务器和客户端。您可以根据需要扩展这些示例，添加更多的功能，例如处理不同类型的消息、管理多个连接等。WebSocket
是实现实时通信的强大工具，适用于聊天应用、实时数据更新等场景。
