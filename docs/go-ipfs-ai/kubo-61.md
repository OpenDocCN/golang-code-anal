# go-ipfs 源码解析 61

# `/opt/kubo/thirdparty/assert/assert.go`

这两段代码定义了两个函数：`Nil` 和 `True`。它们的作用是在 `testing` 包中的 `Nil` 和 `True` 测试函数中进行断言。

具体来说，这两段代码以下划线开头的 `package` 关键字表示这是一个自定义包（Assertion）。接着，这两段代码分别定义了 `Nil` 和 `True` 两个函数。它们都接收两个参数：一个 `err` 类型和一个 `testing.T` 类型的参数 `t`，以及一个或多个 `msgs` 参数。

这两段代码通过使用 `t.Fatal` 和 `t.Run` 函数来触发断言。如果 `Nil` 或 `True` 条件为 `false`，则函数将引发断言，并通过 `t.Fatal` 函数打印错误消息和 `msgs` 参数。如果 `Nil` 或 `True` 条件为 `true`，则函数不会引发断言，而是直接返回。


```
package assert

import "testing"

func Nil(err error, t *testing.T, msgs ...string) {
	if err != nil {
		t.Fatal(msgs, "error:", err)
	}
}

func True(v bool, t *testing.T, msgs ...string) {
	if !v {
		t.Fatal(msgs)
	}
}

```

这两函数分别是 Python中的一个测试框架 `testing` 中的 `func` 函数体。

它们的作用如下：

1. `func False(v bool, t *testing.T, msgs ...string)` 是一个测试函数，它会测试一个名为 `v` 的布尔变量，以及一个名为 `msgs` 的字符串数组。它的实现是在函数体中判断 `v` 是否为 `False`，如果是，函数会输出一个错误消息并返回 `True`，否则返回 `False`。然后，它会让 `t` 测试套中的断言函数执行，如果 `v` 是 `False`，则输出一条错误消息，否则不输出任何消息。

2. `func Err(err error, t *testing.T, msgs ...string)` 同样是测试函数，它会测试一个名为 `err` 的错误变量，以及一个名为 `msgs` 的字符串数组。它的实现是首先检查 `err` 是否为 ` nil`，如果是，函数会输出一条错误消息并返回 `True`，否则会输出 `err` 错误并返回 `False`。然后，它会让 `t` 测试套中的断言函数执行，如果 `err` 是 ` nil`，则输出一条错误消息，否则不输出任何消息。

这两个函数的作用是用来在 `testing` 测试套中进行断言的，通过调用 `func` 函数体，可以让测试套中的断言函数在测试中更方便地进行错误处理和调试。


```
func False(v bool, t *testing.T, msgs ...string) {
	True(!v, t, msgs...)
}

func Err(err error, t *testing.T, msgs ...string) {
	if err == nil {
		t.Fatal(msgs, "error:", err)
	}
}

```

# `/opt/kubo/thirdparty/dir/dir.go`

这段代码定义了一个名为“dir”的包，其中包含一个名为“Writable”的函数。函数的目的是确保创建的目录存在，并且是可读写的。

具体来说，代码首先检查目录是否存在，如果存在，则尝试创建目录并设置其写权限。如果目录不存在，则创建目录并设置其写权限。然后，代码尝试创建一个名为“_check_writable”的文件，如果文件不存在，则尝试创建文件并设置其写权限。如果文件已存在，则尝试打开文件以获取写入权限，否则返回一个错误。

如果上述步骤中出现任何错误，函数将返回一个相应的错误，或者返回一个空错误，表示操作成功。


```
package dir

// TODO move somewhere generic

import (
	"errors"
	"os"
	"path/filepath"
)

// Writable ensures the directory exists and is writable.
func Writable(path string) error {
	// Construct the path if missing
	if err := os.MkdirAll(path, os.ModePerm); err != nil {
		return err
	}
	// Check the directory is writable
	if f, err := os.Create(filepath.Join(path, "._check_writable")); err == nil {
		f.Close()
		os.Remove(f.Name())
	} else {
		return errors.New("'" + path + "' is not writable")
	}
	return nil
}

```

# `/opt/kubo/thirdparty/notifier/notifier.go`

这段代码定义了一个名为`Notifiee`的通用接口，它用于定义客户端实现的通知接口。这个接口多个实现，客户端需要根据自己的需要实现具体的`Notifiee`接口。

接下来，代码引入了两个依赖：`process`包（用于处理进程的信号）和`ratelimit`包（用于限制进程的速率，避免对系统造成过高的负载）。

在`package notifier`的部分，定义了一个名为`processNotifier`函数，这个函数接受一个`process.Proc`作为参数，代表一个正在运行的进程。这个函数的作用是启动一个新进程，用于实现通知的功能。

接下来，定义了一个名为`ratelimitNotifier`函数，这个函数与`processNotifier`函数类似，只不过这个函数使用的是`ratelimit.Ratelimit`包装的`ratelimit.Proc`，而不是普通的`process.Proc`。这个函数的作用是启动一个新进程，用于实现通知的功能，并使用`ratelimit`包限制进程的速率，避免对系统造成过高的负载。


```
// Package notifier provides a simple notification dispatcher
// meant to be embedded in larger structures who wish to allow
// clients to sign up for event notifications.
package notifier

import (
	"sync"

	process "github.com/jbenet/goprocess"
	ratelimit "github.com/jbenet/goprocess/ratelimit"
)

// Notifiee is a generic interface. Clients implement
// their own Notifiee interfaces to ensure type-safety
// of notifications:
```

这是一段定义了RocketNotifiee和Notifiee两个接口的代码。

RocketNotifiee接口定义了Rocket上的通知函数，包括Countdown、LiftedOff、ReachedOrbit、Detached和Landed。

Notifiee接口定义了一个通知代理，负责根据事件类型调用相应的通知函数。

在这段代码中，首先定义了两个接口：RocketNotifiee和Notifiee。接着定义了Rocket上的一个结构体字段notifier，它是notifier.Notifier类型的成员变量。

然后定义了Rocket上的四个通知函数：Countdown、LiftedOff、ReachedOrbit、Detached和Landed。这些函数的实现被放在notifier.Notifier类型的成员变量中。

Notifiee接口定义了一个通知代理，具有composite类型的零值，它包含一个notifier.Notifier类型的字段。根据事件类型调用相应的通知函数，可以推断出该通知代理需要通知的事件类型。

最后，定义了一个名为Rocket上的notifier.Notifier类型的字段，它是Rocket上的notifiee.Notifiee接口的成员变量。


```
//
//	type RocketNotifiee interface{
//	  Countdown(r Rocket, countdown time.Duration)
//	  LiftedOff(Rocket)
//	  ReachedOrbit(Rocket)
//	  Detached(Rocket, Capsule)
//	  Landed(Rocket)
//	}
type Notifiee interface{}

// Notifier is a notification dispatcher. It's meant
// to be composed, and its zero-value is ready to be used.
//
//	type Rocket struct {
//	  notifier notifier.Notifier
```

此代码定义了一个名为 "Notifier" 的结构体，其中包含一个互斥锁 "mu"，一个通知 map "nots"，和一个限制器 "lim"。

