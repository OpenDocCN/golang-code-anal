# v2ray-core源码解析 31

# `common/signal/pubsub/pubsub_test.go`

这段代码是测试Pubsub协议的订阅和发布功能。具体来说，它通过使用v2ray.com的pubsub包实现一个简单的Pubsub服务，包括一个订阅者和一个发布者。在测试中，它创建了一个名为"test"的服务，并订阅了名为"a"的频道。然后，它发布了一个批次到频道"a"中的值为1的消息。接下来，它等待消息被接收，并检查消息是否为1。如果是1，则说明订阅者正确接收了消息，如果不是，则说明出现了错误。然后，它又发布了一个批次到频道"a"中的值为2的消息。接下来，它等待消息被接收，并检查消息是否为2。如果是2，则说明发布者正确发送了消息，如果不是，则说明出现了错误。最后，它还清理了服务并关闭了订阅者。


```go
package pubsub_test

import (
	"testing"

	. "v2ray.com/core/common/signal/pubsub"
)

func TestPubsub(t *testing.T) {
	service := NewService()

	sub := service.Subscribe("a")
	service.Publish("a", 1)

	select {
	case v := <-sub.Wait():
		if v != 1 {
			t.Error("expected subscribed value 1, but got ", v)
		}
	default:
		t.Fail()
	}

	sub.Close()
	service.Publish("a", 2)

	select {
	case <-sub.Wait():
		t.Fail()
	default:
	}

	service.Cleanup()
}

```

# `common/signal/semaphore/semaphore.go`

这段代码定义了一个名为“semaphore”的包，并实现了一个名为“Instance”的结构体，该结构体包含一个名为“token”的通道类型，可以动态增加通道长度。

具体来说，该代码创建了一个名为“s”的结构体实例，该实例包含一个名为“token”的通道，该通道使用了“make”函数动态创建了一个长度为“n”的通道，该通道中所有元素都被初始化为“struct{}”类型的空闲组合。然后，通过一个循环，该通道中被循环生成了“struct{}”类型的元素，每个元素都包含一个空的“struct”类型的空腔，然后将该元素加入到“token”通道的末尾，从而实现了对“token”通道的初始化。

最后，该代码返回了一个名为“Instance”的实例化对象，该对象使用“New”函数可以获取一个具有“n”个通道权限的新实例，并且每个新的实例都会在创建时初始化一个具有相同长度的“token”通道。


```go
package semaphore

// Instance is an implementation of semaphore.
type Instance struct {
	token chan struct{}
}

// New create a new Semaphore with n permits.
func New(n int) *Instance {
	s := &Instance{
		token: make(chan struct{}, n),
	}
	for i := 0; i < n; i++ {
		s.token <- struct{}{}
	}
	return s
}

```

这段代码定义了两个函数，一个接收一个通道通道，另一个发送一个信号到通道。这两个函数作用如下：

1. `Wait()`函数的作用是获取一个通道，允许对通道发送一个请求，并返回通道的`<-chan struct{}>`类型。具体来说，它创建了一个名为`s.token`的变量，并将其设置为通道的`<-chan struct{}>`类型，以便将来的 `<- struct{}` 类型数据可以通过 `<-` 操作符获取。然后，它返回了这个通道，使得调用方可以通过 `<-` 操作符获取通道的内容。
2. `Signal()`函数的作用是释放一个信号到通道。具体来说，它创建了一个名为`s.token`的变量，并将其设置为通道的`<-chan struct{}>`类型，然后发送了一个 `<- struct{}` 类型的数据到通道。这个信号被发送到通道的接收端，允许调用方通过 `<-` 操作符获取通道的内容。


```go
// Wait returns a channel for acquiring a permit.
func (s *Instance) Wait() <-chan struct{} {
	return s.token
}

// Signal releases a permit into the semaphore.
func (s *Instance) Signal() {
	s.token <- struct{}{}
}

```

# `common/stack/bytes.go`

这段代码定义了两个名为`TwoBytes`和`EightBytes`的`[2]byte`和`[8]byte`类型。`TwoBytes`和`EightBytes`类型中都有一个名为`Two`的标量，一个名为`Eight`的标量，以及一个名为`byte`的标量。

根据定义，`TwoBytes`和`EightBytes`类型中的字节数都是固定的，分别为两个字节和八个字节。这两个类型都是在栈上分配的，因此使用了`//go:notinheap`注释来避免在堆上分配内存。

这个代码片段可能是用于在程序中传输数据，将两个字节或八个字节的数据封装到一个`TwoBytes`或`EightBytes`类型的数据中，以便在传输过程中保持数据的正确性。


```go
package stack

// TwoBytes is a [2]byte which is always allocated on stack.
//
//go:notinheap
type TwoBytes [2]byte

// EightBytes is a [8]byte which is always allocated on stack.
//
//go:notinheap
type EightBytes [8]byte

```

# `common/strmatcher/benchmark_test.go`

这段代码是一个用于测试 "strmatcher" 包的 benchmark 代码。具体来说，它创建了一个 "DomainMatcherGroup" 类型的对象，并在其中添加了一系列以 "v2ray.com" 为前缀的域名，这些域名的后面跟着一个数字，代表它们在 v2ray.com 上的 IP 地址。然后，代码使用 "Add" 方法将每个域名加入到了 "DomainMatcherGroup" 对象中。

接下来，代码使用 "BenchmarkDomainMatcherGroup" 函数对 "DomainMatcherGroup" 对象进行测试。具体来说，它创建了一个 "g" 对象，并使用一个循环来遍历所有可能的域名。在循环中，它使用 "strconv.Itoa" 函数将每个域名转换成字符串表示形式，并将其作为参数传递给 "Add" 方法。这样，当 "DomainMatcherGroup" 对象中的所有域名都被添加完后，它将 "g" 对象作为参数传递给 "BenchmarkDomainMatcherGroup" 的 "Run" 函数，该函数将会对 "DomainMatcherGroup" 对象进行测试，并输出测试的结果。

该代码的目的是测试 "DomainMatcherGroup" 对象在处理以 "v2ray.com" 为前缀的域名方面的功能是否正确。它通过创建一个 "DomainMatcherGroup" 对象，将一系列域名添加到该对象中，然后使用 "BenchmarkDomainMatcherGroup" 函数对 "DomainMatcherGroup" 对象进行测试，以验证它是否能够正确匹配以 "v2ray.com" 为前缀的域名。


```go
package strmatcher_test

import (
	"strconv"
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/strmatcher"
)

func BenchmarkDomainMatcherGroup(b *testing.B) {
	g := new(DomainMatcherGroup)

	for i := 1; i <= 1024; i++ {
		g.Add(strconv.Itoa(i)+".v2ray.com", uint32(i))
	}

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = g.Match("0.v2ray.com")
	}
}

```

这两段代码是在测试中使用两个不同的matcher group，一个名为"BenchmarkFullMatcherGroup"，另一个名为"BenchmarkMarchGroup"。

`BenchmarkFullMatcherGroup`的作用是在测试开始时创建一个`FullMatcherGroup`实例，并使用一个循环来添加测试数据。每个测试数据是一个纯数字，以"v2ray.com"为结束符，并在结束符后添加一个计数器。然后使用`g.Match`方法来比较每个测试数据，并记录失败的数量。

`BenchmarkMarchGroup`的作用与`BenchmarkFullMatcherGroup`类似，但创建一个`MatcherGroup`实例，而不是`FullMatcherGroup`实例。然后使用一个循环来创建测试数据，并为每个测试数据创建一个`Domain`对象。最后使用`g.Match`方法来比较测试数据，并记录失败的数量。

