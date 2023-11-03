# v2ray-core源码解析 12

# `app/reverse/errors.generated.go`

这段代码定义了一个名为"reverse"的包，其中包含一个名为"errPathObjHolder"的结构体，以及一个名为"newError"的函数。

函数的参数是一个或多个接口类型的变量，例如"string"、"int"、"bool"等等。这些变量将组成一个值，该值被传递给函数内部的"errors.New"函数，并进一步传递给"errPathObjHolder"结构体中的"WithPathObj"方法，最后返回一个带有错误路径对象的"errors.Error"类型。

函数的作用是创建一个表示错误信息的对象，其中包含了错误消息和错误细节，如错误类型、错误路径、错误信息等。该对象可以被传递给调用方，例如在应用程序中处理错误的函数中，可以使用该函数来返回一个带有错误信息的对象，从而使代码更加健壮和易于维护。


```go
package reverse

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `app/reverse/portal.go`

这段代码是一个 Go 语言编写的package 名称，其中包含了以下内容：

1. `+build !confonly`：这是一个构建模式，其中`+build`表示编译时编译，`!confonly`表示不要在运行时执行任何代码。

2. `package reverse`：定义了一个名为`reverse`的包。

3. `import (`：导入了一些外部依赖项，包括来自`github.com/golang/protobuf/proto`的`protoc`命令，来自`v2ray.com/core/common`的`buf`、`net`、`session`、`task`、`transport`和`features/outbound`包，以及来自`outbound`的`transport`和`features/outbound`包。

4. `package.Progress`：定义了一个名为`PackageProgress`的接口，该接口用于在 `reverse` 包的构建过程中进行实时进度监控。

5. `Run`函数：该函数用于构建 `reverse` 包。

6. `Run时`：该函数用于运行时监控 `reverse` 包的构建过程。

7. `func`：该函数名为`Runtime`。

8. `const`：该函数创建了一个名为`HALF_SECONDS`的常量，值为`30`。

9. `func`：该函数创建了一个名为`CONCURRENT_TCP_ESTABLISH`的常量，值为`false`。

10. `const`：该函数创建了一个名为`TCP_PORT`的常量，值为`12345`。

11. `const`：该函数创建了一个名为`TCP_BUF_SIZE`的常量，值为`1024`。

12. `const`：该函数创建了一个名为`BUFFER_SIZE`的常量，值为`1024`。

13. `const`：该函数创建了一个名为`MAX_FILE_SIZE`的常量，值为`1048576`。

14. `const`：该函数创建了一个名为`THREAD_POOL_SIZE`的常量，值为`16`。

15. `const`：该函数创建了一个名为`CHECK_MAX_MEMORY_USAGE`的常量，值为`false`。

16. `func`：该函数创建了一个名为`Routine`的内部函数。

17. `func`：该函数创建了一个名为`Status`的内部函数。

18. `func`：该函数创建了一个名为`Join`的内部函数。

19. `func`：该函数创建了一个名为`Process`的内部函数。

20. `func`：该函数创建了一个名为`Progress`的内部函数。

21. `func`：该函数创建了一个名为`Stringify`的内部函数。

22. `func`：该函数创建了一个名为`UnaryGet`的内部函数。

23. `func`：该函数创建了一个名为`UnarySet`的内部函数。

24. `func`：该函数创建了一个名为`AtomicUint16`的内部函数。

25. `func`：该函数创建了一个名为`Count`的内部函数。

26. `func`：该函数创建了一个名为`Timed`的内部函数。

27. `func`：该函数创建了一个名为`Tradecount`的内部函数。

28. `func`：该函数创建了一个名为`Exporter`的内部函数。

29. `func`：该函数创建了一个名为`F向北`的内部函数。

30. `func`：该函数创建了一个名为`N狐仙`的内部函数。

31. `func`：该函数创建了一个名为`reverse加班线`的内部函数。


```go
// +build !confonly

package reverse

import (
	"context"
	"sync"
	"time"

	"github.com/golang/protobuf/proto"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/mux"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/task"
	"v2ray.com/core/features/outbound"
	"v2ray.com/core/transport"
	"v2ray.com/core/transport/pipe"
)

```

这段代码定义了一个名为 `Portal` 的 struct 类型，它包含以下字段：

- `ohm`：是一个 `outbound.Manager` 类型的字段，它用于存储一个 Outbound 客户端的实例。
- `tag`：是一个字符串类型的字段，用于存储门户的标签。
- `domain`：是一个字符串类型的字段，用于存储门户的域名。
- `picker`：是一个指向 `StaticMuxPicker` 类型的字段，用于存储一个静态均衡器的实例。
- `client`：是一个指向 `mux.ClientManager` 类型的字段，用于存储一个用于处理客户端请求的客户端经理。

另外，该代码还包含两个函数：

- `NewPortal`：该函数接受一个 `PortalConfig` 类型的参数，并返回一个指向 `Portal` 类型的引用。如果 `PortalConfig` 中的一些字段是空的，该函数会返回一个空引用。
- `fmt.Println`：该函数用于将 `Portal` 类型的实例打印为字符串格式。


```go
type Portal struct {
	ohm    outbound.Manager
	tag    string
	domain string
	picker *StaticMuxPicker
	client *mux.ClientManager
}

func NewPortal(config *PortalConfig, ohm outbound.Manager) (*Portal, error) {
	if config.Tag == "" {
		return nil, newError("portal tag is empty")
	}

	if config.Domain == "" {
		return nil, newError("portal domain is empty")
	}

	picker, err := NewStaticMuxPicker()
	if err != nil {
		return nil, err
	}

	return &Portal{
		ohm:    ohm,
		tag:    config.Tag,
		domain: config.Domain,
		picker: picker,
		client: &mux.ClientManager{
			Picker: picker,
		},
	}, nil
}

```

这段代码定义了两个函数：`func (p *Portal) Start() error` 和 `func (p *Portal) Close() error`。

这两个函数都接受一个指向 `Portal` 类型实体的 `p` 参数。

函数 `func (p *Portal) Start() error` 接受一个 `Outbound` 类型的参数 `Outbound`。这个参数是一个 `transport.Link` 类型的引用，代表一个网络连接。函数的作用是在 `p` 的 `Outbound` 连接上开始监听数据流。函数返回一个 `error` 类型的结果，表示开始监听数据流时是否发生错误。

函数 `func (p *Portal) Close() error` 接受一个 `transport.Link` 类型的参数 `Outbound`。这个参数是一个 `transport.Link` 类型的引用，代表一个网络连接。函数的作用是在 `p` 的 `Outbound` 连接上关闭连接。函数返回一个 `error` 类型的结果，表示关闭连接时是否发生错误。

函数 `func (p *Portal) HandleConnection(ctx context.Context, link *transport.Link) error` 是 `HandleConnection` 函数的实现。它接收一个 `transport.Context` 类型的参数 `ctx` 和一个 `transport.Link` 类型的参数 `link`。函数的作用是在 `p` 的 `Outbound` 连接上处理数据流。函数首先获取一个 `Outbound` 类型的 `OutboundMeta` 参数，这个参数来自于 `OutboundFromContext` 函数。如果 `OutboundMeta` 是 `nil`，那么函数返回一个错误，这个错误说明数据流没有被正确处理。如果 `OutboundMeta` 不是 `nil`，那么函数创建一个 `mux.Client` 类型的客户端，这个客户端会用来创建一个 `transport.Mux` 类型的 `transport.Link` 对象。然后函数会尝试使用 `NewPortalWorker` 函数来创建一个 `PortalWorker` 类型的 `transport.Link` 对象，这个 `PortalWorker` 对象会被添加到 `p` 的 `picker` 数组中。最后函数会使用 `p.client.Dispatch` 函数来处理数据流，这个函数会将数据流发送到 `link` 所代表的网络连接。


```go
func (p *Portal) Start() error {
	return p.ohm.AddHandler(context.Background(), &Outbound{
		portal: p,
		tag:    p.tag,
	})
}

func (p *Portal) Close() error {
	return p.ohm.RemoveHandler(context.Background(), p.tag)
}

func (p *Portal) HandleConnection(ctx context.Context, link *transport.Link) error {
	outboundMeta := session.OutboundFromContext(ctx)
	if outboundMeta == nil {
		return newError("outbound metadata not found").AtError()
	}

	if isDomain(outboundMeta.Target, p.domain) {
		muxClient, err := mux.NewClientWorker(*link, mux.ClientStrategy{})
		if err != nil {
			return newError("failed to create mux client worker").Base(err).AtWarning()
		}

		worker, err := NewPortalWorker(muxClient)
		if err != nil {
			return newError("failed to create portal worker").Base(err)
		}

		p.picker.AddWorker(worker)
		return nil
	}

	return p.client.Dispatch(ctx, link)
}

```

该代码定义了一个名为 `Outbound` 的结构体，它包含一个名为 `portal` 的整数类型和一个名为 `tag` 的字符串类型。

接下来，该结构体实现了一个名为 `Tag` 的方法，该方法返回其 `tag` 成员的值。

然后，该结构体实现了一个名为 `Dispatch` 的方法，该方法接收一个名为 `link` 的传输 `Link` 对象和一个名为 `ctx` 的上下文参数。该方法首先检查 `portal` 是否与 `link` 中的 `Writer` 字段兼容，然后调用 `portal.HandleConnection` 处理连接。如果在过程中出现错误，则将其记录在日志中，并停止 `link.Writer` 的写入。最后，该方法使用 `common.Interrupt` 停止链路的发送，并返回。


```go
type Outbound struct {
	portal *Portal
	tag    string
}

func (o *Outbound) Tag() string {
	return o.tag
}

func (o *Outbound) Dispatch(ctx context.Context, link *transport.Link) {
	if err := o.portal.HandleConnection(ctx, link); err != nil {
		newError("failed to process reverse connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
		common.Interrupt(link.Writer)
	}
}

```



该代码定义了一个名为 `StaticMuxPicker` 的类，以及两个名为 `func` 的函数，分别用于开始和关闭静态选路器的作用。

此外，还定义了一个名为 `StaticMuxPicker` 的类型，该类型有一个名为 `StaticMuxPicker` 的结构体，该结构体包含一个指向 `Outbound` 类型的指针 `o`，以及一个 ` workers` 字段，该字段是一个数组，包含 `Outbound` 类型类型的指针。

最后，定义了一个名为 `NewStaticMuxPicker` 的函数，该函数返回一个指向 `StaticMuxPicker` 类型的指针，该指针包含一个指向 `Outbound` 类型的指针 `o`，以及一个指向 `task.Periodic` 类型的指针 `cTask`，该指针表示周期性任务。函数内部创建了一个 `StaticMuxPicker` 类型的实例，并设置其 `cTask` 字段的值为一个新的 `task.Periodic` 类型的实例，该实例的 `Execute` 字段设置为 `p.cleanup` 函数，`Interval` 字段设置为每 30 秒执行一次。最后，函数返回实例的指针。


```go
func (o *Outbound) Start() error {
	return nil
}

func (o *Outbound) Close() error {
	return nil
}

type StaticMuxPicker struct {
	access  sync.Mutex
	workers []*PortalWorker
	cTask   *task.Periodic
}

func NewStaticMuxPicker() (*StaticMuxPicker, error) {
	p := &StaticMuxPicker{}
	p.cTask = &task.Periodic{
		Execute:  p.cleanup,
		Interval: time.Second * 30,
	}
	p.cTask.Start()
	return p, nil
}

```

此代码是一个名为`cleanup`的函数，接受一个名为`StaticMuxPicker`的指针变量`p`作为参数。该函数的作用是清理`StaticMuxPicker`结构体中所有工作进程，使其不再占用内存。

具体来说，函数首先对`p.access`进行锁定，然后进行一个循环遍历所有`StaticMuxPicker`结构体中的工作进程。对于每个工作进程，如果它还没有被关闭，就会将其添加到`activeWorkers`数组中。然后，函数检查`activeWorkers`数组长度是否与`StaticMuxPicker`结构体中的工作进程总数相等。如果不相等，就会将`activeWorkers`中的所有工作进程重置为当前工作进程数。最后，函数返回一个`nil`表示没有错误发生。


```go
func (p *StaticMuxPicker) cleanup() error {
	p.access.Lock()
	defer p.access.Unlock()

	var activeWorkers []*PortalWorker
	for _, w := range p.workers {
		if !w.Closed() {
			activeWorkers = append(activeWorkers, w)
		}
	}

	if len(activeWorkers) != len(p.workers) {
		p.workers = activeWorkers
	}

	return nil
}

```

该函数的目的是选择一个可用的 Mux 客户端工人，并返回其客户端和错误信息。

具体来说，该函数首先检查 Mux 客户端工人的数量是否为 0，如果是，则输出错误并返回。接着，该函数遍历 Mux 客户端工人，对于每个工人，它检查其是否在从客户端连接中正在卸载数据，如果是，则继续遍历下一个工人。

如果遍历完所有工人后，仍然没有找到可用的 Mux 客户端工人，该函数将尝试从正在连接的客户端中选择一个可用的工人。具体来说，该函数将从所有工人中选择一个具有最小客户端连接数的工人，如果选择的工人不能用于客户端连接，则返回错误并输出。

该函数的实现依赖于 Mux 的代码实现，并且会受到 Mux 客户端连接数量和连接状态的影响。


```go
func (p *StaticMuxPicker) PickAvailable() (*mux.ClientWorker, error) {
	p.access.Lock()
	defer p.access.Unlock()

	if len(p.workers) == 0 {
		return nil, newError("empty worker list")
	}

	var minIdx int = -1
	var minConn uint32 = 9999
	for i, w := range p.workers {
		if w.draining {
			continue
		}
		if w.client.ActiveConnections() < minConn {
			minConn = w.client.ActiveConnections()
			minIdx = i
		}
	}

	if minIdx == -1 {
		for i, w := range p.workers {
			if w.IsFull() {
				continue
			}
			if w.client.ActiveConnections() < minConn {
				minConn = w.client.ActiveConnections()
				minIdx = i
			}
		}
	}

	if minIdx != -1 {
		return p.workers[minIdx].client, nil
	}

	return nil, newError("no mux client worker available")
}

```

该函数接收一个指向静态MuxPicker的指针变量p，以及一个指向PortalWorker的指针变量worker。这个函数的作用是添加一个新的worker到静态MuxPicker中。

具体来说，这个函数首先会尝试获取静态MuxPicker的写入锁，如果锁已经被占用，则需要等待一段随机时间后才能进行写入。如果锁没有被占用，就代表可以写入数据到静态MuxPicker中，然后将新的worker添加到p.workers数组中，并返回。

静态MuxPicker是一个基于Prometheus指标系统的MixMaster，可以用来控制台柜、GUI界面等。这个函数的作用是让用户可以添加新的worker到静态MuxPicker中，以便于用户可以自定义portal worker的功能和触发时机。


```go
func (p *StaticMuxPicker) AddWorker(worker *PortalWorker) {
	p.access.Lock()
	defer p.access.Unlock()

	p.workers = append(p.workers, worker)
}

type PortalWorker struct {
	client   *mux.ClientWorker
	control  *task.Periodic
	writer   buf.Writer
	reader   buf.Reader
	draining bool
}

```

该函数`NewPortalWorker`接受一个`mux.ClientWorker`客户端并返回一个`PortalWorker`实例和错误。

该函数创建了一个`pipe.Stack`并设置了一些选项，然后使用`pipe.New`方法创建了两个`pipe.Transport`实例，分别用于读取和写入数据。这些实例都被设置为通过一个具有指定大小限制的`pipe.Option`进行处理。

接下来，函数创建了一个`transport.Context`实例，并使用该上下文创建了一个`transport.Link`实例，该实例将数据从一个接受端口8080的UDP目标通过一个互联网网络发送到目标端口8080。

然后，函数调用`client.Dispatch`方法，并传递一个带有读取和写入数据的`transport.Link`实例，以及一个用于控制连接的`transport.Control`实例。

最后，函数创建了一个`PortalWorker`实例，该实例使用`client`作为其客户端，并使用`downlinkReader`和`uplinkWriter`作为读取和写入数据的管道。此外，函数还设置了一个`control`字段为`&task.Periodic`类型，该类型表示将每隔两个心跳包发送一个控制消息。最后，函数调用`w.control`设置了一个`task.Periodic`实例的`Execute`字段为`w.heartbeat`函数，该函数会定期执行控制消息的发送。

函数返回一个`PortalWorker`实例，如果没有错误，则返回。


```go
func NewPortalWorker(client *mux.ClientWorker) (*PortalWorker, error) {
	opt := []pipe.Option{pipe.WithSizeLimit(16 * 1024)}
	uplinkReader, uplinkWriter := pipe.New(opt...)
	downlinkReader, downlinkWriter := pipe.New(opt...)

	ctx := context.Background()
	ctx = session.ContextWithOutbound(ctx, &session.Outbound{
		Target: net.UDPDestination(net.DomainAddress(internalDomain), 0),
	})
	f := client.Dispatch(ctx, &transport.Link{
		Reader: uplinkReader,
		Writer: downlinkWriter,
	})
	if !f {
		return nil, newError("unable to dispatch control connection")
	}
	w := &PortalWorker{
		client: client,
		reader: downlinkReader,
		writer: uplinkWriter,
	}
	w.control = &task.Periodic{
		Execute:  w.heartbeat,
		Interval: time.Second * 2,
	}
	w.control.Start()
	return w, nil
}

```

该函数名为`func (w *PortalWorker) heartbeat() error`，表示该函数的作用是定期向客户端发送心跳包，以保持连接活性。

具体来说，该函数以下是这样的逻辑：

1. 如果客户端已经关闭，则返回一个错误，因为客户端 worker 已经停止。

2. 如果正在 drain（写入数据到客户端），或者 writer（从客户端接收数据）为空，则返回一个错误，因为已经处理了所有应该处理的消息。

3. 如果客户端的连接数总数大于 256，则将 draining（写入数据到客户端）设置为 true，并设置msg的状态为 Control_DRAIN，以确保所有客户端连接都能够处理心跳包。然后使用一个延迟函数来在发送完消息后关闭 writer（从客户端接收数据），以确保消息发送成功。最后，使用 writer.WriteMultiBuffer() 方法将消息发送到客户端。

4. 使用 proto.Marshal() 函数将 msg 序列化为字节数组，然后使用 common.MergeBytes() 函数将两个字节数组合并成一个字符串。

5. 使用 w.writer.WriteMultiBuffer() 方法，将字符串中的字节数逐个发送到客户端，以确保客户端能够正确接收消息。


```go
func (w *PortalWorker) heartbeat() error {
	if w.client.Closed() {
		return newError("client worker stopped")
	}

	if w.draining || w.writer == nil {
		return newError("already disposed")
	}

	msg := &Control{}
	msg.FillInRandom()

	if w.client.TotalConnections() > 256 {
		w.draining = true
		msg.State = Control_DRAIN

		defer func() {
			common.Close(w.writer)
			common.Interrupt(w.reader)
			w.writer = nil
		}()
	}

	b, err := proto.Marshal(msg)
	common.Must(err)
	mb := buf.MergeBytes(nil, b)
	return w.writer.WriteMultiBuffer(mb)
}

```

这两函数接收一个名为 `w` 的 *PortalWorker 类型的参数，并返回 *w *PortalWorker 类型的结果。

第一个函数 `IsFull()` 返回一个布尔值，表示一个名为 `w.client` 的 *PortalWorker 对象是否为完全关闭。具体来说，它通过调用 `w.client.IsFull()` 来获取 *PortalWorker 对象的当前状态，如果当前状态是完全关闭，则返回真，否则返回假。

第二个函数 `Closed()` 返回一个布尔值，表示一个名为 `w.client` 的 *PortalWorker 对象是否为关闭。具体来说，它通过调用 `w.client.Closed()` 来获取 *PortalWorker 对象的当前状态，如果当前状态是关闭，则返回真，否则返回假。


```go
func (w *PortalWorker) IsFull() bool {
	return w.client.IsFull()
}

func (w *PortalWorker) Closed() bool {
	return w.client.Closed()
}

```

# `app/reverse/portal_test.go`

这段代码的作用是测试一个名为"reverse.NewStaticMuxPicker"的函数，该函数在名为"reverse_test"的包中定义。

具体来说，这段代码会创建一个名为"picker"的静态MUX Picker，并测试它是否可以正常工作。测试代码首先通过调用"reverse.NewStaticMuxPicker"函数创建一个MUX Picker实例，然后使用该实例的"PickAvailable"方法尝试选择一个可用的Worker。如果选择Worker成功，代码会检查当前Worker是否为空，如果是，就测试选择Worker时是否抛出错误。如果选择Worker失败，代码会检查错误是否为空，如果不是，就会输出错误信息。


```go
package reverse_test

import (
	"testing"

	"v2ray.com/core/app/reverse"
	"v2ray.com/core/common"
)

func TestStaticPickerEmpty(t *testing.T) {
	picker, err := reverse.NewStaticMuxPicker()
	common.Must(err)
	worker, err := picker.PickAvailable()
	if err == nil {
		t.Error("expected error, but nil")
	}
	if worker != nil {
		t.Error("expected nil worker, but not nil")
	}
}

```

# `app/reverse/reverse.go`

这段代码是一个 Go 语言编写的 JavaScript 工具，用于将 Go 语言代码打包成辣鸡（R）运行时。它通过执行以下命令来生成一个名为 `v2ray.com/core/common/errors/errorgen` 的文件：

go build

这将会在当前目录下生成一个名为 `errorgen.go` 的文件。

此外，

//go:generate go run v2ray.com/core/common/errors/errorgen

这条注释还会生成一个名为 `errorgen_example.go` 的文件，这个文件会被视为一个独立的不需要编译的 Go 语言项目。


```go
// +build !confonly

package reverse

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/net"
	"v2ray.com/core/features/outbound"
	"v2ray.com/core/features/routing"
)

```

该代码定义了两个函数：isDomain 和 isInternalDomain，以及一个名为 init 的全局函数。它们的作用如下：

1. isDomain 函数接收一个网络目标（dest）和一个域名（domain），判断该目标是否属于内部域名 "reverse.internal.v2ray.com"。函数首先检查目标地址是否属于网络目标所在的域名，然后判断该域名是否与目标地址的域名相同。如果函数返回 true，说明该目标地址属于内部域名，否则不属于内部域名。

2. isInternalDomain 函数与 isDomain 函数类似，但返回值类型为 bool，而不是 true 和 false。

3. init 函数用于初始化 Reverse 类型的变量 r，并注册相关功能。函数首先检查配置参数是否为空，然后创建一个 Reverse 类型实例，并调用其初始化函数进行初始化。如果初始化函数执行成功，说明函数创建的 Reverse 实例已经初始化完成，可以创建 Reverse 类型的实例并注册使用了。


```go
const (
	internalDomain = "reverse.internal.v2ray.com"
)

func isDomain(dest net.Destination, domain string) bool {
	return dest.Address.Family().IsDomain() && dest.Address.Domain() == domain
}

func isInternalDomain(dest net.Destination) bool {
	return isDomain(dest, internalDomain)
}

func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		r := new(Reverse)
		if err := core.RequireFeatures(ctx, func(d routing.Dispatcher, om outbound.Manager) error {
			return r.Init(config.(*Config), d, om)
		}); err != nil {
			return nil, err
		}
		return r, nil
	}))
}