"mu" 是一个互斥锁，用于保护 "nots" 中的通知不同时被多个 goroutine 同时访问。"nots" 是一个 map，用于存储每个通知的相关信息。"lim" 是一个限速器，用于限制 goroutine 的数量，以便防止超过指定的限制。

该 "Notifier" 结构体实现了 "rateLimited" 函数，该函数接收一个限制器参数 "limit"，如果限制器为零，则不会限制 goroutine。如果限制器为正整数，则会创建一个通知实例，并返回它。

具体来说，创建通知实例时，会创建一个 "mu" 锁，并且在 "lim" 变量上设置限速器。如果调用 "rateLimited" 函数并传入一个非零限制器，则会创建一个通知实例，并设置其 "lim" 和 "mu" 变量。如果调用 "rateLimited" 函数并传入一个零限制器，则会返回一个空通知实例。


```
//	}
type Notifier struct {
	mu   sync.RWMutex // guards notifiees
	nots map[Notifiee]struct{}
	lim  *ratelimit.RateLimiter
}

// RateLimited returns a rate limited Notifier. only limit goroutines
// will be spawned. If limit is zero, no rate limiting happens. This
// is the same as `Notifier{}`.
func RateLimited(limit int) *Notifier {
	n := &Notifier{}
	if limit > 0 {
		n.lim = ratelimit.NewRateLimiter(process.Background(), limit)
	}
	return n
}

```

这段代码定义了一个名为 `Notify` 的函数，接收一个名为 `e` 的 `Notifiee` 参数。这个函数的作用是在接收者（也就是调用方）进行通知时执行。

具体来说，当有新的 `Notifiee` 需要通知时，函数首先会尝试从内存中的 `nots`  map 中查找任何与该 `Notifiee` 相匹配的键。如果找到了，就设置一个新键并插入新值。否则，在 `nots` map 之外创建一个新的键，并将新值设置为 `{}`。最后，函数会释放 `mu` 锁并确保 `nots` map 再次可读写。

这段代码可以被用来实现在代码中跟随类型安全的函数，例如在组合模式（pattern-following）中使用。


```
// Notify signs up Notifiee e for notifications. This function
// is meant to be called behind your own type-safe function(s):
//
//	// generic function for pattern-following
//	func (r *Rocket) Notify(n Notifiee) {
//	  r.notifier.Notify(n)
//	}
//
//	// or as part of other functions
//	func (r *Rocket) Onboard(a Astronaut) {
//	  r.astronauts = append(r.austronauts, a)
//	  r.notifier.Notify(a)
//	}
func (n *Notifier) Notify(e Notifiee) {
	n.mu.Lock()
	if n.nots == nil { // so that zero-value is ready to be used.
		n.nots = make(map[Notifiee]struct{})
	}
	n.nots[e] = struct{}{}
	n.mu.Unlock()
}

```

这段代码定义了一个名为 `StopNotify` 的函数，它接收一个名为 `e` 的 `Notifiee` 参数。

该函数的作用是停止给定的 `Notifiee` 对象通知，并返回。具体来说，它执行以下操作：

1. 尝试从 `n.nots` 数组中删除给定的 `e` 元素。由于 `n.nots` 是一个有序的切片，因此删除操作将只返回 `n.nots` 中第一个元素，即 `n.nots[0]`。
2. 释放对 `n.mu` 锁的引用，这可能有助于在通知删除后释放资源。

由于 `StopNotify` 函数没有返回值，因此您需要在调用它时提供所需的通知对象。


```
// StopNotify stops notifying Notifiee e. This function
// is meant to be called behind your own type-safe function(s):
//
//	// generic function for pattern-following
//	func (r *Rocket) StopNotify(n Notifiee) {
//	  r.notifier.StopNotify(n)
//	}
//
//	// or as part of other functions
//	func (r *Rocket) Detach(c Capsule) {
//	  r.notifier.StopNotify(c)
//	  r.capsule = nil
//	}
func (n *Notifier) StopNotify(e Notifiee) {
	n.mu.Lock()
	if n.nots != nil { // so that zero-value is ready to be used.
		delete(n.nots, e)
	}
	n.mu.Unlock()
}

```

这段代码定义了一个名为"NotifyAll"的函数，该函数接受一个通知者（Notifier）和一个通知（Notification）。该函数的作用是在通知者的所有通知中执行给定的通知函数，并将通知者作为参数传递给通知函数。

具体来说，这段代码实现了一个私有函数"Launch"，该函数接收一个Rocket类型的实例，然后使用给定的通知函数作为参数传递给"notifyAll"函数，以此来通知所有的通知者。

通过调用"make it private"的方式，该函数只能被"NotifyAll"函数内部使用，从而保证了通知者的通知不会被其他地方滥用。


```
// NotifyAll messages the notifier's notifiees with a given notification.
// This is done by calling the given function with each notifiee. It is
// meant to be called with your own type-safe notification functions:
//
//	func (r *Rocket) Launch() {
//	  r.notifyAll(func(n Notifiee) {
//	    n.Launched(r)
//	  })
//	}
//
//	// make it private so only you can use it. This function is necessary
//	// to make sure you only up-cast in one place. You control who you added
//	// to be a notifiee. If Go adds generics, maybe we can get rid of this
//	// method but for now it is like wrapping a type-less container with
//	// a type safe interface.
```

该代码定义了一个名为 `NotifyAll` 的函数，该函数接受一个名为 `notify` 的函数作为参数，并返回。

函数内部，首先对 `notifier` 变量进行锁定，以确保在函数内部对 `notify` 函数进行操作时只有一个实例被创建。函数内部，如果 `notifier` 为 `nil`，则直接返回，因为此时不需要做任何操作。否则，如果 `lim` 也为 `nil`，则循环遍历 `nots` 数组，并在循环内部调用 `notify` 函数。最后，如果 `lim` 为非 `nil`，则在 `lim` 的 `Go` 函数中循环遍历 `nots` 数组，并为每个 `notify` 函数设置一个限速运行的 `worker`，同时使用 `lim` 进行速率限制。


```
//	func (r *Rocket) notifyAll(notify func(Notifiee)) {
//	  r.notifier.NotifyAll(func(n notifier.Notifiee) {
//	    notify(n.(Notifiee))
//	  })
//	}
//
// Note well: each notification is launched in its own goroutine, so they
// can be processed concurrently, and so that whatever the notification does
// it _never_ blocks out the client. This is so that consumers _cannot_ add
// hooks into your object that block you accidentally.
func (n *Notifier) NotifyAll(notify func(Notifiee)) {
	n.mu.Lock()
	defer n.mu.Unlock()

	if n.nots == nil { // so that zero-value is ready to be used.
		return
	}

	// no rate limiting.
	if n.lim == nil {
		for notifiee := range n.nots {
			go notify(notifiee)
		}
		return
	}

	// with rate limiting.
	n.lim.Go(func(worker process.Process) {
		for notifiee := range n.nots {
			notifiee := notifiee // rebind for loop data races
			n.lim.LimitedGo(func(worker process.Process) {
				notify(notifiee)
			})
		}
	})
}

```

# `/opt/kubo/thirdparty/notifier/notifier_test.go`

这段代码定义了一个名为 "notifier" 的包，它包含了以下结构体：


package notifier


