## 使用gin框架进行web开发
### 下载
```shell
go get -u github.com/gin-gonic/gin
```
### 静态文件服务

```go
r.Static("<url>", <path>)

router.Static("/assets", "./assets")
router.StaticFS("/more_static", http.Dir("my_file_system"))
router.StaticFile("/favicon.ico", "./resources/favicon.ico")
```

### 加载html模板

**以匹配的方式**

```go
r.LoadHTMLGlob("templates/*.html", "tempaltes/index.html")
```

**以指定的方式**

```go
r.LoadHTMLFiles("tempaltes/index.html", "templates/login.html")
```

html中调用go文件中的函数：以now作为参数调用formatAsDate函数并在此处显示返回值

```go
router.SetFuncMap(template.FuncMap{
"formatAsDate": formatAsDate, // formatAsDate是一个go函数
})
```

```html
{{.now | formatAsDate}} <!--使用进行占位和调用go函数-->
```

### 服务器推送

服务器推送（Server Push）是一种Web性能优化技术，它允许服务器在客户端请求之前主动将额外的资源推送给客户端。这样，客户端在解析HTML页面时就已经收到了这些资源，
无需再发送额外的请求。这样可以减少延迟，提高页面加载速度和性能。

```go
pusher := c.Writer.Pusher()
pusher.Push("/templates/index.html", nil)
```

### 使用 JSONP 向不同域的服务器请求数据。如果查询参数存在回调，则将回调添加到响应体中。

```go
// /JSONP?callback=x

// 将输出：x({\"foo\":\"bar\"})

c.JSONP(http.StatusOK, gin.H{"foo": "bar",})
```

### Multipart/Urlencoded 绑定

```go
package main

var form struct {
	User string `form:"user" binding:"required"`

	Password string `form:"password" binding:"required"`
}

func main() {
	if c.ShouldBind(&form) == nil {

		if form.User == "user" && form.Password == "password" {

			c.JSON(200, gin.H{"status": "you are logged in"})

		} else {

			c.JSON(401, gin.H{"status": "unauthorized"})

		}

	}
}

```

### JSON 使用 unicode 替换特殊 HTML 字符

例如 < 变为 \ u003c。如果要按字面对这些字符进行编码，则可以使用 PureJSON。

```go
c.JSON(200, gin.H{

"html":  `<b>"Hello, world!"</b>`,

}) //会对转义字符进行替换

c.PureJSON(200, gin.H{

"html": `"Hello, world!"`,

}) //全是明文
```

### Query 和 post form

示例url: http://localhost/post?id=12345&page=1998

#### 查询url中的字段

```go
id := c.Query("id")

page := c.DefaultQuery("page", "0")
```

#### 查询form中的字段

（字段名是form中的name的值）

```go
name := c.PostForm("name")

message := c.PostForm("message")
```

### gin.New和gin.Default的区别

```go
gin.New()

//默认没有使用logger和recover中间件
router := gin.New()
router.Use(gin.Logger())
router.Use(gin.Recovery())
```

### XML/JSON/YAML/ProtoBuf 渲染

#### JSON渲染

```go
package main

import "net/http"

func main() {
	c.JSON(http.StatusOK,

		gin.H{"message": "hey",

			"status": http.StatusOK,
		})

	var msg struct {
		Name string `json:"user"`

		Message string

		Number int
	}

	msg.Name = "Lena"

	msg.Message = "hey"

	msg.Number = 123

	// 注意 msg.Name 在 JSON 中变成了 "user"

	// 将输出：{"user": "Lena", "Message": "hey", "Number": 123}

	c.JSON(http.StatusOK, msg)
}
```

#### XML渲染

```go
package main

import "net/http"

func main() {
	c.XML(http.StatusOK, gin.H{
		"message": "hey",
		"status":  http.StatusOK,
	})
}

```

#### YAML渲染

```go
package main

import "net/http"

func main() {
	c.YAML(http.StatusOK, gin.H{
		"message": "hey",
		"status":  http.StatusOK,
	})
}
```

ProtoBuf渲染

```go
package main

import "net/http"

func main() {
	reps := []int64{int64(1), int64(2)}

	label := "test"

	// protobuf 的具体定义写在 testdata/protoexample 文件中。

	data := &protoexample.Test{

		Label: &label,

		Reps: reps,
	}

	// 请注意，数据在响应中变为二进制数据

	// 将输出被 protoexample.Test protobuf 序列化了的数据

	c.ProtoBuf(http.StatusOK, data)
}
```