```

该代码定义了一个名为 `Reverse` 的结构体，它有两个字段 `bridges` 和 `portals`，分别表示桥和门的历史列表。

该结构体定义了一个名为 `Init` 的方法，该方法接收一个 `Config` 类型、一个 `Dispatcher` 类型和一个 `OHM` 类型的参数。该方法的作用于将 config 中的桥和 portal 初始化到历史列表中，并在初始化完成后返回 void。

对于每个传入的配置桥或 portal，该方法首先调用一个名为 `NewBridge` 的函数，该函数接收一个名为 `bConfig` 的桥配置和一个 `Dispatcher` 类型的参数，并返回一个桥对象。如果初始化失败，该函数返回一个错误。

接下来，该方法使用相同的逻辑重复调用 `NewPortal` 函数，该函数接收一个名为 `pConfig` 的门户配置和一个 `Dispatcher` 类型的参数，并返回一个门户对象。如果初始化失败，该函数返回一个错误。

最后，该方法返回一个 nil 值，表示初始化成功。


```go
type Reverse struct {
	bridges []*Bridge
	portals []*Portal
}

func (r *Reverse) Init(config *Config, d routing.Dispatcher, ohm outbound.Manager) error {
	for _, bConfig := range config.BridgeConfig {
		b, err := NewBridge(bConfig, d)
		if err != nil {
			return err
		}
		r.bridges = append(r.bridges, b)
	}

	for _, pConfig := range config.PortalConfig {
		p, err := NewPortal(pConfig, ohm)
		if err != nil {
			return err
		}
		r.portals = append(r.portals, p)
	}

	return nil
}

