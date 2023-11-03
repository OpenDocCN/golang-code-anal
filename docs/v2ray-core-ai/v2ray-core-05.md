# v2ray-core源码解析 5

# `app/dns/server_test.go`

这段代码是一个 Go 语言编写的测试套件，用于测试 DNS 服务器的功能。它包括以下主要部分：

1. 导入一些必要的库：
	* testing.T
	* time.T
	* dns.DNS
	* net.IP
	* common.DNS
	* package.不得不包含一些常用的库，比如 testing 和 time。
2. 导入一些用于 DNS 服务器的高管：
	* dns.Addr
	* dns.TLSDatabase
	* dns.Text
	* 系统时间 "github.com/google/go-cmp/cmp"
	* "v2ray.com/core/app/dispatcher"
	* "v2ray.com/core/app/dns"
	* "v2ray.com/core/app/policy"
	* "v2ray.com/core/app/proxyman"
	* "v2ray.com/core/app/proxyman/outbound"
	* "v2ray.com/core/app/router"
3. 定义一些用于测试的函数：
	* testing.T(func TestDNS(t *testing.T) {...})
	* testing.T(func TestDNSWithCompliance(t *testing.T) {...})
	* testing.T(func TestDNSWithoutCompliance(t *testing.T) {...})
	* testing.T(func TestDNSWithPrecision(t *testing.T) {...})
	* testing.T(func TestDNSWithToS(t *testing.T) {...})
	* testing.T(func TestDNSWithMarket(t *testing.T) {...})
4. 定义一些用于 DNS 服务器的高管：
	* dns.AddrType
	* dns.TLSDatabase
	* dns.Text
	* 系统时间 "github.com/google/go-cmp/cmp"
	* "v2ray.com/core/app/dispatcher"
	* "v2ray.com/core/app/dns"
	* "v2ray.com/core/app/policy"
	* "v2ray.com/core/app/proxyman"
	* "v2ray.com/core/app/proxyman/outbound"
	* "v2ray.com/core/app/router"
5. 导入一些用于 DNS 客户端的库：
	* "v2ray.com/core/app/dns"
	* "v2ray.com/core/app/policy"
	* "v2ray.com/core/app/proxyman"
	* "v2ray.com/core/app/proxyman/outbound"
	* "v2ray.com/core/app/router"
	* "v2ray.com/core/features/dns"
	* "github.com/miekg/dns"



```go
package dns_test

import (
	"testing"
	"time"

	"github.com/google/go-cmp/cmp"
	"github.com/miekg/dns"

	"v2ray.com/core"
	"v2ray.com/core/app/dispatcher"
	. "v2ray.com/core/app/dns"
	"v2ray.com/core/app/policy"
	"v2ray.com/core/app/proxyman"
	_ "v2ray.com/core/app/proxyman/outbound"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/serial"
	feature_dns "v2ray.com/core/features/dns"
	"v2ray.com/core/proxy/freedom"
	"v2ray.com/core/testing/servers/udp"
)

```

This is a Go program that responds to DNS queries. It uses the `dns` package to query the DNS server for queries about domain names.

The program supports queries for FQ DNS queries, which are queries that only ask for the top-level part of a domain name. For example, a query like `127.0.0.1.IN-ADR.hostname.local. SOA ...` would be interpreted as asking the DNS server for the IP address of the FQ domain "hostname.local.", which is 127.0.0.1 in this case.

The program also supports queries for fully qualified domain names (FQDNS), which are queries that ask for the entire FQ domain name, including the top-level and second-level parts. For example, a query like `hostname.local. IN SOA ...` would be interpreted as asking the DNS server for the FQ domain name "hostname.local.", which would be "hostname.local.", and the IP address of the authoritative server for that domain name.

The program handles multiple queries and returns the IP address of the authoritative server for each query it receives. If a query is not supported by any of the query methods it uses, it returns a response with a "firewall-insecure-query" error.

The program also includes code that logs any DNS queries it receives, which can be useful for debugging purposes.