这两个函数的作用是测试同一个系统，但是使用不同的matcher group。`BenchmarkFullMatcherGroup`测试更多的数据，而`BenchmarkMarchGroup`测试更少的数据，但更有趣。


```go
func BenchmarkFullMatcherGroup(b *testing.B) {
	g := new(FullMatcherGroup)

	for i := 1; i <= 1024; i++ {
		g.Add(strconv.Itoa(i)+".v2ray.com", uint32(i))
	}

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = g.Match("0.v2ray.com")
	}
}

func BenchmarkMarchGroup(b *testing.B) {
	g := new(MatcherGroup)
	for i := 1; i <= 1024; i++ {
		m, err := Domain.New(strconv.Itoa(i) + ".v2ray.com")
		common.Must(err)
		g.Add(m)
	}

	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = g.Match("0.v2ray.com")
	}
}

```

# `common/strmatcher/domain_matcher.go`

这段代码定义了一个名为 "strmatcher" 的 package，其中包含了一个名为 "breakDomain" 的函数和一个名为 "node" 的结构体。

"breakDomain" 函数接收一个字符串参数 "domain"，并返回该字符串中所有以 "." 作为分隔符的子字符串。这个函数的作用是将一个字符串按照 "." 分割，并将结果返回。

"node" 结构体定义了一个节点，其中包含一个包含 "uint32" 类型元素的 "values" 字段，以及一个包含键为 "string" 的 "sub" 字段。这个结构体的作用是在 "breakDomain" 函数中处理返回的子字符串，以便在需要时动态扩展或收缩。

最后，定义了一个名为 "DomainMatcherGroup" 的类型，该类型包含一个 "IndexMatcher" 类型的方法 "breakDomain"，以及一个名为 "isVisible" 的字段，用于指示该结构体是否为 "breakDomain" 函数的可选参数。


```go
package strmatcher

import "strings"

func breakDomain(domain string) []string {
	return strings.Split(domain, ".")
}

type node struct {
	values []uint32
	sub    map[string]*node
}

// DomainMatcherGroup is a IndexMatcher for a large set of Domain matchers.
// Visible for testing only.
```

该代码定义了一个名为DomainMatcherGroup的类型，它表示一个树形结构，用于存储多个不同的域的匹配器。

具体来说，这个结构体包含一个根节点，每个节点都是一个子节点和一个值。它的根节点可以包含多个子节点，每个子节点都是一个单独的节点，它包含一个域名和相应的值。

Add函数接收一个域名和一个值，首先检查根节点是否为空，如果是，就创建一个新的根节点。然后，从当前节点开始遍历子节点，将每个子节点的域名和值存储到一个哈希表中。接着，从子节点继续遍历，将当前节点的值和当前节点的子节点存储到当前节点中。最后，将当前节点的值和值添加到当前节点中。

BreakDomain函数将传入的域名拆分成多个部分，并返回每个部分单独的域名。

整个结构性树形结构可以用来存储多个不同的域，通过Add函数可以组合使用这些不同的域进行匹配。


```go
type DomainMatcherGroup struct {
	root *node
}

func (g *DomainMatcherGroup) Add(domain string, value uint32) {
	if g.root == nil {
		g.root = new(node)
	}

	current := g.root
	parts := breakDomain(domain)
	for i := len(parts) - 1; i >= 0; i-- {
		part := parts[i]
		if current.sub == nil {
			current.sub = make(map[string]*node)
		}
		next := current.sub[part]
		if next == nil {
			next = new(node)
			current.sub[part] = next
		}
		current = next
	}

	current.values = append(current.values, value)
}

```

该函数`addMatcher`接受一个`DomainMatcherGroup`类型的参数`g`和一个`uint32`类型的参数`value`。它的作用是向给定的`DomainMatcherGroup`中添加一个新的`Matcher`。

函数`Match`的接收者是一个`domain`字符串，它用于查找匹配的领域。如果`domain`为空字符串，函数返回一个空切片。

函数内部首先检查`Domain`是否为空字符串。如果是，函数直接返回一个空切片。如果不是，函数将递归地遍历`Domain`并查找与给定领域匹配的第一个元素，并将该元素添加到结果列表中。如果找不到匹配，函数将返回一个空列表。


```go
func (g *DomainMatcherGroup) addMatcher(m domainMatcher, value uint32) {
	g.Add(string(m), value)
}

func (g *DomainMatcherGroup) Match(domain string) []uint32 {
	if domain == "" {
		return nil
	}

	current := g.root
	if current == nil {
		return nil
	}

	nextPart := func(idx int) int {
		for i := idx - 1; i >= 0; i-- {
			if domain[i] == '.' {
				return i
			}
		}
		return -1
	}

	matches := [][]uint32{}
	idx := len(domain)
	for {
		if idx == -1 || current.sub == nil {
			break
		}

		nidx := nextPart(idx)
		part := domain[nidx+1 : idx]
		next := current.sub[part]
		if next == nil {
			break
		}
		current = next
		idx = nidx
		if len(current.values) > 0 {
			matches = append(matches, current.values)
		}
	}
	switch len(matches) {
	case 0:
		return nil
	case 1:
		return matches[0]
	default:
		result := []uint32{}
		for idx := range matches {
			// Insert reversely, the subdomain that matches further ranks higher
			result = append(result, matches[len(matches)-1-idx]...)
		}
		return result
	}
}

```

# `common/strmatcher/domain_matcher_test.go`

This looks like a group of tests that are using the Google DNS API to query for DNS records of various domains. The tests are using the `golang.org/x/net/ctx/ctx.GRPC` package to establish a connection to the DNS server and the `google.golang.org/v1` package to perform the DNS queries.

The `testCases` slice contains a series of test cases, each with a different domain name and the corresponding expected result. For example, one test case might be asking to query for the DNS record of the domain `x.v2ray.com` and expect the query to return the `1` IP address, while another test case might be asking to query for the same domain and expect no response.

The `g.Add` call is adding the DNS records to the Google DNS API client, with the key-value pairs of the domains and the corresponding score (in this case, the score is the number of bit cells set to 1).

It looks like the tests are using the `ctx.GRPC` package to establish a connection to the DNS server and the `google.golang.org/v1` package to perform the DNS queries, just like a standard Go test framework.


```go
package strmatcher_test

import (
	"reflect"
	"testing"

	. "v2ray.com/core/common/strmatcher"
)

func TestDomainMatcherGroup(t *testing.T) {
	g := new(DomainMatcherGroup)
	g.Add("v2ray.com", 1)
	g.Add("google.com", 2)
	g.Add("x.a.com", 3)
	g.Add("a.b.com", 4)
	g.Add("c.a.b.com", 5)
	g.Add("x.y.com", 4)
	g.Add("x.y.com", 6)

	testCases := []struct {
		Domain string
		Result []uint32
	}{
		{
			Domain: "x.v2ray.com",
			Result: []uint32{1},
		},
		{
			Domain: "y.com",
			Result: nil,
		},
		{
			Domain: "a.b.com",
			Result: []uint32{4},
		},
		{ // Matches [c.a.b.com, a.b.com]
			Domain: "c.a.b.com",
			Result: []uint32{5, 4},
		},
		{
			Domain: "c.a..b.com",
			Result: nil,
		},
		{
			Domain: ".com",
			Result: nil,
		},
		{
			Domain: "com",
			Result: nil,
		},
		{
			Domain: "",
			Result: nil,
		},
		{
			Domain: "x.y.com",
			Result: []uint32{4, 6},
		},
	}

	for _, testCase := range testCases {
		r := g.Match(testCase.Domain)
		if !reflect.DeepEqual(r, testCase.Result) {
			t.Error("Failed to match domain: ", testCase.Domain, ", expect ", testCase.Result, ", but got ", r)
		}
	}
}

```