```

该代码定义了一个名为"func"的函数，接收一个名为"Reverse"的类型为"*Reverse"的参数。

函数有两个实现，第一个实现返回值为"*Reverse"类型，第二个实现返回值为"error"。

函数的第一个实现"func (r *Reverse) Type() interface{} {" 通过将参数传递给"interface{}"，返回其类型的包装的引用。在这种情况下，由于"Reverse"是一个匿名类型，其类型为"Reverse"，因此通过引用来获取其类型并将其存储在"r"变量中，以便在函数内部使用。

函数的第二个实现"func (r *Reverse) Start() error "，该函数使用一个for循环来遍历所有连接到端口的"Reverse"实例，并使用"Start"方法来启动连接到端口的操作。如果该尝试发生错误，函数将返回该错误。

函数的第二个实现中，对于每个连接到端口的"Reverse"实例，它调用"Start"方法并检查是否发生了错误。如果错误发生，函数将返回该错误。如果错误得到解决，函数将返回一个 nil 值，表示操作成功完成。


```go
func (r *Reverse) Type() interface{} {
	return (*Reverse)(nil)
}

func (r *Reverse) Start() error {
	for _, b := range r.bridges {
		if err := b.Start(); err != nil {
			return err
		}
	}

	for _, p := range r.portals {
		if err := p.Start(); err != nil {
			return err
		}
	}

	return nil
}

