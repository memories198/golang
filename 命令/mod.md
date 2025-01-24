### **`go mod`** - 模块管理

**常用子命令**:

- `go mod init`: 初始化模块，创建 `go.mod` 文件。
- `go mod tidy`: 清理无用依赖并补充缺失依赖。
- `go mod vendor`: 将依赖项复制到 `vendor` 目录。
- `go mod download`: 下载模块到本地缓存。

**示例**:

```bash
# 初始化模块
go mod init myproject

# 清理依赖
go mod tidy

# 查看所有依赖的列表
go list -m all
```

---