该测试函数的作用是测试一个名为`DomainMatcherGroup`的函数是否符合预期。该函数可能会接受一个`DomainMatcherGroup`对象作为参数，然后使用该函数的`Match`方法测试该参数是否为空字符串。

如果`Match`方法返回的结果为空字符串，则该函数将引发一个`testing.T`类型的异常，并传递给异常处理程序进行进一步处理。如果返回的结果不是空字符串，则不会引发任何异常，该函数也不会输出任何错误。


```go
func TestEmptyDomainMatcherGroup(t *testing.T) {
	g := new(DomainMatcherGroup)
	r := g.Match("v2ray.com")
	if len(r) != 0 {
		t.Error("Expect [], but ", r)
	}
}

```

# `common/strmatcher/full_matcher.go`

这段代码定义了一个名为 `FullMatcherGroup` 的结构体，用于表示一个正则表达式模式中多个验证器的集合。其中，每个验证器对应一个验证规则，由一个匹配器（matcher）和一个匹配的任意长度的 `uint32` 组成。

`Add` 函数接收一个 `domain` 字段和一个 `value`，将其添加到 `g.matchers`  map 中。如果 `g.matchers` 已经是空的，则直接添加。

`addMatcher` 函数接收一个 `FullMatcher` 结构体和一个 `value`，将其添加到 `g.matchers`  map 中。其中，`FullMatcher` 是一个 `strmatcher` 包中的验证器，验证规则的类型是 `fmt.NET.regex.Matcher` 类型。


```go
package strmatcher

type FullMatcherGroup struct {
	matchers map[string][]uint32
}

func (g *FullMatcherGroup) Add(domain string, value uint32) {
	if g.matchers == nil {
		g.matchers = make(map[string][]uint32)
	}

	g.matchers[domain] = append(g.matchers[domain], value)
}

func (g *FullMatcherGroup) addMatcher(m fullMatcher, value uint32) {
	g.Add(string(m), value)
}

```

该函数接收一个 FullMatcherGroup 类型的参数 g，并返回一个 Uint32 类型的数组，其中包含与给定字符串匹配的阈值。

函数首先检查 g.matchers 是否为空，如果是，则返回 nil。否则，函数使用 g.matchers 指向的代理(代理在代码中未定义)来获取与给定字符串匹配的第一个阈值，并将它存储在 g.matchers 上。

函数返回的是 g.matchers 上存储的第一个阈值，而不是整个代理中的所有阈值。如果需要获取整个代理中的所有阈值，可以传中华民族串 g 并使用 IterateThrough 函数遍历整个代理，如下所示：

// 遍历整个代理并打印每个阈值
for _, threshold in g.iterateThrough {
	fmt.Printf("%d\n", threshold)
}

在该示例中，使用了 IterateThrough 函数来遍历代理并打印每个阈值。


```go
func (g *FullMatcherGroup) Match(str string) []uint32 {
	if g.matchers == nil {
		return nil
	}

	return g.matchers[str]
}

```

# `common/strmatcher/full_matcher_test.go`

这段代码是一个用于测试 "strmatcher" 包的函数，它演示了如何使用反射和测试 "v2ray.com"、"google.com" 和 "x.a.com" 三个网站的域名的功能。

首先，这个测试函数创建了一个名为 "FullMatcherGroup" 的对象，它包含一个添加网站的方法，该方法接受一个域名字符串和一个域数参数。然后，它将添加以下四个网站：

1. "v2ray.com"（1个权重）
2. "google.com"（2个权重）
3. "x.a.com"（3个权重）
4. "x.y.com"（4个权重）

接下来，它创建了一个测试覆盖器（testCases）数组，其中包含以下包含多个测试用例的接口：

1. 测试用例：{
		Domain: "v2ray.com",
		Result: []uint32{1},
	}
2. 测试用例：{
		Domain: "y.com",
		Result: nil,
	}
3. 测试用例：{
		Domain: "x.y.com",
		Result: []uint32{4, 6},
	}

最后，它循环遍历 testCases 数组中的每个测试用例，并使用 FullMatcherGroup 对象中的 Match 方法来查找与测试用例中的域名和预期结果是否相等。如果结果不匹配，则在测试失败时打印错误消息。

注意：这段代码缺少必要的包和导入，因此无法正常工作。


```go
package strmatcher_test

import (
	"reflect"
	"testing"

	. "v2ray.com/core/common/strmatcher"
)

func TestFullMatcherGroup(t *testing.T) {
	g := new(FullMatcherGroup)
	g.Add("v2ray.com", 1)
	g.Add("google.com", 2)
	g.Add("x.a.com", 3)
	g.Add("x.y.com", 4)
	g.Add("x.y.com", 6)

	testCases := []struct {
		Domain string
		Result []uint32
	}{
		{
			Domain: "v2ray.com",
			Result: []uint32{1},
		},
		{
			Domain: "y.com",
			Result: nil,
		},
		{
			Domain: "x.y.com",
			Result: []uint32{4, 6},
		},
	}

	for _, testCase := range testCases {
		r := g.Match(testCase.Domain)
		if !reflect.DeepEqual(r, testCase.Result) {
			t.Error("Failed to match domain: ", testCase.Domain, ", expect ", testCase.Result, ", but got ", r)
		}
	}
}

```

这段代码定义了一个名为 `TestEmptyFullMatcherGroup` 的测试函数，属于 `testing.T` 类型的函数。

函数的作用是测试 `FullMatcherGroup` 是否为空或是否可以匹配到 "v2ray.com" 网站。为此，函数创建了一个 `FullMatcherGroup` 对象，并使用 `Match` 方法对其进行测试。如果 `Match` 方法返回的匹配结果为空，则函数将输出 "Expect [], but <https://v2ray.com>"。如果返回的结果不等于空，则不输出任何内容。


```go
func TestEmptyFullMatcherGroup(t *testing.T) {
	g := new(FullMatcherGroup)
	r := g.Match("v2ray.com")
	if len(r) != 0 {
		t.Error("Expect [], but ", r)
	}
}

```

# `common/strmatcher/matchers.go`

这段代码定义了一个名为“strmatcher”的包，其中包含了一个名为“fullMatcher”的类型和一个名为“Match”的函数和一个名为“String”的函数。

“fullMatcher”类型代表了一个可以匹配整个字符串的包字符串。

“Match”函数接受一个字符串参数“s”，并返回一个布尔值“true”或“false”。如果字符串“m”和字符串“s”相等，则返回“true”。

“String”函数返回一个字符串，它是“full”和字符串“m”的组合。

总之，这段代码定义了一个可以匹配整个字符串的包，通过一个名为“fullMatcher”的类型和一个名为“Match”的函数和一个名为“String”的函数来匹配字符串。


```go
package strmatcher

import (
	"regexp"
	"strings"
)

type fullMatcher string

func (m fullMatcher) Match(s string) bool {
	return string(m) == s
}

func (m fullMatcher) String() string {
	return "full:" + string(m)
}

```

这段代码定义了两个类型的Matcher:substrMatcher和domainMatcher，它们都有一个Match函数和一个String函数。

substrMatcher是一个字符串类型的Matcher，它使用type substrMatcher string 来指定Matcher的类型为string。它的Match函数接收一个字符串参数s和一个键钥词domainMatcher，并返回true如果s中包含domainMatcher，否则返回false。它的String函数返回一个字符串格式化后的带有domainMatcher的键词，例如"keyword:mydomain"。

domainMatcher是一个字符串类型的Matcher，它使用type domainMatcher string 来指定Matcher的类型为string。它的Match函数接收一个字符串参数s和一个domainMatcher，并返回true if s中包含domainMatcher，否则返回false。它的String函数返回一个字符串格式化后的带有domainMatcher的键词，例如"mydomain:keyword"。


