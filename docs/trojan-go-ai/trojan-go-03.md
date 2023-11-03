# trojan-go源码解析 3

# `component/mysql.go`

这段代码是一个 Go 语言中的 `package build` 包。它定义了一个名为 `build` 的函数，该函数使用了 `||` 语法，它表示或操作。这个函数接受两个参数 `mysql` 和 `full` 或 `mini`。函数的作用是执行名为 `mysql` 或 `full` 或 `mini` 的构建操作，如果这些操作中有一个成功，那么整个构建操作就会成功，否则所有操作都将失败。

具体来说，这段代码首先导入了 `github.com/p4gefau1t/trojan-go/statistic/mysql` 包，这个包中可能包含了与 MySQL 数据库交互的代码。然后，定义了一个名为 `build` 的函数，该函数使用了 `||` 语法，表示或操作。这个函数接受两个参数 `mysql` 和 `full` 或 `mini`。函数的作用是执行名为 `mysql` 或 `full` 或 `mini` 的构建操作，如果这些操作中有一个成功，那么整个构建操作就会成功，否则所有操作都将失败。


```go
//go:build mysql || full || mini
// +build mysql full mini

package build

import (
	_ "github.com/p4gefau1t/trojan-go/statistic/mysql"
)

```

# `component/nat.go`

这段代码是一个 Go 语言编程语言构建工具中的自然构建模式。它通过设置 `full` 和 `mini` 环境变量来创建一个名为 `nat` 的新环境。`nat` 环境将包含 `github.com/p4gefau1t/trojan-go/proxy/nat` 的引用。

在 `build` 包中，该代码定义了一个名为 `build` 的函数。该函数使用 `||` 运算符来组合 `full` 和 `mini` 环境变量，创建了一个名为 `nat` 的新环境。

通过调用 `build`，用户可以指定使用哪个构建模式来构建 Go 项目。如果用户不指定任何构建模式，则 Go 编译器将使用默认的构建模式，该模式可能包括一些自定义的构建规则。


```go
//go:build nat || full || mini
// +build nat full mini

package build

import (
	_ "github.com/p4gefau1t/trojan-go/proxy/nat"
)

```

# `component/other.go`

这段代码是一个 Go 语言编程语言特性中的一个构建输入，其中 `//go:build other || full` 是从其他源中复制或复制其他源，而 `//+build other full` 是添加到其他源中复制其他源。

具体而言，它从 `github.com/p4gefau1t/trojan-go` 的 `easy` 和 `url` 包中导入了一些类型定义。然后，通过在 `build` 包中定义了一个名为 `build` 的函数，可以创建或打开一个与当前工作目录相同的目录并输出一个名为 `build` 的新目录，将指定的依赖项复制到该目录中。


```go
//go:build other || full
// +build other full

package build

import (
	_ "github.com/p4gefau1t/trojan-go/easy"
	_ "github.com/p4gefau1t/trojan-go/url"
)

```

# `component/server.go`

这段代码是一个 Go 语言中的一个构建函数，它的作用是决定使用哪种构建模式来编译 P4GEfau1t/trojan-go/proxy/server 包。

首先，我们看到了一个名为“//go:build server || full || mini”的注释。这个注释告诉 Go 编译器在遇到无法处理的构建输入时，应该使用默认的构建模式。这个默认构建模式可能是服务器模式（server）、大型模式（full）或者是迷你模式（mini）。

接下来，我们看到了一个名为“// +build server full mini”的注释。这个注释告诉 Go 编译器应该使用“full”这个关键字来指定构建模式。

综合来看，这段代码的作用是告诉 Go 编译器在遇到无法处理的构建输入时，应该使用默认的构建模式，但是如果用户明确指定了“full”这个关键字，那么就应该使用“full”这个关键字来指定构建模式。


```go
//go:build server || full || mini
// +build server full mini

package build

import (
	_ "github.com/p4gefau1t/trojan-go/proxy/server"
)

```

# `config/config.go`

这段代码定义了一个名为 `config` 的包，其中包含了一个 `Creator` 类型和一个 `make` 函数。

`Creator` 类型是一个函数，它接收一个空白字符串并返回一个 `Creator` 实例。这个 `Creator` 函数将接收到的字符串解析成一个 JSON 对象的序列化表示形式。

`make` 函数接受一个 `Creator` 类型的参数，并返回一个 `Creator` 实例。这个 `Creator` 实例将在 `creators`  map 中使用，`creators` map 用来存储由不同构造函数创建的配置结构体。

这段代码还定义了一个名为 `Creator` 的函数，它将接收一个 JSON 对象的序列化表示形式，并返回一个 `Creator` 实例。这个 `Creator` 函数将用于从 JSON 对象中解析配置结构体。

最后，这段代码还导入了一些外部依赖，包括 `gopkg.in/yaml.v3` 和 `encoding/json`。


```go
package config

import (
	"context"
	"encoding/json"

	"gopkg.in/yaml.v3"
)

var creators = make(map[string]Creator)

// Creator creates default config struct for a module
type Creator func() interface{}

// RegisterConfigCreator registers a config struct for parsing
```

这两段代码定义了两个函数：RegisterConfigCreator 和 parseJSON。

RegisterConfigCreator 函数接收一个名称参数和一个创作者对象。函数将创建一个名为 "CONFIG" 加上该名称的配置项，并将创作者对象存储在名为 "config" 的配置项中。这样，以后可以通过调用这个函数来获取配置项中的创作者对象。

parseJSON 函数接收一个字节切片（可能是 JSON 数据）。函数使用一个空字典 `result` 来存储解析后得到的配置项。然后，函数遍历输入数据中的所有配置项名称和创作者对象，将创作者对象存储在字典中。如果解析 JSON 数据时出现错误，函数返回一个非空地图 error。最后，函数返回结果字典和错误。


```go
func RegisterConfigCreator(name string, creator Creator) {
	name += "_CONFIG"
	creators[name] = creator
}

func parseJSON(data []byte) (map[string]interface{}, error) {
	result := make(map[string]interface{})
	for name, creator := range creators {
		config := creator()
		if err := json.Unmarshal(data, config); err != nil {
			return nil, err
		}
		result[name] = config
	}
	return result, nil
}

```

这两段代码都是Go语言中的函数，它们的主要目的是读取并解析YAML和JSON格式的数据。

首先，我们来看一下`parseYAML`函数。它接收一个字节切片（[]byte）作为参数，并返回一个map类型的结果，其中key是字符串类型，value是interface{}类型。这个函数的作用是将收到的YAML数据解析为相应的map类型。函数内部使用一个for循环，遍历一个定义好的创作者函数，这个创作者函数接收一个字节切片（[]byte）并返回一个配置类（map[string]interface{}）。如果返回的函数执行出现错误（如YAML解析失败），则函数返回一个nil类型的错误。

接下来，我们看`WithJSONConfig`函数。它接收一个字节切片（[]byte）作为参数，返回一个context类型和一个错误类型。函数内部首先调用`parseJSON`函数来读取并解析JSON格式的数据，并将结果存储到一个名为`configs`的map类型中。然后，它遍历configs中的每个键值对，将每个键值对的键（string类型）存储到一个名为`ctx`的context类型中，并将每个键对应的值（interface{}类型）存储到当前的上下文（context）中。最后，函数返回`ctx`和错误类型的一个组合，以便在需要时回收上下文。


```go
func parseYAML(data []byte) (map[string]interface{}, error) {
	result := make(map[string]interface{})
	for name, creator := range creators {
		config := creator()
		if err := yaml.Unmarshal(data, config); err != nil {
			return nil, err
		}
		result[name] = config
	}
	return result, nil
}

func WithJSONConfig(ctx context.Context, data []byte) (context.Context, error) {
	var configs map[string]interface{}
	var err error
	configs, err = parseJSON(data)
	if err != nil {
		return ctx, err
	}
	for name, config := range configs {
		ctx = context.WithValue(ctx, name, config)
	}
	return ctx, nil
}

```

这段代码定义了两个函数，分别是`WithYAMLConfig`和`WithConfig`。它们的作用是：

1. `WithYAMLConfig`函数接收一个`ctx`上下文、一个字节切片`data`，并返回一个新上下文和一个错误。

2. `WithConfig`函数接收一个`ctx`上下文和一个`name`参数，并接收一个`cfg`参数，然后返回一个新上下文。

这两个函数的主要作用是为函数`ctx.WithValue`提供配置参数的上下文。函数`ctx.WithValue`接收一个名称（例如，`"my_variable"`）和一个配置参数（例如，`my_value`）。它返回一个新的上下文，其中将`my_variable`设置为`my_value`。上下文新设置的名称将以`_CONFIG`为前缀。