该结构体定义了一个名为 "Router" 的路由器类，以及一个名为 "Notifier" 的通知者类。这两个类都使用了 "testing" 包中的 "RunF投述/n/notify" 函数来通知测试中的结果。

具体来说，这段代码实现了一个通知系统，允许用户注册通知路由器，并在测试中发送测试消息给路由器。当路由器接收到通知消息时，它会通过异步通知的方式将消息传递给所有注册的订阅者。

在 "Router" 类中，通过一个名为 "queue" 的通道来存储测试消息，并使用一个名为 "notifier" 的通知者类来发布通知。在该类中，使用了 "testing" 包中的 "RunF投述/n/notify" 函数来通知测试中的结果。这个函数会等待通知消息队列中的第一个消息，并将其存储到 "queue" 通道中。然后，使用 "fmt" 包中的 "fmt.Printf" 函数来输出通知消息。

在 "testdata.go" 文件中，定义了一些测试数据结构，包括 "Packet" 类型和 "Notifier" 类型。

总之，这段代码实现了一个通知系统，用于在测试中发送测试消息给注册了通知路由器的用户。


```
package notifier

import (
	"fmt"
	"sync"
	"testing"
	"time"
)

// test data structures.
type Router struct {
	queue    chan Packet
	notifier Notifier
}

```

这段代码定义了一个名为 Packet 的结构体，它表示一个未发送的数据包。

定义了一个名为 RouterNotifiee 的接口，它包含三个方法：

	* Enqueued：将一个 Packet 数据包加入路由器的队列中。
	* Forwarded：将一个 Packet 数据包从路由器转发给下一个路由器。
	* Dropped：从路由器删除一个 Packet 数据包。

	* Notify：通知一个 RouterNotifiee 接口关于一个 Packet 数据包的状态变更，例如 Enqueued、Forwarded 或 Dropped。
	* StopNotify：停止通知所有 RouterNotifiee 接口关于一个 Packet 数据包的状态变更，例如 Enqueued、Forwarded 或 Dropped。

	* (r *Router) Notify：实现 Notify 方法，通知一个 RouterNotifiee 接口关于一个 Packet 数据包的状态变更。
	* (r *Router) StopNotify：实现 StopNotify 方法，停止通知所有 RouterNotifiee 接口关于一个 Packet 数据包的状态变更。


```
type Packet struct{}

type RouterNotifiee interface {
	Enqueued(*Router, Packet)
	Forwarded(*Router, Packet)
	Dropped(*Router, Packet)
}

func (r *Router) Notify(n RouterNotifiee) {
	r.notifier.Notify(n)
}

func (r *Router) StopNotify(n RouterNotifiee) {
	r.notifier.StopNotify(n)
}

```

这两段代码定义了两个函数，一个接收者函数和一个通知者函数。

1. 函数接收者函数接收一个包（Packet）并将其传递给其 `Receive` 函数。如果该包被正确地传递给了 `Receive` 函数，那么将包入队到 `r.queue` 队尾，然后调用 `r.notifyAll` 函数通知队列中的所有路由器。通知函数将包装一个通知者（RouterNotifiee），并将其通知队列中的所有路由器。

2. 函数通知者函数接收一个路由器（Router）和一个包（Packet），然后执行以下操作：

  - 如果包已入队，但包中没有路由器，那么将其入队到 `r.queue` 队尾，并调用 `r.notifyAll` 函数通知队列中的所有路由器。

  - 如果包中包含一个路由器，那么将其通知队列中的所有路由器。

注意：`r.notifyAll` 函数是 `func` 函数的回调函数，因此它接收一个通知者（RouterNotifiee）和一个通知函数，而不是接收一个函数作为参数。


```
func (r *Router) notifyAll(notify func(n RouterNotifiee)) {
	r.notifier.NotifyAll(func(n Notifiee) {
		notify(n.(RouterNotifiee))
	})
}

func (r *Router) Receive(p Packet) {
	select {
	case r.queue <- p: // enqueued
		r.notifyAll(func(n RouterNotifiee) {
			n.Enqueued(r, p)
		})

	default: // drop
		r.notifyAll(func(n RouterNotifiee) {
			n.Dropped(r, p)
		})
	}
}

```

该代码定义了一个名为`Router`的结构体`Router`及其方法`Forward()`。

`Forward()`方法接收一个`Router`类型的参数`r`，并使用该参数创建一个指向`Router`的指针`p`。然后，该方法将一个代表`Router`的`RouterNotifye`类型的参数`n`传递给`notifyAll()`方法，该参数接收一个函数作为参数，该函数接收一个`RouterNotifye`类型的参数并执行相应的操作。由于`notifyAll()`方法将整个路由器实例作为参数传递给该函数，因此该函数将遍历整个路由器实例的`RouterNotifye`类型的实例。

对于每个`RouterNotifye`实例，该函数将调用其`Forward()`方法并将`r`和`p`作为参数传递给该方法。然后，该函数将返回，通知路由器继续向后执行。

`Router`结构体定义了一个`Metrics`类型的变量`Metrics`，该变量代表一个路由器的性能指标。该结构体定义了五个变量：

- `enqueued`：路由器队列中已 Enqueued 的元素数量。
- `forwarded`：路由器已向后执行的元素数量。
- `dropped`：路由器已 Dropped 的元素数量。
- `received`：已收到但尚未处理的用户请求数量。
- `sync.Mutex`：一个互斥锁，用于在 `Router` 实例上锁定 `Metrics` 类型的变量，以确保在同一时间只能有一个实例访问它们。


```
func (r *Router) Forward() {
	p := <-r.queue
	r.notifyAll(func(n RouterNotifiee) {
		n.Forwarded(r, p)
	})
}

type Metrics struct {
	enqueued  int
	forwarded int
	dropped   int
	received  chan struct{}
	sync.Mutex
}

```

这两函数主要作用于 Metrics 类型的链表结构， Metrics 链表结构中包含一些实现了 isRouted(int) 函数的节点，这些节点存储了哪些包刷已经转发，以及哪些包还在处理中，所以这两函数的主要作用就是添加新的节点到 Metrics 链表结构中，并更新它的状态。

具体来说，Enqueued 函数接收一个 Router 和一个 Packet 参数，使用 `m.Lock()` 函数获取 Metrics 链表结构的所有节点锁，然后使用 `m.enqueued++` 函数将当前节点 enqueued 计数器加 1，最后使用 `m.Unlock()` 函数释放锁。然后判断是否已经接收到新节点，如果是，则使用 `m.received` 指针和 `struct{}` 构建一个新的 Packet 对象，将新节点加入链表中。

Forwarded 函数与 Enqueued 函数类似，只是处理的是一个 Packet 对象，同样使用了 `m.Lock()` 函数获取 Metrics 链表结构的所有节点锁，然后使用 `m.forwarded++` 函数将当前节点 forwarded 计数器加 1，最后使用 `m.Unlock()` 函数释放锁。然后判断是否已经接收到新节点，如果是，则使用 `m.received` 指针和 `struct{}` 构建一个新的 Packet 对象，将新节点加入链表中。