```

此代码定义了一个名为`func`的函数，接收一个名为`Reverse`的整数类型的指针变量`r`作为参数。函数的作用是关闭给定的后端服务器(可能是一个网络服务器或一个消息代理)，并返回一个包含所有错误的一组错误。

函数内部，首先遍历给定的后端服务器上的所有端口(可能是客户端)，然后遍历每个端口上的所有后端服务器。对于每个后端服务器，函数调用一个名为`Close`的函数并将其传递给一个代表该端口的变量。

然后，函数继续遍历给定的后端服务器上的所有端口，并重复调用`Close`函数。尽管如此，由于在`for`循环中已经包含了所有端口，因此函数将不会遍历到它们中的任何一个端口。

最后，函数返回一个包含所有错误的一组错误。函数将错误合并为一个错误对象，该对象使用`errors.Combine`函数将多个错误合并为一个新的错误。最后，函数返回该错误对象。


```go
func (r *Reverse) Close() error {
	var errs []error
	for _, b := range r.bridges {
		errs = append(errs, b.Close())
	}

	for _, p := range r.portals {
		errs = append(errs, p.Close())
	}

	return errors.Combine(errs...)
}

```

# `app/router/balancing.go`

这段代码定义了一个名为 "router" 的包，其中包含了一个名为 "BalancingStrategy" 的接口和一个名为 "RandomStrategy" 的 struct。

具体来说，这段代码使用了一个名为 "+build !confonly" 的构建模式，其中 "+build" 表示在生产环境构建该包，而 "!confonly" 表示仅在生产环境输出该包的内容，避免在开发模式下误构建。

接下来，该包定义了一个名为 "BalancingStrategy" 的接口，该接口包含一个名为 "PickOutbound" 的函数，用于选择出站方向。

接着，该包定义了一个名为 "RandomStrategy" 的 struct，该 struct包含一个 "策略" 字段，该字段为实现 "BalancingStrategy" 接口的 "PickOutbound" 函数所需的参数类型。

最后，该包的其他部分可能还包括其他依赖库的导入、函数或常量的定义，但以上内容是该包的核心部分。


```go
// +build !confonly

package router

import (
	"v2ray.com/core/common/dice"
	"v2ray.com/core/features/outbound"
)

type BalancingStrategy interface {
	PickOutbound([]string) string
}

type RandomStrategy struct {
}

```

该代码定义了一个名为`func`的函数，接收一个名为`s`的指针变量和一个字符串类型的数组`tags`作为参数。函数返回一个字符串类型的值。

函数的作用是使用`dice.Roll`函数，接收一个大小为`n`的数组`tags`，输出其中的一个元素并返回。`dice.Roll`函数的作用是 Roll 函数，它接收一个大小为`n`的随机整数，并返回一个新的数组，其中包含`n-1`个随机整数和一个指定的元素。

该函数定义了一个名为`Balancer`的类型，该类型包含一个字符串类型的字段`selectors`，一个`BalancingStrategy`类型的字段`strategy`和一个`outbound.Manager`类型的字段`ohm`。`Balancer`类型代表了一个负载均衡器，它可以管理多个后端服务器。

该函数没有输出，也没有进行任何计算。


```go
func (s *RandomStrategy) PickOutbound(tags []string) string {
	n := len(tags)
	if n == 0 {
		panic("0 tags")
	}

	return tags[dice.Roll(n)]
}

type Balancer struct {
	selectors []string
	strategy  BalancingStrategy
	ohm       outbound.Manager
}

```

此函数的作用是获取平衡策略选择的外部链接，并返回该链接和任何错误。

函数接收一个名为“b”的整数类型的指针变量和一个名为“Balancer”的接口类型的变量作为参数。函数内部首先检查“b”是否与“Balancer”兼容，如果不兼容，则返回错误并输出一条错误消息。如果兼容，则继续执行以下步骤。

接下来，函数使用“b.ohm”接口类型的变量所引用的“outbound.HandlerSelector”类型，选择所有与“b.selectors”相同的回溯操作器的标签。如果“b.selectors”为空，则函数将返回一个空字符串并输出一条错误消息。

接下来，函数使用“b.strategy.PickOutbound”函数，将“tags”数组中的所有标签作为输入，选择一个作为输出。如果选择任何标签都返回一个空字符串。

最后，函数返回选择的外部链接和任何错误。


```go
func (b *Balancer) PickOutbound() (string, error) {
	hs, ok := b.ohm.(outbound.HandlerSelector)
	if !ok {
		return "", newError("outbound.Manager is not a HandlerSelector")
	}
	tags := hs.Select(b.selectors)
	if len(tags) == 0 {
		return "", newError("no available outbounds selected")
	}
	tag := b.strategy.PickOutbound(tags)
	if tag == "" {
		return "", newError("balancing strategy returns empty tag")
	}
	return tag, nil
}

```

# `app/router/condition.go`

这段代码是一个 Go 语言编写的package router包的构建命令。具体来说，这段代码的作用是构建一个名为 "router" 的 package，同时禁止在构建时编译数千行字符串（即避免在构建过程中输出长字符串）。

首先，它使用 `build` 命令构建这个包。然后，它使用 `!confonly` 标志告诉 Go 编译器在构建时不要输出任何内容，因为这样会产生冗长的错误信息。

在构建成功后，可能会输出一些日志信息，告诉您已构建的文本文档或打包的资产数量。


```go
// +build !confonly

package router

import (
	"strings"

	"go.starlark.net/starlark"
	"go.starlark.net/syntax"

	"v2ray.com/core/common/net"
	"v2ray.com/core/common/strmatcher"
	"v2ray.com/core/features/routing"
)

```

这段代码定义了一个名为`Condition`的接口类型和`ConditionChan`的实现类型。`Condition`接口类型有一个名为`Apply`的方法，该方法接受一个`routing.Context`，并返回一个`bool`类型的值。`ConditionChan`实现了一个`ConditionChan`类型，该类型的值是一个`Condition`类型的数组，该数组的容量为`8`。

`NewConditionChan`函数创建了一个空的`ConditionChan`类型的实例，返回该实例的引用。

`func (v *ConditionChan) Add(cond condition) *ConditionChan`函数接收一个`Condition`类型的参数`cond`并将其添加到`ConditionChan`类型的实例中。为了满足`ConditionChan`需要一个`Condition`类型的参数，`v`参数被复制了一份，并将其添加到`ConditionChan`类型的实例中。

这两个函数一起工作，使得`ConditionChan`可以像一个条件通道一样工作，接受一个`Condition`类型的参数并返回一个符合条件的`Condition`类型的数组。


```go
type Condition interface {
	Apply(ctx routing.Context) bool
}

type ConditionChan []Condition

func NewConditionChan() *ConditionChan {
	var condChan ConditionChan = make([]Condition, 0, 8)
	return &condChan
}

func (v *ConditionChan) Add(cond Condition) *ConditionChan {
	*v = append(*v, cond)
	return v
}

