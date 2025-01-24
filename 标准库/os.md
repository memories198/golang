
## os篇

在 Go 语言中，可以使用 `os`、`io/ioutil` 和 `bufio` 包来进行文件操作。

### 总结

- `os.Create`：创建一个新文件。
- `os.Open`：打开一个文件用于读取。
- `os.Remove`：删除一个文件。
- ~~`ioutil.ReadFile`：读取整个文件的内容。~~
- ~~`ioutil.WriteFile`：将数据写入文件。~~
- `bufio.NewReader` 和 `bufio.NewWriter`：用于高效地读取和写入文件内容。

### os.Create

#### 作用

创建一个新文件，如果文件已存在则清空该文件，这种清空文件的操作可能会导致数据的丢失，不建议使用，一般情况进行数据的追加比较好。

#### 示例

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.Create("example.txt")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	file.WriteString("Hello, World!")
}
```

### os.Open

#### 作用

打开一个文件用于读取。它只能读取文件，无法修改文件内容。该函数主要用来进行读取配置文件等操作。

#### 示例

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("example.txt") //只读，无法使用file.Write函数写入数据到文件中
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	buf := make([]byte, 1024)
	n, err := file.Read(buf)
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	fmt.Println(string(buf[:n]))
}
```

### os.OpenFile

#### 作用

可以用你想要的权限打开文件。常用于记录日志等操作。

#### 示例

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	//os.O_CREATE表示文件不存在自动创建文件;os.O_RDWR表示以读写的方式打开文件,这种写的方式是从文件开头进行覆盖写入;0664表示创建文件时的权限
	file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR, 0664) //覆盖写入

	//file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0664) //os.O_APPEND表示追加写入
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	//_, err = file.Write([]byte("覆盖开头\n"))
	_, err = file.Write([]byte("hello world1\nhello world2\n"))
	if err != nil {
		log.Println("Error writing to file:", err)
		return
	}
}
```

### os.Remove

#### 作用

删除一个文件。

#### 示例

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	err := os.Remove("example.txt")
	if err != nil {
		fmt.Println("Error removing file:", err)
		return
	}

	fmt.Println("File removed successfully")
}
```

### ioutil.ReadFile

~~(已弃用,老的项目可能会有它的使用)~~

#### 作用

读取整个文件的内容。

#### 示例

```go
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
	content, err := ioutil.ReadFile("example.txt")
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	fmt.Println(string(content))
}
```

### ioutil.WriteFile

~~(已弃用,老的项目可能有用它的使用)~~

#### 作用

将数据写入文件。

#### 示例

```go
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
	data := []byte("Hello, World!")
	err := ioutil.WriteFile("example.txt", data, 0644)
	if err != nil {
		fmt.Println("Error writing file:", err)
		return
	}

	fmt.Println("File written successfully")
}
```

### bufio.NewReader

#### 作用

用于高效地读取文件内容。分批读取，不需要把文件全部加载到内存中，节省内存。

#### 示例

##### 按行分批次读取

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// 打开文件
	file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0664)
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	//读取文件
	reader := bufio.NewReader(file)
	//按行分批次读取
	for {
		line, err := reader.ReadString('\n') //读取文件第一行
		if err != nil {
			if err.Error() != "EOF" {
				fmt.Println("Error reading file:", err)
			}
			break
		}
		fmt.Printf("line:%v", line)
	}
}
```

##### 按大小分批次读取

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// 打开文件
	file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0664)
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	//读取文件
	reader := bufio.NewReader(file)
	//按大小分批次读取，每次最多读取1024个字节
	buf := make([]byte, 1024)
	for {
		n, err := reader.Read(buf) // n表示成功读取出的字节数
		if err != nil {
			if err.Error() != "EOF" {
				fmt.Println("Error reading file:", err)
			}
			break
		}
		fmt.Printf("buf:%v", string(buf[:n]))
	}
}

```

### bufio.NewWriter

#### 作用

用于高效地写入文件。避免频繁的把数据写入文件，而是将数据在内存中准备完成之后，一次性写入到文件中，性能更高。