### 上传文件

#### 上传单文件

```go
// 为 multipart forms 设置较低的内存限制 (默认是 32 MiB)

router.MaxMultipartMemory = 1 << 30 // 1 GiB
file, err := c.FormFile("file")
c.SaveUploadedFile(file, "<保存路径>/"+file.Filename)
```

#### 上传多文件

```go
package main

func main() {

	form, _ := c.MultipartForm()

	files := form.File["upload[]"]

	for _, file := range files {

		c.SaveUploadedFile(file, dst)

	}
}

```

#### 从 reader 读取数据

```go
//发起了一个HTTP GET请求，获取了指定URL的数据。
response, err := http.Get("https://<url>/color.png"):
if err != nil || response.StatusCode != http.StatusOK {
c.Status(http.StatusServiceUnavailable)
return
}

```

```go
reader := response.Body
contentLength := response.ContentLength
contentType := response.Header.Get("Content-Type")
//创建了一个额外的响应头，指定了返回的文件名为"gopher.png"。
extraHeaders := map[string]string{
"Content-Disposition": `attachment; filename="gopher.png"`,
//attachment 表示浏览器应该提示用户下载响应的内容
//inline在浏览器中显示它
}
//以附件形式返回给客户端，状态码为200
c.DataFromReader(http.StatusOK, contentLength, contentType, reader, extraHeaders)
```

### 优雅地重启或停止服务器

```go
package main

import (
	"context"
	"github.com/gin-gonic/gin"
	"log"
	"net/http"
	"os"
	"os/signal"
	"time"
)

func main() {
	router := gin.Default()
	srv := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}

	go func() {
		// 服务连接
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("listen: %s\n", err)
		}
	}()

	// 等待中断信号
	quit := make(chan os.Signal)
	signal.Notify(quit, os.Interrupt)
	<-quit

	//设置 5 秒的超时时间
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	srv.Shutdown(ctx) //关闭服务器
	router.Run(":80")
}

```

### 使用 BasicAuth 中间件进行身份验证

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/admin", gin.BasicAuth(map[string]string{
		//账号和密码
		"foo":    "bar",
		"austin": "1234",
		"lena":   "hello2",
		"manu":   "4321",
	}), func(c *gin.Context) {
		// 获取用户，它是由 BasicAuth 中间件设置的
		user := c.MustGet(gin.AuthUserKey).(string)
		fmt.Println(user)
	})
	r.Run(":80")
}

```

### 使用 HTTP 方法

```go
router.GET("/someGet", getting)
router.POST("/somePost", posting)
router.PUT("/somePut", putting)
router.DELETE("/someDelete", deleting)
router.PATCH("/somePatch", patching)
router.HEAD("/someHead", head)
router.OPTIONS("/someOptions", options)
```

### 使用中间件

```go
package main

import (
	"github.com/gin-gonic/gin"
	"io"
	"os"
)

func main() {
	r := gin.Default()
	//打开日志文件
	logFile, err := os.OpenFile("./log.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		panic(err)
	}
	//将gin产生的日志信息写入终端和log.txt中
	gin.DefaultWriter = io.MultiWriter(logFile, os.Stdout)
	// 添加全局中间件，即日志中间件。该中间件将日志写入gin.DefaultWriter，默认情况下是os.Stdout如果有 panic 的话，会写入 500。
	r.Use(gin.Logger())
	//添加全局中间件，即恢复中间件。该中间件会在有panic时进行恢复，并将状态码设置为500。
	r.Use(gin.Recovery())
	//为特定的路由/benchmark添加中间件MyBenchLogger()，并指定处理函数benchEndpoint。
	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)
	//为路由组使用中间件
	authorized := r.Group("/")
	authorized.Use(AuthRequired())
	{
		authorized.POST("/login", loginEndpoint)
		authorized.POST("/submit", submitEndpoint)
		authorized.POST("/read", readEndpoint)

		// 嵌套路由组
		testing := authorized.Group("testing")
		testing.GET("/analytics", analyticsEndpoint)
	}
	r.Run(":80")
}

