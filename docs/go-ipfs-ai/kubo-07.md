# go-ipfs 源码解析 7

# `config/init.go`

这段代码定义了一个名为 "config" 的包，其中包含了一些用于配置本地安全哈希库的函数。

函数 "Init" 接受一个输出流 "out" 和一个表示哈希库所需加密密钥的二进制位数 "nBitsForKeypair"。它返回一个配置对象 "config" 和一个错误。

函数 "CreateIdentity" 接受一个输出流 "out" 和一个选项 "options"。它返回一个身份 "identity" 和一个错误。如果出现错误，函数将返回 nil。

函数 "InitWithIdentity" 返回一个配置对象 "config" 和一个错误。它首先调用 "CreateIdentity" 函数来创建一个身份，然后使用身份创建一个配置对象。

这里使用了libp2p库，它是Ipfs Boxo项目的核心libp2p库，提供了一些用于创建和管理本地安全哈希库的函数。


```go
package config

import (
	"crypto/rand"
	"encoding/base64"
	"fmt"
	"io"
	"time"

	"github.com/ipfs/boxo/coreiface/options"
	"github.com/libp2p/go-libp2p/core/crypto"
	"github.com/libp2p/go-libp2p/core/peer"
)

func Init(out io.Writer, nBitsForKeypair int) (*Config, error) {
	identity, err := CreateIdentity(out, []options.KeyGenerateOption{options.Key.Size(nBitsForKeypair)})
	if err != nil {
		return nil, err
	}

	return InitWithIdentity(identity)
}

```

This is a Go file that sets up a Kubernetes cluster. It sets up the cluster's datastore, identity, discovery, routing, and other configurations.

Here's the code:
go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/coreos/让自己/v0top0/节假日/datastore"
	"github.com/coreos/让自己/v0top0/节假日/math/private/contract"
	"github.com/coreos/让自己/v0top0/节假日/types"
	"github.com/coreos/让自己/v0top0/ib岐/contract/invoker"
	"github.com/coreos/让自己/v0top0/ib岐/model/address"
	"github.com/coreos/让自己/v0top0/net/rpc"
	"github.com/coreos/node/datastore/backend/futures"
	"github.com/coreos/node/datastore/backend/transaction"
	"github.com/coreos/node/datastore/kv"
	"github.com/coreos/node/datastore/options"
	"github.com/coreos/node/datastore/rpc/datastore"
	"github.com/coreos/node/datastore/rpc/protocol/长达60s"
	"github.com/coreos/node/datastore/service/模擬值/subservice"
	"github.com/coreos/node/datastore/transaction/Options"
	"github.com/coreos/node/datastore/transaction/predicate"
	"github.com/coreos/node/datastore/transaction/service/chunked"
	"github.com/coreos/node/datastore/transaction/service/datastore/client"
	"github.com/coreos/node/datastore/transaction/service/datastore/server"
	"github.com/coreos/node/datastore/transaction/service/datastore/transaction"
	"github.com/coreos/node/datastore/transaction/service/ipns/mock"
	"github.com/coreos/node/datastore/transaction/service/ipns/resolver"
	"github.com/coreos/node/datastore/transaction/service/ipns/translator"
	"github.com/coreos/node/datastore/transaction/service/ipns/upper"
	"github.com/coreos/node/datastore/transaction/service/ipns/worker"
	"github.com/coreos/node/datastore/transaction/service/ipns/world"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ medication"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ nurse"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ Peper再造Offer"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/这份工作的纳闷"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/在这份工作经历中"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/遥控"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/这里的负责人"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/做的更好"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/贡献"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ existence"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 上报"
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 组播 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/订阅 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/时间 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 你好 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 新鲜事物 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 你好你好 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 你好你好你好 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 你对 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 做什么 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 哪里 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 什么时候 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 分组 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 散布 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 创建 "
	"github.com/coreos/node/datastore/transaction/service/ipns/zookeeper/ 加入 "
	"github.com/coreos/node/datastore/trans


```go
func InitWithIdentity(identity Identity) (*Config, error) {
	bootstrapPeers, err := DefaultBootstrapPeers()
	if err != nil {
		return nil, err
	}

	datastore := DefaultDatastoreConfig()

	conf := &Config{
		API: API{
			HTTPHeaders: map[string][]string{},
		},

		// setup the node's default addresses.
		// NOTE: two swarm listen addrs, one tcp, one utp.
		Addresses: addressesConfig(),

		Datastore: datastore,
		Bootstrap: BootstrapPeerStrings(bootstrapPeers),
		Identity:  identity,
		Discovery: Discovery{
			MDNS: MDNS{
				Enabled: true,
			},
		},

		Routing: Routing{
			Type:    nil,
			Methods: nil,
			Routers: nil,
		},

		// setup the node mount points.
		Mounts: Mounts{
			IPFS: "/ipfs",
			IPNS: "/ipns",
		},

		Ipns: Ipns{
			ResolveCacheSize: 128,
		},

		Gateway: Gateway{
			RootRedirect: "",
			NoFetch:      false,
			PathPrefixes: []string{},
			HTTPHeaders:  map[string][]string{},
			APICommands:  []string{},
		},
		Reprovider: Reprovider{
			Interval: nil,
			Strategy: nil,
		},
		Pinning: Pinning{
			RemoteServices: map[string]RemotePinningService{},
		},
		DNS: DNS{
			Resolvers: map[string]string{},
		},
		Migration: Migration{
			DownloadSources: []string{},
			Keep:            "",
		},
	}

	return conf, nil
}

```

这段代码定义了三个变量，分别是：

1. DefaultConnMgrHighWater：表示连接管理器设置为"high water"时的默认值，值为96。
2. DefaultConnMgrLowWater：表示连接管理器设置为"low water"时的默认值，值为32。
3. DefaultConnMgrGracePeriod：表示连接管理器设置为"grace period"时的默认值，单位为时间(以秒为单位)。此值默认为20个时间戳每秒。

这段代码还定义了一个常量DefaultConnMgrType，表示连接管理器的默认类型为"basic"。


```go
// DefaultConnMgrHighWater is the default value for the connection managers
// 'high water' mark.
const DefaultConnMgrHighWater = 96

// DefaultConnMgrLowWater is the default value for the connection managers 'low
// water' mark.
const DefaultConnMgrLowWater = 32

// DefaultConnMgrGracePeriod is the default value for the connection managers
// grace period.
const DefaultConnMgrGracePeriod = time.Second * 20

// DefaultConnMgrType is the default value for the connection managers
// type.
const DefaultConnMgrType = "basic"

```

这段代码定义了一个名为 "addressesConfig" 的函数，返回一个名为 "Addresses" 的结构体，用于配置使接。

函数内部定义了一个叫做 "Swarm" 的切片，包含了一些 IP 地址，用于配置使接。其中，IP 地址中的 "0.0.0.0" 表示所有 IP 地址，也就是所有的主机。IP 地址中的 "::" 表示所有 IPv6 地址，也就是所有的主机。

另外，函数中还定义了 "quic-v1" 和 "webtransport" 这两个地址，这些地址是通过 "quic-v1" 和 "webtransport" 配置回来的。这些地址都是用于配置 TCP 或 UDP 连接的。

函数中还定义了几个常量，分别是：

* DefaultResourceMgrMinInboundConns：这个是一个默认值，用于配置使接的最低带宽连接。它的值为 800。
* addressesConfig：这个函数的名称，也就是函数内部要配置的地址池。


```go
// DefaultResourceMgrMinInboundConns is a MAGIC number that probably a good
// enough number of inbound conns to be a good network citizen.
const DefaultResourceMgrMinInboundConns = 800

func addressesConfig() Addresses {
	return Addresses{
		Swarm: []string{
			"/ip4/0.0.0.0/tcp/4001",
			"/ip6/::/tcp/4001",
			"/ip4/0.0.0.0/udp/4001/quic-v1",
			"/ip4/0.0.0.0/udp/4001/quic-v1/webtransport",
			"/ip6/::/udp/4001/quic-v1",
			"/ip6/::/udp/4001/quic-v1/webtransport",
		},
		Announce:       []string{},
		AppendAnnounce: []string{},
		NoAnnounce:     []string{},
		API:            Strings{"/ip4/127.0.0.1/tcp/5001"},
		Gateway:        Strings{"/ip4/127.0.0.1/tcp/8080"},
	}
}

```