```

这段代码定义了一个名为`ConditionChan`的结构体，它用来管理一组`Condition`类型的变量。这个结构体包含一个方法`Apply`和一个方法`Len`。以下是这两部分代码的详细解释：

go
// Apply applies all conditions registered in this chan.
func (v *ConditionChan) Apply(ctx routing.Context) bool {
	for _, cond := range *v {
		if !cond.Apply(ctx) {
			return false
		}
	}
	return true
}


这段代码的作用是让`ConditionChan`中的所有`Condition`类型的变量都具有行为。如果任何条件的应用失败，则返回`false`。然后，让应用程序逐个应用所有条件，直到所有条件都成功应用或所有条件都失败。最后，返回经过应用程序的条件有效的结果。

go
func (v *ConditionChan) Len() int {
	return len(*v)
}


这段代码返回一个名为`ConditionChan`的结构体中`Condition`类型的变量数量。

go
var matcherTypeMap = map[Domain_Type]strmatcher.Type{
	Domain_Plain:  strmatcher.Substr,
	Domain_Regex:  strmatcher.Regex,
	Domain_Domain: strmatcher.Domain,
	Domain_Full:   strmatcher.Full,
}


这段代码定义了一个名为`matcherTypeMap`的 map 类型，它包含一个名为`Domain_Plain`的键值对，它对应一个名为`strmatcher.Substr`的类型，表示匹配字符串中的一部分。它还包含一个名为`Domain_Regex`的键值对，它对应一个名为`strmatcher.Regex`的类型，表示匹配字符串的模式。


```go
// Apply applies all conditions registered in this chan.
func (v *ConditionChan) Apply(ctx routing.Context) bool {
	for _, cond := range *v {
		if !cond.Apply(ctx) {
			return false
		}
	}
	return true
}

func (v *ConditionChan) Len() int {
	return len(*v)
}

var matcherTypeMap = map[Domain_Type]strmatcher.Type{
	Domain_Plain:  strmatcher.Substr,
	Domain_Regex:  strmatcher.Regex,
	Domain_Domain: strmatcher.Domain,
	Domain_Full:   strmatcher.Full,
}

```

该代码定义了一个名为"domainMatcher"的结构体"DomainMatcher"，用于将给定的域名(通过指针域)映射到相应的 matcher(通过引用来访问域中的值)。

具体来说，代码首先定义了一个名为"func domainToMatcher(domain *Domain) (strmatcher.Matcher, error) "的函数，它接收一个域名指针(*Domain)，返回一个 matcher 类型和一个表示错误的错误对象。函数的核心处理步骤如下：

1. 从 "matcherTypeMap" 数组中查找给定的域名类型，因为该类型不在数组中，则返回一个错误对象。
2. 如果找到了给定的域名类型，尝试使用 "matcherType.New" 函数创建一个 matcher 实例。
3. 如果创建 matcher 失败，则返回一个错误对象。
4. 函数返回 matcher 实例和 nil，表示已经成功将域名映射到 matcher。

该函数定义了一个 "DomainMatcher" 类型，该类型包含一个 "matchers" 字段，该字段是一个整数类型的指针，指向一个名为 "IndexMatcher" 的 matcher 类型。通过将域名和 matcher 类型之间的映射，可以轻松地将域名中的值映射到相应的 matcher 中。


```go
func domainToMatcher(domain *Domain) (strmatcher.Matcher, error) {
	matcherType, f := matcherTypeMap[domain.Type]
	if !f {
		return nil, newError("unsupported domain type", domain.Type)
	}

	matcher, err := matcherType.New(domain.Value)
	if err != nil {
		return nil, newError("failed to create domain matcher").Base(err)
	}

	return matcher, nil
}

type DomainMatcher struct {
	matchers strmatcher.IndexMatcher
}

```

此函数的作用是创建一个名为 `NewDomainMatcher` 的函数类型，该函数接收一个字符串数组 `domains`，并返回一个指向 `DomainMatcher` 类型的指针以及一个 `error` 类型的变量。

函数的实现过程如下：

1. 创建一个字符串匹配器 `g`。
2. 遍历 `domains` 数组中的每个元素。
3. 对于每个元素 `d`，调用 `domainToMatcher` 函数，将 `d` 转换为相应的匹配器。
4. 将得到的匹配器添加到 `g` 中。
5. 返回一个指向 `DomainMatcher` 类型的指针以及一个 `error` 类型的变量。

该函数允许您创建一个匹配所有提供给 `domainToMatcher` 的字符串的匹配器。通过循环遍历 `domains` 数组，您可以为每个元素创建一个独立的匹配器，这对于某些特定的用例可能很有用。


```go
func NewDomainMatcher(domains []*Domain) (*DomainMatcher, error) {
	g := new(strmatcher.MatcherGroup)
	for _, d := range domains {
		m, err := domainToMatcher(d)
		if err != nil {
			return nil, err
		}
		g.Add(m)
	}

	return &DomainMatcher{
		matchers: g,
	}, nil
}

```

此代码定义了一个名为MultiGeoIPMatcher的类，该类实现了一个名为Condition的 Condition 接口，并在其中使用了两个函数：ApplyDomain 和 Apply。

MultiGeoIPMatcher中有一个名为matchers 的成员变量，它是一个切片（slice），其中包含一个或多个 GeoIPMatcher 类型的对象。这个类有一个名为onSource 的成员变量，它是一个布尔值，表示在应用条件时是否仅从源站中获取匹配结果。

ApplyDomain函数接收一个字符串参数域名，并返回一个布尔值，表示应用该条件。该函数首先使用 m.matchers.Match(domain) 获取一个或多个与给定域名匹配的 GeoIPMatcher 对象，然后使用其中的一个或多个对象来匹配实际域名，并返回其应用结果。

Apply函数接收一个名为ctx 的 routing.Context 对象，该对象包含了获取目标域名的上下文信息。该函数首先使用 MultiGeoIPMatcher.ApplyDomain(ctx.GetTargetDomain()) 获取目标域名是否符合应用的条件，然后返回 true 或 false。其中，ctx.GetTargetDomain() 方法返回获取的目标域名，如果该域名包含空格或只包含顶级域名，则认为其不符合应用的条件，返回 false。


```go
func (m *DomainMatcher) ApplyDomain(domain string) bool {
	return len(m.matchers.Match(domain)) > 0
}

// Apply implements Condition.
func (m *DomainMatcher) Apply(ctx routing.Context) bool {
	domain := ctx.GetTargetDomain()
	if len(domain) == 0 {
		return false
	}
	return m.ApplyDomain(domain)
}

type MultiGeoIPMatcher struct {
	matchers []*GeoIPMatcher
	onSource bool
}

```

此代码定义了一个名为`NewMultiGeoIPMatcher`的函数，它接收一个`geoips`切片和一个`onSource`布尔参数。函数返回一个`MultiGeoIPMatcher`类型的指针，如果没有错误，函数将返回该`MultiGeoIPMatcher`的引用，否则返回一个错误。

函数的具体实现包括以下几步：

1. 定义一个`matchers`切片，用于存储所有传入的`GeoIPMatcher`对象。
2. 遍历`geoips`切片，为每个`GeoIPMatcher`对象执行以下操作：
	1. 使用`globalGeoIPContainer.Add`函数将GeoIP编号存储在`geoips`中的`GeoIPMatcher`实例中。
	2. 如果执行上述操作时出现错误，返回错误并停止遍历。
	3. 将处理过的`GeoIPMatcher`对象存储在`matchers`切片中的对应位置。
3. 创建一个`MultiGeoIPMatcher`实例，将`matchers`切片和`onSource`参数作为构造函数的参数传递给`MultiGeoIPMatcher`构造函数。
4. 返回`MultiGeoIPMatcher`实例，如果没有错误。


```go
func NewMultiGeoIPMatcher(geoips []*GeoIP, onSource bool) (*MultiGeoIPMatcher, error) {
	var matchers []*GeoIPMatcher
	for _, geoip := range geoips {
		matcher, err := globalGeoIPContainer.Add(geoip)
		if err != nil {
			return nil, err
		}
		matchers = append(matchers, matcher)
	}

	matcher := &MultiGeoIPMatcher{
		matchers: matchers,
		onSource: onSource,
	}

	return matcher, nil
}