```go
type staticHandler struct {
}

func (*staticHandler) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
	ans := new(dns.Msg)
	ans.Id = r.Id

	var clientIP net.IP

	opt := r.IsEdns0()
	if opt != nil {
		for _, o := range opt.Option {
			if o.Option() == dns.EDNS0SUBNET {
				subnet := o.(*dns.EDNS0_SUBNET)
				clientIP = subnet.Address
			}
		}
	}

	for _, q := range r.Question {
		if q.Name == "google.com." && q.Qtype == dns.TypeA {
			if clientIP == nil {
				rr, _ := dns.NewRR("google.com. IN A 8.8.8.8")
				ans.Answer = append(ans.Answer, rr)
			} else {
				rr, _ := dns.NewRR("google.com. IN A 8.8.4.4")
				ans.Answer = append(ans.Answer, rr)
			}
		} else if q.Name == "api.google.com." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("api.google.com. IN A 8.8.7.7")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "v2.api.google.com." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("v2.api.google.com. IN A 8.8.7.8")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "facebook.com." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("facebook.com. IN A 9.9.9.9")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "ipv6.google.com." && q.Qtype == dns.TypeA {
			rr, err := dns.NewRR("ipv6.google.com. IN A 8.8.8.7")
			common.Must(err)
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "ipv6.google.com." && q.Qtype == dns.TypeAAAA {
			rr, err := dns.NewRR("ipv6.google.com. IN AAAA 2001:4860:4860::8888")
			common.Must(err)
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "notexist.google.com." && q.Qtype == dns.TypeAAAA {
			ans.MsgHdr.Rcode = dns.RcodeNameError
		} else if q.Name == "hostname." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("hostname. IN A 127.0.0.1")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "hostname.local." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("hostname.local. IN A 127.0.0.1")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "hostname.localdomain." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("hostname.localdomain. IN A 127.0.0.1")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "localhost." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("localhost. IN A 127.0.0.2")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "localhost-a." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("localhost-a. IN A 127.0.0.3")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "localhost-b." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("localhost-b. IN A 127.0.0.4")
			ans.Answer = append(ans.Answer, rr)
		} else if q.Name == "Mijia\\ Cloud." && q.Qtype == dns.TypeA {
			rr, _ := dns.NewRR("Mijia\\ Cloud. IN A 127.0.0.1")
			ans.Answer = append(ans.Answer, rr)
		}
	}
	w.WriteMsg(ans)
}

```

This is a Go program that sets up a DNS server on port 80 and listens for DNS queries. It also sets up a load balancer to forward DNS queries to multiple backend servers.

The program uses the CoreOSDNS service for DNS resolution and the Proxyman network transport library for inter-process communication between the components of the DNS server.

The DNS server is configured to listen for DNS queries on port 80 and to return responses for the query "google.com". The query is sent as a network UDP request with an IP address and port number to which the DNS server is listening.

The program uses a load balancer configured to distribute incoming DNS queries to multiple backend servers. The backend servers are assumed to be running the same CoreOSDNS service as the DNS server and are configured to listen for incoming queries on port 80.

The program also sets up a security policy to allow incoming traffic from the IP addresses of the query sender.

The program uses the Go DNS subdomain to perform a DNS lookup for the domain "google.com" and returns the IP address of the server that responded with the query. The output of the DNS lookup is then sent to the load balancer, which routes the query to one of the backend servers based on the "版本文档" configuration.


```go
func TestUDPServerSubnet(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: uint32(port),
					},
				},
				ClientIp: []byte{7, 8, 9, 10},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	ips, err := client.LookupIP("google.com")
	if err != nil {
		t.Fatal("unexpected error: ", err)
	}

	if r := cmp.Diff(ips, []net.IP{{8, 8, 4, 4}}); r != "" {
		t.Fatal(r)
	}
}

```

This is a Go test that checks the DNS server's response to a request to look up an IP address. The IP address is "google.com" and it has the IPv4 address "8.8.8.8 and 8.8.4.4".

The test starts by connecting to the DNS server and looking up the IP address "google.com" using the `LookupIP` method of the `feature_dns.IPv4Lookup` type. The expected result is that the DNS server returns an array containing all IP addresses with the same IP prefix (in this case, "8.8.8.0/24").

The test then uses the `cmp.Diff` method to compare the DNS server's response to the expected result. If there is any difference, it is assumed that the DNS server failed to respond correctly.

The test then checks the response for any NameErrors by using the `feature_dns.RCodeFromError` method to convert the DNS server's error response to an error code and compare it to the expected error code. If the conversion fails, it is assumed that the DNS server responded with an error code.

Finally, the test checks the DNS server's response for a lookup for the IP address "ipv4only.google.com". The IP address is expected to have the IPv4 address "8.8.8.8 and 8.8.4.4". The test uses the `feature_dns.IPv6Lookup` type to look up the IP address and checks the response for any errors. If the DNS server returns an error, it is assumed that the IP address could not be resolved.


```go
func TestUDPServer(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: uint32(port),
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	{
		ips, err := client.LookupIP("google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	{
		ips, err := client.LookupIP("facebook.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{9, 9, 9, 9}}); r != "" {
			t.Fatal(r)
		}
	}

	{
		_, err := client.LookupIP("notexist.google.com")
		if err == nil {
			t.Fatal("nil error")
		}
		if r := feature_dns.RCodeFromError(err); r != uint16(dns.RcodeNameError) {
			t.Fatal("expected NameError, but got ", r)
		}
	}

	{
		clientv6 := client.(feature_dns.IPv6Lookup)
		ips, err := clientv6.LookupIPv6("ipv4only.google.com")
		if err != feature_dns.ErrEmptyResponse {
			t.Fatal("error: ", err)
		}
		if len(ips) != 0 {
			t.Fatal("ips: ", ips)
		}
	}

	dnsServer.Shutdown()

	{
		ips, err := client.LookupIP("google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}
}

```