```
func (m *Metrics) Enqueued(*Router, Packet) {
	m.Lock()
	m.enqueued++
	m.Unlock()
	if m.received != nil {
		m.received <- struct{}{}
	}
}

func (m *Metrics) Forwarded(*Router, Packet) {
	m.Lock()
	m.forwarded++
	m.Unlock()
	if m.received != nil {
		m.received <- struct{}{}
	}
}

```

这两段代码都是函数，但它们的作用不同。

第一段代码的作用是计算并增加一个 Metrics 类型的数据结构中 dropped 的数量。代码首先获取 Metrics 类型中的锁，然后将 dropped 的值设置为 m.dropped，最后使用 Unlock 函数释放锁。如果 m.received 变量不等于 nil，则使用 m.received 包中的一个空 struct 发送给 enqueued 变量，然后使用 m.enqueued 和 m.forwarded 变量更新 Enqueued 和 Forwarded 变量。

第二段代码的作用是计算并返回 Metrics 类型中有关路由器接收、发送数据包数量的信息。代码首先使用 Lock 函数获取 Metrics 类型中的锁，然后使用 range 循环计算并打印 Enqueued、Forwarded 和 dropped 变量。最后，代码使用 fmt.Sprintf 函数将计算结果格式化并返回。


```
func (m *Metrics) Dropped(*Router, Packet) {
	m.Lock()
	m.dropped++
	m.Unlock()
	if m.received != nil {
		m.received <- struct{}{}
	}
}

func (m *Metrics) String() string {
	m.Lock()
	defer m.Unlock()
	return fmt.Sprintf("%d enqueued, %d forwarded, %d in queue, %d dropped",
		m.enqueued, m.forwarded, m.enqueued-m.forwarded, m.dropped)
}

```

该代码的作用是测试一个名为 "TestNotifies" 的函数，该函数接收一个名为 "t" 的测试断言对象，并对其进行测试。

具体来说，该函数创建了一个名为 "m" 的 Metrics 对象，该对象包含一个名为 "received" 的通道，该通道接收一个 "Packet" 类型的数据。

同时，该函数还创建了一个名为 "r" 的 Router 对象，该对象包含一个名为 "queue" 的通道，该通道接收一个名为 "Packet" 类型的数据。

接下来，该函数使用一个循环来接收 Packet 数据并将其发送给 "m.received" 通道。在循环中，如果 "m.enqueued" 通道的值不是 "1 + i"，则函数会输出一条错误消息，并指出 "m.enqueued" 和 "1 + i" 之间的差异。

最后，该函数使用另一个循环来接收 Packet 数据并将其发送给 "m.received" 通道。在循环中，如果 "m.enqueued" 通道的值不是 10，则函数会输出一条错误消息，并指出 "m.enqueued" 和 10 之间的差异。


```
func TestNotifies(t *testing.T) {
	m := Metrics{received: make(chan struct{})}
	r := Router{queue: make(chan Packet, 10)}
	r.Notify(&m)

	for i := 0; i < 10; i++ {
		r.Receive(Packet{})
		<-m.received
		if m.enqueued != (1 + i) {
			t.Error("not notifying correctly", m.enqueued, 1+i)
		}

	}

	for i := 0; i < 10; i++ {
		r.Receive(Packet{})
		<-m.received
		if m.enqueued != 10 {
			t.Error("not notifying correctly", m.enqueued, 10)
		}
		if m.dropped != (1 + i) {
			t.Error("not notifying correctly", m.dropped, 1+i)
		}
	}
}

```

该代码使用了 gRU丁选机标准测试不是办法停止输出通知消息容器接收所有的 Packet 并检查 m.enqueued 变量是否为 5 的和不是停止输出通知消息容器接收所有的 Packet 并检查 m.enqueued 变量是否为 5。

具体来说，该代码实现了以下功能：

1. 创建一个 Metrics 对象 m，其中 received 字段存储一个通道，每次会发送一个 Packet 数据到来时产生一个新的 Packet 并发送到接收者。
2. 创建一个 Router 对象 r，其中 queue 字段存储一个通道，每次收到 Packet 数据时将数据发送到 queue 通道。
3. 在 for 循环中，向 received 通道发送五个 Packet，然后循环接收每一个 Packet 并检查 m.received 变量是否等于该 Packet 中的 received 字段，如果不是，则说明 notifier 函数没有正常工作。
4. 在 for 循环中，向 received 通道发送五个 Packet，然后循环接收每一个 Packet 并检查 m.received 变量是否等于该 Packet 中的 received 字段，如果 m.received 和该 Packet 中的 received 相等，则说明 notifier 函数已经正常工作，接下来就停止 notifier 函数。
5. 循环接收每一个 Packet 并检查 m.enqueued 变量是否等于 5。

函数接收每一个 Packet 并检查 m.received 变量是否等于该 Packet 中的 received 字段。不是，说明 notifier 函数没有正常工作。
函数接收五个 Packet，停止 notifier 函数。
函数接收五个 Packet，停止 notifier 函数。
函数接收五个 Packet，停止 notifier 函数。


```
func TestStopsNotifying(t *testing.T) {
	m := Metrics{received: make(chan struct{})}
	r := Router{queue: make(chan Packet, 10)}
	r.Notify(&m)

	for i := 0; i < 5; i++ {
		r.Receive(Packet{})
		<-m.received
		if m.enqueued != (1 + i) {
			t.Error("not notifying correctly")
		}
	}

	r.StopNotify(&m)

	for i := 0; i < 5; i++ {
		r.Receive(Packet{})
		select {
		case <-m.received:
			t.Error("did not stop notifying")
		default:
		}
		if m.enqueued != 5 {
			t.Error("did not stop notifying")
		}
	}
}

```

这段代码的作用是测试一个名为 `TestThreadsafe` 的函数，该函数使用 `TestThreadsafe` 函数作为测试套件，并通过一系列的 `if` 语句和 `t.log` 函数来输出结果。

具体来说，这段代码会创建一个 `Router` 实例，该实例有一个入队队列 `queue`，以及两个出队队列 `metrics` 实例 `m1` 和 `m2`。然后，该实例会调用 `Notify` 方法并传递两个 `Metrics` 实例，来通知队列中有数据到。

接下来，该实例会创建一个 `for` 循环，其中的 `i` 变量用于计数，用来遍历队列中的数据。在循环内部的 `go` 子句中，会执行一个 `Receive` 操作，来从队列中读取数据并保存到 `metrics` 实例的 `received` 通道中。

接着，该实例会根据当前计数器的值 % 3，来决定是否执行下一个 `Receive` 操作。如果当前计数器的值 % 3 不等于 0，那么该实例就会执行下面的语句，这些语句会在循环内部遍历 `queue` 中的所有数据，并使用 `Forward` 方法将这些数据发送到 `metrics` 实例的 `received` 通道中。

循环结束后，该实例会调用 `Drain` 方法并传递两个 `Metrics` 实例，来通知队列已经没有数据到。

接下来，该实例会输出 `m1`、`m2` 和 `m3` 实例的接收到的数据，并比较这三个接收到的数据是否相同。如果三个接收到的数据不相同，那么该实例就会输出错误信息并错误地检测到 counts 不一致的情况。


