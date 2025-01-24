# gf框架篇
## 文档汇总
[官网](https://goframe.org/)
[详细文档](https://goframe.org/docs/web/start)
[简要文档](https://goframe.org/quick/install)

### 安装gf命令行工具

[下载连接](https://github.com/gogf/gf/releases)
下载后解压、配置环境变量

- **测试**
    ```shell
    gf --version
    ```

### 从配置文件中读取配置

配置文件路径: manifest/config/config.yaml

```yaml
hello: world
num: 999
map:
  k1: v1
  k2: v2
slice:
  - hello: world
  - num: 999
  - map:
      k1: v1
      k2: v2
```

读取配置示例

```go
package main

import (
	"fmt"
	_ "github.com/gogf/gf/contrib/drivers/mysql/v2"
	"github.com/gogf/gf/os/gctx"
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/os/gcfg"
	"github.com/gogf/gf/v2/os/glog"
	"github.com/pingcap/errors"
)

func main() {
	ctx := gctx.New()
	if err := g.Cfg().GetAdapter().(*gcfg.AdapterFile).SetPath("manifest/config"); err != nil {
		glog.Errorf(ctx, "fail to set config path, err: %+v\n", errors.Trace(err))
		return
	}
	g.Cfg().GetAdapter().(*gcfg.AdapterFile).SetFileName("config.yaml")

	str := g.Cfg().MustGet(ctx, "hello").String()
	fmt.Println(str)
	num := g.Cfg().MustGet(ctx, "num").Int()
	fmt.Println(num)

	for k, v := range g.Cfg().MustGet(ctx, "map").Map() {
		fmt.Printf("key: %v, value: %v\n", k, v)
	}

	for k, v := range g.Cfg().MustGet(ctx, "slice").Array() {
		fmt.Printf("key: %v, value: %v\n", k, v)
	}
}
```

### 连接数据库

```go
package main

import (
	_ "github.com/gogf/gf/contrib/drivers/mysql/v2"
	"github.com/gogf/gf/os/gctx"
	"github.com/gogf/gf/v2/database/gdb"
	"github.com/gogf/gf/v2/os/glog"
)

func main() {
	ctx := gctx.New()
	db, err := gdb.New(gdb.ConfigNode{
		Host:  "127.0.0.1",
		Port:  "3306",
		User:  "root",
		Pass:  "1234",
		Type:  "mysql",
		Extra: "loc=Local&parseTime=true",
	})
	if err != nil {
		panic(err)
	}
	tables, err := db.Tables(ctx, "<database>")
	if err != nil {
		panic(err)
	}
	for i, table := range tables {
		glog.Infof(ctx, "index: %d, table: %s", i, table)
	}
	return
}
```

### 生成表代码

编写hack/config.yaml,并创建internal目录

```yaml
gfcli:
  gen:
    dao:
      - link: "mysql:root:1234@tcp(127.0.0.1:3306)/<database>"
        type: "mysql"
        group: "<group>" # 自定义一个默认的数据源组名
        tables: "<table1>, <table2>"
```

### 配置数据源

除了在代码中new一个连接，还可以编写数据源配置文件，直接用配置文件创建连接。配置文件路径为manifest/config/config.yaml

```yaml
database:
  <group>:
    type: "mysql"
    link: "root:1234@tcp(127.0.0.1:3306)/<database>?loc=Local&parseTime=true"
```

```shell
package main

import (
	_ "github.com/gogf/gf/contrib/drivers/mysql/v2"
	"github.com/gogf/gf/os/gctx"
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/os/gcfg"
	"github.com/gogf/gf/v2/os/glog"
	"github.com/pingcap/errors"
)

func main() {
	ctx := gctx.New()
	if err := g.Cfg().GetAdapter().(*gcfg.AdapterFile).SetPath("manifest/config"); err != nil {
		glog.Errorf(ctx, "fail to set config path, err: %+v\n", errors.Trace(err))
		return
	}
	g.Cfg().GetAdapter().(*gcfg.AdapterFile).SetFileName("config.yaml")
	db := g.DB("test")
	tables, err := db.Tables(ctx, "libradb_shark")
	if err != nil {
		panic(err)
	}
	for i, table := range tables {
		glog.Infof(ctx, "index: %d, table: %s", i, table)
	}
}
```

### 开启打印sql查询日志的功能