`WithYAMLConfig`函数首先将收到的`data`字节切片解析为YAML配置。如果解析过程中出现错误，则返回一个新上下文和一个错误。然后，它遍历`configs` map，将每个配置名字符串（例如，`option: value`）设置为`ctx`上下文的`name`属性的值。

`WithConfig`函数接收一个`ctx`上下文和一个名称`name`。它遍历接收到的`cfg`参数，并将它设置为`ctx`上下文的`name`属性的值。然后，它返回一个新上下文。


```go
func WithYAMLConfig(ctx context.Context, data []byte) (context.Context, error) {
	var configs map[string]interface{}
	var err error
	configs, err = parseYAML(data)
	if err != nil {
		return ctx, err
	}
	for name, config := range configs {
		ctx = context.WithValue(ctx, name, config)
	}
	return ctx, nil
}

func WithConfig(ctx context.Context, name string, cfg interface{}) context.Context {
	name += "_CONFIG"
	return context.WithValue(ctx, name, cfg)
}

```

这段代码定义了一个名为FromContext的函数，它接收一个上下文上下文和一个配置参数名称作为参数。函数的作用是从上下文中获取指定配置参数的值，并返回该配置参数的类型。

具体来说，这段代码实现了一个通用函数，名为FromContext，它可以在任何上下文中调用，而不受上下文的生命周期限制。通过调用函数，程序员可以将上下文上下文中的配置参数从上下文中检索出来，并将其存储为指定的类型。

例如，您可以在应用程序的配置文件中定义一个环境变量，例如：


CONFIG_VARIABLE=foo


然后，您可以使用FromContext函数从上下文中获取该变量的值，并将其存储为配置参数类型：


config := FromContext(ctx, "ENV_VARIABLE")


在这种情况下，FromContext函数将返回一个与上下文上下文中的“CONFIG_VARIABLE”具有相同名称和值的类型。


```go
// FromContext extracts config from a context
func FromContext(ctx context.Context, name string) interface{} {
	return ctx.Value(name + "_CONFIG")
}

```

# `config/config_test.go`

这段代码定义了一个名为config的包，其中包含了一些通用的功能，以及一个名为Foo的结构体和一个名为TestStruct的结构体。

Foo结构体包含两个字段，字段1是string类型，字段2是bool类型。其中，字段1和字段2的字符串和yaml格式的交叉引用使用了json和yaml两种数据格式。

TestStruct结构体包含三个字段，字段1是string类型，字段2是bool类型，字段3是一个包含Foo类型的字段。其中，字段1和字段2的字符串和yaml格式的交叉引用使用了json和yaml两种数据格式。

这段代码的作用是定义了一个通用的config包，其中包含了一些常用的功能，以及一个Foo结构体和一个TestStruct结构体。这些结构体和字段可以被用于测试和验证应用程序的配置。


```go
package config

import (
	"context"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
)

type Foo struct {
	Field1 string `json,yaml:"field1"`
	Field2 bool   `json:"field2" yaml:"field2"`
}

type TestStruct struct {
	Field1 string `json,yaml:"field1"`
	Field2 bool   `json,yaml:"field2"`
	Field3 []Foo  `json,yaml:"field3"`
}

```

该代码定义了一个名为`creator`的函数，其返回类型为`TestStruct`的接口类型。

该函数的内容如下：


func creator() *TestStruct {
	return &TestStruct{}
}


函数没有返回值，因为它只是简单地返回一个`TestStruct`类型的引用。

该函数用于注册一个名为`test`的配置创建器，当有请求创建一个包含`TestStruct`类型的数据时，该函数会被调用。

在`TestJSONConfig`函数中，我们注册了上述的`creator`函数。

如果所有的测试都通过了，那么该代码将在编译时生成一个`test.go`文件，其中包含以下代码：


package main

import (
	"testing"

	"github.com/stretchr/testify/assert"

	"golang.org/x/net/http"

	"github.com/stretchr/testify/go.convey"
	"github.com/stretchr/testify/stretchr"
)

type TestStruct struct {
	Field1 string
	Field2 bool
}

