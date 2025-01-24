## grpc编程

gRPC（Google Remote Procedure Call）是一种高性能的、通用的开源RPC框架，可以在分布式系统中实现服务之间的高效通信。以下是使用
gRPC 的步骤，包括如何定义服务、生成代码并在 Go 语言中实现客户端和服务端。

### 安装 gRPC 和 Protocol Buffers

首先，你需要安装 gRPC 和 Protocol Buffers（protobuf）的编译器。

```sh
#!/bin/sh
# 安装 gRPC 和 protobuf 插件
go get google.golang.org/grpc
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# 安装 protobuf 编译器
protoc_install_path=/usr/local/protoc
mkdir -p $protoc_install_path
cd $protoc_install_path
protoc_url=https://github.com/protocolbuffers/protobuf/releases/download/v27.3/protoc-27.3-linux-x86_64.zip
wget protoc_url
unset protoc_url
unzip -f protoc*.zip && rm -rf protoc*.zip
echo "export PATH=\$PATH:$protoc_install_path/bin" >> ~/.bashrc && source ~/.bashrc
unset protoc_install_path
```

### 项目目录结构示例

在 Go 语言中构建一个 gRPC 项目时，推荐采用一种清晰的目录结构，以便于维护和扩展。以下是一个典型的 gRPC 项目的目录结构示例：

```
your_project/
├── cmd/
│   ├── server/
│   │   └── main.go        # 服务端的入口文件
│   └── client/
│       └── main.go        # 客户端的入口文件
├── pb/
│   ├── hello.proto   # 定义 gRPC 服务和消息的 proto 文件
│   └── hello
│       ├── hello.pb.go   # 由 protoc 自动生成的 gRPC 代码
│       └── hello_grpc.pb.go # 由 protoc 自动生成的 gRPC 代码
├── internal/
│   ├── server/
│   │   └── server.go      # gRPC 服务的实现
│   └── client/
│       └── client.go      # gRPC 客户端的实现（可选）
├── config/
│   └── config.yaml        # 配置文件
├── go.mod                 # Go 模块文件
├── go.sum                 # Go 依赖文件
└── README.md              # 项目说明文件
```

### 目录结构说明

