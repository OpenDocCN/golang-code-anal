# go-ipfs 源码解析 38

<!-- omit in toc -->
# Kubo changelog v0.19

## v0.19.2

### Highlights

#### FullRT DHT HTTP Routers

The default HTTP routers are now used when the FullRT DHT client is used. This fixes
the issue where cid.contact is not being queried by default when the accelerated
DHT client was enabled. Read more in ([ipfs/kubo#9841](https://github.com/ipfs/kubo/pull/9841)).

### Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - fix: use default HTTP routers when FullRT DHT client is used (#9841) ([ipfs/kubo#9841](https://github.com/ipfs/kubo/pull/9841))
  - chore: update version

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Gus Eggert | 1 | +65/-53 | 4 |
| Henrique Dias | 1 | +1/-1 | 1 |

## v0.19.1

### 🔦 Highlights

#### DHT Timeouts
In v0.16.0, Kubo added the ability to configure custom content routers and DHTs with the `custom` router type, and as part of this added a default 5 minute timeout to all DHT operations. In some cases with large repos ([example](https://github.com/ipfs/kubo/issues/9722)), this can cause provide and reprovide operations to fail because the timeout is reached. This release removes these timeouts on DHT operations. If users desire these timeouts, they can be added back using [the `custom` router type](https://github.com/ipfs/kubo/blob/master/docs/config.md#routingrouters-parameters).

### Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - chore: update version
  - fix: remove timeout on default DHT operations (#9783) ([ipfs/kubo#9783](https://github.com/ipfs/kubo/pull/9783))
  - chore: update version
- github.com/ipfs/go-blockservice (v0.5.0 -> v0.5.1):
  - chore: release v0.5.1
  - fix: remove busyloop in getBlocks by removing batching
- github.com/libp2p/go-libp2p (v0.26.3 -> v0.26.4):
  - release v0.26.4
  - autorelay: fix busy loop bug and flaky tests in relay finder (#2208) ([libp2p/go-libp2p#2208](https://github.com/libp2p/go-libp2p/pull/2208))
- github.com/libp2p/go-libp2p-routing-helpers (v0.6.1 -> v0.6.2):
  - Release v0.6.2 (#73) ([libp2p/go-libp2p-routing-helpers#73](https://github.com/libp2p/go-libp2p-routing-helpers/pull/73))
  - feat: zero timeout on composed routers should disable timeout (#72) ([libp2p/go-libp2p-routing-helpers#72](https://github.com/libp2p/go-libp2p-routing-helpers/pull/72))

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marco Munizaga | 1 | +347/-46 | 5 |
| Gus Eggert | 3 | +119/-93 | 8 |
| Jorropo | 2 | +20/-32 | 2 |
| galargh | 2 | +2/-2 | 2 |
| Marten Seemann | 1 | +2/-2 | 1 |

<!-- omit in toc -->
## v0.19.0

- [Overview](#overview)
- [🔦 Highlights](#-highlights)
  - [Improving the libp2p resource management integration](#improving-the-libp2p-resource-management-integration)
  - [Gateways](#gateways)
    - [Signed IPNS Record response format](#signed-ipns-record-response-format)
    - [Example fetch and inspect IPNS record](#example-fetch-and-inspect-ipns-record)
  - [Addition of "autoclient" router type](#addition-of-autoclient-router-type)
  - [Deprecation of the `ipfs pubsub` commands and matching HTTP endpoints](#deprecation-of-the-ipfs-pubsub-commands-and-matching-http-endpoints)
- [📝 Changelog](#-changelog)
- [👨‍👩‍👧‍👦 Contributors](#-contributors)

### Overview

### 🔦 Highlights

#### Improving the libp2p resource management integration

There are further followups up on libp2p resource manager improvements in Kubo [0.18.0](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.18.md#improving-libp2p-resource-management-integration-1)
and [0.18.1](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.18.md#improving-libp2p-resource-management-integration):
1. `ipfs swarm limits` and `ipfs swarm stats` have been replaced by `ipfs swarm resources` to provide a single/combined view for limits and their current usage in a more intuitive ordering.
1. Removal of `Swarm.ResourceMgr.Limits` config.  Instead [the power user can specify limits in a .json file that are fed directly to go-libp2p](https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#user-supplied-override-limits).  This allows the power user to take advantage of the [new resource manager types introduced in go-libp2p 0.25](https://github.com/libp2p/go-libp2p/blob/master/CHANGELOG.md#new-resource-manager-types-) including "use default", "unlimited", "block all".
   - Note: we don't expect most users to need these capablities, but they are there if so.
1. [Doc updates](https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md).

#### Gateways

##### Signed IPNS Record response format

This release implements [IPIP-351](https://github.com/ipfs/specs/pull/351) and
adds Gateway support for returning signed (verifiable) `ipns-record` (0x0300)
when `/ipns/{libp2p-key}` is requested with either
`Accept: application/vnd.ipfs.ipns-record` HTTP header
or `?format=ipns-record` URL query parameter.


The Gateway in Kubo already supported [trustless, verifiable retrieval](https://docs.ipfs.tech/reference/http/gateway/#trustless-verifiable-retrieval) of immutable `/ipfs/` namespace.
With `?format=ipns-record`, light HTTP clients are now able to get the same level of verifiability for IPNS websites.

Tooling is limited at the moment, but we are working on [go-libipfs](https://github.com/ipfs/go-libipfs/) examples that illustrate the verifiable HTTP client pattern.

##### Example: fetch IPNS record over HTTP and inspect it with `ipfs name inspect --verify`

```console
$ FILE_CID=$(echo "Hello IPFS" | ipfs add --cid-version 1 -q)
$ IPNS_KEY=$(ipfs key gen test)
$ ipfs name publish /ipfs/$FILE_CID --key=test --ttl=30m
Published to k51q..dvf1: /ipfs/bafk..z244
$ curl "http://127.0.0.1:8080/ipns/$IPNS_KEY?format=ipns-record" > signed.ipns-record
$ ipfs name inspect --verify $IPNS_KEY < signed.ipns-record
Value:         "/ipfs/bafk..."
Validity Type: "EOL"
Validity:      2023-03-09T23:13:34.032977468Z
Sequence:      0
TTL:           1800000000000
PublicKey:     ""
Signature V1:  "m..."
Signature V2:  "m..."
Data:          {...}

Validation results:
 Valid:     true
 PublicKey: 12D3...
```

#### Addition of "autoclient" router type
A new routing type "autoclient" has been added. This mode is similar to "auto", in that it is a hybrid of content routers (including Kademlia and HTTP routers), but it does not run a DHT server. This is similar to the difference between "dhtclient" and "dht" router types.

See the [Routing.Type documentation](https://github.com/ipfs/kubo/blob/master/docs/config.md#routingtype) for more information.

#### Deprecation of the `ipfs pubsub` commands and matching HTTP endpoints

We are deprecating `ipfs pubsub` and all `/api/v0/pubsub/` RPC endpoints and will remove them in the next release.

For more information and rational see [#9717](https://github.com/ipfs/kubo/issues/9717).

### 📝 Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - chore: update version
  - docs: 0.19 changelog ([ipfs/kubo#9707](https://github.com/ipfs/kubo/pull/9707))
  - fix: canonicalize user defined headers
  - fix: apply API.HTTPHeaders to /webui redirect
  - feat: add heap allocs to 'ipfs diag profile'
  - fix: future proof with > rcmgr.DefaultLimit for new enum rcmgr values
  - test: add test for presarvation of unlimited configs for inbound systems
  - fix: preserve Unlimited StreamsInbound in connmgr reconciliation
  - test: fix flaky rcmgr test
  - chore: deprecate the pubsub api
  - test: port peering test from sharness to Go
  - test: use `T.TempDir` to create temporary test directory
  - fix: --verify forgets the verified key
  - test: name --verify forgets the verified key
  - feat: add "autoclient" routing type
  - test: parallelize more of rcmgr Go tests
  - test: port legacy DHT tests to Go
  - fix: t0116-gateway-cache.sh ([ipfs/kubo#9696](https://github.com/ipfs/kubo/pull/9696))
  - docs: add bifrost to early testers ([ipfs/kubo#9699](https://github.com/ipfs/kubo/pull/9699))
  - fix: typo in documentation for install path
  - chore: update version
  - feat: Reduce RM code footprint
  - Doc updates/additions
  - ci: replace junit html generation with gh action
  - test: port rcmgr sharness tests to Go
  - test(gateway): use deterministic CAR fixtures ([ipfs/kubo#9657](https://github.com/ipfs/kubo/pull/9657))
  - feat(gateway): error handling improvements (500, 502, 504) (#9660) ([ipfs/kubo#9660](https://github.com/ipfs/kubo/pull/9660))
  - docs: be clear about swarm.addrfilters (#9661) ([ipfs/kubo#9661](https://github.com/ipfs/kubo/pull/9661))
  - chore: update go-libp2p to v0.26 (#9656) ([ipfs/kubo#9656](https://github.com/ipfs/kubo/pull/9656))
  - feat(pinning): connect some missing go context (#9557) ([ipfs/kubo#9557](https://github.com/ipfs/kubo/pull/9557))
  - fix(gateway): return HTTP 500 on ErrResolveFailed (#9589) ([ipfs/kubo#9589](https://github.com/ipfs/kubo/pull/9589))
  - docs: bulk spelling edits (#9544) ([ipfs/kubo#9544](https://github.com/ipfs/kubo/pull/9544))
  - docs: "remote" errors from resource manager (#9653) ([ipfs/kubo#9653](https://github.com/ipfs/kubo/pull/9653))
  - test: remove gateway tests migrated to go-libipfs
  - fix: update rcmgr for go-libp2p v0.25
  - chore: update go-libp2p to v0.25.1
  - docs(0.18.1): guide users to clean up limits (#9644) ([ipfs/kubo#9644](https://github.com/ipfs/kubo/pull/9644))
  - feat: add NewOptionalInteger function
  - fix: dereference int64 pointer in OptionalInteger.String() (#9640) ([ipfs/kubo#9640](https://github.com/ipfs/kubo/pull/9640))
  - fix: restore wire format for /api/v0/routing/get|put (#9639) ([ipfs/kubo#9639](https://github.com/ipfs/kubo/pull/9639))
  - refactor(gw): move Host (DNSLink and subdomain) handling to go-libipfs (#9624) ([ipfs/kubo#9624](https://github.com/ipfs/kubo/pull/9624))
  - refactor: new go-libipfs/gateway API, deprecate Gateway.Writable (#9616) ([ipfs/kubo#9616](https://github.com/ipfs/kubo/pull/9616))
  - Create Changelog: v0.19 ([ipfs/kubo#9617](https://github.com/ipfs/kubo/pull/9617))
  - refactor: use gateway from go-libipfs (#9588) ([ipfs/kubo#9588](https://github.com/ipfs/kubo/pull/9588))
  - Merge Release: v0.18.1 ([ipfs/kubo#9613](https://github.com/ipfs/kubo/pull/9613))
  - Add overview section
  - Adjust inbound connection limits depending on memory.
  - feat: ipfs-webui 2.22.0
  - chore: bump go-libipfs remove go-bitswap
  - docs: DefaultResourceMgrMinInboundConns
  - feat(gateway): IPNS record response format (IPIP-351) (#9399) ([ipfs/kubo#9399](https://github.com/ipfs/kubo/pull/9399))
  - fix(ipns): honour --ttl flag in 'ipfs name publish' (#9471) ([ipfs/kubo#9471](https://github.com/ipfs/kubo/pull/9471))
  - feat: Pubsub.SeenMessagesStrategy (#9543) ([ipfs/kubo#9543](https://github.com/ipfs/kubo/pull/9543))
  - chore: bump go-libipfs to replace go-block-format
  - Merge Kubo: v0.18 ([ipfs/kubo#9581](https://github.com/ipfs/kubo/pull/9581))
  - fix: clarity: no user supplied rcmgr limits of 0 (#9563) ([ipfs/kubo#9563](https://github.com/ipfs/kubo/pull/9563))
  - fix(gateway): undesired conversions to dag-json and friends (#9566) ([ipfs/kubo#9566](https://github.com/ipfs/kubo/pull/9566))
  - fix: ensure connmgr is smaller then autoscalled ressource limits
  - fix: typo in ensureConnMgrMakeSenseVsResourcesMgr
  - docs: clarify browser descriptions for webtransport
  - fix: update saxon download path
  - fix: refuse to start if connmgr is smaller than ressource limits and not using none connmgr
  - fix: User-Agent sent to HTTP routers
  - test: port gateway sharness tests to Go tests
  - fix: do not download saxon in parallel
  - docs: improve docs/README (#9539) ([ipfs/kubo#9539](https://github.com/ipfs/kubo/pull/9539))
  - test: port CircleCI to GH Actions and improve sharness reporting (#9355) ([ipfs/kubo#9355](https://github.com/ipfs/kubo/pull/9355))
  - chore: migrate from go-ipfs-files to go-libipfs/files (#9535) ([ipfs/kubo#9535](https://github.com/ipfs/kubo/pull/9535))
  - fix: stats dht command when Routing.Type=auto (#9538) ([ipfs/kubo#9538](https://github.com/ipfs/kubo/pull/9538))
  - fix: hint people to changing from RSA peer ids
  - fix(gateway): JSON when Accept is a list
  - fix(test): retry flaky t0125-twonode.sh
  - docs: fix Router config Godoc (#9528) ([ipfs/kubo#9528](https://github.com/ipfs/kubo/pull/9528))
  - fix(ci): flaky sharness test
  - docs(config): ProviderSearchDelay (#9526) ([ipfs/kubo#9526](https://github.com/ipfs/kubo/pull/9526))
  - docs: clarify debug environment variables
  - fix: disable provide over HTTP with Routing.Type=auto (#9511) ([ipfs/kubo#9511](https://github.com/ipfs/kubo/pull/9511))
  - fix(test): stabilize flaky provider tests
  - feat: port pins CLI test
  - Removing QRI from early tester ([ipfs/kubo#9503](https://github.com/ipfs/kubo/pull/9503))
  - Update Version (dev): v0.18 ([ipfs/kubo#9500](https://github.com/ipfs/kubo/pull/9500))
- github.com/ipfs/go-bitfield (v1.0.0 -> v1.1.0):
  - Merge pull request from GHSA-2h6c-j3gf-xp9r
  - sync: update CI config files (#3) ([ipfs/go-bitfield#3](https://github.com/ipfs/go-bitfield/pull/3))
- github.com/ipfs/go-block-format (v0.0.3 -> v0.1.1):
  - chore: release v0.1.1
  - docs: fix wrong copy paste in docs
  - chore: release v0.1.0
  - refactor: deprecate and add stub types to go-libipfs/blocks
  - sync: update CI config files (#34) ([ipfs/go-block-format#34](https://github.com/ipfs/go-block-format/pull/34))
  - remove Makefile ([ipfs/go-block-format#31](https://github.com/ipfs/go-block-format/pull/31))
- github.com/ipfs/go-ipfs-files (v0.0.8 -> v0.3.0):
  -  ([ipfs/go-ipfs-files#59](https://github.com/ipfs/go-ipfs-files/pull/59))
  - docs: add moved noticed [ci skip]
  - Release v0.2.0
  - fix: error when TAR has files outside of root (#56) ([ipfs/go-ipfs-files#56](https://github.com/ipfs/go-ipfs-files/pull/56))
  - sync: update CI config files ([ipfs/go-ipfs-files#55](https://github.com/ipfs/go-ipfs-files/pull/55))
  - chore(Directory): add DirIterator API restriction: iterate only once
  - Release v0.1.1
  - fix: add dragonfly build option for filewriter flags
  - fix: add freebsd build option for filewriter flags
  - Release v0.1.0
  - docs: fix community CONTRIBUTING.md link (#45) ([ipfs/go-ipfs-files#45](https://github.com/ipfs/go-ipfs-files/pull/45))
  - chore(filewriter): cleanup writes (#43) ([ipfs/go-ipfs-files#43](https://github.com/ipfs/go-ipfs-files/pull/43))
  - sync: update CI config files (#44) ([ipfs/go-ipfs-files#44](https://github.com/ipfs/go-ipfs-files/pull/44))
  - sync: update CI config files ([ipfs/go-ipfs-files#40](https://github.com/ipfs/go-ipfs-files/pull/40))
  - fix: manually parse the content disposition to preserve directories ([ipfs/go-ipfs-files#42](https://github.com/ipfs/go-ipfs-files/pull/42))
  - fix: round timestamps down by truncating them to seconds ([ipfs/go-ipfs-files#41](https://github.com/ipfs/go-ipfs-files/pull/41))
  - sync: update CI config files ([ipfs/go-ipfs-files#34](https://github.com/ipfs/go-ipfs-files/pull/34))
  - Fix test failure on Windows caused by nil `sys` in mock `FileInfo` ([ipfs/go-ipfs-files#39](https://github.com/ipfs/go-ipfs-files/pull/39))
  - fix staticcheck ([ipfs/go-ipfs-files#35](https://github.com/ipfs/go-ipfs-files/pull/35))
  - fix linters ([ipfs/go-ipfs-files#33](https://github.com/ipfs/go-ipfs-files/pull/33))
- github.com/ipfs/go-ipfs-pinner (v0.2.1 -> v0.3.0):
  - chore: release v0.3.0 (#27) ([ipfs/go-ipfs-pinner#27](https://github.com/ipfs/go-ipfs-pinner/pull/27))
  - feat!: add and connect missing context, remove RemovePinWithMode (#23) ([ipfs/go-ipfs-pinner#23](https://github.com/ipfs/go-ipfs-pinner/pull/23))
  - sync: update CI config files ([ipfs/go-ipfs-pinner#16](https://github.com/ipfs/go-ipfs-pinner/pull/16))
- github.com/ipfs/go-ipfs-pq (v0.0.2 -> v0.0.3):
  - chore: release v0.0.3
  - fix: enable early GC
  - sync: update CI config files (#10) ([ipfs/go-ipfs-pq#10](https://github.com/ipfs/go-ipfs-pq/pull/10))
  - sync: update CI config files ([ipfs/go-ipfs-pq#8](https://github.com/ipfs/go-ipfs-pq/pull/8))
  - remove Makefile ([ipfs/go-ipfs-pq#7](https://github.com/ipfs/go-ipfs-pq/pull/7))
- github.com/ipfs/go-libipfs (v0.2.0 -> v0.6.2):
  - chore: release 0.6.2 (#211) ([ipfs/go-libipfs#211](https://github.com/ipfs/go-libipfs/pull/211))
  - fix(gateway): 500 on panic, recover on WithHostname
  - refactor: use assert in remaining gateway tests
  - chore: release 0.6.1
  - feat: support HTTP 429 with Retry-After (#194) ([ipfs/go-libipfs#194](https://github.com/ipfs/go-libipfs/pull/194))
  - docs: fix typo in README.md
  - fix(gateway): return 500 for all /ip[nf]s/id failures
  - chore: make gocritic happier
  - feat(gateway): improved error handling, support for 502 and 504 ([ipfs/go-libipfs#182](https://github.com/ipfs/go-libipfs/pull/182))
  - feat: add content path in request context (#184) ([ipfs/go-libipfs#184](https://github.com/ipfs/go-libipfs/pull/184))
  - sync: update CI config files ([ipfs/go-libipfs#159](https://github.com/ipfs/go-libipfs/pull/159))
  - fix(gateway): return HTTP 500 on namesys.ErrResolveFailed (#150) ([ipfs/go-libipfs#150](https://github.com/ipfs/go-libipfs/pull/150))
  - docs(examples): add UnixFS file download over Bitswap (#143) ([ipfs/go-libipfs#143](https://github.com/ipfs/go-libipfs/pull/143))
  - bitswap/server/internal/decision: fix: remove unused private type
  - chore: release v0.6.0
  - bitswap/server/internal/decision: add more non flaky tests
  - bitswap/server/internal/decision: add filtering on CIDs - Ignore cids that are too big. - Kill connection for peers that are using inline CIDs.
  - bitswap/server/internal/decision: rewrite ledger inversion
  - docs(readme): various updates for clarity (#171) ([ipfs/go-libipfs#171](https://github.com/ipfs/go-libipfs/pull/171))
  - feat: metric for implicit index.html in dirs
  - fix(gateway): ensure ipfs_http_gw_get_duration_seconds gets updated
  - test(gateway): migrate Go tests from Kubo ([ipfs/go-libipfs#156](https://github.com/ipfs/go-libipfs/pull/156))
  - docs: fix  link (#165) ([ipfs/go-libipfs#165](https://github.com/ipfs/go-libipfs/pull/165))
  - fix: GetIPNSRecord example gateway implementation (#158) ([ipfs/go-libipfs#158](https://github.com/ipfs/go-libipfs/pull/158))
  - chore: release v0.5.0
  - chore: update go-libp2p to v0.25.1
  - fix(gateway): display correct error with 500 (#160) ([ipfs/go-libipfs#160](https://github.com/ipfs/go-libipfs/pull/160))
  - fix: gateway car example dnslink
  - feat(gateway): add TAR, IPNS Record, DAG-* histograms and spans (#155) ([ipfs/go-libipfs#155](https://github.com/ipfs/go-libipfs/pull/155))
  - feat(gateway): migrate subdomain and dnslink code (#153) ([ipfs/go-libipfs#153](https://github.com/ipfs/go-libipfs/pull/153))
  - docs: add example of gateway that proxies to ?format=raw (#151) ([ipfs/go-libipfs#151](https://github.com/ipfs/go-libipfs/pull/151))
  - docs: add example of gateway backed by CAR file (#147) ([ipfs/go-libipfs#147](https://github.com/ipfs/go-libipfs/pull/147))
  - undefined ([ipfs/go-libipfs#145](https://github.com/ipfs/go-libipfs/pull/145))
  - Extract Gateway Code From Kubo
 ([ipfs/go-libipfs#65](https://github.com/ipfs/go-libipfs/pull/65))
  - Migrate go-bitswap ([ipfs/go-libipfs#63](https://github.com/ipfs/go-libipfs/pull/63))
  - Use `PUT` as method to insert provider records
  - Migrate `go-block-format` ([ipfs/go-libipfs#58](https://github.com/ipfs/go-libipfs/pull/58))
  - chore: add codecov PR comment
  - chore: add a logo and some basics in the README (#37) ([ipfs/go-libipfs#37](https://github.com/ipfs/go-libipfs/pull/37))
- github.com/ipfs/go-namesys (v0.6.0 -> v0.7.0):
  - chore: release 0.7.0 (#36) ([ipfs/go-namesys#36](https://github.com/ipfs/go-namesys/pull/36))
  - feat: use PublishOptions for publishing IPNS records (#35) ([ipfs/go-namesys#35](https://github.com/ipfs/go-namesys/pull/35))
- github.com/ipfs/go-path (v0.3.0 -> v0.3.1):
  - chore: release v0.3.1 (#67) ([ipfs/go-path#67](https://github.com/ipfs/go-path/pull/67))
  - feat: expose ErrInvalidPath and implement .Is function (#66) ([ipfs/go-path#66](https://github.com/ipfs/go-path/pull/66))
  - sync: update CI config files (#60) ([ipfs/go-path#60](https://github.com/ipfs/go-path/pull/60))
  - feat: add basic tracing ([ipfs/go-path#59](https://github.com/ipfs/go-path/pull/59))
- github.com/ipfs/go-peertaskqueue (v0.8.0 -> v0.8.1):
  - chore: release v0.8.1
  - feat: add PushTasksTruncated which only push a limited amount of tasks
  - feat: add (*PeerTaskQueue).Clear which fully removes a peer
  - sync: update CI config files (#26) ([ipfs/go-peertaskqueue#26](https://github.com/ipfs/go-peertaskqueue/pull/26))
- github.com/ipfs/go-unixfs (v0.4.2 -> v0.4.4):
  - chore: release v0.4.4
  - fix: correctly handle return errors
  - fix: correctly handle errors in balancedbuilder's Layout
  - test: fix tests after hamt issues fixes
  - Merge pull request from GHSA-q264-w97q-q778
- github.com/ipfs/go-unixfsnode (v1.5.1 -> v1.5.2):
  - Merge pull request from GHSA-4gj3-6r43-3wfc
- github.com/ipfs/interface-go-ipfs-core (v0.8.2 -> v0.11.0):
  - chore: release v0.11.0
  - test: basic routing interface test
  - chore: release v0.10.0 (#102) ([ipfs/interface-go-ipfs-core#102](https://github.com/ipfs/interface-go-ipfs-core/pull/102))
  - feat: add RoutingAPI to CoreAPI
  - chore: release 0.9.0 (#101) ([ipfs/interface-go-ipfs-core#101](https://github.com/ipfs/interface-go-ipfs-core/pull/101))
  - feat: add namesys publish options (#94) ([ipfs/interface-go-ipfs-core#94](https://github.com/ipfs/interface-go-ipfs-core/pull/94))
- github.com/ipld/go-car (v0.4.0 -> v0.5.0):
  - chore: bump version to 0.5.0
  - fix: remove use of ioutil
  - run gofmt -s
  - bump go.mod to Go 1.18 and run go fix
  - bump go.mod to Go 1.18 and run go fix
  - OpenReadWriteFile: add test
  - blockstore: allow to pass a file to write in (#323) ([ipld/go-car#323](https://github.com/ipld/go-car/pull/323))
  - feat: add `car inspect` command to cmd pkg (#320) ([ipld/go-car#320](https://github.com/ipld/go-car/pull/320))
  - Separate `index.ReadFrom` tests
  - Only read index codec during inspection
  - Upgrade to the latest `go-car/v2`
  - Empty identity CID should be indexed when options are set
- github.com/libp2p/go-libp2p (v0.24.2 -> v0.26.3):
  - Release v0.26.3 (#2197) ([libp2p/go-libp2p#2197](https://github.com/libp2p/go-libp2p/pull/2197))
  - retract v0.26.1, release v0.26.2 (#2153) ([libp2p/go-libp2p#2153](https://github.com/libp2p/go-libp2p/pull/2153))
  - rcmgr: fix JSON marshalling of ResourceManagerStat peer map (#2156) ([libp2p/go-libp2p#2156](https://github.com/libp2p/go-libp2p/pull/2156))
  - release v0.26.1 ([libp2p/go-libp2p#2146](https://github.com/libp2p/go-libp2p/pull/2146))
  - release v0.26.0 (#2133) ([libp2p/go-libp2p#2133](https://github.com/libp2p/go-libp2p/pull/2133))
  - identify: add more detailed metrics (#2126) ([libp2p/go-libp2p#2126](https://github.com/libp2p/go-libp2p/pull/2126))
  - autorelay: refactor relay finder and start autorelay after identify (#2120) ([libp2p/go-libp2p#2120](https://github.com/libp2p/go-libp2p/pull/2120))
  - don't use the time value from the time.Ticker channel (#2127) ([libp2p/go-libp2p#2127](https://github.com/libp2p/go-libp2p/pull/2127))
  - Wrap conn with metrics (#2131) ([libp2p/go-libp2p#2131](https://github.com/libp2p/go-libp2p/pull/2131))
  - chore: update changelog for 0.26.0 (#2132) ([libp2p/go-libp2p#2132](https://github.com/libp2p/go-libp2p/pull/2132))
  - circuitv2: Update proto files to proto3 (#2121) ([libp2p/go-libp2p#2121](https://github.com/libp2p/go-libp2p/pull/2121))
  - swarm: remove parallel tests from swarm tests (#2130) ([libp2p/go-libp2p#2130](https://github.com/libp2p/go-libp2p/pull/2130))
  - circuitv2: add a relay option to disable limits (#2125) ([libp2p/go-libp2p#2125](https://github.com/libp2p/go-libp2p/pull/2125))
  - quic: fix stalled virtual listener (#2122) ([libp2p/go-libp2p#2122](https://github.com/libp2p/go-libp2p/pull/2122))
  - swarm: add early muxer selection to swarm metrics (#2119) ([libp2p/go-libp2p#2119](https://github.com/libp2p/go-libp2p/pull/2119))
  - metrics: add options to disable metrics and to set Prometheus registerer (#2116) ([libp2p/go-libp2p#2116](https://github.com/libp2p/go-libp2p/pull/2116))
  - swarm: add ip_version to metrics (#2114) ([libp2p/go-libp2p#2114](https://github.com/libp2p/go-libp2p/pull/2114))
  - Revert mistaken "Bump timeout"
  - Bump timeout
  - remove all circuit v1 related code (#2107) ([libp2p/go-libp2p#2107](https://github.com/libp2p/go-libp2p/pull/2107))
  - quic: don't send detailed error messages when closing connections (#2112) ([libp2p/go-libp2p#2112](https://github.com/libp2p/go-libp2p/pull/2112))
  - metrics: add no alloc metrics for eventbus, swarm, identify (#2108) ([libp2p/go-libp2p#2108](https://github.com/libp2p/go-libp2p/pull/2108))
  - chore: fix typo in Changelog (#2111) ([libp2p/go-libp2p#2111](https://github.com/libp2p/go-libp2p/pull/2111))
  - chore: update changelog (#2109) ([libp2p/go-libp2p#2109](https://github.com/libp2p/go-libp2p/pull/2109))
  - chore: unify dashboard location (#2110) ([libp2p/go-libp2p#2110](https://github.com/libp2p/go-libp2p/pull/2110))
  - autonat: add metrics (#2086) ([libp2p/go-libp2p#2086](https://github.com/libp2p/go-libp2p/pull/2086))
  - relaymanager: do not start new relay if one already exists (#2093) ([libp2p/go-libp2p#2093](https://github.com/libp2p/go-libp2p/pull/2093))
  - autonat: don't emit reachability changed events on address change (#2092) ([libp2p/go-libp2p#2092](https://github.com/libp2p/go-libp2p/pull/2092))
  - chore: modify changelog entries (#2101) ([libp2p/go-libp2p#2101](https://github.com/libp2p/go-libp2p/pull/2101))
  - Introduce a changelog (#2084) ([libp2p/go-libp2p#2084](https://github.com/libp2p/go-libp2p/pull/2084))
  - use atomic.Int32 and atomic.Int64 (#2096) ([libp2p/go-libp2p#2096](https://github.com/libp2p/go-libp2p/pull/2096))
  - change atomic.Value to atomic.Pointer (#2088) ([libp2p/go-libp2p#2088](https://github.com/libp2p/go-libp2p/pull/2088))
  - use atomic.Bool instead of int32 operations (#2089) ([libp2p/go-libp2p#2089](https://github.com/libp2p/go-libp2p/pull/2089))
  - sync: update CI config files (#2073) ([libp2p/go-libp2p#2073](https://github.com/libp2p/go-libp2p/pull/2073))
  - chore: update examples to v0.25.1 (#2080) ([libp2p/go-libp2p#2080](https://github.com/libp2p/go-libp2p/pull/2080))
  - v0.25.1 (#2082) ([libp2p/go-libp2p#2082](https://github.com/libp2p/go-libp2p/pull/2082))
  - Start host in mocknet (#2078) ([libp2p/go-libp2p#2078](https://github.com/libp2p/go-libp2p/pull/2078))
  - Release v0.25.0 (#2077) ([libp2p/go-libp2p#2077](https://github.com/libp2p/go-libp2p/pull/2077))
  - identify: add some basic metrics (#2069) ([libp2p/go-libp2p#2069](https://github.com/libp2p/go-libp2p/pull/2069))
  - p2p/test/quic: use contexts with a timeout for Connect calls (#2070) ([libp2p/go-libp2p#2070](https://github.com/libp2p/go-libp2p/pull/2070))
  - feat!: rcmgr: Change LimitConfig to use LimitVal type (#2000) ([libp2p/go-libp2p#2000](https://github.com/libp2p/go-libp2p/pull/2000))
  - identify: refactor sending of Identify pushes (#1984) ([libp2p/go-libp2p#1984](https://github.com/libp2p/go-libp2p/pull/1984))
  - Update interop to match spec (#2049) ([libp2p/go-libp2p#2049](https://github.com/libp2p/go-libp2p/pull/2049))
  - chore: git-ignore various flavors of qlog files (#2064) ([libp2p/go-libp2p#2064](https://github.com/libp2p/go-libp2p/pull/2064))
  - rcmgr: add libp2p prefix to all metrics (#2063) ([libp2p/go-libp2p#2063](https://github.com/libp2p/go-libp2p/pull/2063))
  - websocket: Replace gorilla websocket transport with nhooyr websocket transport (#1982) ([libp2p/go-libp2p#1982](https://github.com/libp2p/go-libp2p/pull/1982))
  - rcmgr: Use prometheus SDK for rcmgr metrics (#2044) ([libp2p/go-libp2p#2044](https://github.com/libp2p/go-libp2p/pull/2044))
  - autorelay: Split libp2p.EnableAutoRelay into 2 functions (#2022) ([libp2p/go-libp2p#2022](https://github.com/libp2p/go-libp2p/pull/2022))
  - set names for eventbus event subscriptions (#2057) ([libp2p/go-libp2p#2057](https://github.com/libp2p/go-libp2p/pull/2057))
  - Test cleanup (#2053) ([libp2p/go-libp2p#2053](https://github.com/libp2p/go-libp2p/pull/2053))
  - metrics: use a single slice pool for all metrics tracer (#2054) ([libp2p/go-libp2p#2054](https://github.com/libp2p/go-libp2p/pull/2054))
  - eventbus: add metrics (#2038) ([libp2p/go-libp2p#2038](https://github.com/libp2p/go-libp2p/pull/2038))
  - quic: disable sending of Version Negotiation packets (#2015) ([libp2p/go-libp2p#2015](https://github.com/libp2p/go-libp2p/pull/2015))
  - p2p/test: fix flaky notification test (#2051) ([libp2p/go-libp2p#2051](https://github.com/libp2p/go-libp2p/pull/2051))
  - quic, tcp: only register Prometheus counters when metrics are enabled ([libp2p/go-libp2p#1971](https://github.com/libp2p/go-libp2p/pull/1971))
  - p2p/test: add test for EvtLocalAddressesUpdated event (#2016) ([libp2p/go-libp2p#2016](https://github.com/libp2p/go-libp2p/pull/2016))
  - quic / webtransport: extend test to test dialing draft-29 and v1 (#1957) ([libp2p/go-libp2p#1957](https://github.com/libp2p/go-libp2p/pull/1957))
  - holepunch: fix flaky by not remove holepunch protocol handler (#1948) ([libp2p/go-libp2p#1948](https://github.com/libp2p/go-libp2p/pull/1948))
  - use quic-go and webtransport-go from quic-go organization (#2040) ([libp2p/go-libp2p#2040](https://github.com/libp2p/go-libp2p/pull/2040))
  - Migrate to test-plan composite action (#2039) ([libp2p/go-libp2p#2039](https://github.com/libp2p/go-libp2p/pull/2039))
  - chore: remove license files from the eventbus package (#2042) ([libp2p/go-libp2p#2042](https://github.com/libp2p/go-libp2p/pull/2042))
  - rcmgr: *: Always close connscope (#2037) ([libp2p/go-libp2p#2037](https://github.com/libp2p/go-libp2p/pull/2037))
  - chore: remove textual roadmap in favor for Starmap (#2036) ([libp2p/go-libp2p#2036](https://github.com/libp2p/go-libp2p/pull/2036))
  - swarm metrics: fix datasource for dashboard (#2024) ([libp2p/go-libp2p#2024](https://github.com/libp2p/go-libp2p/pull/2024))
  - consistently use protocol.ID instead of strings (#2004) ([libp2p/go-libp2p#2004](https://github.com/libp2p/go-libp2p/pull/2004))
  - swarm: add a basic metrics tracer (#1973) ([libp2p/go-libp2p#1973](https://github.com/libp2p/go-libp2p/pull/1973))
  - Expose muxer ids (#2012) ([libp2p/go-libp2p#2012](https://github.com/libp2p/go-libp2p/pull/2012))
  - Clean addresses with peer id before adding to addrbook (#2007) ([libp2p/go-libp2p#2007](https://github.com/libp2p/go-libp2p/pull/2007))
  - feat: ci test-plans: Parse test timeout parameter for interop test (#2014) ([libp2p/go-libp2p#2014](https://github.com/libp2p/go-libp2p/pull/2014))
  - Export resource manager errors (#2008) ([libp2p/go-libp2p#2008](https://github.com/libp2p/go-libp2p/pull/2008))
  - peerstore: make it possible to use an empty peer ID (#2006) ([libp2p/go-libp2p#2006](https://github.com/libp2p/go-libp2p/pull/2006))
  - Add ci flakiness score to readme (#2002) ([libp2p/go-libp2p#2002](https://github.com/libp2p/go-libp2p/pull/2002))
  - rcmgr: fix: Ignore zero values when marshalling Limits. (#1998) ([libp2p/go-libp2p#1998](https://github.com/libp2p/go-libp2p/pull/1998))
  - CI: Fast multidimensional Interop tests (#1991) ([libp2p/go-libp2p#1991](https://github.com/libp2p/go-libp2p/pull/1991))
  - feat: add some users to the readme (#1981) ([libp2p/go-libp2p#1981](https://github.com/libp2p/go-libp2p/pull/1981))
  - ci: run go generate as part of the go-check workflow (#1986) ([libp2p/go-libp2p#1986](https://github.com/libp2p/go-libp2p/pull/1986))
  - switch to Google's Protobuf library, make protobufs compile with go generate ([libp2p/go-libp2p#1979](https://github.com/libp2p/go-libp2p/pull/1979))
  - circuitv2: correctly set the transport in the ConnectionState (#1972) ([libp2p/go-libp2p#1972](https://github.com/libp2p/go-libp2p/pull/1972))
  - roadmap: remove optimizations of the TCP-based handshake (#1959) ([libp2p/go-libp2p#1959](https://github.com/libp2p/go-libp2p/pull/1959))
  - identify: remove support for Identify Delta ([libp2p/go-libp2p#1975](https://github.com/libp2p/go-libp2p/pull/1975))
  - core: remove introspection package (#1978) ([libp2p/go-libp2p#1978](https://github.com/libp2p/go-libp2p/pull/1978))
  - identify: remove old code targeting Go 1.17 (#1964) ([libp2p/go-libp2p#1964](https://github.com/libp2p/go-libp2p/pull/1964))
  - add WebTransport to the list of default transports (#1915) ([libp2p/go-libp2p#1915](https://github.com/libp2p/go-libp2p/pull/1915))
  - core/crypto: drop all OpenSSL code paths (#1953) ([libp2p/go-libp2p#1953](https://github.com/libp2p/go-libp2p/pull/1953))
  - chore: use generic LRU cache (#1980) ([libp2p/go-libp2p#1980](https://github.com/libp2p/go-libp2p/pull/1980))
- github.com/libp2p/go-libp2p-kad-dht (v0.20.0 -> v0.21.1):
  - chore: bump to v0.21.1 (#821) ([libp2p/go-libp2p-kad-dht#821](https://github.com/libp2p/go-libp2p-kad-dht/pull/821))
  - feat: send FIND_NODE request to peers on routing table refresh (#810) ([libp2p/go-libp2p-kad-dht#810](https://github.com/libp2p/go-libp2p-kad-dht/pull/810))
  - chore: release v0.21.
  - chore: Update to go libp2p v0.25 ([libp2p/go-libp2p-kad-dht#815](https://github.com/libp2p/go-libp2p-kad-dht/pull/815))
- github.com/libp2p/go-libp2p-pubsub (v0.8.3 -> v0.9.0):
  - chore: update to go-libp2p v0.25 (#517) ([libp2p/go-libp2p-pubsub#517](https://github.com/libp2p/go-libp2p-pubsub/pull/517))
- github.com/libp2p/go-libp2p-routing-helpers (v0.6.0 -> v0.6.1):
  - chore: release v0.6.1
  - fix: cancel parallel routers
- github.com/libp2p/go-msgio (v0.2.0 -> v0.3.0):
  - release v0.3.0 (#39) ([libp2p/go-msgio#39](https://github.com/libp2p/go-msgio/pull/39))
  - switch from deprecated gogo to google.golang.org/protobuf ([libp2p/go-msgio#38](https://github.com/libp2p/go-msgio/pull/38))
  - sync: update CI config files (#36) ([libp2p/go-msgio#36](https://github.com/libp2p/go-msgio/pull/36))
- github.com/lucas-clemente/quic-go (v0.31.1 -> v0.29.1):
  - http3: fix double close of chan when using DontCloseRequestStream
- github.com/multiformats/go-multistream (v0.3.3 -> v0.4.1):
  - release v0.4.1 ([multiformats/go-multistream#101](https://github.com/multiformats/go-multistream/pull/101))
  - Fix errors Is checking ([multiformats/go-multistream#100](https://github.com/multiformats/go-multistream/pull/100))
  - release v0.4.0 (#93) ([multiformats/go-multistream#93](https://github.com/multiformats/go-multistream/pull/93))
  - switch to Go's native fuzzing (#96) ([multiformats/go-multistream#96](https://github.com/multiformats/go-multistream/pull/96))
  - Add not supported protocols to returned errors (#97) ([multiformats/go-multistream#97](https://github.com/multiformats/go-multistream/pull/97))
  - Make MultistreamMuxer and Client APIs generic (#95) ([multiformats/go-multistream#95](https://github.com/multiformats/go-multistream/pull/95))
  - remove MultistreamMuxer.NegotiateLazy (#92) ([multiformats/go-multistream#92](https://github.com/multiformats/go-multistream/pull/92))
  - sync: update CI config files (#91) ([multiformats/go-multistream#91](https://github.com/multiformats/go-multistream/pull/91))
- github.com/warpfork/go-wish (v0.0.0-20200122115046-b9ea61034e4a -> v0.0.0-20220906213052-39a1cc7a02d0):
  - Update readme with deprecation info
- github.com/whyrusleeping/cbor-gen (v0.0.0-20221220214510-0333c149dec0 -> v0.0.0-20230126041949-52956bd4c9aa):
  - add setter to allow reuse of cborreader struct
  - fix typo
  - allow fields to be ignored ([whyrusleeping/cbor-gen#79](https://github.com/whyrusleeping/cbor-gen/pull/79))

</details>

### 👨‍👩‍👧‍👦 Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Dirk McCormick | 128 | +16757/-7211 | 387 |
| Henrique Dias | 69 | +7599/-10016 | 316 |
| hannahhoward | 88 | +8503/-4397 | 271 |
| Jeromy Johnson | 244 | +6544/-4034 | 774 |
| Marten Seemann | 64 | +4870/-5628 | 266 |
| Steven Allen | 296 | +4769/-3517 | 972 |
| Brian Tiger Chow | 250 | +5520/-2579 | 435 |
| Jorropo | 64 | +4237/-3548 | 302 |
| Sukun | 18 | +4327/-1093 | 132 |
| Marco Munizaga | 35 | +2809/-1294 | 94 |
| Gus Eggert | 20 | +2523/-1476 | 99 |
| Adin Schmahmann | 15 | +683/-2625 | 69 |
| Marcin Rataj | 73 | +2348/-882 | 133 |
| whyrusleeping | 12 | +1683/-1338 | 23 |
| Jeromy | 99 | +1754/-1181 | 453 |
| Juan Batiz-Benet | 69 | +1182/-678 | 149 |
| Lars Gierth | 31 | +827/-358 | 92 |
| Paul Wolneykien | 2 | +670/-338 | 9 |
| Laurent Senta | 16 | +806/-134 | 53 |
| Henry | 19 | +438/-372 | 36 |
| Michael Muré | 8 | +400/-387 | 19 |
| Łukasz Magiera | 56 | +413/-354 | 117 |
| Jakub Sztandera | 40 | +413/-251 | 100 |
| Justin Johnson | 2 | +479/-165 | 5 |
| Piotr Galar | 7 | +227/-378 | 24 |
| Kevin Atkinson | 11 | +252/-232 | 49 |
| web3-bot | 17 | +236/-240 | 59 |
| Petar Maymounkov | 2 | +348/-84 | 11 |
| Hector Sanjuan | 38 | +206/-223 | 85 |
| Antonio Navarro Perez | 9 | +259/-95 | 17 |
| keks | 22 | +233/-118 | 24 |
| Ho-Sheng Hsiao | 3 | +170/-170 | 30 |
| Lucas Molas | 6 | +266/-54 | 16 |
| Mildred Ki'Lya | 4 | +280/-35 | 7 |
| Steve Loeppky | 5 | +147/-156 | 9 |
| rht | 14 | +97/-188 | 20 |
| Prithvi Shahi | 6 | +89/-193 | 11 |
| Ian Davis | 6 | +198/-75 | 11 |
| taylor | 1 | +180/-89 | 8 |
| ᴍᴀᴛᴛ ʙᴇʟʟ | 14 | +158/-104 | 18 |
| Chris Boddy | 6 | +190/-45 | 8 |
| Rod Vagg | 3 | +203/-28 | 15 |
| Masih H. Derkani | 8 | +165/-61 | 16 |
| Kevin Wallace | 4 | +194/-27 | 7 |
| Mohsin Zaidi | 1 | +179/-41 | 5 |
| ElPaisano | 1 | +110/-110 | 22 |
| Simon Zhu | 6 | +177/-32 | 8 |
| galargh | 9 | +80/-120 | 14 |
| Tomasz Zdybał | 1 | +180/-1 | 4 |
| dgrisham | 3 | +176/-2 | 4 |
| Michael Avila | 3 | +116/-59 | 8 |
| Raúl Kripalani | 2 | +85/-77 | 34 |
| Dr Ian Preston | 11 | +101/-48 | 11 |
| JP Hastings-Spital | 1 | +145/-0 | 2 |
| George Antoniadis | 6 | +59/-58 | 43 |
| Kevin Neaton | 2 | +97/-16 | 4 |
| Adrian Lanzafame | 6 | +81/-25 | 7 |
| Dennis Trautwein | 3 | +89/-9 | 5 |
| mathew-cf | 2 | +82/-9 | 5 |
| tg | 1 | +41/-33 | 1 |
| Eng Zer Jun | 1 | +15/-54 | 5 |
| zramsay | 4 | +15/-53 | 12 |
| muXxer | 1 | +28/-33 | 4 |
| Thomas Eizinger | 1 | +24/-37 | 4 |
| Remco Bloemen | 2 | +28/-18 | 3 |
| Manuel Alonso | 1 | +36/-9 | 1 |
| vyzo | 4 | +26/-12 | 13 |
| Djalil Dreamski | 3 | +27/-9 | 3 |
| Thomas Gardner | 2 | +32/-3 | 4 |
| Jan Winkelmann | 2 | +23/-12 | 8 |
| Artem Andreenko | 1 | +16/-19 | 1 |
| James Stanley | 1 | +34/-0 | 1 |
| Brendan McMillion | 1 | +10/-17 | 3 |
| Jack Loughran | 1 | +22/-0 | 3 |
| Peter Wu | 2 | +12/-9 | 2 |
| Gowtham G | 4 | +14/-7 | 4 |
| Tor Arne Vestbø | 3 | +19/-1 | 3 |
| Cory Schwartz | 1 | +8/-12 | 5 |
| Peter Rabbitson | 1 | +15/-4 | 1 |
| David Dias | 1 | +9/-9 | 1 |
| Will Scott | 1 | +13/-4 | 2 |
| Eric Myhre | 1 | +15/-2 | 1 |
| Stephen Whitmore | 1 | +8/-8 | 1 |
| Rafael Ramalho | 5 | +11/-5 | 5 |
| Christian Couder | 1 | +14/-2 | 1 |
| W. Trevor King | 2 | +9/-6 | 3 |
| Steven Vandevelde | 1 | +11/-3 | 1 |
| Knut Ahlers | 3 | +9/-5 | 3 |
| Bob Potter | 1 | +3/-10 | 1 |
| Russell Dempsey | 4 | +8/-4 | 4 |
| Diogo Silva | 4 | +8/-4 | 4 |
| Dave Justice | 1 | +8/-4 | 1 |
| Andy Leap | 2 | +2/-10 | 2 |
| divingpetrel | 1 | +7/-4 | 2 |
| Iaroslav Gridin | 1 | +9/-2 | 1 |
| Dominic Della Valle | 3 | +5/-5 | 3 |
| Vijayee Kulkaa | 1 | +3/-6 | 1 |
| Friedel Ziegelmayer | 3 | +6/-3 | 3 |
| Stephen Solka | 1 | +1/-7 | 1 |
| Richard Littauer | 3 | +4/-4 | 3 |
| Franky W | 2 | +4/-4 | 2 |
| Dimitris Apostolou | 2 | +4/-4 | 3 |
| Adrian Ulrich | 1 | +8/-0 | 1 |
| Masashi Salvador Mitsuzawa | 1 | +5/-1 | 1 |
| Gabe | 1 | +3/-3 | 1 |
| zuuluuz | 1 | +4/-1 | 1 |
| myml | 1 | +5/-0 | 1 |
| swedneck | 1 | +3/-1 | 1 |
| Wayback Archiver | 1 | +2/-2 | 1 |
| Vladimir Ivanov | 1 | +2/-2 | 1 |
| Péter Szilágyi | 1 | +2/-2 | 1 |
| Karthik Bala | 1 | +2/-2 | 1 |
| Etienne Laurin | 1 | +1/-3 | 1 |
| Shotaro Yamada | 1 | +2/-1 | 1 |
| Robert Carlsen | 1 | +2/-1 | 1 |
| Oli Evans | 1 | +2/-1 | 1 |
| Dan McQuillan | 1 | +2/-1 | 1 |
| susarlanikhilesh | 1 | +1/-1 | 1 |
| mateon1 | 1 | +1/-1 | 1 |
| kpcyrd | 1 | +1/-1 | 1 |
| bbenshoof | 1 | +1/-1 | 1 |
| ZenGround0 | 1 | +1/-1 | 1 |
| Will Hawkins | 1 | +1/-1 | 1 |
| Tommi Virtanen | 1 | +1/-1 | 1 |
| Seungbae Yu | 1 | +1/-1 | 1 |
| Riishab Joshi | 1 | +1/-1 | 1 |
| Kubo Mage | 1 | +1/-1 | 1 |
| Ivan | 1 | +1/-1 | 1 |
| Guillaume Renault | 1 | +1/-1 | 1 |
| Anjor Kanekar | 1 | +1/-1 | 1 |
| Andrew Chin | 1 | +1/-1 | 1 |
| Abdul Rauf | 1 | +1/-1 | 1 |
| makeworld | 1 | +1/-0 | 1 |


# go-ipfs changelog v0.2

## 0.2.3 - 2015-03-01

* Alpha Release

## 2015-01-31:

* bootstrap addresses now have .../ipfs/... in format
  config file Bootstrap field changed accordingly. users
  can upgrade cleanly with:

      ipfs bootstrap >boostrap_peers
      ipfs bootstrap rm --all
      <install new ipfs>
      <manually add .../ipfs/... to addrs in bootstrap_peers>
      ipfs bootstrap add <bootstrap_peers


# Kubo changelog v0.20

- [v0.20.0](#v0200)

## v0.20.0

- [Overview](#overview)
- [🔦 Highlights](#-highlights)
  - [Boxo under the covers](#boxo-under-the-covers)
  - [HTTP Gateway](#http-gateway)
    - [Switch to `boxo/gateway` library](#switch-to-boxogateway-library)
    - [Improved testing](#improved-testing)
    - [Trace Context support](#trace-context-support)
    - [Removed legacy features](#removed-legacy-features)
  - [`--empty-repo` is now the default](#--empty-repo-is-now-the-default)
  - [Reminder: `ipfs pubsub` commands and matching HTTP endpoints are deprecated and will be removed](#reminder-ipfs-pubsub-commands-and-matching-http-endpoints-are-deprecated-and-will-be-removed)
- [📝 Changelog](#-changelog)
- [👨‍👩‍👧‍👦 Contributors](#-contributors)

### Overview

### 🔦 Highlights

#### Boxo under the covers
We have consolidated many IPFS repos into [Boxo](https://github.com/ipfs/boxo), and this release switches Kubo over to use Boxo instead of those repos, resulting in the removal of 27 dependencies from Kubo:

- github.com/ipfs/go-bitswap
- github.com/ipfs/go-ipfs-files
- github.com/ipfs/tar-utils
- gihtub.com/ipfs/go-block-format
- github.com/ipfs/interface-go-ipfs-core
- github.com/ipfs/go-unixfs
- github.com/ipfs/go-pinning-service-http-client
- github.com/ipfs/go-path
- github.com/ipfs/go-namesys
- github.com/ipfs/go-mfs
- github.com/ipfs/go-ipfs-provider
- github.com/ipfs/go-ipfs-pinner
- github.com/ipfs/go-ipfs-keystore
- github.com/ipfs/go-filestore
- github.com/ipfs/go-ipns
- github.com/ipfs/go-blockservice
- github.com/ipfs/go-ipfs-chunker
- github.com/ipfs/go-fetcher
- github.com/ipfs/go-ipfs-blockstore
- github.com/ipfs/go-ipfs-posinfo
- github.com/ipfs/go-ipfs-util
- github.com/ipfs/go-ipfs-ds-help
- github.com/ipfs/go-verifcid
- github.com/ipfs/go-ipfs-exchange-offline
- github.com/ipfs/go-ipfs-routing
- github.com/ipfs/go-ipfs-exchange-interface
- github.com/ipfs/go-libipfs

Note: if you consume these in your own code, we recommend migrating to Boxo. To ease this process, there's a [tool which will help migrate your code to Boxo](https://github.com/ipfs/boxo#migrating-to-box).

You can learn more about the [Boxo 0.8 release](https://github.com/ipfs/boxo/releases/tag/v0.8.0) that Kubo now depends and the general effort to get Boxo to be a stable foundation [here](https://github.com/ipfs/boxo/issues/196).

#### HTTP Gateway

##### Switch to `boxo/gateway` library

Gateway code was extracted and refactored into a standalone library that now
lives in [boxo/gateway](https://github.com/ipfs/boxo/tree/main/gateway). This
enabled us to clean up some legacy code and remove dependency on Kubo
internals.

The GO API is still being refined, but now operates on higher level abstraction
defined by `gateway.IPFSBackend` interface.  It is now possible to embed
gateway functionality without the rest of Kubo.

See the [car](https://github.com/ipfs/boxo/tree/main/examples/gateway/car)
and [proxy](https://github.com/ipfs/boxo/tree/main/examples/gateway/proxy)
examples, or more advanced
[bifrost-gateway](https://github.com/ipfs/bifrost-gateway).

##### Improved testing

We are also in the progress of moving away from gateway testing being based on
Kubo sharness tests, and are working on
[ipfs/gateway-conformance](https://github.com/ipfs/gateway-conformance) test
suite that is vendor agnostic and can be run against arbitrary HTTP endpoint to
test specific subset of [HTTP Gateways specifications](https://specs.ipfs.tech/http-gateways/).

##### Trace Context support

We've introduced initial support for `traceparent` header from [W3C's Trace
Context spec](https://w3c.github.io/trace-context/).

If `traceparent` header is
present in the gateway request, one can use its `trace-id` part to inspect
trace spans via selected exporter such as Jaeger UI
([docs](https://github.com/ipfs/boxo/blob/main/docs/tracing.md#using-jaeger-ui),
[demo](https://user-images.githubusercontent.com/157609/231312374-bafc2035-1fc6-4d6b-901b-9e4af039807c.png)).

To learn more, see [tracing docs](https://github.com/ipfs/boxo/blob/main/docs/tracing.md).

##### Removed legacy features

- Some Kubo-specific prometheus metrics are no longer available.
  - An up-to-date list of gateway metrics can be found in [boxo/gateway/metrics.go](https://github.com/ipfs/boxo/blob/main/gateway/metrics.go).
- The legacy opt-in `Gateway.Writable` is no longer available as of Kubo 0.20.
  - We are working on developing a modern replacement.
    To support our efforts, please leave a comment describing your use case in
    [ipfs/specs#375](https://github.com/ipfs/specs/issues/375).

#### `--empty-repo` is now the default

When creating a repository with `ipfs init`, `--empty-repo=true` is now the default. This means
that your repository will be empty by default instead of containing the introduction files.
You can read more about the rationale behind this decision on the [tracking issue](https://github.com/ipfs/kubo/issues/9757).

#### Reminder: `ipfs pubsub` commands and matching HTTP endpoints are deprecated and will be removed

`ipfs pubsub` commands and all `/api/v0/pubsub/` RPC endpoints and will be removed in the next release. For more information and rational see [#9717](https://github.com/ipfs/kubo/issues/9717).

### 📝 Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - fix: deadlock on retrieving WebTransport addresses (#9857) ([ipfs/kubo#9857](https://github.com/ipfs/kubo/pull/9857))
  - docs(config): remove mentions of relay v1 (#9860) ([ipfs/kubo#9860](https://github.com/ipfs/kubo/pull/9860))
  - Merge branch 'master' into merge-release-v0.19.2
  - docs: add changelog for v0.19.2
  - feat: webui@3.0.0 (#9835) ([ipfs/kubo#9835](https://github.com/ipfs/kubo/pull/9835))
  - fix: use default HTTP routers when FullRT DHT client is used (#9841) ([ipfs/kubo#9841](https://github.com/ipfs/kubo/pull/9841))
  - chore: update version
  - docs: add `ipfs pubsub` deprecation reminder to changelog (#9827) ([ipfs/kubo#9827](https://github.com/ipfs/kubo/pull/9827))
  - docs: preparing 0.20 changelog for release (#9799) ([ipfs/kubo#9799](https://github.com/ipfs/kubo/pull/9799))
  - feat: boxo tracing and traceparent support (#9811) ([ipfs/kubo#9811](https://github.com/ipfs/kubo/pull/9811))
  - chore: update version
  - chore: update version
  - update go-libp2p to v0.27.0
  - docs: add optimistic provide feature description
  - feat: add experimental optimistic provide
  - fix(ci): speed up docker build (#9800) ([ipfs/kubo#9800](https://github.com/ipfs/kubo/pull/9800))
  - feat(tracing): use OTEL_PROPAGATORS as per OTel spec (#9801) ([ipfs/kubo#9801](https://github.com/ipfs/kubo/pull/9801))
  - docs: fix jaeger command (#9797) ([ipfs/kubo#9797](https://github.com/ipfs/kubo/pull/9797))
  - Merge Release: v0.19.1 (#9794) ([ipfs/kubo#9794](https://github.com/ipfs/kubo/pull/9794))
  - chore: upgrade OpenTelemetry dependencies (#9736) ([ipfs/kubo#9736](https://github.com/ipfs/kubo/pull/9736))
  - test: fix flaky content routing over HTTP test (#9772) ([ipfs/kubo#9772](https://github.com/ipfs/kubo/pull/9772))
  - feat: allow injecting custom path resolvers (#9750) ([ipfs/kubo#9750](https://github.com/ipfs/kubo/pull/9750))
  - feat: add changelog entry for router timeouts for v0.19.1 (#9784) ([ipfs/kubo#9784](https://github.com/ipfs/kubo/pull/9784))
  - feat(gw): new metrics and HTTP range support (#9786) ([ipfs/kubo#9786](https://github.com/ipfs/kubo/pull/9786))
  - feat!: make --empty-repo default (#9758) ([ipfs/kubo#9758](https://github.com/ipfs/kubo/pull/9758))
  - fix: remove timeout on default DHT operations (#9783) ([ipfs/kubo#9783](https://github.com/ipfs/kubo/pull/9783))
  - refactor: switch gateway code to new API from go-libipfs (#9681) ([ipfs/kubo#9681](https://github.com/ipfs/kubo/pull/9681))
  - test: port remote pinning tests to Go (#9720) ([ipfs/kubo#9720](https://github.com/ipfs/kubo/pull/9720))
  - feat: add identify option to swarm peers command
  - test: port routing DHT tests to Go (#9709) ([ipfs/kubo#9709](https://github.com/ipfs/kubo/pull/9709))
  - test: fix autoclient flakiness (#9769) ([ipfs/kubo#9769](https://github.com/ipfs/kubo/pull/9769))
  - test: skip flaky pubsub test (#9770) ([ipfs/kubo#9770](https://github.com/ipfs/kubo/pull/9770))
  - chore: migrate go-libipfs to boxo
  - feat: add tracing to the commands client
  - feat: add client-side metrics for routing-v1 client
  - test: increase max wait time for peering assertion
  - feat: remove writable gateway (#9743) ([ipfs/kubo#9743](https://github.com/ipfs/kubo/pull/9743))
  - Process Improvement: v0.18.0 ([ipfs/kubo#9484](https://github.com/ipfs/kubo/pull/9484))
  - fix: deadlock while racing `ipfs dag import` and `ipfs repo gc`
  - feat: improve dag/import (#9721) ([ipfs/kubo#9721](https://github.com/ipfs/kubo/pull/9721))
  - ci: remove circleci config ([ipfs/kubo#9687](https://github.com/ipfs/kubo/pull/9687))
  - docs: use fx.Decorate instead of fx.Replace in examples (#9725) ([ipfs/kubo#9725](https://github.com/ipfs/kubo/pull/9725))
  - Create Changelog: v0.20 ([ipfs/kubo#9742](https://github.com/ipfs/kubo/pull/9742))
  - Merge Release: v0.19.0 ([ipfs/kubo#9741](https://github.com/ipfs/kubo/pull/9741))
  - feat(gateway): invalid CID returns 400 Bad Request (#9726) ([ipfs/kubo#9726](https://github.com/ipfs/kubo/pull/9726))
  - fix: remove outdated changelog part ([ipfs/kubo#9739](https://github.com/ipfs/kubo/pull/9739))
  - docs: 0.19 changelog ([ipfs/kubo#9707](https://github.com/ipfs/kubo/pull/9707))
  - fix: canonicalize user defined headers
  - fix: apply API.HTTPHeaders to /webui redirect
  - feat: add heap allocs to 'ipfs diag profile'
  - fix: future proof with > rcmgr.DefaultLimit for new enum rcmgr values
  - test: add test for presarvation of unlimited configs for inbound systems
  - fix: preserve Unlimited StreamsInbound in connmgr reconciliation
  - test: fix flaky rcmgr test
  - chore: deprecate the pubsub api
  - Revert "chore: add hamt directory sharding test"
  - chore: add hamt directory sharding test
  - test: port peering test from sharness to Go
  - test: use `T.TempDir` to create temporary test directory
  - fix: --verify forgets the verified key
  - test: name --verify forgets the verified key
  - chore: fix toc in changelog for 0.18
  - feat: add "autoclient" routing type
  - test: parallelize more of rcmgr Go tests
  - test: port legacy DHT tests to Go
  - fix: t0116-gateway-cache.sh ([ipfs/kubo#9696](https://github.com/ipfs/kubo/pull/9696))
  - docs: add bifrost to early testers ([ipfs/kubo#9699](https://github.com/ipfs/kubo/pull/9699))
  - fix: typo in documentation for install path
  - docs: fix typos
  - Update Version: v0.19 ([ipfs/kubo#9698](https://github.com/ipfs/kubo/pull/9698))
- github.com/ipfs/go-block-format (v0.1.1 -> v0.1.2):
  - chore: release v0.1.2
  - Revert deprecation and go-libipfs/blocks stub types
  - docs: deprecation notice [ci skip]
- github.com/ipfs/go-cid (v0.3.2 -> v0.4.1):
  - v0.4.1
  - Add unit test for unexpected eof
  - Update cid.go
  - CidFromReader should not wrap valid EOF return.
  - chore: version 0.4.0
  - feat: wrap parsing errors into ErrInvalidCid
  - fix: use crypto/rand.Read
  - Fix README.md example error (#146) ([ipfs/go-cid#146](https://github.com/ipfs/go-cid/pull/146))
- github.com/ipfs/go-delegated-routing (v0.7.0 -> v0.8.0):
  - chore: release v0.8.0
  - chore: migrate from go-ipns to boxo
  - docs: add deprecation notice [ci skip]
- github.com/ipfs/go-graphsync (v0.14.1 -> v0.14.4):
  - Update version to cover latest fixes (#419) ([ipfs/go-graphsync#419](https://github.com/ipfs/go-graphsync/pull/419))
  - Bring changes from #412
  - Bring changes from #391
  - fix: calling message queue Shutdown twice causes panic (because close is called twice on done channel) (#414) ([ipfs/go-graphsync#414](https://github.com/ipfs/go-graphsync/pull/414))
  - docs(CHANGELOG): update for v0.14.3
  - fix: wire up proper linksystem to traverser (#411) ([ipfs/go-graphsync#411](https://github.com/ipfs/go-graphsync/pull/411))
  - sync: update CI config files (#378) ([ipfs/go-graphsync#378](https://github.com/ipfs/go-graphsync/pull/378))
  - chore: remove social links (#398) ([ipfs/go-graphsync#398](https://github.com/ipfs/go-graphsync/pull/398))
  - Removes `main` branch callout.
  - release v0.14.2
- github.com/ipfs/go-ipfs-blockstore (v1.2.0 -> v1.3.0):
  - chore: release v1.3.0
  - feat: stub and deprecate NewBlockstoreNoPrefix
  - Accept options for blockstore: start with WriteThrough and NoPrefix
  - Allow using a NewWriteThrough() blockstore.
  - sync: update CI config files (#105) ([ipfs/go-ipfs-blockstore#105](https://github.com/ipfs/go-ipfs-blockstore/pull/105))
  - feat: fast-path for PutMany, falling back to Put for single block call (#97) ([ipfs/go-ipfs-blockstore#97](https://github.com/ipfs/go-ipfs-blockstore/pull/97))
- github.com/ipfs/go-ipfs-cmds (v0.8.2 -> v0.9.0):
  - chore: release v0.9.0
  - chore: change go-libipfs to boxo
- github.com/ipfs/go-libipfs (v0.6.2 -> v0.7.0):
  - chore: bump to 0.7.0 (#213) ([ipfs/go-libipfs#213](https://github.com/ipfs/go-libipfs/pull/213))
  - feat: return 400 on /ipfs/invalid-cid (#205) ([ipfs/go-libipfs#205](https://github.com/ipfs/go-libipfs/pull/205))
  - docs: add note in README that go-libipfs is not comprehensive (#163) ([ipfs/go-libipfs#163](https://github.com/ipfs/go-libipfs/pull/163))
- github.com/ipfs/go-merkledag (v0.9.0 -> v0.10.0):
  - chore: bump version to 0.10.0
  - fix: switch to crypto/rand.Read
  - stop using the deprecated io/ioutil package
- github.com/ipfs/go-unixfs (v0.4.4 -> v0.4.5):
  - chore: release v0.4.5
  - chore: remove go-libipfs dependency
- github.com/ipfs/go-unixfsnode (v1.5.2 -> v1.6.0):
  - chore: bump v1.6.0
  - feat: add UnixFSPathSelectorBuilder ([ipfs/go-unixfsnode#45](https://github.com/ipfs/go-unixfsnode/pull/45))
  - fix: update state to allow iter continuance on NotFound errors
  - chore!: make PBLinkItr private - not intended for public use
  - fix: propagate iteration errors
- github.com/ipld/go-car/v2 (v2.5.1 -> v2.9.1-0.20230325062757-fff0e4397a3d):
  - chore: unmigrate from go-libipfs
  - Create CODEOWNERS
  - blockstore: give a direct access to the index for read operations
  - blockstore: only close the file on error in OpenReadWrite, not OpenReadWriteFile
  - fix: handle (and test) WholeCID vs not; fast Has() path for storage
  - ReadWrite: faster Has() by using the in-memory index instead of reading on disk
  - fix: let `extract` skip missing unixfs shard links
  - fix: error when no files extracted
  - fix: make -f optional, read from stdin if omitted
  - fix: update cmd/car/README with latest description
  - chore: add test cases for extract modes
  - feat: extract accepts '-' as an output path for stdout
  - feat: extract specific path, accept stdin as streaming input
  - fix: if we don't read the full block data, don't error on !EOF
  - blockstore: try to close during Finalize(), even in case of previous error
  - ReadWrite: add an alternative FinalizeReadOnly+Close flow
  - feat: add WithTrustedCar() reader option (#381) ([ipld/go-car#381](https://github.com/ipld/go-car/pull/381))
  - blockstore: fast path for AllKeysChan using the index
  - fix: switch to crypto/rand.Read
  - stop using the deprecated io/ioutil package
  - fix(doc): fix storage package doc formatting
  - fix: return errors for unsupported operations
  - chore: move insertionindex into store pkg
  - chore: add experimental note
  - fix: minor lint & windows fd test problems
  - feat: docs for StorageCar interfaces
  - feat: ReadableWritable; dedupe shared code
  - feat: add Writable functionality to StorageCar
  - feat: StorageCar as a Readable storage, separate from blockstore
  - feat(blockstore): implement a streaming read only storage
  - feat(cmd): add index create subcommand to create an external carv2 index ([ipld/go-car#350](https://github.com/ipld/go-car/pull/350))
  - chore: bump version to 0.6.0
  - fix: use goreleaser instead
  - Allow using WalkOption in WriteCar function ([ipld/go-car#357](https://github.com/ipld/go-car/pull/357))
  - fix: update go-block-format to the version that includes the stubs
  - feat: upgrade from go-block-format to go-libipfs/blocks
  - cleanup readme a bit to make the cli more discoverable (#353) ([ipld/go-car#353](https://github.com/ipld/go-car/pull/353))
  - Update install instructions in README.md
  - Add a debugging form for car files. (#341) ([ipld/go-car#341](https://github.com/ipld/go-car/pull/341))
  -  ([ipld/go-car#340](https://github.com/ipld/go-car/pull/340))
- github.com/ipld/go-codec-dagpb (v1.5.0 -> v1.6.0):
  - Update version.json
- github.com/ipld/go-ipld-prime (v0.19.0 -> v0.20.0):
  - Prepare v0.20.0
  - fix(datamodel): add tests to Copy, make it complain on nil
  - feat(dagcbor): mode to allow parsing undelimited streamed objects
  - Fix mispatched package declaration.
  - Add several pieces of docs to schema/dmt.
  - Additional access to schema/dmt package; schema concatenation feature ([ipld/go-ipld-prime#483](https://github.com/ipld/go-ipld-prime/pull/483))
  - Fix hash mismatch error on matching link pointer
  - feat: support errors.Is for schema errors
- github.com/ipld/go-ipld-prime/storage/bsadapter (v0.0.0-20211210234204-ce2a1c70cd73 -> v0.0.0-20230102063945-1a409dc236dd):
  - build(deps): bump github.com/ipfs/go-blockservice
  - Fix mispatched package declaration.
  - Add several pieces of docs to schema/dmt.
  - Additional access to schema/dmt package; schema concatenation feature ([ipld/go-ipld-prime/storage/bsadapter#483](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/483))
  - fix: go mod tidy
  - build(deps): bump github.com/frankban/quicktest from 1.14.3 to 1.14.4
  - Fix hash mismatch error on matching link pointer
  - build(deps): bump github.com/warpfork/go-testmark from 0.10.0 to 0.11.0
  - feat: support errors.Is for schema errors
  - build(deps): bump github.com/multiformats/go-multicodec
  - Prepare v0.19.0
  - fix: correct json codec links & bytes handling
  - build(deps): bump github.com/google/go-cmp from 0.5.8 to 0.5.9 (#468) ([ipld/go-ipld-prime/storage/bsadapter#468](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/468))
  - build(deps): bump github.com/ipfs/go-cid from 0.3.0 to 0.3.2 (#466) ([ipld/go-ipld-prime/storage/bsadapter#466](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/466))
  - build(deps): bump github.com/ipfs/go-cid in /storage/bsrvadapter (#464) ([ipld/go-ipld-prime/storage/bsadapter#464](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/464))
  - test(basicnode): increase test coverage for int and map types (#454) ([ipld/go-ipld-prime/storage/bsadapter#454](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/454))
  - build(deps): bump github.com/ipfs/go-cid in /storage/bsrvadapter
  - build(deps): bump github.com/ipfs/go-cid from 0.2.0 to 0.3.0
  - build(deps): bump github.com/multiformats/go-multicodec
  - fix: remove reliance on ioutil
  - fix: update sub-package modules
  - build(deps): bump github.com/multiformats/go-multihash
  - build(deps): bump github.com/ipfs/go-datastore in /storage/dsadapter
  - update .github/workflows/go-check.yml
  - update .github/workflows/go-test.yml
  - run gofmt -s
  - bump go.mod to Go 1.18 and run go fix
  - bump go.mod to Go 1.18 and run go fix
  - bump go.mod to Go 1.18 and run go fix
  - bump go.mod to Go 1.18 and run go fix
  - feat: add kinded union to gendemo
  - fix: go mod 1.17 compat problems
  - build(deps): bump github.com/ipfs/go-blockservice
  - Prepare v0.18.0
  - fix(deps): update benchmarks go.sum
  - build(deps): bump github.com/multiformats/go-multihash
  - feat(bindnode): add a BindnodeRegistry utility (#437) ([ipld/go-ipld-prime/storage/bsadapter#437](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/437))
  - feat(bindnode): support full uint64 range
  - chore(bindnode): remove typed functions for options
  - chore(bindnode): docs and minor tweaks
  - feat(bindnode): make Any converters work for List and Map values
  - fix(bindnode): shorten converter option names, minor perf improvements
  - fix(bindnode): only custom convert AssignNull for Any converter
  - feat(bindnode): pass Null on to nullable custom converters
  - chore(bindnode): config helper refactor w/ short-circuit
  - feat(bindnode): add AddCustomTypeAnyConverter() to handle `Any` fields
  - feat(bindnode): add AddCustomTypeXConverter() options for most scalar kinds
  - chore(bindnode): back out of reflection for converters
  - feat(bindnode): switch to converter functions instead of type
  - feat(bindnode): allow custom type conversions with options
  - feat: add release checklist (#442) ([ipld/go-ipld-prime/storage/bsadapter#442](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/442))
  - Prepare v0.17.0
  - feat: introduce UIntNode interface, used within DAG-CBOR codec
  - add option to not parse beyond end of structure (#435) ([ipld/go-ipld-prime/storage/bsadapter#435](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/435))
  - sync benchmarks go.sum
  - build(deps): bump github.com/multiformats/go-multicodec
  - patch: first draft. ([ipld/go-ipld-prime/storage/bsadapter#350](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/350))
  - feat(bindnode): infer links and Any from Go types (#432) ([ipld/go-ipld-prime/storage/bsadapter#432](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/432))
  - fix(codecs): error on cid.Undef links in dag{json,cbor} encoding (#433) ([ipld/go-ipld-prime/storage/bsadapter#433](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/433))
  - chore(bindnode): add test for sub-node unwrapping
  - fix(bindnode): more helpful error message for enum value footgun
  - fix(bindnode): panic early if API has been passed ptr-to-ptr
  - fix(deps): mod tidy for dependencies
  - build(deps): bump github.com/warpfork/go-testmark from 0.3.0 to 0.10.0
  - build(deps): bump github.com/multiformats/go-multicodec
  - build(deps): bump github.com/ipfs/go-cid from 0.0.4 to 0.2.0
  - build(deps): bump github.com/google/go-cmp from 0.5.7 to 0.5.8
  - build(deps): bump github.com/frankban/quicktest from 1.14.2 to 1.14.3
  - build(deps): bump github.com/ipfs/go-cid in /storage/bsrvadapter
  - chore(deps): expand dependabot to sub-modules
  - chore(deps): add dependabot config
  - printer: fix printing of floats
  - add version.json file (#411) ([ipld/go-ipld-prime/storage/bsadapter#411](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/411))
  - ci: use GOFLAGS to control test tags
  - ci: disable coverpkg using custom workflow insertion
  - ci: add initial web3 unified-ci files
  - fix: make 32-bit safe and stable & add to CI
  - ci: add go-check.yml workflow from unified-ci
  - ci: go mod tidy
  - fix: staticcheck and govet fixes
  - test: make tests work on Windows, add Windows to CI (#405) ([ipld/go-ipld-prime/storage/bsadapter#405](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/405))
  - schema: enable inline types through dsl parser & compiler (#404) ([ipld/go-ipld-prime/storage/bsadapter#404](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/404))
  - node/bindnode: allow nilable types for IPLD optional/nullable
  - test(ci): enable macos in GitHub Actions
  - test(gen-go): disable parallelism when testing on macos
  - storage: update deps
  - dsl support for stringjoin struct repr and stringprefix union repr ([ipld/go-ipld-prime/storage/bsadapter#397](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/397))
  - codec/dagcbor: add DecodeOptions.ExperimentalDeterminism
  - node/bindnode: add some more docs
  - start testing on Go 1.18.x, drop Go 1.16.x
  - readme: getting started pointers.
  - readme: bindnode definitely needs a mention!
  - Readme updates!
  - datamodel: document that repr prototypes produce type nodes
  - node/bindnode: minor fuzz improvements
  - gengo: update readme.
  - fix(dagcbor): don't accept trailing bytes
  - schema/dmt: reject duplicate or missing union repr members
  - node/bindnode: actually check schemadmt.Compile errors when fuzzing
  - node/bindnode: avoid OOM when inferring from cyclic IPLD schemas
  - schema/dmt: require enum reprs to refer valid members
  - skip NaN/Inf errors for dag-json
  - node/bindnode: refuse to decode empty union values
  - schema/dmt: error in Compile if union reprs refer to unknown members
  - node/bindnode: start fuzzing with schema/dmt and codec/dagcbor
  - mark v0.16.0
  - node/bindnode: enforce pointer requirement for nullable maps
  - Implement WalkTransforming traversal (#376) ([ipld/go-ipld-prime/storage/bsadapter#376](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/376))
  - docs(datamodel): add comment to LargeBytesNode
  - Add partial-match traversal of large bytes (#375) ([ipld/go-ipld-prime/storage/bsadapter#375](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/375))
  - Implement option to start traversals at a path ([ipld/go-ipld-prime/storage/bsadapter#358](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/358))
  - add top-level "go value with schema" example
  - Support optional `LargeBytesNode` interface (#372) ([ipld/go-ipld-prime/storage/bsadapter#372](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/372))
  - node/bindnode: support pointers to datamodel.Node to bind with Any
  - fix(bindnode): tuple struct iterator should handle absent fields properly
  - node/bindnode: make AssignNode work at the repr level
  - node/bindnode: add support for unsigned integers
  - node/bindnode: cover even more edge case panics
  - node/bindnode: polish some more AsT panics
  - schema/dmt: stop using a fake test to generate code ([ipld/go-ipld-prime/storage/bsadapter#356](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/356))
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
  - selectors: fix for edge case around recursion clauses with an immediate edge. ([ipld/go-ipld-prime/storage/bsadapter#334](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/334))
  - node/bindnode: improve support for pointer types
  - node/bindnode: subtract all absents in Length at the repr level
  - fix(codecs): error when encoding maps whose lengths don't match entry count
  - schema: avoid alloc and copy in Struct and Enum methods
  - node/bindnode: allow mapping int-repr enums with Go integers
  - schema,node/bindnode: add support for Any
  - signaling ADLs in selectors (#301) ([ipld/go-ipld-prime/storage/bsadapter#301](https://github.com/ipld/go-ipld-prime/storage/bsadapter/pull/301))
  - node/bindnode: add support for enums
  - schema/...: add support for enum int representations
  - node/bindnode: allow binding cidlink.Link to links
- github.com/libp2p/go-libp2p (v0.26.4 -> v0.27.3):
  - release v0.27.3
  - quic virtual listener: don't panic when quic-go's accept call errors (#2276) ([libp2p/go-libp2p#2276](https://github.com/libp2p/go-libp2p/pull/2276))
  - Release v0.27.2 (#2270) ([libp2p/go-libp2p#2270](https://github.com/libp2p/go-libp2p/pull/2270))
  - release v0.27.1 (#2252) ([libp2p/go-libp2p#2252](https://github.com/libp2p/go-libp2p/pull/2252))
  - Infer public webtransport addrs from quic-v1 addrs. (#2251) ([libp2p/go-libp2p#2251](https://github.com/libp2p/go-libp2p/pull/2251))
  - basichost: don't allocate when deduplicating multiaddrs (#2206) ([libp2p/go-libp2p#2206](https://github.com/libp2p/go-libp2p/pull/2206))
  - identify: fix normalization of interface listen addresses (#2250) ([libp2p/go-libp2p#2250](https://github.com/libp2p/go-libp2p/pull/2250))
  - autonat: fix flaky TestAutoNATDialRefused (#2245) ([libp2p/go-libp2p#2245](https://github.com/libp2p/go-libp2p/pull/2245))
  - basichost: remove stray print statement in test (#2249) ([libp2p/go-libp2p#2249](https://github.com/libp2p/go-libp2p/pull/2249))
  - swarm: fix multiaddr comparison in ListenClose (#2247) ([libp2p/go-libp2p#2247](https://github.com/libp2p/go-libp2p/pull/2247))
  - release v0.27.0 (#2242) ([libp2p/go-libp2p#2242](https://github.com/libp2p/go-libp2p/pull/2242))
  - add a security policy (#2238) ([libp2p/go-libp2p#2238](https://github.com/libp2p/go-libp2p/pull/2238))
  - chore: 0.27.0 changelog entries (#2241) ([libp2p/go-libp2p#2241](https://github.com/libp2p/go-libp2p/pull/2241))
  - correctly handle WebTransport addresses without certhashes (#2239) ([libp2p/go-libp2p#2239](https://github.com/libp2p/go-libp2p/pull/2239))
  - autorelay: add metrics (#2185) ([libp2p/go-libp2p#2185](https://github.com/libp2p/go-libp2p/pull/2185))
  - autonat: don't change status on dial request refused (#2225) ([libp2p/go-libp2p#2225](https://github.com/libp2p/go-libp2p/pull/2225))
  - autonat: fix closing of listeners in dialPolicy tests (#2226) ([libp2p/go-libp2p#2226](https://github.com/libp2p/go-libp2p/pull/2226))
  - discovery (backoff): fix typo in comment (#2214) ([libp2p/go-libp2p#2214](https://github.com/libp2p/go-libp2p/pull/2214))
  - relaysvc: flaky TestReachabilityChangeEvent (#2215) ([libp2p/go-libp2p#2215](https://github.com/libp2p/go-libp2p/pull/2215))
  - Add wss transport to interop tester impl (#2178) ([libp2p/go-libp2p#2178](https://github.com/libp2p/go-libp2p/pull/2178))
  - tests: add a stream read deadline transport test (#2210) ([libp2p/go-libp2p#2210](https://github.com/libp2p/go-libp2p/pull/2210))
  - autorelay: fix busy loop bug and flaky tests in relay finder (#2208) ([libp2p/go-libp2p#2208](https://github.com/libp2p/go-libp2p/pull/2208))
  - tests: test mplex and Yamux, Noise and TLS in transport tests (#2209) ([libp2p/go-libp2p#2209](https://github.com/libp2p/go-libp2p/pull/2209))
  - tests: add some basic transport integration tests (#2207) ([libp2p/go-libp2p#2207](https://github.com/libp2p/go-libp2p/pull/2207))
  - autorelay: remove unused semaphore (#2184) ([libp2p/go-libp2p#2184](https://github.com/libp2p/go-libp2p/pull/2184))
  - basichost: prevent duplicate dials (#2196) ([libp2p/go-libp2p#2196](https://github.com/libp2p/go-libp2p/pull/2196))
  - websocket: don't set a WSS multiaddr for accepted unencrypted conns (#2199) ([libp2p/go-libp2p#2199](https://github.com/libp2p/go-libp2p/pull/2199))
  - websocket: Don't limit message sizes in the websocket reader (#2193) ([libp2p/go-libp2p#2193](https://github.com/libp2p/go-libp2p/pull/2193))
  - identify: fix stale comment (#2179) ([libp2p/go-libp2p#2179](https://github.com/libp2p/go-libp2p/pull/2179))
  - relay service: add metrics (#2154) ([libp2p/go-libp2p#2154](https://github.com/libp2p/go-libp2p/pull/2154))
  - identify: Fix IdentifyWait when Connected events happen out of order (#2173) ([libp2p/go-libp2p#2173](https://github.com/libp2p/go-libp2p/pull/2173))
  - chore: fix ressource manager's README (#2168) ([libp2p/go-libp2p#2168](https://github.com/libp2p/go-libp2p/pull/2168))
  - relay: fix deadlock when closing (#2171) ([libp2p/go-libp2p#2171](https://github.com/libp2p/go-libp2p/pull/2171))
  - core: remove LocalPrivateKey method from network.Conn interface (#2144) ([libp2p/go-libp2p#2144](https://github.com/libp2p/go-libp2p/pull/2144))
  - routed host: return connection error instead of routing error (#2169) ([libp2p/go-libp2p#2169](https://github.com/libp2p/go-libp2p/pull/2169))
  - connmgr: reduce log level for closing connections (#2165) ([libp2p/go-libp2p#2165](https://github.com/libp2p/go-libp2p/pull/2165))
  - circuitv2: cleanup relay service properly (#2164) ([libp2p/go-libp2p#2164](https://github.com/libp2p/go-libp2p/pull/2164))
  - chore: add patch release to changelog (#2151) ([libp2p/go-libp2p#2151](https://github.com/libp2p/go-libp2p/pull/2151))
  - chore: remove superfluous testing section from README (#2150) ([libp2p/go-libp2p#2150](https://github.com/libp2p/go-libp2p/pull/2150))
  - autonat: don't use autonat for address discovery (#2148) ([libp2p/go-libp2p#2148](https://github.com/libp2p/go-libp2p/pull/2148))
  - swarm metrics: fix connection direction (#2147) ([libp2p/go-libp2p#2147](https://github.com/libp2p/go-libp2p/pull/2147))
  - connmgr: Use eventually equal helper in connmgr tests (#2128) ([libp2p/go-libp2p#2128](https://github.com/libp2p/go-libp2p/pull/2128))
  - swarm: emit PeerConnectedness event from swarm instead of from hosts (#1574) ([libp2p/go-libp2p#1574](https://github.com/libp2p/go-libp2p/pull/1574))
  - relay: initialize the ASN util when starting the service (#2143) ([libp2p/go-libp2p#2143](https://github.com/libp2p/go-libp2p/pull/2143))
  - Fix flaky TestMetricsNoAllocNoCover test (#2142) ([libp2p/go-libp2p#2142](https://github.com/libp2p/go-libp2p/pull/2142))
  - identify: Bump timeouts/sleep in tests (#2135) ([libp2p/go-libp2p#2135](https://github.com/libp2p/go-libp2p/pull/2135))
  - Add sleep to fix flaky test (#2129) ([libp2p/go-libp2p#2129](https://github.com/libp2p/go-libp2p/pull/2129))
  - basic_host: Fix flaky tests (#2136) ([libp2p/go-libp2p#2136](https://github.com/libp2p/go-libp2p/pull/2136))
  - swarm: Check context once more before dialing (#2139) ([libp2p/go-libp2p#2139](https://github.com/libp2p/go-libp2p/pull/2139))
- github.com/libp2p/go-libp2p-asn-util (v0.2.0 -> v0.3.0):
  - release v0.3.0 (#26) ([libp2p/go-libp2p-asn-util#26](https://github.com/libp2p/go-libp2p-asn-util/pull/26))
  - initialize the store lazily (#25) ([libp2p/go-libp2p-asn-util#25](https://github.com/libp2p/go-libp2p-asn-util/pull/25))
- github.com/libp2p/go-libp2p-gostream (v0.5.0 -> v0.6.0):
  - Update libp2p ([libp2p/go-libp2p-gostream#80](https://github.com/libp2p/go-libp2p-gostream/pull/80))
  - fix typo in README (#75) ([libp2p/go-libp2p-gostream#75](https://github.com/libp2p/go-libp2p-gostream/pull/75))
- github.com/libp2p/go-libp2p-http (v0.4.0 -> v0.5.0):
  - sync: update CI config files ([libp2p/go-libp2p-http#82](https://github.com/libp2p/go-libp2p-http/pull/82))
- github.com/libp2p/go-libp2p-kad-dht (v0.21.1 -> v0.23.0):
  - Release v0.23.0
  - Specified CODEOWNERS ([libp2p/go-libp2p-kad-dht#828](https://github.com/libp2p/go-libp2p-kad-dht/pull/828))
  - fix: optimistic provide ci checks in tests ([libp2p/go-libp2p-kad-dht#833](https://github.com/libp2p/go-libp2p-kad-dht/pull/833))
  - feat: add experimental optimistic provide (#783) ([libp2p/go-libp2p-kad-dht#783](https://github.com/libp2p/go-libp2p-kad-dht/pull/783))
  - feat: rework tracing a bit
  - feat: add basic tracing
  - chore: release v0.22.0
  - chore: migrate go-libipfs to boxo
  - Fix multiple ProviderAddrTTL definitions #795 ([libp2p/go-libp2p-kad-dht#831](https://github.com/libp2p/go-libp2p-kad-dht/pull/831))
  - Increase provider Multiaddress TTL ([libp2p/go-libp2p-kad-dht#795](https://github.com/libp2p/go-libp2p-kad-dht/pull/795))
  - Make provider manager options configurable in `fullrt` ([libp2p/go-libp2p-kad-dht#829](https://github.com/libp2p/go-libp2p-kad-dht/pull/829))
  - Adjust PeerSet logic in the DHT lookup process ([libp2p/go-libp2p-kad-dht#802](https://github.com/libp2p/go-libp2p-kad-dht/pull/802))
  - added maintainers in the README ([libp2p/go-libp2p-kad-dht#826](https://github.com/libp2p/go-libp2p-kad-dht/pull/826))
  - Allow DHT crawler to be swappable
  - Introduce options to parameterize config of the accelerated DHT client ([libp2p/go-libp2p-kad-dht#822](https://github.com/libp2p/go-libp2p-kad-dht/pull/822))
- github.com/libp2p/go-libp2p-pubsub (v0.9.0 -> v0.9.3):
  - Fix Memory Leak In New Timecache Implementations (#528) ([libp2p/go-libp2p-pubsub#528](https://github.com/libp2p/go-libp2p-pubsub/pull/528))
  - Default validator support (#525) ([libp2p/go-libp2p-pubsub#525](https://github.com/libp2p/go-libp2p-pubsub/pull/525))
  - Refactor timecache implementations (#523) ([libp2p/go-libp2p-pubsub#523](https://github.com/libp2p/go-libp2p-pubsub/pull/523))
  - fix(timecache): remove panic in first seen cache on Add (#522) ([libp2p/go-libp2p-pubsub#522](https://github.com/libp2p/go-libp2p-pubsub/pull/522))
  - chore: update go version and dependencies (#516) ([libp2p/go-libp2p-pubsub#516](https://github.com/libp2p/go-libp2p-pubsub/pull/516))
- github.com/multiformats/go-multiaddr (v0.8.0 -> v0.9.0):
  - Release v0.9.0 ([multiformats/go-multiaddr#196](https://github.com/multiformats/go-multiaddr/pull/196))
  - Update webrtc protocols after rename ([multiformats/go-multiaddr#195](https://github.com/multiformats/go-multiaddr/pull/195))
- github.com/multiformats/go-multibase (v0.1.1 -> v0.2.0):
  - chore: bump v0.2.0
  - fix: math/rand -> crypto/rand
  - fuzz: add Decoder fuzzing
- github.com/multiformats/go-multicodec (v0.7.0 -> v0.8.1):
  - Bump version to release `ipns-record` code
  - chore: update submodules and go generate
  - deps: upgrade stringer to compatible version
  - v0.8.0
  - chore: update submodules and go generate
- github.com/warpfork/go-testmark (v0.10.0 -> v0.11.0):
  - Quick changelog to note we have an API update.
  - Index fix ([warpfork/go-testmark#13](https://github.com/warpfork/go-testmark/pull/13))
  - Link to python implementation in the readme!

</details>

### 👨‍👩‍👧‍👦 Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Rod Vagg | 40 | +4214/-1400 | 102 |
| Sukun | 12 | +3541/-267 | 34 |
| Gus Eggert | 22 | +2387/-1160 | 81 |
| galargh | 23 | +1331/-1734 | 34 |
| Henrique Dias | 23 | +681/-1167 | 79 |
| Marco Munizaga | 19 | +1500/-187 | 55 |
| Jorropo | 25 | +897/-597 | 180 |
| Dennis Trautwein | 4 | +990/-60 | 14 |
| Marten Seemann | 18 | +443/-450 | 53 |
| vyzo | 2 | +595/-152 | 11 |
| Michael Muré | 8 | +427/-182 | 18 |
| Will | 2 | +536/-15 | 5 |
| Adin Schmahmann | 3 | +327/-125 | 11 |
| hannahhoward | 2 | +344/-1 | 4 |
| Arthur Gavazza | 1 | +210/-50 | 4 |
| Hector Sanjuan | 6 | +181/-77 | 13 |
| Masih H. Derkani | 5 | +214/-42 | 12 |
| Calvin Behling | 4 | +158/-58 | 11 |
| Eric Myhre | 7 | +113/-27 | 15 |
| Marcin Rataj | 5 | +72/-30 | 5 |
| Steve Loeppky | 2 | +99/-0 | 2 |
| Piotr Galar | 9 | +60/-18 | 9 |
| gammazero | 4 | +69/-0 | 8 |
| Prithvi Shahi | 2 | +55/-14 | 2 |
| Eng Zer Jun | 1 | +15/-54 | 5 |
| Laurent Senta | 3 | +44/-2 | 3 |
| Ian Davis | 1 | +35/-0 | 1 |
| web3-bot | 4 | +19/-13 | 7 |
| guillaumemichel | 2 | +18/-14 | 3 |
| Guillaume Michel - guissou | 4 | +24/-8 | 4 |
| omahs | 1 | +9/-9 | 3 |
| cortze | 3 | +9/-9 | 3 |
| Nishant Das | 1 | +9/-5 | 3 |
| Hlib Kanunnikov | 2 | +11/-3 | 3 |
| Andrew Gillis | 3 | +6/-8 | 3 |
| Johnny | 1 | +0/-10 | 1 |
| Rafał Leszko | 1 | +4/-4 | 1 |
| Dirk McCormick | 1 | +4/-1 | 1 |
| Antonio Navarro Perez | 1 | +4/-1 | 1 |
| RichΛrd | 2 | +2/-2 | 2 |
| Russell Dempsey | 1 | +2/-1 | 1 |
| Winterhuman | 1 | +1/-1 | 1 |
| Will Hawkins | 1 | +1/-1 | 1 |
| Nikhilesh Susarla | 1 | +1/-1 | 1 |
| Kubo Mage | 1 | +1/-1 | 1 |
| Bryan White | 1 | +1/-1 | 1 |




# Kubo changelog v0.21

- [v0.21.1](#v0211)
- [v0.21.0](#v0210)

## v0.21.1

- Update go-libp2p:
  - [v0.27.8](https://github.com/libp2p/go-libp2p/releases/tag/v0.27.8)
  - [v0.27.9](https://github.com/libp2p/go-libp2p/releases/tag/v0.27.9)
- Update Boxo to v0.10.3 ([ipfs/boxo#412](https://github.com/ipfs/boxo/pull/412)).

## v0.21.0

- [Overview](#overview)
- [🔦 Highlights](#-highlights)
  - [Saving previously seen nodes for later bootstrapping](#saving-previously-seen-nodes-for-later-bootstrapping)
  - [Gateway: `DeserializedResponses` config flag](#gateway-deserializedresponses-config-flag)
  - [`client/rpc` migration of `go-ipfs-http-client`](#clientrpc-migration-of-go-ipfs-http-client)
  - [Gateway: DAG-CBOR/-JSON previews and improved error pages](#gateway-dag-cbor-json-previews-and-improved-error-pages)
  - [Gateway: subdomain redirects are now `text/html`](#gateway-subdomain-redirects-are-now-texthtml)
  - [Gateway: support for partial CAR export parameters (IPIP-402)](#gateway-support-for-partial-car-export-parameters-ipip-402)
  - [`ipfs dag stat` deduping statistics](#ipfs-dag-stat-deduping-statistics)
  - [Accelerated DHT Client is no longer experimental](#accelerated-dht-client-is-no-longer-experimental)
- [📝 Changelog](#-changelog)
- [👨‍👩‍👧‍👦 Contributors](#-contributors)

### Overview

### 🔦 Highlights

#### Saving previously seen nodes for later bootstrapping

Kubo now stores a subset of connected peers as backup bootstrap nodes ([kubo#8856](https://github.com/ipfs/kubo/pull/8856)).
These nodes are used in addition to the explicitly defined bootstrappers in the
[`Bootstrap`](https://github.com/ipfs/kubo/blob/master/docs/config.md#bootstrap) configuration.

This enhancement improves the resiliency of the system, as it eliminates the
necessity of relying solely on the default bootstrappers operated by Protocol
Labs for joining the public IPFS swarm. Previously, this level of robustness
was only available in LAN contexts with [mDNS peer discovery](https://github.com/ipfs/kubo/blob/master/docs/config.md#discoverymdns)
enabled.

With this update, the same level of robustness is applied to peers that lack
mDNS peers and solely rely on the public DHT.

#### Gateway: `DeserializedResponses` config flag

This release introduces the
[`Gateway.DeserializedResponses`](https://github.com/ipfs/kubo/blob/master/docs/config.md#gatewaydeserializedresponses)
configuration flag.

With this flag, one can explicitly configure whether the gateway responds to
deserialized requests or not. By default, this flag is enabled.

Disabling deserialized responses allows the
gateway to operate
as a [Trustless Gateway](https://specs.ipfs.tech/http-gateways/trustless-gateway/)
limited to three [verifiable](https://docs.ipfs.tech/reference/http/gateway/#trustless-verifiable-retrieval)
response types:
[application/vnd.ipld.raw](https://www.iana.org/assignments/media-types/application/vnd.ipld.raw),
[application/vnd.ipld.car](https://www.iana.org/assignments/media-types/application/vnd.ipld.car),
and [application/vnd.ipfs.ipns-record](https://www.iana.org/assignments/media-types/application/vnd.ipfs.ipns-record).

With deserialized responses disabled, the Kubo gateway can serve as a block
backend for other software (like
[bifrost-gateway](https://github.com/ipfs/bifrost-gateway#readme),
[IPFS in Chromium](https://github.com/little-bear-labs/ipfs-chromium/blob/main/README.md)
etc) without the usual risks associated with hosting deserialized data behind
third-party CIDs.

#### `client/rpc` migration of `go-ipfs-http-client`

The [`go-ipfs-http-client`](https://github.com/ipfs/go-ipfs-http-client) RPC has
been migrated into [`kubo/client/rpc`](../../client/rpc).

With this change the two will be kept in sync, in some previous releases we
updated the CoreAPI with new Kubo features but forgot to port thoses to the
http-client, making it impossible to use them together with the same coreapi
version.

For smooth transition `v0.7.0` of `go-ipfs-http-client` provides updated stubs
for Kubo `v0.21`.

#### Gateway: DAG-CBOR/-JSON previews and improved error pages

In this release, we improved the HTML templates of our HTTP gateway:

1. You can now preview the contents of a DAG-CBOR and DAG-JSON document from your browser, as well as follow any IPLD Links ([CBOR Tag 42](https://github.com/ipld/cid-cbor/)) contained within them.
2. The HTML directory listings now contain [updated, higher-definition icons](https://user-images.githubusercontent.com/5447088/241224419-5385793a-d3bb-40aa-8cb0-0382b5bc56a0.png).
3. On gateway error, instead of a plain text error message, web browsers will now get a friendly HTML response with more details regarding the problem.

HTML responses are returned when request's `Accept` header includes `text/html`.

| DAG-CBOR Preview | Error Page |
| ---- | ---- |
| ![DAG-CBOR Preview](https://github.com/ipfs/boxo/assets/5447088/973f05d1-5731-4469-9da5-d1d776891899) | ![Error Page](https://github.com/ipfs/boxo/assets/5447088/14c453df-adbc-4634-b038-133121914550) |

#### Gateway: subdomain redirects are now `text/html`

HTTP 301 redirects [from path to subdomain](https://specs.ipfs.tech/http-gateways/subdomain-gateway/#migrating-from-path-to-subdomain-gateway)
no longer include the target data in the body.
The data is returned only once, with the final HTTP 200 returned from the
target subdomain.

The HTTP 301 body now includes human-readable `text/html` message
for clients that do not follow redirects by default:

```console
$ curl "https://subdomain-gw.example.net/ipfs/${cid}/"
<a href="https://${cid}.ipfs.subdomain-gw.example.net/">Moved Permanently</a>.
```

Rationale can be found in [kubo#9913](https://github.com/ipfs/kubo/pull/9913).

#### Gateway: support for partial CAR export parameters (IPIP-402)

The gateway now supports optional CAR export parameters
`dag-scope=block|entity|all` and `entity-bytes=from:to` as specified in
[IPIP-402](https://specs.ipfs.tech/ipips/ipip-0402/).

Batch block retrieval minimizes round trips, catering to the requirements of
light HTTP clients for directory enumeration, range requests, and content path
resolution.

#### `ipfs dag stat` deduping statistics

`ipfs dat stat` now accept multiple CIDs and will dump advanced statistics
on the number of shared blocks and size of each CID.

```console
$ ipfs dag stat --progress=false QmfXuRxzyVy5H2LssLgtXrKCrNvDY8UBvMp2aoW8LS8AYA QmfZDyu2UFfUhL4VdHaw7Hofivmn5D4DdQj38Lwo86RsnB

CID                                           	Blocks         	Size
QmfXuRxzyVy5H2LssLgtXrKCrNvDY8UBvMp2aoW8LS8AYA	3              	2151
QmfZDyu2UFfUhL4VdHaw7Hofivmn5D4DdQj38Lwo86RsnB	4              	3223

Summary
Total Size: 3326
Unique Blocks: 5
Shared Size: 2048
Ratio: 1.615755
```

`ipfs --enc=json dag stat`'s keys are a non breaking change, new keys have been added but old keys with previous sementics are still here.

#### Accelerated DHT Client is no longer experimental

The [accelerated DHT client](docs/config.md#routingaccelerateddhtclient) is now
the main recommended solution for users who are hosting lots of data.
By trading some upfront DHT caching and increased memory usage,
one gets provider throughput improvements up to 6 millions times bigger dataset.
See [the docs](docs/config.md#routingaccelerateddhtclient) for more info.

The `Experimental.AcceleratedDHTClient` flag moved to [`Routing.AcceleratedDHTClient`](/docs/config.md#routingaccelerateddhtclient).
A config migration has been added to handle this automatically.

A new tracker estimates the providing speed and warns users if they
should be using AcceleratedDHTClient because they are falling behind.

### 📝 Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - fix: correctly handle migration of configs
  - fix(gateway): include CORS on subdomain redirects (#9994) ([ipfs/kubo#9994](https://github.com/ipfs/kubo/pull/9994))
  - fix: docker repository initialization race condition
  - chore: update version
  -  ([ipfs/kubo#9981](https://github.com/ipfs/kubo/pull/9981))
  -  ([ipfs/kubo#9960](https://github.com/ipfs/kubo/pull/9960))
  -  ([ipfs/kubo#9936](https://github.com/ipfs/kubo/pull/9936))
- github.com/ipfs/boxo (v0.8.1 -> v0.10.2-0.20230629143123-2d3edc552442):
  - chore: version 0.10.2
  - fix(gateway): include CORS on subdomain redirects (#395) ([ipfs/boxo#395](https://github.com/ipfs/boxo/pull/395))
  - fix(gateway): ensure 'X-Ipfs-Root' header is valid (#337) ([ipfs/boxo#337](https://github.com/ipfs/boxo/pull/337))
  - docs: prepare changelog for next release [ci skip]
  - chore: version 0.10.1 (#359) ([ipfs/boxo#359](https://github.com/ipfs/boxo/pull/359))
  - fix(gateway): allow CAR trustless requests with path
  - blockstore: replace go.uber.org/atomic with sync/atomic
  - fix(gateway): remove handleUnsupportedHeaders after go-ipfs 0.13 (#350) ([ipfs/boxo#350](https://github.com/ipfs/boxo/pull/350))
  - docs: update RELEASE.md based on 0.9 release (#343) ([ipfs/boxo#343](https://github.com/ipfs/boxo/pull/343))
  - chore: v0.10.0 (#345) ([ipfs/boxo#345](https://github.com/ipfs/boxo/pull/345))
  - docs(changelog): car params from ipip-402
  - docs(changelog): add gateway deserialized responses (#341) ([ipfs/boxo#341](https://github.com/ipfs/boxo/pull/341))
  - feat(gateway): implement IPIP-402 extensions for gateway CAR requests (#303) ([ipfs/boxo#303](https://github.com/ipfs/boxo/pull/303))
  - chore: release v0.9.0
  - changelog: update for 0.8.1 and 0.9.0
  - provider: second round of reprovider refactor
  - feat(unixfs): change protobuf package name to unixfs.v1.pb to prevent collisions with go-unixfs. Also regenerate protobufs with latest gogo
  - feat(ipld/merkledag): remove use of go-ipld-format global registry
  - feat(ipld/merkledag): updated to use its own global go-ipld-legacy registry instead of a shared global registry
  - chore: do not rely on deprecated logger
  - changelog: add changelog for async pin listing (#336) ([ipfs/boxo#336](https://github.com/ipfs/boxo/pull/336))
  - pinner: change the interface to have async pin listing
  - provider: revert throughput callback and related refactor
  - fix(gateway): question marks in url.Path when redirecting (#313) ([ipfs/boxo#313](https://github.com/ipfs/boxo/pull/313))
  - fix(gateway)!: no duplicate payload during subdomain redirects (#326) ([ipfs/boxo#326](https://github.com/ipfs/boxo/pull/326))
  - provider: add breaking changes to the changelog (#330) ([ipfs/boxo#330](https://github.com/ipfs/boxo/pull/330))
  - relocated magic numbers, updated Reprovide Interval from 24h to 22h
  - provider: refactor to only maintain one batched implementation and add throughput callback
  - feat(gateway): HTML preview for dag-cbor and dag-json (#315) ([ipfs/boxo#315](https://github.com/ipfs/boxo/pull/315))
  - coreiface: add a testing.T argument to the provider
  - feat(gateway): improved templates, user friendly errors (#298) ([ipfs/boxo#298](https://github.com/ipfs/boxo/pull/298))
  - feat(gateway)!: deserialised responses turned off by default (#252) ([ipfs/boxo#252](https://github.com/ipfs/boxo/pull/252))
  - fix(gw): missing return in error case ([ipfs/boxo#319](https://github.com/ipfs/boxo/pull/319))
  - feat(routing/http): pass records limit on routing.FindProviders (#299) ([ipfs/boxo#299](https://github.com/ipfs/boxo/pull/299))
  - bitswap/client: fix PeerResponseTrackerProbabilityOneKnownOneUnknownPeer
  - feat(gw): add ipfs_http_gw_car_stream_fail_duration_seconds (#312) ([ipfs/boxo#312](https://github.com/ipfs/boxo/pull/312))
  - feat(gw): add ipfs_http_gw_request_types metric (#311) ([ipfs/boxo#311](https://github.com/ipfs/boxo/pull/311))
  - refactor: simplify ipns validation in example
  - feat: add deprecator
  - fix(routing/v1): add newline in NDJSON responses (#300) ([ipfs/boxo#300](https://github.com/ipfs/boxo/pull/300))
  - feat(gateway): redirect ipns b58mh to cid (#236) ([ipfs/boxo#236](https://github.com/ipfs/boxo/pull/236))
  - refactor: replace assert.Nil for assert.NoError
  - tar: add test cases for validatePlatformPath
  - feat(ipns): helper ValidateWithPeerID and UnmarshalIpnsEntry (#294) ([ipfs/boxo#294](https://github.com/ipfs/boxo/pull/294))
  - Revert "feat: reusable ipns verify (#292)"
  - feat: reusable ipns verify (#292) ([ipfs/boxo#292](https://github.com/ipfs/boxo/pull/292))
  - refactor: remove badger, leveldb dependencies (#286) ([ipfs/boxo#286](https://github.com/ipfs/boxo/pull/286))
  - feat(routing/http): add streaming support (#18) ([ipfs/boxo#18](https://github.com/ipfs/boxo/pull/18))
  - feat(routing): allow-offline with routing put (#278) ([ipfs/boxo#278](https://github.com/ipfs/boxo/pull/278))
  - refactor(gateway): switch to xxhash/v2 (#285) ([ipfs/boxo#285](https://github.com/ipfs/boxo/pull/285))
- github.com/ipfs/go-ipfs-util (v0.0.2 -> v0.0.3):
  - docs: remove contribution section
  - chore: bump version
  - chore: deprecate types and readme
  - sync: update CI config files (#12) ([ipfs/go-ipfs-util#12](https://github.com/ipfs/go-ipfs-util/pull/12))
  - fix staticcheck ([ipfs/go-ipfs-util#9](https://github.com/ipfs/go-ipfs-util/pull/9))
- github.com/ipfs/go-ipld-format (v0.4.0 -> v0.5.0):
  - chore: release version v0.5.0
  - feat: remove block decoding global registry
  - sync: update CI config files (#75) ([ipfs/go-ipld-format#75](https://github.com/ipfs/go-ipld-format/pull/75))
  - sync: update CI config files (#74) ([ipfs/go-ipld-format#74](https://github.com/ipfs/go-ipld-format/pull/74))
- github.com/ipfs/go-ipld-legacy (v0.1.1 -> v0.2.1):
  - v0.2.1 ([ipfs/go-ipld-legacy#15](https://github.com/ipfs/go-ipld-legacy/pull/15))
  - Expose a constructor for making a decoder with an existing link system ([ipfs/go-ipld-legacy#14](https://github.com/ipfs/go-ipld-legacy/pull/14))
  - Update to v0.2.0 ([ipfs/go-ipld-legacy#13](https://github.com/ipfs/go-ipld-legacy/pull/13))
  - Remove global variable ([ipfs/go-ipld-legacy#12](https://github.com/ipfs/go-ipld-legacy/pull/12))
  - sync: update CI config files (#8) ([ipfs/go-ipld-legacy#8](https://github.com/ipfs/go-ipld-legacy/pull/8))
- github.com/ipfs/go-unixfsnode (v1.6.0 -> v1.7.1):
  - chore: bump to v1.7.1
  - test: remove unnecessary t.Log
  - test: check if reader reads only necessary blocks
  - fix: do not read extra block if offset = at+childSize
  - doc: added simple doc for testutil package
  - bump v1.7.0
  - feat(testutil): add test data generation utils (extracted from Lassie)
- github.com/libp2p/go-libp2p (v0.27.3 -> v0.27.7):
  - Release v0.27.7 (#2374) ([libp2p/go-libp2p#2374](https://github.com/libp2p/go-libp2p/pull/2374))
  - Release v0.27.6 (#2359) ([libp2p/go-libp2p#2359](https://github.com/libp2p/go-libp2p/pull/2359))
  - Release v0.27.5 (#2324) ([libp2p/go-libp2p#2324](https://github.com/libp2p/go-libp2p/pull/2324))
  - Bump version to v0.27.4
  - identify: reject signed peer records on peer ID mismatch
  - swarm: change maps with multiaddress keys to use strings (#2284) ([libp2p/go-libp2p#2284](https://github.com/libp2p/go-libp2p/pull/2284))
  - identify: avoid spuriously triggering pushes (#2299) ([libp2p/go-libp2p#2299](https://github.com/libp2p/go-libp2p/pull/2299))
- github.com/libp2p/go-libp2p-kad-dht (v0.23.0 -> v0.24.2):
  - chore: release v0.24.2
  - chore: release v0.24.1
  - fix: decrease tests noise, update kbucket and fix fixRTIUfNeeded
  - refactor: remove goprocess
  - fix: leaking go routines
  - chore: release v0.24.0
  - fix: don't add unresponsive DHT servers to the Routing Table (#820) ([libp2p/go-libp2p-kad-dht#820](https://github.com/libp2p/go-libp2p-kad-dht/pull/820))
- github.com/libp2p/go-libp2p-kbucket (v0.5.0 -> v0.6.3):
  - fix: fix abba bug in UsefullNewPeer ([libp2p/go-libp2p-kbucket#122](https://github.com/libp2p/go-libp2p-kbucket/pull/122))
  - chore: release v0.6.2 ([libp2p/go-libp2p-kbucket#121](https://github.com/libp2p/go-libp2p-kbucket/pull/121))
  - Replacing UsefulPeer() with UsefulNewPeer() ([libp2p/go-libp2p-kbucket#120](https://github.com/libp2p/go-libp2p-kbucket/pull/120))
  - chore: release 0.6.1 ([libp2p/go-libp2p-kbucket#119](https://github.com/libp2p/go-libp2p-kbucket/pull/119))
  - UsefulPeer function ([libp2p/go-libp2p-kbucket#113](https://github.com/libp2p/go-libp2p-kbucket/pull/113))
  - Fixed peer replacement with bucket size of 1. ([libp2p/go-libp2p-kbucket#117](https://github.com/libp2p/go-libp2p-kbucket/pull/117))
  - GenRandomKey function ([libp2p/go-libp2p-kbucket#116](https://github.com/libp2p/go-libp2p-kbucket/pull/116))
  - Removed maintainers from readme ([libp2p/go-libp2p-kbucket#115](https://github.com/libp2p/go-libp2p-kbucket/pull/115))
  - Add maintainers ([libp2p/go-libp2p-kbucket#114](https://github.com/libp2p/go-libp2p-kbucket/pull/114))
  - sync: update CI config files (#112) ([libp2p/go-libp2p-kbucket#112](https://github.com/libp2p/go-libp2p-kbucket/pull/112))
- github.com/libp2p/go-libp2p-routing-helpers (v0.6.2 -> v0.7.0):
  - chore: release v0.7.0
  - fix: iterate over keys manually in ProvideMany
- github.com/libp2p/go-reuseport (v0.2.0 -> v0.3.0):
  - release v0.3.0 (#103) ([libp2p/go-reuseport#103](https://github.com/libp2p/go-reuseport/pull/103))
  - fix error handling when setting socket options (#102) ([libp2p/go-reuseport#102](https://github.com/libp2p/go-reuseport/pull/102))
  - minor README updates (#96) ([libp2p/go-reuseport#96](https://github.com/libp2p/go-reuseport/pull/96))
  - sync: update CI config files (#94) ([libp2p/go-reuseport#94](https://github.com/libp2p/go-reuseport/pull/94))
  - feat: add a DialTimeout function ([libp2p/go-reuseport#92](https://github.com/libp2p/go-reuseport/pull/92))
- github.com/multiformats/go-multicodec (v0.8.1 -> v0.9.0):
  - Bump v0.9.0
  - Bump v0.8.2
  - chore: update submodules and go generate
  - chore: update submodules and go generate
  - chore: update submodules and go generate
  - chore: update submodules and go generate
  - chore: update submodules and go generate
  - chore: update submodules and go generate
- github.com/multiformats/go-multihash (v0.2.1 -> v0.2.3):
  - chore: release v0.2.3
  - perf: outline logic in Decode to allow for stack allocations
  - chore: release v0.2.2
  - sha256: drop minio in favor of crypto/sha256 for go1.21 and above
  - sync: update CI config files (#169) ([multiformats/go-multihash#169](https://github.com/multiformats/go-multihash/pull/169))
  - add handler for hasher.Write returned error ([multiformats/go-multihash#167](https://github.com/multiformats/go-multihash/pull/167))
  - sync: update CI config files (#165) ([multiformats/go-multihash#165](https://github.com/multiformats/go-multihash/pull/165))
  - test: add benchmark for all hash functions Sum

</details>

### 👨‍👩‍👧‍👦 Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Jorropo | 47 | +4394/-4458 | 202 |
| Henrique Dias | 48 | +4344/-3962 | 205 |
| Łukasz Magiera | 68 | +3604/-886 | 172 |
| Adin Schmahmann | 8 | +1754/-1057 | 37 |
| galargh | 7 | +1355/-1302 | 15 |
| Gus Eggert | 7 | +1566/-655 | 33 |
| rvagg | 1 | +396/-389 | 3 |
| Michael Muré | 3 | +547/-202 | 14 |
| Guillaume Michel - guissou | 5 | +153/-494 | 17 |
| guillaumemichel | 15 | +446/-189 | 28 |
| Laurent Senta | 4 | +472/-152 | 29 |
| Rod Vagg | 6 | +554/-37 | 23 |
| Marcin Rataj | 11 | +330/-82 | 21 |
| Arthur Gavazza | 1 | +296/-87 | 7 |
| Lucas Molas | 1 | +323/-56 | 6 |
| Marco Munizaga | 5 | +227/-97 | 17 |
| Alex | 8 | +163/-116 | 10 |
| Steven Allen | 11 | +154/-114 | 14 |
| Marten Seemann | 6 | +214/-41 | 12 |
| web3-bot | 9 | +76/-75 | 28 |
| Hector Sanjuan | 2 | +5/-96 | 4 |
| Sukun | 1 | +83/-17 | 3 |
| Steve Loeppky | 2 | +100/-0 | 2 |
| Edgar Lee | 1 | +46/-46 | 12 |
| Ivan Schasny | 1 | +67/-5 | 4 |
| imthe1 | 1 | +65/-3 | 5 |
| godcong | 2 | +30/-31 | 5 |
| Will Scott | 4 | +36/-23 | 6 |
| Petar Maymounkov | 1 | +45/-9 | 1 |
| Ross Jones | 1 | +43/-1 | 2 |
| William Entriken | 1 | +38/-0 | 1 |
| João Pedro | 1 | +35/-0 | 1 |
| jhertz | 1 | +21/-0 | 2 |
| Nikhilesh Susarla | 1 | +21/-0 | 3 |
| Matt Joiner | 1 | +11/-9 | 2 |
| Vlad | 2 | +4/-2 | 2 |
| Russell Dempsey | 2 | +4/-2 | 2 |
| Will | 2 | +2/-2 | 2 |
| Piotr Galar | 1 | +1/-1 | 1 |
| Joel Gustafson | 1 | +1/-1 | 1 |
| Dennis Trautwein | 1 | +1/-1 | 1 |
| Bryan Stenson | 1 | +1/-1 | 1 |