```go
type substrMatcher string

func (m substrMatcher) Match(s string) bool {
	return strings.Contains(s, string(m))
}

func (m substrMatcher) String() string {
	return "keyword:" + string(m)
}

type domainMatcher string

func (m domainMatcher) Match(s string) bool {
	pattern := string(m)
	if !strings.HasSuffix(s, pattern) {
		return false
	}
	return len(s) == len(pattern) || s[len(s)-len(pattern)-1] == '.'
}

```

这段代码定义了一个名为 `regexMatcher` 的类型，该类型包含一个名为 `pattern` 的整数类型变量和一个名为 `Match` 的布尔类型变量，还有一个名为 `String` 的字符类型变量。

函数 `func` 返回一个字符串类型，该函数将 `m` 域的值与字符串拼接，然后将结果返回。

函数 `String` 返回一个字符串类型，该函数将 `m` 的模式与字符串拼接，然后将结果返回。

函数 `func` 的参数是一个名为 `m` 的整数类型变量和一个名为 `domainMatcher` 的函数指针类型，该函数指针类型接收一个字符串参数。

函数 `func` 的参数接收字符串 `s`，返回值是字符串 `m` 在 `s` 中的匹配结果，返回值是布尔类型 `true` 或 `false`。

函数 `func` 的参数接收整数 `m`，返回字符串 `regexp:` 和 `pattern:`，该函数指针类型包含 `m` 的模式和匹配的字符串。


```go
func (m domainMatcher) String() string {
	return "domain:" + string(m)
}

type regexMatcher struct {
	pattern *regexp.Regexp
}

func (m *regexMatcher) Match(s string) bool {
	return m.pattern.MatchString(s)
}

func (m *regexMatcher) String() string {
	return "regexp:" + m.pattern.String()
}

```

# `common/strmatcher/matchers_test.go`

This appears to be a testing function for v2ray.com. The function takes an input string and an output boolean, and it uses regular expressions to check if the input string matches the expected pattern. It returns true if the input string matches the pattern and false if it does not match.

The testing is done using several test cases. Each test case has a different type of pattern and input, and it is used to verify that the regular expression matches the expected pattern for different inputs.

For example, one test case has an input string "v2ray.com" and an output boolean true, which means the input string should match the expected pattern. Another test case has the same input string but with an output boolean false, which means the input string should not match the expected pattern.

The last test case has a more complex pattern, which is a regular expression that matches "v2rayxcom". It is used to verify that the regular expression matches the expected pattern for input strings that include the "x" suffix.


```go
package strmatcher_test

import (
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/strmatcher"
)

func TestMatcher(t *testing.T) {
	cases := []struct {
		pattern string
		mType   Type
		input   string
		output  bool
	}{
		{
			pattern: "v2ray.com",
			mType:   Domain,
			input:   "www.v2ray.com",
			output:  true,
		},
		{
			pattern: "v2ray.com",
			mType:   Domain,
			input:   "v2ray.com",
			output:  true,
		},
		{
			pattern: "v2ray.com",
			mType:   Domain,
			input:   "www.v3ray.com",
			output:  false,
		},
		{
			pattern: "v2ray.com",
			mType:   Domain,
			input:   "2ray.com",
			output:  false,
		},
		{
			pattern: "v2ray.com",
			mType:   Domain,
			input:   "xv2ray.com",
			output:  false,
		},
		{
			pattern: "v2ray.com",
			mType:   Full,
			input:   "v2ray.com",
			output:  true,
		},
		{
			pattern: "v2ray.com",
			mType:   Full,
			input:   "xv2ray.com",
			output:  false,
		},
		{
			pattern: "v2ray.com",
			mType:   Regex,
			input:   "v2rayxcom",
			output:  true,
		},
	}
	for _, test := range cases {
		matcher, err := test.mType.New(test.pattern)
		common.Must(err)
		if m := matcher.Match(test.input); m != test.output {
			t.Error("unexpected output: ", m, " for test case ", test)
		}
	}
}

```

# `common/strmatcher/strmatcher.go`

这段代码定义了一个名为"strmatcher"的包，其中定义了一个名为"Matcher"的接口，以及一个名为"Type"的类型声明。

"Matcher"接口定义了一个 matcher，用于检查一个字符串是否匹配给定的模式。该接口包含一个名为"Match"的方法，该方法返回一个布尔值，表示给定字符串是否匹配模式。"Match"方法还包含一个名为"String"的静态方法，返回匹配到的字符串的哈希值。

"Type"类型声明定义了一个名为"byte"的类型，该类型可能用于表示字节数组或单个字节。


```go
package strmatcher

import (
	"regexp"
)

// Matcher is the interface to determine a string matches a pattern.
type Matcher interface {
	// Match returns true if the given string matches a predefined pattern.
	Match(string) bool
	String() string
}

// Type is the type of the matcher.
type Type byte

```

这段代码定义了一个名为 `Matcher` 的接口，以及三种不同类型的 `Matcher`: `Full`, `Substr`, `Domain`, 和 `Regex`。

`Full` 类型表示输入字符串必须完全等于给定的模式，例如，`"abcdefg"` 必须等于 `"abcdefg"`。

`Substr` 类型表示输入字符串必须包含给定的模式作为子字符串，例如，`"abcdefg"`，但 `"abcdefgabc"` 不符合要求。

`Domain` 类型表示输入字符串必须是一个域名或 itself，即输入字符串必须是一个 URL 的域名。

`Regex` 类型表示输入字符串必须匹配给定的正则表达式模式，例如，`"abcdefg"` 必须匹配 `"abcdefg"`。

该函数接受一个模式字符串 `pattern` 并返回一个 `Matcher` 对象，如果 `pattern` 存在错误则返回一个错误对象。如果 `Matcher` 类型 `Full`, `Substr`, `Domain`, 或 `Regex` 接受 `pattern` 并返回 `Matcher` 对象，否则返回 `nil`。


```go
const (
	// Full is the type of matcher that the input string must exactly equal to the pattern.
	Full Type = iota
	// Substr is the type of matcher that the input string must contain the pattern as a sub-string.
	Substr
	// Domain is the type of matcher that the input string must be a sub-domain or itself of the pattern.
	Domain
	// Regex is the type of matcher that the input string must matches the regular-expression pattern.
	Regex
)

// New creates a new Matcher based on the given pattern.
func (t Type) New(pattern string) (Matcher, error) {
	switch t {
	case Full:
		return fullMatcher(pattern), nil
	case Substr:
		return substrMatcher(pattern), nil
	case Domain:
		return domainMatcher(pattern), nil
	case Regex:
		r, err := regexp.Compile(pattern)
		if err != nil {
			return nil, err
		}
		return &regexMatcher{
			pattern: r,
		}, nil
	default:
		panic("Unknown type")
	}
}

```

这段代码定义了一个名为IndexMatcher的接口，表示一组与给定输入匹配的matcher。其中，MatcherGroup是IndexMatcher接口的实现，提供了对一组matcher的计数、完整的matcher和域的matcher的访问。

具体来说，这段代码实现了一个MatcherGroup实例，其中包含一个计数器、一个完整的matcher和一个域的matcher。此外，还定义了一个名为OtherMatchers的切片，用于存储除了给定输入匹配的其他matcher。

这段代码的目的是提供一个简单的方法来匹配字符串输入，并返回一个与输入匹配的matcher的索引。可以用于许多不同的用例，如需要根据用户输入查找输入内容所属的域名，或者在需要将输入内容与已知的参考文献进行匹配时使用。