func TestJSONConfig(t *testing.T) {
	type testCase struct {
		name        string
		description string
		filename      string
		field1      string
		field2      bool
		expected     bool
		targetFn    func(t *testing.T, fs *schema.Scheme) error
		配置Section string
		target        interface{}
		预期的地方*net/http.Response
		弯曲的末端*http.Response
	}
	t.TrueSig(func(t *testing.T) {
		t.Parallel()
		configBytes, err := ioutil.ReadAll(t.Source)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		config, err := schema.Must parse(configBytes)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		t.Parallel()
		schema, err := config.TestCaseSection(config.TestCaseSectionIndex)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		if len(schema.Case) == 0 {
			t.Fatalf("no test cases for %s", config.Name)
		}
		t.Parallel()
		assert.Equal(t, schema.Case[0].Name, "test")
		assert.Equal(t, schema.Case[0].Description, "test some configs")
		assert.Equal(t, schema.Case[0].Caller, "my/test/config")
		assert.Equal(t, schema.Case[0].Args[0].Type(), "*net.Response")
		assert.Equal(t, schema.Case[0].Args[0].ID("field1"), "test1")
		assert.Equal(t, schema.Case[0].Args[0].String("field2"), "true")
		assert.Equal(t, schema.Case[0].Call("my/test/config").Field1, "test1")
		assert.Equal(t, schema.Case[0].Call("my/test/config").Field2, true)
		assert.Equal(t, schema.Case[0].Call("my/test/config").End, nil)
		t.Parallel()
		target, err := schema.Case[0].Call("my/test/config").Field1
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		expected, err := net.MustCallEnd(t.Context(), target)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		if expected.StatusCode != http.StatusOK {
			t.Fatalf("expected status: %d", expected.StatusCode)
		}
		t.Parallel()
		if !schema.Case[0].Call("my/test/config").Field2 {
			t.Fatalf("field2 should be true")
		}
		t.Parallel()
		target, err := net.MustCallEnd(t.Context(), target)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		expected, err := net.MustCallEnd(t.Context(), target)
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		if expected.StatusCode != http.StatusOK {
			t.Fatalf("expected status: %d", expected.StatusCode)
		}
		t.Parallel()
		assert.Equal(t, expected.Body, "some configs")
		t.Parallel()
		// Add more test cases if needed
		// ...
		t.Parallel()
		t.Success()
	})
	go func() {
		//读取所有文件
		files, err := ioutil.ReadAll([]string{"test.go", "test_config.go"})
		if err != nil {
			t.Fatalf("fatal err: %v", err)
		}
		for _, f := range files {
			t.Run(f, func(t *testing.T) {
				configBytes, err := ioutil.ReadAll(f)
				if err != nil {
					t.Fatalf("fatal err: %v", err)
				}
				config, err := schema.Mustparse(configBytes)
				if err != nil {
						t.Fatalf("fatal err: %v", err)
					}
					schema, err := config.TestCaseSection(config.TestCaseSectionIndex)
					if err != nil {
							t.Fatalf("fatal err: %v", err)
						}
						assert.Equal(t, schema.Case[0].Name, "test")
						assert.Equal(t, schema.Case[0].Description, "test some configs")
						assert.Equal(t, schema.Case[0].Caller, "my/test/config")
						assert.Equal(t, schema.Case[0].Args[0].Type(), "*net.Response")
						assert.Equal(t, schema.Case[0].Args[0].ID("field1"), "test1")
						assert.Equal(t, schema.Case[0].Args[0].String("field2"), "true")
						assert.Equal(t, schema.Case[0].Call("my/test/config").Field1, "test1")
							assert.Equal(t, schema.Case[0].Args[0].String("field2"), "true")
							assert.Equal(t, schema.Case[0].Call("my/test/config").End, nil)
	


```go
func creator() interface{} {
	return &TestStruct{}
}

func TestJSONConfig(t *testing.T) {
	RegisterConfigCreator("test", creator)
	data := []byte(`
	{
		"field1": "test1",
		"field2": true,
		"field3": [
			{
				"field1": "aaaa",
				"field2": true
			}
		]
	}
	`)
	ctx, err := WithJSONConfig(context.Background(), data)
	common.Must(err)
	c := FromContext(ctx, "test").(*TestStruct)
	if c.Field1 != "test1" || c.Field2 != true {
		t.Fail()
	}
}

```

这段代码是一个名为 "TestYAMLConfig" 的函数，它属于 testing 包。它的作用是测试一个名为 "test" 的 YAML 配置文件。具体来说，它注册了一个名为 "test" 的 YAML 配置创建器，当 YAML 文件中包含 "test" 名称的配置项时，就会执行这个注册器。

接下来，它读取一个包含 YAML 数据的字节数组，并使用 "WithYAMLConfig" 函数与 YAML 文件结合的上下文。然后，它设置一个名为 "test" 的上下文，并将之前的字节数组、错误条件和测试结构（TestStruct）作为参数传入。最后，它通过 "FromContext" 函数返回一个名为 "test" 的上下文的 "TestStruct" 类型，并对 "Field1"、"Field2" 和 "Field3" 字段的值进行比较，如果任何值不正确，就会打印出 "t.Fail" 并崩溃应用程序。


```go
func TestYAMLConfig(t *testing.T) {
	RegisterConfigCreator("test", creator)
	data := []byte(`
field1: 012345678
field2: true
field3:
  - field1: test
    field2: true
`)
	ctx, err := WithYAMLConfig(context.Background(), data)
	common.Must(err)
	c := FromContext(ctx, "test").(*TestStruct)
	if c.Field1 != "012345678" || c.Field2 != true || c.Field3[0].Field1 != "test" {
		t.Fail()
	}
}

```

# `constant/constant.go`

这段代码定义了两个变量 `Version` 和 `Commit`，并将它们的值设置为 "Custom Version" 和 "Unknown Git Commit ID"。然后，它导入了两个常量 `CustomVersion` 和 `UnknownGitCommitID`。

可能是在一个 Go 程序中，用于定义和导出一些通用的常量。


```go
package constant

var (
	Version = "Custom Version"
	Commit  = "Unknown Git Commit ID"
)

```

---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
---



---
title: "简介"
draft: false
weight: 10
---

# Trojan-Go

这里是Trojan-Go的文档，你可以在左侧的导航栏中找到一些使用技巧，以及完整的配置文件说明。

Trojan-Go是使用Go语言实现的完整的Trojan代理，和Trojan协议以及原版的配置文件格式兼容。支持并且兼容Trojan-GFW版本的绝大多数功能，并扩展了更多的实用功能。

Trojan-Go的的首要目标是保障传输安全性和隐蔽性。在此前提下，尽可能提升传输性能和易用性。

如果你遇到配置和使用方面的问题，发现了软件Bug，或是有更好的想法，欢迎加入Trojan-Go的[Telegram交流反馈群](https://t.me/trojan_go_chat)。

----

> Across the Great Wall, we can reach every corner in the world.
>
> (越过长城，走向世界。)


---
title: "使用Shadowsocks AEAD进行二次加密"
draft: false
weight: 8
---

### 注意，Trojan不支持这个特性

Trojan协议本身无加密，其安全性依赖于下层的TLS。在一般情况下，TLS安全性很好，并不需要再次加密Trojan流量。但是，某些场景下，你可能无法保证TLS隧道的安全性：

- 你使用了Websocket，经过不可信的CDN进行中转（如国内CDN）

- 你与服务器的连接遭到了GFW针对TLS的中间人攻击

- 你的证书失效，无法验证证书有效性

- 你使用了无法保证密码学安全的可插拔传输层

等等。

Trojan-Go支持使用Shadowsocks AEAD对Trojan-Go进行加密。其本质是在Trojan协议下方加上一层Shadowsocks AEAD加密。服务端和客户端必须同时开启，且密码和加密方式必须一致，否则无法进行通讯。

要开启AEAD加密，只需添加一个```goshadowsocks```选项：

```gojson
...
"shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "1234567890"
}
```

```gomethod```如果省略，则默认使用AES-128-GCM。更多信息，参见“完整的配置文件”一节。


---
title: "使用API动态管理用户"
draft: false
weight: 10
---

### 注意，Trojan不支持这个特性

Trojan-Go使用gRPC提供了一组API，API支持以下功能：

- 用户信息增删改查

- 流量统计

- 速度统计

- IP连接数统计

Trojan-Go本身集成了API控制功能，也即可以使用一个Trojan-Go实例控制另一个Trojan-Go服务器。

你需要在你需要被控制的服务端配置添加API设置，例如：

```gojson
{
    ...
    "api": {
        "enabled": true,
        "api_addr": "127.0.0.1",
        "api_port": 10000,
    }
}
```

然后启动Trojan-Go服务器

```goshell
./trojan-go -config ./server.json
```

然后可以使用另一个Trojan-Go连接该服务器进行管理，基本命令格式为

```goshell
./trojan-go -api-addr SERVER_API_ADDRESS -api COMMAND
```

其中```goSERVER_API_ADDRESS```为API地址和端口，如127.0.0.1:10000

```goCOMMAND```为API命令，合法的命令有

- list 列出所有用户

- get 获取某个用户信息

- set 设置某个用户信息（添加/删除/修改）

下面是一些例子

1. 列出所有用户信息

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api list
    ```

    所有的用户信息将以json的形式导出，信息包括在线IP数量，实时速度，总上传和下载流量等。下面是一个返回的结果的例子

    ```gojson
    [{"user":{"hash":"d63dc919e201d7bc4c825630d2cf25fdc93d4b2f0d46706d29038d01"},"status":{"traffic_total":{"upload_traffic":36393,"download_traffic":186478},"speed_current":{"upload_speed":25210,"download_speed":72384},"speed_limit":{"upload_speed":5242880,"download_speed":5242880},"ip_limit":50}}]
    ```

    流量单位均为字节。

2. 获取一个用户信息

    可以使用 -target-password 指定密码，也可以使用 -target-hash 指定目标用户密码的SHA224散列值。格式和list命令相同

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api get -target-password password
    ```

    或者

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api get -target-hash d63dc919e201d7bc4c825630d2cf25fdc93d4b2f0d46706d29038d01
    ```

    以上两条命令等价，下面的例子统一使用明文密码的方式，散列值指定某个用户的方式以此类推。

    该用户信息将以json的形式导出，格式与list命令类似。下面是一个返回的结果的例子

    ```gojson
    {"user":{"hash":"d63dc919e201d7bc4c825630d2cf25fdc93d4b2f0d46706d29038d01"},"status":{"traffic_total":{"upload_traffic":36393,"download_traffic":186478},"speed_current":{"upload_speed":25210,"download_speed":72384},"speed_limit":{"upload_speed":5242880,"download_speed":5242880},"ip_limit":50}}
    ```

3. 添加一个用户信息

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api set -add-profile -target-password password
    ```

4. 删除一个用户信息

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api set -delete-profile -target-password password
    ```

5. 修改一个用户信息

    ```goshell
    ./trojan-go -api-addr 127.0.0.1:10000 -api set -modify-profile -target-password password \
        -ip-limit 3 \
        -upload-speed-limit 5242880 \
        -download-speed-limit 5242880
    ```

    这个命令将密码为password的用户上传和下载速度限制为5MiB/s，同时连接的IP数量限制为3个，注意这里5242880的单位是字节。如果填写0或者负数，则表示不进行限制。


---
title: "自定义协议栈"
draft: false
weight: 8
---

### 注意，Trojan不支持这个特性

Trojan-Go允许高级用户自定义协议栈。在自定义模式下，Trojan-Go将放弃对协议栈的控制，允许用户操作底层协议栈组合。例如

- 在一层TLS上再建立一层或更多层TLS加密

- 使用TLS传输Websocket流量，在Websocket层上再建立一层TLS，在第二层TLS上再使用Shadowsocks AEAD进行加密传输

- 在TCP连接上，使用Shadowsocks的AEAD加密传输Trojan协议

- 将一个入站Trojan的TLS流量解包后重新用TLS包装为新的出站Trojan流量

等等。

**如果你不了解网络相关知识，请不要尝试使用这个功能。不正确的配置可能导致Trojan-Go无法正常工作，或是导致性能和安全性方面的问题。**

Trojan-Go将所有协议抽象为隧道，每个隧道可能提供客户端，负责发送；也可能提供服务端，负责接受；或者两者皆提供。自定义协议栈即自定义隧道的堆叠方式。

### 在继续配置之前，请先阅读开发指南中“基本介绍”一节，确保已经理解Trojan-Go运作方式

下面是Trojan-Go支持的隧道和他们的属性:

| 隧道        | 需要下层提供流 | 需要下层提供包 | 向上层提供流 | 向上层提供包 | 可以作为入站 | 可以作为出站 |
| ----------- | -------------- | -------------- | ------------ | ------------ | ------------ | ------------ |
| transport   | n              | n              | y            | y            | y            | y            |
| dokodemo    | n              | n              | y            | y            | y            | n            |
| tproxy      | n              | n              | y            | y            | y            | n            |
| tls         | y              | n              | y            | n            | y            | y            |
| trojan      | y              | n              | y            | y            | y            | y            |
| mux         | y              | n              | y            | n            | y            | y            |
| simplesocks | y              | n              | y            | y            | y            | y            |
| shadowsocks | y              | n              | y            | n            | y            | y            |
| websocket   | y              | n              | y            | n            | y            | y            |
| freedom     | n              | n              | y            | y            | n            | y            |
| socks       | y              | y              | y            | y            | y            | n            |
| http        | y              | n              | y            | n            | y            | n            |
| router      | y              | y              | y            | y            | n            | y            |
| adapter     | n              | n              | y            | y            | y            | n            |

自定义协议栈的工作方式是，定义树/链上节点并分别它们起名（tag）并添加配置，然后使用tag组成的有向路径，描述这棵树/链。例如，对于一个典型的Trojan-Go服务器，可以如此描述：

入站，一共两条路径，tls节点将自动识别trojan和websocket流量并进行分发

- transport->tls->trojan

- transport->tls->websocket->trojan

出站，只能有一条路径

- router->freedom

对于入站，从根开始描述多条路径，组成一棵**多叉树**（也可以退化为一条链），不满足树性质的图将导致未定义的行为；对于出站，必须描述一条**链**。

每条路径必须满足这样的条件：

1. 必须以**不需要下层提供流或包**的隧道开始(transport/adapter/tproxy/dokodemo等)

2. 必须以**能向上层提供包和流**的隧道终止(trojan/simplesocks/freedom等)

3. 出站单链上，隧道必须都可作为出站。入站的所有路径上，隧道必须都可作为入站。

要启用自定义协议栈，将```gorun_type```指定为custom，此时除```goinbound```和```gooutbound```之外的其他选项将被忽略。

下面是一个例子，你可以在此基础上插入或减少协议节点。配置文件为简明起见，使用YAML进行配置，你也可以使用JSON来配置，除格式不同之外，效果是等价的。

客户端 client.yaml

```goyaml
run-type: custom

inbound:
  node:
    - protocol: adapter
      tag: adapter
      config:
        local-addr: 127.0.0.1
        local-port: 1080
    - protocol: socks
      tag: socks
      config:
        local-addr: 127.0.0.1
        local-port: 1080
  path:
    -
      - adapter
      - socks

outbound:
  node:
    - protocol: transport
      tag: transport
      config:
        remote-addr: you_server
        remote-port: 443

    - protocol: tls
      tag: tls
      config:
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan
      config:
        password:
          - 12345678

  path:
    -
      - transport
      - tls
      - trojan

```

服务端 server.yaml

```goyaml
run-type: custom

inbound:
  node:
    - protocol: websocket
      tag: websocket
      config:
        websocket:
            enabled: true
            hostname: example.com
            path: /ws

    - protocol: transport
      tag: transport
      config:
        local-addr: 0.0.0.0
        local-port: 443
        remote-addr: 127.0.0.1
        remote-port: 80

    - protocol: tls
      tag: tls
      config:
        remote-addr: 127.0.0.1
        remote-port: 80
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan1
      config:
        remote-addr: 127.0.0.1
        remote-port: 80
        password:
          - 12345678

    - protocol: trojan
      tag: trojan2
      config:
        remote-addr: 127.0.0.1
        remote-port: 80
        password:
          - 87654321

  path:
    -
      - transport
      - tls
      - trojan1
    -
      - transport
      - tls
      - websocket
      - trojan2

outbound:
  node:
    - protocol: freedom
      tag: freedom

  path:
    -
      - freedom
```


---
title: "隧道和反向代理"
draft: false
weight: 5
---

你可以使用Trojan-Go建立隧道。一个典型的应用是，使用Trojan-Go在本地建立一个无污染的DNS服务器，下面是一个配置的例子

```gojson
{
    "run_type": "forward",
    "local_addr": "127.0.0.1",
    "local_port": 53,
    "remote_addr": "your_awesome_server",
    "remote_port": 443,
    "target_addr": "8.8.8.8",
    "target_port": 53,
    "password": [
        "your_awesome_password"
    ]
}
```

forward本质上是一个客户端，不过你需要填入```gotarget_addr```和```gotarget_port```字段，指明反向代理的目标。

使用这份配置文件后，本地53的TCP和UDP端口将被监听，所有的向本地53端口发送的TCP或者UDP数据，都会通过TLS隧道转发给远端服务器your_awesome_server，远端服务器得到回应后，数据会通过隧道返回到本地53端口。 也就是说，你可以将127.0.0.1当作一个DNS服务器，本地查询的结果和远端服务器查询的结果是一致的。你可以使用这个配置避开DNS污染。

同样的原理，你可以在本地搭建一个Google的镜像

```gojson
{
    "run_type": "forward",
    "local_addr": "127.0.0.1",
    "local_port": 443,
    "remote_addr": "your_awesome_server",
    "remote_port": 443,
    "target_addr": "www.google.com",
    "target_port": 443,
    "password": [
        "your_awesome_password"
    ]
}
```

访问```gohttps://127.0.0.1```即可访问谷歌主页，但是注意这里由于谷歌服务器提供的https证书是google.com的证书，而当前域名为127.0.0.1，因此浏览器会引发一个证书错误的警告。

类似的，可以使用forward传输其他代理协议。例如，使用Trojan-Go传输shadowsocks的流量，远端主机开启ss服务器，监听127.0.0.1:12345，并且远端服务器在443端口开启了正常的Trojan-Go服务器。你可以如此指定配置

```gojson
{
    "run_type": "forward",
    "local_addr": "0.0.0.0",
    "local_port": 54321,
    "remote_addr": "your_awesome_server",
    "remote_port": 443,
    "target_addr": "www.google.com",
    "target_port": 12345,
    "password": [
        "your_awesome_password"
    ]
}
```

此后，任何连接本机54321端口的TCP/UDP连接，等同于连接远端12345端口。你可以使用shadowsocks客户端连接本地的54321端口，ss流量将使用trojan的隧道连接传输至远端12345端口的ss服务器。


---
title: "启用多路复用提升网络并发性能"
draft: false
weight: 1
---

### 注意，Trojan不支持这个特性

Trojan-Go支持使用多路复用提升网络并发性能。

Trojan协议基于TLS。在一个TLS安全连接建立之前，连接双方需要进行密钥协商和交换等步骤确保后续通讯的安全性。这个过程即为TLS握手。

目前GFW对于TLS握手存在审查和干扰，同时由于出口网络拥塞的原因，普通的线路完成TLS握手通常需要将近一秒甚至更长的时间。这可能导致浏览网页和观看视频的延迟提高。

Trojan-Go使用多路复用的方式解决这一问题。每个建立的TLS连接将承载多个TCP连接。当新的代理请求到来时，不需要和服务器握手发起一个新的TLS连接，而是尽可能重复使用已有的TLS连接。以此减少频繁TLS握手和TCP握手的带来的延迟。

启用多路复用不会增加你的链路速度（甚至会有所减少），而且可能会增加服务器和客户端的计算负担。可以粗略地理解为，多路复用牺牲网络吞吐和CPU功耗，换取更低的延迟。在高并发的情景下，如浏览含有大量图片的网页时，或者发送大量UDP请求时，可以提升使用体验。

激活```gomux```模块，只需要将```gomux```选项中```goenabled```字段设为true即可，下面是一个客户端的例子

```gojson
...
"mux" :{
    "enabled": true
}
```

只需要配置客户端即可，服务端可以自动适配，无需配置```gomux```选项。

完整的mux配置如下

```gojson
"mux": {
    "enabled": false,
    "concurrency": 8,
    "idle_timeout": 60
}
```

```goconcurrency```是每个TLS连接最多可以承载的TCP连接数。这个数值越大，每个TLS连接被复用的程度就更高，握手导致的延迟越低。但服务器和客户端的计算负担也会越大，这有可能使你的网络吞吐量降低。如果你的线路的TLS握手极端缓慢，你可以将这个数值设置为-1，Trojan-Go将只进行一次TLS握手，只使用唯一的一条TLS连接进行传输。

```goidle_timeout```指的是每个TLS连接空闲多长时间后关闭。设置超时时间，**可能**有助于减少不必要的长连接存活确认(Keep Alive)流量传输引发GFW的探测。你可以将这个数值设置为-1，TLS连接在空闲时将被立即关闭。


---
title: "透明代理"
draft: false
weight: 11
---

### 注意，Trojan不完全支持这个特性（UDP）

Trojan-Go支持基于tproxy的透明TCP/UDP代理。

要开启透明代理模式，将一份正确的客户端配置（配置方式参见基本配置部分）其中的```gorun_type```修改为```gonat```，并按照需求修改本地监听端口即可。

之后需要添加iptables规则。这里假定你的网关具有两个网卡，下面这份配置将其中一个网卡（局域网）的入站包转交给Trojan-Go，由Trojan-Go通过隧道，通过另一个网卡（互联网）发送到远端Trojan-Go服务器。你需要将下面的```go$SERVER_IP```，```go$TROJAN_GO_PORT```，```go$INTERFACE```替换为自己的配置。

```goshell
# 新建TROJAN_GO链
iptables -t mangle -N TROJAN_GO

# 绕过Trojan-Go服务器地址
iptables -t mangle -A TROJAN_GO -d $SERVER_IP -j RETURN

# 绕过私有地址
iptables -t mangle -A TROJAN_GO -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN_GO -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN_GO -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A TROJAN_GO -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A TROJAN_GO -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A TROJAN_GO -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A TROJAN_GO -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A TROJAN_GO -d 240.0.0.0/4 -j RETURN

# 未命中上文的规则的包，打上标记
iptables -t mangle -A TROJAN_GO -j TPROXY -p tcp --on-port $TROJAN_GO_PORT --tproxy-mark 0x01/0x01
iptables -t mangle -A TROJAN_GO -j TPROXY -p udp --on-port $TROJAN_GO_PORT --tproxy-mark 0x01/0x01

# 从$INTERFACE网卡流入的所有TCP/UDP包，跳转TROJAN_GO链
iptables -t mangle -A PREROUTING -p tcp -i $INTERFACE -j TROJAN_GO
iptables -t mangle -A PREROUTING -p udp -i $INTERFACE -j TROJAN_GO

# 添加路由，打上标记的包重新进入本地回环
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
```

配置完成后**以root权限启动**Trojan-Go客户端：

```goshell
sudo trojan-go
```


---
title: "一种基于SNI代理的多路径分流中继方案"
draft: false
weight: 6
---

## 前言

Trojan 是一种通过 TLS 封装后进行加密数据传输的工具，利用其 TLS 的特性，我们可以通过 SNI 代理实现在同一主机端口上实现不同路径的分流中继。

## 所需工具及其他准备

- 中继机：nginx 1.11.5 及以上版本
- 落地机：trojan 服务端（无版本要求）

## 配置方法

为了便于说明，这里使用了两台中继主机和两台落地主机。  
四台主机所绑定的域名分别为 (a/b/c/d).example.com。如图所示。  
相互连接一共4条路径。分别为 a-c、a-d、b-c、b-d 。

```gotext
                        +-----------------+           +--------------------+
                        |                 +---------->+                    |
                        |   VPS RELAY A   |           |   VPS ENDPOINT C   |
                  +---->+                 |   +------>+                    |
                  |     |  a.example.com  |   |       |   c.example.com    |
                  |     |                 +------+    |                    |
  +----------+    |     +-----------------+   |  |    +--------------------+
  |          |    |                           |  |
  |  client  +----+                           |  |
  |          |    |                           |  |
  +----------+    |     +-----------------+   |  |    +--------------------+
                  |     |                 |   |  |    |                    |
                  |     |   VPS RELAY B   |   |  +--->+   VPS ENDPOINT D   |
                  +---->+                 +---+       |                    |
                        |  b.example.com  |           |   d.example.com    |
                        |                 +---------->+                    |
                        +-----------------+           +--------------------+
```

### 配置路径域名和相应的证书

首先我们需要将每条路径分别分配一个域名，并使其解析到分别的入口主机上。  

```gotext
a-c.example.com CNAME a.example.com  
a-d.example.com CNAME a.example.com  
b-c.example.com CNAME b.example.com  
b-d.example.com CNAME b.example.com
```

然后我们需要在落地主机上部署所有目标路径的证书  
由于解析记录和主机 IP 不符，HTTP 验证无法通过。这里建议使用 DNS 验证方式签发证书。  
具体 DNS 验证插件需要根据您的域名 DNS 解析托管商选择，这里使用了 AWS Route 53。  

```goshell
certbot certonly --dns-route53 -d a-c.example.com -d b-c.example.com // 主机 C 上
certbot certonly --dns-route53 -d a-d.example.com -d b-d.example.com // 主机 D 上
```

### 配置 SNI 代理

这里我们使用 nginx 的 ssl_preread 模块实现 SNI 代理。  
请安装 nginx 后按如下方法修过 nginx.conf 文件。  
注意这里不是 HTTP 服务，请不要写在虚拟主机的配置中。

这里给出主机 A 上的对应配置，主机 B 同理。

```gonginx
stream {
  map $ssl_preread_server_name $name {
    a-c.example.com   c.example.com;  # 将 a-c 路径流量转发至主机 C
    a-d.example.com   d.example.com;  # 将 a-d 路径流量转发至主机 D

    # 如果此主机上需要配置其他占用 443 端口的服务 （例如 web 服务和 Trojan 服务）
    # 请使那些服务监听在其他本地端口（这里使用了 4000）
    # 所有不匹配上方 SNI 的 TLS 请求都会转发至此端口，如不需要可以删除此行
    default           localhost:4000;
  }

  server {
    listen      443; # 监听 443 端口
    proxy_pass  $name;
    ssl_preread on;
  }
}
```

### 配置落地 Trojan 服务

在之前的配置中我们使用了一个证书签发了所有目标路径的域名，所以这里我们可以使用一个 Trojan 服务端处理所有目标路径的请求。  
Trojan 的配置和通常配置方法无异，这里还是提供一份例子。无关的配置已省略。

```gojson
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "ssl": {
        "cert": "/path/to/certificate.crt",
        "key": "/path/to/private.key",
    }
    ...
}
```

小提示：如果需要在落地主机上对不同路径分别使用独立的 Trojan 服务端（比如需要分别接入各自的计费服务），可以在落地机上再配置一个 SNI 代理，并分别转发至本地不同的 Trojan 服务端监听端口。由于配置与前面所提到的过程基本相同，这里便不再赘述。

## 总结

通过以上介绍的配置方法，我们可以在单一端口上实现多入口多出口多级中继的 Trojan 流量转发。  
对于多级中继，只需在中间节点上按相同思路配置 SNI 代理即可。


---
title: "使用Shadowsocks插件/可插拔传输层"
draft: false
weight: 7
---

### 注意，Trojan不支持这个特性

Trojan-Go支持可插拔的传输层。原则上，Trojan-Go可以使用任何有TCP隧道功能的软件作为传输层，如v2ray、shadowsocks、kcp等。同时，Trojan-Go也兼容Shadowsocks的SIP003插件标准，如GoQuiet，v2ray-plugin等。也可以使用Tor的传输层插件，如obfs4，meek等。

你可以使用这些插件，替换Trojan-Go的TLS传输层。

开启可插拔传输层插件后，Trojan-Go客户端将会把**流量明文**直接传输给客户端本地的插件处理。由客户端插件负责进行加密和混淆，并将流量传输给服务端的插件。服务端的插件接收到流量，进行解密和解析，将**流量明文**传输给服务端本地的Trojan-Go服务端。

你可以使用任何插件对流量进行加密和混淆，只需添加"transport_plugin"选项，并指定插件的可执行文件的路径，并做好配置即可。

我们更建议**自行设计协议并开发相应插件**。因为目前现有的所有插件无法对接Trojan-Go的对抗主动探测的特性，而且部分插件并无加密能力。如果你对开发插件有兴趣，欢迎在"实现细节和开发指南"一节中查看插件设计的指南。

例如，可以使用符合SIP003标准的v2ray-plugin，下面是一个例子:

**这个配置中使用了websocket明文传输未经加密的trojan协议，存在安全隐患。这个配置仅作为演示使用。**

**不要在任何情况下使用这个配置穿透GFW。**

服务端配置：

```gojson
...（省略）
"transport_plugin": {
    "enabled": true,
    "type": "shadowsocks",
    "command": "./v2ray-plugin",
    "arg": ["-server", "-host", "www.baidu.com"]
}
```

客户端配置：

```gojson
...（省略）
"transport_plugin": {
    "enabled": true,
    "type": "shadowsocks",
    "command": "./v2ray-plugin",
    "arg": ["-host", "www.baidu.com"]
}
```

注意，v2ray-plugin插件需要指定```go-server```参数来区分客户端和服务端。更多关于该插件详细的说明，参考v2ray-plugin的文档。

启动Trojan-Go后，你可以看到v2ray-plugin启动的输出。插件将把流量伪装为Websocket流量并传输。

非SIP003标准的插件可能需要不同的配置，你可以指定```gotype```为"other"，并自行指定插件地址，插件启动参数、环境变量。


---
title: "国内直连和广告屏蔽"
draft: false
weight: 3
---

### 注意，Trojan不支持这个特性

Trojan-Go内建的路由模块可以帮助你实现国内直连，即客户端对于国内网站不经过代理，直接连接。

路由模块在客户端可以配置三种策略(```gobypass```, ```goproxy```, ```goblock```)，在服务端只可使用```goblock```策略。

下面是一个例子

```gojson
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_server",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "sni": "your-domain-name.com"
    },
    "mux" :{
        "enabled": true
    },
    "router":{
        "enabled": true,
        "bypass": [
            "geoip:cn",
            "geoip:private",
            "geosite:cn",
            "geosite:geolocation-cn"
        ],
        "block": [
            "geosite:category-ads"
        ],
        "proxy": [
            "geosite:geolocation-!cn"
        ]
    }
}
```

这个配置文件激活了router模块，使用的是白名单的模式，当匹配到中国大陆或者局域网的IP/域名时，直接连接。如果是广告运营商的域名，则直接断开连接。

所需要的数据库```gogeoip.dat```和```gogeosite.dat```已经包含在release的压缩包中，直接使用即可。它们来自V2Ray的[domain-list-community](https://github.com/v2fly/domain-list-community)和[geoip](https://github.com/v2fly/geoip)。

你可以使用如```gogeosite:cn```、```gogeosite:geolocation-!cn```、```gogeosite:category-ads-all```、```gogeosite:bilibili```的形式来指定某一类域名，所有可用的tag可以在[domain-list-community](https://github.com/v2fly/domain-list-community)仓库的[```godata```](https://github.com/v2fly/domain-list-community/tree/master/data)目录中找到。```gogeosite.dat``` 更详细使用说明，参考[V2Ray/Routing路由#预定义域名列表](https://www.v2fly.org/config/routing.html#预定义域名列表)。

你可以使用如```gogeoip:cn```、```gogeoip:hk```、```gogeoip:us```、```gogeoip:private```的形式来指定某一类IP。`geoip:private`为特殊项，囊括了内网IP和保留IP，其余类别囊括了各个国家/地区的IP地址段。各国家/地区的代号参考[维基百科](https://zh.wikipedia.org/wiki/%E5%9C%8B%E5%AE%B6%E5%9C%B0%E5%8D%80%E4%BB%A3%E7%A2%BC)。

你也可以配置自己的路由规则。例如，想要屏蔽所有example.com域名以及其子域名，以及192.168.1.0/24，添加下面的规则。

```gojson
"block": [
    "domain:example.com",
    "cidr:192.168.1.0/24"
]
```

支持的格式有

- "domain:"，子域名匹配

- "full:"，完全域名匹配

- "regexp:"，正则表达式匹配

- "cidr:"，CIDR匹配

更详细的说明参考"完整的配置文件"一节。


---
title: "使用Websocket进行CDN转发和抵抗中间人攻击"
draft: false
weight: 2
---

### 注意，Trojan不支持这个特性

Trojan-Go支持使用TLS+Websocket承载Trojan协议，使得利用CDN进行流量中转成为可能。

Trojan协议本身不带加密，安全性依赖外层的TLS。但流量一旦经过CDN，TLS对CDN是透明的。其服务提供者可以对TLS的明文内容进行审查。**如果你使用的是不可信任的CDN（任何在中国大陆注册备案的CDN服务均应被视为不可信任），请务必开启Shadowsocks AEAD对Webosocket流量进行加密，以避免遭到识别和审查。**

服务器和客户端配置文件中同时添加websocket选项，并将其```goenabled```字段设置为true，并填写```gopath```字段和```gohost```字段即可启用Websocket支持。下面是一个完整的Websocket选项:

```gojson
"websocket": {
    "enabled": true,
    "path": "/your-websocket-path",
    "host": "example.com"
}
```

```gohost```是主机名，一般填写域名。客户端```gohost```是可选的，填写你的域名。如果留空，将会使用```goremote_addr```填充。

```gopath```指的是websocket所在的URL路径，必须以斜杠("/")开始。路径并无特别要求，满足URL基本格式即可，但要保证客户端和服务端的```gopath```一致。```gopath```应当选择较长的字符串，以避免遭到GFW直接的主动探测。

客户端的```gohost```将包含在Websocket的握手HTTP请求中，发送给CDN服务器，必须有效；服务端和客户端```gopath```必须一致，否则Websocket握手无法进行。

下面是一个客户端配置文件的例子

```gojson
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "www.your_awesome_domain_name.com",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "websocket": {
        "enabled": true,
        "path": "/your-websocket-path",
        "host": "example.com"
    },
    "shadowsocks": {
        "enabled": true,
        "password": "12345678"
    }
}
```


---
title: "高级配置"
draft: false
weight: 30
---

这一部分内容将介绍更复杂的Trojan-Go配置方法。对于与Trojan原版不兼容的特性，将在小节中明确标明。


---
title: "正确配置Trojan-Go"
draft: false
weight: 22
---

下面将介绍如何正确配置Trojan-Go以完全隐藏你的代理节点特征。

在开始之前，你需要

- 一个服务器，且未被GFW封锁

- 一个域名，可以使用免费的域名服务，如.tk等

- Trojan-Go，可以从release页面下载

- 证书和密钥，可以从letsencrypt等机构免费申请签发

### 服务端配置

我们的目标是，使得你的服务器和正常的HTTPS网站表现相同。

首先你需要一个HTTP服务器，可以使用nginx，apache，caddy等配置一个本地HTTP服务器，也可以使用别人的HTTP服务器。HTTP服务器的作用是，当GFW主动探测时，向它展示一个完全正常的Web页面。

**你需要在```goremote_addr```和```goremote_port```指定这个HTTP服务器的地址。```goremote_addr```可以是IP或者域名。Trojan-Go将会测试这个HTTP服务器是否工作正常，如果不正常，Trojan-Go会拒绝启动。**

下面是一份比较安全的服务器配置server.json，需要你在本地80端口配置一个HTTP服务（必要，你也可以使用其他的网站HTTP服务器，如"remote_addr": "example.com"），在1234端口配置一个HTTPS服务，或是一个展示"400 Bad Request"的静态HTTP网页服务。（可选，可以删除```gofallback_port```字段，跳过这个步骤）

```gojson
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "your_awesome_password"
    ],
    "ssl": {
        "cert": "server.crt",
        "key": "server.key",
        "fallback_port": 1234
    }
}
```

这个配置文件使Trojan-Go在服务器的所有IP地址上(0.0.0.0)监听443端口，分别使用server.crt和server.key作为证书和密钥进行TLS握手。你应该使用尽可能复杂的密码，同时确保客户端和服务端```gopassword```是一致的。注意，**Trojan-Go会检测你的HTTP服务器```gohttp://remote_addr:remote_port```是否正常工作。如果你的HTTP服务器工作不正常，Trojan-Go将拒绝启动。**

当一个客户端试图连接Trojan-Go的监听端口时，会发生下面的事情：

- 如果TLS握手成功，检测到TLS的内容非Trojan协议（有可能是HTTP请求，或者来自GFW的主动探测）。Trojan-Go将TLS连接代理到本地127.0.0.1:80上的HTTP服务。这时在远端看来，Trojan-Go服务就是一个HTTPS网站。

- 如果TLS握手成功，并且被确认是Trojan协议头部，并且其中的密码正确，那么服务器将解析来自客户端的请求并进行代理，否则和上一步的处理方法相同。

- 如果TLS握手失败，说明对方使用的不是TLS协议进行连接。此时Trojan-Go将这个TCP连接代理到本地127.0.0.1:1234上运行的HTTPS服务（或者HTTP服务），返回一个展示400 Bad Reqeust的HTTP页面。```gofallback_port```是一个可选选项，如果没有填写，Trojan-Go会直接终止连接。虽然是可选的，但是还是强烈建议填写。

你可以通过使用浏览器访问你的域名```gohttps://your-domain-name.com```来验证。如果工作正常，你的浏览器会显示一个正常的HTTPS保护的Web页面，页面内容与服务器本机80端口上的页面一致。你还可以使用```gohttp://your-domain-name.com:443```验证```gofallback_port```工作是否正常。

事实上，你甚至可以将Trojan-Go当作你的HTTPS服务器，用来给你的网站提供HTTPS服务。访客可以正常地通过Trojan-Go浏览你的网站，而和代理流量互不影响。但是注意，不要在```goremote_port```和```gofallback_port```搭建有高实时性需求的服务，Trojan-Go识别到非Trojan协议流量时会有意增加少许延迟以抵抗GFW基于时间的检测。

配置完成后，可以使用

```goshell
./trojan-go -config ./server.json
```

启动服务端。

### 客户端配置

对应的客户端配置client.json

```gojson
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_awesome_server",
    "remote_port": 443,
    "password": [
        "your_awesome_password"
    ],
    "ssl": {
        "sni": "your-domain-name.com"
    }
}
```

这个客户端配置使Trojan-Go开启一个监听在本地1080端口的socks5/http代理（自动识别），远端服务器为your_awesome_server:443，your_awesome_server可以是IP或者域名。

如果你在```goremote_addr```中填写的是域名，```gosni```可以省略。如果你在```goremote_addr```填写的是IP地址，```gosni```字段应当填写你申请证书的对应域名，或者你自己签发的证书的Common Name，而且必须一致。注意，```gosni```字段目前的在TLS协议中是**明文传送**的(目的是使服务器提供相应证书)。GFW已经被证实具有SNI探测和阻断能力，所以不要填写类似```gogoogle.com```等已经被封锁的域名，否则很有可能导致你的服务器也被遭到封锁。

配置完成后，可以使用

```goshell
./trojan-go -config ./client.json
```

启动客户端。

更多关于配置文件的信息，可以在左侧导航栏中找到相应介绍。


---
title: "完整的配置文件"
draft: false
weight: 30
---

下面是一个完整的配置文件，其中的必填选项有

- ```gorun_type```

- ```golocal_addr```

- ```golocal_port```

- ```goremote_addr```

- ```goremote_port```

对于服务器```goserver```，```gokey```和```gocert```为必填。

对于客户端```goclient```，反向代理隧道```goforward```，以及透明代理```gonat```，```gopassword```必填

其余未填的选项，用下面给出的值进行填充。

*Trojan-Go支持对人类更友好的YAML语法，配置文件的基本结构与JSON相同，效果等价。但是为了遵守YAML的命名习惯，你需要把下划线("_")转换为横杠("-")，如```goremote_addr```在YAML文件中为```goremote-addr```*

```gojson
{
  "run_type": *required*,
  "local_addr": *required*,
  "local_port": *required*,
  "remote_addr": *required*,
  "remote_port": *required*,
  "log_level": 1,
  "log_file": "",
  "password": [],
  "disable_http_check": false,
  "udp_timeout": 60,
  "ssl": {
    "verify": true,
    "verify_hostname": true,
    "cert": *required*,
    "key": *required*,
    "key_password": "",
    "cipher": "",
    "curves": "",
    "prefer_server_cipher": false,
    "sni": "",
    "alpn": [
      "http/1.1"
    ],
    "session_ticket": true,
    "reuse_session": true,
    "plain_http_response": "",
    "fallback_addr": "",
    "fallback_port": 0,
    "fingerprint": ""
  },
  "tcp": {
    "no_delay": true,
    "keep_alive": true,
    "prefer_ipv4": false
  },
  "mux": {
    "enabled": false,
    "concurrency": 8,
    "idle_timeout": 60
  },
  "router": {
    "enabled": false,
    "bypass": [],
    "proxy": [],
    "block": [],
    "default_policy": "proxy",
    "domain_strategy": "as_is",
    "geoip": "$PROGRAM_DIR$/geoip.dat",
    "geosite": "$PROGRAM_DIR$/geosite.dat"
  },
  "websocket": {
    "enabled": false,
    "path": "",
    "host": ""
  },
  "shadowsocks": {
    "enabled": false,
    "method": "AES-128-GCM",
    "password": ""
  },
  "transport_plugin": {
    "enabled": false,
    "type": "",
    "command": "",
    "option": "",
    "arg": [],
    "env": []
  },
  "forward_proxy": {
    "enabled": false,
    "proxy_addr": "",
    "proxy_port": 0,
    "username": "",
    "password": ""
  },
  "mysql": {
    "enabled": false,
    "server_addr": "localhost",
    "server_port": 3306,
    "database": "",
    "username": "",
    "password": "",
    "check_rate": 60
  },
  "api": {
    "enabled": false,
    "api_addr": "",
    "api_port": 0,
    "ssl": {
      "enabled": false,
      "key": "",
      "cert": "",
      "verify_client": false,
      "client_cert": []
    }
  }
}
```

## 说明

### 一般选项

对于client/nat/forward，```goremote_xxxx```应当填写你的trojan服务器地址和端口号，```golocal_xxxx```对应本地开放的socks5/http代理地址（自动适配）

对于server，```golocal_xxxx```对应trojan服务器监听地址（强烈建议使用443端口），```goremote_xxxx```填写识别到非trojan流量时代理到的HTTP服务地址，通常填写本地80端口。

```golog_level```指定日志等级。等级越高，输出的信息越少。合法的值有

- 0 输出Debug以上日志（所有日志）

- 1 输出Info及以上日志

- 2 输出Warning及以上日志

- 3 输出Error及以上日志

- 4 输出Fatal及以上日志

- 5 完全不输出日志

```golog_file```指定日志输出文件路径。如果未指定则使用标准输出。

```gopassword```可以填入多个密码。除了使用配置文件配置密码之外，trojan-go还支持使用mysql配置密码，参见下文。客户端的密码，只有与服务端配置文件中或者在数据库中的密码记录一致，才能通过服务端的校验，正常使用代理服务。

```godisable_http_check```是否禁用HTTP伪装服务器可用性检查。

```goudp_timeout``` UDP会话超时时间。

### ```gossl```选项

```goverify```表示客户端(client/nat/forward)是否校验服务端提供的证书合法性，默认开启。出于安全性考虑，这个选项不应该在实际场景中选择false，否则可能遭受中间人攻击。如果使用自签名或者自签发的证书，开启```goverify```会导致校验失败。这种情况下，应当保持```goverify```开启，然后在```gocert```中填写服务端的证书，即可正常连接。

```goverify_hostname```表示服务端是否校验客户端提供的SNI与服务端设置的一致性。如果服务端SNI字段留空，认证将被强制关闭。

服务端必须填入```gocert```和```gokey```，对应服务器的证书和私钥文件，请注意证书是否有效/过期。如果使用权威CA签发的证书，客户端(client/nat/forward)可以不填写```gocert```。如果使用自签名或者自签发的证书，应当在的```gocert```处填入服务器证书文件，否则可能导致校验失败。

```gosni```指的是TLS客户端请求中的服务器名字段，一般和证书的Common Name相同。如果你使用let'sencrypt等机构签发的证书，这里填入你的域名。对于客户端，如果这一项未填，将使用```goremote_addr```填充。你应当指定一个有效的SNI（和远端证书CN一致），否则客户端可能无法验证远端证书有效性从而无法连接；对于服务端，若此项不填，则使用证书中Common Name作为SNI校验依据，支持通配符如*.example.com。

```gofingerprint```用于指定客户端TLS Client Hello指纹伪造类型，以抵抗GFW对于TLS Client Hello指纹的特征识别和阻断。trojan-go使用[utls](https://github.com/refraction-networking/utls)进行指纹伪造，默认伪造Firefox的指纹。合法的值有

- ""，不使用指纹伪造（默认）

- "firefox"，伪造Firefox指纹

- "chrome"，伪造Chrome指纹

- "ios"，伪造iOS指纹

一旦指纹的值被设置，客户端的```gocipher```，```gocurves```，```goalpn```，```gosession_ticket```等有可能影响指纹的字段将使用该指纹的特定设置覆写。
```goalpn```为TLS的应用层协议协商指定协议。在TLS Client/Server Hello中传输，协商应用层使用的协议，仅用作指纹伪造，并无实际作用。**如果使用了CDN，错误的alpn字段可能导致与CDN协商得到错误的应用层协议**。

```goprefer_server_cipher```客户端是否偏好选择服务端在协商中提供的密码学套件。

```gocipher```TLS使用的密码学套件。```gocipher13``字段与此字段合并。只有在你明确知道自己在做什么的情况下，才应该去填写此项以修改trojan-go使用的TLS密码学套件。**正常情况下，你应该将其留空或者不填**，trojan-go会根据当前硬件平台以及远端的情况，自动选择最合适的加密算法以提升性能和安全性。如果需要填写，密码学套件名用分号(":")分隔，按优先顺序排列。Go的TLS库中弃用了TLS1.2中部分不安全的密码学套件，并完全支持TLS1.3。默认情况下，trojan-go将优先使用更安全的TLS1.3。

```curves```go指定TLS在ECDHE中偏好使用的椭圆曲线。只有你明确知道自己在做什么的情况下，才应该填写此项。曲线名称用分号(":")分隔，按优先顺序排列。

```plain_http_response```go指服务端TLS握手失败时，明文发送的原始数据（原始TCP数据）。这个字段填入该文件路径。推荐使用```fallback_port```go而不是该字段。

```fallback_addr```go和```fallback_port```go指服务端TLS握手失败时，trojan-go将该连接重定向到该地址。这是trojan-go的特性，以便更好地隐蔽服务器，抵抗GFW的主动检测，使得服务器的443端口在遭遇非TLS协议的探测时，行为与正常服务器完全一致。当服务器接受了一个连接但无法进行TLS握手时，如果```fallback_port```go不为空，则流量将会被代理至fallback_addr:fallback_port。如果```fallback_addr```go为空，则用```remote_addr```go填充。例如，你可以在本地使用nginx开启一个https服务，当你的服务器443端口被非TLS协议请求时（比如http请求），trojan-go将代理至本地https服务器，nginx将使用http协议明文返回一个400 Bad Request页面。你可以通过使用浏览器访问```http://your-domain-name.com:443```go进行验证。

```key_log```goTLS密钥日志的文件路径。如果填写则开启密钥日志。**记录密钥将破坏TLS的安全性，此项不应该用于除调试以外的其他任何用途。**

### ```mux```go多路复用选项

多路复用是trojan-go的特性。如果服务器和客户端都是trojan-go，可以开启mux多路复用以减少高并发情景下的延迟（只需要客户端开启此选项即可，服务端自动适配）。

注意，多路复用的意义在于降低握手延迟，而不是提升链路速度。相反，它会增加客户端和服务端的CPU和内存消耗，从而可能造成速度下降。

```enabled```go是否开启多路复用。

```concurrency```go指单个TLS隧道可以承载的最大连接数，默认为8。这个数值越大，多连接并发时TLS由于握手产生的延迟就越低，但网络吞吐量可能会有所降低，填入负数或者0表示所有连接只使用一个TLS隧道承载。

```idle_timeout```go空闲超时时间。指TLS隧道在空闲多长时间之后关闭，单位为秒。如果数值为负值或0，则一旦TLS隧道空闲，则立即关闭。

### ```router```go路由选项

路由功能是trojan-go的特性。trojan-go的路由策略有三种。

- Proxy 代理。将请求通过TLS隧道进行代理，由trojan服务器和目的地址进行连接。

- Bypass 绕过。直接在本地和目的地址进行连接。

- Block 封锁。不代理请求，直接关闭连接。

在```proxy```go, ```bypass```go, ```block```go字段中填入对应列表geoip/geosite或路由规则，trojan-go即根据列表中的IP（CIDR）或域名执行相应路由策略。客户端(client)可以配置三种策略，服务端(server)只可配置block策略。

```enabled```go是否开启路由模块。

```default_policy```go指的是三个列表匹配均失败后，使用的默认策略，默认为"proxy"，即进行代理。合法的值有

- "proxy"

- "bypass"

- "block"

含义同上。

```domain_strategy```go域名解析策略，默认"as_is"。合法的值有：

- "as_is"，只在各列表中的域名规则内进行匹配。

- "ip_if_non_match"，先在各列表中的域名规则内进行匹配；如果不匹配，则解析为IP后，在各列表中的IP地址规则内进行匹配。该策略可能导致DNS泄漏或遭到污染。

- "ip_on_demand"，先解析为IP，在各列表中的IP地址规则内进行匹配；如果不匹配，则在各列表中的域名规则内进行匹配。该策略可能导致DNS泄漏或遭到污染。

```geoip```go和```geosite```go字段指geoip和geosite数据库文件路径，默认使用程序所在目录的geoip.dat和geosite.dat。也可以通过指定环境变量TROJAN_GO_LOCATION_ASSET指定工作目录。

### ```websocket```go选项

Websocket传输是trojan-go的特性。在**正常的直接连接代理节点**的情况下，开启这个选项不会改善你的链路速度（甚至有可能下降），也不会提升你的连接安全性。你只应该在需要利用CDN进行中转，或利用nginx等服务器根据路径分发的情况下，使用websocket。

```enabled```go表示是否启用Websocket承载流量，服务端开启后同时支持一般Trojan协议和基于websocket的Trojan协议，客户端开启后将只使用websocket承载所有Trojan协议流量。

```path```go指的是Websocket使用的URL路径，必须以斜杠("/")开头，如"/longlongwebsocketpath"，并且服务器和客户端必须一致。

```host```goWebsocket握手时，HTTP请求中使用的主机名。客户端如果留空则使用```remote_addr```go填充。如果使用了CDN，这个选项一般填入域名。不正确的```host```go可能导致CDN无法转发请求。

### ``shadowsocks`` AEAD加密选项

此选项用于替代弃用的混淆加密和双重TLS。如果此选项被设置启用，Trojan协议层下将插入一层Shadowsocks AEAD加密层。也即（已经加密的）TLS隧道内，所有的Trojan协议将再使用AEAD方法进行加密。注意，此选项和Websocket是否开启无关。无论Websocket是否开启，所有Trojan流量都会被再进行一次加密。

注意，开启这个选项将有可能降低传输性能，你只应该在不信任承载Trojan协议的传输信道的情况下，启用这个选项。例如：

- 你使用了Websocket，经过不可信的CDN进行中转（如国内CDN）

- 你与服务器的连接遭到了GFW针对TLS的中间人攻击

- 你的证书失效，无法验证证书有效性

- 你使用了无法保证密码学安全的可插拔传输层

等等。

由于使用的是AEAD，trojan-go可以正确判断请求是否有效，是否遭到主动探测，并作出相应的响应。

```enabled```go是否启用Shadowsocks AEAD加密Trojan协议层。

```method```go加密方式。合法的值有：

- "CHACHA20-IETF-POLY1305"

- "AES-128-GCM" (默认)

- "AES-256-GCM"

```password```go用于生成主密钥的密码。如果启用AEAD加密，必须确保客户端和服务端一致。

### ```transport_plugin```go传输层插件选项

```enabled```go是否启用传输层插件替代TLS传输。一旦启用传输层插件支持，trojan-go将会把**未经TLS加密的trojan协议流量明文传输给插件**，以允许用户对流量进行自定义的混淆和加密。

```type```go插件类型。目前支持的类型有

- "shadowsocks"，支持符合[SIP003](https://shadowsocks.org/en/spec/Plugin.html)标准的shadowsocks混淆插件。trojan-go将在启动时按照SIP003标准替换环境变量并修改自身配置(```remote_addr/remote_port/local_addr/local_port```go)，使插件与远端直接通讯，而trojan-go仅监听/连接插件。

- "plaintext"，使用明文传输。选择此项，trojan-go不会修改任何地址配置(```remote_addr/remote_port/local_addr/local_port```go)，也不会启动```command```go中插件，仅移除最底层的TLS传输层并使用TCP明文传输。此选项目的为支持nginx等接管TLS并进行分流，以及高级用户进行调试测试。**请勿直接使用明文传输模式穿透防火墙。**

- "other"，其他插件。选择此项，trojan-go不会修改任何地址配置(```remote_addr/remote_port/local_addr/local_port```go)，但会启动```command```go中插件并传入参数和环境变量。

```command```go传输层插件可执行文件的路径。trojan-go将在启动时一并执行它。

```arg```go传输层插件启动参数。这是一个列表，例如```["-config", "test.json"]```go。

```env```go传输层插件环境变量。这是一个列表，例如```["VAR1=foo", "VAR2=bar"]```go。

```option```go传输层插件配置（SIP003)。例如```"obfs=http;obfs-host=www.baidu.com"```go。

### ```tcp```go选项

```no_delay```goTCP封包是否直接发出而不等待缓冲区填满。

```keep_alive```go是否启用TCP心跳存活检测。

```prefer_ipv4```go是否优先使用IPv4地址。

### ```mysql```go数据库选项

trojan-go兼容trojan的基于mysql的用户管理方式，但更推荐的方式是使用API。

```enabled```go表示是否启用mysql数据库进行用户验证。

```check_rate```go是trojan-go从MySQL获取用户数据并更新缓存的间隔时间，单位为秒。

其他选项可以顾名思义，不再赘述。

users表结构和trojan版本定义一致，下面是一个创建users表的例子。注意这里的password指的是密码经过SHA224散列之后的值（字符串），流量download, upload, quota的单位是字节。你可以通过修改数据库users表中的用户记录的方式，添加和删除用户，或者指定用户的流量配额。trojan-go会根据所有的用户流量配额，自动更新当前有效的用户列表。如果download+upload>quota，trojan-go服务器将拒绝该用户的连接。

```mysql
CREATE TABLE users (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    username VARCHAR(64) NOT NULL,
    password CHAR(56) NOT NULL,
    quota BIGINT NOT NULL DEFAULT 0,
    download BIGINT UNSIGNED NOT NULL DEFAULT 0,
    upload BIGINT UNSIGNED NOT NULL DEFAULT 0,
    PRIMARY KEY (id),
    INDEX (password)
);
```go

### ```forward_proxy```go前置代理选项

前置代理选项允许使用其他代理承载trojan-go的流量

```enabled```go是否启用前置代理(socks5)。

```proxy_addr```go前置代理的主机地址。

```proxy_port```go前置代理的端口号。

```username```go ```password```go代理的用户和密码，如果留空则不使用认证。

### ```api```go选项

trojan-go基于gRPC提供了API，以支持服务端和客户端的管理和统计。可以实现客户端的流量和速度统计，服务端各用户的流量和速度统计，用户的动态增删和限速等。

```enabled```go是否启用API功能。

```api_addr```gogRPC监听的地址。

```api_port```gogRPC监听的端口。

```ssl```go TLS相关设置。

- ```enabled```go是否使用TLS传输gRPC流量。

- ```key```go，```cert```go服务器私钥和证书。

- ```verify_client```go是否认证客户端证书。

- ```client_cert```如果开启客户端认证，此处填入认证的客户端证书列表。

警告：**不要将未开启TLS双向认证的API服务直接暴露在互联网上，否则可能导致各类安全问题。**