这段代码定义了一个名为`DefaultDatastoreConfig`的内部函数，它返回一个`Datastore`类型的数据存储器实例。

该函数使用了`StorageMax`、`StorageGCWatermark`、`GCPeriod`和`BloomFilterSize`等函数，这些函数用于设置数据存储器的最大容量、 watermark 阈值、 period 和 Bloom Filter 大小。这些设置决定了数据存储器将如何使用存储空间，以及如何决定何时淘汰过时的数据。

此外，该函数还定义了一个名为`badgerSpec`的内部函数，它返回一个包含键为`type`和`prefix`的Map类型的数据。`type`键表示数据存储器的类型，`prefix`键表示数据存储器的名称前缀。`badgerSpec`函数返回的Map类型的数据包含一个名为`badgerds`的键值对，其中包含用于指定`badgerds`数据存储器的数据。

最后，该函数的作用域为`badgerSpec`，返回的`Map[string:]interface{}`类型的数据为`badgerSpec`函数作用域内的所有键的值的Map类型实例。


```go
// DefaultDatastoreConfig is an internal function exported to aid in testing.
func DefaultDatastoreConfig() Datastore {
	return Datastore{
		StorageMax:         "10GB",
		StorageGCWatermark: 90, // 90%
		GCPeriod:           "1h",
		BloomFilterSize:    0,
		Spec:               flatfsSpec(),
	}
}

func badgerSpec() map[string]interface{} {
	return map[string]interface{}{
		"type":   "measure",
		"prefix": "badger.datastore",
		"child": map[string]interface{}{
			"type":       "badgerds",
			"path":       "badgerds",
			"syncWrites": false,
			"truncate":   true,
		},
	}
}

```

该代码定义了一个名为 flatfsSpec 的函数类型，其返回值是一个包含一个名为 "type" 的键，其值为 "mount"，以及一个包含一个名为 "mounts" 的键，其值为一个包含一个名为 "type" 的键，其值为 "measure"，并且包含一个名为 "mountpoint" 的键，其值为 "/blocks"，以及一个名为 "type" 的键，其值为 "measure"，并且包含一个名为 "prefix" 的键，其值为 "flatfs.datastore"，以及一个名为 "child" 的键，其值为一个包含一个名为 "type" 的键，其值为 "flatfs"，以及一个名为 "path" 的键，其值为 "/".

这个函数的作用是返回一个用于将 "flatfs" 挂载点的数据库的配置参数的 map 类型对象.其中，第一个键 "type" 的值为 "mount"，表示将 "flatfs" 挂载到一个数据存储库中，第二个键 "mounts" 的值是一个包含两个 "type" 键，每个键都有一个 "value" 值为 "measure"，这表示该数据存储库支持测量数据存储，第三个键 "mountpoint" 的值设置为 "/blocks"，表示数据存储库的挂载点是数据存储库中的 "blocks" 目录的子目录.第四个键 "type" 的值为 "measure"，表示该数据存储库支持列族数据存储，第五个键 "prefix" 的值设置为 "flatfs.datastore"，表示数据存储库中的数据文件前缀是 "flatfs.datastore".第六个键 "child" 的值是一个包含一个名为 "type" 的键，其值为 "flatfs"，表示该数据存储库的父数据存储库是 "flatfs".第七个键 "path" 的值设置为 "/"，表示数据存储库中的目录根目录是数据存储库的根目录.


```go
func flatfsSpec() map[string]interface{} {
	return map[string]interface{}{
		"type": "mount",
		"mounts": []interface{}{
			map[string]interface{}{
				"mountpoint": "/blocks",
				"type":       "measure",
				"prefix":     "flatfs.datastore",
				"child": map[string]interface{}{
					"type":      "flatfs",
					"path":      "blocks",
					"sync":      true,
					"shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",
				},
			},
			map[string]interface{}{
				"mountpoint": "/",
				"type":       "measure",
				"prefix":     "leveldb.datastore",
				"child": map[string]interface{}{
					"type":        "levelds",
					"path":        "datastore",
					"compression": "none",
				},
			},
		},
	}
}

```

This is a function that generates a RSA or Ed25519 key pair based on the specified settings. The key pair is stored in memory, and an encoded version of the key is stored in the `skbytes` field.

The function first sets the `settings.Algorithm` to the desired key type, and then sets the `settings.Size` or generates it if it's not specified. The `crypto.GenerateKeyPair` function is used to generate the key pair, and the generated `private` key is stored in the `sk` variable. The generated `public` key is stored in the `pk` variable.

Finally, the function checks the generated key type and, if it's not specified, an error is returned. The encoded version of the key is also stored in the `skbytes` field, but that is only done for the `rsa` algorithm, which requires this field to be set to `true`.


```go
// CreateIdentity initializes a new identity.
func CreateIdentity(out io.Writer, opts []options.KeyGenerateOption) (Identity, error) {
	// TODO guard higher up
	ident := Identity{}

	settings, err := options.KeyGenerateOptions(opts...)
	if err != nil {
		return ident, err
	}

	var sk crypto.PrivKey
	var pk crypto.PubKey

	switch settings.Algorithm {
	case "rsa":
		if settings.Size == -1 {
			settings.Size = options.DefaultRSALen
		}

		fmt.Fprintf(out, "generating %d-bit RSA keypair...", settings.Size)

		priv, pub, err := crypto.GenerateKeyPair(crypto.RSA, settings.Size)
		if err != nil {
			return ident, err
		}

		sk = priv
		pk = pub
	case "ed25519":
		if settings.Size != -1 {
			return ident, fmt.Errorf("number of key bits does not apply when using ed25519 keys")
		}
		fmt.Fprintf(out, "generating ED25519 keypair...")
		priv, pub, err := crypto.GenerateEd25519Key(rand.Reader)
		if err != nil {
			return ident, err
		}

		sk = priv
		pk = pub
	default:
		return ident, fmt.Errorf("unrecognized key type: %s", settings.Algorithm)
	}
	fmt.Fprintf(out, "done\n")

	// currently storing key unencrypted. in the future we need to encrypt it.
	// TODO(security)
	skbytes, err := crypto.MarshalPrivateKey(sk)
	if err != nil {
		return ident, err
	}
	ident.PrivKey = base64.StdEncoding.EncodeToString(skbytes)

	id, err := peer.IDFromPublicKey(pk)
	if err != nil {
		return ident, err
	}
	ident.PeerID = id.String()
	fmt.Fprintf(out, "peer identity: %s\n", ident.PeerID)
	return ident, nil
}

```

# `config/init_test.go`

这段代码定义了一个名为 config 的包，其中包含了一些用于测试身份验证创建工具的函数。

首先导入了一些必要的库：

- bytes：用于创建字节缓冲区
- testing：用于用于 testing 的包
- libp2p：用于实现 IPFS 网络的内容，这里用于创建测试
- crypto_pb：用于实现 ed25519 和 rsa 身份验证工具的协议的包

然后定义了一个名为 TestCreateIdentity 的函数，它接收一个由选项 KeyGenerateOption 组成的缓冲区和一个字符串作为输入参数。

这个函数的作用是测试 CreateIdentity 函数的正确性。如果函数在使用时遇到错误，它会在 TestFunction 错误函数中记录错误并输出错误信息。

函数的实现非常简单：

1. 首先，使用 bytes.NewBuffer 创建一个空的字节缓冲区，并设置其类型为 []options.KeyGenerateOption。

2. 如果出现错误，函数会记录错误并跳过该错误。

3. 使用 CreateIdentity 函数创建一个新身份，并将其解码为私人密钥。

4. 然后，使用解码为私钥的 ID 作为输入，再次调用 CreateIdentity 函数，并将其解码为公钥。

5. 最后，检查新身份和公钥的类型是否为期望的类型 (在这里是 ed25519 和 rsa)。

6. 重复 step 3 和 4 两次，确保创建两个不同的身份，并测试 CreateIdentity 函数的正确性。