This is a Go program that performs a DNS query to Google.com using the "google.com" IP address and a randomly chosen port. It uses the Internet Assigned Numbers Authority (IANA) data to determine if the query domain name matches the IP address or not.

The program uses the following configuration parameters:

* IPAddr: The IP address to query
* Port: The port number to use for the query
* PrioritizedDomain: The domain to prioritize for the query (optional)

The program uses a `core.OutboundHandlerConfig` for the Outbound traffic and a `feature_dns.Client` for the DNS query.

The program performs the DNS query using the `client.LookupIP` method, which takes a `domain` parameter to specify the IP address to query for. The method returns a slice of `IPv4Addresses` representing the IP addresses that match the query domain name.

The program uses a comparison to compare the query result to the expected result. If the query result is not a match, the program will print an error and stop at that point. If the query result matches the expected result, the program will not print anything and continue with the next step.

The program uses the `time.Now` method to calculate the start and end times for the DNS query. If the start time is 2 seconds or more after the end time, the program will print an error and stop at that point.


```go
func TestPrioritizedDomain(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: 9999, /* unreachable */
					},
				},
				NameServer: []*NameServer{
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							{
								Type:   DomainMatchingType_Full,
								Domain: "google.com",
							},
						},
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	startTime := time.Now()

	{
		ips, err := client.LookupIP("google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	endTime := time.Now()
	if startTime.After(endTime.Add(time.Second * 2)) {
		t.Error("DNS query doesn't finish in 2 seconds.")
	}
}

```

This is a Go program that sets up a DNS server to resolve hostnames to IP addresses using the花生壳 tool. It creates a Kubernetes ConfigMap, a Kubernetes Service, and a Kubernetes Deployment.

The program reads a ConfigMap template from a file, which specifies the server's name servers，端口， and the IP address to bind to. It then creates a ConfigMessage for the ConfigMap, which is then serialized and deserialized from the computer's network stack.

The program then sets up an Outbound Handler for the service, which will resolve hostnames to IP addresses using the花生壳 tool.


```go
func TestUDPServerIPv6(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: uint32(port),
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)
	client6 := client.(feature_dns.IPv6Lookup)

	{
		ips, err := client6.LookupIPv6("ipv6.google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{32, 1, 72, 96, 72, 96, 0, 0, 0, 0, 0, 0, 0, 0, 136, 136}}); r != "" {
			t.Fatal(r)
		}
	}
}

```

This is a Go program that sets up a DNS server to resolve hostnames to IP addresses.

It creates a DNS server using the `net` package, which provides UDP networking services for the Go language.

The DNS server listens for incoming requests on port 53 and resolves hostnames to IP addresses using a combination of a whitelist and a resolve network.

The `whitelist` is a list of IP addresses that are allowed to be used to resolve hostnames to IP addresses.

The `resolve` is a function that resolves a hostname to an IP address using the whitelist and other configured networks.

The `core` package provides a base implementation for the core DNS server, including a handler for incoming requests.

The `feature_dns` package provides a helper function to check if a given function is a valid feature for the `core` package.

The `serial` package provides serialization and deserialization for message passing between the server and the client.

The `policy` package provides the necessary configuration for the network policies, such as firewall rules.

The `dispatcher` package provides the necessary middleware for the handler to communicate with the `core` package and the network policies.

The `typed_message` package provides the type安全的 serialization for message passing between the server and the client.

The `poa` package provides the Policy Orchestration API.


```go
func TestStaticHostDomain(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: uint32(port),
					},
				},
				StaticHosts: []*Config_HostMapping{
					{
						Type:          DomainMatchingType_Full,
						Domain:        "example.com",
						ProxiedDomain: "google.com",
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	{
		ips, err := client.LookupIP("example.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	dnsServer.Shutdown()
}

```

This is a Go program that performs a DNS query to a Google domain using the "代理曼"代理和 "策略"配置.它还配置了一个 "DNS服务器" 和 "Outbound 代理" 选项。

程序首先定义了一个 "纵向切分" 配置字符串，用于设置 DNS 查询的分片。然后它创建了一个 "核心代理" 和 "配置" 类型的代理实例，并使用 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置字符串转换为 "Serial" 类型，以便将配置存储到 "dispatcher" 和 "proxyman" 类型。

接下来程序创建了 "Outbound 代理" 和 "DNS服务器" 类型的实例。程序使用 "GetFeature" 函数获取了指定的 "核心代理" 实例，然后使用 "GetFeature" 函数获取了指定的 "DNS服务器" 实例。

程序使用 "纵向切分" 配置字符串将 DNS 查询切分为多个部分，并使用 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置存储到 "dispatcher" 和 "proxyman" 类型。

程序使用 "Dispatcher" 和 "ProxyMan" 类型来处理 DNS 查询的代理和配置。

最后，程序使用 "纵向切分" 配置字符串和 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置存储到 "dispatcher" 和 "proxyman" 类型。然后，程序使用 "GetFeature" 函数获取了指定的 "核心代理" 实例，并使用其 "Outbound" 选项将 DNS 查询发送到指定的代理实例。