```

这段代码定义了一个名为`MultiGeoIPMatcher`的`Condition`类型，用于在路由器上下文中检查是否匹配IP地址。

该函数`Apply`接收一个`routing.Context`作为参数，并在上下文中获取源IP和目标IP。如果`m.onSource`为`true`，则从源IP中获取ips，否则从目标IP中获取ips。

接下来，函数遍历获得的ips，并将每个ips传递给一个名为`matchers`的切片，每个`matchers`都是一个`GeoIPMatcher`类型的参数。

在循环中，函数使用`matchers`中的`Match`函数来检查每个获得的ip是否与`matchers`中的任何`Match`函数匹配。如果匹配成功，则返回`true`，否则继续循环。

最后，函数返回`false`作为结果，表示没有找到匹配的ip地址。


```go
// Apply implements Condition.
func (m *MultiGeoIPMatcher) Apply(ctx routing.Context) bool {
	var ips []net.IP
	if m.onSource {
		ips = ctx.GetSourceIPs()
	} else {
		ips = ctx.GetTargetIPs()
	}
	for _, ip := range ips {
		for _, matcher := range m.matchers {
			if matcher.Match(ip) {
				return true
			}
		}
	}
	return false
}

```

该代码定义了一个名为 "PortMatcher" 的结构体，它表示一个端口匹配器。该结构体有两个成员变量，一个名为 "port" 的整型变量和一个名为 "onSource" 的布尔变量。

此外，该结构体还包含一个名为 "NewPortMatcher" 的函数和一个名为 "Apply" 的函数。

"NewPortMatcher" 函数的实现比较简单：它接收一个 "list" 参数和一个 "onSource" 参数，然后使用 "net.PortListFromProto" 函数将 "list" 参数转换为一个网络端口列表，并将 "onSource" 参数设置为 "true" 或 "false"。

"Apply" 函数的实现比较复杂：它接收一个 "ctx" 参数，然后使用 "port.Contains" 方法判断源或目标端口是否在 "port" 变量所包含的端口中。如果 "onSource" 参数为 "true"，则只检查源端口；如果 "onSource" 参数为 "false"，则检查目标端口。


```go
type PortMatcher struct {
	port     net.MemoryPortList
	onSource bool
}

// NewPortMatcher create a new port matcher that can match source or destination port
func NewPortMatcher(list *net.PortList, onSource bool) *PortMatcher {
	return &PortMatcher{
		port:     net.PortListFromProto(list),
		onSource: onSource,
	}
}

// Apply implements Condition.
func (v *PortMatcher) Apply(ctx routing.Context) bool {
	if v.onSource {
		return v.port.Contains(ctx.GetSourcePort())
	} else {
		return v.port.Contains(ctx.GetTargetPort())
	}
}

```

此代码定义了一个名为NetworkMatcher的结构体，它包含一个长度为8的布尔数组。该结构体实现了Condition接口，其中包括一个名为Apply的Method。

具体来说，该代码的作用是创建一个NetworkMatcher实例，该实例由一个长度为8的布尔数组和一个名为Apply的Method组成。在创建该实例时，使用了网络中所有网络的名称作为参数传递给网络Matcher的构造函数。然后，使用一个循环来将每个网络的名称转换为对应的布尔值，并将这些布尔值存储在NetworkMatcher的布尔数组中。最后，返回该NetworkMatcher实例，以便在Apply方法中进行更广泛的检查。

Apply方法的实现如下：首先，使用获取ctx的IntValue函数获取获取的Network对象的网络名称。然后，使用网络对象的名称在NetworkMatcher的布尔数组中查找对应的布尔值。如果找到了该网络名称，则返回相应的布尔值，否则返回false。


```go
type NetworkMatcher struct {
	list [8]bool
}

func NewNetworkMatcher(network []net.Network) NetworkMatcher {
	var matcher NetworkMatcher
	for _, n := range network {
		matcher.list[int(n)] = true
	}
	return matcher
}

// Apply implements Condition.
func (v NetworkMatcher) Apply(ctx routing.Context) bool {
	return v.list[int(ctx.GetNetwork())]
}

```

这段代码定义了一个名为UserMatcher的结构体，该结构体有一个名为user的 slice(切片)和一个名为NewUserMatcher的函数。函数接收一个包含多个字符串的 slice(切片)，将其中的所有元素复制到一个新 slice 中，最后返回一个新的 UserMatcher 结构体，该结构体包含一个包含新 slice 中元素的 field。

更具体地说，这段代码创建了一个名为UserMatcher的结构体，该结构体有一个名为user的 slice(切片)。函数NewUserMatcher接收一个包含多个字符串的 slice(切片)，将其中的所有元素复制到一个新 slice 中，然后将新 slice 赋值给UserMatcher的user字段。这样，当函数调用者调用NewUserMatcher时，可以得到一个UserMatcher结构体，其中包含一个包含所有传递给函数的用户的字符串的 slice。


```go
type UserMatcher struct {
	user []string
}

func NewUserMatcher(users []string) *UserMatcher {
	usersCopy := make([]string, 0, len(users))
	for _, user := range users {
		if len(user) > 0 {
			usersCopy = append(usersCopy, user)
		}
	}
	return &UserMatcher{
		user: usersCopy,
	}
}

```

该代码定义了一个名为InboundTagMatcher的结构体，表示从一个请求中获取标记的标签。

接着定义了一个名为Apply的函数，该函数接收一个上下文（routing.Context）和一个标记器（Condition）类型的参数，标记器在函数中应用筛选条件。函数实现为返回一个布尔值，表示是否匹配上来的用户。

函数中首先获取一个标记器实例，然后遍历标记器中的所有标记（由user变量获取）。接着，遍历用户标记器中的所有标记，如果当前标记与用户标记器中的标记相同，则返回true，否则返回false。

最后，定义了一个名为InboundTagMatcher的结构体，该结构体包含一个包含所有标记的切片（切片类型）。


```go
// Apply implements Condition.
func (v *UserMatcher) Apply(ctx routing.Context) bool {
	user := ctx.GetUser()
	if len(user) == 0 {
		return false
	}
	for _, u := range v.user {
		if u == user {
			return true
		}
	}
	return false
}

type InboundTagMatcher struct {
	tags []string
}

```

该函数的作用是创建一个名为 `NewInboundTagMatcher` 的函数，接受一个字符串数组 `tags`，并返回一个指向 `InboundTagMatcher` 类型的指针。该函数会将传入的 `tags` 数组进行遍历，对于每个元素，如果它的长度大于 0，则将其添加到 `tagsCopy` 数组中。最后，函数返回一个指向包含 `tagsCopy` 数组和空字符串的 `InboundTagMatcher` 类型的指针。

函数的实现使用了以下步骤：

1. 创建一个名为 `tagsCopy` 的新数组，其大小与传入的 `tags` 数组相同，但元素类型为字符串。
2. 遍历 `tags` 数组中的每个元素，如果该元素的长度大于 0，则将其添加到 `tagsCopy` 数组中。
3. 返回一个指向包含 `tagsCopy` 数组和空字符串的 `InboundTagMatcher` 类型的指针。

函数的 `Apply` 函数的实现如下：

1. 获取传入的 `tag` 字段。
2. 遍历 `tags` 数组中的每个元素，如果当前元素与 `tag` 字相同，则返回 `true`，否则继续遍历。
3. 返回 `false`，因为 `tag` 的长度为 0 时，函数返回 `false`。


```go
func NewInboundTagMatcher(tags []string) *InboundTagMatcher {
	tagsCopy := make([]string, 0, len(tags))
	for _, tag := range tags {
		if len(tag) > 0 {
			tagsCopy = append(tagsCopy, tag)
		}
	}
	return &InboundTagMatcher{
		tags: tagsCopy,
	}
}

// Apply implements Condition.
func (v *InboundTagMatcher) Apply(ctx routing.Context) bool {
	tag := ctx.GetInboundTag()
	if len(tag) == 0 {
		return false
	}
	for _, t := range v.tags {
		if t == tag {
			return true
		}
	}
	return false
}

```

该代码定义了一个名为 "ProtocolMatcher" 的结构体，它包含一个名为 "protocols" 的类型为 "[]string" 的字段。

然后，该代码实现了一个名为 "NewProtocolMatcher" 的函数，该函数接收一个名为 "protocols" 的类型为 "[]string" 的参数，并返回一个名为 "ProtocolMatcher" 的结构体类型的变量。

函数实现了一个简单的过程：遍历传入的 "protocols" 列表中的每个字符串，将其长度大于零的字符串复制到一个名为 "pCopy" 的新列表中。这样，当遍历完所有字符串并创建 "ProtocolMatcher" 结构体时，"pCopy" 中就包含了一个以 "protocols" 中的第一个字符串为开头的字符串，从而可以创建一个 "ProtocolMatcher" 实例。


```go
type ProtocolMatcher struct {
	protocols []string
}