```go
package config

import (
	"bytes"
	"testing"

	"github.com/ipfs/boxo/coreiface/options"
	crypto_pb "github.com/libp2p/go-libp2p/core/crypto/pb"
)

func TestCreateIdentity(t *testing.T) {
	writer := bytes.NewBuffer(nil)
	id, err := CreateIdentity(writer, []options.KeyGenerateOption{options.Key.Type(options.Ed25519Key)})
	if err != nil {
		t.Fatal(err)
	}
	pk, err := id.DecodePrivateKey("")
	if err != nil {
		t.Fatal(err)
	}
	if pk.Type() != crypto_pb.KeyType_Ed25519 {
		t.Fatal("unexpected type:", pk.Type())
	}

	id, err = CreateIdentity(writer, []options.KeyGenerateOption{options.Key.Type(options.RSAKey)})
	if err != nil {
		t.Fatal(err)
	}
	pk, err = id.DecodePrivateKey("")
	if err != nil {
		t.Fatal(err)
	}
	if pk.Type() != crypto_pb.KeyType_RSA {
		t.Fatal("unexpected type:", pk.Type())
	}
}

```

这段代码是一个名为 `TestCreateIdentityOptions` 的函数，它属于一个名为 `func` 的类型，这个函数的参数是一个测试应用程序的状态 `testing.T` 类型的变量 `t`。

函数的作用是测试 `CreateIdentity` 函数，确保它正确地处理了一个 `bytes.Buffer` 类型的变量 `w`，并且传递了一个包含两个选项的 `options.KeyGenerateOption` 结构体。

具体来说，函数首先创建了一个 `bytes.Buffer` 类型的变量 `w`，然后使用 `CreateIdentity` 函数和传递给它的 `options.KeyGenerateOption` 来设置 `w` 的内容和类型。函数使用 `options.Ed25519Key` 类型来设置第一个选项，指定了一个 2048 位的密钥，并使用 `options.Key.Size(2048)` 来设置第二个选项，指定 `w` 的大小为 2048。

如果函数在执行时遇到错误，它将输出一个错误信息并 `t` 变量，具体错误信息将在 `t` 参数上被预定。


```go
func TestCreateIdentityOptions(t *testing.T) {
	var w bytes.Buffer

	// ed25519 keys with bit size must fail.
	_, err := CreateIdentity(&w, []options.KeyGenerateOption{
		options.Key.Type(options.Ed25519Key),
		options.Key.Size(2048),
	})
	if err == nil {
		t.Errorf("ed25519 keys cannot have a custom bit size")
	}
}

```

# `config/internal.go`

这段代码定义了一个名为`Internal`的结构体类型，它代表了应用程序内部的一个组件。这个组件包含了以下字段：

- `Bitswap`：一个`InternalBitswap`类型的字段。这个字段被标记为`json:`,`omitempty`，这意味着它是一个json格式的字段，并且我们应该忽略它的值，因为我们正在编写自己的代码，所以它的值是空的。
- `UnixFSShardingSizeThreshold`：一个`OptionalString`类型的字段。这个字段被标记为`json:`,`omitempty`，这意味着它是一个json格式的字段，并且我们应该忽略它的值，因为我们正在编写自己的代码，所以它的值是空的。
- `Libp2pForceReachability`：一个`OptionalString`类型的字段。这个字段被标记为`json:`,`omitempty`，这意味着它是一个json格式的字段，并且我们应该忽略它的值，因为我们正在编写自己的代码，所以它的值是空的。
- `BackupBootstrapInterval`：一个`OptionalDuration`类型的字段。这个字段被标记为`json:`,`omitempty`，这意味着它是一个json格式的字段，并且我们应该忽略它的值，因为我们正在编写自己的代码，所以它的值是空的。

此外，还有一段注释。


```go
package config

type Internal struct {
	// All marked as omitempty since we are expecting to make changes to all subcomponents of Internal
	Bitswap                     *InternalBitswap  `json:",omitempty"`
	UnixFSShardingSizeThreshold *OptionalString   `json:",omitempty"`
	Libp2pForceReachability     *OptionalString   `json:",omitempty"`
	BackupBootstrapInterval     *OptionalDuration `json:",omitempty"`
}

type InternalBitswap struct {
	TaskWorkerCount             OptionalInteger
	EngineBlockstoreWorkerCount OptionalInteger
	EngineTaskWorkerCount       OptionalInteger
	MaxOutstandingBytesPerPeer  OptionalInteger
	ProviderSearchDelay         OptionalDuration
}

```

# `config/ipns.go`

这段代码定义了一个名为`Ipns`的结构体，它包含了`RepublishPeriod`和`RecordLifetime`字段，以及一个名为`ResolveCacheSize`的整数字段。同时，它还包含一个名为`UsePubsub`的布尔字段，它的值为`true`，意味着它 Enable namesys-pubsub。

这个结构体的目的是定义一个配置类，用于配置物联网设备中的一个名为`pubsub`的系统。它通过设置`RepublishPeriod`和`RecordLifetime`字段来自定义发布和记录的生命周期。`ResolveCacheSize`字段用于配置缓存的大小，以避免在`pubsub`系统出现故障时，由于缓存导致的多次请求。

`UsePubsub`字段用于配置是否启用`pubsub`系统。当`UsePubsub`为`true`时，它将启用所有的`pubsub`配置选项。


```go
package config

type Ipns struct {
	RepublishPeriod string
	RecordLifetime  string

	ResolveCacheSize int

	// Enable namesys pubsub (--enable-namesys-pubsub)
	UsePubsub Flag `json:",omitempty"`
}

```

# `config/migration.go`

这段代码定义了一个名为 config 的包，其中包含一个名为 DefaultMigrationKeep 的常量，一个名为 DefaultMigrationDownloads 的字符串数组，以及一个名为 Migration 的结构体。

DefaultMigrationKeep 是一个字符串，表示当下载迁移时，如何保存这些下载源。

DefaultMigrationDownloads 是一个字符串数组，表示哪些下载源可以选择。这个数组包含了两个元素，一个元素是 "HTTPS"，另一个元素是 "IPFS"。

Migration 是一个结构体，它定义了如何下载迁移，以及是否在下载完成后将下载的源保存到 IPFS 中。它的字段包括 DownloadSources、Keep 和 Options。

下载源的下载策略可以通过 ProvideOptions 方法进行配置。如果 ProvideOptions 的值为 "cache"，那么下载的源将使用缓存存储；如果 ProvideOptions 的值为 "pin"，那么下载的源将使用 IPFS 存储，而不是使用默认网关；如果 ProvideOptions 的值为空字符串，那么下载的源将使用默认的源。

Migration 的字段包括 DownloadSources、Keep 和 Options。DownloadSources 是用于下载迁移的源，Keep 是用于下载完成后保存迁移的选项，而 Options 是用于下载源的选项。


```go
package config

const DefaultMigrationKeep = "cache"

var DefaultMigrationDownloadSources = []string{"HTTPS", "IPFS"}

// Migration configures how migrations are downloaded and if the downloads are
// added to IPFS locally.
type Migration struct {
	// Sources in order of preference, where "IPFS" means use IPFS and "HTTPS"
	// means use default gateways. Any other values are interpreted as
	// hostnames for custom gateways. Empty list means "use default sources".
	DownloadSources []string
	// Whether or not to keep the migration after downloading it.
	// Options are "discard", "cache", "pin".  Empty string for default.
	Keep string
}

```

# `config/migration_test.go`

这段代码是一个 Go 语言中的 Go-标准库中的测试函数，用于测试一个名为 `Migration` 的结构体。

首先，定义了一个名为 `config` 的包，并导入了 `testing` 和 `encoding/json` 包。

然后，定义了一个名为 `Migration` 的结构体，其中包含以下字段：

* `DownloadSources`：一个字符串数组，包含三个下载源。
* `Keep`：一个字符串，表示是否将下载源缓存到内存中。

接着，如果 `str` 变量为空，则执行以下操作：

* 将 `str` 中的 JSON 字符串解码为 JSON 切片，并将其存储为 `cfg` 结构体中的 `DownloadSources` 字段。
* 检查 `cfg` 结构体中 `DownloadSources` 字段的长度是否为 3。
* 检查 `cfg.DownloadSources` 是否和 `expect` 数组元素完全相同。

最后，如果 `cfg.Keep` 不是 `"cache"`，则执行以下操作：

* 打印错误并退出测试。

整个函数的作用是测试一个名为 `Migration` 的结构体，以验证其是否符合预期。