程序还使用 "纵向切分" 配置字符串和 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置存储到 "dispatcher" 和 "proxyman" 类型。然后，程序使用 "GetFeature" 函数获取了指定的 "DNS服务器" 实例，并使用其 "Outbound" 选项将 DNS 查询发送到指定的代理实例。

程序使用 "纵向切分" 配置字符串和 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置存储到 "dispatcher" 和 "proxyman" 类型。然后，程序使用 "GetFeature" 函数获取了指定的 "核心代理" 实例，并使用其 "Outbound" 选项将 DNS 查询发送到指定的代理实例。

程序使用 "纵向切分" 配置字符串和 "golang.org/x/net/proxy/config" 包中的 "Serial" 函数将配置存储到 "dispatcher" 和 "proxyman" 类型。然后，程序使用 "GetFeature" 函数获取了指定的 "DNS服务器" 实例，并使用其 "Outbound" 选项将 DNS 查询发送到指定的代理实例。


```go
func TestIPMatch(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServer: []*NameServer{
					// private dns, not match
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						Geoip: []*router.GeoIP{
							{
								CountryCode: "local",
								Cidr: []*router.CIDR{
									{
										// inner ip, will not match
										Ip:     []byte{192, 168, 11, 1},
										Prefix: 32,
									},
								},
							},
						},
					},
					// second dns, match ip
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						Geoip: []*router.GeoIP{
							{
								CountryCode: "test",
								Cidr: []*router.CIDR{
									{
										Ip:     []byte{8, 8, 8, 8},
										Prefix: 32,
									},
								},
							},
							{
								CountryCode: "test",
								Cidr: []*router.CIDR{
									{
										Ip:     []byte{8, 8, 8, 4},
										Prefix: 32,
									},
								},
							},
						},
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	startTime := time.Now()

	{
		ips, err := client.LookupIP("google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	endTime := time.Now()
	if startTime.After(endTime.Add(time.Second * 2)) {
		t.Error("DNS query doesn't finish in 2 seconds.")
	}
}

```

This is a Go test that checks if the DNS server provided by Cloudflare is working correctly. The test uses the `client.LookupIP` method to query the DNS server for the IP address of "localhost-a" (or "127.0.0.1" in this case), and the `cmp.Diff` method to compare the query result to the expected result.

The expected results are that the query should return the IP address "127.0.0.2" for the IP address "localhost-a". The query that is being tested in this case is that the DNS server should return the IP addresses "127.0.0.2" and "127.0.0.3" for the IP addresses "localhost-a" and "localhost-b".

If the query returns any other IP addresses, or if the test takes longer than 2 seconds, the test will panicked and indicate that the DNS server is not working correctly.


```go
func TestLocalDomain(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: 9999, /* unreachable */
					},
				},
				NameServer: []*NameServer{
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							// Equivalent of dotless:localhost
							{Type: DomainMatchingType_Regex, Domain: "^[^.]*localhost[^.]*$"},
						},
						Geoip: []*router.GeoIP{
							{ // Will match localhost, localhost-a and localhost-b,
								CountryCode: "local",
								Cidr: []*router.CIDR{
									{Ip: []byte{127, 0, 0, 2}, Prefix: 32},
									{Ip: []byte{127, 0, 0, 3}, Prefix: 32},
									{Ip: []byte{127, 0, 0, 4}, Prefix: 32},
								},
							},
						},
					},
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							// Equivalent of dotless: and domain:local
							{Type: DomainMatchingType_Regex, Domain: "^[^.]*$"},
							{Type: DomainMatchingType_Subdomain, Domain: "local"},
							{Type: DomainMatchingType_Subdomain, Domain: "localdomain"},
						},
					},
				},
				StaticHosts: []*Config_HostMapping{
					{
						Type:   DomainMatchingType_Full,
						Domain: "hostnamestatic",
						Ip:     [][]byte{{127, 0, 0, 53}},
					},
					{
						Type:          DomainMatchingType_Full,
						Domain:        "hostnamealias",
						ProxiedDomain: "hostname.localdomain",
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	startTime := time.Now()

	{ // Will match dotless:
		ips, err := client.LookupIP("hostname")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match domain:local
		ips, err := client.LookupIP("hostname.local")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match static ip
		ips, err := client.LookupIP("hostnamestatic")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 53}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match domain replacing
		ips, err := client.LookupIP("hostnamealias")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match dotless:localhost, but not expectIPs: 127.0.0.2, 127.0.0.3, then matches at dotless:
		ips, err := client.LookupIP("localhost")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 2}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match dotless:localhost, and expectIPs: 127.0.0.2, 127.0.0.3
		ips, err := client.LookupIP("localhost-a")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 3}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match dotless:localhost, and expectIPs: 127.0.0.2, 127.0.0.3
		ips, err := client.LookupIP("localhost-b")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 4}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match dotless:
		ips, err := client.LookupIP("Mijia Cloud")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
			t.Fatal(r)
		}
	}

	endTime := time.Now()
	if startTime.After(endTime.Add(time.Second * 2)) {
		t.Error("DNS query doesn't finish in 2 seconds.")
	}
}

```