```go
// IndexMatcher is the interface for matching with a group of matchers.
type IndexMatcher interface {
	// Match returns the index of a matcher that matches the input. It returns empty array if no such matcher exists.
	Match(input string) []uint32
}

type matcherEntry struct {
	m  Matcher
	id uint32
}

// MatcherGroup is an implementation of IndexMatcher.
// Empty initialization works.
type MatcherGroup struct {
	count         uint32
	fullMatcher   FullMatcherGroup
	domainMatcher DomainMatcherGroup
	otherMatchers []matcherEntry
}

```

这段代码定义了一个名为MatcherGroup的表示器类型，以及一个名为Add的函数。该函数接收一个Matcher对象作为参数，并将其加入到MatcherGroup中。函数的实现包括以下几个步骤：

1. 增加MatcherGroup的计数器。
2. 如果接收的Matcher对象是一个fullMatcher类型，则将该Matcher对象的计数器添加到其addMatcher函数中。
3. 如果接收的Matcher对象是一个domainMatcher类型，则将该Matcher对象的计数器添加到其addMatcher函数中。
4. 如果接收的Matcher对象是一个otherMatchers类型，则将该Matcher对象的ID添加到g.otherMatchers的数组中，并创建一个包含该Matcher对象的entry的addMatcher函数中。
5. 返回MatcherGroup的计数器值。

由于该函数使用了switch语句，它会根据Matcher对象的类型来执行不同的操作。如果Matcher对象是一个fullMatcher或domainMatcher，则该函数会将该Matcher对象的计数器添加到其addMatcher函数中。如果Matcher对象是一个otherMatchers类型，则该函数会将该Matcher对象的ID添加到g.otherMatchers的数组中，并创建一个包含该Matcher对象的entry的addMatcher函数中。

因此，该函数的作用是将一个Matcher对象加入到MatcherGroup中，并根据Matcher对象的类型来执行不同的操作。


```go
// Add adds a new Matcher into the MatcherGroup, and returns its index. The index will never be 0.
func (g *MatcherGroup) Add(m Matcher) uint32 {
	g.count++
	c := g.count

	switch tm := m.(type) {
	case fullMatcher:
		g.fullMatcher.addMatcher(tm, c)
	case domainMatcher:
		g.domainMatcher.addMatcher(tm, c)
	default:
		g.otherMatchers = append(g.otherMatchers, matcherEntry{
			m:  m,
			id: c,
		})
	}

	return c
}

```

该代码定义了一个名为Match的函数，它实现了IndexMatcher.Match。

该函数接收一个正则表达式参数pattern，并返回一个长度为pattern中不同匹配项的长度的uint32切片。

函数内部首先调用IndexMatcher.Match函数来获取正则表达式中所有匹配项的ID。然后，它使用for循环遍历该MatcherGroup中的所有其他Matchers，如果其中一个Matcher的match函数返回true并且ID与当前正在遍历的Matcher的ID不同时，将ID添加到结果切片中。

最后，函数返回MatcherGroup中所有Matchers的数量。


```go
// Match implements IndexMatcher.Match.
func (g *MatcherGroup) Match(pattern string) []uint32 {
	result := []uint32{}
	result = append(result, g.fullMatcher.Match(pattern)...)
	result = append(result, g.domainMatcher.Match(pattern)...)
	for _, e := range g.otherMatchers {
		if e.m.Match(pattern) {
			result = append(result, e.id)
		}
	}
	return result
}

// Size returns the number of matchers in the MatcherGroup.
func (g *MatcherGroup) Size() uint32 {
	return g.count
}

```

# `common/strmatcher/strmatcher_test.go`

This appears to be a unit test for a JavaScript function that uses the Google Charts API to load a chart for a given domain.

The test is using the MatcherGroup library to compare the output of the Google Charts API to the input provided by the test case.

The MatcherGroup library is used to group together the matchers (in this case, the functions that compare the input to the output) and to apply the grouping to the test cases.

Each test case is given a different input and is expected to produce the same output as the test case. The test case is then compared to the MatcherGroup using the Match() method.

If the output of the MatcherGroup does not match the output of the Google Charts API for the test case, the test will report an error and indicate that the test failed.


```go
package strmatcher_test

import (
	"reflect"
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/strmatcher"
)

// See https://github.com/v2fly/v2ray-core/issues/92#issuecomment-673238489
func TestMatcherGroup(t *testing.T) {
	rules := []struct {
		Type   Type
		Domain string
	}{
		{
			Type:   Regex,
			Domain: "apis\\.us$",
		},
		{
			Type:   Substr,
			Domain: "apis",
		},
		{
			Type:   Domain,
			Domain: "googleapis.com",
		},
		{
			Type:   Domain,
			Domain: "com",
		},
		{
			Type:   Full,
			Domain: "www.baidu.com",
		},
		{
			Type:   Substr,
			Domain: "apis",
		},
		{
			Type:   Domain,
			Domain: "googleapis.com",
		},
		{
			Type:   Full,
			Domain: "fonts.googleapis.com",
		},
		{
			Type:   Full,
			Domain: "www.baidu.com",
		},
		{
			Type:   Domain,
			Domain: "example.com",
		},
	}
	cases := []struct {
		Input  string
		Output []uint32
	}{
		{
			Input:  "www.baidu.com",
			Output: []uint32{5, 9, 4},
		},
		{
			Input:  "fonts.googleapis.com",
			Output: []uint32{8, 3, 7, 4, 2, 6},
		},
		{
			Input:  "example.googleapis.com",
			Output: []uint32{3, 7, 4, 2, 6},
		},
		{
			Input:  "testapis.us",
			Output: []uint32{1, 2, 6},
		},
		{
			Input:  "example.com",
			Output: []uint32{10, 4},
		},
	}
	matcherGroup := &MatcherGroup{}
	for _, rule := range rules {
		matcher, err := rule.Type.New(rule.Domain)
		common.Must(err)
		matcherGroup.Add(matcher)
	}
	for _, test := range cases {
		if m := matcherGroup.Match(test.Input); !reflect.DeepEqual(m, test.Output) {
			t.Error("unexpected output: ", m, " for test case ", test)
		}
	}
}

```

# `common/task/common.go`

这段代码定义了一个名为 "task" 的包，其中包含一个名为 "Close" 的函数。该函数接受一个接口 "v" 作为参数，并返回一个名为 "func" 的函数，该函数实现了一个与 "Close" 名称相同的函数。

具体来说，这个 "Close" 函数接受任何类型的 "v" 作为参数，并返回一个名为 "func" 的函数，该函数实现了一个与 "Close" 名称相同的函数。由于 "Close" 函数的返回类型是 "func() error"，因此当调用 "Close" 函数时，会返回一个与 "Close" 函数返回类型相同的错误。

由于 "Close" 函数的实现非常简单，它只是通过调用 "common.Close" 函数来关闭传入的 "v" 对象。因此，这个函数的使用场合可能会比较有限，仅限于那些需要一个简单的 "Close" 函数来关闭传入对象的场景。


```go
package task

import "v2ray.com/core/common"

// Close returns a func() that closes v.
func Close(v interface{}) func() error {
	return func() error {
		return common.Close(v)
	}
}

```

# `common/task/periodic.go`

这段代码定义了一个名为"task"的包，其中包含一个名为"Periodic"的类型，这个类型表示一个周期性任务。

"Periodic"结构体包含一个"Interval"字段，表示任务执行的周期；以及一个名为"Execute"的函数，表示任务执行的代码。

另外，代码中还定义了一个名为"Periodic"的函数，该函数接受一个"Periodic"类型的参数，表示一个具体的周期性任务。

最后，代码中还定义了一个名为"task"的常量，用于在整个程序中访问"Periodic"类型的实例。