```go
package config

import (
	"encoding/json"
	"testing"
)

func TestMigrationDecode(t *testing.T) {
	str := `
		{
			"DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
			"Keep": "cache"
		}
	`

	var cfg Migration
	if err := json.Unmarshal([]byte(str), &cfg); err != nil {
		t.Errorf("failed while unmarshalling migration struct: %s", err)
	}

	if len(cfg.DownloadSources) != 3 {
		t.Fatal("wrong number of DownloadSources")
	}
	expect := []string{"IPFS", "HTTP", "127.0.0.1"}
	for i := range expect {
		if cfg.DownloadSources[i] != expect[i] {
			t.Errorf("wrong DownloadSource at %d", i)
		}
	}

	if cfg.Keep != "cache" {
		t.Error("wrong value for Keep")
	}
}

```

# `config/mounts.go`

这段代码定义了一个名为 config 的包，其中包含一个名为 Mounts 的结构体类型。Mounts 结构体包含三个字段，分别表示文件系统的类型、挂载点的类型和允许 FUSE 协议的其他选项。

具体来说，这段代码定义了一个 Mounts 结构体，其中包含以下字段：

1. IPFS：字符串类型的 mount 点，表示以后 IPFS 文件系统类型为主要的文件系统。
2. IPNS：字符串类型的 mount 点，表示以后 IPNS 文件系统类型为主要的文件系统。
3. FuseAllowOther：布尔类型的选项，表示是否允许 FUSE 其他选项。当这个选项为 true 时，允许使用其他 FUSE 选项。

这段代码并没有定义任何函数或其他结构体，也没有输出任何内容。它仅仅定义了一个名为 Mounts 的结构体类型，用于表示文件系统的挂载点选项。


```go
package config

// Mounts stores the (string) mount points.
type Mounts struct {
	IPFS           string
	IPNS           string
	FuseAllowOther bool
}

```

# `config/peering.go`

这段代码定义了一个名为`Peering`的结构体，用于配置网络中的对等连接。该结构体包含一个或多个`peer.AddrInfo`类型的`Peers`字段，用于指定要与哪些对等节点建立连接。

`package config`定义了一个导出该`Peering`结构体的常量。

`import "github.com/libp2p/go-libp2p/core/peer"`导入了一个来自`github.com/libp2p/go-libp2p/core`包的`peer`类型。

`type Peering struct {`定义了一个名为`Peering`的结构体，包含一个或多个`peer.AddrInfo`类型的字段。

`Peers []peer.AddrInfo`定义了一个名为`Peers`的类型，该类型包含一个或多个`peer.AddrInfo`类型的字段，用于指定要与哪些对等节点建立连接。

`}`定义了一个名为`Peering`的结构体，包含一个或多个`peer.AddrInfo`类型的字段，用于指定要与哪些对等节点建立连接。


```go
package config

import "github.com/libp2p/go-libp2p/core/peer"

// Peering configures the peering service.
type Peering struct {
	// Peers lists the nodes to attempt to stay connected with.
	Peers []peer.AddrInfo
}

```

# `config/plugins.go`

这段代码定义了一个名为`config`的包，其中包含一个名为`Plugins`的结构体，它是一个键值对类型的地图，包含一个键`Plugin`类型和一个`Plugin`类型的变量。这里，`Plugin`类型被定义为`type Plugin struct {`的结构体，其中包含一个名为`Disabled`的布尔类型和一个`Config`类型的`interface{}`接口。

`Plugins`结构体还包含一个名为`map[string]Plugin`的键值对类型，它的键是`Plugin`类型的字符串，值是`Plugin`类型的变量。这个键值对类型被用来 map 一些`Plugin`类型的变量到一个字符串上，从而将它们作为键来存储。

该代码段省略了一个名为`TODO: Loader Path?`的注释，它提示您在不安全的代码中不应该包含加载器路径。因此，这个注释并没有对代码的功能产生影响。


```go
package config

type Plugins struct {
	Plugins map[string]Plugin
	// TODO: Loader Path? Leaving that out for now due to security concerns.
}

type Plugin struct {
	Disabled bool
	Config   interface{}
}

```

# `config/profile.go`

这段代码定义了一个名为“config”的包，其中包含了一个名为“Profile”的结构体。这个结构体定义了一个名为“Transformer”的函数，它接收一个名为“Config”的参数，并对它应用一些处理，然后返回一个错误。

同时，这个包也引入了两个名为“fmt”和“net”的包，以及一个名为“time”的包。

这里的作用是定义了一个名为“Transformer”的函数类型，这个类型有一个名为“Profile”的接口，这个接口包含一个名为“Description”的属性和一个名为“Transform”的函数。这个“Profile”接口的实现者（定义者）需要实现这个接口，然后就可以定义一个具体的“Transform”函数，这个函数的实现就是对传入的“Config”进行一些处理。

然后，定义了一个名为“Profile”的结构体，这个结构体包含一个名为“Description”的属性和一个名为“Transform”的函数，还有一个名为“InitOnly”的布尔属性和一个通用的“Transform”函数。这个结构体表示了一个配置文件中的一个配置项，这个配置项可以被任何具体的“Profile”函数所应用，但是这个配置项只能在“InitOnly”为真时被应用。


```go
package config

import (
	"fmt"
	"net"
	"time"
)

// Transformer is a function which takes configuration and applies some filter to it.
type Transformer func(c *Config) error

// Profile contains the profile transformer the description of the profile.
type Profile struct {
	// Description briefly describes the functionality of the profile.
	Description string

	// Transform takes ipfs configuration and applies the profile to it.
	Transform Transformer

	// InitOnly specifies that this profile can only be applied on init.
	InitOnly bool
}

```

The answer to your question is correct. According to the documents you provided, `filters` is a list of IPv4 and IPv6 prefixes that are considered private, local only, or unrouteable.

The `/ip4/` prefixes are defined as IPv4 unicast routes that are not intended for distribution to end-users. These routes are commonly referred to as "filter routes".

The `/ip6/` prefixes are defined as IPv6 unicast routes that are not intended for distribution to end-users. These routes are commonly referred to as "mover routes".

The `/ip4/10.*.*.*/` to `/ip4/192.168.*.*.*/` prefixes are defined as local-only routes and are intended for use by local networks only.

The `/ip4/172.16.*.*.*/` to `/ip4/192.168.*.*.*/` prefixes are defined as unrouteable routes and should not be分配给任何路由器学习。

The `/ip6/100.*.*.*/` to `/ip6/2001:2.*.*.*/` prefixes are defined as move routes and are intended for use by mobile devices moving between different connectivity sources.

The `/ip6/2001:db8.*.*.*/` to `/ip6/2001:db8::/` prefixes are defined as multicast routes and should not be distributed to any application or service.

The `/ip6/fc00::/` prefix is defined as a global unicast route that is not intended for distribution to end-users.

The `/ip6/fe80::/` prefix is defined as a multicast route that is intended for use by devices using Zigbee and other low-power, low-data-rate communications protocols.

The `/ip6/fe80:0:0:0:0:0/` prefix is defined as a one-to-one, global, unicast route that is intended for use by devices using Zigbee and other low-power, low-data-rate communications protocols.


```go
// defaultServerFilters has is a list of IPv4 and IPv6 prefixes that are private, local only, or unrouteable.
// according to https://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml
// and https://www.iana.org/assignments/iana-ipv6-special-registry/iana-ipv6-special-registry.xhtml
var defaultServerFilters = []string{
	"/ip4/10.0.0.0/ipcidr/8",
	"/ip4/100.64.0.0/ipcidr/10",
	"/ip4/169.254.0.0/ipcidr/16",
	"/ip4/172.16.0.0/ipcidr/12",
	"/ip4/192.0.0.0/ipcidr/24",
	"/ip4/192.0.2.0/ipcidr/24",
	"/ip4/192.168.0.0/ipcidr/16",
	"/ip4/198.18.0.0/ipcidr/15",
	"/ip4/198.51.100.0/ipcidr/24",
	"/ip4/203.0.113.0/ipcidr/24",
	"/ip4/240.0.0.0/ipcidr/4",
	"/ip6/100::/ipcidr/64",
	"/ip6/2001:2::/ipcidr/48",
	"/ip6/2001:db8::/ipcidr/32",
	"/ip6/fc00::/ipcidr/7",
	"/ip6/fe80::/ipcidr/10",
}

```