```

### 只绑定 url 查询字符串

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
)

func main() {
	r := gin.Default()
	// 示例 URL：http://localhost/?Name=JohnDoe&Address=123MainSt
	r.GET("/", func(c *gin.Context) {
		person := &struct {
			Name    string
			Address string
		}{}
		if c.ShouldBindQuery(&person) == nil {
			log.Println(person.Name)
			log.Println(person.Address)
		}
	})
	r.Run(":80")
}



```

### 在中间件中使用 Goroutine

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
	"time"
)

func main() {
	r := gin.Default()
	r.GET("/long_async", func(c *gin.Context) {
		// 创建在 goroutine 中使用的副本
		cCp := c.Copy()
		go func() {
			// 用 time.Sleep() 模拟一个长任务。
			time.Sleep(5 * time.Second)
			// 请注意使用的是复制的上下文 "cCp"，这一点很重要
			log.Println("Done! in path " + cCp.Request.URL.Path)
		}()
	})

	r.GET("/long_sync", func(c *gin.Context) {
		// 用 time.Sleep() 模拟一个长任务。
		time.Sleep(5 * time.Second)
		// 因为没有使用 goroutine，不需要拷贝上下文
		log.Println("Done! in path " + c.Request.URL.Path)
	})

	r.Run(":80")
}

```

### 多模板

Gin 默认允许只使用一个 html 模板。 查看多模板渲染 以使用 go 1.6 block template 等功能。

```html
<!--base.html-->
<div>
    {{template "content-index" .}}
</div>
```

```html
<!--index.html-->
{{define "content-index"}}
<h1>Welcome to the Index Page!</h1>
{{end}}
```

```go
package main

import "github.com/gin-gonic/gin"

//定义一个多模板渲染函数
func createMyRender() multitemplate.Renderer {
	r := multitemplate.NewRenderer()
	r.AddFromFiles("index", "templates/base.html", "templates/index.html")
	return r
}

func main() {
	router := gin.Default()
	router.HTMLRender = createMyRender()
	router.GET("/", func(c *gin.Context) {
		c.HTML(200, "index", gin.H{
			"title": "Html5 Template Engine",
		})
	})
	router.Run(":80")
}
```

### 如何记录日志

```go
package main

import (
	"github.com/gin-gonic/gin"
	"io"
	"os"
)

func main() {
	r := gin.New()
	// 禁用控制台颜色，将日志写入文件时不需要控制台颜色。
	gin.DisableConsoleColor()

	// 记录到文件。
	f, _ := os.Create("gin.log")
	gin.DefaultWriter = io.MultiWriter(f)

	// 如果需要同时将日志写入文件和控制台，请使用以下代码。
	// gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
	r.Run(":80")

}

```

### 定义路由日志的格式

```
//默认的路由注册格式
[GIN-debug] POST /foo --> main.main.func1 (3 handlers)
[GIN-debug] GET /bar --> main.main.func2 (3 handlers)
[GIN-debug] GET /status --> main.main.func3 (3 handlers)
//使用 gin.DebugPrintRouteFunc 指定格式
gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
log.Printf("endpoint %v %v %v %v\n", httpMethod, absolutePath, handlerName, nuHandlers)
}
//修改后的路由注册格式
2023/12/28 11:16:30 endpoint POST /foo main.main.func2 3
2023/12/28 11:16:30 endpoint GET /bar main.main.func3 3
2023/12/28 11:16:30 endpoint GET /status main.main.func4 3
```

### 请求访问格式

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

func logFormat(param gin.LogFormatterParams) string {
	// 自定义的格式
	return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
		param.ClientIP,
		param.TimeStamp.Format(time.DateTime),
		param.Method,
		param.Path,
		param.Request.Proto,
		param.StatusCode,
		param.Latency,
		param.Request.UserAgent(),
		param.ErrorMessage,
	)
}

func main() {
	r := gin.New()
	r.Use(gin.LoggerWithFormatter(logFormat))
	r.GET("/ping", func(c *gin.Context) {
		c.String(http.StatusOK, "pong")
	})
	r.Run(":80")

}

```

### 将 request body 绑定到不同的结构体中

```go
package main

type formA struct {
	Foo string `json:"foo" xml:"foo" binding:"required"`
}

type formB struct {
	Bar string `json:"bar" xml:"bar" binding:"required"`
}

```