This is a Go test that checks if the DNS query from a client to a Google server returns the expected IP address. The test includes several variations, each starting with a different server and DNS query, and uses the `cmp.Diff` function to compare the query response from each server to the expected response. If any of the queries return an unexpected IP address or take longer than expected, the test will fail and the tester will be notified. The test also checks if the DNS query takes longer than 2 seconds and if the query finishes before that.


```go
func TestMultiMatchPrioritizedDomain(t *testing.T) {
	port := udp.PickPort()

	dnsServer := dns.Server{
		Addr:    "127.0.0.1:" + port.String(),
		Net:     "udp",
		Handler: &staticHandler{},
		UDPSize: 1200,
	}

	go dnsServer.ListenAndServe()
	time.Sleep(time.Second)

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&Config{
				NameServers: []*net.Endpoint{
					{
						Network: net.Network_UDP,
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: 9999, /* unreachable */
					},
				},
				NameServer: []*NameServer{
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							{
								Type:   DomainMatchingType_Subdomain,
								Domain: "google.com",
							},
						},
						Geoip: []*router.GeoIP{
							{ // Will only match 8.8.8.8 and 8.8.4.4
								Cidr: []*router.CIDR{
									{Ip: []byte{8, 8, 8, 8}, Prefix: 32},
									{Ip: []byte{8, 8, 4, 4}, Prefix: 32},
								},
							},
						},
					},
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							{
								Type:   DomainMatchingType_Subdomain,
								Domain: "google.com",
							},
						},
						Geoip: []*router.GeoIP{
							{ // Will match 8.8.8.8 and 8.8.8.7, etc
								Cidr: []*router.CIDR{
									{Ip: []byte{8, 8, 8, 7}, Prefix: 24},
								},
							},
						},
					},
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							{
								Type:   DomainMatchingType_Subdomain,
								Domain: "api.google.com",
							},
						},
						Geoip: []*router.GeoIP{
							{ // Will only match 8.8.7.7 (api.google.com)
								Cidr: []*router.CIDR{
									{Ip: []byte{8, 8, 7, 7}, Prefix: 32},
								},
							},
						},
					},
					{
						Address: &net.Endpoint{
							Network: net.Network_UDP,
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{127, 0, 0, 1},
								},
							},
							Port: uint32(port),
						},
						PrioritizedDomain: []*NameServer_PriorityDomain{
							{
								Type:   DomainMatchingType_Full,
								Domain: "v2.api.google.com",
							},
						},
						Geoip: []*router.GeoIP{
							{ // Will only match 8.8.7.8 (v2.api.google.com)
								Cidr: []*router.CIDR{
									{Ip: []byte{8, 8, 7, 8}, Prefix: 32},
								},
							},
						},
					},
				},
			}),
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
			serial.ToTypedMessage(&policy.Config{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	v, err := core.New(config)
	common.Must(err)

	client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

	startTime := time.Now()

	{ // Will match server 1,2 and server 1 returns expected ip
		ips, err := client.LookupIP("google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match server 1,2 and server 1 returns unexpected ip, then server 2 returns expected one
		clientv4 := client.(feature_dns.IPv4Lookup)
		ips, err := clientv4.LookupIPv4("ipv6.google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 7}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match server 3,1,2 and server 3 returns expected one
		ips, err := client.LookupIP("api.google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 7, 7}}); r != "" {
			t.Fatal(r)
		}
	}

	{ // Will match server 4,3,1,2 and server 4 returns expected one
		ips, err := client.LookupIP("v2.api.google.com")
		if err != nil {
			t.Fatal("unexpected error: ", err)
		}

		if r := cmp.Diff(ips, []net.IP{{8, 8, 7, 8}}); r != "" {
			t.Fatal(r)
		}
	}

	endTime := time.Now()
	if startTime.After(endTime.Add(time.Second * 2)) {
		t.Error("DNS query doesn't finish in 2 seconds.")
	}
}

```

# `app/dns/udpns.go`

这段代码是一个 Go 语言编写的 DNS 服务器，它用于实现 UDP 协议中的 DNS 查询服务。它的作用是为用户提供一个高效、可靠的 DNS 服务，允许用户通过 DNS 查询获取目标主机的信息。

具体来说，这段代码实现了一个 DNS 服务器，当有客户端发起 DNS 查询请求时，它会通过 UDP 协议传输到客户端的 DNS 服务器，然后处理客户端的请求信息，最后返回查询结果。客户端发送完 DNS 查询请求后，服务器会将其保存到 DNS 缓存中，以提高下一次查询的速度。

此外，这段代码还实现了一个高可用性的设计，即使主服务器出现故障，自动心跳机制也可以确保在故障服务器被修复或者关闭后，从辅服务器自动接管 DNS 查询服务，保证服务的高可用性。