// This code defines two maps, Profiles and LocalDiscovery, which hold the configuration transformers.

// The Profiles map has two entries, one for "server" and one for "local-discovery".
// The server entry has a Transform function that disables local host discovery and adds a list of default server filters to the
addresses.NoAnnounce setting.

// The local-discovery entry has a Transform function that sets default values for the
description, discovery.mdns.enabled, and swarm.disableNatPortMap settings.

// The transforms are applied to the respective Profiles settings.
//
// var Profiles is a map holding configuration transformers.
// Docs are in docs/config.md.
var Profiles = map[string]Profile{
	"server": {
		Description: `Disables local host discovery, recommended when
running IPFS on machines with public IPv4 addresses.`,

		Transform: func(c *Config) error {
			c.Addresses.NoAnnounce = appendSingle(c.Addresses.NoAnnounce, defaultServerFilters)
			c.Swarm.AddrFilters = appendSingle(c.Swarm.AddrFilters, defaultServerFilters)
			c.Discovery.MDNS.Enabled = false
			c.Swarm.DisableNatPortMap = true
			return nil
		},
	},

	"local-discovery": {
		Description: `Sets default values to fields affected by the server
		Transform: func(c *Config) error {
			c.Description = defaultLocalDiscoveryDescription
			c.Discovery.MDNS.Enabled = true
			return nil
		},
	},
}


```go
// Profiles is a map holding configuration transformers. Docs are in docs/config.md.
var Profiles = map[string]Profile{
	"server": {
		Description: `Disables local host discovery, recommended when
running IPFS on machines with public IPv4 addresses.`,

		Transform: func(c *Config) error {
			c.Addresses.NoAnnounce = appendSingle(c.Addresses.NoAnnounce, defaultServerFilters)
			c.Swarm.AddrFilters = appendSingle(c.Swarm.AddrFilters, defaultServerFilters)
			c.Discovery.MDNS.Enabled = false
			c.Swarm.DisableNatPortMap = true
			return nil
		},
	},

	"local-discovery": {
		Description: `Sets default values to fields affected by the server
```

This code is a Go package that provides a configuration-related interface for enabling or disabling various network settings of the IPFS (InterPlanetary File System) daemon.

When a new configuration is being created or updated, the `profile` section is checked. If the `enables Discovery in local networks` setting is enabled, the code will apply the necessary changes to the configuration.

The `Transform` function is used to modify the `Config` struct. The `NoAnnounce` address filter is deleted, and the `AddrFilters` address filter is also deleted, effectively disabling the default server filters. The `MDNS.Enabled` setting is set to `true`, and the `Swarm.DisableNatPortMap` setting is set to `false`, allowing the daemon to use the local IP address for NAT port mapping.

In the `test` section, a transformation is applied to the configuration when a new configuration is being created or updated. The API address is set to `"/ip4/127.0.0.1/tcp/0"`. The `Gateway` address is also set to `"/ip4/127.0.0.1/tcp/0"`. The `Swarm` address is set to `[]string{"/ip4/127.0.0.1/tcp/0"}`. The `Discovery.MDNS.Enabled` setting is set to `false`.

In the `default-networking` section, the default settings for the network are restored. This means that the IPFS daemon will not use any of the default settings for the network, such as the local IP address for NAT port mapping.


```go
profile, enables discovery in local networks.`,

		Transform: func(c *Config) error {
			c.Addresses.NoAnnounce = deleteEntries(c.Addresses.NoAnnounce, defaultServerFilters)
			c.Swarm.AddrFilters = deleteEntries(c.Swarm.AddrFilters, defaultServerFilters)
			c.Discovery.MDNS.Enabled = true
			c.Swarm.DisableNatPortMap = false
			return nil
		},
	},
	"test": {
		Description: `Reduces external interference of IPFS daemon, this
is useful when using the daemon in test environments.`,

		Transform: func(c *Config) error {
			c.Addresses.API = Strings{"/ip4/127.0.0.1/tcp/0"}
			c.Addresses.Gateway = Strings{"/ip4/127.0.0.1/tcp/0"}
			c.Addresses.Swarm = []string{
				"/ip4/127.0.0.1/tcp/0",
			}

			c.Swarm.DisableNatPortMap = true

			c.Bootstrap = []string{}
			c.Discovery.MDNS.Enabled = false
			return nil
		},
	},
	"default-networking": {
		Description: `Restores default network settings.
```

这段代码是一个 Go 语言中的函数，它描述了一个测试程序的逆向剖分报告。

具体来说，这段代码实现了以下功能：

1. 根据输入的 `Config` 对象，将其 `Addresses` 字段设置为一个包含多个 IP 地址的哈希表。
2. 调用 `DefaultBootstrapPeers` 函数获取一个或多个默认网关对等体，并将它们添加到 `Bootstrap` 字段中。
3. 将 `Swarm.DisableNatPortMap` 字段设置为 `false`，以禁用发现过程中使用的端口映射。
4. 将 `Discovery.MDNS.Enabled` 字段设置为 `true`，以启用键值 DNS 服务。

这个函数的作用是设置一个测试程序的逆向剖分报告，用于报告程序在启动时使用的默认网关对等体和键值 DNS 服务的设置。


```go
Inverse profile of the test profile.`,

		Transform: func(c *Config) error {
			c.Addresses = addressesConfig()

			bootstrapPeers, err := DefaultBootstrapPeers()
			if err != nil {
				return err
			}
			c.Bootstrap = appendSingle(c.Bootstrap, BootstrapPeerStrings(bootstrapPeers))

			c.Swarm.DisableNatPortMap = false
			c.Discovery.MDNS.Enabled = true
			return nil
		},
	},
	"default-datastore": {
		Description: `Configures the node to use the default datastore (flatfs).

```

这段代码的作用是初始化一个名为"flatfs"的datastore，当创建节点时，只允许在节点初始化时应用此配置，然后将datastore的配置设置为flatfsSpec()，实现对datastore的初始化。Transform函数确保flatfsSpec()设置正确，并返回一个nil值，从而避免在运行时抛出错误。


```go
Read the "flatfs" profile description for more information on this datastore.

This profile may only be applied when first initializing the node.
`,

		InitOnly: true,
		Transform: func(c *Config) error {
			c.Datastore.Spec = flatfsSpec()
			return nil
		},
	},
	"flatfs": {
		Description: `Configures the node to use the flatfs datastore.

This is the most battle-tested and reliable datastore. 
```

This code defines a `badgerds` profile in a `config.yaml` file for configuring the behavior of the node.

The `badgerds` profile is used to configure the node to use the experimental badger datastore. The datastore is designed to be simple, reliable, and minimize memory usage, as per the first condition.

The second condition requires the node to run garbage collection in a way that reclaims free space as soon as possible. This is achieved by the node maintaining a close eye on the amount of available memory and using it efficiently when it needs to garbage collect.

The third condition states that the node is fine with the default speed of data import, or prefer to use `--nocopy`. This suggests that the node should not perform any additional processing on the imported data and should simply copy it, which minimizes the amount of memory used.

The `InitOnly` condition is set to `true`, which means that the profile should only be applied when the node is first initialized.


```go
You should use this datastore if:

* You need a very simple and very reliable datastore, and you trust your
  filesystem. This datastore stores each block as a separate file in the
  underlying filesystem so it's unlikely to loose data unless there's an issue
  with the underlying file system.
* You need to run garbage collection in a way that reclaims free space as soon as possible.
* You want to minimize memory usage.
* You are ok with the default speed of data import, or prefer to use --nocopy.

This profile may only be applied when first initializing the node.
`,

		InitOnly: true,
		Transform: func(c *Config) error {
			c.Datastore.Spec = flatfsSpec()
			return nil
		},
	},
	"badgerds": {
		Description: `Configures the node to use the experimental badger datastore.

```

 Lowpower 数据存储库配置。这个数据存储库主要用于那些对性能，特别是添加大量文件的速度至关重要的场景。然而，请注意：

* 当数据存储库的存储空间小于几个吉字节时，这个数据存储库不会正确回收空间。如果您使用 IPFS 并启用垃圾回收（--enable-gc），您计划在您的 IPFS 节点上存储非常少的数据，而磁盘利用率比性能更重要，那么请考虑使用 flatfs。
* 这个数据存储库可能需要使用多达几个吉字节的多内存。这对于中等大小的数据存储库是没问题的，但如果您的数据集超过了这个范围，您可能会遇到性能问题。
* 当前的实现是基于旧的 Badger 1.x版本，而该版本已不再得到上游团队的支持。

这个配置文件仅在节点初始化时应用。


```go
Use this datastore if some aspects of performance, 
especially the speed of adding many gigabytes of files, are critical.
However, be aware that:

* This datastore will not properly reclaim space when your datastore is
  smaller than several gigabytes.  If you run IPFS with --enable-gc, you plan
  on storing very little data in your IPFS node, and disk usage is more
  critical than performance, consider using flatfs.
* This datastore uses up to several gigabytes of memory.  
* Good for medium-size datastores, but may run into performance issues
  if your dataset is bigger than a terabyte.
* The current implementation is based on old badger 1.x
  which is no longer supported by the upstream team.

This profile may only be applied when first initializing the node.`,

		InitOnly: true,
		Transform: func(c *Config) error {
			c.Datastore.Spec = badgerSpec()
			return nil
		},
	},
	"lowpower": {
		Description: `Reduces daemon overhead on the system. May affect node
```

这段代码定义了一个名为 `functionality` 的函数，它与内容发现和数据下载有关。函数的作用是优化性能，但可能会影响内容发现。

首先，它设置了一些参数，包括路由类型、自动自适应网络服务模式、克隆器间隔和风暴的连接管理器类型。这些参数的设置可能会影响内容发现和下载的性能。

然后，它定义了一个名为 `randomports` 的配置项。当使用 `randomports` 时，它会使用随机端口进行风暴连接，以避免使用静态端口导致的内容重复问题。

最后，函数的实现包含在两个可选的函数中：`Transform: func(c *Config) error` 和 `"randomports": {...}`。第一个函数定义了内容发现的参数，而第二个函数定义了使用随机端口的参数。


```go
functionality - performance of content discovery and data
fetching may be degraded.
`,
		Transform: func(c *Config) error {
			c.Routing.Type = NewOptionalString("dhtclient") // TODO: https://github.com/ipfs/kubo/issues/9480
			c.AutoNAT.ServiceMode = AutoNATServiceDisabled
			c.Reprovider.Interval = NewOptionalDuration(0)

			lowWater := int64(20)
			highWater := int64(40)
			gracePeriod := time.Minute
			c.Swarm.ConnMgr.Type = NewOptionalString("basic")
			c.Swarm.ConnMgr.LowWater = &OptionalInteger{value: &lowWater}
			c.Swarm.ConnMgr.HighWater = &OptionalInteger{value: &highWater}
			c.Swarm.ConnMgr.GracePeriod = &OptionalDuration{&gracePeriod}
			return nil
		},
	},
	"randomports": {
		Description: `Use a random port number for swarm.`,

		Transform: func(c *Config) error {
			port, err := getAvailablePort()
			if err != nil {
				return err
			}
			c.Addresses.Swarm = []string{
				fmt.Sprintf("/ip4/0.0.0.0/tcp/%d", port),
				fmt.Sprintf("/ip6/::/tcp/%d", port),
			}
			return nil
		},
	},
}

```

这两段代码的功能如下：

1. getAvailablePort()函数的作用是获取一个可用的TCP端口号，并返回它。函数首先通过调用net.Listen函数，指定使用TCP协议，并监听所有接口，同时指定端口为0。如果函数执行成功，则返回一个可用的TCP端口号，否则返回一个错误。

2. appendSingle函数的作用是将两个输入的字符串a和b合并，并将它们存储在一个新的字符串out中。函数会遍历a和b中的所有字符串，并检查它们是否存在于一个名为m的map中。如果它们不在map中，则将它们添加到out中。如果它们已经在map中，但需要更新map以包含它们，则m中相应的字段将被更新为true。最后，函数返回out。


```go
func getAvailablePort() (port int, err error) {
	ln, err := net.Listen("tcp", "[::]:0")
	if err != nil {
		return 0, err
	}
	defer ln.Close()
	port = ln.Addr().(*net.TCPAddr).Port
	return port, nil
}

func appendSingle(a []string, b []string) []string {
	out := make([]string, 0, len(a)+len(b))
	m := map[string]bool{}
	for _, f := range a {
		if !m[f] {
			out = append(out, f)
		}
		m[f] = true
	}
	for _, f := range b {
		if !m[f] {
			out = append(out, f)
		}
		m[f] = true
	}
	return out
}

```

此代码定义了两个函数：`func deleteEntries(arr []string, del []string) []string` 和 `func mapKeys(m map[string]struct{}) []string`。

函数 `deleteEntries` 接收两个参数：一个字符串数组 `arr` 和一个字符串数组 `del`。函数的作用是创建一个空字符串数组，并在其中删除给定数组中的所有元素，前提是这些元素在映射 `arr` 中存在。函数的实现使用了两个嵌套的循环，第一个循环遍历 `arr`，第二个循环遍历 `del`。在内部，使用 `map[string]struct{}` 存储每个元素的键值对，并为每个元素设置一个新键值对。然后，使用 `delete` 函数从 `map` 中删除给定键的元素。最后，函数返回一个新数组，其中包含映射 `arr` 中所有元素的键。

函数 `mapKeys` 接收一个 `map[string]struct{}` 类型的参数 `m`。函数的作用是返回一个字符串数组，其中包含所有键，这些键属于 `map` 中的所有元素。函数的实现类似于 `map.String` 函数，但会删除其中的键值对。具体实现是，遍历 `map` 中的所有键，将键存储在一个新数组中，并返回新数组。


```go
func deleteEntries(arr []string, del []string) []string {
	m := map[string]struct{}{}
	for _, f := range arr {
		m[f] = struct{}{}
	}
	for _, f := range del {
		delete(m, f)
	}
	return mapKeys(m)
}

func mapKeys(m map[string]struct{}) []string {
	out := make([]string, 0, len(m))
	for f := range m {
		out = append(out, f)
	}
	return out
}

```

# `config/provider.go`

这段代码定义了一个名为`Provider`的结构体类型，它包含一个名为`Strategy`的整型字段，用于指定在暴露服务时应该宣告哪些策略。

`package config`定义了这个结构体类型的名称，以及它所在的命名空间。

这个结构体类型的实例可以用来创建一个`Provider`对象，这个对象可以用来访问一个键值对，包含键的策略名称和键的描述。通过创建一个`Provider`实例并设置其`Strategy`字段的值，可以指定在暴露服务时应该宣告哪些策略。例如，如果将`Provider`实例设置为`Provider{Strategy: "whitelist", Description: "Exclusive access to the protected resource."}`，那么当暴露服务时，它将只宣告键名为`"whitelist"`的策略，并将其描述为`"Exclusive access to the protected resource."`。


```go
package config

type Provider struct {
	Strategy string // Which keys to announce
}

```

# `config/pubsub.go`

这段代码定义了一个名为 config 的包，其中包含两个字符串类型的变量 LastSeenMessagesStrategy 和 FirstSeenMessagesStrategy，它们都表示消息TTL策略的类型。LastSeenMessagesStrategy 表示基于最后一次看到消息的时间来计算TTL数量，而 FirstSeenMessagesStrategy 表示基于消息第一次被看到的时间来计算TTL数量。DefaultSeenMessagesStrategy 是如果没有特定的策略被指定，则使用 LastSeenMessagesStrategy 的策略，这个策略基于上次看到消息的时间来计算TTL数量。


```go
package config

const (
	// LastSeenMessagesStrategy is a strategy that calculates the TTL countdown
	// based on the last time a Pubsub message is seen. This means that if a message
	// is received and then seen again within the specified TTL window, it
	// won't be emitted until the TTL countdown expires from the last time the
	// message was seen.
	LastSeenMessagesStrategy = "last-seen"

	// FirstSeenMessagesStrategy is a strategy that calculates the TTL
	// countdown based on the first time a Pubsub message is seen. This means that if
	// a message is received and then seen again within the specified TTL
	// window, it won't be emitted.
	FirstSeenMessagesStrategy = "first-seen"

	// DefaultSeenMessagesStrategy is the strategy that is used by default if
	// no Pubsub.SeenMessagesStrategy is specified.
	DefaultSeenMessagesStrategy = LastSeenMessagesStrategy
)

```

该代码定义了一个名为 `PubsubConfig` 的结构体，用于配置在启用 pubsub 发布时使用的组件。

该结构体包含了以下字段：

- `Router`：路由，可以是流式订阅（floodsub）或广播订阅（gossipsub）中的任何一种。
- `DisableSigning`：是否禁用消息签名。默认值为 false。
- `Enabled`：是否启用 pubsub 发布。默认值为 false。
- `SeenMessagesTTL`：可见消息超时时间，用于限制在同一消息中出现的时间窗口。
- `SeenMessagesStrategy`：用于计算时间戳降幂的设置。

该结构体的 `json` 字段定义了如何使用 JSON 数据序列化或反序列化它。


```go
type PubsubConfig struct {
	// Router can be either floodsub (legacy) or gossipsub (new and
	// backwards compatible).
	Router string

	// DisableSigning disables message signing. Message signing is *enabled*
	// by default.
	DisableSigning bool

	// Enable pubsub (--enable-pubsub-experiment)
	Enabled Flag `json:",omitempty"`

	// SeenMessagesTTL is a value that controls the time window within which
	// duplicate messages will be identified and won't be emitted.
	SeenMessagesTTL *OptionalDuration `json:",omitempty"`

	// SeenMessagesStrategy is a setting that determines how the time-to-live
	// (TTL) countdown for deduplicating messages is calculated.
	SeenMessagesStrategy *OptionalString `json:",omitempty"`
}

```

# `config/remotepin.go`

这段代码定义了一个名为`config`的包，其中包含以下变量：

- `RemoteServicesPath`变量表示用于存储远程服务访问路径的变量。
- `PinningConcealSelector`变量表示用于选择隐藏Pinning服务中某些服务的选项的字符串数组，这些服务使用`Pinning`作为其选项的一部分。

接着，定义了一个名为`Pinning`的结构体，它包含一个键值对`RemoteServices`的`map`字段，用于存储远程服务映射。

然后，定义了一个名为`RemotePinningService`的结构体，它包含一个名为`API`的`map`字段，用于存储远程 Pinning 服务的 API，以及一个名为`Policies`的`map`字段，用于存储远程 Pinning 服务的策略。

最后，通过创建一个名为`config`的包来定义这些结构体，并为这些变量提供默认值。


```go
package config

var (
	RemoteServicesPath     = "Pinning.RemoteServices"
	PinningConcealSelector = []string{"Pinning", "RemoteServices", "*", "API", "Key"}
)

type Pinning struct {
	RemoteServices map[string]RemotePinningService
}

type RemotePinningService struct {
	API      RemotePinningServiceAPI
	Policies RemotePinningServicePolicies
}

```

这段代码定义了一个名为RemotePinningServiceAPI的结构体，它包含一个Endpoint和一个Key。

接下来定义了一个名为RemotePinningServicePolicies的结构体，它包含一个MFSRemotePinningServiceMFSPolicy变量。

然后定义了一个名为RemotePinningServiceMFSPolicy的结构体，它包含一个Enable字段和一个PinName字段，以及一个RepinInterval字段。

接着，在RemotePinningServiceAPI中声明了一个靜態成员函数getPolicy，它的第一个参数是一个MFSRemotePinningServiceMFSPolicy类型的变量。

最后，在RemotePinningServiceAPI的定义中，定义了Endpoint字段和Key字段，它们都是该结构体所需的成员变量。


```go
type RemotePinningServiceAPI struct {
	Endpoint string
	Key      string
}

type RemotePinningServicePolicies struct {
	MFS RemotePinningServiceMFSPolicy
}

type RemotePinningServiceMFSPolicy struct {
	// Enable enables watching for changes in MFS and re-pinning the MFS root cid whenever a change occurs.
	Enable bool
	// Name is the pin name for MFS.
	PinName string
	// RepinInterval determines the repin interval when the policy is enabled. In ns, us, ms, s, m, h.
	RepinInterval string
}

```

# `config/reprovider.go`

这段代码定义了一个名为 `Reprovider` 的结构体，它用于设置本地存储的对象何时将其复制到网络中。

首先，它导入了 `time` 和 `optional` 包。`time` 包提供了 `Hour` 类型的时间段，用于设置间隔。`optional` 包则允许我们使用 `Optional` 类型来定义一个非空值，如果该值不存在，则不会输出错误。

接着，它定义了一个名为 `DefaultReproviderInterval` 的变量，它表示将本地存储的对象复制到网络中的间隔时间。`DefaultReproviderStrategy` 变量则定义了我们应该使用哪种策略来通知其他人本地存储的对象已经复制到网络中了。

最后，它定义了一个名为 `Reprovider` 的结构体类型，它包含一个名为 `Interval` 的字段，它是一个可选的 `Duration` 类型，用于设置将本地存储的对象复制到网络中的时间间隔。还有一个名为 `Strategy` 的字段，它是一个可选的 `String` 类型，用于设置我们应该使用哪种策略来通知其他人本地存储的对象已经复制到网络中了。

该代码定义了一个 `Reprovider` 结构体类型，用于设置本地存储的对象何时将其复制到网络中。我们可以创建一个 `Reprovider` 实例，并使用它的 `Interval` 和 `Strategy` 字段来设置本地存储的对象的复制策略。


```go
package config

import "time"

const (
	DefaultReproviderInterval = time.Hour * 22 // https://github.com/ipfs/kubo/pull/9326
	DefaultReproviderStrategy = "all"
)

type Reprovider struct {
	Interval *OptionalDuration `json:",omitempty"` // Time period to reprovide locally stored objects to the network
	Strategy *OptionalString   `json:",omitempty"` // Which keys to announce
}

```

# `config/routing.go`

这段代码定义了一个名为 `config` 的包，其中包含一个名为 `Routing` 的结构体，它定义了在 libp2p 路由中使用的配置选项。

具体来说，这个结构体包含以下字段：

- `Type`：设置默认的 daemon 路由模式，可以是多选题，包括 "auto"、"autoclient"、"dht"、"dhtclient"、"dhtserver" 和 "none"。如果没有设置，或者设置为 "auto"，则使用 DHT 和隐含路由器。当设置为 "custom" 时，使用用户提供的路由器。

- `AcceleratedDHTClient`：设置加速 DHT 客户端是否启用。

- `Routers`：一个包含路由器名称的字符串数组，用于设置路由器。

- `Methods`：一个包含方法名称的字符串数组，用于设置路由器方法。

该 `config` 包的作用是定义了 libp2p 路由器使用的配置选项，从而使开发者能够更轻松地设置路由器，并选择如何使用加速 DHT。


```go
package config

import (
	"encoding/json"
	"fmt"
	"runtime"
)

// Routing defines configuration options for libp2p routing.
type Routing struct {
	// Type sets default daemon routing mode.
	//
	// Can be one of "auto", "autoclient", "dht", "dhtclient", "dhtserver", "none", or "custom".
	// When unset or set to "auto", DHT and implicit routers are used.
	// When "custom" is set, user-provided Routing.Routers is used.
	Type *OptionalString `json:",omitempty"`

	AcceleratedDHTClient bool

	Routers Routers

	Methods Methods
}

```

这段代码定义了一个名为 "Router" 的 struct，它表示一个 HTTP 路由器。该 struct 包含一个类型 "RouterType"，用于标识路由器类型。另外，该 struct 还包含一个 "Parameters" 字段，表示额外的配置信息，比如路由器的端点类型、URL 参数、请求方法等。

该 struct 还定义了一个名为 "Methods" 的 map，它用于存储 HTTP 路由器中的路由器方法。该 map 包含了所有方法名称列表中定义的方法名称，以及对应的请求方法。

最后，该 struct 还定义了一个名为 "Routers" 的 map，它用于存储 HTTP 路由器中的路由器配置信息。

该 struct 的主要作用是定义一个 HTTP 路由器，并管理一个包含多个 HTTP 路由器的方法的集合。通过 "Methods" 中的键值对，可以指定路由器要执行的 HTTP 方法，而通过 "Routers" 中的键值对，可以设置路由器的参数。


```go
type Router struct {
	// Router type ID. See RouterType for more info.
	Type RouterType

	// Parameters are extra configuration that this router might need.
	// A common one for HTTP router is "Endpoint".
	Parameters interface{}
}

type (
	Routers map[string]RouterParser
	Methods map[MethodName]Method
)

func (m Methods) Check() error {
	// Check supported methods
	for _, mn := range MethodNameList {
		_, ok := m[mn]
		if !ok {
			return fmt.Errorf("method name %q is missing from Routing.Methods config param", mn)
		}
	}

	// Check unsupported methods
	for k := range m {
		seen := false
		for _, mn := range MethodNameList {
			if mn == k {
				seen = true
				break
			}
		}

		if seen {
			continue
		}

		return fmt.Errorf("method name %q is not a supported method on Routing.Methods config param", k)
	}

	return nil
}

```

该代码定义了一个名为 `RouterParser` 的结构体，它包含一个名为 `UnmarshalJSON` 的方法。

该方法的参数 `b` 是一个字节切片（[]byte），它用于将路由器解析器中的参数和其对应的 JSON 数据进行反序列化。

在 `UnmarshalJSON` 方法中，首先创建一个名为 `out` 的切片，该切片将存储从 `Router` 类型中获取到的参数。然后，使用 `json.Unmarshal` 函数将传入的 `b` 字节切片序列化为一个 JSON 数据结构。

接着，从 JSON 数据结构中获取参数类型，并使用该类型创建一个名为 `p` 的切片，该切片将存储解析后的参数。然后，根据路由器解析器获取到的参数类型，将 `p` 切片中的值复制到 `out.Parameters` 字段中，从而将参数复制到 `Router` 实例中。

最后，由于在序列化过程中可能出现错误，可能会导致有些参数被丢失，因此在实际应用中需要进行适当的错误处理。


```go
type RouterParser struct {
	Router
}

func (r *RouterParser) UnmarshalJSON(b []byte) error {
	out := Router{}
	out.Parameters = &json.RawMessage{}
	if err := json.Unmarshal(b, &out); err != nil {
		return err
	}
	raw := out.Parameters.(*json.RawMessage)

	var p interface{}
	switch out.Type {
	case RouterTypeHTTP:
		p = &HTTPRouterParams{}
	case RouterTypeDHT:
		p = &DHTRouterParams{}
	case RouterTypeSequential:
		p = &ComposableRouterParams{}
	case RouterTypeParallel:
		p = &ComposableRouterParams{}
	}

	if err := json.Unmarshal(*raw, &p); err != nil {
		return err
	}

	r.Router.Type = out.Type
	r.Router.Parameters = p

	return nil
}

```

这段代码定义了一个名为 `RouterType` 的枚举类型，它用于表示不同的路由类型。枚举类型 `RouterType` 的取值包括 `RouterTypeHTTP`、`RouterTypeDHT`、`RouterTypeSequential` 和 `RouterTypeParallel`。这些枚举类型分别表示不同的路由模式，如 HTTP、DHT、序列化和并行路由等。

接下来，定义了一个名为 `DHTMode` 的枚举类型，它用于表示不同的 DHT 模式，包括 `DHTModeServer`、`DHTModeClient` 和 `DHTModeAuto`。

接着，定义了一个名为 `RouterTypeConfig` 的结构体，它用于在运行时获取 `RouterType` 和 `DHTMode` 的组合。这个结构体包含两个方法：`RouterType()` 和 `DHTMode()`。通过调用这两层函数，我们可以实例化不同的路由和 DHT 模式。

最后，定义了一个名为 `Router` 的函数，它接受一个路由类型和 DHT 模式作为参数。这个函数会根据这些设置返回一个具体的路由实例，我们可以通过调用 `Router.Execute()` 来执行路由操作。


```go
// Type is the routing type.
// Depending of the type we need to instantiate different Routing implementations.
type RouterType string

const (
	RouterTypeHTTP       RouterType = "http"       // HTTP JSON API for delegated routing systems (IPIP-337).
	RouterTypeDHT        RouterType = "dht"        // DHT router.
	RouterTypeSequential RouterType = "sequential" // Router helper to execute several routers sequentially.
	RouterTypeParallel   RouterType = "parallel"   // Router helper to execute several routers in parallel.
)

type DHTMode string

const (
	DHTModeServer DHTMode = "server"
	DHTModeClient DHTMode = "client"
	DHTModeAuto   DHTMode = "auto"
)

```

这段代码定义了一个名为HTTPRouterParams的结构体，用于表示HTTPRouter的参数。

HTTPRouterParams包含以下字段：

* Endpoint：路由的起点，表示将请求发送到这个URL获取信息。
* MaxProvideBatchSize：用于防止在每次请求中发送超过MaxProvideBatchSize个CID。
* Servers：这个字段指定了要使用哪些服务器来提供信息，但是它本身并不会被发送。
* MaxProvideConcurrency：限制在每次提供内容时使用的线程数，使用默认值20。

另外，还定义了一个名为MethodName的一维字符数组，用于表示HTTPRouter中可用的方法名称。


```go
type MethodName string

const (
	MethodNameProvide       MethodName = "provide"
	MethodNameFindProviders MethodName = "find-providers"
	MethodNameFindPeers     MethodName = "find-peers"
	MethodNameGetIPNS       MethodName = "get-ipns"
	MethodNamePutIPNS       MethodName = "put-ipns"
)

var MethodNameList = []MethodName{MethodNameProvide, MethodNameFindPeers, MethodNameFindProviders, MethodNameGetIPNS, MethodNamePutIPNS}

type HTTPRouterParams struct {
	// Endpoint is the URL where the routing implementation will point to get the information.
	Endpoint string

	// MaxProvideBatchSize determines the maximum amount of CIDs sent per batch.
	// Servers might not accept more than 100 elements per batch. 100 elements by default.
	MaxProvideBatchSize int

	// MaxProvideConcurrency determines the number of threads used when providing content. GOMAXPROCS by default.
	MaxProvideConcurrency int
}

```

此代码定义了一个名为HTTPRouterParams的函数类型，以及一个名为DHTRouterParams的结构体类型。函数类型使用了一个名为hrp的指针变量，该指针变量接收一个HTTPRouterParams类型的参数。

函数FillDefaults()函数接收一个空指针参数，然后执行以下操作：

1. 如果hrp.MaxProvideBatchSize的值为0，则将hrp.MaxProvideBatchSize的值设为100。
2. 如果hrp.MaxProvideConcurrency的值为0，则将hrp.MaxProvideConcurrency的值设置为 runtime.GOMAXPROCS(0)，其中runtime.GOMAXPROCS函数返回0表示不会限制并发连接数。

这个函数的作用是设置HTTPRouterParams类型的参数以使用DHT（分布式哈希表）协议，以便在默认情况下正确处理DHT路由参数。


```go
func (hrp *HTTPRouterParams) FillDefaults() {
	if hrp.MaxProvideBatchSize == 0 {
		hrp.MaxProvideBatchSize = 100
	}

	if hrp.MaxProvideConcurrency == 0 {
		hrp.MaxProvideConcurrency = runtime.GOMAXPROCS(0)
	}
}

type DHTRouterParams struct {
	Mode                 DHTMode
	AcceleratedDHTClient bool `json:",omitempty"`
	PublicIPNetwork      bool
}

```

这段代码定义了一个名为 `ComposableRouterParams` 的结构体，它表示一个路由参数类，包含了路由器和超时时间等参数。

`Routers` 字段是一个包含多个 `ConfigRouter` 实例的数组，每个 `ConfigRouter` 实例包含一个路由名称 (`RouterName`) 和一个超时时间 (`Timeout`)。

`Timeout` 字段是一个可选的 `OptionalDuration` 类型，表示路由器超时的时间，如果超时时间内发生错误，则关闭路由器。

`ExecuteAfter` 字段是一个可选的 `OptionalDuration` 类型，表示路由器在执行业务后需要等待的时间，如果 `ExecuteAfter` 的值为 `nil`，则表示不需要等待。

`Method` 字段是一个包含路由名称的 `Method` 类型，通常在路由处理程序中使用。


```go
type ComposableRouterParams struct {
	Routers []ConfigRouter
	Timeout *OptionalDuration `json:",omitempty"`
}

type ConfigRouter struct {
	RouterName   string
	Timeout      Duration
	IgnoreErrors bool
	ExecuteAfter *OptionalDuration `json:",omitempty"`
}

type Method struct {
	RouterName string
}

```