```go
package task

import (
	"sync"
	"time"
)

// Periodic is a task that runs periodically.
type Periodic struct {
	// Interval of the task being run
	Interval time.Duration
	// Execute is the task function
	Execute func() error

	access  sync.Mutex
	timer   *time.Timer
	running bool
}

```

这段代码定义了两个函数，一个是`hasClosed()`，另一个是`checkedExecute()`。这两个函数都与一个名为`Periodic`的接口有关。

函数`hasClosed()`的作用是判断一个`Periodic`对象是否已经关闭。函数首先会尝试获取对象的访问权限，然后挂起当前线程。接着，它会判断对象是否正在运行，如果不是，则函数返回`false`。如果正在运行，则函数会执行`execute()`函数并检查是否成功。如果执行失败，函数会再次尝试获取访问权限，然后释放当前线程。如果仍然无法关闭对象，函数将返回`nil`。

函数`checkedExecute()`的作用是在确保对象处于关闭状态后再执行`Execute()`函数。如果对象已经关闭，函数直接返回。否则，函数会尝试获取当前线程的访问权限，然后设置对象为运行中状态。接着，它会设置一个定时器，在一定时间后再次调用`checkedExecute()`函数。

在这个程序中，定时器设置了一个时间间隔，当`Periodic`对象在这个时间间隔内没有调用`checkedExecute()`函数时，它将销毁并释放当前线程。


```go
func (t *Periodic) hasClosed() bool {
	t.access.Lock()
	defer t.access.Unlock()

	return !t.running
}

func (t *Periodic) checkedExecute() error {
	if t.hasClosed() {
		return nil
	}

	if err := t.Execute(); err != nil {
		t.access.Lock()
		t.running = false
		t.access.Unlock()
		return err
	}

	t.access.Lock()
	defer t.access.Unlock()

	if !t.running {
		return nil
	}

	t.timer = time.AfterFunc(t.Interval, func() {
		t.checkedExecute() // nolint: errcheck
	})

	return nil
}

```

这段代码定义了一个名为`Periodic`的辅助类型，该类型实现了` common.Runnable`接口。这个辅助类型有一个`Start`方法，它的作用是在`Periodic`类型的实例启动时执行一些设置，并确保`Periodic`实例在正确的时机上下文。

具体来说，这段代码的作用如下：

1. 首先，代码创建了一个名为`t`的`Periodic`实例。
2. 如果`t`实例已经在运行中，那么代码会尝试解除`t`实例的运行状态，并返回一个`nil`。
3. 如果`t`实例还没有运行中，那么代码会将`t`实例设置为`true`，并尝试调用`Periodic`实例的`checkedExecute`方法。如果这个方法返回一个`error`，那么代码会将`t`实例设置为`false`，并再次尝试调用`Periodic`实例的`checkedExecute`方法。
4. 如果`Periodic`实例的运行状态不是`true`，那么代码会尝试解除`t`实例的运行状态，并返回一个`error`。
5. 最后，如果`Periodic`实例的运行状态为`true`，那么代码会返回一个`nil`，表示`Periodic`实例可以正常启动。

综上所述，这段代码的作用是定义了一个`Periodic`辅助类型，用于在`Runnable`实例启动时执行一些设置，并确保`Runnable`实例在正确的时机上下文。


```go
// Start implements common.Runnable.
func (t *Periodic) Start() error {
	t.access.Lock()
	if t.running {
		t.access.Unlock()
		return nil
	}
	t.running = true
	t.access.Unlock()

	if err := t.checkedExecute(); err != nil {
		t.access.Lock()
		t.running = false
		t.access.Unlock()
		return err
	}

	return nil
}

```

这段代码定义了一个名为`Periodic`的上下文类型，该类型代表一个周期性任务，并且实现了`common.Closable`接口。

在这个具体的实现中，当一个`Periodic`对象被创建时，会执行一些操作来关闭它所代表的周期性任务。具体来说，代码会执行以下操作：

1. 获取`Periodic`对象的引用，并加锁保护，以确保同一时刻只有一个`Periodic`对象可以访问它所代表的周期性任务。

2. 设置`Periodic`对象的`running`字段为`false`，表示当前这个`Periodic`对象不在运行。

3. 如果`Periodic`对象还有一个`timer`指针，会执行以下操作：

  1. 停止`timer`所代表的周期性任务的运行。

  2. 清除`timer`所代表的周期性任务的时间限制。

  3. 将`timer`设置为`nil`，表示这个`Periodic`对象不再需要使用定时任务。

关闭`Periodic`对象的方式可以在任何时候调用`Close()`方法，它会返回一个`error`类型，表示关闭是否成功。


```go
// Close implements common.Closable.
func (t *Periodic) Close() error {
	t.access.Lock()
	defer t.access.Unlock()

	t.running = false
	if t.timer != nil {
		t.timer.Stop()
		t.timer = nil
	}

	return nil
}

```

# `common/task/periodic_test.go`

该代码的作用是测试一个名为 "Periodic" 的函数，它用于执行定期的、心跳的、非阻塞的任务。

具体来说，该代码创建了一个名为 "task" 的实例，该实例实现了Periodic接口。它包含了一个定时器，每隔2秒钟执行一次 "Execute" 方法，执行后会将 "value" 加1，然后返回 nil。

接着，代码创建了一个 "Periodic" 类型的实例，并调用了 "Start" 方法。然后，代码使用 "time.Sleep" 方法让应用程序休眠了5秒钟，然后再次执行 "Start" 方法。

在每次执行 "Execute" 方法后，代码使用 "common.Must" 函数确保 "task" 对象已经关闭并且已经停止心跳。接着，代码再次让应用程序休眠，并执行下一次 "Execute" 方法。

在 "Periodic" 执行了3次 "Execute" 方法后，代码打印出 "value" 的值，并使用 "t.Fatal" 函数打印错误消息，如果 "value" 的值不是3，则会引发该函数的 "t.Fatal" 函数并导致失败。在 "Periodic" 执行了4次 "Execute" 方法后，代码打印出 "value" 的值，并使用 "t.Fatal" 函数打印错误消息，如果 "value" 的值不是3，则会引发该函数的 "t.Fatal" 函数并导致失败。最后，在 "Periodic" 执行了5次 "Execute" 方法后，代码打印出 "value" 的值，并使用 "t.Fatal" 函数打印错误消息，如果 "value" 的值不是5，则会引发该函数的 "t.Fatal" 函数并导致失败。


```go
package task_test

import (
	"testing"
	"time"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/task"
)

func TestPeriodicTaskStop(t *testing.T) {
	value := 0
	task := &Periodic{
		Interval: time.Second * 2,
		Execute: func() error {
			value++
			return nil
		},
	}
	common.Must(task.Start())
	time.Sleep(time.Second * 5)
	common.Must(task.Close())
	if value != 3 {
		t.Fatal("expected 3, but got ", value)
	}
	time.Sleep(time.Second * 4)
	if value != 3 {
		t.Fatal("expected 3, but got ", value)
	}
	common.Must(task.Start())
	time.Sleep(time.Second * 3)
	if value != 5 {
		t.Fatal("Expected 5, but ", value)
	}
	common.Must(task.Close())
}

```

# `common/task/task.go`

这段代码定义了一个名为 "task" 的包，其中包含了一些函数和类型声明。

主要作用是定义了一个名为 "OnSuccess" 的函数，该函数接收两个参数：一个 "f" 函数和一个 "g" 函数。这两个函数分别执行 "f" 和 "g" 中的代码，并在 "f" 函数返回错误时执行 "g" 函数，以处理可能出现的错误。