一次绑定，可以使用 c.ShouldBind

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func SomeHandler(c *gin.Context) {
	objA := formA{}
	objB := formB{}
	// c.ShouldBind 使用了 c.Request.Body，不可重用。前面的绑定成功后将不会在绑定后面的(form表单请求）
	if errA := c.ShouldBind(&objA); errA == nil {
		c.String(http.StatusOK, `the body should be formA`)
		fmt.Println("`the body should be formA`", objA.Foo)
	} else if errB := c.ShouldBind(&objB); errB == nil {
		c.String(http.StatusOK, `the body should be formB`)
		fmt.Println("`the body should be formB`", objB.Bar)
	} else {
	}
}
func main() {
	r := gin.Default()
	r.GET("/", SomeHandler)
	r.Run()
}

```

多次绑定，可以使用 c.ShouldBindBodyWith

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"net/http"
)

func SomeHandler(c *gin.Context) {
	objA := formA{}
	objB := formB{}
	// c.ShouldBind 使用了 c.Request.Body，不可重用。
	if errA := c.ShouldBindBodyWith(&objA, binding.JSON); errA == nil {
		c.String(http.StatusOK, `the body should be formA`)
	}
	if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
		c.String(http.StatusOK, `the body should be formB`)

	} else {
		fmt.Println("binding error")
	}
	fmt.Println("`the body should be formA`", objA.Foo)
	fmt.Println("`the body should be formB`", objB.Bar)
}

```

c.ShouldBindBodyWith 会在绑定之前将 body 存储到上下文中。 这会对性能造成轻微影响，如果调用一次就能完成绑定的话，那就不要用这个方法。
只有某些格式需要此功能，如 JSON, XML, MsgPack, ProtoBuf。 对于其他格式, 如 Query, Form, FormPost, FormMultipart 可以多次调用
c.ShouldBind() 而不会造成任任何性能损失

ShouldBindJSON、
ShouldBindXML、
ShouldBindUri、
Bind

### 控制日志输出颜色

#### 日志颜色化

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 强制日志颜色化
	gin.ForceConsoleColor()

	// 用默认中间件创建一个 gin 路由:
	// 日志和恢复（无崩溃）中间件
	router := gin.Default()

	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	router.Run(":8080")

}
```

#### 禁止日志颜色化

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 禁止日志的颜色
	gin.DisableConsoleColor()

	// 用默认中间件创建一个 gin 路由:
	// 日志和恢复（无崩溃）中间件
	router := gin.Default()

	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	router.Run(":8080")

}
```

### 支持 Let's Encrypt

```go
package main

import (
	"github.com/gin-gonic/autotls"
	"github.com/gin-gonic/gin"
	"log"
)

func main() {
	r := gin.Default()

	// Ping handler
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	log.Fatal(autotls.Run(r, "example1.com", "example2.com"))
}

```

#### 自定义 autocert manager 示例

```go
package main

import (
	"github.com/gin-gonic/autotls"
	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/acme/autocert"
	"log"
)

func main() {
	r := gin.Default()

	// Ping handler
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	m := autocert.Manager{
		Prompt:     autocert.AcceptTOS,
		HostPolicy: autocert.HostWhitelist("example1.com", "example2.com"),
		Cache:      autocert.DirCache("/var/www/.cache"),
	}

	log.Fatal(autotls.RunWithManager(r, &m))
}

```

### 映射查询字符串或表单参数

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {
		ids := c.QueryMap("ids")        //url中的参数ids
		names := c.PostFormMap("names") //form中的参数

		fmt.Printf("ids: %v; names: %v", ids, names)

	})

	router.Run(":80")
}

```

### 查询字符串参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	// 使用现有的基础请求对象解析查询字符串参数。
	// 示例 URL： /welcome?firstname=Jane&lastname=Doe
	router.GET("/welcome", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")
		lastname := c.Query("lastname")
		// c.Request.URL.Query().Get("lastname") 的一种快捷方式

		c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
	})

	router.Run(":80")
}

```

### 绑定 HTML 复选框

```go
package main

import "github.com/gin-gonic/gin"

type myForm struct {
	Colors []string `form:"colors[]"`
}

func formHandler(c *gin.Context) {
	var fakeForm myForm
	c.ShouldBind(&fakeForm)
	c.JSON(200, gin.H{"color": fakeForm.Colors})
}

func main() {
	router := gin.Default()

	router.GET("/", formHandler)
	router.Run(":80")
}

```

#### 绑定 Uri

