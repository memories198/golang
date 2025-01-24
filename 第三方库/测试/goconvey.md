# goconvey
`goconvey` 是一个在 Go 语言中非常流行的测试框架，它提供了丰富的功能来支持你的测试需求，包括 Web UI 来实时查看测试结果。`goconvey` 提供了 BDD（行为驱动开发）风格的测试语法，使得测试代码更加易读和易维护。

## 安装

首先，你需要安装 `goconvey`。你可以通过 `go get` 命令来安装它：

```bash
go get github.com/smartystreets/goconvey/convey
```

## 基本用法

在 Go 项目中，你可以这样编写测试：

1. 导入 `goconvey` 包。
2. 使用 `Convey` 函数开始你的测试，这个函数接受一个测试的描述和一个函数，所有的测试逻辑都在这个函数内部编写。
3. 使用 `So` 函数来断言。`So` 接受一个实际值，一个比较器，和一个或多个期望值。

下面是一个简单的例子，假设我们有一个函数 `Add`，我们要测试这个函数：

```go
package example

// Add adds two integers and returns the result.
func Add(a, b int) int {
    return a + b
}
```

对应的测试代码如下：

```go
package example

import (
    "testing"
    . "github.com/smartystreets/goconvey/convey"
)

func TestAdd(t *testing.T) {
    Convey("Given two integer values", t, func() {
        a := 2
        b := 3

        Convey("When add the two values", func() {
            result := Add(a, b)

            Convey("The result should be the sum of the two values", func() {
                So(result, ShouldEqual, 5)
            })
        })
    })
}
```

## 代码解释

- `Convey` 的第一个参数是这个测试步骤的描述。
- 第二个参数 `t` 是 `*testing.T` 对象，它是 Go 测试的标准参数。
- 在 `Convey` 函数内部，你可以嵌套更多的 `Convey` 调用来组织你的测试逻辑。
- `So` 函数是断言，它检查第一个参数（实际值）是否符合第三个参数（期望值），通过第二个参数指定的比较器（这里是 `ShouldEqual`）。

## 运行测试

你可以通过标准的 `go test` 命令来运行测试：

```bash
go test
```

如果你想利用 `goconvey` 提供的 Web UI 功能，可以在你的项目目录下运行：

```bash
$GOPATH/bin/goconvey
```

这将启动一个 Web 服务器，默认在 `http://localhost:8080`，你可以在浏览器中查看实时的测试结果。
