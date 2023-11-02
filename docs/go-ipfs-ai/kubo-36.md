# go-ipfs 源码解析 36

# go-ipfs changelog 2022

## v0.13.1 2022-07-06

This release includes security fixes for various DOS vectors when importing untrusted user input with `ipfs dag import`
and the [`v0/dag/import`](https://docs.ipfs.tech/reference/kubo/rpc/#api-v0-dag-import) endpoint.

View the linked [security advisory](https://github.com/ipfs/go-ipfs/security/advisories/GHSA-f2gr-7299-487h) for more information.

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/go-ipfs:
  - chore: update car
- github.com/ipld/go-car (v0.3.2 -> v0.4.0) & (v2.1.1 -> v2.4.0):
  - Bump version in prep for releasing go-car `v0`
  - Revert changes to `insertionindex`
  - Revert changes to `index.Index` while keeping most of security fixes
  - Return error when section length is invalid `varint`
  - Drop repeated package name from `CarStats`
  - Benchmark `Reader.Inspect` with and without hash validation
  - Use consistent CID mismatch error in `Inspect` and `BlockReader.Next`
  - Use streaming APIs to verify the hash of blocks in CAR `Inspect`
  - test: add fuzzing for reader#Inspect
  - feat: add block hash validation to Inspect()
  - feat: add Reader#Inspect() function to check basic validity of a CAR and return stats
  - Remove support for `ForEach` enumeration from car-index-sorted
  - Use a fix code as the multihash code for `CarIndexSorted`
  - Fix testutil assertion logic and update index generation tests
  - fix: tighter constraint of singleWidthIndex width, add index recommendation docs
  - fix: explicitly disable serialization of insertionindex
  - feat: MaxAllowed{Header,Section}Size option
  - feat: MaxAllowedSectionSize default to 32M
  - fix: use CidFromReader() which has overread and OOM protection
  - fix: staticcheck catches
  - fix: revert to internalio.NewOffsetReadSeeker in Reader#IndexReader
  - fix index comparisons
  - feat: Refactor indexes to put storage considerations on consumers
  - test: v2 add fuzzing of the index
  - fix: v2 don't divide by zero in width indexes
  - fix: v2 don't allocate indexes too big
  - test: v2 add fuzzing to Reader
  - fix: v2 don't accept overflowing offsets while reading v2 headers
  - test: v2 add fuzzing to BlockReader
  - fix: v2 don't OOM if the header size is too big
  - test: add fuzzing of NewCarReader
  - fix: do bound check while checking for CIDv0
  - fix: don't OOM if the header size is too big
  - Add API to regenerate index from CARv1 or CARv2
  - PrototypeChooser support (#305) ([ipld/go-car#305](https://github.com/ipld/go-car/pull/305))
  - bump to newer blockstore err not found (#301) ([ipld/go-car#301](https://github.com/ipld/go-car/pull/301))
  - Car command supports for `largebytes` nodes (#296) ([ipld/go-car#296](https://github.com/ipld/go-car/pull/296))
  - fix(test): rootless fixture should have no roots, not null roots
  - Allow extracton of a raw unixfs file (#284) ([ipld/go-car#284](https://github.com/ipld/go-car/pull/284))
  - cmd/car: use a better install command in the README
  - feat: --version selector for `car create` & update deps
  - feat: add option to create blockstore that writes a plain CARv1 (#288) ([ipld/go-car#288](https://github.com/ipld/go-car/pull/288))
  - add `car detach-index list` to list detached index contents (#287) ([ipld/go-car#287](https://github.com/ipld/go-car/pull/287))
  - add `car root` command (#283) ([ipld/go-car#283](https://github.com/ipld/go-car/pull/283))
  - make specification of root cid in get-dag command optional (#281) ([ipld/go-car#281](https://github.com/ipld/go-car/pull/281))
  - Update `version.json` after manual tag push
  - Update v2 to context datastores (#275) ([ipld/go-car#275](https://github.com/ipld/go-car/pull/275))
  - update context datastore ([ipld/go-car#273](https://github.com/ipld/go-car/pull/273))
  - Traversal-based car creation (#269) ([ipld/go-car#269](https://github.com/ipld/go-car/pull/269))
  - Seek to start before index generation in `ReadOnly` blockstore
  - support extraction of unixfs content stored in car files (#263) ([ipld/go-car#263](https://github.com/ipld/go-car/pull/263))
  - Add a bare bones readme to the car CLI (#262) ([ipld/go-car#262](https://github.com/ipld/go-car/pull/262))
  - sync: update CI config files (#261) ([ipld/go-car#261](https://github.com/ipld/go-car/pull/261))
  - fix!: use -version=n instead of -v1 for index command
  - feat: fix get-dag and add version=1 option
  - creation of car from file / directory (#246) ([ipld/go-car#246](https://github.com/ipld/go-car/pull/246))
  - forEach iterates over index in stable order (#258) ([ipld/go-car#258](https://github.com/ipld/go-car/pull/258))
- github.com/multiformats/go-multicodec (v0.4.1 -> v0.5.0):
  - Bump version to 0.5.0
  - Bump version to 0.4.2
  - deps: update stringer version in go generate command
  - docs(readme): improved usage examples (#66) ([multiformats/go-multicodec#66](https://github.com/multiformats/go-multicodec/pull/66))

</details>

### ❤  Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Masih H. Derkani | 27 | +1494/-1446 | 100 |
| Rod Vagg | 31 | +2021/-606 | 105 |
| Will | 19 | +1898/-151 | 69 |
| Jorropo | 27 | +1638/-248 | 76 |
| Aayush Rajasekaran | 1 | +130/-100 | 10 |
| whyrusleeping | 1 | +24/-22 | 4 |
| Marcin Rataj | 1 | +27/-1 | 1 |

## v0.13.0 2022-05-04

We're happy to announce go-ipfs 0.13.0, packed full of changes and improvements!

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [release process](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

- [🛠 BREAKING CHANGES](#---breaking-changes)
  * [`ipfs block put` command](#-ipfs-block-put--command)
  * [`ipfs cid codecs` command](#-ipfs-cid-codecs--command)
  * [`Swarm` configuration](#-swarm--configuration)
  * [Circuit Relay V1 is deprecated](#circuit-relay-v1-is-deprecated)
  * [`ls` requests for `/multistream/1.0.0` are removed](#-ls--requests-for---multistream-100--are-removed)
  * [Gateway Items](#gateway-items)
- [🔦 Highlights](#---highlights)
  * [🧑‍💼 libp2p Network Resource Manager (`Swarm.ResourceMgr`)](#------libp2p-network-resource-manager---swarmresourcemgr--)
  * [🔃 Relay V2 client with auto discovery (`Swarm.RelayClient`)](#---relay-v2-client-with-auto-discovery---swarmrelayclient--)
  * [🌉 HTTP Gateway improvements](#---http-gateway-improvements)
    + [🍱 Support for Block and CAR response formats](#---support-for-block-and-car-response-formats)
    + [🐎 Fast listing generation for huge  directories](#---fast-listing-generation-for-huge--directories)
    + [🎫 Improved `Etag` and `If-None-Match` for bandwidth savings](#---improved--etag--and--if-none-match--for-bandwidth-savings)
    + [⛓️ Added X-Ipfs-Roots for smarter HTTP caches](#---added-x-ipfs-roots-for-smarter-http-caches)
    + [🌡️ Added metrics per response type](#----added-metrics-per-response-type)
  * [🕵️ OpenTelemetry tracing](#----opentelemetry-tracing)
    + [How to use Jaeger UI for visual tracing?](#how-to-use-jaeger-ui-for-visual-tracing-)
  * [🩺 Built-in `ipfs diag profile` to ease debugging](#---built-in--ipfs-diag-profile--to-ease-debugging)
  * [🔑 Support for PEM/PKCS8 for key import/export](#---support-for-pem-pkcs8-for-key-import-export)
  * [🧹 Using standard IPLD codec names across the CLI/HTTP API](#---using-standard-ipld-codec-names-across-the-cli-http-api)
  * [🐳 Custom initialization for Docker](#---custom-initialization-for-docker)
  * [RPC API docs for experimental and deprecated commands](#rpc-api-docs-for-experimental-and-deprecated-commands)
  * [Yamux over Mplex](#yamux-over-mplex)

### 🛠 BREAKING CHANGES

#### `ipfs block put` command

`ipfs block put` command returns a CIDv1 with `raw` codec by default now.
- `ipfs block put --cid-codec` makes `block put` return CID with alternative codec
  - This impacts only the returned CID; it does not trigger any validation or data transformation.
  - Retrieving a block with a different codec or CID version than it was put with is valid.
  - Codec names are validated against tables from [go-multicodec](https://github.com/multiformats/go-multicodec) library.
- `ipfs block put --format` is deprecated. It used incorrect codec names and should be avoided for new deployments. Use it only if you need the old, invalid behavior, namely:
  - `ipfs block put --format=v0` will produce CIDv0 (implicit dag-pb)
  - `ipfs block put --format=cbor` will produce CIDv1 with dag-cbor (!)
  - `ipfs block put --format=protobuf` will produce CIDv1 with dag-pb (!)

#### `ipfs cid codecs` command
- Now lists codecs from [go-multicodec](https://github.com/multiformats/go-multicodec) library.
- `ipfs cid codecs --supported` can be passed to only show codecs supported in various go-ipfs commands.

#### `ipfs cid format` command
- `--codec` was removed and replaced with `--mc` to ensure existing users are aware of the following changes:
  - `--mc protobuf` now correctly points to code `0x50` (was `0x70`, which is `dab-pg`)
  - `--mc cbor` now correctly points to code `0x51` (was `0x71`, which is `dag-cbor`)

#### `Swarm` configuration
- Daemon will refuse to start if long-deprecated RelayV1 config key `Swarm.EnableAutoRelay` or `Swarm.DisableRelay` is set to `true`.
- If `Swarm.Transports.Network.Relay` is disabled,  then `Swarm.RelayService` and `Swarm.RelayClient` are also disabled (unless they have been explicitly enabled).

#### Circuit Relay V1 is deprecated
- By default, `Swarm.RelayClient` does not use Circuit Relay V1. Circuit V1 support is only enabled when `Swarm.RelayClient.StaticRelays` are specified.

#### `ls` requests for `/multistream/1.0.0` are removed
- go-libp2p 0.19 removed support for undocumented `ls` command ([PR](https://github.com/multiformats/go-multistream/pull/76)).  If you are still using it for internal testing, it is time to refactor  ([example](https://github.com/ipfs/go-ipfs/commit/39047bcf61163096d1c965283d671c7c487c9173))

#### Gateway Behavior
Directory listings returned by the HTTP Gateway won't have size column if the directory is bigger than [`Gateway.FastDirIndexThreshold`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#gatewayfastdirindexthreshold) config (default is 100).

To understand the wider context why we made these changes, read *Highlights* below.

### 🔦 Highlights

#### 🧑‍💼 libp2p Network Resource Manager (`Swarm.ResourceMgr`)

*You can now easily bound how much resource usage libp2p consumes!  This aids in protecting nodes from consuming more resources then are available to them.*

The [libp2p Network Resource Manager](https://github.com/libp2p/go-libp2p-resource-manager#readme) is disabled by default, but can be enabled via:

`ipfs config --json Swarm.ResourceMgr.Enabled true`

When enabled, it applies some safe defaults that can be inspected and adjusted with:

- `ipfs swarm stats --help`
- `ipfs swarm limit --help`

User changes persist to config at [`Swarm.ResourceMgr`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmresourcemgr).

The Resource Manager will be enabled by default in a future release.

#### 🔃 Relay V2 client with auto discovery (`Swarm.RelayClient`)

*All the pieces are enabled for [hole-punching](https://blog.ipfs.io/2022-01-20-libp2p-hole-punching/) by default, improving connecting with nodes behind NATs and Firewalls!*

This release enables [`Swarm.RelayClient`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayclient) by default, along with circuit v2 relay discovery provided by go-libp2p [v0.19.0](https://github.com/libp2p/go-libp2p/releases/tag/v0.19.0).  This means:
1. go-ipfs will coordinate with the counterparty using a [relayed connection](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md), to [upgrade to a direct connection](https://github.com/libp2p/specs/blob/master/relay/DCUtR.md) through a NAT/firewall whenever possible.
2. go-ipfs daemon will automatically use [public relays](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayservice) if it detects that it cannot be reached from the public internet (e.g., it's behind a firewall).  This results in a `/p2p-circuit` address from a public relay.

**Notes:**
- [`Swarm.RelayClient`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayclient) does not use Circuit Relay V1 nodes any more. Circuit V1 support is only enabled when static relays are specified in [`Swarm.RelayClient.StaticRelays`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayclientstaticrelays).
- One can opt-out via  [`Swarm.EnableHolePunching`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmenableholepunching).


####  🌉 HTTP Gateway improvements

HTTP Gateway enables seamless interop with the existing Web, clients, user agents, tools, frameworks and libraries.

This release ships the first batch of improvements that enable creation of faster and smarter CDNs, and unblocks creation of light clients for Mobile and IoT.

Details below.

##### 🍱 Support for Block and CAR response formats

*Alternative response formats from Gateway can be requested to avoid needing to trust a gateway.*

For now, `{format}` is limited to two options:
- `raw` –  fetching single block
- `car` – fetching entire DAG behind a CID as a [CARv1 stream](https://ipld.io/specs/transport/car/carv1/)

When not set, the default UnixFS response is returned.

*Why these two formats?* Requesting Block or CAR for `/ipfs/{cid}` allows a client to **use gateways in a trustless fashion**. These types of gateway responses can be verified locally and rejected if digest inside of requested CID does not match received bytes. This enables creation of "light IPFS clients" which use HTTP Gateways as inexpensive transport for [content-addressed](https://docs.ipfs.tech/concepts/content-addressing/) data, unlocking use in Mobile and IoT contexts.


Future releases will [add support for dag-json and dag-cbor responses](https://github.com/ipfs/go-ipfs/issues/8823).

There are two ways for requesting CID specific response format:
1. HTTP header: `Accept: application/vnd.ipld.{format}`
- Examples: [application/vnd.ipld.car](https://www.iana.org/assignments/media-types/application/vnd.ipld.car), [application/vnd.ipld.raw](https://www.iana.org/assignments/media-types/application/vnd.ipld.raw)
2.  URL parameter: `?format=`
-  Useful for creating "Download CAR" links.

*Usage examples:*

1. Downloading a single raw Block and manually importing it to the local datastore:

```goconsole
$ curl  -H 'Accept: application/vnd.ipld.raw' "http://127.0.0.1:8080/ipfs/QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN" --output block.bin
$ cat block.bin | ipfs block put
$ ipfs cat QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
hello
```

2. Downloading entire DAG as a CAR file and importing it:

```goconsole
$ ipfs resolve -r  /ipns/webui.ipfs.io
/ipfs/bafybeiednzu62vskme5wpoj4bjjikeg3xovfpp4t7vxk5ty2jxdi4mv4bu
$ curl  -H 'Accept: application/vnd.ipld.car' "http://127.0.0.1:8080/ipfs/bafybeiednzu62vskme5wpoj4bjjikeg3xovfpp4t7vxk5ty2jxdi4mv4bu" --output webui.car
$ ipfs dag import webui.car
$ ipfs dag stat bafybeiednzu62vskme5wpoj4bjjikeg3xovfpp4t7vxk5ty2jxdi4mv4bu --offline
Size: 27684934, NumBlocks: 394
```

See also:

- [Content Addressable aRchives (CAR / .car) Specifications](https://ipld.io/specs/transport/car/)
- [IANA media type](https://www.iana.org/assignments/media-types/media-types.xhtml) definitions: [application/vnd.ipld.car](https://www.iana.org/assignments/media-types/application/vnd.ipld.car), [application/vnd.ipld.raw](https://www.iana.org/assignments/media-types/application/vnd.ipld.raw)
- [ipfs-car](https://www.npmjs.com/package/ipfs-car) - CLI tool for verifying and unpacking CAR files
- [go-car](https://github.com/ipld/go-car), [js-car](https://github.com/ipld/js-car/) – CAR libraries for GO and JS

##### 🐎 Fast listing generation for huge  directories

*Added [`Gateway.FastDirIndexThreshold`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#gatewayfastdirindexthreshold) configuration, which allows for fast listings of big directories, without the linear slowdown caused by reading size metadata from child nodes.*

As an example, the CID `bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm` represents UnixFS directory with over 10k (10100) of files.

Opening it with go-ipfs 0.12 would require fetching size information of each file, which would take a long long time, most likely causing timeout in the browser or CDN, and introducing unnecessary burden on the gateway node.

go-ipfs 0.13 opens it instantly, because the number of items is bigger than the default [`Gateway.FastDirIndexThreshold`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#gatewayfastdirindexthreshold) and only the root UnixFS node needs to be resolved before the HTML Dir Index is returned to the user.

Notes:
- The default threshold is 100 items.
- Setting to 0 will enable fast listings for all directories.
- CLI users will note that this is equivalent to running `ipfs ls -s --size=false --resolve-type=false /ipfs/bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm`. Now the same speed is available on the gateways.

##### 🎫 Improved `Etag` and `If-None-Match` for bandwidth savings

*Every response type has an unique [`Etag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) which can be used by the client or CDN to save bandwidth, as a gateway does not need to resend a full response if the content was not changed.*

Gateway evaluates Etags sent by a client in [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) and returns status code 304 (Not Modified) on strong or weak match  ([RFC 7232, 2.3](https://datatracker.ietf.org/doc/html/rfc7232#section-2.3)).

##### ⛓️ Added X-Ipfs-Roots for smarter HTTP caches

`X-Ipfs-Roots` is now returned with every Gateway response. It is a way to indicate all CIDs required for resolving path segments from `X-Ipfs-Path`. Together, these two headers are meant to improve interop with existing HTTP software (load-balancers, caches, CDNs).

This additional information allows HTTP caches and CDNs to make better decisions around cache invalidation: not just invalidate everything under specific IPNS website when the root changes, but do more fine-grained cache invalidation by detecting when only a specific subdirectory (branch of a [DAG](https://docs.ipfs.tech/concepts/glossary/#dag)) changes.

##### 🌡️ Added metrics per response type

New metrics can be found at `/debug/metrics/prometheus` on the RPC API port (`127.0.0.1:5001` is the default):

- `gw_first_content_block_get_latency_seconds` – the time until the first content block is received on GET from the gateway (no matter the content or response types)
- `gw_unixfs_file_get_duration_seconds` – the time to serve an entire UnixFS file from the gateway
- `gw_unixfs_gen_dir_listing_get_duration_seconds` – the time to serve a generated UnixFS HTML directory listing from the gateway
- `gw_car_stream_get_duration_seconds` – the time to GET an entire CAR stream from the gateway
- `gw_raw_block_get_duration_seconds` – The time to GET an entire raw Block from the gateway


#### 🕵️ OpenTelemetry tracing

*Opt-in tracing support with many spans for tracing the duration of specific tasks performed by go-ipfs.*

See [Tracing](https://github.com/ipfs/go-ipfs/blob/master/docs/environment-variables.md#tracing) for details.

We will continue to add tracing instrumentation throughout IPFS subcomponents over time.

##### How to use Jaeger UI for visual tracing?

One can use the `jaegertracing/all-in-one` Docker image to run a full Jaeger stack and configure go-ipfs to publish traces to it (here, in an ephemeral container):

```goconsole
$ docker run --rm -it --name jaeger \
    -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
    -p 5775:5775/udp \
    -p 6831:6831/udp \
    -p 6832:6832/udp \
    -p 5778:5778 \
    -p 16686:16686 \
    -p 14268:14268 \
    -p 14250:14250 \
    -p 9411:9411 \
    jaegertracing/all-in-one
```

Then, in other terminal, start go-ipfs with Jaeger tracing enabled:
```go
$ OTEL_TRACES_EXPORTER=jaeger ipfs daemon
```

Finally, the [Jaeger UI](https://github.com/jaegertracing/jaeger-ui#readme) is available at http://localhost:16686

Below are examples of visual tracing for Gateway requests. (Note: this a preview how useful this insight is.  Details may look different now, as we are constantly improving tracing annotations across the go-ipfs codebase.)

| CAR | Block | File | Directory |
| ---- | ---- | ---- | ---- |
| ![2022-04-01_01-46](https://user-images.githubusercontent.com/157609/161167986-951d5c8c-9a5e-464d-bc20-81eb5ccbdc22.png) | ![block_2022-04-01_01-47](https://user-images.githubusercontent.com/157609/161167983-e8cac0ce-0575-4271-8cb8-4d44a0d5d786.png) |  ![2022-04-01_01-49](https://user-images.githubusercontent.com/157609/161167978-e19aa44c-f5a4-45f4-b7c7-14c313ab1dee.png) |  ![dir_2022-04-01_01-48](https://user-images.githubusercontent.com/157609/161167981-456ca52b-3e87-4042-916b-8db149071228.png) |

#### 🩺 Built-in `ipfs diag profile` to ease debugging
The `diag profile` command has been expanded to include all information that was previously included in the `collect-profiles.sh` script, and the script has been removed. Profiles are now collected in parallel, so that profile collection is much faster. Specific profiles can also be selected for targeted debugging.

See `ipfs diag profile --help` for more details.

For general debugging information, see [the debug guide](https://github.com/ipfs/go-ipfs/blob/master/docs/debug-guide.md).

#### 🔑 Support for PEM/PKCS8 for key import/export

It is now possible to import or export private keys wrapped in interoperable [PEM PKCS8](https://en.wikipedia.org/wiki/PKCS_8) by passing `--format=pem-pkcs8-cleartext` to `ipfs key import` and `export` commands.

This improved interop allows for key generation outside of the IPFS node:

```goconsole
$ openssl genpkey -algorithm ED25519 > ed25519.pem
$ ipfs key import test-openssl -f pem-pkcs8-cleartext ed25519.pem
```

Or using external tools like the standard `openssl` to get a PEM file with the public key:
```goconsole
  $ ipfs key export testkey --format=pem-pkcs8-cleartext -o privkey.pem
  $ openssl pkey -in privkey.pem -pubout > pubkey.pem
```

#### 🧹 Using standard IPLD codec names across the CLI/HTTP API

This release makes necessary (breaking) changes in effort to use canonical codec names from [multicodec/table.csv](https://github.com/multiformats/multicodec/blob/master/table.csv). We also switched to CIDv1 in `block put`.  The breaking changes are discussed above.


#### 🐳 Custom initialization for Docker

Docker images published at https://hub.docker.com/r/ipfs/go-ipfs/  now support custom initialization by mounting scripts in the `/container-init.d` directory in the container. Scripts can set custom configuration using `ipfs config`, or otherwise customize the container before the daemon is started.

Scripts are executed sequentially and in lexicographic order, before the IPFS daemon is started and after `ipfs init` is run and the swarm keys are copied (if the IPFS repo needs initialization).

For more information, see:
- Documentation: [ Run IPFS inside Docker](https://docs.ipfs.tech/how-to/run-ipfs-inside-docker/) <!-- TODO: needs https://github.com/ipfs/ipfs-docs/pull/1115/files -->
- Examples in [ipfs-shipyard/go-ipfs-docker-examples](https://github.com/ipfs-shipyard/go-ipfs-docker-examples).

#### RPC API docs for experimental and deprecated commands

https://docs.ipfs.tech/reference/kubo/rpc/ now includes separate sections for _experimental_ and _deprecated_ commands.

We also display a warning in the command line:

```goconsole
$ ipfs name pubsub state --help
WARNING:   EXPERIMENTAL, command may change in future releases
```

#### Yamux over Mplex

The more fully featured yamux stream multiplexer is now prioritized over mplex for outgoing connections.

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/go-ipfs:
  - feat: upgrade to go-libp2p-kad-dht@v0.16.0 (#9005) ([ipfs/go-ipfs#9005](https://github.com/ipfs/go-ipfs/pull/9005))
  - docs: fix typo in the `swarm/peering` help text
  - feat: disable resource manager by default (#9003) ([ipfs/go-ipfs#9003](https://github.com/ipfs/go-ipfs/pull/9003))
  - fix: adjust rcmgr limits for accelerated DHT client rt refresh (#8982) ([ipfs/go-ipfs#8982](https://github.com/ipfs/go-ipfs/pull/8982))
  - fix(ci): make go-ipfs-as-a-library work without external peers (#8978) ([ipfs/go-ipfs#8978](https://github.com/ipfs/go-ipfs/pull/8978))
  - feat: log when resource manager limits are exceeded (#8980) ([ipfs/go-ipfs#8980](https://github.com/ipfs/go-ipfs/pull/8980))
  - fix: JS caching via Access-Control-Expose-Headers (#8984) ([ipfs/go-ipfs#8984](https://github.com/ipfs/go-ipfs/pull/8984))
  - docs: fix abstractions typo
  - fix: hanging goroutine in get fileArchive handler
  - fix(node/libp2p): disable rcmgr checkImplicitDefaults ([ipfs/go-ipfs#8965](https://github.com/ipfs/go-ipfs/pull/8965))
  - pubsub multibase encoding (#8933) ([ipfs/go-ipfs#8933](https://github.com/ipfs/go-ipfs/pull/8933))
  - 'pin rm' helptext: rewrite description as object is not removed from local storage (immediately) ([ipfs/go-ipfs#8947](https://github.com/ipfs/go-ipfs/pull/8947))
  -  ([ipfs/go-ipfs#8934](https://github.com/ipfs/go-ipfs/pull/8934))
  - Add instructions to resolve repo migration error (#8946) ([ipfs/go-ipfs#8946](https://github.com/ipfs/go-ipfs/pull/8946))
  - fix: use path instead of filepath for asset embeds to support Windows
  - Release v0.13.0-rc1
  - docs: v0.13.0 changelog ([ipfs/go-ipfs#8941](https://github.com/ipfs/go-ipfs/pull/8941))
  - chore: build with go 1.18.1 ([ipfs/go-ipfs#8932](https://github.com/ipfs/go-ipfs/pull/8932))
  - docs(tracing): update env var docs for new tracing env vars
  - feat: enable Resource Manager by default
  - chore: Update test/dependencies to match go-ipfs dependencies. (#8928) ([ipfs/go-ipfs#8928](https://github.com/ipfs/go-ipfs/pull/8928))
  - chore: fix linting errors (#8930) ([ipfs/go-ipfs#8930](https://github.com/ipfs/go-ipfs/pull/8930))
  - docs: Swarm.ResourceMgr.Limits
  - feat: EnableHolePunching by default ([ipfs/go-ipfs#8748](https://github.com/ipfs/go-ipfs/pull/8748))
  - ci: add more golang strictness checks ([ipfs/go-ipfs#8931](https://github.com/ipfs/go-ipfs/pull/8931))
  - feat(gateway): Gateway.FastDirIndexThreshold (#8853) ([ipfs/go-ipfs#8853](https://github.com/ipfs/go-ipfs/pull/8853))
  - docs: replace all git.io links with their actual URLs
  - feat: relay v2 discovery (go-libp2p v0.19.0) (#8868) ([ipfs/go-ipfs#8868](https://github.com/ipfs/go-ipfs/pull/8868))
  - fix(cmds): add: reject files with different import dir
  - chore: mark 'log tail' experimental (#8912) ([ipfs/go-ipfs#8912](https://github.com/ipfs/go-ipfs/pull/8912))
  - feat: persist limits to Swarm.ResourceMgr.Limits  (#8901) ([ipfs/go-ipfs#8901](https://github.com/ipfs/go-ipfs/pull/8901))
  - fix: build after Go 1.17 and Prometheus upgrades (#8916) ([ipfs/go-ipfs#8916](https://github.com/ipfs/go-ipfs/pull/8916))
  - feat(tracing): use OpenTelemetry env vars where possible (#8875) ([ipfs/go-ipfs#8875](https://github.com/ipfs/go-ipfs/pull/8875))
  - feat(cmds): allow to set the configuration file path ([ipfs/go-ipfs#8634](https://github.com/ipfs/go-ipfs/pull/8634))
  - chore: deprecate /api/v0/dns (#8893) ([ipfs/go-ipfs#8893](https://github.com/ipfs/go-ipfs/pull/8893))
  - fix(cmds): CIDv1 and correct multicodecs in 'block put' and 'cid codecs' (#8568) ([ipfs/go-ipfs#8568](https://github.com/ipfs/go-ipfs/pull/8568))
  - feat(gw): improved If-None-Match support (#8891) ([ipfs/go-ipfs#8891](https://github.com/ipfs/go-ipfs/pull/8891))
  - Update Go version to 1.17 (#8815) ([ipfs/go-ipfs#8815](https://github.com/ipfs/go-ipfs/pull/8815))
  - chore(gw): extract logical functions to improve readability (#8885) ([ipfs/go-ipfs#8885](https://github.com/ipfs/go-ipfs/pull/8885))
  - feat(docker): /container-init.d for advanced initialization (#6577) ([ipfs/go-ipfs#6577](https://github.com/ipfs/go-ipfs/pull/6577))
  - feat: port collect-profiles.sh to 'ipfs diag profile' (#8786) ([ipfs/go-ipfs#8786](https://github.com/ipfs/go-ipfs/pull/8786))
  - fix: assets: correctly use the argument err in the WalkDirFunc Hashing Files
  - Change `assets.Asset` from a `func` to the embed.FS
  - Remove gobindata
  - fix: fix context plumbing in gateway handlers (#8871) ([ipfs/go-ipfs#8871](https://github.com/ipfs/go-ipfs/pull/8871))
  - fix(gw): missing return if dir fails to finalize (#8806) ([ipfs/go-ipfs#8806](https://github.com/ipfs/go-ipfs/pull/8806))
  - fix(gw): update metrics only when payload data sent (#8827) ([ipfs/go-ipfs#8827](https://github.com/ipfs/go-ipfs/pull/8827))
  - Merge branch 'release'
  - feat: detect changes in go-libp2p-resource-manager (#8857) ([ipfs/go-ipfs#8857](https://github.com/ipfs/go-ipfs/pull/8857))
  - feat: opt-in Swarm.ResourceMgr (go-libp2p v0.18) (#8680) ([ipfs/go-ipfs#8680](https://github.com/ipfs/go-ipfs/pull/8680))
  - feat(cmds): add support for CAR v2 imports (#8854) ([ipfs/go-ipfs#8854](https://github.com/ipfs/go-ipfs/pull/8854))
  - docs(logging): environment variables (#8833) ([ipfs/go-ipfs#8833](https://github.com/ipfs/go-ipfs/pull/8833))
  - fix: unknown fetcher type error (#8830) ([ipfs/go-ipfs#8830](https://github.com/ipfs/go-ipfs/pull/8830))
  - chore: deprecate tar commands (#8849) ([ipfs/go-ipfs#8849](https://github.com/ipfs/go-ipfs/pull/8849))
  - chore: bump go-ipld-format v0.4.0 and fix related sharness tests ([ipfs/go-ipfs#8838](https://github.com/ipfs/go-ipfs/pull/8838))
  - feat: add basic gateway tracing (#8595) ([ipfs/go-ipfs#8595](https://github.com/ipfs/go-ipfs/pull/8595))
  - fix(cli): ipfs add with multiple files of same name (#8493) ([ipfs/go-ipfs#8493](https://github.com/ipfs/go-ipfs/pull/8493))
  - fix(gw): validate requested CAR version (#8835) ([ipfs/go-ipfs#8835](https://github.com/ipfs/go-ipfs/pull/8835))
  - feat: re-enable docker sharness tests (#8808) ([ipfs/go-ipfs#8808](https://github.com/ipfs/go-ipfs/pull/8808))
  - docs: gateway.md (#8825) ([ipfs/go-ipfs#8825](https://github.com/ipfs/go-ipfs/pull/8825))
  - fix(core/commands): do not cache config (#8824) ([ipfs/go-ipfs#8824](https://github.com/ipfs/go-ipfs/pull/8824))
  - remove unused import (#8787) ([ipfs/go-ipfs#8787](https://github.com/ipfs/go-ipfs/pull/8787))
  - feat(cmds): document deprecated RPC API commands (#8802) ([ipfs/go-ipfs#8802](https://github.com/ipfs/go-ipfs/pull/8802))
  - fix(fsrepo): deep merge when setting config ([ipfs/go-ipfs#8793](https://github.com/ipfs/go-ipfs/pull/8793))
  - feat: add gateway histogram metrics (#8443) ([ipfs/go-ipfs#8443](https://github.com/ipfs/go-ipfs/pull/8443))
  - ErrNotFound changes: bubble tagged libraries. ([ipfs/go-ipfs#8803](https://github.com/ipfs/go-ipfs/pull/8803))
  - Fix typos
 ([ipfs/go-ipfs#8757](https://github.com/ipfs/go-ipfs/pull/8757))
  - feat(gateway): Block and CAR response formats (#8758) ([ipfs/go-ipfs#8758](https://github.com/ipfs/go-ipfs/pull/8758))
  - fix: allow ipfs-companion browser extension to access RPC API (#8690) ([ipfs/go-ipfs#8690](https://github.com/ipfs/go-ipfs/pull/8690))
  - fix(core/node): unwrap fx error in node construction ([ipfs/go-ipfs#8638](https://github.com/ipfs/go-ipfs/pull/8638))
  - Update PATCH_RELEASE_TEMPLATE.md
  - feat: add full goroutine stack dump (#8790) ([ipfs/go-ipfs#8790](https://github.com/ipfs/go-ipfs/pull/8790))
  - feat(cmds): extend block size check for dag|block put (#8751) ([ipfs/go-ipfs#8751](https://github.com/ipfs/go-ipfs/pull/8751))
  - feat: add endpoint for enabling block profiling (#8469) ([ipfs/go-ipfs#8469](https://github.com/ipfs/go-ipfs/pull/8469))
  - fix(cmds): option for progress bar in cat/get (#8686) ([ipfs/go-ipfs#8686](https://github.com/ipfs/go-ipfs/pull/8686))
  - docs: note the default reprovider strategy as all (#8603) ([ipfs/go-ipfs#8603](https://github.com/ipfs/go-ipfs/pull/8603))
  - fix: listen on loopback for API and gateway ports in docker-compose.yaml (#8773) ([ipfs/go-ipfs#8773](https://github.com/ipfs/go-ipfs/pull/8773))
  - fix(discovery): fix daemon not starting due to mdns startup failure (#8704) ([ipfs/go-ipfs#8704](https://github.com/ipfs/go-ipfs/pull/8704))
 ([ipfs/go-ipfs#8756](https://github.com/ipfs/go-ipfs/pull/8756))
  - feat: ipfs-webui v2.15 (#8712) ([ipfs/go-ipfs#8712](https://github.com/ipfs/go-ipfs/pull/8712))
  - feat: X-Ipfs-Roots for smarter HTTP caches (#8720) ([ipfs/go-ipfs#8720](https://github.com/ipfs/go-ipfs/pull/8720))
  - chore: add instructions for Chocolatey release
  - fix prioritization of stream muxers ([ipfs/go-ipfs#8750](https://github.com/ipfs/go-ipfs/pull/8750))
  - fix(cmds/keystore): do not allow to import keys we don't generate (#8733) ([ipfs/go-ipfs#8733](https://github.com/ipfs/go-ipfs/pull/8733))
  - docs: add Internal.UnixFSShardingSizeThreshold (#8723) ([ipfs/go-ipfs#8723](https://github.com/ipfs/go-ipfs/pull/8723))
  - feat(cmd): add silent option for repo gc (#7147) ([ipfs/go-ipfs#7147](https://github.com/ipfs/go-ipfs/pull/7147))
  - docs(changelog): update v0.12.0 release notes
  - Merge branch 'release'
  - fix: installation without sudo (#8715) ([ipfs/go-ipfs#8715](https://github.com/ipfs/go-ipfs/pull/8715))
  - Fix typos (#8726) ([ipfs/go-ipfs#8726](https://github.com/ipfs/go-ipfs/pull/8726))
  - fix(build): Recognize Go Beta versions in makefile (#8677) ([ipfs/go-ipfs#8677](https://github.com/ipfs/go-ipfs/pull/8677))
  - chore(cmds): encapsulate ipfs rm logic in another function (#8574) ([ipfs/go-ipfs#8574](https://github.com/ipfs/go-ipfs/pull/8574))
  - feat: warn user when 'pin remote add' while offline (#8621) ([ipfs/go-ipfs#8621](https://github.com/ipfs/go-ipfs/pull/8621))
  - chore(gateway): debug logging for the http requests (#8518) ([ipfs/go-ipfs#8518](https://github.com/ipfs/go-ipfs/pull/8518))
  - docker: build for arm cpu (#8633) ([ipfs/go-ipfs#8633](https://github.com/ipfs/go-ipfs/pull/8633))
  - feat: refactor Fetcher interface used for downloading migrations (#8728) ([ipfs/go-ipfs#8728](https://github.com/ipfs/go-ipfs/pull/8728))
  - feat: log multifetcher errors
  - docs: optionalInteger|String|Duration (#8729) ([ipfs/go-ipfs#8729](https://github.com/ipfs/go-ipfs/pull/8729))
  - feat: DNS.MaxCacheTTL for DNS-over-HTTPS resolvers (#8615) ([ipfs/go-ipfs#8615](https://github.com/ipfs/go-ipfs/pull/8615))
  - feat(cmds): ipfs id: support --offline option (#8626) ([ipfs/go-ipfs#8626](https://github.com/ipfs/go-ipfs/pull/8626))
  - feat(cmds): add cleartext PEM/PKCS8 for key import/export (#8616) ([ipfs/go-ipfs#8616](https://github.com/ipfs/go-ipfs/pull/8616))
  - docs: update badger section in config.md (#8662) ([ipfs/go-ipfs#8662](https://github.com/ipfs/go-ipfs/pull/8662))
  - docs: fix typo
  - Adding PowerShell to Minimal Go Installation
  - Fixed typos in docs/config.md
  - docs: add Snap note about customizing IPFS_PATH (#8584) ([ipfs/go-ipfs#8584](https://github.com/ipfs/go-ipfs/pull/8584))
  - Fix typo ([ipfs/go-ipfs#8625](https://github.com/ipfs/go-ipfs/pull/8625))
  - chore: update version to v0.13.0-dev
- github.com/ipfs/go-bitswap (v0.5.1 -> v0.6.0):
  - v0.6.0
  - Use ipld.ErrNotFound
  - feat: add peer block filter option (#549) ([ipfs/go-bitswap#549](https://github.com/ipfs/go-bitswap/pull/549))
  - configurable target message size ([ipfs/go-bitswap#546](https://github.com/ipfs/go-bitswap/pull/546))
- github.com/ipfs/go-blockservice (v0.2.1 -> v0.3.0):
  - v0.3.0
  - s/log/logger
  - Use ipld.ErrNotFound instead of ErrNotFound
- github.com/ipfs/go-cid (v0.1.0 -> v0.2.0):
  - fix: remove invalid multicodec2string mappings (#137) ([ipfs/go-cid#137](https://github.com/ipfs/go-cid/pull/137))
  - sync: update CI config files (#136) ([ipfs/go-cid#136](https://github.com/ipfs/go-cid/pull/136))
  - Benchmark existing ways to check for `IDENTITY` CIDs
  - avoid double alloc in NewCidV1
  - sync: update CI config files ([ipfs/go-cid#131](https://github.com/ipfs/go-cid/pull/131))
- github.com/ipfs/go-cidutil (v0.0.2 -> v0.1.0):
 ([ipfs/go-cidutil#36](https://github.com/ipfs/go-cidutil/pull/36))
  - sync: update CI config files ([ipfs/go-cidutil#35](https://github.com/ipfs/go-cidutil/pull/35))
  - sync: update CI config files (#34) ([ipfs/go-cidutil#34](https://github.com/ipfs/go-cidutil/pull/34))
  - fix staticcheck ([ipfs/go-cidutil#31](https://github.com/ipfs/go-cidutil/pull/31))
  - add license file so it can be found by go-licenses ([ipfs/go-cidutil#27](https://github.com/ipfs/go-cidutil/pull/27))
  - test: fix for base32 switch ([ipfs/go-cidutil#16](https://github.com/ipfs/go-cidutil/pull/16))
  - doc: add a lead maintainer
- github.com/ipfs/go-filestore (v1.1.0 -> v1.2.0):
  - v1.2.0
  - refactor: follow the happy left practice in Filestore.DeleteBlock
  - Use ipld.ErrNotFound
- github.com/ipfs/go-graphsync (v0.11.0 -> v0.13.1):
  - docs(CHANGELOG): update for v0.13.1
  - feat(ipld): wrap bindnode with panic protection (#368) ([ipfs/go-graphsync#368](https://github.com/ipfs/go-graphsync/pull/368))
  - docs(CHANGELOG): update for v0.13.0 (#366) ([ipfs/go-graphsync#366](https://github.com/ipfs/go-graphsync/pull/366))
  - fix(impl): delete file
  - Minimal alternate metadata type support (#365) ([ipfs/go-graphsync#365](https://github.com/ipfs/go-graphsync/pull/365))
  - Fix unixfs fetch (#364) ([ipfs/go-graphsync#364](https://github.com/ipfs/go-graphsync/pull/364))
  - [Feature] UUIDs, protocol versioning, v2 protocol w/ dag-cbor messaging (#332) ([ipfs/go-graphsync#332](https://github.com/ipfs/go-graphsync/pull/332))
  - feat(CHANGELOG): update for v0.12.0
  - Use do not send blocks for pause/resume & prevent processing of blocks on cancelled requests (#333) ([ipfs/go-graphsync#333](https://github.com/ipfs/go-graphsync/pull/333))
  - Support unixfs reification in default linksystem (#329) ([ipfs/go-graphsync#329](https://github.com/ipfs/go-graphsync/pull/329))
  - Don't run hooks on blocks we didn't have (#331) ([ipfs/go-graphsync#331](https://github.com/ipfs/go-graphsync/pull/331))
  - feat(responsemanager): trace full messages via links to responses (#325) ([ipfs/go-graphsync#325](https://github.com/ipfs/go-graphsync/pull/325))
  - chore(requestmanager): rename processResponses internals for consistency (#328) ([ipfs/go-graphsync#328](https://github.com/ipfs/go-graphsync/pull/328))
  - Response message tracing (#327) ([ipfs/go-graphsync#327](https://github.com/ipfs/go-graphsync/pull/327))
  - fix(testutil): fix tracing span collection (#324) ([ipfs/go-graphsync#324](https://github.com/ipfs/go-graphsync/pull/324))
  - docs(CHANGELOG): update for v0.11.5 release
  - feat(requestmanager): add tracing for response messages & block processing (#322) ([ipfs/go-graphsync#322](https://github.com/ipfs/go-graphsync/pull/322))
  - ipldutil: simplify state synchronization, add docs (#300) ([ipfs/go-graphsync#300](https://github.com/ipfs/go-graphsync/pull/300))
  - docs(CHANGELOG): update for v0.11.4 release
  - Scrub response errors (#320) ([ipfs/go-graphsync#320](https://github.com/ipfs/go-graphsync/pull/320))
  - fix(responsemanager): remove unused maxInProcessRequests parameter (#319) ([ipfs/go-graphsync#319](https://github.com/ipfs/go-graphsync/pull/319))
  - feat(responsemanager): allow ctx augmentation via queued request hook
  - make go test with coverpkg=./...
  - docs(CHANGELOG): update for v0.11.3
  - Merge tag 'v0.10.9'
  - feat: add basic tracing for responses (#291) ([ipfs/go-graphsync#291](https://github.com/ipfs/go-graphsync/pull/291))
  - fix(impl): remove accidental legacy field (#310) ([ipfs/go-graphsync#310](https://github.com/ipfs/go-graphsync/pull/310))
  - docs(CHANGELOG): update for v0.11.2
  - Merge branch 'release/v0.10.8'
  - feat(taskqueue): fix race on peer state gather (#303) ([ipfs/go-graphsync#303](https://github.com/ipfs/go-graphsync/pull/303))
  - feat(responsemanager): clarify response completion (#304) ([ipfs/go-graphsync#304](https://github.com/ipfs/go-graphsync/pull/304))
  - docs(CHANGELOG): update for v0.11.1
  - Merge branch 'release/v0.10.7'
  - Expose task queue diagnostics (#302) ([ipfs/go-graphsync#302](https://github.com/ipfs/go-graphsync/pull/302))
  - chore: short-circuit unnecessary message processing
  - Add a bit of logging (#301) ([ipfs/go-graphsync#301](https://github.com/ipfs/go-graphsync/pull/301))
  - Peer Stats function (#298) ([ipfs/go-graphsync#298](https://github.com/ipfs/go-graphsync/pull/298))
  - fix: use sync.Cond to handle no-task blocking wait (#299) ([ipfs/go-graphsync#299](https://github.com/ipfs/go-graphsync/pull/299))
  - ipldutil: use chooser APIs from dagpb and basicnode (#292) ([ipfs/go-graphsync#292](https://github.com/ipfs/go-graphsync/pull/292))
  - testutil/chaintypes: simplify maintenance of codegen (#294) ([ipfs/go-graphsync#294](https://github.com/ipfs/go-graphsync/pull/294))
  - fix(test): increase 1s timeouts to 2s for slow CI (#289) ([ipfs/go-graphsync#289](https://github.com/ipfs/go-graphsync/pull/289))
  - docs(tests): document tracing test helper utilities
  - feat: add basic OT tracing for incoming requests
  - fix(responsemanager): make fix more global
  - fix(responsemanager): fix flaky tests
  - feat: add WorkerTaskQueue#WaitForNoActiveTasks() for tests (#284) ([ipfs/go-graphsync#284](https://github.com/ipfs/go-graphsync/pull/284))
- github.com/ipfs/go-ipfs-blockstore (v1.1.2 -> v1.2.0):
  - v0.2.0 ([ipfs/go-ipfs-blockstore#98](https://github.com/ipfs/go-ipfs-blockstore/pull/98))
  - s/log/logger
  - Use ipld.ErrNotFound for NotFound errors
- github.com/ipfs/go-ipfs-cmds (v0.6.0 -> v0.8.1):
  - fix(cli/parse): extract dir before name ([ipfs/go-ipfs-cmds#230](https://github.com/ipfs/go-ipfs-cmds/pull/230))
  - Version 0.8.0 ([ipfs/go-ipfs-cmds#228](https://github.com/ipfs/go-ipfs-cmds/pull/228))
  - fix(cli): use NewSliceDirectory for duplicate file paths ([ipfs/go-ipfs-cmds#220](https://github.com/ipfs/go-ipfs-cmds/pull/220))
  - chore: release v0.7.0 ([ipfs/go-ipfs-cmds#227](https://github.com/ipfs/go-ipfs-cmds/pull/227))
  - feat(Command): add status for the helptext ([ipfs/go-ipfs-cmds#225](https://github.com/ipfs/go-ipfs-cmds/pull/225))
  - allow header and set header in client ([ipfs/go-ipfs-cmds#226](https://github.com/ipfs/go-ipfs-cmds/pull/226))
  - sync: update CI config files (#221) ([ipfs/go-ipfs-cmds#221](https://github.com/ipfs/go-ipfs-cmds/pull/221))
  - fix: chanResponseEmitter cancel being ineffective
  - add: tests for postrun execution
  - fix: postrun's run condition in Execute
  - fix: exec deadlock when emitter is not Typer intf
  - sync: update CI config files ([ipfs/go-ipfs-cmds#207](https://github.com/ipfs/go-ipfs-cmds/pull/207))
  - fix: preserve windows file paths ([ipfs/go-ipfs-cmds#214](https://github.com/ipfs/go-ipfs-cmds/pull/214))
  - Resolve `staticcheck` issue in prep for unified CI ([ipfs/go-ipfs-cmds#212](https://github.com/ipfs/go-ipfs-cmds/pull/212))
- github.com/ipfs/go-ipfs-exchange-offline (v0.1.1 -> v0.2.0):
  - v0.2.0
  - Improve NotFound error description
- github.com/ipfs/go-ipfs-files (v0.0.9 -> v0.1.1):
  - Release v0.1.1
  - fix: add dragonfly build option for filewriter flags
  - fix: add freebsd build option for filewriter flags
  - Release v0.1.0
  - docs: fix community CONTRIBUTING.md link (#45) ([ipfs/go-ipfs-files#45](https://github.com/ipfs/go-ipfs-files/pull/45))
  - chore(filewriter): cleanup writes (#43) ([ipfs/go-ipfs-files#43](https://github.com/ipfs/go-ipfs-files/pull/43))
  - sync: update CI config files (#44) ([ipfs/go-ipfs-files#44](https://github.com/ipfs/go-ipfs-files/pull/44))
- github.com/ipfs/go-ipld-format (v0.2.0 -> v0.4.0):
  - chore: release version v0.4.0
  - feat: use new more clearer format in ErrNotFound
  - chore: bump version to 0.3.1
  - fix: make Undef ErrNotFound string consistent with Def version
  - Version 0.3.0
  - ErrNotFound: change error string ([ipfs/go-ipld-format#69](https://github.com/ipfs/go-ipld-format/pull/69))
  - Revert "Revert "Add IsErrNotFound() method"" ([ipfs/go-ipld-format#68](https://github.com/ipfs/go-ipld-format/pull/68))
  - sync: update CI config files (#67) ([ipfs/go-ipld-format#67](https://github.com/ipfs/go-ipld-format/pull/67))
  - ignore statticheck error for EndOfDag ([ipfs/go-ipld-format#62](https://github.com/ipfs/go-ipld-format/pull/62))
  - remove Makefile ([ipfs/go-ipld-format#59](https://github.com/ipfs/go-ipld-format/pull/59))
  - fix staticcheck ([ipfs/go-ipld-format#60](https://github.com/ipfs/go-ipld-format/pull/60))
  - Allowing custom NavigableNode implementations ([ipfs/go-ipld-format#58](https://github.com/ipfs/go-ipld-format/pull/58))
- github.com/ipfs/go-ipld-legacy (v0.1.0 -> v0.1.1):
  - feat(node): add json.Marshaller method ([ipfs/go-ipld-legacy#7](https://github.com/ipfs/go-ipld-legacy/pull/7))
- github.com/ipfs/go-log/v2 (v2.3.0 -> v2.5.1):
  - feat: add logger option to skip a number of stack frames ([ipfs/go-log#132](https://github.com/ipfs/go-log/pull/132))
  - release v2.5.0 (#131) ([ipfs/go-log#131](https://github.com/ipfs/go-log/pull/131))
  - config inspection (#129) ([ipfs/go-log#129](https://github.com/ipfs/go-log/pull/129))
  - release v2.4.0 (#127) ([ipfs/go-log#127](https://github.com/ipfs/go-log/pull/127))
  - sync: update CI config files (#125) ([ipfs/go-log#125](https://github.com/ipfs/go-log/pull/125))
  - fix: cannot call SetPrimaryCore after using a Tee logger ([ipfs/go-log#121](https://github.com/ipfs/go-log/pull/121))
  - Document environment variables ([ipfs/go-log#120](https://github.com/ipfs/go-log/pull/120))
  - sync: update CI config files (#119) ([ipfs/go-log#119](https://github.com/ipfs/go-log/pull/119))
  - Add WithStacktrace utility ([ipfs/go-log#118](https://github.com/ipfs/go-log/pull/118))
  - In addition to StdOut/Err check the outfile for TTYness ([ipfs/go-log#117](https://github.com/ipfs/go-log/pull/117))
- github.com/ipfs/go-merkledag (v0.5.1 -> v0.6.0):
  - v0.6.0
  - Improve ErrNotFound
- github.com/ipfs/go-namesys (v0.4.0 -> v0.5.0):
  - Version 0.5.0
  - fix: CIDv1 error with go-libp2p 0.19 (#32) ([ipfs/go-namesys#32](https://github.com/ipfs/go-namesys/pull/32))
  - feat: add tracing (#30) ([ipfs/go-namesys#30](https://github.com/ipfs/go-namesys/pull/30))
  - fix(publisher): fix garbled code output (#28) ([ipfs/go-namesys#28](https://github.com/ipfs/go-namesys/pull/28))
- github.com/ipfs/go-path (v0.2.1 -> v0.3.0):
  - Release v0.3.0 ([ipfs/go-path#55](https://github.com/ipfs/go-path/pull/55))
  - Resolver: convert to interface. ([ipfs/go-path#53](https://github.com/ipfs/go-path/pull/53))
  - Release v0.2.2 (#52) ([ipfs/go-path#52](https://github.com/ipfs/go-path/pull/52))
  - chore: improve error message for invalid ipfs paths ([ipfs/go-path#51](https://github.com/ipfs/go-path/pull/51))
- github.com/ipfs/go-peertaskqueue (v0.7.0 -> v0.7.1):
  - Add topic inspector ([ipfs/go-peertaskqueue#20](https://github.com/ipfs/go-peertaskqueue/pull/20))
- github.com/ipfs/go-pinning-service-http-client (v0.1.0 -> v0.1.1):
  - chore: release v0.1.1
  - fix: error handling while enumerating pins
  - sync: update CI config files (#15) ([ipfs/go-pinning-service-http-client#15](https://github.com/ipfs/go-pinning-service-http-client/pull/15))
  - Resolve lint issues prior to CI integration
- github.com/ipfs/go-unixfsnode (v1.1.3 -> v1.4.0):
  - 1.4.0 release ([ipfs/go-unixfsnode#29](https://github.com/ipfs/go-unixfsnode/pull/29))
  - Partial file test ([ipfs/go-unixfsnode#26](https://github.com/ipfs/go-unixfsnode/pull/26))
  - Add unixfs to UnixFS path selector tail ([ipfs/go-unixfsnode#28](https://github.com/ipfs/go-unixfsnode/pull/28))
  - release v1.3.0 ([ipfs/go-unixfsnode#25](https://github.com/ipfs/go-unixfsnode/pull/25))
  - add AsLargeBytes support to unixfs files (#24) ([ipfs/go-unixfsnode#24](https://github.com/ipfs/go-unixfsnode/pull/24))
  - fix: add extra test to span the shard/no-shard boundary
  - fix: more Tsize fixes, fix HAMT and make it match go-unixfs output
  - fix: encode Tsize correctly everywhere (using wrapped LinkSystem)
  - docs(version): tag 1.2.0
  - Update deps for ADL selectors ([ipfs/go-unixfsnode#20](https://github.com/ipfs/go-unixfsnode/pull/20))
  - add license (#17) ([ipfs/go-unixfsnode#17](https://github.com/ipfs/go-unixfsnode/pull/17))
  - handle empty files (#15) ([ipfs/go-unixfsnode#15](https://github.com/ipfs/go-unixfsnode/pull/15))
  - Add ADL/single-node-view of a full unixFS file. (#14) ([ipfs/go-unixfsnode#14](https://github.com/ipfs/go-unixfsnode/pull/14))
  - sync: update CI config files (#13) ([ipfs/go-unixfsnode#13](https://github.com/ipfs/go-unixfsnode/pull/13))
  - Add builder for unixfs dags (#12) ([ipfs/go-unixfsnode#12](https://github.com/ipfs/go-unixfsnode/pull/12))
- github.com/ipfs/interface-go-ipfs-core (v0.5.2 -> v0.7.0):
  - refactor(block): CIDv1 and BlockPutSettings CidPrefix (#80) ([ipfs/interface-go-ipfs-core#80](https://github.com/ipfs/interface-go-ipfs-core/pull/80))
  - chore: release v0.6.2
  - fix: use IPLD.ErrNotFound instead of string comparison in tests
  - fix: document error (#74) ([ipfs/interface-go-ipfs-core#74](https://github.com/ipfs/interface-go-ipfs-core/pull/74))
  - version: release 0.6.1
  - v0.6.0
  - Update tests to use ipld.IsNotFound to check for notfound errors
  - sync: update CI config files (#79) ([ipfs/interface-go-ipfs-core#79](https://github.com/ipfs/interface-go-ipfs-core/pull/79))
- github.com/ipld/go-codec-dagpb (v1.3.2 -> v1.4.0):
  - bump to v1.4.0 given that we updated ipld-prime
  - add a decode-then-encode roundtrip fuzzer
  - 1.3.1
  - fix: use protowire for Links bytes decoding
  - delete useless code
  - sync: update CI config files (#33) ([ipld/go-codec-dagpb#33](https://github.com/ipld/go-codec-dagpb/pull/33))
- github.com/ipld/go-ipld-prime (v0.14.2 -> v0.16.0):
  - mark v0.16.0
  - node/bindnode: enforce pointer requirement for nullable maps
  - Implement WalkTransforming traversal (#376) ([ipld/go-ipld-prime#376](https://github.com/ipld/go-ipld-prime/pull/376))
  - docs(datamodel): add comment to LargeBytesNode
  - Add partial-match traversal of large bytes (#375) ([ipld/go-ipld-prime#375](https://github.com/ipld/go-ipld-prime/pull/375))
  - Implement option to start traversals at a path ([ipld/go-ipld-prime#358](https://github.com/ipld/go-ipld-prime/pull/358))
  - add top-level "go value with schema" example
  - Support optional `LargeBytesNode` interface (#372) ([ipld/go-ipld-prime#372](https://github.com/ipld/go-ipld-prime/pull/372))
  - node/bindnode: support pointers to datamodel.Node to bind with Any
  - fix(bindnode): tuple struct iterator should handle absent fields properly
  - node/bindnode: make AssignNode work at the repr level
  - node/bindnode: add support for unsigned integers
  - node/bindnode: cover even more edge case panics
  - node/bindnode: polish some more AsT panics
  - schema/dmt: stop using a fake test to generate code ([ipld/go-ipld-prime#356](https://github.com/ipld/go-ipld-prime/pull/356))
  - schema: remove one review note; add another.
  - fix: minor EncodedLength fixes, add tests to fully exercise
  - feat: add dagcbor.EncodedLength(Node) to calculate length without encoding
  - chore: rename Garbage() to Generate()
  - fix: minor garbage nits
  - fix: Garbage() takes rand parameter, tweak algorithms, improve docs
  - feat: add Garbage() Node generator
  - node/bindnode: introduce an assembler that always errors
  - node/bindnode: polish panics on invalid AssignT calls
  - datamodel: don't panic when stringifying an empty KindSet
  - node/bindnode: start using ipld.LoadSchema APIs
  - selectors: fix for edge case around recursion clauses with an immediate edge. ([ipld/go-ipld-prime#334](https://github.com/ipld/go-ipld-prime/pull/334))
  - node/bindnode: improve support for pointer types
  - node/bindnode: subtract all absents in Length at the repr level
  - fix(codecs): error when encoding maps whose lengths don't match entry count
  - schema: avoid alloc and copy in Struct and Enum methods
  - node/bindnode: allow mapping int-repr enums with Go integers
  - schema,node/bindnode: add support for Any
  - signaling ADLs in selectors (#301) ([ipld/go-ipld-prime#301](https://github.com/ipld/go-ipld-prime/pull/301))
  - node/bindnode: add support for enums
  - schema/...: add support for enum int representations
  - node/bindnode: allow binding cidlink.Link to links
  - Update to context datastores (#312) ([ipld/go-ipld-prime#312](https://github.com/ipld/go-ipld-prime/pull/312))
  - schema: add support for struct tuple reprs
  - Allow parsing padding in dag-json bytes fields (#309) ([ipld/go-ipld-prime#309](https://github.com/ipld/go-ipld-prime/pull/309))
- github.com/libp2p/go-doh-resolver (v0.3.1 -> v0.4.0):
  - Release v0.4.0 (#16) ([libp2p/go-doh-resolver#16](https://github.com/libp2p/go-doh-resolver/pull/16))
  - sync: update CI config files (#14) ([libp2p/go-doh-resolver#14](https://github.com/libp2p/go-doh-resolver/pull/14))
  - Add a max TTL for cached entries ([libp2p/go-doh-resolver#12](https://github.com/libp2p/go-doh-resolver/pull/12))
  - Perform test locally instead of using a live dns resolution ([libp2p/go-doh-resolver#13](https://github.com/libp2p/go-doh-resolver/pull/13))
  - sync: update CI config files (#7) ([libp2p/go-doh-resolver#7](https://github.com/libp2p/go-doh-resolver/pull/7))
  - fix staticcheck ([libp2p/go-doh-resolver#6](https://github.com/libp2p/go-doh-resolver/pull/6))
- github.com/libp2p/go-libp2p (v0.16.0 -> v0.19.4):
  - update go-yamux to v3.1.2, release v0.19.4 (#1590) ([libp2p/go-libp2p#1590](https://github.com/libp2p/go-libp2p/pull/1590))
  - update quic-go to v0.27.1, release v0.19.3 (#1518) ([libp2p/go-libp2p#1518](https://github.com/libp2p/go-libp2p/pull/1518))
  - release v0.19.2
  - holepunch: fix incorrect message type for the SYNC message (#1478) ([libp2p/go-libp2p#1478](https://github.com/libp2p/go-libp2p/pull/1478))
  - fix race condition in holepunch service, release v0.19.1 ([libp2p/go-libp2p#1474](https://github.com/libp2p/go-libp2p/pull/1474))
  - release v0.19.0 (#1408) ([libp2p/go-libp2p#1408](https://github.com/libp2p/go-libp2p/pull/1408))
  - Close resource manager when host closes (#1343) ([libp2p/go-libp2p#1343](https://github.com/libp2p/go-libp2p/pull/1343))
  - fix flaky reconnect test (#1406) ([libp2p/go-libp2p#1406](https://github.com/libp2p/go-libp2p/pull/1406))
  - make sure to not oversubscribe to relays (#1404) ([libp2p/go-libp2p#1404](https://github.com/libp2p/go-libp2p/pull/1404))
  - rewrite the reconnect test (#1399) ([libp2p/go-libp2p#1399](https://github.com/libp2p/go-libp2p/pull/1399))
  - don't try to reconnect to already connected relays (#1401) ([libp2p/go-libp2p#1401](https://github.com/libp2p/go-libp2p/pull/1401))
  - reduce flakiness of AutoRelay TestBackoff test (#1400) ([libp2p/go-libp2p#1400](https://github.com/libp2p/go-libp2p/pull/1400))
  - improve AutoRelay v1 handling ([libp2p/go-libp2p#1396](https://github.com/libp2p/go-libp2p/pull/1396))
  - remove note about gx from README (#1385) ([libp2p/go-libp2p#1385](https://github.com/libp2p/go-libp2p/pull/1385))
  - use the vcs information from ReadBuildInfo in Go 1.18 ([libp2p/go-libp2p#1381](https://github.com/libp2p/go-libp2p/pull/1381))
  - fix race condition in AutoRelay candidate handling (#1383) ([libp2p/go-libp2p#1383](https://github.com/libp2p/go-libp2p/pull/1383))
  - implement relay v2 discovery ([libp2p/go-libp2p#1368](https://github.com/libp2p/go-libp2p/pull/1368))
  - fix go vet error in proxy example (#1377) ([libp2p/go-libp2p#1377](https://github.com/libp2p/go-libp2p/pull/1377))
  - Resolve addresses when creating a new stream (#1342) ([libp2p/go-libp2p#1342](https://github.com/libp2p/go-libp2p/pull/1342))
  - remove mplex from the list of default muxers (#1344) ([libp2p/go-libp2p#1344](https://github.com/libp2p/go-libp2p/pull/1344))
  - refactor the holepunching code ([libp2p/go-libp2p#1355](https://github.com/libp2p/go-libp2p/pull/1355))
  - speed up the connmgr tests (#1354) ([libp2p/go-libp2p#1354](https://github.com/libp2p/go-libp2p/pull/1354))
  - update go-libp2p-resource manager, release v0.18.0 (#1361) ([libp2p/go-libp2p#1361](https://github.com/libp2p/go-libp2p/pull/1361))
  - fix flaky BackoffConnector test (#1353) ([libp2p/go-libp2p#1353](https://github.com/libp2p/go-libp2p/pull/1353))
  - release v0.18.0-rc6 (#1350) ([libp2p/go-libp2p#1350](https://github.com/libp2p/go-libp2p/pull/1350))
  - release v0.18.0-rc5 ([libp2p/go-libp2p#1341](https://github.com/libp2p/go-libp2p/pull/1341))
  - update README (#1330) ([libp2p/go-libp2p#1330](https://github.com/libp2p/go-libp2p/pull/1330))
  - fix parsing of IP addresses for zeroconf initialization (#1338) ([libp2p/go-libp2p#1338](https://github.com/libp2p/go-libp2p/pull/1338))
  - fix flaky TestBackoffConnector test (#1328) ([libp2p/go-libp2p#1328](https://github.com/libp2p/go-libp2p/pull/1328))
  - release v0.18.0-rc4 ([libp2p/go-libp2p#1327](https://github.com/libp2p/go-libp2p/pull/1327))
  - fix (and speed up) flaky TestBackoffConnector test (#1316) ([libp2p/go-libp2p#1316](https://github.com/libp2p/go-libp2p/pull/1316))
  - fix flaky TestAutoRelay test (#1322) ([libp2p/go-libp2p#1322](https://github.com/libp2p/go-libp2p/pull/1322))
  - deflake resource manager tests, take 2 ([libp2p/go-libp2p#1318](https://github.com/libp2p/go-libp2p/pull/1318))
  - fix race condition causing TestAutoNATServiceDialError test failure (#1312) ([libp2p/go-libp2p#1312](https://github.com/libp2p/go-libp2p/pull/1312))
  - disable flaky relay example test on CI (#1219) ([libp2p/go-libp2p#1219](https://github.com/libp2p/go-libp2p/pull/1219))
  - fix flaky resource manager tests ([libp2p/go-libp2p#1315](https://github.com/libp2p/go-libp2p/pull/1315))
  - update deps, fixing nil peer scope pointer issues in connection upgrading ([libp2p/go-libp2p#1309](https://github.com/libp2p/go-libp2p/pull/1309))
  - release v0.18.0-rc2 ([libp2p/go-libp2p#1306](https://github.com/libp2p/go-libp2p/pull/1306))
  - add semaphore to control push/delta concurrency ([libp2p/go-libp2p#1305](https://github.com/libp2p/go-libp2p/pull/1305))
  - release v0.18.0-rc1 ([libp2p/go-libp2p#1300](https://github.com/libp2p/go-libp2p/pull/1300))
  - default connection manager ([libp2p/go-libp2p#1299](https://github.com/libp2p/go-libp2p/pull/1299))
  - Basic resource manager integration tests ([libp2p/go-libp2p#1296](https://github.com/libp2p/go-libp2p/pull/1296))
  - use the resource manager ([libp2p/go-libp2p#1275](https://github.com/libp2p/go-libp2p/pull/1275))
  - move the go-libp2p-connmgr here ([libp2p/go-libp2p#1297](https://github.com/libp2p/go-libp2p/pull/1297))
  - move go-libp2p-discovery here ([libp2p/go-libp2p#1291](https://github.com/libp2p/go-libp2p/pull/1291))
  - speed up identify tests ([libp2p/go-libp2p#1294](https://github.com/libp2p/go-libp2p/pull/1294))
  - don't close the connection when opening the identify stream fails ([libp2p/go-libp2p#1293](https://github.com/libp2p/go-libp2p/pull/1293))
  - use the netutil package that was moved to go-libp2p-testing (#1263) ([libp2p/go-libp2p#1263](https://github.com/libp2p/go-libp2p/pull/1263))
  - speed up the autorelay test, fix flaky TestAutoRelay test ([libp2p/go-libp2p#1272](https://github.com/libp2p/go-libp2p/pull/1272))
  - fix flaky TestStreamsStress test (#1288) ([libp2p/go-libp2p#1288](https://github.com/libp2p/go-libp2p/pull/1288))
  - add an option for the swarm dial timeout ([libp2p/go-libp2p#1271](https://github.com/libp2p/go-libp2p/pull/1271))
  - use the transport.Upgrader interface ([libp2p/go-libp2p#1277](https://github.com/libp2p/go-libp2p/pull/1277))
  - fix typo in options.go (#1274) ([libp2p/go-libp2p#1274](https://github.com/libp2p/go-libp2p/pull/1274))
  - remove direct dependency on libp2p/go-addr-util ([libp2p/go-libp2p#1279](https://github.com/libp2p/go-libp2p/pull/1279))
  - fix flaky TestNotifications test ([libp2p/go-libp2p#1278](https://github.com/libp2p/go-libp2p/pull/1278))
  - move go-libp2p-autonat to p2p/host/autonat ([libp2p/go-libp2p#1273](https://github.com/libp2p/go-libp2p/pull/1273))
  - require the expiration field of the circuit v2 Reservation protobuf ([libp2p/go-libp2p#1269](https://github.com/libp2p/go-libp2p/pull/1269))
  - run reconnect test using QUIC ([libp2p/go-libp2p#1268](https://github.com/libp2p/go-libp2p/pull/1268))
  - remove goprocess from the mock package ([libp2p/go-libp2p#1266](https://github.com/libp2p/go-libp2p/pull/1266))
  - release v0.17.0 (#1265) ([libp2p/go-libp2p#1265](https://github.com/libp2p/go-libp2p/pull/1265))
  - use the new network.ConnStats ([libp2p/go-libp2p#1262](https://github.com/libp2p/go-libp2p/pull/1262))
  - move the peerstoremanager to the host ([libp2p/go-libp2p#1258](https://github.com/libp2p/go-libp2p/pull/1258))
  - reduce the default stream protocol negotiation timeout (#1254) ([libp2p/go-libp2p#1254](https://github.com/libp2p/go-libp2p/pull/1254))
  - Doc: QUIC is default when no transports set (#1250) ([libp2p/go-libp2p#1250](https://github.com/libp2p/go-libp2p/pull/1250))
  - exclude web3-bot from mkreleaselog ([libp2p/go-libp2p#1248](https://github.com/libp2p/go-libp2p/pull/1248))
  - identify: also match observed against listening addresses ([libp2p/go-libp2p#1255](https://github.com/libp2p/go-libp2p/pull/1255))
  - make it possible to run the auto relays tests multiple times ([libp2p/go-libp2p#1253](https://github.com/libp2p/go-libp2p/pull/1253))
- github.com/libp2p/go-libp2p-asn-util (v0.1.0 -> v0.2.0):
  - Release 0.2.0 (#21) ([libp2p/go-libp2p-asn-util#21](https://github.com/libp2p/go-libp2p-asn-util/pull/21))
  - perf: replace the ipv6 map by an array of struct (#20) ([libp2p/go-libp2p-asn-util#20](https://github.com/libp2p/go-libp2p-asn-util/pull/20))
  - sync: update CI config files (#18) ([libp2p/go-libp2p-asn-util#18](https://github.com/libp2p/go-libp2p-asn-util/pull/18))
- github.com/libp2p/go-libp2p-blankhost (v0.2.0 -> v0.3.0):
  - add a WithEventBus constructor option ([libp2p/go-libp2p-blankhost#61](https://github.com/libp2p/go-libp2p-blankhost/pull/61))
  - emit the EvtPeerConnectednessChanged event ([libp2p/go-libp2p-blankhost#58](https://github.com/libp2p/go-libp2p-blankhost/pull/58))
  - chore: update go-log to v2 ([libp2p/go-libp2p-blankhost#59](https://github.com/libp2p/go-libp2p-blankhost/pull/59))
  - Remove invalid links ([libp2p/go-libp2p-blankhost#57](https://github.com/libp2p/go-libp2p-blankhost/pull/57))
  - fix go vet ([libp2p/go-libp2p-blankhost#53](https://github.com/libp2p/go-libp2p-blankhost/pull/53))
- github.com/libp2p/go-libp2p-circuit (v0.4.0 -> v0.6.0):
  - release v0.6.0 (#151) ([libp2p/go-libp2p-circuit#151](https://github.com/libp2p/go-libp2p-circuit/pull/151))
  - chore: update go-log to v2 (#147) ([libp2p/go-libp2p-circuit#147](https://github.com/libp2p/go-libp2p-circuit/pull/147))
  - release v0.5.0 (#150) ([libp2p/go-libp2p-circuit#150](https://github.com/libp2p/go-libp2p-circuit/pull/150))
  - use the resource manager ([libp2p/go-libp2p-circuit#148](https://github.com/libp2p/go-libp2p-circuit/pull/148))
  - use the transport.Upgrader interface ([libp2p/go-libp2p-circuit#149](https://github.com/libp2p/go-libp2p-circuit/pull/149))
  - sync: update CI config files (#144) ([libp2p/go-libp2p-circuit#144](https://github.com/libp2p/go-libp2p-circuit/pull/144))
  -  ([libp2p/go-libp2p-circuit#143](https://github.com/libp2p/go-libp2p-circuit/pull/143))
  - add a Close method, remove the context from the constructor ([libp2p/go-libp2p-circuit#141](https://github.com/libp2p/go-libp2p-circuit/pull/141))
  - chore: update go-libp2p-core, go-libp2p-swarm ([libp2p/go-libp2p-circuit#140](https://github.com/libp2p/go-libp2p-circuit/pull/140))
  - remove the circuit v2 code ([libp2p/go-libp2p-circuit#139](https://github.com/libp2p/go-libp2p-circuit/pull/139))
  - implement circuit v2 ([libp2p/go-libp2p-circuit#136](https://github.com/libp2p/go-libp2p-circuit/pull/136))
  - remove deprecated types ([libp2p/go-libp2p-circuit#135](https://github.com/libp2p/go-libp2p-circuit/pull/135))
  - fix race condition in TestActiveRelay ([libp2p/go-libp2p-circuit#133](https://github.com/libp2p/go-libp2p-circuit/pull/133))
  - minor staticcheck fixes ([libp2p/go-libp2p-circuit#126](https://github.com/libp2p/go-libp2p-circuit/pull/126))
  - Timeout Stream Read ([libp2p/go-libp2p-circuit#124](https://github.com/libp2p/go-libp2p-circuit/pull/124))
- github.com/libp2p/go-libp2p-core (v0.11.0 -> v0.15.1):
  - release v0.15.1 (#246) ([libp2p/go-libp2p-core#246](https://github.com/libp2p/go-libp2p-core/pull/246))
  - feat: harden encoding/decoding functions against panics (#243) ([libp2p/go-libp2p-core#243](https://github.com/libp2p/go-libp2p-core/pull/243))
  - release v0.15.0 (#242) ([libp2p/go-libp2p-core#242](https://github.com/libp2p/go-libp2p-core/pull/242))
  - sync: update CI config files (#241) ([libp2p/go-libp2p-core#241](https://github.com/libp2p/go-libp2p-core/pull/241))
  - fix: switch to go-multicodec mappings (#240) ([libp2p/go-libp2p-core#240](https://github.com/libp2p/go-libp2p-core/pull/240))
  - chore: add `String()` method to `IDSlice` type (#238) ([libp2p/go-libp2p-core#238](https://github.com/libp2p/go-libp2p-core/pull/238))
  - release v0.14.0 (#235) ([libp2p/go-libp2p-core#235](https://github.com/libp2p/go-libp2p-core/pull/235))
  - Network Resource Manager interface (#229) ([libp2p/go-libp2p-core#229](https://github.com/libp2p/go-libp2p-core/pull/229))
  - introduce a transport.Upgrader interface (#232) ([libp2p/go-libp2p-core#232](https://github.com/libp2p/go-libp2p-core/pull/232))
  - remove the transport.AcceptTimeout (#231) ([libp2p/go-libp2p-core#231](https://github.com/libp2p/go-libp2p-core/pull/231))
  - remove the DialTimeout (#230) ([libp2p/go-libp2p-core#230](https://github.com/libp2p/go-libp2p-core/pull/230))
  - remove duplicate io.Closer on Network interface (#228) ([libp2p/go-libp2p-core#228](https://github.com/libp2p/go-libp2p-core/pull/228))
  - release v0.13.0 (#227) ([libp2p/go-libp2p-core#227](https://github.com/libp2p/go-libp2p-core/pull/227))
  - rename network.Stat to Stats, introduce ConnStats (#226) ([libp2p/go-libp2p-core#226](https://github.com/libp2p/go-libp2p-core/pull/226))
  - release v0.12.0 (#223) ([libp2p/go-libp2p-core#223](https://github.com/libp2p/go-libp2p-core/pull/223))
  - generate ecdsa public key from an input public key (#219) ([libp2p/go-libp2p-core#219](https://github.com/libp2p/go-libp2p-core/pull/219))
  - add RemovePeer method to PeerMetadata, Metrics, ProtoBook and Keybook (#218) ([libp2p/go-libp2p-core#218](https://github.com/libp2p/go-libp2p-core/pull/218))
- github.com/libp2p/go-libp2p-kad-dht (v0.15.0 -> v0.16.0):
  - Version 0.16.0 (#774) ([libp2p/go-libp2p-kad-dht#774](https://github.com/libp2p/go-libp2p-kad-dht/pull/774))
  - feat: add error log when resource manager throttles crawler (#772) ([libp2p/go-libp2p-kad-dht#772](https://github.com/libp2p/go-libp2p-kad-dht/pull/772))
  - fix: incorrect format handling ([libp2p/go-libp2p-kad-dht#771](https://github.com/libp2p/go-libp2p-kad-dht/pull/771))
  - Upgrade to go-libp2p v0.16.0 (#756) ([libp2p/go-libp2p-kad-dht#756](https://github.com/libp2p/go-libp2p-kad-dht/pull/756))
  - sync: update CI config files ([libp2p/go-libp2p-kad-dht#758](https://github.com/libp2p/go-libp2p-kad-dht/pull/758))
- github.com/libp2p/go-libp2p-mplex (v0.4.1 -> v0.7.0):
  - release v0.7.0 (#36) ([libp2p/go-libp2p-mplex#36](https://github.com/libp2p/go-libp2p-mplex/pull/36))
  - release v0.6.0 (#32) ([libp2p/go-libp2p-mplex#32](https://github.com/libp2p/go-libp2p-mplex/pull/32))
  - update mplex (#31) ([libp2p/go-libp2p-mplex#31](https://github.com/libp2p/go-libp2p-mplex/pull/31))
  - release v0.5.0 (#30) ([libp2p/go-libp2p-mplex#30](https://github.com/libp2p/go-libp2p-mplex/pull/30))
  - implement the new network.MuxedConn interface (#29) ([libp2p/go-libp2p-mplex#29](https://github.com/libp2p/go-libp2p-mplex/pull/29))
  - sync: update CI config files (#28) ([libp2p/go-libp2p-mplex#28](https://github.com/libp2p/go-libp2p-mplex/pull/28))
  - remove Makefile ([libp2p/go-libp2p-mplex#25](https://github.com/libp2p/go-libp2p-mplex/pull/25))
- github.com/libp2p/go-libp2p-noise (v0.3.0 -> v0.4.0):
  - release v0.4.0 (#112) ([libp2p/go-libp2p-noise#112](https://github.com/libp2p/go-libp2p-noise/pull/112))
  - catch panics during the handshake (#111) ([libp2p/go-libp2p-noise#111](https://github.com/libp2p/go-libp2p-noise/pull/111))
  - sync: update CI config files (#106) ([libp2p/go-libp2p-noise#106](https://github.com/libp2p/go-libp2p-noise/pull/106))
  - update README to reflect that Noise is enabled by default ([libp2p/go-libp2p-noise#101](https://github.com/libp2p/go-libp2p-noise/pull/101))
- github.com/libp2p/go-libp2p-peerstore (v0.4.0 -> v0.6.0):
  - release v0.6.0 ([libp2p/go-libp2p-peerstore#189](https://github.com/libp2p/go-libp2p-peerstore/pull/189))
  - remove the pstoremanager (will be moved to the Host) ([libp2p/go-libp2p-peerstore#188](https://github.com/libp2p/go-libp2p-peerstore/pull/188))
  - release v0.5.0 (#187) ([libp2p/go-libp2p-peerstore#187](https://github.com/libp2p/go-libp2p-peerstore/pull/187))
  - remove metadata interning ([libp2p/go-libp2p-peerstore#185](https://github.com/libp2p/go-libp2p-peerstore/pull/185))
  - when passed an event bus, automatically clean up disconnected peers ([libp2p/go-libp2p-peerstore#184](https://github.com/libp2p/go-libp2p-peerstore/pull/184))
  - implement the RemovePeer method ([libp2p/go-libp2p-peerstore#174](https://github.com/libp2p/go-libp2p-peerstore/pull/174))
  - chore: update go-log to v2 (#179) ([libp2p/go-libp2p-peerstore#179](https://github.com/libp2p/go-libp2p-peerstore/pull/179))
- github.com/libp2p/go-libp2p-pubsub (v0.6.0 -> v0.6.1):
  - add tests for clearing the peerPromises map
  - properly clear the peerPromises map
  - more info
  - add to MinTopicSize godoc re topic size
- github.com/libp2p/go-libp2p-quic-transport (v0.15.0 -> v0.17.0):
  - release v0.17.0 (#269) ([libp2p/go-libp2p-quic-transport#269](https://github.com/libp2p/go-libp2p-quic-transport/pull/269))
  - update quic-go to v0.27.0 (#264) ([libp2p/go-libp2p-quic-transport#264](https://github.com/libp2p/go-libp2p-quic-transport/pull/264))
  - release v0.16.1 (#261) ([libp2p/go-libp2p-quic-transport#261](https://github.com/libp2p/go-libp2p-quic-transport/pull/261))
  - Prevent data race in allowWindowIncrease (#259) ([libp2p/go-libp2p-quic-transport#259](https://github.com/libp2p/go-libp2p-quic-transport/pull/259))
  - release v0.16.0 (#258) ([libp2p/go-libp2p-quic-transport#258](https://github.com/libp2p/go-libp2p-quic-transport/pull/258))
  - use the Resource Manager ([libp2p/go-libp2p-quic-transport#249](https://github.com/libp2p/go-libp2p-quic-transport/pull/249))
  - don't start a Go routine for every connection dialed ([libp2p/go-libp2p-quic-transport#252](https://github.com/libp2p/go-libp2p-quic-transport/pull/252))
  - migrate to standard Go tests, stop using Ginkgo ([libp2p/go-libp2p-quic-transport#250](https://github.com/libp2p/go-libp2p-quic-transport/pull/250))
  - chore: remove Codecov config (#251) ([libp2p/go-libp2p-quic-transport#251](https://github.com/libp2p/go-libp2p-quic-transport/pull/251))
  - reduce the maximum number of incoming streams to 256 (#243) ([libp2p/go-libp2p-quic-transport#243](https://github.com/libp2p/go-libp2p-quic-transport/pull/243))
  - chore: update go-log to v2 (#242) ([libp2p/go-libp2p-quic-transport#242](https://github.com/libp2p/go-libp2p-quic-transport/pull/242))
- github.com/libp2p/go-libp2p-swarm (v0.8.0 -> v0.10.2):
  - bump version to v0.10.2 ([libp2p/go-libp2p-swarm#316](https://github.com/libp2p/go-libp2p-swarm/pull/316))
  - Refactor dial worker loop into an object and fix bug ([libp2p/go-libp2p-swarm#315](https://github.com/libp2p/go-libp2p-swarm/pull/315))
  - release v0.10.1 ([libp2p/go-libp2p-swarm#313](https://github.com/libp2p/go-libp2p-swarm/pull/313))
  - release the stream scope if the conn fails to open a new stream ([libp2p/go-libp2p-swarm#312](https://github.com/libp2p/go-libp2p-swarm/pull/312))
  - release v0.10.0 (#311) ([libp2p/go-libp2p-swarm#311](https://github.com/libp2p/go-libp2p-swarm/pull/311))
  - add support for the resource manager ([libp2p/go-libp2p-swarm#308](https://github.com/libp2p/go-libp2p-swarm/pull/308))
  - use the transport.Upgrader interface ([libp2p/go-libp2p-swarm#309](https://github.com/libp2p/go-libp2p-swarm/pull/309))
  - remove dependency on go-addr-util ([libp2p/go-libp2p-swarm#300](https://github.com/libp2p/go-libp2p-swarm/pull/300))
  - stop using transport.DialTimeout in tests (#307) ([libp2p/go-libp2p-swarm#307](https://github.com/libp2p/go-libp2p-swarm/pull/307))
  - increment active dial counter in dial worker loop ([libp2p/go-libp2p-swarm#305](https://github.com/libp2p/go-libp2p-swarm/pull/305))
  - speed up the dial tests ([libp2p/go-libp2p-swarm#301](https://github.com/libp2p/go-libp2p-swarm/pull/301))
  - stop using the deprecated libp2p/go-maddr-filter ([libp2p/go-libp2p-swarm#303](https://github.com/libp2p/go-libp2p-swarm/pull/303))
  - add constructor options for timeout, stop using transport.DialTimeout ([libp2p/go-libp2p-swarm#302](https://github.com/libp2p/go-libp2p-swarm/pull/302))
  - release v0.9.0 (#299) ([libp2p/go-libp2p-swarm#299](https://github.com/libp2p/go-libp2p-swarm/pull/299))
  - count the number of streams on a connection for the stats ([libp2p/go-libp2p-swarm#298](https://github.com/libp2p/go-libp2p-swarm/pull/298))
  - chore: update go-log to v2 (#294) ([libp2p/go-libp2p-swarm#294](https://github.com/libp2p/go-libp2p-swarm/pull/294))
- github.com/libp2p/go-libp2p-testing (v0.5.0 -> v0.9.2):
  - release v0.9.2 (#56) ([libp2p/go-libp2p-testing#56](https://github.com/libp2p/go-libp2p-testing/pull/56))
  - fix memory allocation check in SubtestStreamReset (#55) ([libp2p/go-libp2p-testing#55](https://github.com/libp2p/go-libp2p-testing/pull/55))
  - release v0.9.1 (#54) ([libp2p/go-libp2p-testing#54](https://github.com/libp2p/go-libp2p-testing/pull/54))
  - remove stray debug statements for memory allocations
  - release v0.9.0 (#53) ([libp2p/go-libp2p-testing#53](https://github.com/libp2p/go-libp2p-testing/pull/53))
  - add tests for memory management ([libp2p/go-libp2p-testing#52](https://github.com/libp2p/go-libp2p-testing/pull/52))
  - release v0.8.0 (#50) ([libp2p/go-libp2p-testing#50](https://github.com/libp2p/go-libp2p-testing/pull/50))
  - use io.ReadFull in muxer test, use require.Equal to compare buffers (#49) ([libp2p/go-libp2p-testing#49](https://github.com/libp2p/go-libp2p-testing/pull/49))
  - release v0.7.0 (#47) ([libp2p/go-libp2p-testing#47](https://github.com/libp2p/go-libp2p-testing/pull/47))
  - add mocks for the resource manager ([libp2p/go-libp2p-testing#46](https://github.com/libp2p/go-libp2p-testing/pull/46))
  - merge libp2p/go-libp2p-netutil into this repo ([libp2p/go-libp2p-testing#45](https://github.com/libp2p/go-libp2p-testing/pull/45))
  - reduce the number of connections in the stream muxer stress test (#44) ([libp2p/go-libp2p-testing#44](https://github.com/libp2p/go-libp2p-testing/pull/44))
  - release v0.6.0 (#42) ([libp2p/go-libp2p-testing#42](https://github.com/libp2p/go-libp2p-testing/pull/42))
  - expose a map, not a slice, of muxer tests (#41) ([libp2p/go-libp2p-testing#41](https://github.com/libp2p/go-libp2p-testing/pull/41))
  - sync: update CI config files (#40) ([libp2p/go-libp2p-testing#40](https://github.com/libp2p/go-libp2p-testing/pull/40))
- github.com/libp2p/go-libp2p-tls (v0.3.1 -> v0.4.1):
  - release v0.4.1 (#112) ([libp2p/go-libp2p-tls#112](https://github.com/libp2p/go-libp2p-tls/pull/112))
  - feat: catch panics in TLS negotiation ([libp2p/go-libp2p-tls#111](https://github.com/libp2p/go-libp2p-tls/pull/111))
  - release v0.4.0 (#110) ([libp2p/go-libp2p-tls#110](https://github.com/libp2p/go-libp2p-tls/pull/110))
  - use tls.Conn.HandshakeContext instead of tls.Conn.Handshake (#106) ([libp2p/go-libp2p-tls#106](https://github.com/libp2p/go-libp2p-tls/pull/106))
  - remove paragraph about Go modules from README (#104) ([libp2p/go-libp2p-tls#104](https://github.com/libp2p/go-libp2p-tls/pull/104))
  - migrate to standard Go tests, stop using Ginkgo ([libp2p/go-libp2p-tls#105](https://github.com/libp2p/go-libp2p-tls/pull/105))
  - chore: remove Codecov config (#103) ([libp2p/go-libp2p-tls#103](https://github.com/libp2p/go-libp2p-tls/pull/103))
- github.com/libp2p/go-libp2p-transport-upgrader (v0.5.0 -> v0.7.1):
  - release v0.7.1 ([libp2p/go-libp2p-transport-upgrader#105](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/105))
  - Fix nil peer scope issues ([libp2p/go-libp2p-transport-upgrader#104](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/104))
  - release v0.7.0 (#103) ([libp2p/go-libp2p-transport-upgrader#103](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/103))
  - use the Resource Manager ([libp2p/go-libp2p-transport-upgrader#99](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/99))
  - rename the package to upgrader ([libp2p/go-libp2p-transport-upgrader#101](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/101))
  - use the new transport.Upgrader interface ([libp2p/go-libp2p-transport-upgrader#100](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/100))
  - reset the temporary error catcher delay after successful accept ([libp2p/go-libp2p-transport-upgrader#97](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/97))
  - make the accept timeout configurable, stop using transport.AcceptTimeout ([libp2p/go-libp2p-transport-upgrader#98](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/98))
  - release v0.6.0 (#95) ([libp2p/go-libp2p-transport-upgrader#95](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/95))
  - remove note about go.mod and Go 1.11 in README (#94) ([libp2p/go-libp2p-transport-upgrader#94](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/94))
  - fix flaky TestAcceptQueueBacklogged test ([libp2p/go-libp2p-transport-upgrader#96](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/96))
  - chore: remove Codecov config (#93) ([libp2p/go-libp2p-transport-upgrader#93](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/93))
  - use the new network.ConnStats ([libp2p/go-libp2p-transport-upgrader#92](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/92))
  - sync: update CI config files (#89) ([libp2p/go-libp2p-transport-upgrader#89](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/89))
  - chore: update go-log ([libp2p/go-libp2p-transport-upgrader#88](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/88))
- github.com/libp2p/go-libp2p-yamux (v0.6.0 -> v0.9.1):
  - release v0.9.1 (#55) ([libp2p/go-libp2p-yamux#55](https://github.com/libp2p/go-libp2p-yamux/pull/55))
  - release v0.9.0 (#53) ([libp2p/go-libp2p-yamux#53](https://github.com/libp2p/go-libp2p-yamux/pull/53))
  - release v0.8.2 (#50) ([libp2p/go-libp2p-yamux#50](https://github.com/libp2p/go-libp2p-yamux/pull/50))
  - disable the incoming streams limit (#49) ([libp2p/go-libp2p-yamux#49](https://github.com/libp2p/go-libp2p-yamux/pull/49))
  - Release v0.8.1 ([libp2p/go-libp2p-yamux#48](https://github.com/libp2p/go-libp2p-yamux/pull/48))
  - release v0.8.0 (#47) ([libp2p/go-libp2p-yamux#47](https://github.com/libp2p/go-libp2p-yamux/pull/47))
  - pass the PeerScope to yamux ( satisfying its MemoryManger interface) (#46) ([libp2p/go-libp2p-yamux#46](https://github.com/libp2p/go-libp2p-yamux/pull/46))
  - release v0.7.0 (#43) ([libp2p/go-libp2p-yamux#43](https://github.com/libp2p/go-libp2p-yamux/pull/43))
  - sync: update CI config files (#42) ([libp2p/go-libp2p-yamux#42](https://github.com/libp2p/go-libp2p-yamux/pull/42))
  - reduce the number of max incoming stream to 256 ([libp2p/go-libp2p-yamux#41](https://github.com/libp2p/go-libp2p-yamux/pull/41))
- github.com/libp2p/go-mplex (v0.3.0 -> v0.7.0):
  - release v0.7.0 (#112) ([libp2p/go-mplex#112](https://github.com/libp2p/go-mplex/pull/112))
  - catch panics in handleIncoming and handleOutgoing (#109) ([libp2p/go-mplex#109](https://github.com/libp2p/go-mplex/pull/109))
  - remove benchmark tests (#111) ([libp2p/go-mplex#111](https://github.com/libp2p/go-mplex/pull/111))
  - release v0.6.0 (#105) ([libp2p/go-mplex#105](https://github.com/libp2p/go-mplex/pull/105))
  - fix incorrect reset of timer fired variable (#104) ([libp2p/go-mplex#104](https://github.com/libp2p/go-mplex/pull/104))
  - Mplex salvage operations, part II (#102) ([libp2p/go-mplex#102](https://github.com/libp2p/go-mplex/pull/102))
  - release v0.5.0 (#100) ([libp2p/go-mplex#100](https://github.com/libp2p/go-mplex/pull/100))
  - Salvage mplex in the age of resource management (#99) ([libp2p/go-mplex#99](https://github.com/libp2p/go-mplex/pull/99))
  - release v0.4.0 (#97) ([libp2p/go-mplex#97](https://github.com/libp2p/go-mplex/pull/97))
  - add a MemoryManager interface to control memory allocations ([libp2p/go-mplex#96](https://github.com/libp2p/go-mplex/pull/96))
  - sync: update CI config files (#93) ([libp2p/go-mplex#93](https://github.com/libp2p/go-mplex/pull/93))
  - chore: update go-log to v2 ([libp2p/go-mplex#92](https://github.com/libp2p/go-mplex/pull/92))
  - chore: remove Codecov config ([libp2p/go-mplex#91](https://github.com/libp2p/go-mplex/pull/91))
  - sync: update CI config files (#90) ([libp2p/go-mplex#90](https://github.com/libp2p/go-mplex/pull/90))
  - multiplex: add (*Multiplex).CloseChan ([libp2p/go-mplex#89](https://github.com/libp2p/go-mplex/pull/89))
  - add a Go Reference badge to the README ([libp2p/go-mplex#88](https://github.com/libp2p/go-mplex/pull/88))
  - sync: update CI config files ([libp2p/go-mplex#85](https://github.com/libp2p/go-mplex/pull/85))
  - Fix up tests & vet ([libp2p/go-mplex#84](https://github.com/libp2p/go-mplex/pull/84))
  - Bump lodash from 4.17.19 to 4.17.21 in /interop/js ([libp2p/go-mplex#83](https://github.com/libp2p/go-mplex/pull/83))
- github.com/libp2p/go-msgio (v0.1.0 -> v0.2.0):
  - release v0.2.0 (#34) ([libp2p/go-msgio#34](https://github.com/libp2p/go-msgio/pull/34))
  - print recovered panics to stderr (#33) ([libp2p/go-msgio#33](https://github.com/libp2p/go-msgio/pull/33))
  - catch panics when reading / writing protobuf messages (#31) ([libp2p/go-msgio#31](https://github.com/libp2p/go-msgio/pull/31))
  - remove outdated section about channels from README (#32) ([libp2p/go-msgio#32](https://github.com/libp2p/go-msgio/pull/32))
  - sync: update CI config files (#28) ([libp2p/go-msgio#28](https://github.com/libp2p/go-msgio/pull/28))
- github.com/libp2p/go-netroute (v0.1.6 -> v0.2.0):
  - release v0.2.0 (#21) ([libp2p/go-netroute#21](https://github.com/libp2p/go-netroute/pull/21))
  - move some functions from go-sockaddr here, remove go-sockaddr dependency ([libp2p/go-netroute#22](https://github.com/libp2p/go-netroute/pull/22))
  - ignore the error on the RouteMessage on Darwin ([libp2p/go-netroute#20](https://github.com/libp2p/go-netroute/pull/20))
  - sync: update CI config files (#19) ([libp2p/go-netroute#19](https://github.com/libp2p/go-netroute/pull/19))
  - sync: update CI config files (#18) ([libp2p/go-netroute#18](https://github.com/libp2p/go-netroute/pull/18))
  - skip loopback addr as indication of v6 routes ([libp2p/go-netroute#17](https://github.com/libp2p/go-netroute/pull/17))
  - fix staticcheck lint issues ([libp2p/go-netroute#15](https://github.com/libp2p/go-netroute/pull/15))
- github.com/libp2p/go-stream-muxer-multistream (v0.3.0 -> v0.4.0):
  - release v0.4.0 (#23) ([libp2p/go-stream-muxer-multistream#23](https://github.com/libp2p/go-stream-muxer-multistream/pull/23))
  - implement the new Multiplexer.NewConn interface ([libp2p/go-stream-muxer-multistream#22](https://github.com/libp2p/go-stream-muxer-multistream/pull/22))
  - sync: update CI config files (#19) ([libp2p/go-stream-muxer-multistream#19](https://github.com/libp2p/go-stream-muxer-multistream/pull/19))
- github.com/libp2p/go-tcp-transport (v0.4.0 -> v0.5.1):
  - release v0.5.1 (#116) ([libp2p/go-tcp-transport#116](https://github.com/libp2p/go-tcp-transport/pull/116))
  - fix: drop raw EINVAL (from keepalives) errors as well (#115) ([libp2p/go-tcp-transport#115](https://github.com/libp2p/go-tcp-transport/pull/115))
  - release v0.5.0 (#114) ([libp2p/go-tcp-transport#114](https://github.com/libp2p/go-tcp-transport/pull/114))
  - use the ResourceManager ([libp2p/go-tcp-transport#110](https://github.com/libp2p/go-tcp-transport/pull/110))
  - use the transport.Upgrader interface ([libp2p/go-tcp-transport#111](https://github.com/libp2p/go-tcp-transport/pull/111))
  - describe how to use options in README ([libp2p/go-tcp-transport#105](https://github.com/libp2p/go-tcp-transport/pull/105))
- github.com/libp2p/go-ws-transport (v0.5.0 -> v0.6.0):
  - release v0.6.0 (#113) ([libp2p/go-ws-transport#113](https://github.com/libp2p/go-ws-transport/pull/113))
  - use the resource manager ([libp2p/go-ws-transport#109](https://github.com/libp2p/go-ws-transport/pull/109))
  - chore: remove Codecov config (#112) ([libp2p/go-ws-transport#112](https://github.com/libp2p/go-ws-transport/pull/112))
  - remove contexts from libp2p constructors in README (#111) ([libp2p/go-ws-transport#111](https://github.com/libp2p/go-ws-transport/pull/111))
  - use the transport.Upgrader interface ([libp2p/go-ws-transport#110](https://github.com/libp2p/go-ws-transport/pull/110))
  - sync: update CI config files (#108) ([libp2p/go-ws-transport#108](https://github.com/libp2p/go-ws-transport/pull/108))
  - sync: update CI config files (#106) ([libp2p/go-ws-transport#106](https://github.com/libp2p/go-ws-transport/pull/106))
- github.com/lucas-clemente/quic-go (v0.24.0 -> v0.27.1):
  - don't send path MTU probe packets on a timer
  - stop using the deprecated net.Error.Temporary, update golangci-lint to v1.45.2 ([lucas-clemente/quic-go#3367](https://github.com/lucas-clemente/quic-go/pull/3367))
  - add support for serializing Extended CONNECT requests (#3360) ([lucas-clemente/quic-go#3360](https://github.com/lucas-clemente/quic-go/pull/3360))
  - improve the error thrown when building with an unsupported Go version ([lucas-clemente/quic-go#3364](https://github.com/lucas-clemente/quic-go/pull/3364))
  - remove nextdns from list of projects using quic-go (#3363) ([lucas-clemente/quic-go#3363](https://github.com/lucas-clemente/quic-go/pull/3363))
  - rename the Session to Connection ([lucas-clemente/quic-go#3361](https://github.com/lucas-clemente/quic-go/pull/3361))
  - respect the request context when dialing ([lucas-clemente/quic-go#3359](https://github.com/lucas-clemente/quic-go/pull/3359))
  - update HTTP/3 Datagram to draft-ietf-masque-h3-datagram-07 (#3355) ([lucas-clemente/quic-go#3355](https://github.com/lucas-clemente/quic-go/pull/3355))
  - add support for the Extended CONNECT method (#3357) ([lucas-clemente/quic-go#3357](https://github.com/lucas-clemente/quic-go/pull/3357))
  - remove the SkipSchemeCheck RoundTripOpt (#3353) ([lucas-clemente/quic-go#3353](https://github.com/lucas-clemente/quic-go/pull/3353))
  - remove parser logic for HTTP/3 DUPLICATE_PUSH frame (#3356) ([lucas-clemente/quic-go#3356](https://github.com/lucas-clemente/quic-go/pull/3356))
  - improve code coverage of random number generator test (#3358) ([lucas-clemente/quic-go#3358](https://github.com/lucas-clemente/quic-go/pull/3358))
  - advertise multiple listeners via Alt-Svc and improve perf of SetQuicHeaders (#3352) ([lucas-clemente/quic-go#3352](https://github.com/lucas-clemente/quic-go/pull/3352))
  - avoid recursion when skipping unknown HTTP/3 frames (#3354) ([lucas-clemente/quic-go#3354](https://github.com/lucas-clemente/quic-go/pull/3354))
  - Implement http3.Server.ServeListener (#3349) ([lucas-clemente/quic-go#3349](https://github.com/lucas-clemente/quic-go/pull/3349))
  - update for Go 1.18 ([lucas-clemente/quic-go#3345](https://github.com/lucas-clemente/quic-go/pull/3345))
  - don't print a receive buffer warning for closed connections (#3346) ([lucas-clemente/quic-go#3346](https://github.com/lucas-clemente/quic-go/pull/3346))
  - move set DF implementation to separate files & avoid the need for OOBCapablePacketConn (#3334) ([lucas-clemente/quic-go#3334](https://github.com/lucas-clemente/quic-go/pull/3334))
  - add env to disable the receive buffer warning (#3339) ([lucas-clemente/quic-go#3339](https://github.com/lucas-clemente/quic-go/pull/3339))
  - fix typo (#3333) ([lucas-clemente/quic-go#3333](https://github.com/lucas-clemente/quic-go/pull/3333))
  - sendQueue: ignore "datagram too large" error (#3328) ([lucas-clemente/quic-go#3328](https://github.com/lucas-clemente/quic-go/pull/3328))
  - add OONI Probe to list of projects in README (#3324) ([lucas-clemente/quic-go#3324](https://github.com/lucas-clemente/quic-go/pull/3324))
  - remove build status badges from README (#3325) ([lucas-clemente/quic-go#3325](https://github.com/lucas-clemente/quic-go/pull/3325))
  - add a AllowConnectionWindowIncrease config option ([lucas-clemente/quic-go#3317](https://github.com/lucas-clemente/quic-go/pull/3317))
  - Update README.md (#3315) ([lucas-clemente/quic-go#3315](https://github.com/lucas-clemente/quic-go/pull/3315))
  - fix some typos in documentation and tests
  - remove unneeded calls to goimports when generating mocks ([lucas-clemente/quic-go#3313](https://github.com/lucas-clemente/quic-go/pull/3313))
  - fix comment about congestionWindow value (#3310) ([lucas-clemente/quic-go#3310](https://github.com/lucas-clemente/quic-go/pull/3310))
  - fix typo *connections (#3309) ([lucas-clemente/quic-go#3309](https://github.com/lucas-clemente/quic-go/pull/3309))
  - add support for Go 1.18 ([lucas-clemente/quic-go#3298](https://github.com/lucas-clemente/quic-go/pull/3298))
- github.com/multiformats/go-base32 (v0.0.3 -> v0.0.4):
  - optimize encode ([multiformats/go-base32#1](https://github.com/multiformats/go-base32/pull/1))
  - Fix `staticcheck` issue
- github.com/multiformats/go-multiaddr (v0.4.1 -> v0.5.0):
  - release v0.5.0 (#171) ([multiformats/go-multiaddr#171](https://github.com/multiformats/go-multiaddr/pull/171))
  - remove wrong (and redundant) IsIpv6LinkLocal ([multiformats/go-multiaddr#170](https://github.com/multiformats/go-multiaddr/pull/170))
  - move ResolveUnspecifiedAddress(es) and FilterAddrs here from libp2p/go-addr-util ([multiformats/go-multiaddr#168](https://github.com/multiformats/go-multiaddr/pull/168))
  - sync: update CI config files (#167) ([multiformats/go-multiaddr#167](https://github.com/multiformats/go-multiaddr/pull/167))
- github.com/multiformats/go-multicodec (v0.3.0 -> v0.4.1):
  - Version v0.4.1 ([multiformats/go-multicodec#64](https://github.com/multiformats/go-multicodec/pull/64))
  - update table with new codecs ([multiformats/go-multicodec#63](https://github.com/multiformats/go-multicodec/pull/63))
  - bump version to v0.4.0
  - sync: update CI config files (#60) ([multiformats/go-multicodec#60](https://github.com/multiformats/go-multicodec/pull/60))
  - add Code.Tag method
  - add the KnownCodes API
  - use "go run pkg@version" assuming Go 1.17 or later
  - update submodule and re-generate
  - update to newer multicodec table ([multiformats/go-multicodec#57](https://github.com/multiformats/go-multicodec/pull/57))
  - Update `multicodec` submodule to `1bcdc08` for CARv2 index codec
- github.com/multiformats/go-multistream (v0.2.2 -> v0.3.0):
  - release v0.3.0 (#82) ([multiformats/go-multistream#82](https://github.com/multiformats/go-multistream/pull/82))
  - catch panics (#81) ([multiformats/go-multistream#81](https://github.com/multiformats/go-multistream/pull/81))
  - sync: update CI config files (#78) ([multiformats/go-multistream#78](https://github.com/multiformats/go-multistream/pull/78))
  - reduce the maximum read buffer size from 64 to 1 kB ([multiformats/go-multistream#77](https://github.com/multiformats/go-multistream/pull/77))
  - remove unused ls command ([multiformats/go-multistream#76](https://github.com/multiformats/go-multistream/pull/76))
  - chore: remove empty file cases.md ([multiformats/go-multistream#75](https://github.com/multiformats/go-multistream/pull/75))
  - chore: remove .gx ([multiformats/go-multistream#72](https://github.com/multiformats/go-multistream/pull/72))
  - don't commit the fuzzing binary ([multiformats/go-multistream#74](https://github.com/multiformats/go-multistream/pull/74))
  - sync: update CI config files (#71) ([multiformats/go-multistream#71](https://github.com/multiformats/go-multistream/pull/71))
  - remove Makefile ([multiformats/go-multistream#67](https://github.com/multiformats/go-multistream/pull/67))

</details>

### ❤ Contributors
| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 347 | +14453/-12552 | 842 |
| Rod Vagg | 28 | +8848/-4033 | 214 |
| vyzo | 133 | +7959/-1783 | 231 |
| hannahhoward | 40 | +3761/-1652 | 175 |
| Will Scott | 39 | +2885/-1784 | 93 |
| Daniel Martí | 32 | +3043/-969 | 103 |
| Adin Schmahmann | 48 | +3439/-536 | 121 |
| Gus Eggert | 29 | +2644/-788 | 123 |
| Steven Allen | 87 | +2417/-840 | 135 |
| Marcin Rataj | 29 | +2312/-942 | 75 |
| Will | 11 | +2520/-62 | 56 |
| Lucas Molas | 28 | +1602/-578 | 90 |
| Raúl Kripalani | 18 | +1519/-271 | 38 |
| Brian Tiger Chow | 20 | +833/-379 | 40 |
| Masih H. Derkani | 5 | +514/-460 | 8 |
| Jeromy Johnson | 53 | +646/-302 | 83 |
| Łukasz Magiera | 26 | +592/-245 | 43 |
| Artem Mikheev | 2 | +616/-120 | 5 |
| Franky W | 2 | +49/-525 | 9 |
| Laurent Senta | 3 | +468/-82 | 52 |
| Hector Sanjuan | 32 | +253/-187 | 62 |
| Juan Batiz-Benet | 8 | +285/-80 | 18 |
| Justin Johnson | 2 | +181/-88 | 2 |
| Thibault Meunier | 5 | +216/-28 | 8 |
| James Wetter | 2 | +234/-1 | 2 |
| web3-bot | 36 | +158/-66 | 62 |
| gammazero | 7 | +140/-84 | 12 |
| Rachel Chen | 2 | +165/-57 | 17 |
| Jorropo | 18 | +108/-99 | 26 |
| Toby | 2 | +107/-86 | 11 |
| Antonio Navarro Perez | 4 | +82/-103 | 9 |
| Dominic Della Valle | 4 | +148/-33 | 6 |
| Ian Davis | 2 | +152/-28 | 6 |
| Kyle Huntsman | 2 | +172/-6 | 5 |
| huoju | 4 | +127/-41 | 6 |
| Jeromy | 19 | +71/-58 | 31 |
| Lars Gierth | 12 | +63/-54 | 20 |
| Eric Myhre | 3 | +95/-15 | 8 |
| Caian Benedicto | 1 | +69/-12 | 6 |
| Raúl Kripalani | 2 | +63/-13 | 2 |
| Anton Petrov | 1 | +73/-0 | 1 |
| hunjixin | 2 | +67/-2 | 5 |
| odanado | 1 | +61/-0 | 1 |
| Andrew Gillis | 2 | +61/-0 | 3 |
| Kevin Atkinson | 6 | +21/-34 | 7 |
| Richard Ramos | 1 | +51/-0 | 2 |
| Manuel Alonso | 1 | +42/-9 | 2 |
| Jakub Sztandera | 10 | +37/-13 | 13 |
| Aarsh Shah | 1 | +39/-5 | 2 |
| Dave Justice | 1 | +32/-4 | 2 |
| Tommi Virtanen | 3 | +23/-9 | 4 |
| tarekbadr | 1 | +30/-1 | 1 |
| whyrusleeping | 1 | +26/-4 | 3 |
| Petar Maymounkov | 2 | +30/-0 | 4 |
| rht | 3 | +17/-10 | 4 |
| Miguel Mota | 1 | +23/-0 | 1 |
| Manfred Touron | 1 | +21/-2 | 2 |
| watjurk | 1 | +17/-5 | 1 |
| SukkaW | 1 | +11/-11 | 5 |
| Nicholas Bollweg | 1 | +21/-0 | 1 |
| Ho-Sheng Hsiao | 2 | +11/-10 | 6 |
| chblodg | 1 | +18/-2 | 1 |
| Friedel Ziegelmayer | 2 | +18/-0 | 2 |
| Shu Shen | 2 | +15/-2 | 3 |
| Peter Rabbitson | 1 | +15/-2 | 1 |
| galargh | 2 | +15/-0 | 2 |
| ᴍᴀᴛᴛ ʙᴇʟʟ | 3 | +13/-1 | 4 |
| aarshkshah1992 | 3 | +12/-2 | 3 |
| RubenKelevra | 4 | +5/-8 | 5 |
| Feiran Yang | 1 | +11/-0 | 2 |
| zramsay | 2 | +0/-10 | 2 |
| Teran McKinney | 1 | +8/-2 | 1 |
| Richard Littauer | 2 | +5/-5 | 5 |
| Elijah | 1 | +10/-0 | 1 |
| Dimitris Apostolou | 2 | +5/-5 | 5 |
| Michael Avila | 3 | +8/-1 | 4 |
| siiky | 3 | +4/-4 | 3 |
| Somajit | 1 | +4/-4 | 1 |
| Sherod Taylor | 1 | +0/-8 | 2 |
| Eclésio Junior | 1 | +8/-0 | 1 |
| godcong | 3 | +4/-3 | 3 |
| Piotr Galar | 3 | +3/-4 | 3 |
| jwh | 1 | +6/-0 | 2 |
| dependabot[bot] | 1 | +3/-3 | 1 |
| Volker Mische | 1 | +4/-2 | 1 |
| Aayush Rajasekaran | 1 | +3/-3 | 1 |
| rene | 2 | +3/-2 | 2 |
| keks | 1 | +5/-0 | 1 |
| Hlib | 1 | +4/-1 | 2 |
| Arash Payan | 1 | +5/-0 | 1 |
| Wayback Archiver | 1 | +2/-2 | 1 |
| T Mo | 1 | +2/-2 | 1 |
| Ju Huo | 1 | +2/-2 | 1 |
| Ivan | 2 | +2/-2 | 2 |
| Ettore Di Giacinto | 2 | +3/-1 | 2 |
| Christian Couder | 1 | +3/-1 | 1 |
| ningmingxiao | 1 | +0/-3 | 1 |
| 市川恭佑 (ebi) | 1 | +1/-1 | 1 |
| star | 1 | +0/-2 | 1 |
| alliswell | 1 | +0/-2 | 1 |
| Preston Van Loon | 1 | +2/-0 | 1 |
| Nguyễn Gia Phong | 1 | +1/-1 | 1 |
| Nato Boram | 1 | +1/-1 | 1 |
| Mildred Ki'Lya | 1 | +2/-0 | 2 |
| Michael Burns | 1 | +1/-1 | 1 |
| Glenn | 1 | +1/-1 | 1 |
| George Antoniadis | 1 | +1/-1 | 1 |
| David Florness | 1 | +1/-1 | 1 |
| Daniel Norman | 1 | +1/-1 | 1 |
| Coenie Beyers | 1 | +1/-1 | 1 |
| Benedikt Spies | 1 | +1/-1 | 1 |
| Abdul Rauf | 1 | +1/-1 | 1 |
| makeworld | 1 | +1/-0 | 1 |
| ignoramous | 1 | +0/-1 | 1 |
| Omicron166 | 1 | +0/-1 | 1 |
| Jan Winkelmann | 1 | +1/-0 | 1 |
| Dr Ian Preston | 1 | +1/-0 | 1 |
| Baptiste Jonglez | 1 | +1/-0 | 1 |


# Kubo changelog v0.14

## v0.14.0

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

- [🛠 BREAKING CHANGES](#-breaking-changes)
  - [Removed `mdns_legacy` implementation](#removed-mdns_legacy-implementation)
- [🔦 Highlights](#-highlights)
  - [🛣️ Delegated Routing](#-delegated-routing)
  - [👥 Rename to Kubo](#-rename-to-kubo)
  - [🎒 `ipfs repo migrate`](#-ipfs-repo-migrate)
  - [🚀 Emoji support in Multibase](#-emoji-support-in-multibase)

### 🛠 BREAKING CHANGES

#### Removed `mdns_legacy` implementation

The modern DNS-SD compatible [zeroconf implementation](https://github.com/libp2p/zeroconf#readme)
(based on [this specification](https://github.com/libp2p/specs/blob/master/discovery/mdns.md))
has been running next to the `mdns_legacy` for a while (since v0.11). During
this transitional period Kubo nodes were sending twice as many LAN packets,
which ends with this release: we've [removed](https://github.com/ipfs/kubo/pull/9048) the legacy implementation.

### 🔦 Highlights

#### 🛣️ Delegated Routing

Content routing is the a term used to describe the problem of finding providers for a given piece of content.
If you have a hash, or CID of some data, how do you find who has it?
In IPFS, until now, only a DHT was used as a decentralized answer to content routing.
Now, content routing can be handled by clients implementing the [Reframe protocol](https://github.com/ipfs/specs/tree/main/reframe#readme).

Example configuration usage using the [Filecoin Network Indexer](https://docs.cid.contact/filecoin-network-indexer/overview):

```go
ipfs config Routing.Routers.CidContact --json '{
  "Type": "reframe",
  "Parameters": {
    "Endpoint": "https://cid.contact/reframe"
  }
}'

```

#### 👥 Rename to Kubo

We've renamed Go-IPFS to Kubo ([details](https://github.com/ipfs/go-ipfs/issues/8959)).

Published artifacts use `kubo` now, and are available at:

- https://dist.ipfs.tech/kubo/
- https://hub.docker.com/r/ipfs/kubo/

To minimize the impact on infrastructure that autoupdates on a new release,
the same binaries are still published under the old name at:

- https://dist.ipfs.tech/go-ipfs/
- https://hub.docker.com/r/ipfs/go-ipfs/

The libp2p identify useragent of Kubo has also been changed from `go-ipfs` to `kubo`.

#### 🎒 `ipfs repo migrate`

This new command allows the you to run the repo migration without starting the daemon.

See `ipfs repo migrate --help` for more info.

#### 🚀 Emoji support in Multibase

Kubo now supports [`base256emoji`](https://github.com/multiformats/multibase/blob/master/rfcs/Base256Emoji.md) encoding in all [Multibase](https://docs.ipfs.tech/concepts/glossary/#multibase) contexts. Use it for testing Unicode support, as visual aid while explaining Multiformats, or just for fun:

```console
$ echo -n "test" | ipfs multibase encode -b base256emoji -
🚀😈✋🌈😈

$ echo -n "🚀😈✋🌈😈" | ipfs multibase decode -
test

$ ipfs cid format -v 1 -b base256emoji bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi
🚀🪐⭐💻😅❓💎🌈🌸🌚💰💍🌒😵🐶💁🤐🌎👼🙃🙅☺🌚😞🤤⭐🚀😃✈🌕😚🍻💜🐷⚽✌😊
```

[`/ipfs/🚀🪐⭐💻😅❓💎🌈🌸🌚💰💍🌒😵🐶💁🤐🌎👼🙃🙅☺🌚😞🤤⭐🚀😃✈🌕😚🍻💜🐷⚽✌😊`](https://ipfs.io/ipfs/🚀🪐⭐💻😅❓💎🌈🌸🌚💰💍🌒😵🐶💁🤐🌎👼🙃🙅☺🌚😞🤤⭐🚀😃✈🌕😚🍻💜🐷⚽✌😊)

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - chore: bump to v0.14.0
  - docs(changelog): finish v0.14.0 changelog
  - fix(gw): cache-control of index.html websites
  - chore(license): fix broken link to apache-2.0
  - fix: kubo in daemon and cli stdout
  - backport: merge commit '839b0848a' into release-v0.14.0
  - chore: Release v0.14-rc1
  - docs: fix v0.14's changelog format
  - chore: update go-multibase 🚀
  - feat(routing): Delegated Routing (#8997) ([ipfs/kubo#8997](https://github.com/ipfs/kubo/pull/8997))
  - chore: changelogs split
  - feat(gw): Cache-Control: only-if-cached
  - chore(deps): webui v2.15.1
  - Follow-ups after repository rename
 ([ipfs/kubo#9098](https://github.com/ipfs/kubo/pull/9098))
  - docs: refine wording
  - docs: refine the wording of provider strategies
  - refactor: rename to kubo
 ([ipfs/kubo#8958](https://github.com/ipfs/kubo/pull/8958))
  - fix: correct cache-control in car responses
  - docs: v0.13.1 (#9093) ([ipfs/kubo#9093](https://github.com/ipfs/kubo/pull/9093))
  - chore: update go-car ([ipfs/kubo#9089](https://github.com/ipfs/kubo/pull/9089))
  - update go-libp2p to v0.20.3 ([ipfs/kubo#9038](https://github.com/ipfs/kubo/pull/9038))
  - docs: add SECURITY.md (#9062) ([ipfs/kubo#9062](https://github.com/ipfs/kubo/pull/9062))
  - fix: remove mdns_legacy & Discovery.MDNS.Interval
  - refactor: prealloc slices with known sizes (#8892) ([ipfs/kubo#8892](https://github.com/ipfs/kubo/pull/8892))
  - docs: fix typo in `cid/base32`
  - docs: mark Swarm.ResourceMgr as experimental
  - chore: replace ioutil with io and os (#8969) ([ipfs/kubo#8969](https://github.com/ipfs/kubo/pull/8969))
  - feat: add a public function on peering to get the state
  - fix: honor url filename when downloading as CAR/BLOCK
  - Merge branch 'release'
  - chore: GitHub format
  - fix(cmd/config): make config edit subcommand work on windows
  - chore: bump Go to 1.18.3 (#9021) ([ipfs/kubo#9021](https://github.com/ipfs/kubo/pull/9021))
  - feat: upgrade to go-libp2p-kad-dht@v0.16.0 (#9005) ([ipfs/kubo#9005](https://github.com/ipfs/kubo/pull/9005))
  - docs: fix typo in the `swarm/peering` help text
  - feat: disable resource manager by default (#9003) ([ipfs/kubo#9003](https://github.com/ipfs/kubo/pull/9003))
  - fix: adjust rcmgr limits for accelerated DHT client rt refresh (#8982) ([ipfs/kubo#8982](https://github.com/ipfs/kubo/pull/8982))
  - fix(ci): make go-ipfs-as-a-library work without external peers (#8978) ([ipfs/kubo#8978](https://github.com/ipfs/kubo/pull/8978))
  - feat: log when resource manager limits are exceeded (#8980) ([ipfs/kubo#8980](https://github.com/ipfs/kubo/pull/8980))
  - fix: JS caching via Access-Control-Expose-Headers (#8984) ([ipfs/kubo#8984](https://github.com/ipfs/kubo/pull/8984))
  - docs: fix abstractions typo
  - fix: hanging goroutine in get fileArchive handler
  - chore: mark fuse experimental (#8962) ([ipfs/kubo#8962](https://github.com/ipfs/kubo/pull/8962))
  - fix(node/libp2p): disable rcmgr checkImplicitDefaults ([ipfs/kubo#8965](https://github.com/ipfs/kubo/pull/8965))
  - Add 'ipfs repo migrate' command (#8428) ([ipfs/kubo#8428](https://github.com/ipfs/kubo/pull/8428))
  - pubsub multibase encoding (#8933) ([ipfs/kubo#8933](https://github.com/ipfs/kubo/pull/8933))
  - 'pin rm' helptext: rewrite description as object is not removed from local storage (immediately) ([ipfs/kubo#8947](https://github.com/ipfs/kubo/pull/8947))
  -  ([ipfs/kubo#8934](https://github.com/ipfs/kubo/pull/8934))
  - Add instructions to resolve repo migration error (#8946) ([ipfs/kubo#8946](https://github.com/ipfs/kubo/pull/8946))
  - fix: use path instead of filepath for asset embeds to support Windows
  - chore: update version to v0.14.0-dev
- github.com/ipfs/go-bitswap (v0.6.0 -> v0.7.0):
  - chore: release v0.7.0 (#566) ([ipfs/go-bitswap#566](https://github.com/ipfs/go-bitswap/pull/566))
  - feat: coalesce and queue connection event handling (#565) ([ipfs/go-bitswap#565](https://github.com/ipfs/go-bitswap/pull/565))
  - fix initialisation example in README (#552) ([ipfs/go-bitswap#552](https://github.com/ipfs/go-bitswap/pull/552))
- github.com/ipfs/go-unixfs (v0.3.1 -> v0.4.0):
  - Set version to v0.3.2 ([ipfs/go-unixfs#122](https://github.com/ipfs/go-unixfs/pull/122))
  - Make switchToSharding more efficient
- github.com/ipld/go-ipld-prime (v0.16.0 -> v0.17.0):
  failed to fetch repo
- github.com/libp2p/go-libp2p (v0.19.4 -> v0.20.3):
  - Release 0.20.3 (#1623) ([libp2p/go-libp2p#1623](https://github.com/libp2p/go-libp2p/pull/1623))
  - release v0.20.2
  - feat: allow dialing wss peers using DNS multiaddrs
  - update go-yamux to v3.1.2, release v0.20.1 (#1591) ([libp2p/go-libp2p#1591](https://github.com/libp2p/go-libp2p/pull/1591))
  - release v0.20.0 (#1530) ([libp2p/go-libp2p#1530](https://github.com/libp2p/go-libp2p/pull/1530))
  - update go-libp2p-core, remove stream methods from network.Notifiee (#1521) ([libp2p/go-libp2p#1521](https://github.com/libp2p/go-libp2p/pull/1521))
  - autonat: return E_DIAL_REFUSED when skipping dial (#1527) ([libp2p/go-libp2p#1527](https://github.com/libp2p/go-libp2p/pull/1527))
  - move go-stream-muxer-multistream here ([libp2p/go-libp2p#1511](https://github.com/libp2p/go-libp2p/pull/1511))
  - remove dependency on go-libp2p-testing/suites/sec (#1510) ([libp2p/go-libp2p#1510](https://github.com/libp2p/go-libp2p/pull/1510))
  - backoff: fix flaky tests in backoff cache (#1516) ([libp2p/go-libp2p#1516](https://github.com/libp2p/go-libp2p/pull/1516))
  - identify: fix flaky tests (#1515) ([libp2p/go-libp2p#1515](https://github.com/libp2p/go-libp2p/pull/1515))
  - quic: increase timeout in hole punching test (#1495) ([libp2p/go-libp2p#1495](https://github.com/libp2p/go-libp2p/pull/1495))
  - Fix badge image in README (#1517) ([libp2p/go-libp2p#1517](https://github.com/libp2p/go-libp2p/pull/1517))
  - move go-libp2p-nat here ([libp2p/go-libp2p#1513](https://github.com/libp2p/go-libp2p/pull/1513))
  - move go-reuseport-transport here ([libp2p/go-libp2p#1459](https://github.com/libp2p/go-libp2p/pull/1459))
  - holepunch: fix flaky TestEndToEndSimConnect test (#1508) ([libp2p/go-libp2p#1508](https://github.com/libp2p/go-libp2p/pull/1508))
  - swarm: fix flaky TestDialExistingConnection test (#1509) ([libp2p/go-libp2p#1509](https://github.com/libp2p/go-libp2p/pull/1509))
  - tcp: limit the number of connections in tcp suite test on non-linux hosts (#1507) ([libp2p/go-libp2p#1507](https://github.com/libp2p/go-libp2p/pull/1507))
  - increase overly short require.Eventually intervals (#1501) ([libp2p/go-libp2p#1501](https://github.com/libp2p/go-libp2p/pull/1501))
  - tls: fix flaky handshake cancelation test (#1503) ([libp2p/go-libp2p#1503](https://github.com/libp2p/go-libp2p/pull/1503))
  - merge the transport test suite from go-libp2p-testing here ([libp2p/go-libp2p#1496](https://github.com/libp2p/go-libp2p/pull/1496))
  - fix racy connection comparison in TestDialWorkerLoopBasic (#1499) ([libp2p/go-libp2p#1499](https://github.com/libp2p/go-libp2p/pull/1499))
  - swarm: fix race condition in TestFailFirst (#1490) ([libp2p/go-libp2p#1490](https://github.com/libp2p/go-libp2p/pull/1490))
  - basichost: fix flaky TestSignedPeerRecordWithNoListenAddrs (#1488) ([libp2p/go-libp2p#1488](https://github.com/libp2p/go-libp2p/pull/1488))
  - swarm: fix flaky and racy TestDialExistingConnection (#1491) ([libp2p/go-libp2p#1491](https://github.com/libp2p/go-libp2p/pull/1491))
  - quic: adjust timeout for reuse garbage collector detection in tests (#1487) ([libp2p/go-libp2p#1487](https://github.com/libp2p/go-libp2p/pull/1487))
  - quic: fix flaky TestResourceManagerAcceptDenied (#1485) ([libp2p/go-libp2p#1485](https://github.com/libp2p/go-libp2p/pull/1485))
  - quic: deflake the holepunching test (#1484) ([libp2p/go-libp2p#1484](https://github.com/libp2p/go-libp2p/pull/1484))
  - holepunch: fix incorrect message type for the SYNC message (#1478) ([libp2p/go-libp2p#1478](https://github.com/libp2p/go-libp2p/pull/1478))
  - use real keys in tests instead of go-libp2p-testing/netutil fake keys (#1475) ([libp2p/go-libp2p#1475](https://github.com/libp2p/go-libp2p/pull/1475))
  - quic: fix flaky TestResourceManagerAcceptDenied ([libp2p/go-libp2p#1461](https://github.com/libp2p/go-libp2p/pull/1461))
  - move go-libp2p-pnet here ([libp2p/go-libp2p#1465](https://github.com/libp2p/go-libp2p/pull/1465))
  - move go-libp2p-tls here ([libp2p/go-libp2p#1466](https://github.com/libp2p/go-libp2p/pull/1466))
  - fix race condition in relayFinder ([libp2p/go-libp2p#1469](https://github.com/libp2p/go-libp2p/pull/1469))
  - fix race condition in holepunch service (#1473) ([libp2p/go-libp2p#1473](https://github.com/libp2p/go-libp2p/pull/1473))
  - Update README to include supported Go Versions (#1470) ([libp2p/go-libp2p#1470](https://github.com/libp2p/go-libp2p/pull/1470))
  - move go-libp2p-noise here ([libp2p/go-libp2p#1462](https://github.com/libp2p/go-libp2p/pull/1462))
  - move go-libp2p-transport-upgrader here ([libp2p/go-libp2p#1463](https://github.com/libp2p/go-libp2p/pull/1463))
  - move go-conn-security-multistream here ([libp2p/go-libp2p#1460](https://github.com/libp2p/go-libp2p/pull/1460))
  - move go-libp2p-mplex here ([libp2p/go-libp2p#1450](https://github.com/libp2p/go-libp2p/pull/1450))
  - use yamux instead of mplex in tests (#1456) ([libp2p/go-libp2p#1456](https://github.com/libp2p/go-libp2p/pull/1456))
  - rename the yamux package (#1452) ([libp2p/go-libp2p#1452](https://github.com/libp2p/go-libp2p/pull/1452))
  - swarm: don't check return value of str.Close in TestResourceManager (#1453) ([libp2p/go-libp2p#1453](https://github.com/libp2p/go-libp2p/pull/1453))
  - move go-libp2p-yamux here ([libp2p/go-libp2p#1439](https://github.com/libp2p/go-libp2p/pull/1439))
  - quic: fix flaky TestConnectionGating test (#1442) ([libp2p/go-libp2p#1442](https://github.com/libp2p/go-libp2p/pull/1442))
  - quic: fix flaky TestReuseGarbageCollect test (#1446) ([libp2p/go-libp2p#1446](https://github.com/libp2p/go-libp2p/pull/1446))
  - quic: fix flaky holepunching test (#1443) ([libp2p/go-libp2p#1443](https://github.com/libp2p/go-libp2p/pull/1443))
  - move go-libp2p-quic-transport here ([libp2p/go-libp2p#1424](https://github.com/libp2p/go-libp2p/pull/1424))
  - remove flaky TestTcpSimultaneousConnect (#1425) ([libp2p/go-libp2p#1425](https://github.com/libp2p/go-libp2p/pull/1425))
  - move go-ws-transport here ([libp2p/go-libp2p#1422](https://github.com/libp2p/go-libp2p/pull/1422))
  - update go-multistream, stop using deprecated NegotiateLazy (#1417) ([libp2p/go-libp2p#1417](https://github.com/libp2p/go-libp2p/pull/1417))
  - fix flaky TestResourceManagerAcceptStream test (#1420) ([libp2p/go-libp2p#1420](https://github.com/libp2p/go-libp2p/pull/1420))
  - move go-tcp-transport here ([libp2p/go-libp2p#1418](https://github.com/libp2p/go-libp2p/pull/1418))
  - move the go-libp2p-swarm here ([libp2p/go-libp2p#1414](https://github.com/libp2p/go-libp2p/pull/1414))
  - reduce flakiness of backoff cache tests (#1415) ([libp2p/go-libp2p#1415](https://github.com/libp2p/go-libp2p/pull/1415))
  - move the go-libp2p-blankhost here ([libp2p/go-libp2p#1411](https://github.com/libp2p/go-libp2p/pull/1411))
- github.com/libp2p/go-libp2p-core (v0.15.1 -> v0.16.1):
  - release v0.16.1 (#255) ([libp2p/go-libp2p-core#255](https://github.com/libp2p/go-libp2p-core/pull/255))
  - force usage of github.com/btcsuite/btcd v0.22.1 or newer (#254) ([libp2p/go-libp2p-core#254](https://github.com/libp2p/go-libp2p-core/pull/254))
  - release v0.16.0 (#251) ([libp2p/go-libp2p-core#251](https://github.com/libp2p/go-libp2p-core/pull/251))
  - remove OpenedStream and ClosedStream from Notifiee interface (#250) ([libp2p/go-libp2p-core#250](https://github.com/libp2p/go-libp2p-core/pull/250))
  - deprecate Negotiator.NegotiateLazy (#249) ([libp2p/go-libp2p-core#249](https://github.com/libp2p/go-libp2p-core/pull/249))
  - update btcec dependency (#247) ([libp2p/go-libp2p-core#247](https://github.com/libp2p/go-libp2p-core/pull/247))
- github.com/libp2p/go-libp2p-discovery (v0.6.0 -> v0.7.0):
  - deprecate this repo (#84) ([libp2p/go-libp2p-discovery#84](https://github.com/libp2p/go-libp2p-discovery/pull/84))
  - remove dependency on the go-libp2p-peerstore/addr package (#82) ([libp2p/go-libp2p-discovery#82](https://github.com/libp2p/go-libp2p-discovery/pull/82))
  - fix flaky TestBackoffDiscoveryMultipleBackoff test on CI (#80) ([libp2p/go-libp2p-discovery#80](https://github.com/libp2p/go-libp2p-discovery/pull/80))
  - chore: update go-log to v2 ([libp2p/go-libp2p-discovery#76](https://github.com/libp2p/go-libp2p-discovery/pull/76))
  - sync: update CI config files (#74) ([libp2p/go-libp2p-discovery#74](https://github.com/libp2p/go-libp2p-discovery/pull/74))
- github.com/libp2p/go-libp2p-swarm (v0.10.2 -> v0.11.0):
  - deprecate this repo (#320) ([libp2p/go-libp2p-swarm#320](https://github.com/libp2p/go-libp2p-swarm/pull/320))
  - sync: update CI config files ([libp2p/go-libp2p-swarm#317](https://github.com/libp2p/go-libp2p-swarm/pull/317))
- github.com/libp2p/go-reuseport (v0.1.0 -> v0.2.0):
  - release v0.2.0 (#90) ([libp2p/go-reuseport#90](https://github.com/libp2p/go-reuseport/pull/90))
  - sync: update CI config files (#86) ([libp2p/go-reuseport#86](https://github.com/libp2p/go-reuseport/pull/86))
- github.com/multiformats/go-multibase (v0.0.3 -> v0.1.0):
  - chore: release v0.1.0
  - feat: add UTF-8 support and base256emoji
  - submodule: spec/
  - sync: update CI config files (#48) ([multiformats/go-multibase#48](https://github.com/multiformats/go-multibase/pull/48))
  - fix staticcheck ([multiformats/go-multibase#41](https://github.com/multiformats/go-multibase/pull/41))
  - Fix vet warnings about conversion of int to string ([multiformats/go-multibase#39](https://github.com/multiformats/go-multibase/pull/39))
- github.com/multiformats/go-multihash (v0.1.0 -> v0.2.0):
  - chore: replace blake2b implementation by golang.org/x/crypto ([multiformats/go-multihash#157](https://github.com/multiformats/go-multihash/pull/157))
  - sync: update CI config files ([multiformats/go-multihash#156](https://github.com/multiformats/go-multihash/pull/156))
- github.com/multiformats/go-multistream (v0.3.0 -> v0.3.3):
  - Release v0.3.3 ([multiformats/go-multistream#90](https://github.com/multiformats/go-multistream/pull/90))
  - Ignore error if can't write back multistream protocol id ([multiformats/go-multistream#89](https://github.com/multiformats/go-multistream/pull/89))
  - release v0.3.2 (#88) ([multiformats/go-multistream#88](https://github.com/multiformats/go-multistream/pull/88))
  - Ignore error if can't write back echoed protocol in negotiate (#87) ([multiformats/go-multistream#87](https://github.com/multiformats/go-multistream/pull/87))
  - release v0.3.1 (#86) ([multiformats/go-multistream#86](https://github.com/multiformats/go-multistream/pull/86))
  - deprecate NegotiateLazy (#85) ([multiformats/go-multistream#85](https://github.com/multiformats/go-multistream/pull/85))
  - return an ErrNotSupported when lazy negotiation fails (#84) ([multiformats/go-multistream#84](https://github.com/multiformats/go-multistream/pull/84))
- github.com/warpfork/go-testmark (v0.9.0 -> v0.10.0):
  - testexec: support a hunk named 'input' for stdin.
  - readme: link to other implementations!
  - readme: discuss autopatching and fixture regeneration
  - readme: discuss extensions, and introduce testexec as an example.

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 376 | +11584/-15055 | 894 |
| Jorropo | 18 | +11649/-11249 | 81 |
| noot | 43 | +5974/-3332 | 170 |
| Steven Allen | 173 | +5206/-3124 | 282 |
| Yusef Napora | 49 | +1911/-3606 | 124 |
| Juan Batiz-Benet | 14 | +3933/-53 | 48 |
| Jeromy | 84 | +2140/-1328 | 240 |
| vyzo | 51 | +2057/-1126 | 79 |
| Raúl Kripalani | 39 | +1993/-867 | 103 |
| Jeromy Johnson | 52 | +1700/-1081 | 233 |
| Antonio Navarro Perez | 4 | +1874/-729 | 34 |
| Aarsh Shah | 24 | +1428/-504 | 54 |
| Marcin Rataj | 19 | +1051/-855 | 251 |
| Alex Browne | 25 | +1207/-582 | 49 |
| Jakub Sztandera | 29 | +898/-335 | 63 |
| Friedel Ziegelmayer | 11 | +491/-284 | 18 |
| Will Scott | 6 | +240/-319 | 17 |
| Marco Munizaga | 11 | +377/-141 | 17 |
| Hlib | 8 | +269/-135 | 15 |
| Gus Eggert | 5 | +325/-63 | 19 |
| lnykww | 1 | +275/-50 | 4 |
| Łukasz Magiera | 3 | +196/-58 | 7 |
| Matt Joiner | 14 | +79/-55 | 17 |
| Eric Myhre | 4 | +122/-6 | 5 |
| Andrew Gillis | 1 | +111/-6 | 4 |
| Fazlul Shahriar | 2 | +84/-31 | 5 |
| tg | 1 | +70/-15 | 2 |
| Cory Schwartz | 4 | +50/-28 | 11 |
| Lars Gierth | 3 | +33/-26 | 3 |
| Cole Brown | 2 | +37/-16 | 9 |
| web3-bot | 7 | +38/-11 | 18 |
| Alvin Reyes | 1 | +34/-14 | 1 |
| Hector Sanjuan | 4 | +34/-8 | 5 |
| Guilhem Fanton | 2 | +28/-10 | 6 |
| Brian Meek | 1 | +14/-17 | 4 |
| Hlib Kanunnikov | 1 | +25/-3 | 1 |
| Adin Schmahmann | 5 | +15/-13 | 5 |
| Henrique Dias | 1 | +24/-2 | 4 |
| Dennis Trautwein | 1 | +20/-4 | 2 |
| galargh | 2 | +18/-2 | 2 |
| M. Hawn | 3 | +10/-10 | 7 |
| Can ZHANG | 1 | +12/-3 | 1 |
| Masih H. Derkani | 1 | +4/-10 | 2 |
| gammazero | 1 | +6/-6 | 2 |
| Ikko Ashimine | 1 | +6/-6 | 2 |
| Daniel N | 2 | +6/-5 | 2 |
| watjurk | 1 | +8/-2 | 1 |
| John Steidley | 2 | +4/-4 | 3 |
| Aaron Bieber | 1 | +6/-2 | 1 |
| Kishan Mohanbhai Sagathiya | 1 | +6/-1 | 1 |
| siiky | 3 | +3/-3 | 3 |
| Lucas Molas | 1 | +5/-1 | 1 |
| Kevin Atkinson | 1 | +3/-3 | 1 |
| Aayush Rajasekaran | 1 | +5/-1 | 1 |
| T Mo | 1 | +2/-2 | 1 |
| Piotr Galar | 1 | +2/-2 | 1 |
| Arber Avdullahu | 1 | +2/-2 | 1 |
| Russell Dempsey | 1 | +2/-1 | 1 |
| anders | 1 | +1/-1 | 1 |
| RubenKelevra | 1 | +1/-1 | 1 |
| Jonathan Rudenberg | 1 | +1/-1 | 1 |
| Ettore Di Giacinto | 1 | +2/-0 | 1 |
| Daniel Norman | 1 | +1/-1 | 1 |
| Chawye Hsu | 1 | +1/-1 | 1 |
| Aliabbas Merchant | 1 | +1/-1 | 1 |
| can | 1 | +1/-0 | 1 |
| Ed Mazurek | 1 | +0/-0 | 1 |