```
func TestThreadsafe(t *testing.T) {
	N := 1000
	r := Router{queue: make(chan Packet, 10)}
	m1 := Metrics{received: make(chan struct{})}
	m2 := Metrics{received: make(chan struct{})}
	m3 := Metrics{received: make(chan struct{})}
	r.Notify(&m1)
	r.Notify(&m2)
	r.Notify(&m3)

	var n int
	var wg sync.WaitGroup
	for i := 0; i < N; i++ {
		n++
		wg.Add(1)
		go func() {
			defer wg.Done()
			r.Receive(Packet{})
		}()

		if i%3 == 0 {
			n++
			wg.Add(1)
			go func() {
				defer wg.Done()
				r.Forward()
			}()
		}
	}

	// drain queues
	for i := 0; i < (n * 3); i++ {
		select {
		case <-m1.received:
		case <-m2.received:
		case <-m3.received:
		}
	}

	wg.Wait()

	// counts should be correct and all agree. and this should
	// run fine under `go test -race -cpu=5`

	t.Log("m1", m1.String())
	t.Log("m2", m2.String())
	t.Log("m3", m3.String())

	if m1.String() != m2.String() || m2.String() != m3.String() {
		t.Error("counts disagree")
	}
}

```

此代码定义了一个名为 `highwatermark` 的结构体，其中包含一个整数类型的 `mu` 成员变量、一个整数类型的 `mark` 成员变量和一个整数类型的 `limit` 成员变量，还有一个名为 `errs` 的通道类型成员变量，用于传递错误信息。

该 `highwatermark` 结构体定义了一个 `incr` 方法，该方法使用了 `mu` 成员变量，因为 `mu` 是 `sync.Mutex` 类型，可以保证在同一时间只有一个锁对可以访问。

`incr` 方法的实现如下：

1. 获取 `mu` 成员变量的引用，并加1，将结果存储回 `mark` 成员变量中。
2. 如果 `mark` 成员变量已经达到了 `limit`，则调用 `errs` 通道中的 `fmt.Errorf` 函数，并将以下错误消息作为参数传入：`"went over limit: %d/%d"`(其中 `%d` 是 `limit` 的下标，`/` 是 `limit` 与 `mark` 之间分隔符)。
3. 释放 `mu` 成员变量，以便下一轮访问时可以重新加锁。

该 `highwatermark` 结构体可以被用于锁定一组对同一时间有限制的高水标记(float、int 等)，以防止同时有多个进程对它们进行写操作，从而避免潜在的并发问题。


```
type highwatermark struct {
	mu    sync.Mutex
	mark  int
	limit int
	errs  chan error
}

func (m *highwatermark) incr() {
	m.mu.Lock()
	m.mark++
	// fmt.Println("incr", m.mark)
	if m.mark > m.limit {
		m.errs <- fmt.Errorf("went over limit: %d/%d", m.mark, m.limit)
	}
	m.mu.Unlock()
}

```

This is a Go test that checks the behavior of a highwatermarker when it is rate limited. The highwatermarker is given a timeout of 10 seconds and a limit of 9.

The `highwatermark` function is defined with a timeout and a buffer of 100 errs. It is responsible for the rate limiting.

The `RateLimited` function is defined to use the `highwatermark` function.

The `TestLimited` function creates a highwatermark with a limit of 3 and a timeout of 10 seconds. It also creates a `RateLimited` instance that stops after 3 rounds.

The `_` is followed by an asterisk `_` which is used to indicate that the tests in this package are independent of each other.

The `intro` part is the type of the test.

The `select` methods are used to wait for the events that are being observed by the ratelimited.

The `NotifyAll` is a callback that is used to register a closure that will be invoked by the highwatermark.

The `close` method is used to close the `errs` channel that is passed as an argument to the `RateLimited` function.


```
func (m *highwatermark) decr() {
	m.mu.Lock()
	m.mark--
	// fmt.Println("decr", m.mark)
	if m.mark < 0 {
		m.errs <- fmt.Errorf("went under zero: %d/%d", m.mark, m.limit)
	}
	m.mu.Unlock()
}

func TestLimited(t *testing.T) {
	timeout := 10 * time.Second // huge timeout.
	limit := 9

	hwm := highwatermark{limit: limit, errs: make(chan error, 100)}
	n := RateLimited(limit) // will stop after 3 rounds
	n.Notify(1)
	n.Notify(2)
	n.Notify(3)

	entr := make(chan struct{})
	exit := make(chan struct{})
	done := make(chan struct{})
	go func() {
		for i := 0; i < 10; i++ {
			// fmt.Printf("round: %d\n", i)
			n.NotifyAll(func(e Notifiee) {
				hwm.incr()
				entr <- struct{}{}
				<-exit // wait
				hwm.decr()
			})
		}
		done <- struct{}{}
	}()

	for i := 0; i < 30; {
		select {
		case <-entr:
			continue // let as many enter as possible
		case <-time.After(1 * time.Millisecond):
		}

		// let one exit
		select {
		case <-entr:
			continue // in case of timing issues.
		case exit <- struct{}{}:
		case <-time.After(timeout):
			t.Error("got stuck")
		}
		i++
	}

	select {
	case <-done: // two parts done
	case <-time.After(timeout):
		t.Error("did not finish")
	}

	close(hwm.errs)
	for err := range hwm.errs {
		t.Error(err)
	}
}

```

# `/opt/kubo/thirdparty/unit/unit.go`

这段代码定义了一个名为 "Information" 的枚举类型，它有两个成员变量：

- `_` 是一个匿名成员变量，初始化为 iota，相当于 `nextInt(0)` 函数，用于生成下一个成员变量的值。
- `KB`、`MB`、`GB`、`TB` 和 `PB` 是枚举类型的成员变量，分别对应 10、100、1000 和 10000 的阶乘，可以用作计数单位，存储每个成员变量的值。
- `fmt.Printf` 函数用于格式化输出，输出每个成员变量的值。

所以，整个枚举类型 `Information` 可以被看作是一个定义了不同阶数下计数单位的信息，输出的结果是：

- 如果 `i` 是偶数，输出对应阶数的计数单位。
- 如果 `i` 是奇数，输出对应阶数的计数单位并输出一个换行符。


```
package unit

import "fmt"

type Information int64

const (
	_  Information = iota // ignore first value by assigning to blank identifier
	KB             = 1 << (10 * iota)
	MB
	GB
	TB
	PB
	EB
)

```

这是一段 TypeScript 代码，定义了一个名为 `func` 的函数，该函数接收一个整数参数 `i`，并返回一个字符串类型的值。

函数的实现主要分为以下几步：

1. 将整数参数 `i` 转换成浮点数类型，名为 `tmp`。
2. 根据 `i` 与给定常量的大小关系，将 `tmp` 转换成对应的字符串类型。
3. 遍历类型 `switch` 中的各个分支，根据当前分支计算 `tmp` 除以对应常量的商，并将字符串类型设置为对应常量的符号。
4. 返回 `tmp` 和对应符号的字符串类型。

这段代码主要实现了将一个整数参数转换成对应的字符串类型功能。


```
func (i Information) String() string {
	tmp := int64(i)

	// default
	d := tmp
	symbol := "B"

	switch {
	case i > EB:
		d = tmp / EB
		symbol = "EB"
	case i > PB:
		d = tmp / PB
		symbol = "PB"
	case i > TB:
		d = tmp / TB
		symbol = "TB"
	case i > GB:
		d = tmp / GB
		symbol = "GB"
	case i > MB:
		d = tmp / MB
		symbol = "MB"
	case i > KB:
		d = tmp / KB
		symbol = "KB"
	}
	return fmt.Sprintf("%d %s", d, symbol)
}

```

