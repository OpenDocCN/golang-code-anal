# go-ipfs 源码解析 33

<h1 align="center">
  <br>
  <a href="#readme"><img src="https://github.com/ipfs-shipyard/nopfs/blob/41484a818e6542314f784da852fc41b76f2d48a6/logo.png?raw=true" alt="content blocking logo" title="content blocking in Kubo" width="200"></a>
  <br>
  Content Blocking in Kubo
  <br>
</h1>

Kubo ships with built-in support for denylist format from [IPIP-383](https://github.com/ipfs/specs/pull/383).

## Default behavior

Official Kubo build does not ship with any denylists enabled by default.

Content blocking is an opt-in decision made by the operator of `ipfs daemon`.

## How to enable blocking

Place a `*.deny` file in one of directories:

- `$IPFS_PATH/denylists/` (`$HOME/.ipfs/denylists/` if `IPFS_PATH` is not set)
- `$XDG_CONFIG_HOME/ipfs/denylists/` (`$HOME/.config/ipfs/denylists/` if `XDG_CONFIG_HOME` is not  set)
- `/etc/ipfs/denylists/` (global)

Files need to be present before starting the `ipfs daemon` in order to be watched for updates.

If a new denylist file is added, `ipfs daemon` needs to be restarted.

CLI and Gateway users will receive errors in response to request impacted by a blocklist:

```go
Error: /ipfs/QmQvjk82hPkSaZsyJ8vNER5cmzKW7HyGX5XVusK7EAenCN is blocked and cannot be provided
```

End user is not informed about the exact reason, see [How to
debug](#how-to-debug) if you need to find out which line of which denylist
caused the request to be blocked.

## Denylist file format

[NOpfs](https://github.com/ipfs-shipyard/nopfs) supports the format from [IPIP-383](https://github.com/ipfs/specs/pull/383).

Clear-text rules are simple: just put content paths to block, one per line.
Paths with unicode and whitespace need to be percend-encoded:

```go
/ipfs/QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR
/ipfs/bafybeihfg3d7rdltd43u3tfvncx7n5loqofbsobojcadtmokrljfthuc7y/927%20-%20Standards/927%20-%20Standards.png
```

Sensitive content paths can be double-hashed to block without revealing them.
Double-hashed list example: https://badbits.dwebops.pub/badbits.deny

See [IPIP-383](https://github.com/ipfs/specs/pull/383) for detailed format specification and more examples.

## How to suspend blocking without removing denylists

Set `IPFS_CONTENT_BLOCKING_DISABLE` environment variable to `true` and restart the daemon.


## How to debug

Debug logging of `nopfs` subsystem can be enabled with `GOLOG_LOG_LEVEL="nopfs=debug"`

All block events are logged as warnings on a separate level named `nopfs-blocks`.

To only log requests for blocked content set `GOLOG_LOG_LEVEL="nopfs-blocks=warn"`:

```go
WARN (...) QmRFniDxwxoG2n4AcnGhRdjqDjCM5YeUcBE75K8WXmioH3: blocked (test.deny:9)
```




# Customizing Kubo

You may want to customize Kubo if you want to reuse most of Kubo's machinery. This document discusses some approaches you may consider for customizing Kubo, and their tradeoffs.

Some common use cases for customizing Kubo include:

- Using a custom datastore for storing blocks, pins, or other Kubo metadata
- Adding a custom data transfer protocol into Kubo
- Customizing Kubo internals, such as adding allowlist/blocklist functionality to Bitswap
- Adding new commands, interfaces, functionality, etc. to Kubo while reusing the libp2p swarm
- Building on top of Kubo's configuration and config migration functionality

## Summary
This table summarizes the tradeoffs between the approaches below:

|                     | [Boxo](#boxo-build-your-own-binary) | [Kubo Plugin](#kubo-plugins) | [Bespoke Extension Point](#bespoke-extension-points) | [Go Plugin](#go-plugins) | [Fork](#fork-kubo) |
|:-------------------:|:-----------------------------------:|:----------------------------:|:----------------------------------------------------:|:------------------------:|:------------------:|
|     Supported?      |                  ✅                  |              ✅               |                          ✅                           |            ❌             |         ❌          |
|    Future-proof?    |                  ✅                  |              ❌               |                          ✅                           |            ❌             |         ❌          |
| Fully customizable? |                  ✅                  |              ✅               |                          ❌                           |            ✅             |         ✅          |
| Fast to implement?  |                  ❌                  |              ✅               |                          ✅                           |            ✅             |         ✅          |
| Dynamic at runtime? |                  ❌                  |              ❌               |                          ✅                           |            ✅             |         ❌          |
|  Add new commands?  |                  ❌                  |              ✅               |                          ❌                           |            ✅             |         ✅          |

## Boxo: build your own binary
The best way to reuse Kubo functionality is to pick the functionality you need directly from [Boxo](https://github.com/ipfs/boxo) and compile your own binary.

Boxo's raison d'etre is to be an IPFS component toolbox to support building custom-made implementations and applications. If your use case is not easy to implement with Boxo, you may want to consider adding whatever functionality is needed to Boxo instead of customizing Kubo, so that the community can benefit. If you are interested in this option, please reach out to Boxo maintainers, who will be happy to help you scope & plan the work. See [Boxo's FAQ](https://github.com/ipfs/boxo#help) for more info.

## Kubo Plugins
Kubo plugins are a set of interfaces that may be implemented and injected into Kubo. Generally you should recompile the Kubo binary with your plugins added. A popular example of a Kubo plugin is [go-ds-s3](https://github.com/ipfs/go-ds-s3), which can be used to store blocks in Amazon S3.

Some plugins, such as the `fx` plugin, allow deep customization of Kubo internals. As a result, Kubo maintainers can't guarantee backwards compatibility with these, so you may need to adapt to breaking changes when upgrading to new Kubo versions.

For more information about the different types of Kubo plugins, see [plugins.md](./plugins.md).

Kubo plugins can also be injected at runtime using Go plugins (see below), but these are hard to use and not well supported by Go, so we don't recommend them.

## Bespoke Extension Points
Certain Kubo functionality may have their own extension points. For example:

* Kubo supports the [Routing v1](https://github.com/ipfs/specs/blob/main/routing/ROUTING_V1_HTTP.md) API for delegating content routing to external processes
* Kubo supports the [Pinning Service API](https://github.com/ipfs/pinning-services-api-spec) for delegating pinning to external processes
* Kubo supports [DNSLink](https://dnslink.dev/) for delegating name->CID mappings to DNS

(This list is not exhaustive.)

These can generally be developed and deployed as sidecars (or full external services) without modifying the Kubo binary.

## Go Plugins
Go provides [dynamic plugins](https://pkg.go.dev/plugin) which can be loaded at runtime into a Go binary.

Kubo currently works with Go plugins. But using Go plugins requires that you compile the plugin using the exact same version of the Go toolchain with the same configuration (build flags, environment variables, etc.). As a result, you likely need to build Kubo and the plugins together at the same time, and at that point you may as well just compile the functionality directly into Kubo and avoid Go plugins.

As a result, we don't recommend using Go plugins, and are likely to remove them in a future release of Kubo.

## Fork Kubo
The "nuclear option" is to fork Kubo into your own repo, make your changes, and periodically sync your repo with the Kubo repo. This can be a good option if your changes are significant and you can commit to keeping your repo in sync with Kubo.

Kubo maintainers can't make any backwards compatibility guarantees about Kubo internals, so by choosing this option you're accepting the risk that you may need to spend more time adapting to breaking changes.


# Datastore Configuration Options

This document describes the different possible values for the `Datastore.Spec`
field in the ipfs configuration file.

## flatfs

Stores each key value pair as a file on the filesystem.

The shardFunc is prefixed with `/repo/flatfs/shard/v1` then followed by a descriptor of the sharding strategy. Some example values are:
- `/repo/flatfs/shard/v1/next-to-last/2`
  - Shards on the two next to last characters of the key
- `/repo/flatfs/shard/v1/prefix/2`
  - Shards based on the two character prefix of the key

```gojson
{
	"type": "flatfs",
	"path": "<relative path within repo for flatfs root>",
	"shardFunc": "<a descriptor of the sharding scheme>",
	"sync": true|false
}
```

NOTE: flatfs must only be used as a block store (mounted at `/blocks`) as it only partially implements the datastore interface. You can mount flatfs for /blocks only using the mount datastore (described below).

## levelds
Uses a leveldb database to store key value pairs.

```gojson
{
	"type": "levelds",
	"path": "<location of db inside repo>",
	"compression": "none" | "snappy",
}
```

## badgerds

Uses [badger](https://github.com/dgraph-io/badger) as a key value store.

* `syncWrites`: Flush every write to disk before continuing. Setting this to false is safe as kubo will automatically flush writes to disk before and after performing critical operations like pinning. However, you can set this to true to be extra-safe (at the cost of a 2-3x slowdown when adding files).
* `truncate`: Truncate the DB if a partially written sector is found (defaults to true). There is no good reason to set this to false unless you want to manually recover partially written (and unpinned) blocks if kubo crashes half-way through a adding a file.

```gojson
{
	"type": "badgerds",
	"path": "<location of badger inside repo>",
	"syncWrites": true|false,
	"truncate": true|false,
}
```

## mount

Allows specified datastores to handle keys prefixed with a given path.
The mountpoints are added as keys within the child datastore definitions.

```gojson
{
	"type": "mount",
	"mounts": [
		{
			// Insert other datastore definition here, but add the following key:
			"mountpoint": "/path/to/handle"
		},
		{
			// Insert other datastore definition here, but add the following key:
			"mountpoint": "/path/to/handle"
		},
	]
}
```

## measure

This datastore is a wrapper that adds metrics tracking to any datastore.

```gojson
{
	"type": "measure",
	"prefix": "sometag.datastore",
	"child": { datastore being wrapped }
}
```



# General performance debugging guidelines

This is a document for helping debug Kubo. Please add to it if you can!

# Table of Contents

- [General performance debugging guidelines](#general-performance-debugging-guidelines)
- [Table of Contents](#table-of-contents)
    - [Beginning](#beginning)
    - [Analyzing the stack dump](#analyzing-the-stack-dump)
    - [Analyzing the CPU Profile](#analyzing-the-cpu-profile)
    - [Analyzing vars and memory statistics](#analyzing-vars-and-memory-statistics)
    - [Tracing](#tracing)
    - [Other](#other)

### Beginning

When you see ipfs doing something (using lots of CPU, memory, or otherwise
being weird), the first thing you want to do is gather all the relevant
profiling information.

There's a command (`ipfs diag profile`) that will do this for you and
bundle the results up into a zip file, ready to be attached to a bug report.

If you feel intrepid, you can dump this information and investigate it yourself:

- goroutine dump
  - `curl localhost:5001/debug/pprof/goroutine\?debug=2 > ipfs.stacks`
- 30 second cpu profile
  - `curl localhost:5001/debug/pprof/profile > ipfs.cpuprof`
- heap trace dump
  - `curl localhost:5001/debug/pprof/heap > ipfs.heap`
- memory statistics (in json, see "memstats" object)
  - `curl localhost:5001/debug/vars > ipfs.vars`
- system information
  - `ipfs diag sys > ipfs.sysinfo`


### Analyzing the stack dump

The first thing to look for is hung goroutines -- any goroutine that's been stuck
for over a minute will note that in the trace. It looks something like:

```go
goroutine 2306090 [semacquire, 458 minutes]:
sync.runtime_Semacquire(0xc8222fd3e4)
  /home/whyrusleeping/go/src/runtime/sema.go:47 +0x26
sync.(*Mutex).Lock(0xc8222fd3e0)
  /home/whyrusleeping/go/src/sync/mutex.go:83 +0x1c4
gx/ipfs/QmedFDs1WHcv3bcknfo64dw4mT1112yptW1H65Y2Wc7KTV/yamux.(*Session).Close(0xc8222fd340, 0x0, 0x0)
  /home/whyrusleeping/gopkg/src/gx/ipfs/QmedFDs1WHcv3bcknfo64dw4mT1112yptW1H65Y2Wc7KTV/yamux/session.go:205 +0x55
gx/ipfs/QmWSJzRkCMJFHYUQZxKwPX8WA7XipaPtfiwMPARP51ymfn/go-stream-muxer/yamux.(*conn).Close(0xc8222fd340, 0x0, 0x0)
  /home/whyrusleeping/gopkg/src/gx/ipfs/QmWSJzRkCMJFHYUQZxKwPX8WA7XipaPtfiwMPARP51ymfn/go-stream-muxer/yamux/yamux.go:39 +0x2d
gx/ipfs/QmZK81vcgMhpb2t7GNbozk7qzt6Rj4zFqitpvsWT9mduW8/go-peerstream.(*Conn).Close(0xc8257a2000, 0x0, 0x0)
  /home/whyrusleeping/gopkg/src/gx/ipfs/QmZK81vcgMhpb2t7GNbozk7qzt6Rj4zFqitpvsWT9mduW8/go-peerstream/conn.go:156 +0x1f2
created by gx/ipfs/QmZK81vcgMhpb2t7GNbozk7qzt6Rj4zFqitpvsWT9mduW8/go-peerstream.(*Conn).GoClose
  /home/whyrusleeping/gopkg/src/gx/ipfs/QmZK81vcgMhpb2t7GNbozk7qzt6Rj4zFqitpvsWT9mduW8/go-peerstream/conn.go:131 +0xab
```

At the top, you can see that this goroutine (number 2306090) has been waiting
to acquire a semaphore for 458 minutes. That seems bad. Looking at the rest of
the trace, we see the exact line it's waiting on is line 47 of runtime/sema.go.
That's not particularly helpful, so we move on. Next, we see that call was made
by line 205 of yamux/session.go in the `Close` method of `yamux.Session`. This
one appears to be the issue.

Given that information, look for another goroutine that might be
holding the semaphore in question in the rest of the stack dump.
(If you need help doing this, ping and we'll stub this out.)

There are a few different reasons that goroutines can be hung:
- `semacquire` means we're waiting to take a lock or semaphore.
- `select` means that the goroutine is hanging in a select statement and none of
  the cases are yielding anything.
- `chan receive` and `chan send` are waiting for a channel to be received from
  or sent on, respectively.
- `IO wait` generally means that we are waiting on a socket to read or write
  data, although it *can* mean we are waiting on a very slow filesystem.

If you see any of those tags _without_ a `,
X minutes` suffix, that generally means there isn't a problem -- you just caught
that goroutine in the middle of a short wait for something. If the wait time is
over a few minutes, that either means that goroutine doesn't do much, or
something is pretty wrong.

If you're seeing a lot of goroutines, consider using
[stackparse](https://github.com/whyrusleeping/stackparse) to filter, sort, and summarize them.

### Analyzing the CPU Profile

The go team wrote an [excellent article on profiling go
programs](http://blog.golang.org/profiling-go-programs). If you've already
gathered the above information, you can skip down to where they start talking
about `go tool pprof`. My go-to method of analyzing these is to run the `web`
command, which generates an SVG dotgraph and opens it in your browser. This is
the quickest way to easily point out where the hot spots in the code are.

### Analyzing vars and memory statistics

The output is JSON formatted and includes badger store statistics, the command line run, and the output from Go's [runtime.ReadMemStats](https://golang.org/pkg/runtime/#ReadMemStats). The [MemStats](https://golang.org/pkg/runtime/#MemStats) has useful information about memory allocation and garbage collection.

### Tracing

Experimental tracing via OpenTelemetry suite of tools is available.
See `tracing/doc.go` for more details.

### Other

If you have any questions, or want us to analyze some weird kubo behaviour,
just let us know, and be sure to include all the profiling information
mentioned at the top.


# New multi-router configuration system

- Start Date: 2022-08-15
- Related Issues:
  - https://github.com/ipfs/kubo/issues/9188
  - https://github.com/ipfs/kubo/issues/9079
  - https://github.com/ipfs/kubo/pull/9877

## Summary

Previously we only used the Amino DHT for content routing and content
providing.

Kubo 0.14 introduced experimental support for [delegated routing using Reframe protocol](https://github.com/ipfs/kubo/pull/8997).
Since then,  Reframe got deprecated and superseded by [Routing V1 HTTP API](https://specs.ipfs.tech/routing/http-routing-v1/).

Kubo 0.23.0 release added support for [self-hosting Routing V1 HTTP API server](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.23.md#self-hosting-routingv1-endpoint-for-delegated-routing-needs).

Now we need a better way to add different routers using different protocols
like [Routing V1](https://specs.ipfs.tech/routing/http-routing-v1/) or Amino
DHT, and be able to configure them (future routing systems to come) to cover different use cases.

## Motivation

The actual routing implementation is not enough. Some users need to have more options when configuring the routing system. The new implementations should be able to:

- [x] Be user-friendly and easy enough to configure, but also versatile
- [x] Configurable Router execution order
     - [x] Delay some of the Router methods execution when they will be executed on parallel
- [x] Configure which method of a giving router will be used
- [x] Mark some router methods as mandatory to make the execution fails if that method fails

## Detailed design

### Configuration file description

The `Routing` configuration section will contain the following keys:

#### Type

`Type` will be still in use to avoid complexity for the user that only wants to use Kubo with the default behavior. We are going to add a new type, `custom`, that will use the new router systems. `none` type will deactivate **all** routers, default dht and delegated ones.

#### Routers

`Routers` will be a key-value list of routers that will be available to use. The key is the router name and the value is all the needed configurations for that router. the `Type` will define the routing kind. The main router types will be `reframe` and `dht`, but we will implement two special routers used to execute a set of routers in parallel or sequentially: `parallel` router and `sequential` router.

Depending on the routing type, it will use different parameters:

##### Reframe

Params:

- `"Endpoint"`: URL endpoint implementing Reframe protocol.

##### Amino DHT

Params:
- `"Mode"`: Mode used by the Amino DHT. Possible values: "server", "client", "auto"
- `"AcceleratedDHTClient"`: Set to `true` if you want to use the experimentalDHT.
- `"PublicIPNetwork"`: Set to `true` to create a `WAN` Amino DHT. Set to `false` to create a `LAN` DHT.

##### Parallel

Params:
- `Routers`: A list of routers that will be executed in parallel:
    - `Name:string`: Name of the router. It should be one of the previously added to `Routers` list.
    - `Timeout:duration`: Local timeout. It accepts strings compatible with Go `time.ParseDuration(string)`. Time will start counting when this specific router is called, and it will stop when the router returns, or we reach the specified timeout.
    - `ExecuteAfter:duration`: Providing this param will delay the execution of that router at the specified time. It accepts strings compatible with Go `time.ParseDuration(string)`.
    - `IgnoreErrors:bool`: It will specify if that router should be ignored if an error occurred.
- `Timeout:duration`: Global timeout.  It accepts strings compatible with Go `time.ParseDuration(string)`.
##### Sequential

Params:
- `Routers`: A list of routers that will be executed in order:
    - `Name:string`: Name of the router. It should be one of the previously added to `Routers` list.
    - `Timeout:duration`: Local timeout. It accepts strings compatible with Go `time.ParseDuration(string)`. Time will start counting when this specific router is called, and it will stop when the router returns, or we reach the specified timeout.
    - `IgnoreErrors:bool`: It will specify if that router should be ignored if an error occurred.
- `Timeout:duration`: Global timeout.  It accepts strings compatible with Go `time.ParseDuration(string)`.
#### Methods

`Methods:map` will define which routers will be executed per method. The key will be the name of the method: `"provide"`, `"find-providers"`, `"find-peers"`, `"put-ipns"`, `"get-ipns"`. All methods must be added to the list. This will make configuration discoverable giving good errors to the user if a method is missing.

The value will contain:
- `RouterName:string`: Name of the router. It should be one of the previously added to `Routers` list.

#### Configuration file example:

```gojson
"Routing": {
  "Type": "custom",
  "Routers": {
    "storetheindex": {
      "Type": "reframe",
      "Parameters": {
        "Endpoint": "https://cid.contact/reframe"
      }
    },
    "dht-lan": {
      "Type": "dht",
      "Parameters": {
        "Mode": "server",
        "PublicIPNetwork": false,
        "AcceleratedDHTClient": false
      }
    },
    "dht-wan": {
      "Type": "dht",
      "Parameters": {
        "Mode": "auto",
        "PublicIPNetwork": true,
        "AcceleratedDHTClient": false
      }
    },
    "find-providers-router": {
      "Type": "parallel",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan",
            "IgnoreErrors": true
          },
          {
            "RouterName": "dht-wan"
          },
          {
            "RouterName": "storetheindex"
          }
        ]
      }
    },
    "provide-router": {
      "Type": "parallel",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan",
            "IgnoreErrors": true
          },
          {
            "RouterName": "dht-wan",
            "ExecuteAfter": "100ms",
            "Timeout": "100ms"
          },
          {
            "RouterName": "storetheindex",
            "ExecuteAfter": "100ms"
          }
        ]
      }
    },
    "get-ipns-router": {
      "Type": "sequential",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan",
            "IgnoreErrors": true
          },
          {
            "RouterName": "dht-wan",
            "Timeout": "300ms"
          },
          {
            "RouterName": "storetheindex",
            "Timeout": "300ms"
          }
        ]
      }
    },
    "put-ipns-router": {
      "Type": "parallel",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan"
          },
          {
            "RouterName": "dht-wan"
          },
          {
            "RouterName": "storetheindex"
          }
        ]
      }
    }
  },
  "Methods": {
    "find-providers": {
      "RouterName": "find-providers-router"
    },
    "provide": {
      "RouterName": "provide-router"
    },
    "get-ipns": {
      "RouterName": "get-ipns-router"
    },
    "put-ipns": {
      "RouterName": "put-ipns-router"
    }
  }
}
```

Added YAML for clarity:

```goyaml
---
Type: custom
Routers:
  storetheindex:
    Type: reframe
    Parameters:
      Endpoint: https://cid.contact/reframe
  dht-lan:
    Type: dht
    Parameters:
      Mode: server
      PublicIPNetwork: false
      AcceleratedDHTClient: false
  dht-wan:
    Type: dht
    Parameters:
      Mode: auto
      PublicIPNetwork: true
      AcceleratedDHTClient: false
  find-providers-router:
    Type: parallel
    Parameters:
      Routers:
      - RouterName: dht-lan
        IgnoreErrors: true
      - RouterName: dht-wan
      - RouterName: storetheindex
  provide-router:
    Type: parallel
    Parameters:
      Routers:
      - RouterName: dht-lan
        IgnoreErrors: true
      - RouterName: dht-wan
        ExecuteAfter: 100ms
        Timeout: 100ms
      - RouterName: storetheindex
        ExecuteAfter: 100ms
  get-ipns-router:
    Type: sequential
    Parameters:
      Routers:
      - RouterName: dht-lan
        IgnoreErrors: true
      - RouterName: dht-wan
        Timeout: 300ms
      - RouterName: storetheindex
        Timeout: 300ms
  put-ipns-router:
    Type: parallel
    Parameters:
      Routers:
      - RouterName: dht-lan
      - RouterName: dht-wan
      - RouterName: storetheindex
Methods:
  find-providers:
    RouterName: find-providers-router
  provide:
    RouterName: provide-router
  get-ipns:
    RouterName: get-ipns-router
  put-ipns:
    RouterName: put-ipns-router
```

### Error cases
 - If any of the routers fails, the output will be an error by default.
 - You can use `IgnoreErrors:true` to ignore errors for a specific router output
 - To avoid any error at the output, you must ignore all router errors.

### Implementation Details

#### Methods

All routers must implement the `routing.Routing` interface:

```gogo=
type Routing interface {
    ContentRouting
    PeerRouting
    ValueStore

    Bootstrap(context.Context) error
}
```

All methods involved:

```gogo=
type Routing interface {
    Provide(context.Context, cid.Cid, bool) error
    FindProvidersAsync(context.Context, cid.Cid, int) <-chan peer.AddrInfo
    
    FindPeer(context.Context, peer.ID) (peer.AddrInfo, error)

    PutValue(context.Context, string, []byte, ...Option) error
    GetValue(context.Context, string, ...Option) ([]byte, error)
    SearchValue(context.Context, string, ...Option) (<-chan []byte, error)

    Bootstrap(context.Context) error
}
```
We can configure which methods will be used per routing implementation. Methods names used in the configuration file will be:

- `Provide`: `"provide"`
- `FindProvidersAsync`: `"find-providers"`
- `FindPeer`: `"find-peers"`
- `PutValue`: `"put-ipns"`
- `GetValue`, `SearchValue`: `"get-ipns"`
- `Bootstrap`: It will be always executed when needed.

#### Routers

We need to implement the `parallel` and `sequential` routers and stop using `routinghelpers.Tiered` router implementation.

Add cycle detection to avoid to user some headaches.

Also we need to implement an internal router, that will define the router used per method.

#### Other considerations

- We need to refactor how DHT routers are created to be able to use and add any amount of custom DHT routers.
- We need to add a new `custom` router type to be able to use the new routing system.
- Bitswap WANT broadcasting is not included on this document, but it can be added in next iterations.
- This document will live in docs/design-notes for historical reasons and future reference.

## Test fixtures

As test fixtures we can add different use cases here and see how the configuration will look like.

### Mimic previous dual DHT config

```gojson
"Routing": {
  "Type": "custom",
  "Routers": {
    "dht-lan": {
      "Type": "dht",
      "Parameters": {
        "Mode": "server",
        "PublicIPNetwork": false
      }
    },
    "dht-wan": {
      "Type": "dht",
      "Parameters": {
        "Mode": "auto",
        "PublicIPNetwork": true
      }
    },
    "parallel-dht-strict": {
      "Type": "parallel",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan"
          },
          {
            "RouterName": "dht-wan"
          }
        ]
      }
    },
    "parallel-dht": {
      "Type": "parallel",
      "Parameters": {
        "Routers": [
          {
            "RouterName": "dht-lan",
            "IgnoreError": true
          },
          {
            "RouterName": "dht-wan"
          }
        ]
      }
    }
  },
  "Methods": {
    "provide": {
      "RouterName": "dht-wan"
    },
    "find-providers": {
      "RouterName": "parallel-dht-strict"
    },
    "find-peers": {
      "RouterName": "parallel-dht-strict"
    },
    "get-ipns": {
      "RouterName": "parallel-dht"
    },
    "put-ipns": {
      "RouterName": "parallel-dht"
    }
  }
}
```
YAML representation for clarity:

```goyaml
---
Type: custom
Routers:
  dht-lan:
    Type: dht
    Parameters:
      Mode: server
      PublicIPNetwork: false
  dht-wan:
    Type: dht
    Parameters:
      Mode: auto
      PublicIPNetwork: true
  parallel-dht-strict:
    Type: parallel
    Parameters:
      Routers:
      - RouterName: dht-lan
      - RouterName: dht-wan
  parallel-dht:
    Type: parallel
    Parameters:
      Routers:
      - RouterName: dht-lan
        IgnoreError: true
      - RouterName: dht-wan
Methods:
  provide:
    RouterName: dht-wan
  find-providers:
    RouterName: parallel-dht-strict
  find-peers:
    RouterName: parallel-dht-strict
  get-ipns:
    RouterName: parallel-dht
  put-ipns:
    RouterName: parallel-dht

```

### Compatibility

~~We need to create a config migration using [fs-repo-migrations](https://github.com/ipfs/fs-repo-migrations). We should remove the `Routing.Type` param and add the configuration specified [previously](#Mimic-previous-dual-DHT-config).~~

We don't need to create any config migration! To avoid to the users the hassle of understanding how the new routing system works, we are gonna keep the old behavior. We will add the Type `custom` to make available the new Routing system.

### Security

No new security implications or considerations were found.

### Alternatives

I got ideas from all of the following links to create this design document:

- https://github.com/ipfs/kubo/issues/9079#issuecomment-1211288268
- https://github.com/ipfs/kubo/issues/9157
- https://github.com/ipfs/kubo/issues/9079#issuecomment-1205000253
- https://www.notion.so/pl-strflt/Delegated-Routing-Thoughts-very-very-WIP-0543bc51b1bd4d63a061b0f28e195d38
- https://gist.github.com/guseggert/effa027ff4cbadd7f67598efb6704d12

### Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).


# EARLY TESTERS PROGRAMME

## What is it?

The early testers programme allows groups using Kubo in production to self-volunteer to help test `kubo` release candidates to ensure that no regressions that might affect production systems make it into the final release. While we invite the _entire_ community to help test releases, members of the early testers program are expected to participate directly and actively in every release.

## What are the expectations?

Members of the early tester program are expected to work closely with us to:

* Provide high quality, actionable feedback.
* Work directly with us to debug regressions in the release.
* Help ensure a rock-solid, timely release.

We will ask early testers to participate at two points in the process:

* When Kubo enters the second release stage (public beta), early testers will be asked to test Kubo on non-production infrastructure. This may involve things like:
  - Running integration tests against the release candidate.
  - Running simulations/benchmarks on the release candidate.
  - Manually testing the release candidate to check for regressions.
* When Kubo enters the third release stage (soft release), early testers will be asked to partially deploy the release candidate to production infrastructure. Release candidates at this stage are expected to be identical to the final release. However, this stage allows the Kubo team to fix any last-minute regressions without cutting an entirely new release.

## Who has signed up?

- [ ] Charity Engine (@rytiss, @tristanolive)
- [ ] Fission (@bmann)
- [ ] Infura (@MichaelMure)
- [ ] OrbitDB (@haydenyoung)
- [ ] pacman.store (@RubenKelevra)
- [ ] Pinata (@obo20)
- [ ] PL EngRes bifrost (@gmasgras)
- [ ] Siderus (@koalalorenzo)
- [ ] Textile (@sanderpick)

## How to sign up?

Simply submit a PR to this document by adding your project name and contact.


# Kubo environment variables

## `IPFS_PATH`

Sets the location of the IPFS repo (where the config, blocks, etc.
are stored).

Default: ~/.ipfs

## `IPFS_LOGGING`

Specifies the log level for Kubo.

`IPFS_LOGGING` is a deprecated alias for the `GOLOG_LOG_LEVEL` environment variable.  See below.

## `IPFS_LOGGING_FMT`

Specifies the log message format.

`IPFS_LOGGING_FMT` is a deprecated alias for the `GOLOG_LOG_FMT` environment variable.  See below.

## `GOLOG_LOG_LEVEL`

Specifies the log-level, both globally and on a per-subsystem basis.  Level can be one of:

* `debug`
* `info`
* `warn`
* `error`
* `dpanic`
* `panic`
* `fatal`

Per-subsystem levels can be specified with `subsystem=level`.  One global level and one or more per-subsystem levels
can be specified by separating them with commas.

Default: `error`

Example:

```goconsole
GOLOG_LOG_LEVEL="error,core/server=debug" ipfs daemon
```

Logging can also be configured at runtime, both globally and on a per-subsystem basis, with the `ipfs log` command.

## `GOLOG_LOG_FMT`

Specifies the log message format.  It supports the following values:

- `color` -- human readable, colorized (ANSI) output
- `nocolor` -- human readable, plain-text output.
- `json` -- structured JSON.

For example, to log structured JSON (for easier parsing):

```gobash
export GOLOG_LOG_FMT="json"
```
The logging format defaults to `color` when the output is a terminal, and `nocolor` otherwise.

## `GOLOG_FILE`

Sets the file to which Kubo logs. By default, Kubo logs to standard error.

## `GOLOG_TRACING_FILE`

Sets the file to which Kubo sends tracing events. By default, tracing is
disabled.

This log can be read at runtime (without writing it to a file) using the `ipfs
log tail` command.

Warning: Enabling tracing will likely affect performance.

## `IPFS_FUSE_DEBUG`

If SET, enables fuse debug logging.

Default: false

## `YAMUX_DEBUG`

If SET, enables debug logging for the yamux stream muxer.

Default: false

## `IPFS_FD_MAX`

Sets the file descriptor limit for Kubo. If Kubo fails to set the file
descriptor limit, it will log an error.

Defaults: 2048

## `IPFS_DIST_PATH`

IPFS Content Path from which Kubo fetches repo migrations (when the daemon
is launched with the `--migrate` flag).

Default: `/ipfs/<cid>` (the exact path is hardcoded in
`migrations.CurrentIpfsDist`, depends on the IPFS version)

## `IPFS_NS_MAP`

Adds static namesys records for deterministic tests and debugging.
Useful for testing things like DNSLink without real DNS lookup.

Example:

```goconsole
$ IPFS_NS_MAP="dnslink-test1.example.com:/ipfs/bafkreicysg23kiwv34eg2d7qweipxwosdo2py4ldv42nbauguluen5v6am,dnslink-test2.example.com:/ipns/dnslink-test1.example.com" ipfs daemon
...
$ ipfs resolve -r /ipns/dnslink-test2.example.com
/ipfs/bafkreicysg23kiwv34eg2d7qweipxwosdo2py4ldv42nbauguluen5v6am
```

## `IPFS_HTTP_ROUTERS`

Overrides all implicit HTTP routers enabled when `Routing.Type=auto` with
the space-separated list of URLs provided in this variable.
Useful for testing and debugging in offline contexts.

Example:

```goconsole
$ ipfs config Routing.Type auto
$ IPFS_HTTP_ROUTERS="http://127.0.0.1:7423" ipfs daemon
```

The above will replace implicit HTTP routers with single one, allowing for
inspection/debug of HTTP requests sent by Kubo via `while true ; do nc -l 7423; done`
or more advanced tools like [mitmproxy](https://docs.mitmproxy.org/stable/#mitmproxy).


## `IPFS_CONTENT_BLOCKING_DISABLE`

Disables the content-blocking subsystem. No denylists will be watched and no
content will be blocked.

## `LIBP2P_TCP_REUSEPORT`

Kubo tries to reuse the same source port for all connections to improve NAT
traversal. If this is an issue, you can disable it by setting
`LIBP2P_TCP_REUSEPORT` to false.

Default: true

## `LIBP2P_MUX_PREFS`

Deprecated: Use the `Swarm.Transports.Multiplexers` config field.

Tells Kubo which multiplexers to use in which order.

Default: "/yamux/1.0.0 /mplex/6.7.0"

## `LIBP2P_RCMGR`

Forces [libp2p Network Resource Manager](https://github.com/libp2p/go-libp2p-resource-manager#readme)
to be enabled (`1`) or disabled (`0`).
When set, overrides [`Swarm.ResourceMgr.Enabled`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmresourcemgrenabled) from the config.

Default: use config (not set)

## `LIBP2P_DEBUG_RCMGR`

Enables tracing of [libp2p Network Resource Manager](https://github.com/libp2p/go-libp2p-resource-manager#readme)
and outputs it to `rcmgr.json.gz`


Default: disabled (not set)

# Tracing

For tracing configuration, please check: https://github.com/ipfs/boxo/blob/main/docs/tracing.md


# Experimental features of Kubo

This document contains a list of experimental features in Kubo.
These features, commands, and APIs aren't mature, and you shouldn't rely on them.
Once they reach maturity, there's going to be mention in the changelog and
release posts. If they don't reach maturity, the same applies, and their code is
removed.

Subscribe to https://github.com/ipfs/kubo/issues/3397 to get updates.

When you add a new experimental feature to kubo or change an experimental
feature, you MUST please make a PR updating this document, and link the PR in
the above issue.

- [Raw leaves for unixfs files](#raw-leaves-for-unixfs-files)
- [ipfs filestore](#ipfs-filestore)
- [ipfs urlstore](#ipfs-urlstore)
- [Private Networks](#private-networks)
- [ipfs p2p](#ipfs-p2p)
- [p2p http proxy](#p2p-http-proxy)
- [FUSE](#fuse)
- [Plugins](#plugins)
- [Directory Sharding / HAMT](#directory-sharding--hamt)
- [IPNS PubSub](#ipns-pubsub)
- [AutoRelay](#autorelay)
- [Strategic Providing](#strategic-providing)
- [Graphsync](#graphsync)
- [Noise](#noise)
- [Optimistic Provide](#optimistic-provide)
- [HTTP Gateway over Libp2p](#http-gateway-over-libp2p)

---

## Raw Leaves for unixfs files

Allows files to be added with no formatting in the leaf nodes of the graph.

### State

Stable but not used by default.

### In Version

0.4.5

### How to enable

Use `--raw-leaves` flag when calling `ipfs add`. This will save some space when adding files.

### Road to being a real feature

Enabling this feature _by default_ will change the CIDs (hashes) of all newly imported files and will prevent newly imported files from deduplicating against previously imported files. While we do intend on enabling this by default, we plan on doing so once we have a large batch of "hash-changing" features we can enable all at once.

## ipfs filestore

Allows files to be added without duplicating the space they take up on disk.

### State

Experimental.

### In Version

0.4.7

### How to enable

Modify your ipfs config:
```go
ipfs config --json Experimental.FilestoreEnabled true
```

Then restart your IPFS node to reload your config.

Finally, when adding files with ipfs add, pass the --nocopy flag to use the
filestore instead of copying the files into your local IPFS repo.

### Road to being a real feature

- [ ] Needs more people to use and report on how well it works.
- [ ] Need to address error states and failure conditions
- [ ] Need to write docs on usage, advantages, disadvantages
- [ ] Need to merge utility commands to aid in maintenance and repair of filestore

## ipfs urlstore

Allows ipfs to retrieve blocks contents via a URL instead of storing it in the datastore

### State

Experimental.

### In Version

v0.4.17

### How to enable

Modify your ipfs config:
```go
ipfs config --json Experimental.UrlstoreEnabled true
```

And then add a file at a specific URL using `ipfs urlstore add <url>`

### Road to being a real feature
- [ ] Needs more people to use and report on how well it works.
- [ ] Need to address error states and failure conditions
- [ ] Need to write docs on usage, advantages, disadvantages
- [ ] Need to implement caching
- [ ] Need to add metrics to monitor performance

## Private Networks

It allows ipfs to only connect to other peers who have a shared secret key.

### State

Stable but not quite ready for prime-time.

### In Version

0.4.7

### How to enable

Generate a pre-shared-key using [ipfs-swarm-key-gen](https://github.com/Kubuxu/go-ipfs-swarm-key-gen)):
```go
go get github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
ipfs-swarm-key-gen > ~/.ipfs/swarm.key
```

To join a given private network, get the key file from someone in the network
and save it to `~/.ipfs/swarm.key` (If you are using a custom `$IPFS_PATH`, put
it in there instead).

When using this feature, you will not be able to connect to the default bootstrap
nodes (Since we aren't part of your private network) so you will need to set up
your own bootstrap nodes.

First, to prevent your node from even trying to connect to the default bootstrap nodes, run:
```gobash
ipfs bootstrap rm --all
```

Then add your own bootstrap peers with:
```gobash
ipfs bootstrap add <multiaddr>
```

For example:
```go
ipfs bootstrap add /ip4/104.236.76.40/tcp/4001/p2p/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
```

Bootstrap nodes are no different from all other nodes in the network apart from
the function they serve.

To be extra cautious, You can also set the `LIBP2P_FORCE_PNET` environment
variable to `1` to force the usage of private networks. If no private network is
configured, the daemon will fail to start.

### Road to being a real feature

- [x] Needs more people to use and report on how well it works
- [ ] More documentation
- [ ] Needs better tooling/UX.

## ipfs p2p

Allows tunneling of TCP connections through Libp2p streams. If you've ever used
port forwarding with SSH (the `-L` option in OpenSSH), this feature is quite
similar.

### State

Experimental, will be stabilized in 0.6.0

### In Version

0.4.10

### How to enable

The `p2p` command needs to be enabled in the config:

```gosh
> ipfs config --json Experimental.Libp2pStreamMounting true
```

### How to use

**Netcat example:**

First, pick a protocol name for your application. Think of the protocol name as
a port number, just significantly more user-friendly. In this example, we're
going to use `/x/kickass/1.0`.

***Setup:***

1. A "server" node with peer ID `$SERVER_ID`
2. A "client" node.

***On the "server" node:***

First, start your application and have it listen for TCP connections on
port `$APP_PORT`.

Then, configure the p2p listener by running:

```gosh
> ipfs p2p listen /x/kickass/1.0 /ip4/127.0.0.1/tcp/$APP_PORT
```

This will configure IPFS to forward all incoming `/x/kickass/1.0` streams to
`127.0.0.1:$APP_PORT` (opening a new connection to `127.0.0.1:$APP_PORT` per
incoming stream.

***On the "client" node:***

First, configure the client p2p dialer, so that it forwards all inbound
connections on `127.0.0.1:SOME_PORT` to the server node listening
on `/x/kickass/1.0`.

```gosh
> ipfs p2p forward /x/kickass/1.0 /ip4/127.0.0.1/tcp/$SOME_PORT /p2p/$SERVER_ID
```

Next, have your application open a connection to `127.0.0.1:$SOME_PORT`. This
connection will be forwarded to the service running on `127.0.0.1:$APP_PORT` on
the remote machine. You can test it with netcat:

***On "server" node:***
```gosh
> nc -v -l -p $APP_PORT
```

***On "client" node:***
```gosh
> nc -v 127.0.0.1 $SOME_PORT
```

You should now see that a connection has been established and be able to
exchange messages between netcat instances.

(note that depending on your netcat version you may need to drop the `-v` flag)

**SSH example**

**Setup:**

1. A "server" node with peer ID `$SERVER_ID` and running ssh server on the
   default port.
2. A "client" node.

_you can get `$SERVER_ID` by running `ipfs id -f "<id>\n"`_

***First, on the "server" node:***

```gosh
ipfs p2p listen /x/ssh /ip4/127.0.0.1/tcp/22
```

***Then, on "client" node:***

```gosh
ipfs p2p forward /x/ssh /ip4/127.0.0.1/tcp/2222 /p2p/$SERVER_ID
```

You should now be able to connect to your ssh server through a libp2p connection
with `ssh [user]@127.0.0.1 -p 2222`.


### Road to being a real feature

- [ ] More documentation

## p2p http proxy

Allows proxying of HTTP requests over p2p streams. This allows serving any standard HTTP app over p2p streams.

### State

Experimental

### In Version

0.4.19

### How to enable

The `p2p` command needs to be enabled in the config:

```gosh
> ipfs config --json Experimental.Libp2pStreamMounting true
```

On the client, the p2p HTTP proxy needs to be enabled in the config:

```gosh
> ipfs config --json Experimental.P2pHttpProxy true
```

### How to use

**Netcat example:**

First, pick a protocol name for your application. Think of the protocol name as
a port number, just significantly more user-friendly. In this example, we're
going to use `/http`.

***Setup:***

1. A "server" node with peer ID `$SERVER_ID`
2. A "client" node.

***On the "server" node:***

First, start your application and have it listen for TCP connections on
port `$APP_PORT`.

Then, configure the p2p listener by running:

```gosh
> ipfs p2p listen --allow-custom-protocol /http /ip4/127.0.0.1/tcp/$APP_PORT
```

This will configure IPFS to forward all incoming `/http` streams to
`127.0.0.1:$APP_PORT` (opening a new connection to `127.0.0.1:$APP_PORT` per incoming stream.

***On the "client" node:***

Next, have your application make a http request to `127.0.0.1:8080/p2p/$SERVER_ID/http/$FORWARDED_PATH`. This
connection will be forwarded to the service running on `127.0.0.1:$APP_PORT` on
the remote machine (which needs to be a http server!) with path `$FORWARDED_PATH`. You can test it with netcat:

***On "server" node:***
```gosh
> echo -e "HTTP/1.1 200\nContent-length: 11\n\nIPFS rocks!" | nc -l -p $APP_PORT
```

***On "client" node:***
```gosh
> curl http://localhost:8080/p2p/$SERVER_ID/http/
```

You should now see the resulting HTTP response: IPFS rocks!

### Custom protocol names

We also support the use of protocol names of the form /x/$NAME/http where $NAME doesn't contain any "/"'s

### Road to being a real feature

- [ ] Needs p2p streams to graduate from experiments
- [ ] Needs more people to use and report on how well it works / fits use cases
- [ ] More documentation
- [ ] Need better integration with the subdomain gateway feature.

## FUSE

FUSE makes it possible to mount `/ipfs` and `/ipns` namespaces in your OS,
allowing arbitrary apps access to IPFS using a subset of filesystem abstractions.

It is considered  EXPERIMENTAL due to limited (and buggy) support on some platforms.

See [fuse.md](./fuse.md) for more details.

## Plugins

### In Version
0.4.11

### State
Experimental

Plugins allow adding functionality without the need to recompile the daemon.

### Basic Usage:

See [Plugin docs](./plugins.md)

### Road to being a real feature

- [x] More plugins and plugin types
- [ ] A way to reliably build and distribute plugins.
- [ ] Better support for platforms other than Linux & MacOS
- [ ] Feedback on stability

## Directory Sharding / HAMT

### In Version

- 0.4.8:
  - Introduced `Experimental.ShardingEnabled` which enabled sharding globally.
  - All-or-nothing, unnecessary sharding of small directories.

- 0.11.0 :
  - Removed support for `Experimental.ShardingEnabled`
  - Replaced with automatic sharding based on the block size

### State

Replaced by autosharding.

The `Experimental.ShardingEnabled` config field is no longer used, please remove it from your configs.

kubo now automatically shards when directory block is bigger than 256KB, ensuring every block is small enough to be exchanged with other peers

## IPNS pubsub

### In Version

0.4.14 :
  - Introduced

0.5.0 :
   - No longer needs to use the DHT for the first resolution
   - When discovering PubSub peers via the DHT, the DHT key is different from previous versions
      - This leads to 0.5 IPNS pubsub peers and 0.4 IPNS pubsub peers not being able to find each other in the DHT
   - Robustness improvements

0.11.0 :
  - Can be enabled via `Ipns.UsePubsub` flag in config

### State

Experimental, default-disabled.

Utilizes pubsub for publishing ipns records in real time.

When it is enabled:
- IPNS publishers push records to a name-specific pubsub topic,
  in addition to publishing to the DHT.
- IPNS resolvers subscribe to the name-specific topic on first
  resolution and receive subsequently published records through pubsub in real time.
  This makes subsequent resolutions instant, as they are resolved through the local cache.

Both the publisher and the resolver nodes need to have the feature enabled for it to work effectively.

Note: While IPNS pubsub has been available since 0.4.14, it received major changes in 0.5.0.
Users interested in this feature should upgrade to at least 0.5.0

### How to enable

Run your daemon with the `--enable-namesys-pubsub` flag
or modify your ipfs config and restart the daemon:
```go
ipfs config --json Ipns.UsePubsub true
```

NOTE:
- This feature implicitly enables [ipfs pubsub](#ipfs-pubsub).
- Passing `--enable-namesys-pubsub` CLI flag overrides `Ipns.UsePubsub` config.

### Road to being a real feature

- [ ] Needs more people to use and report on how well it works
- [ ] Pubsub enabled as a real feature

## AutoRelay

### In Version

- 0.4.19 :
  - Introduced Circuit Relay v1
- 0.11.0 :
  - Deprecated v1
  - Introduced [Circuit Relay v2](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md)

### State

Experimental, disabled by default.

Automatically discovers relays and advertises relay addresses when the node is behind an impenetrable NAT.

### How to enable

Modify your ipfs config:

```go
ipfs config --json Swarm.RelayClient.Enabled true
```

### Road to being a real feature

- [ ] needs testing
- [ ] needs to be automatically enabled when AutoNAT detects node is behind an impenetrable NAT.


## Strategic Providing

### State

Experimental, disabled by default.

Replaces the existing provide mechanism with a robust, strategic provider system. Currently enabling this option will provide nothing.

### How to enable

Modify your ipfs config:

```go
ipfs config --json Experimental.StrategicProviding true
```

### Road to being a real feature

- [ ] needs real-world testing
- [ ] needs adoption
- [ ] needs to support all provider subsystem features
    - [X] provide nothing
    - [ ] provide roots
    - [ ] provide all
    - [ ] provide strategic

## GraphSync

### State

Experimental, disabled by default.

[GraphSync](https://github.com/ipfs/go-graphsync) is the next-gen graph exchange
protocol for IPFS.

When this feature is enabled, IPFS will make files available over the graphsync
protocol. However, IPFS will not currently use this protocol to _fetch_ files.

### How to enable

Modify your ipfs config:

```go
ipfs config --json Experimental.GraphsyncEnabled true
```

### Road to being a real feature

- [ ] We need to confirm that it can't be used to DoS a node. The server-side logic for GraphSync is quite complex and, if we're not careful, the server might end up performing unbounded work when responding to a malicious request.

## Noise

### State

Stable, enabled by default

[Noise](https://github.com/libp2p/specs/tree/master/noise) libp2p transport based on the [Noise Protocol Framework](https://noiseprotocol.org/noise.html). While TLS remains the default transport in Kubo, Noise is easier to implement and is thus the "interop" transport between IPFS and libp2p implementations.

## Optimistic Provide

### In Version

0.20.0

### State

Experimental, disabled by default.

When the Amino DHT client tries to store a provider in the DHT, it typically searches for the 20 peers that are closest to the
target key. However, this process can be time-consuming, as the search terminates only after no closer peers are found
among the three currently (during the query) known closest ones. In cases where these closest peers are slow to respond
(which often happens if they are located at the edge of the DHT network), the query gets blocked by the slowest peer.

To address this issue, the `OptimisticProvide` feature can be enabled. This feature allows the client to estimate the
network size and determine how close a peer _likely_ needs to be to the target key to be within the 20 closest peers.
While searching for the closest peers in the DHT, the client will _optimistically_ store the provider record with peers
and abort the query completely when the set of currently known 20 closest peers are also _likely_ the actual 20 closest
ones. This heuristic approach can significantly speed up the process, resulting in a speed improvement of 2x to >10x.

When it is enabled:

- Amino DHT provide operations should complete much faster than with it disabled
- This can be tested with commands such as `ipfs routing provide`

**Tradeoffs**

There are now the classic client, the accelerated DHT client, and optimistic provide that improve the provider process.
There are different trade-offs with all of them. The accelerated DHT client is still faster to provide large amounts
of provider records at the cost of high resource requirements. Optimistic provide doesn't have the high resource
requirements but might not choose optimal peers and is not as fast as the accelerated client, but still much faster
than the classic client.

**Caveats:**

1. Providing optimistically requires a current network size estimation. This estimation is calculated through routing
   table refresh queries and is only available after the daemon has been running for some time. If there is no network
   size estimation available the client will transparently fall back to the classic approach.
2. The chosen peers to store the provider records might not be the actual closest ones. Measurements showed that this
   is not a problem.
3. The optimistic provide process returns already after 15 out of the 20 provider records were stored with peers. The
   reasoning here is that one out of the remaining 5 peers are very likely to time out and delay the whole process. To
   limit the number of in-flight async requests there is the second `OptimisticProvideJobsPoolSize` setting. Currently,
   this is set to 60. This means that at most 60 parallel background requests are allowed to be in-flight. If this
   limit is exceeded optimistic provide will block until all 20 provider records are written. This is still 2x faster
   than the classic approach but not as fast as returning early which yields >10x speed-ups.
4. Since the in-flight background requests are likely to time out, they are not consuming many resources and the job
   pool size could probably be much higher.

For more information, see:

- Project doc: https://protocollabs.notion.site/Optimistic-Provide-2c79745820fa45649d48de038516b814
- go-libp2p-kad-dht: https://github.com/libp2p/go-libp2p-kad-dht/pull/783

### Configuring
To enable:

```go
ipfs config --json Experimental.OptimisticProvide true
```

If you want to change the `OptimisticProvideJobsPoolSize` setting from its default of 60:

```go
ipfs config --json Experimental.OptimisticProvideJobsPoolSize 120
```

### Road to being a real feature

- [ ] Needs more people to use and report on how well it works
- [ ] Should prove at least equivalent availability of provider records as the classic approach

## HTTP Gateway over Libp2p

### In Version

0.23.0

### State

Experimental, disabled by default.

Enables serving a subset of the [IPFS HTTP Gateway](https://specs.ipfs.tech/http-gateways/) semantics over libp2p `/http/1.1` protocol.

Notes:
- This feature only about serving verifiable gateway requests over libp2p:
  - Deserialized responses are not supported.
  - Only operate on `/ipfs` resources (no `/ipns` atm)
  - Only support requests for `application/vnd.ipld.raw` and
    `application/vnd.ipld.car` (from [Trustless Gateway Specification](https://specs.ipfs.tech/http-gateways/trustless-gateway/),
    where data integrity can be verified).
  - Only serve data that is already local to the node (i.e. similar to a
    [`Gateway.NoFetch`](https://github.com/ipfs/kubo/blob/master/docs/config.md#gatewaynofetch))
- While Kubo currently mounts the gateway API at the root (i.e. `/`) of the
  libp2p `/http/1.1` protocol, that is subject to change.
  - The way to reliably discover where a given HTTP protocol is mounted on a
    libp2p endpoint is via the `.well-known/libp2p` resource specified in the
    [http+libp2p specification](https://github.com/libp2p/specs/pull/508)
    - The identifier of the protocol mount point under `/http/1.1` listener is
      `/ipfs/gateway`, as noted in
      [ipfs/specs#434](https://github.com/ipfs/specs/pull/434).

### How to enable

Modify your ipfs config:

```go
ipfs config --json Experimental.GatewayOverLibp2p true
```

### Road to being a real feature

- [ ] Needs more people to use and report on how well it works
- [ ] Needs UX work for exposing non-recursive "HTTP transport" (NoFetch) over both libp2p and plain TCP (and sharing the configuration)
- [ ] Needs a mechanism for HTTP handler to signal supported features ([IPIP-425](https://github.com/ipfs/specs/pull/425))
- [ ] Needs an option for Kubo to detect peers that have it enabled and prefer HTTP transport before falling back to bitswap (and use CAR if peer supports dag-scope=entity from [IPIP-402](https://github.com/ipfs/specs/pull/402))