func NewProtocolMatcher(protocols []string) *ProtocolMatcher {
	pCopy := make([]string, 0, len(protocols))

	for _, p := range protocols {
		if len(p) > 0 {
			pCopy = append(pCopy, p)
		}
	}

	return &ProtocolMatcher{
		protocols: pCopy,
	}
}

```

该代码定义了一个名为 AttributeMatcher 的结构体，表示一个可以应用于输入数据判断条件的协议 matcher。该结构体包含一个指向协议 matcher 的指针（在大多数情况下，是 *ProtocolMatcher 类型的指针），以及一个指向 AttributeMatcher 的指针（在大多数情况下，是 *AttributeMatcher 类型的指针）。

函数 Apply 接收一个名为 ctx 的路由上下文，并返回一个布尔值，表示条件是否成立。函数首先获取上下文中的协议，如果协议长度为 0，则返回 false。接下来，函数遍历协议数组 m.protocols，检查输入的协议是否与 m.protocols 中的任何协议字符串以某种方式开头。如果是，则返回 true；如果不是，则返回 false。最后，函数返回前一个条件的返回值，即 false。

该函数的实现是通过对协议数组中的每个协议进行模糊匹配，实现的逻辑与实现方式与题目的描述相符。


```go
// Apply implements Condition.
func (m *ProtocolMatcher) Apply(ctx routing.Context) bool {
	protocol := ctx.GetProtocol()
	if len(protocol) == 0 {
		return false
	}
	for _, p := range m.protocols {
		if strings.HasPrefix(protocol, p) {
			return true
		}
	}
	return false
}

type AttributeMatcher struct {
	program *starlark.Program
}

```

此代码定义了一个名为 `NewAttributeMatcher` 的函数，它接受一个 `code` 参数，该参数是一个字符串。函数返回一个指向 `AttributeMatcher` 类型的指针以及一个 `error` 类型的指针。

函数的作用是创建一个可以验证 StarLark 文件中的 `Attribute` 规则的函数。具体来说，它首先使用 `syntax.Parse` 函数将一个名为 `.star` 的文件中的 `satisfied` 属性规则字符串解析为 Lark 语法树。然后，它使用 `starlark.FileProgram` 函数将该语法树应用于名为 `attrs` 的文件中的函数，该函数返回一个真值，如果文件中的函数返回真。最后，函数创建一个 `AttributeMatcher` 类型的指针，其中 `program` 字段包含解析后的语法树，并将其返回。如果函数在创建或解析语法树时出现错误，它将返回一个指向错误对象的指针。


```go
func NewAttributeMatcher(code string) (*AttributeMatcher, error) {
	starFile, err := syntax.Parse("attr.star", "satisfied=("+code+")", 0)
	if err != nil {
		return nil, newError("attr rule").Base(err)
	}
	p, err := starlark.FileProgram(starFile, func(name string) bool {
		return name == "attrs"
	})
	if err != nil {
		return nil, err
	}
	return &AttributeMatcher{
		program: p,
	}, nil
}

```

这段代码定义了一个名为 `Match` 的函数，它接受一个名为 `attrs` 的 map 类型，代表输入的一个或多个属性的名称，并返回一个布尔值，表示是否匹配了所有的属性。

函数内部首先创建了一个名为 `attrsDict` 的 map 类型，然后遍历 `attrs` 的 key 和 value，将其分别映射为 `starlark.String` 类型并添加到 `attrsDict` 中。

接着，定义了一个名为 `predefined` 的 map 类型，其中包含一个名为 `attrs` 的键，其值为 `attrsDict`。

然后，创建了一个名为 `thread` 的 `starlark.Thread` 实例，并设置 `name` 为 `"matcher"`。接着调用 `m.program.Init` 函数，传递 `attrs` 和 `predefined`，并指定 `thread` 为 `thread`，`predefined` 为 `predefined`。

最后，获取 `m.program.Init` 函数的输出结果，即 `satisfied` 的值，并判断其是否为 `nil` 且返回 `true`，表示匹配成功。


```go
// Match implements attributes matching.
func (m *AttributeMatcher) Match(attrs map[string]string) bool {
	attrsDict := new(starlark.Dict)
	for key, value := range attrs {
		attrsDict.SetKey(starlark.String(key), starlark.String(value))
	}

	predefined := make(starlark.StringDict)
	predefined["attrs"] = attrsDict

	thread := &starlark.Thread{
		Name: "matcher",
	}
	results, err := m.program.Init(thread, predefined)
	if err != nil {
		newError("attr matcher").Base(err).WriteToLog()
	}
	satisfied := results["satisfied"]
	return satisfied != nil && bool(satisfied.Truth())
}

```

此代码定义了一个名为`AttributeMatcher`的`Condition`类型，该类型实现了`Condition`接口。

在`Apply`函数中，首先获取上下文上下文`ctx`中的`Attributes`字段，如果该字段为`nil`，则返回`false`。否则，代码会执行`m.Match`函数，该函数将匹配`ctx.Attributes`与条件`m`中的条件比较。

`Match`函数的行为是检查`ctx.Attributes`是否与`m.Condition`匹配。如果两个匹配，则返回`true`，否则返回`false`。

如果`Apply`函数返回`true`，则该`Condition`将被应用到路由的上下文中，即满足该条件的请求将继续在请求路径上流动。


```go
// Apply implements Condition.
func (m *AttributeMatcher) Apply(ctx routing.Context) bool {
	attributes := ctx.GetAttributes()
	if attributes == nil {
		return false
	}
	return m.Match(attributes)
}

```

# `app/router/condition_geoip.go`

这段代码是一个 Rust 代码片段，它定义了一个名为 "router" 的 package，其中包含了一个名为 "ipv6" 的结构体。

这里使用了几个特殊的含义：

* "+build" 表示这是一个构建函数，告诉编译器在编译之前对源代码进行一些处理，以提高编译效率。
* "!confonly" 表示这是一个只读的编译选项，告诉编译器不要修改这个源文件的内容，以免出错。

接下来它会定义一个名为 "ipv6" 的结构体，结构体有两个成员变量，分别是一个 "uint64" 类型的 "a" 和一个 "uint64" 类型的 "b"。

这里的 "ipv6" 结构体表示 IPv6 中的一个地址，它由两个 64 位的值组成，一个用于 IPv6 的前缀，一个用于 IPv6 的自治系统 ID。


```go
// +build !confonly

package router

import (
	"encoding/binary"
	"sort"

	"v2ray.com/core/common/net"
)

type ipv6 struct {
	a uint64
	b uint64
}

```

该代码定义了一个名为 GeoIPMatcher 的结构体，表示地理IP地址匹配器。该结构体包含以下字段：

* countryCode：国家代码，以 ISO 3166-1 标准表示。
* ip4：IPv4 地址，以 0x0001:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:0000:00


```go
type GeoIPMatcher struct {
	countryCode string
	ip4         []uint32
	prefix4     []uint8
	ip6         []ipv6
	prefix6     []uint8
}

func normalize4(ip uint32, prefix uint8) uint32 {
	return (ip >> (32 - prefix)) << (32 - prefix)
}

func normalize6(ip ipv6, prefix uint8) ipv6 {
	if prefix <= 64 {
		ip.a = (ip.a >> (64 - prefix)) << (64 - prefix)
		ip.b = 0
	} else {
		ip.b = (ip.b >> (128 - prefix)) << (128 - prefix)
	}
	return ip
}