# `/opt/kubo/thirdparty/unit/unit_test.go`

这段代码是一个名为 "unit" 的包，其中包含了一些函数来测试不同 "容量" 大小的整数是否符合预期的字节数。

具体来说，这段代码的作用是检查以下条件是否成立：

- 如果 1 千兆字节(KB)不等于 1 兆字节(MB)，那么程序会崩溃并打印错误信息。
- 如果 1 兆字节(MB)不等于 1 吉字节(GB)，那么程序会失败并打印错误信息。
- 如果 1 吉字节(GB)不等于 1 太字节(TB)，那么程序会失败并打印错误信息。
- 如果 1 太字节(TB)不等于 1 拍字节(PB)，那么程序会失败并打印错误信息。
- 如果 1 拍字节(PB)不等于 1 艾字节(EB)，那么程序会失败并打印错误信息。

这些测试用例基于一组已知的容量单位，包括千克(KB)、兆字节(MB)、吉字节(GB)、太字节(TB)和拍字节(PB)，以及一个尚未知的单位 "艾字节"(EB)。程序将检查这些单位是否符合预期的字节数。如果任何一个测试用例失败，程序将会崩溃并打印错误信息。


```
package unit

import "testing"

// and the award for most meta goes to...

func TestByteSizeUnit(t *testing.T) {
	if 1*KB != 1*1024 {
		t.Fatal(1 * KB)
	}
	if 1*MB != 1*1024*1024 {
		t.Fail()
	}
	if 1*GB != 1*1024*1024*1024 {
		t.Fail()
	}
	if 1*TB != 1*1024*1024*1024*1024 {
		t.Fail()
	}
	if 1*PB != 1*1024*1024*1024*1024*1024 {
		t.Fail()
	}
	if 1*EB != 1*1024*1024*1024*1024*1024*1024 {
		t.Fail()
	}
}

```

# `/opt/kubo/thirdparty/verifbs/verifbs.go`

该代码定义了一个名为 VerifBSGC 的 struct，它包含一个指向名为 bstore 的 gcBlockstore 的引用。该 struct 还包含一个名为 VerifCID 的成员变量，该成员变量引用了一个名为 verifCID 的 gcCID，以及一个名为 blocks 的成员变量，该成员变量引用了一个名为 blocks 的 gcBlockFormat。

VerifBSGC  struct 代表了一个基于 Boxo 协议的验证事务，可以在 Boxo 网络中使用。通过使用 bstore 和 verifCID，可以安全地在 Boxo 网络中存储和验证数据。使用 blocks 和 cid 可以生成并验证 blocks， blocks 是块的 JSON 格式描述，cid 是 Content ID，用于标识和跟踪文件的来源。


```
package verifbs

import (
	"context"

	bstore "github.com/ipfs/boxo/blockstore"
	"github.com/ipfs/boxo/verifcid"
	blocks "github.com/ipfs/go-block-format"
	cid "github.com/ipfs/go-cid"
)

type VerifBSGC struct {
	bstore.GCBlockstore
}

```

这段代码定义了两个函数，一个是`Put`函数，另一个是`PutMany`函数。这两个函数的作用是向一个块系统中的验证后提交(Confirmation)区块链中添加多个块。

具体来说，这两个函数都会接收一个块(Block)对象作为参数，并使用一个块系统中的` GCBlockstore`实例来执行操作。`Put`函数会在接收的块对象之后尝试验证其是否符合系统的允许名单(Allowlist)。如果是，则返回块的` Put`操作，否则返回一个错误。`PutMany`函数会接收多个块并尝试验证它们是否符合系统的允许名单。如果在验证过程中任何块都不符合允许名单，函数将返回一个错误。

两个函数的实现都在一个名为`VerifBSGC`的`Verif`链上下文中定义，该上下文提供了一个块系统中的` GCBlockstore`实例，可以让用户通过`Put`和`PutMany`函数来执行块的写入和提交操作。


```
func (bs *VerifBSGC) Put(ctx context.Context, b blocks.Block) error {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
		return err
	}
	return bs.GCBlockstore.Put(ctx, b)
}

func (bs *VerifBSGC) PutMany(ctx context.Context, blks []blocks.Block) error {
	for _, b := range blks {
		if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
			return err
		}
	}
	return bs.GCBlockstore.PutMany(ctx, blks)
}

```

此代码定义了一个名为 VerifBS 的 struct，该 struct 实现了 blockstore.Blockstore 和 verifcid.Verifcid 的接口。

func (bs *VerifBS) Get(ctx context.Context, c cid.Cid) (blocks.Block, error) {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c); err != nil {
		return nil, err
	}
	return bs.GCBlockstore.Get(ctx, c)
}

func (bs *VerifBS) Put(ctx context.Context, b blocks.Block) error {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
		return err
	}
	return bs.Blockstore.Put(ctx, b)
}

该代码创建了一个名为 VerifBS 的 struct，该 struct 实现了 blockstore.Blockstore 和 verifcid.Verifcid 的接口。

在 `Get` 函数中，使用 `verifcid.ValidateCid` 验证传入的 `c` 是否属于指定的允许列表，如果验证失败，则返回 `nil` 和相应的错误。

在 `Put` 函数中，验证传入的 `b` 是否属于指定的允许列表，如果验证失败，则返回 `nil` 和相应的错误。

另外，`blockstore.GCBlockstore` 和 `verifcid.Verifcid` 是直接从 `blockstore` 和 `verifcid` 包中使用的。


```
func (bs *VerifBSGC) Get(ctx context.Context, c cid.Cid) (blocks.Block, error) {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c); err != nil {
		return nil, err
	}
	return bs.GCBlockstore.Get(ctx, c)
}

type VerifBS struct {
	bstore.Blockstore
}

func (bs *VerifBS) Put(ctx context.Context, b blocks.Block) error {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
		return err
	}
	return bs.Blockstore.Put(ctx, b)
}

```

此代码定义了两个函数，分别为`PutMany`和`Get`函数。

1. `PutMany`函数接收一个`VerifBS`类型的参数`bs`和一个包含`blocks`块的切片`blks`，并在`ctx`上下文上下文中执行。该函数的作用是验证给定的`cid`是否属于给定的白名单中，如果是，则返回操作的成功。

2. `Get`函数接收一个`context.Context`和一个白名单`c`，并在`ctx`上下文上下期内执行。该函数的作用是从白名单中获取指定的`cid`，并返回相应的块。如果给定的`cid`不在白名单中，函数将返回`nil`。


```
func (bs *VerifBS) PutMany(ctx context.Context, blks []blocks.Block) error {
	for _, b := range blks {
		if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
			return err
		}
	}
	return bs.Blockstore.PutMany(ctx, blks)
}

func (bs *VerifBS) Get(ctx context.Context, c cid.Cid) (blocks.Block, error) {
	if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c); err != nil {
		return nil, err
	}
	return bs.Blockstore.Get(ctx, c)
}

```

# `/opt/kubo/tracing/doc.go`