```go
// +build !confonly

package dns

import (
	"context"
	"strings"
	"sync"
	"sync/atomic"
	"time"

	"golang.org/x/net/dns/dnsmessage"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/dns"
	udp_proto "v2ray.com/core/common/protocol/udp"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal/pubsub"
	"v2ray.com/core/common/task"
	dns_feature "v2ray.com/core/features/dns"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport/internet/udp"
)

```

该代码定义了一个名为"ClassicNameServer"的结构体，用于实现DNS服务器。具体来说，它实现了以下几个功能：

1. 创建一个名为"ClassicNameServer"的同步读写互斥锁(RWMutex)，用于在读写同时进行操作时保证数据的一致性。
2. 设置DNS服务器的名称(即服务器的主机名)，并将其存储在一个名为"name"的变量中，以便在代码中进行使用。
3. 创建一个名为"address"的变量，用于存储服务器的目标IP地址，然后将其存储在一个名为"ips"的 map 中，以便在代码中进行存储和更新。
4. 创建一个名为"requests"的 map，用于存储DNS查询请求。
5. 创建一个名为"pub"的变量，用于存储DNS服务器发布的公共服务，然后将其存储在名为"pub"的变量中。
6. 创建一个名为"clientIP"的变量，用于存储目标服务器的IP地址，然后将其存储在名为"clientIP"的变量中。
7. 创建一个名为"reqID"的变量，用于标识每个DNS查询请求，然后将其存储在名为"reqID"的变量中。
8. 创建一个名为"Cleanup"的变量，用于存储清理定时器，以便在代码中进行管理。
9. 创建一个名为"handleResponse"的函数，用于处理DNS查询请求，并将其作为名为"pub"的函数的输入参数。
10. 创建一个名为"AtInfo"的函数，用于获取DNS查询请求的info字段，并将其作为函数输入参数。
11. 创建一个名为"WriteToLog"的函数，用于将信息日志到配置日志中，以便进行调试。

最后，函数返回一个名为"ClassicNameServer"的结构体，用于将创建的DNS服务器用于将客户端的查询请求发送到公共服务器的回显请求。


```go
type ClassicNameServer struct {
	sync.RWMutex
	name      string
	address   net.Destination
	ips       map[string]record
	requests  map[uint16]dnsRequest
	pub       *pubsub.Service
	udpServer *udp.Dispatcher
	cleanup   *task.Periodic
	reqID     uint32
	clientIP  net.IP
}

func NewClassicNameServer(address net.Destination, dispatcher routing.Dispatcher, clientIP net.IP) *ClassicNameServer {

	// default to 53 if unspecific
	if address.Port == 0 {
		address.Port = net.Port(53)
	}

	s := &ClassicNameServer{
		address:  address,
		ips:      make(map[string]record),
		requests: make(map[uint16]dnsRequest),
		clientIP: clientIP,
		pub:      pubsub.NewService(),
		name:     strings.ToUpper(address.String()),
	}
	s.cleanup = &task.Periodic{
		Interval: time.Minute,
		Execute:  s.Cleanup,
	}
	s.udpServer = udp.NewDispatcher(dispatcher, s.HandleResponse)
	newError("DNS: created udp client inited for ", address.NetAddr()).AtInfo().WriteToLog()
	return s
}

```

该代码定义了两个函数：`func (s *ClassicNameServer) Name() string` 和 `func (s *ClassicNameServer) Cleanup() error`。

函数 `Name()` 返回一个字符串，表示服务器实例的名称。

函数 `Cleanup()` 返回一个错误，用于在清理操作完成后通知用户。该函数在开始清理前创建了一个临时的锁，以避免在正在进行的 DNS 查询中移动或删除记录。清理操作包括：

1. 移除不再需要的 IP 地址。
2. 移除不再需要的 DNS 查询请求。
3. 如果服务器实例中已经没有 IP 地址或 DNS 查询请求，那么创建一个空的字典来存储 IP 地址。
4. 删除不再需要的 DNS 查询请求。
5. 如果服务器中仍然有 DNS 查询请求，那么将其存储在请求 map 中。


```go
func (s *ClassicNameServer) Name() string {
	return s.name
}

func (s *ClassicNameServer) Cleanup() error {
	now := time.Now()
	s.Lock()
	defer s.Unlock()

	if len(s.ips) == 0 && len(s.requests) == 0 {
		return newError(s.name, " nothing to do. stopping...")
	}

	for domain, record := range s.ips {
		if record.A != nil && record.A.Expire.Before(now) {
			record.A = nil
		}
		if record.AAAA != nil && record.AAAA.Expire.Before(now) {
			record.AAAA = nil
		}

		if record.A == nil && record.AAAA == nil {
			delete(s.ips, domain)
		} else {
			s.ips[domain] = record
		}
	}

	if len(s.ips) == 0 {
		s.ips = make(map[string]record)
	}

	for id, req := range s.requests {
		if req.expire.Before(now) {
			delete(s.requests, id)
		}
	}

	if len(s.requests) == 0 {
		s.requests = make(map[uint16]dnsRequest)
	}

	return nil
}

```