- **cmd/**: 存放项目的可执行文件的入口代码。通常会有 `server/` 和 `client/` 两个子目录，分别存放服务端和客户端的入口文件。

- **pb/**: 存放 `.proto` 文件及其生成的代码。为了保持代码整洁，建议将所有的 proto 文件和生成的 gRPC 代码放在这个目录下。

- **internal/**: 存放项目内部的业务逻辑实现。`server/` 目录下存放 gRPC 服务的具体实现，`client/`
  目录可以用于存放客户端的具体实现（如果需要）。

- **config/**: 存放项目的配置文件，例如 `config.yaml`。这个目录可以根据需要存放多个配置文件，支持不同的环境（如开发、测试、生产）。

- **go.mod** 和 **go.sum**: Go 的依赖管理文件。

- **README.md**: 项目的说明文件，描述项目的目的、如何构建和运行等信息。

### 定义 gRPC 服务

创建一个 `.proto` 文件，定义你的 gRPC 服务和消息类型。例如，创建 `helloworld.proto`：

```proto
syntax = "proto3";
//生成的路径，生成在当前路径的hello目录中，hell没有时会自动创建
option go_package = "./hello";

// 定义请求消息
message HelloRequest {
  string name = 1;
}

// 定义响应消息
message HelloReply {
  string message = 1;
}

// 定义服务
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}
```

### 编译 .proto 文件

使用 `protoc` 编译 `.proto` 文件，生成 Go 代码：

```sh
#可以分开生成
protoc --go_out=. hello.proto
protoc --go-grpc_out=. hello.proto 
#也可以合并生成
protoc --go_out=. --go-grpc_out=. hello.proto
```

### 实现 gRPC 服务端

在 Go 中实现 gRPC 服务端：

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"log"
	"net"
)

type server struct {
	hello.UnimplementedGreeterServer
}

func (s *server) SayHello(ctx context.Context, in *hello.HelloRequest) (*hello.HelloReply, error) {
	return &hello.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	hello.RegisterGreeterServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

### 实现 gRPC 客户端

在 Go 中实现 gRPC 客户端：

```go
package main

import (
	"context"
	"google.golang.org/grpc/credentials/insecure"
	"log"
	"time"

	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.NewClient("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := hello.NewGreeterClient(conn)

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	r, err := c.SayHello(ctx, &hello.HelloRequest{Name: "world"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
}
```

### 为grpc服务使用TLS证书

#### 生成证书

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/x509"
	"crypto/x509/pkix"
	"encoding/pem"
	"fmt"
	"math/big"
	"os"
	"time"
)

func main() {
	// 生成私钥
	priv, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		fmt.Println("Failed to generate private key:", err)
		return
	}

	// 创建证书模板
	template := x509.Certificate{
		SerialNumber: big.NewInt(1),
		Subject: pkix.Name{
			Organization: []string{"Memory's Organization"},
		},
		NotBefore:             time.Now(),
		NotAfter:              time.Now().Add(365 * 24 * time.Hour), // 一年有效期
		KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
		ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
		BasicConstraintsValid: true,
		//设置适用的域名，不设置会无法验证
		DNSNames: []string{
			"localhost",
		},
	}

	// 使用私钥创建自签名证书
	derBytes, err := x509.CreateCertificate(rand.Reader, &template, &template, &priv.PublicKey, priv)
	if err != nil {
		fmt.Println("Failed to create certificate:", err)
		return
	}

	// 保存证书到文件
	certOut, err := os.Create("cert.pem")
	if err != nil {
		fmt.Println("Failed to open cert.pem for writing:", err)
		return
	}
	pem.Encode(certOut, &pem.Block{Type: "CERTIFICATE", Bytes: derBytes})
	certOut.Close()
	fmt.Println("Written cert.pem")

	// 保存私钥到文件
	keyOut, err := os.Create("key.pem")
	if err != nil {
		fmt.Println("Failed to open key.pem for writing:", err)
		return
	}
	pem.Encode(keyOut, pemBlockForKey(priv))
	keyOut.Close()
	fmt.Println("Written key.pem")
}

// pemBlockForKey 返回PEM编码的私钥
func pemBlockForKey(priv *ecdsa.PrivateKey) *pem.Block {
	b, err := x509.MarshalECPrivateKey(priv)
	if err != nil {
		fmt.Println("Unable to marshal ECDSA private key:", err)
		os.Exit(2)
	}

	return &pem.Block{Type: "EC PRIVATE KEY", Bytes: b}
}

```

#### 在服务端使用证书

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"log"
	"net"
	"net/http"
	"test/pb/hello"
)

type server struct {
	hello.UnimplementedGreeterServer
}

func (s *server) SayHello(ctx context.Context, in *hello.HelloRequest) (*hello.HelloReply, error) {
	return &hello.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
	// 创建 gRPC 服务器
	grpcServer := grpc.NewServer()
	hello.RegisterGreeterServer(grpcServer, &server{})

	lis, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
	//err = grpcServer.Serve(lis) //没有使用TLS协议
	//if err != nil {
	//	log.Fatalf("failed to serve: %v", err)
	//}
	err = http.ServeTLS(lis, grpcServer, "certificate/cert.pem", "certificate/key.pem")
	if err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

```go
package main

import (
	"context"
	"crypto/tls"
	"fmt"
	"google.golang.org/grpc"
	"log"
	"net/http"
	"test/pb/hello"
)

type server struct {
	hello.UnimplementedGreeterServer
}

func (s *server) SayHello(ctx context.Context, in *hello.HelloRequest) (*hello.HelloReply, error) {
	return &hello.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
	// 创建 gRPC 服务器
	grpcServer := grpc.NewServer()
	hello.RegisterGreeterServer(grpcServer, &server{})

	// 使用 HTTP/3 运行 gRPC 服务器
	//http3Server := &http3.Server{
	//	Addr:    "localhost:8080",
	//	Handler: grpcServer,
	//}
	//if err := http3Server.ListenAndServeTLS("certificate/cert.pem", "certificate/key.pem"); err != nil {
	//	log.Fatalf("failed to serve: %v", err)
	//}
	//lis, err := net.Listen("tcp", ":8080")
	//if err != nil {
	//	log.Fatalf("failed to serve: %v", err)
	//}
	//err = grpcServer.Serve(lis)
	//if err != nil {
	//	log.Fatalf("failed to serve: %v", err)
	//}
	//err = http.ServeTLS(lis, grpcServer, "certificate/cert.pem", "certificate/key.pem")
	//if err != nil {
	//	log.Fatalf("failed to serve: %v", err)
	//}

	pair, err := tls.LoadX509KeyPair("certificate/cert.pem", "certificate/key.pem")
	if err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
	server1 := &http.Server{
		Handler: grpcServer,
		Addr:    ":8080",
		TLSConfig: &tls.Config{
			Certificates: []tls.Certificate{pair},
			ClientAuth:   tls.NoClientCert,
			//ClientAuth: tls.RequireAnyClientCert,
			MinVersion: tls.VersionTLS10,
			MaxVersion: tls.VersionTLS13,
			//ServerName: "localhost",
		},
	}
	fmt.Println(server1.ListenAndServeTLS("", "").Error())
}


```

#### 客户端代码

```go
package main

import (
	"context"
	"google.golang.org/grpc/credentials"
	"log"
	"test/pb/hello"
	"time"

	"google.golang.org/grpc"
)

func main() {
	creds, err := credentials.NewClientTLSFromFile("certificate/cert.pem", "")
	if err != nil {
		log.Fatalf("failed to load client TLS credentials: %v", err)
	}
	conn, err := grpc.NewClient("localhost:8080", grpc.WithTransportCredentials(creds))
	//conn, err := grpc.NewClient("localhost:8080", grpc.WithTransportCredentials(insecure.NewCredentials()))

	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	c := hello.NewGreeterClient(conn)

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	r, err := c.SayHello(ctx, &hello.HelloRequest{Name: "world"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
}

```