具体来说，代码中定义了一个名为 "OnSuccess" 的函数，它接收两个整数类型的参数 "f" 和 "g"。这两个函数都返回一个整数类型的函数返回值，一个代表成功，一个代表失败。函数内部首先调用 "f"，如果出现错误，则执行回调函数 "g"，否则直接返回。这样，当 "f" 函数返回成功后，仍然有可能出现错误，调用 "g" 函数进行错误处理；而当 "f" 函数返回失败，则先调用 "g" 函数处理错误，再调用 "f" 函数继续执行。

另外，代码中还定义了一个名为 "semaphore" 的类型，它是一个名为 "semaphore" 的类型，但具体含义并未定义。


```go
package task

import (
	"context"

	"v2ray.com/core/common/signal/semaphore"
)

// OnSuccess executes g() after f() returns nil.
func OnSuccess(f func() error, g func() error) func() error {
	return func() error {
		if err := f(); err != nil {
			return err
		}
		return g()
	}
}

```

这段代码的作用是运行一组任务并返回第一个错误。它使用了 `exec` 命令在并行中运行这些任务。如果所有任务都成功完成，它将返回一个 ` nil` 错误。否则，它将返回一个测试运行时任务返回的错误。

该代码使用了一个互斥锁 `semaphore` 来确保在所有任务完成之前只有一个锁处于可用状态。对于每个任务，它使用 `semaphore.New` 方法创建一个名为 `done` 的通道，该频道允许 `Run` 函数以轴线方式写入 `done` 通道。如果一个任务失败并产生一个错误，它将发送一个信号到互斥锁中，此时 `done` 通道中的所有错误都会被唤醒，然后调用 `<-semaphore.Wait>` 它会等待信号并返回失败的结果。如果所有任务都成功完成，它将返回一个 ` nil` 错误。


```go
// Run executes a list of tasks in parallel, returns the first error encountered or nil if all tasks pass.
func Run(ctx context.Context, tasks ...func() error) error {
	n := len(tasks)
	s := semaphore.New(n)
	done := make(chan error, 1)

	for _, task := range tasks {
		<-s.Wait()
		go func(f func() error) {
			err := f()
			if err == nil {
				s.Signal()
				return
			}

			select {
			case done <- err:
			default:
			}
		}(task)
	}

	for i := 0; i < n; i++ {
		select {
		case err := <-done:
			return err
		case <-ctx.Done():
			return ctx.Err()
		case <-s.Wait():
		}
	}

	return nil
}

```

# `common/task/task_test.go`

这段代码是一个 Go 语言编写的测试包，用于测试任务(task)的实现。具体来说，它包括以下功能：

1. 导入必要的包：`context`、`errors`、`strings`、`testing`、`time`、`github.com/google/go-cmp/cmp`。

2. 定义了一个名为 `task_test` 的包。

3. 导入 `task.Transport` 类型的依赖。

4. 定义了一个名为 `testCircuit` 的函数，它是 `task.Transport` 类型的一个实例，用于测试 `task.Transport` 类型的接口。函数的参数包括一个 `context.Context` 类型的上下文、一个 `task.Transport` 类型的参数 `transport` 和一个 `testing.T` 类型的测试者。

5. 实现了一个名为 `testCircuit` 的函数，它接收一个 `context.Context` 类型的上下文、一个 `task.Transport` 类型的参数 `transport` 和一个 `testing.T` 类型的测试者。函数首先创建一个空的 `task.Transport` 上下文 `ctx` 和一个客户端 `transport`。然后，它使用 `transport.Send` 方法发送一个消息到服务器。最后，它使用 `ctx.Log` 方法记录一条日志，内容为 "test message"。

6. 在 `testCircuit` 的函数中，使用 `ctx.Log` 方法记录了一条日志，内容为 "test message"。这可以用于日誌记录和调试。

7. 编译并运行测试。


```go
package task_test

import (
	"context"
	"errors"
	"strings"
	"testing"
	"time"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	. "v2ray.com/core/common/task"
)

```

这段代码是一个名为 `TestExecuteParallel` 的函数，它使用了 Go 的 `testing` 和 `context` 包，用于测试一个名为 `ExecuteParallel` 的函数。

具体来说，这段代码的作用是模拟一个并行执行测试操作的场景，并发送多个请求来测试结果。

首先，函数内定义了一个名为 `Run` 的函数，它使用了 `context.Background()` 来创建一个新的事件循环，并且使用了 `func() error` 类型的函数来返回一个发生错误的情况。在函数内部，使用了一个时间 `time.Millisecond` 间隔来模拟两次请求之间的时间间隔。然后，使用 `errors.New` 函数来创建一个自定义的错误类型，称为 "test"。

接下来，定义了一个名为 `func() error` 的函数，使用与上面创建的 `Run` 函数相同的时间间隔来模拟两次请求之间的时间间隔。然后，使用 `cmp.Diff` 函数来比较 `err` 和 "test" 之间的差异，如果两个差异为空，则表示函数执行成功。否则，函数会输出 `r` 变量，表示差异，以便于后续检查。

最后，在 `TestExecuteParallel` 函数内部，创建了一个名为 `cmp.Diff` 的函数，用于比较 `err` 和 "test" 之间的差异。如果两个差异为空，则表示函数执行成功。否则，函数会输出 `r` 变量，表示差异，以便于后续检查。


```go
func TestExecuteParallel(t *testing.T) {
	err := Run(context.Background(),
		func() error {
			time.Sleep(time.Millisecond * 200)
			return errors.New("test")
		}, func() error {
			time.Sleep(time.Millisecond * 500)
			return errors.New("test2")
		})

	if r := cmp.Diff(err.Error(), "test"); r != "" {
		t.Error(r)
	}
}

```

这段代码是用于测试一个名为 `TestExecuteParallelContextCancel` 的函数。函数内部创建了一个取消上下文（Context）并返回。

具体来说，这段代码的作用如下：

1. 创建一个取消上下文并返回，使用 `context.WithCancel` 函数实现。
2. 创建一个函数，使用 `Run` 函数实现，该函数包含两个空括号，分别代表开始和结束的时间戳。在开始时间戳和结束时间戳之间，执行一个连续的 2 秒钟的睡眠。然后，使用这两个睡眠时间，创建一个错误对象。
3. 在另一个函数中，使用 `cancel` 函数取消刚刚创建的上下文，并返回一个 nil 值。
4. 调用 `cancel` 函数，并且由于上下文已经取消，不会输出任何错误信息，但是函数会输出一个预期的错误信息，即 "expected error string to contain 'canceled', but actually not: <error message>"。


```go
func TestExecuteParallelContextCancel(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	err := Run(ctx, func() error {
		time.Sleep(time.Millisecond * 2000)
		return errors.New("test")
	}, func() error {
		time.Sleep(time.Millisecond * 5000)
		return errors.New("test2")
	}, func() error {
		cancel()
		return nil
	})

	errStr := err.Error()
	if !strings.Contains(errStr, "canceled") {
		t.Error("expected error string to contain 'canceled', but actually not: ", errStr)
	}
}

```

这两段代码是在测试框架中定义的函数，它们的目的是在测试框架运行时对两个不同的函数执行多次，以验证它们的性能和正确性。

`func BenchmarkExecuteOne(b *testing.B)` 和 `func BenchmarkExecuteTwo(b *testing.B)` 是两个不同的函数，它们的名字和签名略有不同，但它们的参数和实现方式都相同。它们的目的是在测试框架运行时分别调用 `noop` 函数，并将测试框架的输出结果打印出来。

具体来说，这两段代码的作用是模拟测试框架的运行过程。在测试框架启动后，它们会创建两个自称为 `noop` 的函数，并在测试框架的 `BenchmarkExecuteOne` 和 `BenchmarkExecuteTwo` 函数中分别传入这两个函数。这些传入的函数会在测试框架运行时执行多次，每次调用 `noop` 函数时，测试框架会将调用结果输出到控制台。