这段代码是一个名为"HandleResponse"的函数，属于名为"ClassicNameServer"的结构的"udp\_proto"包的实现。

函数接收两个参数：一个指向"udp\_proto.Packet"类型的变量"packet"，以及一个字符串类型的"ctx"参数。

函数首先调用一个名为"parseResponse"的函数，该函数接收一个字节数组，尝试解析DNS请求报文的负载，如果失败，则返回错误并记录到日志中。

如果解析成功，函数将"packet.Payload.Bytes()"的字节数组中的IPv4地址作为"ipRec"变量存储，然后尝试使用"s.requests" slice中的具有相应ID的请求，如果找到，则执行以下操作：删除该请求，然后执行新的"ipRec"操作。

如果未找到具有相应ID的请求，函数将记录错误并返回。

接下来，函数根据请求的类型，创建一个"rec"记录，并设置为"ipRec"。然后使用"ipRec.IP"字段更新"s.updateIP"函数中的IPv4地址。最后，函数使用"time.Since"函数获取响应时间，并使用"s.updateIP"函数更新"s.updateIP"函数中的IPv4地址。


```go
func (s *ClassicNameServer) HandleResponse(ctx context.Context, packet *udp_proto.Packet) {

	ipRec, err := parseResponse(packet.Payload.Bytes())
	if err != nil {
		newError(s.name, " fail to parse responded DNS udp").AtError().WriteToLog()
		return
	}

	s.Lock()
	id := ipRec.ReqID
	req, ok := s.requests[id]
	if ok {
		// remove the pending request
		delete(s.requests, id)
	}
	s.Unlock()
	if !ok {
		newError(s.name, " cannot find the pending request").AtError().WriteToLog()
		return
	}

	var rec record
	switch req.reqType {
	case dnsmessage.TypeA:
		rec.A = ipRec
	case dnsmessage.TypeAAAA:
		rec.AAAA = ipRec
	}

	elapsed := time.Since(req.start)
	newError(s.name, " got answer: ", req.domain, " ", req.reqType, " -> ", ipRec.IP, " ", elapsed).AtInfo().WriteToLog()
	if len(req.domain) > 0 && (rec.A != nil || rec.AAAA != nil) {
		s.updateIP(req.domain, rec)
	}
}

```

这段代码的作用是更新一个名为ClassicNameServer的的服务器的IP记录。它接受一个域名参数s (指向ClassicNameServer结构体的指针)，一个新的IP记录newRec (包含A和AAAA类型的字段)，并尝试更新该域名下的IP记录。

具体来说，代码首先获取当前域名下的IP记录rec，然后检查新IP记录A和AAAA是否与rec的A和AAAA类型相同。如果是，就更新了新的IP记录，并将其存储在rec中。然后，代码检查新IP记录A和AAAA是否都已知，如果是，就在服务器上发布一个消息，通知有新的IP记录已发布。最后，代码使用s.pub发布一条消息，通知服务器的pub代理，并在清理过程中启动清理操作。


```go
func (s *ClassicNameServer) updateIP(domain string, newRec record) {
	s.Lock()

	newError(s.name, " updating IP records for domain:", domain).AtDebug().WriteToLog()
	rec := s.ips[domain]

	updated := false
	if isNewer(rec.A, newRec.A) {
		rec.A = newRec.A
		updated = true
	}
	if isNewer(rec.AAAA, newRec.AAAA) {
		rec.AAAA = newRec.AAAA
		updated = true
	}

	if updated {
		s.ips[domain] = rec
	}
	if newRec.A != nil {
		s.pub.Publish(domain+"4", nil)
	}
	if newRec.AAAA != nil {
		s.pub.Publish(domain+"6", nil)
	}
	s.Unlock()
	common.Must(s.cleanup.Start())
}

```

这段代码定义了三个函数，分别作用于ClassicNameServer类的实例：

1. `func (s *ClassicNameServer) newReqID() uint16`：返回一个新的请求ID，使用原子操作原子性地从reqID数组中递增，并返回增加后的值。

2. `func (s *ClassicNameServer) addPendingRequest(req *dnsRequest)`：添加一个正在等待确认的请求到队列中，使用互斥锁确保只可能有一个实例同时执行该函数，然后使用递归关系将该请求添加到其它的实例中。

3. `func (s *ClassicNameServer) sendQuery(ctx context.Context, domain string, option IPOption)`：发送一个查询DNS记录的请求，并返回结果。函数使用了`session.ExportIDToError`函数来捕获错误，并使用`session.InboundFromContext`函数来检测是否有入站流量。如果检测到入站流量，则创建一个入站上下文并将结果写入错误日志。如果没有入站流量，则创建一个未入站上下文并开始发送查询。

函数的作用是将ClassicNameServer类的实例进行了一些操作，包括创建一个新的请求ID、添加正在等待确认的请求到队列中、发送查询DNS记录。


