### **`go get`** - 获取远程依赖

**用途**: 下载并安装指定的 Go 模块或包。

**示例**:

```bash
# 下载指定版本的包
go get example.com/pkg@v1.2.3
# 下载最新版本的包
go get example.com/pkg
```

**注意**: 从 Go 1.17 起，`go get` 专注于模块依赖管理。对于安装工具，请使用 `go install`。

---