这段代码是一个 Go 语言写的开源库中的跟踪逻辑配置代码。它用于配置用于 go-ipfs 的跟踪器，并帮助在栈中保持一致的命名约定。

首先，它定义了一个名为 "tracing" 的环境变量，用于设置跟踪器的选项。它通过设置 OTEL_TRACES_EXPORTER 环境变量来指定要使用的跟踪器 exporter，包括 otlp、jaeger、zipkin 和 file。

然后，它通过判断环境变量中是否包含 "otlp"、"jaeger" 或 "zipkin" 来决定是否使用这些 exporter。如果环境中包含了这三个或更多 exporter，那么它将 configure the tracer to use them，并设置 "tracing=实验性" 环境变量来告诉您的应用程序此正在使用实验性跟踪器。

最后，它通过在配置文件中设置一系列选项来进一步配置跟踪器的行为。它包括设置跟踪器输出目录、设置 exporter 的时间跨度以及设置跟踪器的一些选项，如不输出 span、不存储 span 等。


```
// Package tracing contains the tracing logic for go-ipfs, including configuring the tracer and
// helping keep consistent naming conventions across the stack.
//
// NOTE: Tracing is currently experimental. Span names may change unexpectedly, spans may be removed,
// and backwards-incompatible changes may be made to tracing configuration, options, and defaults.
//
// Tracing is configured through environment variables, as consistent with the OpenTelemetry spec as possible:
//
// https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/sdk-environment-variables.md
//
// OTEL_TRACES_EXPORTER: a comma-separated list of exporters:
//   - otlp
//   - jaeger
//   - zipkin
//   - file
```

这段代码定义了一系列环境变量，具体作用取决于将这些变量分配给了哪个出口。这些变量通常被称为标准环境变量，包括但不限于OTEL_EXPORTER_JAEGER_AGENT_HOST、OTEL_EXPORTER_JAEGER_AGENT_PORT、OTEL_EXPORTER_JAEGER_ENDPOINT、OTEL_EXPORTER_JAEGER_USER和OTEL_EXPORTER_JAEGER_PASSWORD。这些变量对于使用Jaeger和OTLP HTTP/gRPC等出口 exporter 来说非常重要，进口 exporter 不需要关心这些变量。


```
//
// Different exporters have their own set of environment variables, depending on the exporter. These are typically
// standard environment variables. Some common ones:
//
// Jaeger:
//
//   - OTEL_EXPORTER_JAEGER_AGENT_HOST
//   - OTEL_EXPORTER_JAEGER_AGENT_PORT
//   - OTEL_EXPORTER_JAEGER_ENDPOINT
//   - OTEL_EXPORTER_JAEGER_USER
//   - OTEL_EXPORTER_JAEGER_PASSWORD
//
// OTLP HTTP/gRPC:
//
//   - OTEL_EXPORTER_OTLP_PROTOCOL
```

这段代码定义了一个名为"otel-exporter"的OTLP（OpenTracing Logging）exporter，可用于将Zipkin和File格式的日志数据导出到grpc和http/protobuf协议。

具体来说，这段代码定义了以下六个OTLP选项：

1. "grpc"：用于以grpc协议导出OTLP数据。
2. "http/protobuf": 用于以http/protobuf协议导出OTLP数据。
3. "otel-exporter-otlp-endpoint": Zipkin服务的出口点，用于导出OTLP数据。
4. "otel-exporter-otlp-certificate": 用于身份验证的证书，用于导出OTLP数据。
5. "otel-exporter-otlp-headers": 导出OTLP数据的头部信息。
6. "otel-exporter-otlp-compression": 数据压缩选项，用于压缩OTLP数据。
7. "otel-exporter-otlp-timeout": 设置Zipkin服务的超时时间（秒）。

当这个OTLP exporter被部署到Kubernetes后，可以通过以下方式来启动它：

1. 使用Kubernetes Deployment：创建一个Deployment，将otel-exporter的source代码、Docker镜像、env变量、values.yaml文件、 and一个MatplotlibConfig文件一起部署进去。
2. 使用Kubernetes Service：创建一个Service，将otel-exporter的port 8080、endpoint、path和tokenizer一起指定，并将其部署到Kubernetes集群中。


```
//     one of [grpc, http/protobuf]
//     default: grpc
//   - OTEL_EXPORTER_OTLP_ENDPOINT
//   - OTEL_EXPORTER_OTLP_CERTIFICATE
//   - OTEL_EXPORTER_OTLP_HEADERS
//   - OTEL_EXPORTER_OTLP_COMPRESSION
//   - OTEL_EXPORTER_OTLP_TIMEOUT
//
// Zipkin:
//
//   - OTEL_EXPORTER_ZIPKIN_ENDPOINT
//
// File:
//
//   - OTEL_EXPORTER_FILE_PATH
```

这段代码是一个用于配置文件，指定了用于写入JSON副本的路径。如果没有指定路径，则默认为当前工作目录下的traces.json文件。

代码中使用了docker run命令，用于运行Docker容器。在这里，我们创建了一个名为"jaeger"的容器，设置了一些环境变量，以便在容器中运行Jaeger经纪人。

具体来说，这段代码的作用是配置一个Docker容器，将包含Jaeger经纪人组件的环境变量，并指定Jaeger经纪人代理的端口，以便在代理的端口上监听来自代理的 trace。同时，还可以配置其他代理的端口，以便在代理的端口上监听来自代理的 trace。


```
//     file path to write JSON traces
//     default: `$PWD/traces.json`
//
// For example, if you run a local IPFS daemon, you can use the jaegertracing/all-in-one Docker image to run
// a full Jaeger stack and configure go-ipfs to publish traces to it:
//
//	docker run -d --name jaeger \
//	  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
//	  -p 5775:5775/udp \
//	  -p 6831:6831/udp \
//	  -p 6832:6832/udp \
//	  -p 5778:5778 \
//	  -p 16686:16686 \
//	  -p 14268:14268 \
//	  -p 14269:14269 \
```

这段代码是一个 Go 语言编写的 Jaeger UI 示例，它使用了指定了要跟踪的 Jaeger TracerProvider。

具体来说，这段代码导入了包括 Jaeger UI 在内的多个依赖，包括OTEL_TRACES_EXPORTER、jaeger、ipfs、daemon等。它还定义了一个名为OTEL_TRACES_EXPORTER的变量，并将其设置为指向全局变量jaeger的daemon的IPFS名字。

通过调用jaeger的ipfs daemon，它将会在 Jaeger UI 中打开一个http://localhost:16686的界面，用户可以在这个界面中查看和分析 spans。

此外，这段代码还定义了一个名为RequestedSpan的变量，它是一个 span，可能会用于在一些验证请求的操作中。


```
//	  -p 14250:14250 \
//	  -p 9411:9411 \
//	  jaegertracing/all-in-one
//	OTEL_TRACES_EXPORTER=jaeger ipfs daemon
//
//	# In this example the Jaeger UI is available at http://localhost:16686.
//
// Span names follow a convention of <Component>.<Span>, some examples:
//
//   - component=Gateway + span=Request -> Gateway.Request
//   - component=CoreAPI.PinAPI + span=Verify.CheckPin -> CoreAPI.PinAPI.Verify.CheckPin
//
// We follow the OpenTelemetry convention of using whatever TracerProvider is registered globally.
package tracing

```