#### 示例

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0664)
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	//写入数据到文件
	writer := bufio.NewWriter(file)
	_, err = writer.WriteString("Hello, World!\n")
	if err != nil {
		log.Println("Error writing string:", err)
		return
	}
	//将数据从bufio包的内存缓冲区写入到文件缓冲区，不进行这一步数据无法成功写入文件
	err = writer.Flush()
	if err != nil {
		log.Println("Flush error:", err)
		return
	}
}
```

#### 写入操作的补充

写入文件实质上是把数据保存到磁盘，但是这种保存到磁盘操作一般由操作系统自行完成，我们只需要把数据写入到内存中的文件缓冲区，操作系统会自行决定何时把缓冲区中的数据写入磁盘，但是有时我们需要保证可靠性，希望由我们自己触发写入磁盘的操作...

##### 示例

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.OpenFile("example.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, 0664)
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	//将数据写入到文件缓冲区，WriteString函数直接把数据写入文件缓冲区
	//操作系统检测到文件缓冲区有数据之后，会根据系统的负载情况自行决定什么时候把缓冲区中的内容写入到磁盘(落盘)，一般情况都能写入成功
	_, err = file.WriteString("Hello, World!\n")
	if err != nil {
		log.Println("Error writing to file:", err)
		return
	}

	//这一步不是必要的，因为操作系统会自动把文件缓冲区的内容写入文件(被动落盘)
	//下面是一个主动让操作系统进行落盘的操作，因为操作系统自行决定落盘时间可能会在高负载情况下出现数据丢失的情况，例如：突然宕机导致文件缓冲区的数据没有及时写入磁盘
	//将文件缓冲区中的数据写入磁盘
	err = file.Sync() //(主动落盘)
	if err != nil {
		log.Println("Error syncing file:", err)
		return
	}
}
```

### 常量
在 Go 语言的 `os` 包中，提供了与操作系统相关的功能，例如文件系统操作、环境变量访问和进程管理等。`os` 包中的一些常用的变量可以帮助我们处理文件、目录和进程等任务。下面是一些常用的变量及其解释：

#### os.Args
- **类型**: `[]string`
- **描述**: `os.Args` 是一个字符串切片，它包含了命令行参数。`os.Args[0]` 是执行的程序的路径，`os.Args[1:]` 是程序执行时传递的其他命令行参数。

  **示例**:
  ```go
  fmt.Println(os.Args) // 输出命令行参数
  ```

  如果程序的命令行是 `go run main.go arg1 arg2`，则 `os.Args` 会是 `["main.go", "arg1", "arg2"]`。

标准库中实现的解析选项的功能较为简单，且不区分长短选项，所有的选项都使用`-`或`--`指定。

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	// os.Args 返回一个字符串切片，第一个元素是程序名，后面的元素是传递的参数
	args := os.Args

	// 打印程序名
	fmt.Println("程序名:", args[0])

	// 打印传递的参数
	if len(args) == 1 {
		fmt.Println("没有传递参数")
		return
	}

	if len(args) > 1 {
		fmt.Println("传递的参数:")
		for i, arg := range args[1:] {
			fmt.Printf("参数 %d: %s\n", i+1, arg)
		}
	}

	// 定义命令行选项,定义包括三个参数: 选项名称、默认值、选项的含义。
	// 定义好后需要使用flag.Parse()解析。解析完后可以使用flag.Args()获取非选项参数
	// 传递的时候所有的选项名使用-或--指定，因为标准库不区分长短选项

	name := flag.String("name", "World", "The name to greet")
	age := flag.Int("age", 0, "Your age")
	height := flag.Int("h", 0, "Your height")

	verbose := flag.Bool("verbose", true, "Enable verbose output")

	// 解析命令行选项
	flag.Parse()

	// 获取剩余的非选项参数
	remainingArgs := flag.Args()

	// 打印选项和参数
	fmt.Printf("Hello, %s!\n", *name)
	fmt.Printf("Your age is: %d\n", *age)
	fmt.Printf("Your height is: %d\n", *height)

	if *verbose {
		fmt.Println("Verbose mode is enabled.")
	}

	if len(remainingArgs) > 0 {
		fmt.Println("Remaining arguments:")
		for i, arg := range remainingArgs {
			fmt.Printf("Argument %d: %s\n", i+1, arg)
		}
	} else {
		fmt.Println("No remaining arguments.")
	}
}