```go

package main

import "github.com/gin-gonic/gin"

type Person struct {
	ID   string `uri:"id" binding:"required,uuid"`
	Name string `uri:"name" binding:"required"`
}

func main() {
	router := gin.Default()
	router.GET("/:name/:id", func(c *gin.Context) {
		var person Person
		if err := c.ShouldBindUri(&person); err != nil {
			c.JSON(400, gin.H{"msg": err.Error()})
			return
		}
		c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
	})
	router.Run(":80")
}
```

测试:

```
$ curl -v localhost:8088/thinkerou/987fbc97-4bed-5078-9f07-9141ba07c9f3
$ curl -v localhost:8088/thinkerou/not-uuid
```

### 绑定表单数据至自定义结构体

```go
package main

import "github.com/gin-gonic/gin"

type StructA struct {
	FieldA string `form:"field_a"`
}

type StructB struct {
	NestedStruct StructA
	FieldB       string `form:"field_b"`
}

func GetDataB(c *gin.Context) {
	var b StructB
	c.Bind(&b)
	c.JSON(200, gin.H{
		"a": b.NestedStruct,
		"b": b.FieldB,
	})
}
func main() {
	r := gin.Default()
	r.GET("/getb", GetDataB)
	r.Run(":80")
}
```

### 自定义 HTTP 配置

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

func main() {
	router := gin.Default()

	s := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	s.ListenAndServe()
}

```

### 自定义中间件

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
	"time"
)

func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()

		// 设置 example 变量
		c.Set("example", "12345")

		// 请求前

		c.Next()

		// 请求后
		latency := time.Since(t)
		log.Print(latency)

		// 获取发送的 status
		status := c.Writer.Status()
		log.Println(status)
	}
}
func main() {
	r := gin.New()
	r.Use(Logger())

	r.GET("/test", func(c *gin.Context) {
		example := c.MustGet("example").(string)

		// 打印："12345"
		log.Println(example)
	})

	// 监听并在 0.0.0.0:8080 上启动服务
	r.Run(":8080")
}
```

### 自定义日志文件

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"time"
)

func main() {
	router := gin.New()
	// LoggerWithFormatter 中间件会写入日志到 gin.DefaultWriter
	// 默认 gin.DefaultWriter = os.Stdout
	router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
		// 你的自定义格式
		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
			param.ClientIP,
			param.TimeStamp.Format(time.RFC1123),
			param.Method,
			param.Path,
			param.Request.Proto,
			param.StatusCode,
			param.Latency,
			param.Request.UserAgent(),
			param.ErrorMessage,
		)
	}))
	router.Use(gin.Recovery())
	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})
	router.Run(":8080")
}

```

### 自定义验证器

```go
package main

import (
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/validator/v10"
)

// Booking 包含绑定和验证的数据。
type Booking struct {
	CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
	CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn,bookabledate" time_format:"2006-01-02"`
}

var bookableDate validator.Func = func(fl validator.FieldLevel) bool {
	date, ok := fl.Field().Interface().(time.Time)
	if ok {
		today := time.Now()
		if today.After(date) {
			return false
		}
	}
	return true
}

func main() {
	route := gin.Default()

	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterValidation("bookabledate", bookableDate)
	}

	route.GET("/bookable", getBookable)
	route.Run(":8085")
}

func getBookable(c *gin.Context) {
	var b Booking
	if err := c.ShouldBindWith(&b, binding.Query); err == nil {
		c.JSON(http.StatusOK, gin.H{"message": "Booking dates are valid!"})
	} else {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	}
}



```

```
//两个日期需在当前日期之后
localhost:8085/bookable?check_in=2024-01-11&check_out=2024-01-31
```

### 设置和获取 Cookie

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	router.GET("/cookie", func(c *gin.Context) {
		//尝试获取cookie
		cookie, err := c.Cookie("gin_cookie")

		if err != nil { //获取失败就设置cookie
			cookie = "NotSet"
			c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
		}

		fmt.Printf("Cookie value: %s \n", cookie)
	})

	router.Run(":80")
}

```

### 路由参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()
	// 此 handler 将匹配 /user/john 但不会匹配 /user/ 或者 /user
	router.GET("/user/:name", func(c *gin.Context) {
		name := c.Param("name")
		c.String(http.StatusOK, "Hello %s", name)
	})

	// 此 handler 将匹配 /user/john/ 和 /user/john/send
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		message := name + " is " + action
		c.String(http.StatusOK, message)
	})
	router.Run(":80")
}