```

This code defines an `Init` function for a `GeoIPMatcher` struct that initializes the IP4 and IP6 counts and stores the IP addresses, prefixes, and the corresponding CIDR blocks in the `ip4` and `ip6` arrays and the `prefix6` array, respectively.

The function takes a slice of CIDR blocks as input, sorts them by their CIDR blocks, and initializes the `ip4` and `ip6` counts to zero.

The function then loops through each CIDR block, normalizes the IP4 and IP6 addresses to the corresponding prefix, and appends the IP4 and IP6 addresses, prefixes, and CIDR blocks to the corresponding arrays.

The function returns an error if the input CIDR blocks are not valid.


```go
func (m *GeoIPMatcher) Init(cidrs []*CIDR) error {
	ip4Count := 0
	ip6Count := 0

	for _, cidr := range cidrs {
		ip := cidr.Ip
		switch len(ip) {
		case 4:
			ip4Count++
		case 16:
			ip6Count++
		default:
			return newError("unexpect ip length: ", len(ip))
		}
	}

	cidrList := CIDRList(cidrs)
	sort.Sort(&cidrList)

	m.ip4 = make([]uint32, 0, ip4Count)
	m.prefix4 = make([]uint8, 0, ip4Count)
	m.ip6 = make([]ipv6, 0, ip6Count)
	m.prefix6 = make([]uint8, 0, ip6Count)

	for _, cidr := range cidrs {
		ip := cidr.Ip
		prefix := uint8(cidr.Prefix)
		switch len(ip) {
		case 4:
			m.ip4 = append(m.ip4, normalize4(binary.BigEndian.Uint32(ip), prefix))
			m.prefix4 = append(m.prefix4, prefix)
		case 16:
			ip6 := ipv6{
				a: binary.BigEndian.Uint64(ip[0:8]),
				b: binary.BigEndian.Uint64(ip[8:16]),
			}
			ip6 = normalize6(ip6, prefix)

			m.ip6 = append(m.ip6, ip6)
			m.prefix6 = append(m.prefix6, prefix)
		}
	}

	return nil
}

```

该函数是一个名为"func"的函数，接受一个名为"m"的指针和一个名为"ip"的整数参数。函数返回一个名为"true"或"false"的布尔值。

函数的作用是判断给定的IP地址是否匹配给定的GeoIP匹配器。如果匹配器中没有设置任何IPv4地址，则返回 false。如果IP地址小于匹配器中的第一个IPv4地址，则返回 false。

函数的核心部分包括以下步骤：

1. 检查给定的IP地址是否为空字符串。如果是，返回 false。
2. 如果给定的IP地址长度小于匹配器中的IPv4地址长度，则返回 false。
3. 定义变量l、r、x，分别代表last、right、expanded IPv4 address。
4. 循环从expanded IPv4 address 到size-1(size-1 是指匹配器中IPv4地址的长度)，检查给定的IP地址是否小于等于匹配器中的IPv4地址。如果是，将r设为当前expanded IPv4 address，继续循环。
5. 如果当前IP地址等于匹配器中的IPv4地址，则返回 true。
6. 如果当前IP地址不等于匹配器中的IPv4地址，则将l的值递增，继续循环，直到l大于等于size-1。
7. 如果所有的循环都已完成，仍然没有找到匹配的IPv4 address，则返回 false。

函数中还包含一个辅助函数normalize4(ip, prefix4)，该函数的作用是将IP地址和预缀转化为4位整数，使得函数内部使用相同的预缀。


```go
func (m *GeoIPMatcher) match4(ip uint32) bool {
	if len(m.ip4) == 0 {
		return false
	}

	if ip < m.ip4[0] {
		return false
	}

	size := uint32(len(m.ip4))
	l := uint32(0)
	r := size
	for l < r {
		x := ((l + r) >> 1)
		if ip < m.ip4[x] {
			r = x
			continue
		}

		nip := normalize4(ip, m.prefix4[x])
		if nip == m.ip4[x] {
			return true
		}

		l = x + 1
	}

	return l > 0 && normalize4(ip, m.prefix4[l-1]) == m.ip4[l-1]
}

```

这两段代码定义了两个函数，分别是 `less6` 和 `match6`。这两个函数都使用 IPv6 地址作为输入参数，并返回一个布尔值。

`less6` 函数的实现比较简单，它比较两个 IPv6 地址是否符合某些条件。具体来说，它首先检查两个输入地址是否相等，如果是，再检查第二个地址的某个字段是否小于第一个地址的相应字段，如果是，那么这两个地址就符合条件，函数返回 `true`。否则，如果有一个输入地址符合条件，那么这两个地址就不符合条件，函数返回 `false`。

`match6` 函数的实现稍微复杂一些，它通过 IPv6 地址来匹配一个预定义的 IPv6 前缀。具体来说，它首先检查传递给它的 IPv6 地址是否为空，如果是，那么函数返回 `false`。然后，它遍历传递给它的 IPv6 地址和预定义前缀的长度，比较这两个地址是否匹配。如果两个地址完全匹配，那么函数返回 `true`；否则，如果有一个地址的前缀和另一个地址的地址匹配，那么函数继续比较前缀是否和另一个地址匹配，直到匹配成功或者前缀长度为 0，函数返回 `true`。


```go
func less6(a ipv6, b ipv6) bool {
	return a.a < b.a || (a.a == b.a && a.b < b.b)
}

func (m *GeoIPMatcher) match6(ip ipv6) bool {
	if len(m.ip6) == 0 {
		return false
	}

	if less6(ip, m.ip6[0]) {
		return false
	}

	size := uint32(len(m.ip6))
	l := uint32(0)
	r := size
	for l < r {
		x := (l + r) / 2
		if less6(ip, m.ip6[x]) {
			r = x
			continue
		}

		if normalize6(ip, m.prefix6[x]) == m.ip6[x] {
			return true
		}

		l = x + 1
	}

	return l > 0 && normalize6(ip, m.prefix6[l-1]) == m.ip6[l-1]
}

```

该代码是一个名为GeoIPMatcher的函数，它接受一个IP地址参数。函数返回IP地址是否被GeoIP包含返回true，否则返回false。

函数内部使用了一个switch语句，根据IP地址长度来执行不同的判断。如果IP地址长度为4，则函数会执行m.match4函数，该函数会使用一个字节切片来调用IPv4类型的m变量，并传递IPv4类型的参数。如果IP地址长度为16，则函数会执行m.match6函数，该函数会使用一个IPv6类型的参数来调用m变量，并传递IPv6类型的IPv6地址。

如果IP地址长度不匹配，函数会返回false。


```go
// Match returns true if the given ip is included by the GeoIP.
func (m *GeoIPMatcher) Match(ip net.IP) bool {
	switch len(ip) {
	case 4:
		return m.match4(binary.BigEndian.Uint32(ip))
	case 16:
		return m.match6(ipv6{
			a: binary.BigEndian.Uint64(ip[0:8]),
			b: binary.BigEndian.Uint64(ip[8:16]),
		})
	default:
		return false
	}
}

```

这段代码定义了一个名为`GeoIPMatcherContainer`的结构体，该结构体用于保存 GeoIPMatcher 的副本，每个副本通过国家码进行唯一标识。

该容器`c`包含一个名为`matchers`的数组，该数组用于存储所有的 GeoIPMatcher。

在`Add`方法中，如果给定的`geoip`对象包含一个非空的国家码，则尝试在`c.matchers`数组中查找是否存在该国家码的`GeoIPMatcher`。如果找到了，则返回该`GeoIPMatcher`对象的引用，否则返回一个`nil`错误。

如果给定的`geoip`对象的国家码为空，则创建一个新的`GeoIPMatcher`对象，并将其添加到`c.matchers`数组中。

请注意，该代码在测试时需要定义一个测试用例，以确保其正常运行。


```go
// GeoIPMatcherContainer is a container for GeoIPMatchers. It keeps unique copies of GeoIPMatcher by country code.
type GeoIPMatcherContainer struct {
	matchers []*GeoIPMatcher
}

// Add adds a new GeoIP set into the container.
// If the country code of GeoIP is not empty, GeoIPMatcherContainer will try to find an existing one, instead of adding a new one.
func (c *GeoIPMatcherContainer) Add(geoip *GeoIP) (*GeoIPMatcher, error) {
	if len(geoip.CountryCode) > 0 {
		for _, m := range c.matchers {
			if m.countryCode == geoip.CountryCode {
				return m, nil
			}
		}
	}

	m := &GeoIPMatcher{
		countryCode: geoip.CountryCode,
	}
	if err := m.Init(geoip.Cidr); err != nil {
		return nil, err
	}
	if len(geoip.CountryCode) > 0 {
		c.matchers = append(c.matchers, m)
	}
	return m, nil
}

```

这段代码定义了一个名为 `var` 的变量，其类型为 `GeoIPMatcherContainer`。

这里 `GeoIPMatcherContainer` 是一个实现了 `GeoIPMatcher` 接口的类，它用于将输入的地理ID（geolocation ID）与指定的匹配模式匹配，返回匹配到的第一个地理ID。

因此，这段代码的作用是定义了一个 `GeoIPMatcherContainer` 类型的变量，用于匹配输入的地理ID。


```go
var (
	globalGeoIPContainer GeoIPMatcherContainer
)

```