# `/opt/kubo/tracing/tracing.go`

这段代码是一个 Go 语言中的 package，它定义了一个名为 "tracing" 的包。通过导入其他包，它实现了对整个应用程序的跟踪，并将其输出到设置的输出设备。

具体来说，这段代码实现了一个完整的 Tracer 中间件，用于记录应用程序中的每个 API 调用。通过使用 OpenTelemetry SDK，这段代码可以捕获跟踪数据，并将其存储到配置的输出设备，如文件、日志、控制台等。在代码中，首先定义了一个名为 "context" 的常量，用于存储当前正在处理的任务。接着，定义了一个名为 "fmt" 的函数，用于将跟踪数据格式化为字符串。

紧接着，导入了一系列来自其他包的函数和变量，包括：

- "github.com/ipfs/boxo/tracing"：来自 "boxo" 包的 "Tracer" 函数；
- "github.com/ipfs/kubo"：来自 "kubo" 包的 "Scope" 和 "Service" 函数；
- "go.opentelemetry.io/otel" 和 "go.opentelemetry.io/otel/sdk/trace"：来自 "opentelemetry" 包的 "Tracer" 和 "RateLimiter" 函数；
- "go.opentelemetry.io/otel/semconv/v1.4.0"：来自 "semconv" 包的 "SemanticConverter" 函数；
- "traceapi"：来自 "traceapi" 包的 "NewTracer" 和 "NewSdkContext" 函数。

通过这些函数和变量，可以创建一个 Tracer 实例，并设置跟踪配置，例如，将输出设备设置为文件，每秒 50 秒记录一次数据。在 "main" 函数中，创建了一个新的上下文实例，并使用上面创建的 Tracer 记录一个 API 调用。最后，输出了记录的数据，格式化为字符串并输出到控制台。


```
package tracing

import (
	"context"
	"fmt"

	"github.com/ipfs/boxo/tracing"
	version "github.com/ipfs/kubo"
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/sdk/resource"
	"go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
	traceapi "go.opentelemetry.io/otel/trace"
)

```

此代码定义了一个名为“shutdownTracerProvider”的接口，以及一个名为“noopShutdownTracerProvider”的结构体。

shutdownTracerProvider接口定义了一个名为“Tracer”的函数，该函数接受一个“instrumentationName”参数和一个可选的“opts”参数，然后返回一个实现了traceapi.Tracer接口的函数。注意，此代码不直接使用提供的TracerProvider接口，以避免在将新方法添加到接口时导致go-ipfs崩溃。

noopShutdownTracerProvider结构体实现了一个名为“Shutdown”的函数，该函数接受一个“ctx”参数，然后返回一个“error”类型的函数。

最后，该代码创建了一个名为“NewTracerProvider”的函数，该函数接收一个TracerProvider接口的实现，然后使用它创建和配置一个TracerProvider。


```
// shutdownTracerProvider adds a shutdown method for tracer providers.
//
// Note that this doesn't directly use the provided TracerProvider interface
// to avoid build breaking go-ipfs if new methods are added to it.
type shutdownTracerProvider interface {
	Tracer(instrumentationName string, opts ...traceapi.TracerOption) traceapi.Tracer
	Shutdown(ctx context.Context) error
}

// noopShutdownTracerProvider adds a no-op Shutdown method to a TracerProvider.
type noopShutdownTracerProvider struct{ traceapi.TracerProvider }

func (n *noopShutdownTracerProvider) Shutdown(ctx context.Context) error { return nil }

// NewTracerProvider creates and configures a TracerProvider.
```

此代码定义了一个名为 `NewTracerProvider` 的函数，它接收一个 `context.Context` 并返回一个 `shutdownTracerProvider` 和一个错误。

函数的作用是创建一个新的 Tracer Provider。根据 Tracing Provider 的类型，此函数使用了一个 Shutdown Tracer Provider，该 Tracer Provider 不会在日志中写入任何信息，而只会等待关闭。

具体实现过程如下：

1. 创建一个空 Tracer Provider，如果失败则返回一个错误。
2. 获取所有的 exporter，并将它们添加到 Tracer Provider 选项中。每个 exporter 都是一个 Tracer Provider，它可以在 batcher 上使用，因此函数需要遍历所有 exporter 并添加它们到选项中。
3. 如果选项中没有 exporter，则函数返回一个 Noop Tracer Provider，该 Tracer Provider 不会写入任何信息。
4. 创建一个新的 Tracer Provider，它使用了一个 Shutdown Tracer Provider，该 Tracer Provider 不会在日志中写入任何信息。
5. 创建一个新的资源，并使用 `resource.Merge` 函数将资源设置为默认值，然后设置服务名称和版本，最后使用 `resource.NewSchemaless` 函数创建一个默认的命名空间。
6. 将步骤 3 到 5 中创建的所有内容添加到 Tracer Provider 选项中，并返回 Tracer Provider。

函数的副作用包括：

1. 创建了一个新的 Tracer Provider。
2. 设置了一个默认的命名空间。
3. 创建了一个新的 Tracer Provider。


```
func NewTracerProvider(ctx context.Context) (shutdownTracerProvider, error) {
	exporters, err := tracing.NewSpanExporters(ctx)
	if err != nil {
		return nil, err
	}
	if len(exporters) == 0 {
		return &noopShutdownTracerProvider{TracerProvider: traceapi.NewNoopTracerProvider()}, nil
	}

	options := []trace.TracerProviderOption{}

	for _, exporter := range exporters {
		options = append(options, trace.WithBatcher(exporter))
	}

	r, err := resource.Merge(
		resource.Default(),
		resource.NewSchemaless(
			semconv.ServiceNameKey.String("Kubo"),
			semconv.ServiceVersionKey.String(version.CurrentVersionNumber),
		),
	)
	if err != nil {
		return nil, err
	}
	options = append(options, trace.WithResource(r))

	return trace.NewTracerProvider(options...), nil
}

```

此代码定义了一个名为 `Span` 的函数，它使用遗留的 IPFS 调试惯例创建了一个新的 spans。

具体来说，函数的第一个参数 `ctx` 是上下文句柄，用于提供跨越API服务器和组件之间的数据传递。第二个参数 `componentName` 是用于指定要跟踪的组件名称，第三个参数 `spanName` 是用于标识新生成的跨度名称。最后一个参数 `opts...` 是传递给 `traceapi.SpanStartOption` 的选项参数，其中 `traceapi.SpanStartOption` 是 `traceapi` 包中定义的选项参数类型。

函数返回两个值：一个是 `ctx` 上下文，另一个是新生成的 `traceapi.Span`。函数内部使用 `otel.Tracer("Kubo").Start` 来开始一个新的跟踪旅程，并传递给 `traceapi.SpanStartOption` 类型的参数，用于开始一个新的跨度。


```
// Span starts a new span using the standard IPFS tracing conventions.
func Span(ctx context.Context, componentName string, spanName string, opts ...traceapi.SpanStartOption) (context.Context, traceapi.Span) {
	return otel.Tracer("Kubo").Start(ctx, fmt.Sprintf("%s.%s", componentName, spanName), opts...)
}

```