```go
func (s *ClassicNameServer) newReqID() uint16 {
	return uint16(atomic.AddUint32(&s.reqID, 1))
}

func (s *ClassicNameServer) addPendingRequest(req *dnsRequest) {
	s.Lock()
	defer s.Unlock()

	id := req.msg.ID
	req.expire = time.Now().Add(time.Second * 8)
	s.requests[id] = *req
}

func (s *ClassicNameServer) sendQuery(ctx context.Context, domain string, option IPOption) {
	newError(s.name, " querying DNS for: ", domain).AtDebug().WriteToLog(session.ExportIDToError(ctx))

	reqs := buildReqMsgs(domain, option, s.newReqID, genEDNS0Options(s.clientIP))

	for _, req := range reqs {
		s.addPendingRequest(req)
		b, _ := dns.PackMessage(req.msg)
		udpCtx := context.Background()
		if inbound := session.InboundFromContext(ctx); inbound != nil {
			udpCtx = session.ContextWithInbound(udpCtx, inbound)
		}
		udpCtx = session.ContextWithContent(udpCtx, &session.Content{
			Protocol: "dns",
		})
		s.udpServer.Dispatch(udpCtx, s.address, b)
	}
}

```

该函数的作用是获取给定域名下的IP地址，并返回一个IP数组或错误。它使用了基于超时准备的锁（RLock）和读时锁（RUnlock）来确保在尝试获取记录时不会阻止其他函数调用。

具体来说，它首先尝试在内存中的ips数组中查找给定域名下的记录，如果找到了，则获取该记录中的IPv4或IPv6地址，并将它们添加到ips数组中。接下来，根据传入的IP选项（IPv4或IPv6），它将获取所有可能的A、AAAA和AAAAA服务器地址。如果所有IP地址都被找到，函数将返回ips数组，否则返回一个空字符串并处理错误。


```go
func (s *ClassicNameServer) findIPsForDomain(domain string, option IPOption) ([]net.IP, error) {
	s.RLock()
	record, found := s.ips[domain]
	s.RUnlock()

	if !found {
		return nil, errRecordNotFound
	}

	var ips []net.Address
	var lastErr error
	if option.IPv4Enable {
		a, err := record.A.getIPs()
		if err != nil {
			lastErr = err
		}
		ips = append(ips, a...)
	}

	if option.IPv6Enable {
		aaaa, err := record.AAAA.getIPs()
		if err != nil {
			lastErr = err
		}
		ips = append(ips, aaaa...)
	}

	if len(ips) > 0 {
		return toNetIP(ips), nil
	}

	if lastErr != nil {
		return nil, lastErr
	}

	return nil, dns_feature.ErrEmptyResponse
}

```

该函数的作用是查询给定域名的服务器是否支持指定Option中的IP选项，并返回相应的IP地址。它属于一个名为ClassicNameServer的类型的实例。

具体来说，函数接收三个参数：

1. s：ClassicNameServer类型的实例，它包含了服务器的一些状态，如当前连接的客户端列表和一些内部数据结构。
2. domain：要查询的域名，它用于指定查询范围。
3. option：一个IPOption类型的参数，它包含了查询时需要设置的选项，如IPv4或IPv6。

函数首先查询fqdn（完整的域名，除去顶级域名）是否存在于服务器中的缓存中，如果存在，则直接返回缓存中的结果，否则继续下一步。

接着，函数使用s.findIPsForDomain函数查询服务器是否知道该域名，如果知道，则返回相应的IP地址。这个函数可能会在缓存中查找，如果没有，则需要向服务器发送请求并接收响应，然后将IP地址存储在缓存中。

如果服务器返回的错误是errRecordNotFound，函数会记录到错误并输出错误信息，然后返回一个空 slice，表示查询失败。


```go
func (s *ClassicNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {

	fqdn := Fqdn(domain)

	ips, err := s.findIPsForDomain(fqdn, option)
	if err != errRecordNotFound {
		newError(s.name, " cache HIT ", domain, " -> ", ips).Base(err).AtDebug().WriteToLog()
		return ips, err
	}

	// ipv4 and ipv6 belong to different subscription groups
	var sub4, sub6 *pubsub.Subscriber
	if option.IPv4Enable {
		sub4 = s.pub.Subscribe(fqdn + "4")
		defer sub4.Close()
	}
	if option.IPv6Enable {
		sub6 = s.pub.Subscribe(fqdn + "6")
		defer sub6.Close()
	}
	done := make(chan interface{})
	go func() {
		if sub4 != nil {
			select {
			case <-sub4.Wait():
			case <-ctx.Done():
			}
		}
		if sub6 != nil {
			select {
			case <-sub6.Wait():
			case <-ctx.Done():
			}
		}
		close(done)
	}()
	s.sendQuery(ctx, fqdn, option)

	for {
		ips, err := s.findIPsForDomain(fqdn, option)
		if err != errRecordNotFound {
			return ips, err
		}

		select {
		case <-ctx.Done():
			return nil, ctx.Err()
		case <-done:
		}
	}
}

```