```

### 路由组

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	// 简单的路由组: v1
	v1 := router.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.POST("/read", readEndpoint)
	}

	// 简单的路由组: v2
	v2 := router.Group("/v2")
	{
		v2.POST("/login", loginEndpoint)
		v2.POST("/submit", submitEndpoint)
		v2.POST("/read", readEndpoint)
	}
	router.Run(":80")
}
```

### 运行多个服务

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"golang.org/x/sync/errgroup"
)

var (
	g errgroup.Group
)

func router01() http.Handler {
	e := gin.New()
	e.Use(gin.Logger())
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 01",
			},
		)
	})

	return e
}

func router02() http.Handler {
	e := gin.New()
	e.Use(gin.Logger())
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 02",
			},
		)
	})

	return e
}

func main() {
	server01 := &http.Server{
		Addr:         ":8080",
		Handler:      router01(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	server02 := &http.Server{
		Addr:         ":8081",
		Handler:      router02(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	g.Go(func() error {
		return server01.ListenAndServe()
	})

	g.Go(func() error {
		return server02.ListenAndServe()
	})

	if err := g.Wait(); err != nil {
		log.Fatal(err)
	}
}

```

### 重定向

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/test", func(c *gin.Context) {
		c.Request.URL.Path = "/test2"
		r.HandleContext(c)
	})
	r.GET("/test2", func(c *gin.Context) {
		c.JSON(200, gin.H{"hello": "world"})
	})
	r.Run(":80")
}

```

### 静态资源嵌入

```go
package main

import (
	"time"
)

var (
	_Assetsbfa8d115ce0617d89507412d5393a462f8e9b003 = "<!doctype html>\n<body>\n  <p>Can you see this? → {{.Bar}}</p>\n</body>\n"
	_Assets3737a75b5254ed1f6d588b40a3449721f9ea86c2 = "<!doctype html>\n<body>\n  <p>Hello, {{.Foo}}</p>\n</body>\n"
)

// Assets returns go-assets FileSystem
var Assets = assets.NewFileSystem(map[string][]string{"/": {"html"}, "/html": {"bar.tmpl", "index.tmpl"}}, map[string]*assets.File{
	"/": {
		Path:     "/",
		FileMode: 0x800001ed,
		Mtime:    time.Unix(1524365738, 1524365738517125470),
		Data:     nil,
	}, "/html": {
		Path:     "/html",
		FileMode: 0x800001ed,
		Mtime:    time.Unix(1524365491, 1524365491289799093),
		Data:     nil,
	}, "/html/bar.tmpl": {
		Path:     "/html/bar.tmpl",
		FileMode: 0x1a4,
		Mtime:    time.Unix(1524365491, 1524365491289611557),
		Data:     []byte(_Assetsbfa8d115ce0617d89507412d5393a462f8e9b003),
	}, "/html/index.tmpl": {
		Path:     "/html/index.tmpl",
		FileMode: 0x1a4,
		Mtime:    time.Unix(1524365491, 1524365491289995821),
		Data:     []byte(_Assets3737a75b5254ed1f6d588b40a3449721f9ea86c2),
	},
}, "")

```

```go
package main

import (
	"github.com/gin-gonic/gin"
	"html/template"
	"io/ioutil"
	"net/http"
	"strings"
)

func main() {
	r := gin.New()
	t, err := loadTemplate()
	if err != nil {
		panic(err)
	}
	r.SetHTMLTemplate(t)
	r.GET("/", func(c *gin.Context) {
		c.HTML(http.StatusOK, "/html/index.tmpl", gin.H{
			"Foo": "World",
		})
	})
	r.GET("/bar", func(c *gin.Context) {
		c.HTML(http.StatusOK, "/html/bar.tmpl", gin.H{
			"Bar": "World",
		})
	})
	r.Run(":8080")
}

func loadTemplate() (*template.Template, error) {
	t := template.New("")
	for name, file := range Assets.Files {
		if file.IsDir() || !strings.HasSuffix(name, ".tmpl") {
			continue
		}
		h, err := ioutil.ReadAll(file)
		if err != nil {
			return nil, err
		}
		t, err = t.New(name).Parse(string(h))
		if err != nil {
			return nil, err
		}
	}
	return t, nil
}

```