这两段代码会输出以下结果：

[0] 2 3 4 5 6 7 8 9 10 [1] 2 3 4 5 6 7 8 9 10 [0] 2 3 4 5 6 7 8 9 10 [1] 2 3 4 5 6 7 8 9 10

这些输出结果表明，函数 `noop` 在测试框架中多次调用时，每次调用的结果都不同。


```go
func BenchmarkExecuteOne(b *testing.B) {
	noop := func() error {
		return nil
	}
	for i := 0; i < b.N; i++ {
		common.Must(Run(context.Background(), noop))
	}
}

func BenchmarkExecuteTwo(b *testing.B) {
	noop := func() error {
		return nil
	}
	for i := 0; i < b.N; i++ {
		common.Must(Run(context.Background(), noop, noop))
	}
}

```

# `common/uuid/uuid.go`

这段代码的作用是生成一个UUID（ universally unique identifier ），并将其存储在byteGroups数组中。

具体来说，该代码首先从v2ray.com/core/common/uuid导入了一个名为"uuid"的包，然后从该包中导入了"v2ray.com/core/common/common/uuid"包。接着，定义了一个名为byteGroups的整型变量，其值为8、4、4、4、12五个字节。

接着，定义了一个名为randvar的整型变量，并从crypto/rand包中获取一个随机整数，然后将其赋值给randvar。

最后，定义了一个名为hexEncode的函数，该函数接受一个字节切片（也就是byteGroups数组）作为参数，并使用十六进制编码将其转换为字符串。

总的来说，这段代码的作用是生成一个唯一的UUID，并将其存储在byteGroups数组中，可以用于标识和区分不同的数据。


```go
package uuid // import "v2ray.com/core/common/uuid"

import (
	"bytes"
	"crypto/rand"
	"encoding/hex"

	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
)

var (
	byteGroups = []int{8, 4, 4, 4, 12}
)

```

这段代码定义了一个名为 UUID 的类型，其字节数为 16。

在函数 (u *UUID) String() 中，首先创建一个名为 u 的 UUID 类型的变量。

然后，函数返回一个字符串表示该 UUID。为了实现这个功能，函数将 UUID 的字节数组转换为字符串，并使用一系列的 " hypens" (下划线) 来输出 UUID。

具体来说，函数的作用可以分为以下几个步骤：

1. 将 UUID 的字节数组转换为一个字符串，使用 hex.EncodeToString() 函数实现。这个字符串包含了 UUID 的所有字节。

2. 输出第一个字节。

3. 遍历 UUID 的所有字节，将它们分成固定长度的组，并输出每个组的内容。

4. 将组名和组内的字节一起输出。

总而言之，这段代码定义了一个名为 UUID 的类型，并实现了一个名为 (u *UUID) String() 的函数，用于将 UUID 的字节数组转换为字符串并输出 UUID。


```go
type UUID [16]byte

// String returns the string representation of this UUID.
func (u *UUID) String() string {
	bytes := u.Bytes()
	result := hex.EncodeToString(bytes[0 : byteGroups[0]/2])
	start := byteGroups[0] / 2
	for i := 1; i < len(byteGroups); i++ {
		nBytes := byteGroups[i] / 2
		result += "-"
		result += hex.EncodeToString(bytes[start : start+nBytes])
		start += nBytes
	}
	return result
}

```

该代码定义了一个名为UUID的UUID类型，并实现了两个函数：

1. `Bytes()`函数返回一个字节序列的UUID表示形式。该函数将UUID的字节序列复制到字节数组中，并返回该字节数组。

2. `Equals()`函数比较两个UUID是否相等。该函数接收两个UUID的指针，首先检查两个UUID是否为空(即`u`和`another`都为`nil`)，如果是，则两个UUID显然不相等，返回`false`。然后，函数比较两个UUID的字节序列是否相等。具体来说，函数将两个UUID的字节序列分别提取出来，然后比较它们是否相等。如果两个UUID的字节序列相等，则函数返回`true`。

该代码的实现基于以下假设：

- `u`和`another`都保存了原始的UUID值。
- `bytes.Equal()`函数用于比较两个字节序列是否相等，这个函数会返回`true`如果两个字节序列相等，否则返回`false`。

该代码对于需要将UUID转换为字节序列或比较两个UUID是否相等的需求非常重要。


```go
// Bytes returns the bytes representation of this UUID.
func (u *UUID) Bytes() []byte {
	return u[:]
}

// Equals returns true if this UUID equals another UUID by value.
func (u *UUID) Equals(another *UUID) bool {
	if u == nil && another == nil {
		return true
	}
	if u == nil || another == nil {
		return false
	}
	return bytes.Equal(u.Bytes(), another.Bytes())
}

```

这段代码定义了两个名为 `New` 和 `ParseBytes` 的函数，以及它们的接口 `UUID`。

`New` 函数使用 `rand` 函数生成一个随机的 UUID，并将其返回。

`ParseBytes` 函数接受一个字节切片 `b`，将其转换为 UUID 对象。如果 `b` 长度不足 16 字节，函数将返回并指出原因。否则，函数将复制 `b` 中的字节到一个新的 UUID 对象中，并返回该对象。

函数的实现看起来是为了在应用程序中生成和解析 UUID。不过，这些函数并没有对 UUID 的类型进行确认，因此使用它们时需要根据需要进行确认。


```go
// New creates a UUID with random value.
func New() UUID {
	var uuid UUID
	common.Must2(rand.Read(uuid.Bytes()))
	return uuid
}

// ParseBytes converts a UUID in byte form to object.
func ParseBytes(b []byte) (UUID, error) {
	var uuid UUID
	if len(b) != 16 {
		return uuid, errors.New("invalid UUID: ", b)
	}
	copy(uuid[:], b)
	return uuid, nil
}

```

这段代码定义了一个名为 `ParseString` 的函数，它接受一个字符串参数 `str`，并返回一个 `UUID` 对象和一个错误对象 `error`。

函数的作用是将给定的字符串 `str` 解析为 UUID 对象。函数的核心部分如下：

1. 检查给定的字符串 `str` 是否符合解析 UUID 的条件，如果不符合，则返回 `errors.New` 并返回错误对象。
2. 将字符串 `str` 转换为字节切片 `text`。
3. 将 `text` 中的前 32 位字节格式的字节数据存储到一个 `var` 类型的 `uuid` 变量中。
4. 使用 `for` 循环遍历 `text` 切片，对于每个字节格式的子串，按照以下步骤操作：
	1. 如果给定的前缀字节 `text[0]` 是 '-'，则将 `text` 切片后移一位，去掉前缀字节。
	2. 尝试使用 `hex.Decode` 函数将前缀字节 `text[0]` 解析为字符串，并将其存储到一个 `var` 类型的 `text` 切片中的第二个元素。
	3. 去掉前缀字节后，继续从 `text` 切片中间的位置开始，向前 `byteGroup` 大小的字符。
	4. 如果解析过程中出现错误，返回 `error` 并返回错误对象。
5. 如果所有步骤都正确，返回 `uuid` 变量，如果没有错误，返回 `nil`。


```go
// ParseString converts a UUID in string form to object.
func ParseString(str string) (UUID, error) {
	var uuid UUID

	text := []byte(str)
	if len(text) < 32 {
		return uuid, errors.New("invalid UUID: ", str)
	}

	b := uuid.Bytes()

	for _, byteGroup := range byteGroups {
		if text[0] == '-' {
			text = text[1:]
		}

		if _, err := hex.Decode(b[:byteGroup/2], text[:byteGroup]); err != nil {
			return uuid, err
		}

		text = text[byteGroup:]
		b = b[byteGroup/2:]
	}

	return uuid, nil
}

```