```
#### os.Environ()
- **类型**: `[]string`
- **描述**: `os.Environ()` 返回一个包含当前进程的所有环境变量的切片，每个元素是一个 `key=value` 形式的字符串。

  **示例**:
  ```go
  envs := os.Environ()
  for _, env := range envs {
      fmt.Println(env) // 输出所有环境变量
  }
  ```

#### os.Getenv(key string)
- **类型**: `string`
- **描述**: `os.Getenv` 用于获取指定环境变量的值。如果环境变量不存在，返回空字符串。

  **示例**:
  ```go
  value := os.Getenv("HOME")
  fmt.Println(value) // 输出 HOME 环境变量的值
  ```

#### os.IsExist(err error)
- **类型**: `func(err error) bool`
- **描述**: `os.IsExist` 是一个辅助函数，用来判断错误 `err` 是否表示文件或目录已存在的情况。如果错误类型为 `os.ErrExist`，则返回 `true`。

  **示例**:
  ```go
  err := os.Mkdir("existingDir", 0755)
  if os.IsExist(err) {
      fmt.Println("Directory already exists")
  }
  ```

#### os.IsNotExist(err error)
- **类型**: `func(err error) bool`
- **描述**: `os.IsNotExist` 用于判断错误 `err` 是否表示文件或目录不存在的错误类型。如果错误是 `os.ErrNotExist`，则返回 `true`。

  **示例**:
  ```go
  err := os.Stat("nonexistentfile")
  if os.IsNotExist(err) {
      fmt.Println("File does not exist")
  }
  ```

#### os.O_CREATE
- **类型**: `int`
- **描述**: `os.O_CREATE` 是一个常量，表示打开文件时，如果文件不存在则创建文件。它通常与其他文件打开标志一起使用。

  **示例**:
  ```go
  file, err := os.OpenFile("test.txt", os.O_RDWR|os.O_CREATE, 0666)
  if err != nil {
      fmt.Println(err)
  }
  defer file.Close()
  ```

#### os.O_EXCL
- **类型**: `int`
- **描述**: `os.O_EXCL` 是一个常量，用于在文件打开时，如果文件已经存在则返回错误。与 `os.O_CREATE` 一起使用，表示如果文件已存在则返回错误。

  **示例**:
  ```go
  file, err := os.OpenFile("test.txt", os.O_RDWR|os.O_CREATE|os.O_EXCL, 0666)
  if err != nil {
      fmt.Println("Error: File already exists")
  }
  defer file.Close()
  ```

#### os.O_RDONLY
- **类型**: `int`
- **描述**: `os.O_RDONLY` 是一个常量，用于表示以只读模式打开文件。

  **示例**:
  ```go
  file, err := os.OpenFile("test.txt", os.O_RDONLY, 0666)
  if err != nil {
      fmt.Println(err)
  }
  defer file.Close()
  ```

#### os.O_WRONLY
- **类型**: `int`
- **描述**: `os.O_WRONLY` 是一个常量，表示以只写模式打开文件。

  **示例**:
  ```go
  file, err := os.OpenFile("test.txt", os.O_WRONLY, 0666)
  if err != nil {
      fmt.Println(err)
  }
  defer file.Close()
  ```

#### os.O_RDWR
- **类型**: `int`
- **描述**: `os.O_RDWR` 是一个常量，用于表示以读写模式打开文件。

  **示例**:
  ```go
  file, err := os.OpenFile("test.txt", os.O_RDWR, 0666)
  if err != nil {
      fmt.Println(err)
  }
  defer file.Close()
  ```

#### os.Stdout
- **类型**: `*os.File`
- **描述**: `os.Stdout` 是一个指向标准输出流的 `*os.File` 类型的变量。你可以通过它直接输出到控制台。

  **示例**:
  ```go
  fmt.Fprintln(os.Stdout, "Hello, World!")
  ```

#### os.Stderr
- **类型**: `*os.File`
- **描述**: `os.Stderr` 是一个指向标准错误输出流的 `*os.File` 类型的变量。你可以通过它将错误信息输出到控制台的标准错误流。

  **示例**:
  ```go
  fmt.Fprintln(os.Stderr, "Error occurred!")
  ```

#### os.Stdin
- **类型**: `*os.File`
- **描述**: `os.Stdin` 是一个指向标准输入流的 `*os.File` 类型的变量。你可以通过它读取来自键盘的输入。

  **示例**:
  ```go
  var input string
  fmt.Fscan(os.Stdin, &input)
  fmt.Println("You entered:", input)
  ```

#### os.TempDir()
- **类型**: `func() string`
- **描述**: `os.TempDir` 返回操作系统的临时目录路径。你可以在这里存储临时文件。

  **示例**:
  ```go
  fmt.Println("Temporary directory:", os.TempDir())
  ```



