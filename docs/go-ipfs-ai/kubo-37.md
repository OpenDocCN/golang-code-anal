# go-ipfs 源码解析 37

# Kubo changelog v0.15

## v0.15.0

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

- [Kubo changelog v0.15](#kubo-changelog-v015)
  - [v0.15.0](#v0150)
    - [Overview](#overview)
    - [🔦 Highlights](#-highlights)
      - [#️⃣ Blake 3 support](#️⃣-blake-3-support)
      - [💉 Fx Options plugin](#-fx-options-plugin)
      - [📁 `$IPFS_PATH/gateway` file](#-ipfs_pathgateway-file)
    - [Changelog](#changelog)
    - [Contributors](#contributors)


### 🔦 Highlights

This is a release mainly with bugfixing and library updates.
We are improving release speed and cadence trying to have a new release every 5 weeks.

#### #️⃣ Blake 3 support

You can now use `blake3` as a valid hash function:
- `ipfs block put --mhtype=blake3`
- `ipfs add --hash=blake3`

It uses a 32 bytes default size.
And verify up to 128 bytes.
Because `blake3` is variable output hash function, you can use a different digest length, set `mhlen`: `ipfs block put --mhtype=blake3 --mhlen=64`, `ipfs add` doesn't have this option yet.

#### 💉 Fx Options plugin

This adds a plugin interface that lets the plugin modify the fx options that are passed to fx when the app is initialized.
This means plugins can inject their own implementations of Kubo interfaces.
This enables granular customization of Kubo behavior by plugins, such as:

- Bitswap with custom filters (e.g. for CID blocking)
- Custom interface implementations such as Pinner or DAGService
- Dynamic configuration of libp2p ...

Here's an example plugin that overrides the default Pinner with a custom one:

```go
func (p *PinnerPlugin) Options(info core.FXNodeInfo) ([]fx.Option, error) {
	pinner := mypinner.New()    
	return append(info.FXOptions, fx.Replace(fx.Annotate(pinner, fx.As(new(pin.Pinner))))), nil
}
```

Extra plugin info [here](https://github.com/ipfs/kubo/blob/master/docs/plugins.md#fx-experimental).

#### 📁 `$IPFS_PATH/gateway` file

This adds a new file in the `IPFS_PATH` folder similar to `$IPFS_PATH/api` containing an address based on [`Addresses.Gateway`](https://github.com/ipfs/kubo/blob/master/docs/config.md#addressesgateway) configuration.

This file is in URL (RFC1738) format.

```console
$ cat ~/.ipfs/gateway
http://127.0.0.1:8080
```

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - chore: Release v0.15.0-rc1
  - Update RELEASE_ISSUE_TEMPLATE.md for 0.15
  - docs(add): skip binary name in helptext
  - docs(cli): clarify CID determinism in add command
  - docs(cli): clarify CAR format in dag export|import
  - test(gw): cors preflight with custom header
  - feat: make corehttp a reusable component ([ipfs/kubo#9070](https://github.com/ipfs/kubo/pull/9070))
  - feat: go-libp2p v0.21 (rcmgr auto scaling) ([ipfs/kubo#9074](https://github.com/ipfs/kubo/pull/9074))
  -  ([ipfs/kubo#9024](https://github.com/ipfs/kubo/pull/9024))
  -  ([ipfs/kubo#9100](https://github.com/ipfs/kubo/pull/9100))
  -  ([ipfs/kubo#9095](https://github.com/ipfs/kubo/pull/9095))
  - chore(cmd): add shutdown to CLI help ([ipfs/kubo#9194](https://github.com/ipfs/kubo/pull/9194))
  - docs: add fx plugin documentation to plugins.md (#9191) ([ipfs/kubo#9191](https://github.com/ipfs/kubo/pull/9191))
  - chore: switch to dist.ipfs.tech
  - feat: add fx options plugin
  - feat: add blake3 support
  - Add reference to Experimental config doc (#9181) ([ipfs/kubo#9181](https://github.com/ipfs/kubo/pull/9181))
  - feat: add $IPFS_PATH/gateway file
  - docs: replace `docs.ipfs.io` with `docs.ipfs.tech` (#9158) ([ipfs/kubo#9158](https://github.com/ipfs/kubo/pull/9158))
  - chore: fix markdown link syntax typo for AutoNAT.ServiceMode
  - chore: bump go-blockservice to only do put once
  - docs: update Arch Linux installation instructions
  - chore: update kubo-as-a-library example
  - docs(readme): add maintainer info (#9141) ([ipfs/kubo#9141](https://github.com/ipfs/kubo/pull/9141))
  - fix(gw): 404 when a valid DAG is missing link
  - fix(gw): directory URL normalization ([ipfs/kubo#9123](https://github.com/ipfs/kubo/pull/9123))
  - docs(config): add link to someguy router
  - fix: typo in README
  - Reproducible Builds: Update GOFLAGS for -trimpath
  - Merge v0.14.0 back into master
  - fix(gw): cache-control of index.html websites
  - chore(license): fix broken link to apache-2.0
  - fix: kubo in daemon and cli stdout
  - docs(readme): move content to docs website (#9102) ([ipfs/kubo#9102](https://github.com/ipfs/kubo/pull/9102))
  - fix(gw): no backlink when listing root dir
- github.com/ipfs/go-bitswap (v0.7.0 -> v0.9.0):
  - chore: release v0.9.0
  - feat: split client and server ([ipfs/go-bitswap#570](https://github.com/ipfs/go-bitswap/pull/570))
  - chore: remove goprocess from blockstoremanager
  - Don't add blocks to the datastore ([ipfs/go-bitswap#571](https://github.com/ipfs/go-bitswap/pull/571))
  - Remove dependency on travis package from go-libp2p-testing ([ipfs/go-bitswap#569](https://github.com/ipfs/go-bitswap/pull/569))
  - feat: add basic tracing (#562) ([ipfs/go-bitswap#562](https://github.com/ipfs/go-bitswap/pull/562))
- github.com/ipfs/go-blockservice (v0.3.0 -> v0.4.0):
  - write blocks retrieved from the exchange to the blockstore ([ipfs/go-blockservice#92](https://github.com/ipfs/go-blockservice/pull/92))
  - feat: add basic tracing ([ipfs/go-blockservice#91](https://github.com/ipfs/go-blockservice/pull/91))
- github.com/ipfs/go-ipfs-exchange-interface (v0.1.0 -> v0.2.0):
  - Rename HasBlock to NotifyNewBlocks, and make it accept multiple blocks ([ipfs/go-ipfs-exchange-interface#23](https://github.com/ipfs/go-ipfs-exchange-interface/pull/23))
- github.com/ipfs/go-ipfs-exchange-offline (v0.2.0 -> v0.3.0):
  - Exchange don't add blocks on their own anymore ([ipfs/go-ipfs-exchange-offline#47](https://github.com/ipfs/go-ipfs-exchange-offline/pull/47))
- github.com/ipfs/go-verifcid (v0.0.1 -> v0.0.2):
  - chore: release v0.0.2
  - feat: add blake3 as a good hash
  - sync: update CI config files (#12) ([ipfs/go-verifcid#12](https://github.com/ipfs/go-verifcid/pull/12))
  - Add license ([ipfs/go-verifcid#8](https://github.com/ipfs/go-verifcid/pull/8))
- github.com/ipld/go-codec-dagpb (v1.4.0 -> v1.4.1):
  - v1.4.1 bump
- github.com/libp2p/go-buffer-pool (v0.0.2 -> v0.1.0):
  - release v0.1.0 (#30) ([libp2p/go-buffer-pool#30](https://github.com/libp2p/go-buffer-pool/pull/30))
  - panic if a negative length is passed to BufferPool.Get (#28) ([libp2p/go-buffer-pool#28](https://github.com/libp2p/go-buffer-pool/pull/28))
  - sync: update CI config files (#22) ([libp2p/go-buffer-pool#22](https://github.com/libp2p/go-buffer-pool/pull/22))
  - sync: update CI config files (#20) ([libp2p/go-buffer-pool#20](https://github.com/libp2p/go-buffer-pool/pull/20))
  - test: fix gc test on go 1.16 ([libp2p/go-buffer-pool#18](https://github.com/libp2p/go-buffer-pool/pull/18))
  - fix staticcheck ([libp2p/go-buffer-pool#16](https://github.com/libp2p/go-buffer-pool/pull/16))
  - test: make sure we have the correct number of pools ([libp2p/go-buffer-pool#10](https://github.com/libp2p/go-buffer-pool/pull/10))
- github.com/libp2p/go-libp2p (v0.20.3 -> v0.21.0):
  - Release v0.21.0 (#1648) ([libp2p/go-libp2p#1648](https://github.com/libp2p/go-libp2p/pull/1648))
  - ping: optimize random number generation (#1658) ([libp2p/go-libp2p#1658](https://github.com/libp2p/go-libp2p/pull/1658))
  - feat: switch noise to use minio's SHA256 implementation (#1657) ([libp2p/go-libp2p#1657](https://github.com/libp2p/go-libp2p/pull/1657))
  - swarm: mark dialing WebTransport addresses as expensive (#1650) ([libp2p/go-libp2p#1650](https://github.com/libp2p/go-libp2p/pull/1650))
  - routedhost: fix decoding of relay peer ID (#1644) ([libp2p/go-libp2p#1644](https://github.com/libp2p/go-libp2p/pull/1644))
  - Release v0.21.0 RC (#1638) ([libp2p/go-libp2p#1638](https://github.com/libp2p/go-libp2p/pull/1638))
  - fix: return the best _acceptable_ conn in NewStream (#1604) ([libp2p/go-libp2p#1604](https://github.com/libp2p/go-libp2p/pull/1604))
  - use autoscaling limits (#1637) ([libp2p/go-libp2p#1637](https://github.com/libp2p/go-libp2p/pull/1637))
  - docs: point to SetDefaultServiceLimits in ResourceManager option (#1636) ([libp2p/go-libp2p#1636](https://github.com/libp2p/go-libp2p/pull/1636))
  - chore: update deps (#1634) ([libp2p/go-libp2p#1634](https://github.com/libp2p/go-libp2p/pull/1634))
  - Pass endpoint information to resource manager's OpenConnection (#1633) ([libp2p/go-libp2p#1633](https://github.com/libp2p/go-libp2p/pull/1633))
  - Add canonical peer status logs (#1624) ([libp2p/go-libp2p#1624](https://github.com/libp2p/go-libp2p/pull/1624))
  - move go-libp2p-circuit here ([libp2p/go-libp2p#1626](https://github.com/libp2p/go-libp2p/pull/1626))
  - swarm: fix logging of accepted connections (#1629) ([libp2p/go-libp2p#1629](https://github.com/libp2p/go-libp2p/pull/1629))
  - fix: deny connections to peers in the right place (#1627) ([libp2p/go-libp2p#1627](https://github.com/libp2p/go-libp2p/pull/1627))
  - ping: fix flaky test (#1617) ([libp2p/go-libp2p#1617](https://github.com/libp2p/go-libp2p/pull/1617))
  - chore: use the new multiaddr.Contains function (#1618) ([libp2p/go-libp2p#1618](https://github.com/libp2p/go-libp2p/pull/1618))
  - chore: stop using the deprecated mux.MuxedConn (#1614) ([libp2p/go-libp2p#1614](https://github.com/libp2p/go-libp2p/pull/1614))
  - logging: Add canonical log for misbehaving peers (#1600) ([libp2p/go-libp2p#1600](https://github.com/libp2p/go-libp2p/pull/1600))
  - use multiaddr ipcidr to parse multiaddr filters (#1606) ([libp2p/go-libp2p#1606](https://github.com/libp2p/go-libp2p/pull/1606))
  - tcp: unexport TcpTransport.Upgrader (#1596) ([libp2p/go-libp2p#1596](https://github.com/libp2p/go-libp2p/pull/1596))
  - muxer: expose func to create MuxedConn from backing Conn (#1609) ([libp2p/go-libp2p#1609](https://github.com/libp2p/go-libp2p/pull/1609))
  - remove legacy mDNS implementation (#1192) ([libp2p/go-libp2p#1192](https://github.com/libp2p/go-libp2p/pull/1192))
  - feat: allow dialing wss peers using DNS multiaddrs
  - fix natManager to close natManager.nat (#1468) ([libp2p/go-libp2p#1468](https://github.com/libp2p/go-libp2p/pull/1468))
  - Expose DefaultPerPeerRateLimit as var (#1580) ([libp2p/go-libp2p#1580](https://github.com/libp2p/go-libp2p/pull/1580))
  - swarm: add ListenClose (#1586) ([libp2p/go-libp2p#1586](https://github.com/libp2p/go-libp2p/pull/1586))
  - identify: Fix flaky tests (#1555) ([libp2p/go-libp2p#1555](https://github.com/libp2p/go-libp2p/pull/1555))
  - autonat: fix flaky TestAutoNATPrivate  (#1581) ([libp2p/go-libp2p#1581](https://github.com/libp2p/go-libp2p/pull/1581))
  - pstoremanager: fix test timeout (#1588) ([libp2p/go-libp2p#1588](https://github.com/libp2p/go-libp2p/pull/1588))
  - swarm: send notifications synchronously (#1562) ([libp2p/go-libp2p#1562](https://github.com/libp2p/go-libp2p/pull/1562))
  - basichost: fix flaky TestSignedPeerRecordWithNoListenAddrs (#1559) ([libp2p/go-libp2p#1559](https://github.com/libp2p/go-libp2p/pull/1559))
  - identify: fix flaky TestIdentifyDeltaOnProtocolChange (again) (#1582) ([libp2p/go-libp2p#1582](https://github.com/libp2p/go-libp2p/pull/1582))
  - tls: fix flaky TestInvalidCerts on Windows ([libp2p/go-libp2p#1560](https://github.com/libp2p/go-libp2p/pull/1560))
  - chore: log autorelay start failure error ([libp2p/go-libp2p#1583](https://github.com/libp2p/go-libp2p/pull/1583))
  - Add sanity check assertion (#1570) ([libp2p/go-libp2p#1570](https://github.com/libp2p/go-libp2p/pull/1570))
  - swarm: speed up the TestDialWorkerLoopConcurrentFailureStress test (#1573) ([libp2p/go-libp2p#1573](https://github.com/libp2p/go-libp2p/pull/1573))
  - chore: update examples to go-libp2p v0.20.0 (#1557) ([libp2p/go-libp2p#1557](https://github.com/libp2p/go-libp2p/pull/1557))
  - Wait a couple seconds for ID event (#1568) ([libp2p/go-libp2p#1568](https://github.com/libp2p/go-libp2p/pull/1568))
  - remove workspace and packages section from README (#1563) ([libp2p/go-libp2p#1563](https://github.com/libp2p/go-libp2p/pull/1563))
  - fix: mkreleaselog exclude autogenerated files (#1567) ([libp2p/go-libp2p#1567](https://github.com/libp2p/go-libp2p/pull/1567))
  - move resource manager integration tests to p2p/test/ (#1561) ([libp2p/go-libp2p#1561](https://github.com/libp2p/go-libp2p/pull/1561))
  - swarm: only dial a single transport in TestDialWorkerLoopBasic (#1526) ([libp2p/go-libp2p#1526](https://github.com/libp2p/go-libp2p/pull/1526))
- github.com/libp2p/go-libp2p-core (v0.16.1 -> v0.19.1):
  - Update version.json
  - Remove btcsuite/btcd dep (#272) ([libp2p/go-libp2p-core#272](https://github.com/libp2p/go-libp2p-core/pull/272))
  - Release v0.19.0 (#271) ([libp2p/go-libp2p-core#271](https://github.com/libp2p/go-libp2p-core/pull/271))
  - Add endpoint parameter to the OpenConnection method for ResourceManager (#257) ([libp2p/go-libp2p-core#257](https://github.com/libp2p/go-libp2p-core/pull/257))
  - Release v0.18.0 (#270) ([libp2p/go-libp2p-core#270](https://github.com/libp2p/go-libp2p-core/pull/270))
  - Add canonical peer status logging with sampling (#269) ([libp2p/go-libp2p-core#269](https://github.com/libp2p/go-libp2p-core/pull/269))
  - canonicallog: reduce log level to warning (#268) ([libp2p/go-libp2p-core#268](https://github.com/libp2p/go-libp2p-core/pull/268))
  - Only log once if we failed to convert from netAddr (#264) ([libp2p/go-libp2p-core#264](https://github.com/libp2p/go-libp2p-core/pull/264))
  - remove deprecated mux package (#265) ([libp2p/go-libp2p-core#265](https://github.com/libp2p/go-libp2p-core/pull/265))
  - remove the peer.Set (#261) ([libp2p/go-libp2p-core#261](https://github.com/libp2p/go-libp2p-core/pull/261))
  - Bump version (#259) ([libp2p/go-libp2p-core#259](https://github.com/libp2p/go-libp2p-core/pull/259))
  - Add canonical log for misbehaving peers (#258) ([libp2p/go-libp2p-core#258](https://github.com/libp2p/go-libp2p-core/pull/258))
- github.com/libp2p/go-libp2p-kad-dht (v0.16.0 -> v0.17.0):
  - Chore: bump version to v0.17.0
  - Update go-libp2p to v0.20.3 ([libp2p/go-libp2p-kad-dht#778](https://github.com/libp2p/go-libp2p-kad-dht/pull/778))
- github.com/libp2p/go-libp2p-peerstore (v0.6.0 -> v0.7.1):
  - Release v0.7.1 ([libp2p/go-libp2p-peerstore#202](https://github.com/libp2p/go-libp2p-peerstore/pull/202))
  - stop using the peer.Set (#201) ([libp2p/go-libp2p-peerstore#201](https://github.com/libp2p/go-libp2p-peerstore/pull/201))
  - feat: Use a clock interface in pstoreds as well ([libp2p/go-libp2p-peerstore#200](https://github.com/libp2p/go-libp2p-peerstore/pull/200))
  - feat: use a clock interface to better support testing for pstoremem ([libp2p/go-libp2p-peerstore#199](https://github.com/libp2p/go-libp2p-peerstore/pull/199))
  - pstoremem: fix slice preallocation in GetProtocols (#198) ([libp2p/go-libp2p-peerstore#198](https://github.com/libp2p/go-libp2p-peerstore/pull/198))
  - remove all calls to peer.ID.Validate ([libp2p/go-libp2p-peerstore#194](https://github.com/libp2p/go-libp2p-peerstore/pull/194))
  - remove the addr package ([libp2p/go-libp2p-peerstore#195](https://github.com/libp2p/go-libp2p-peerstore/pull/195))
  - move AddrList to pstoremen, unexport it ([libp2p/go-libp2p-peerstore#193](https://github.com/libp2p/go-libp2p-peerstore/pull/193))
  - optimize allocations in the memory address book ([libp2p/go-libp2p-peerstore#191](https://github.com/libp2p/go-libp2p-peerstore/pull/191))
  - implement a clean shutdown for the memory address book ([libp2p/go-libp2p-peerstore#192](https://github.com/libp2p/go-libp2p-peerstore/pull/192))
- github.com/libp2p/go-libp2p-resource-manager (v0.3.0 -> v0.5.3):
  - Chore: release patch v0.5.3 ([libp2p/go-libp2p-resource-manager#77](https://github.com/libp2p/go-libp2p-resource-manager/pull/77))
  - Add namespace to metrics ([libp2p/go-libp2p-resource-manager#79](https://github.com/libp2p/go-libp2p-resource-manager/pull/79))
  - Fix usage of make to reserve capacity, not values ([libp2p/go-libp2p-resource-manager#76](https://github.com/libp2p/go-libp2p-resource-manager/pull/76))
  - Add package docs ([libp2p/go-libp2p-resource-manager#75](https://github.com/libp2p/go-libp2p-resource-manager/pull/75))
  - chore: Release v0.5.2 ([libp2p/go-libp2p-resource-manager#74](https://github.com/libp2p/go-libp2p-resource-manager/pull/74))
  - Record which direction the resource was blocked ([libp2p/go-libp2p-resource-manager#72](https://github.com/libp2p/go-libp2p-resource-manager/pull/72))
  - Simplify mem graphs in stock Grafana dashboard ([libp2p/go-libp2p-resource-manager#73](https://github.com/libp2p/go-libp2p-resource-manager/pull/73))
  - feat: Handle multiple instances in stock Grafana dashboard ([libp2p/go-libp2p-resource-manager#70](https://github.com/libp2p/go-libp2p-resource-manager/pull/70))
  - Use templated version of Grafana dashboard json ([libp2p/go-libp2p-resource-manager#69](https://github.com/libp2p/go-libp2p-resource-manager/pull/69))
  - Release v0.5.1 ([libp2p/go-libp2p-resource-manager#66](https://github.com/libp2p/go-libp2p-resource-manager/pull/66))
  - Implement `json.Marshaler` interface for LimitConfig ([libp2p/go-libp2p-resource-manager#67](https://github.com/libp2p/go-libp2p-resource-manager/pull/67))
  - Don't wait for a chan that will never close ([libp2p/go-libp2p-resource-manager#65](https://github.com/libp2p/go-libp2p-resource-manager/pull/65))
  - release v0.5.0 ([libp2p/go-libp2p-resource-manager#60](https://github.com/libp2p/go-libp2p-resource-manager/pull/60))
  - Add docs around WithAllowlistedMultiaddrs. Expose allowlist ([libp2p/go-libp2p-resource-manager#63](https://github.com/libp2p/go-libp2p-resource-manager/pull/63))
  - fix marshalling of allowlisted scopes ([libp2p/go-libp2p-resource-manager#62](https://github.com/libp2p/go-libp2p-resource-manager/pull/62))
  - docs: describe how the limiter is configured, and how limits are scaled (#59) ([libp2p/go-libp2p-resource-manager#59](https://github.com/libp2p/go-libp2p-resource-manager/pull/59))
  - don't limit the number of FDs on Windows (#58) ([libp2p/go-libp2p-resource-manager#58](https://github.com/libp2p/go-libp2p-resource-manager/pull/58))
  - Add ability to configure allowlist limits ([libp2p/go-libp2p-resource-manager#57](https://github.com/libp2p/go-libp2p-resource-manager/pull/57))
  - rewrite limits to allow auto-scaling ([libp2p/go-libp2p-resource-manager#48](https://github.com/libp2p/go-libp2p-resource-manager/pull/48))
  - Release v0.4.0 ([libp2p/go-libp2p-resource-manager#56](https://github.com/libp2p/go-libp2p-resource-manager/pull/56))
  - feat: Out of the box metrics for resource manager ([libp2p/go-libp2p-resource-manager#54](https://github.com/libp2p/go-libp2p-resource-manager/pull/54))
  - feat: Allowlist ([libp2p/go-libp2p-resource-manager#47](https://github.com/libp2p/go-libp2p-resource-manager/pull/47))
  - trace the scope as a JSON object (#52) ([libp2p/go-libp2p-resource-manager#52](https://github.com/libp2p/go-libp2p-resource-manager/pull/52))
  - include current limits in debug messages ([libp2p/go-libp2p-resource-manager#42](https://github.com/libp2p/go-libp2p-resource-manager/pull/42))
  - add an ID to spans (#44) ([libp2p/go-libp2p-resource-manager#44](https://github.com/libp2p/go-libp2p-resource-manager/pull/44))
  - add a DefaultLimitConfig with infinite limits (#41) ([libp2p/go-libp2p-resource-manager#41](https://github.com/libp2p/go-libp2p-resource-manager/pull/41))
  - export the TraceEvt (#40) ([libp2p/go-libp2p-resource-manager#40](https://github.com/libp2p/go-libp2p-resource-manager/pull/40))
  - trace exact timestamps (#39) ([libp2p/go-libp2p-resource-manager#39](https://github.com/libp2p/go-libp2p-resource-manager/pull/39))
  - skip events that don't change anything in tracer (#38) ([libp2p/go-libp2p-resource-manager#38](https://github.com/libp2p/go-libp2p-resource-manager/pull/38))
  - fix typos in MetricsReporter docs
  - fix shadowing of service name (#37) ([libp2p/go-libp2p-resource-manager#37](https://github.com/libp2p/go-libp2p-resource-manager/pull/37))
  - add a timestamp to trace events (#34) ([libp2p/go-libp2p-resource-manager#34](https://github.com/libp2p/go-libp2p-resource-manager/pull/34))
- github.com/libp2p/go-libp2p-testing (v0.9.2 -> v0.11.0):
  - Release v0.11.0 ([libp2p/go-libp2p-testing#64](https://github.com/libp2p/go-libp2p-testing/pull/64))
  - Remove unused bench file and dep ([libp2p/go-libp2p-testing#63](https://github.com/libp2p/go-libp2p-testing/pull/63))
  - Release v0.10.0 ([libp2p/go-libp2p-testing#62](https://github.com/libp2p/go-libp2p-testing/pull/62))
  - Update go-libp2p-core dep ([libp2p/go-libp2p-testing#61](https://github.com/libp2p/go-libp2p-testing/pull/61))
  - remove suites (#60) ([libp2p/go-libp2p-testing#60](https://github.com/libp2p/go-libp2p-testing/pull/60))
  - don't continue on read / write error in stream suite (#59) ([libp2p/go-libp2p-testing#59](https://github.com/libp2p/go-libp2p-testing/pull/59))
  - remove debug logging from stream and muxer suite ([libp2p/go-libp2p-testing#58](https://github.com/libp2p/go-libp2p-testing/pull/58))
  - remove Travis package (#57) ([libp2p/go-libp2p-testing#57](https://github.com/libp2p/go-libp2p-testing/pull/57))
- github.com/lucas-clemente/quic-go (v0.27.1 -> v0.28.0):
  - update for Go 1.19beta1 (#3460) ([lucas-clemente/quic-go#3460](https://github.com/lucas-clemente/quic-go/pull/3460))
  - Deduplicate Alt-Svc header values (#3461) ([lucas-clemente/quic-go#3461](https://github.com/lucas-clemente/quic-go/pull/3461))
  - only set DF for sockets that can handle it (#3448) ([lucas-clemente/quic-go#3448](https://github.com/lucas-clemente/quic-go/pull/3448))
  - fix flaky HTTP/3 request body test (#3447) ([lucas-clemente/quic-go#3447](https://github.com/lucas-clemente/quic-go/pull/3447))
  - make the keep alive interval configurable (#3444) ([lucas-clemente/quic-go#3444](https://github.com/lucas-clemente/quic-go/pull/3444))
  - implement QUIC v2 ([lucas-clemente/quic-go#3432](https://github.com/lucas-clemente/quic-go/pull/3432))
  - allow HTTP clients and servers to take over the request stream ([lucas-clemente/quic-go#3437](https://github.com/lucas-clemente/quic-go/pull/3437))
  - remove the http3.DataStreamer (#3435) ([lucas-clemente/quic-go#3435](https://github.com/lucas-clemente/quic-go/pull/3435))
  - always reset header buffer, even when QPACK encoding fails (#3436) ([lucas-clemente/quic-go#3436](https://github.com/lucas-clemente/quic-go/pull/3436))
  - Change "HTTP/3" to "HTTP/3.0". (#3439) ([lucas-clemente/quic-go#3439](https://github.com/lucas-clemente/quic-go/pull/3439))
  - remove stray http3 connection file
  - pass frame / stream type parsing errors to the hijacker callbacks ([lucas-clemente/quic-go#3429](https://github.com/lucas-clemente/quic-go/pull/3429))
  - add test for bidirectional stream hijacker (#3434) ([lucas-clemente/quic-go#3434](https://github.com/lucas-clemente/quic-go/pull/3434))
  - make it possible to parse a varint at the end of a reader (#3428) ([lucas-clemente/quic-go#3428](https://github.com/lucas-clemente/quic-go/pull/3428))
  - don't ignore errors that occur when the TLS ClientHello is generated ([lucas-clemente/quic-go#3424](https://github.com/lucas-clemente/quic-go/pull/3424))
  - don't send path MTU probe packets on a timer (#3423) ([lucas-clemente/quic-go#3423](https://github.com/lucas-clemente/quic-go/pull/3423))
  - introduce a http3.RoundTripOpt to prevent closing of request stream (#3411) ([lucas-clemente/quic-go#3411](https://github.com/lucas-clemente/quic-go/pull/3411))
  - don't close the request stream when http3.DataStreamer was used (#3413) ([lucas-clemente/quic-go#3413](https://github.com/lucas-clemente/quic-go/pull/3413))
  - do not embed http.Server in http3.Server (#3397) ([lucas-clemente/quic-go#3397](https://github.com/lucas-clemente/quic-go/pull/3397))
  - remove error return value from ComposeVersionNegotiation (#3410) ([lucas-clemente/quic-go#3410](https://github.com/lucas-clemente/quic-go/pull/3410))
  - don't set receive buffer if it is already large enough (#3407) ([lucas-clemente/quic-go#3407](https://github.com/lucas-clemente/quic-go/pull/3407))
  - clone TLS conf in newClient (#3400) ([lucas-clemente/quic-go#3400](https://github.com/lucas-clemente/quic-go/pull/3400))
  - remove warning comments of stable implementation (#3399) ([lucas-clemente/quic-go#3399](https://github.com/lucas-clemente/quic-go/pull/3399))
  - fix parsing of request path for Extended CONNECT requests (#3388) ([lucas-clemente/quic-go#3388](https://github.com/lucas-clemente/quic-go/pull/3388))
  - update docs to reflect that we support RFC 9221 (Unreliable Datagrams) (#3382) ([lucas-clemente/quic-go#3382](https://github.com/lucas-clemente/quic-go/pull/3382))
  - fix deadlock on concurrent http3.Server.Serve and Close calls (#3387) ([lucas-clemente/quic-go#3387](https://github.com/lucas-clemente/quic-go/pull/3387))
  - reduce flakiness of deadline integration tests (#3383) ([lucas-clemente/quic-go#3383](https://github.com/lucas-clemente/quic-go/pull/3383))
  - protect against concurrent use of Stream.Write (#3381) ([lucas-clemente/quic-go#3381](https://github.com/lucas-clemente/quic-go/pull/3381))
  - protect against concurrent use of Stream.Read (#3380) ([lucas-clemente/quic-go#3380](https://github.com/lucas-clemente/quic-go/pull/3380))
  - Expose quic server closed err (#3395) ([lucas-clemente/quic-go#3395](https://github.com/lucas-clemente/quic-go/pull/3395))
  - implement HTTP/3 unidirectional stream hijacking (#3389) ([lucas-clemente/quic-go#3389](https://github.com/lucas-clemente/quic-go/pull/3389))
  - add LocalAddr and RemoteAddr functions to http3.StreamCreator (#3384) ([lucas-clemente/quic-go#3384](https://github.com/lucas-clemente/quic-go/pull/3384))
  - extend the HTTP/3 API for WebTransport support ([lucas-clemente/quic-go#3362](https://github.com/lucas-clemente/quic-go/pull/3362))
  - remove unneeded network from custom dial function used in HTTP/3 (#3368) ([lucas-clemente/quic-go#3368](https://github.com/lucas-clemente/quic-go/pull/3368))
- github.com/multiformats/go-multiaddr (v0.5.0 -> v0.6.0):
  - release v0.6.0 ([multiformats/go-multiaddr#178](https://github.com/multiformats/go-multiaddr/pull/178))
  - add WebTransport multiaddr components ([multiformats/go-multiaddr#176](https://github.com/multiformats/go-multiaddr/pull/176))
  - add ipcidr support (#177) ([multiformats/go-multiaddr#177](https://github.com/multiformats/go-multiaddr/pull/177))
  - add a Contains function (#172) ([multiformats/go-multiaddr#172](https://github.com/multiformats/go-multiaddr/pull/172))
- github.com/multiformats/go-multibase (v0.1.0 -> v0.1.1):
  - chore: release version 0.1.1
  - fix: add new emoji codepoint for Base256Emoji 🐉
- github.com/multiformats/go-multihash (v0.2.0 -> v0.2.1):
  - chore: release v0.2.1
  - feat: adding tests and finish variable sized functions
  - feat: add support for variable length hash functions
  - adding blake3 tests and fixing an incorrect error message. (#158) ([multiformats/go-multihash#158](https://github.com/multiformats/go-multihash/pull/158))

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 129 | +5612/-9895 | 345 |
| Marco Munizaga | 109 | +7689/-3221 | 181 |
| vyzo | 64 | +3972/-657 | 125 |
| Jorropo | 19 | +1977/-1611 | 109 |
| Steven Allen | 30 | +633/-593 | 54 |
| Jeromy Johnson | 5 | +1032/-64 | 16 |
| Marcin Rataj | 21 | +406/-200 | 59 |
| Michael Muré | 6 | +335/-250 | 14 |
| Gus Eggert | 8 | +336/-104 | 31 |
| Claudia Richoux | 3 | +181/-63 | 9 |
| Steve Loeppky | 11 | +95/-141 | 11 |
| Ian Davis | 4 | +126/-58 | 6 |
| hareku | 3 | +172/-6 | 7 |
| Ivan Trubach | 1 | +98/-74 | 6 |
| Raúl Kripalani | 2 | +69/-62 | 9 |
| Seungbae Yu | 1 | +41/-41 | 13 |
| Julien Muret | 1 | +60/-7 | 2 |
| Mark Gaiser | 1 | +64/-0 | 5 |
| Lars Gierth | 1 | +20/-29 | 4 |
| Cole Brown | 4 | +27/-19 | 4 |
| Chao Fei | 2 | +15/-30 | 9 |
| Nuno Diegues | 2 | +25/-18 | 9 |
| Jakub Sztandera | 1 | +37/-0 | 3 |
| Wiktor Jurkiewicz | 1 | +13/-5 | 1 |
| c r | 1 | +11/-6 | 3 |
| Christian Stewart | 1 | +15/-2 | 4 |
| Matt Robenolt | 1 | +15/-1 | 2 |
| aarshkshah1992 | 2 | +8/-2 | 2 |
| link2xt | 1 | +4/-4 | 1 |
| Aaron Riekenberg | 1 | +4/-4 | 4 |
| web3-bot | 3 | +7/-0 | 3 |
| Adrian Lanzafame | 1 | +3/-3 | 1 |
| Dmitriy Ryajov | 2 | +2/-3 | 2 |
| Brendan O'Brien | 1 | +5/-0 | 1 |
| millken | 1 | +1/-1 | 1 |
| lostystyg | 1 | +1/-1 | 1 |
| kpcyrd | 1 | +1/-1 | 1 |
| anders | 1 | +1/-1 | 1 |
| Rod Vagg | 1 | +1/-1 | 1 |
| Matt Joiner | 1 | +1/-1 | 1 |
| Leo Balduf | 1 | +1/-1 | 1 |
| Didrik Nordström | 1 | +2/-0 | 1 |
| Daniel Norman | 1 | +1/-1 | 1 |
| Antonio Navarro Perez | 1 | +1/-1 | 1 |
| Adin Schmahmann | 1 | +1/-1 | 1 |
| Lucas Molas | 1 | +1/-0 | 1 |


# Kubo changelog v0.16

## v0.16.0

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

- [Kubo changelog v0.16](#kubo-changelog-v016)
  - [v0.16.0](#v0160)
    - [Overview](#overview)
    - [🔦 Highlights](#-highlights)
      - [🛣️ More configurable delegated routing system](#️-more-configurable-delegated-routing-system)
      - [🌍 WebTransport new experimental Transport](#-webtransport-new-experimental-transport)
      - [🗃️ Hardened IPNS record verification](#-hardened-ipns-record-verification)
      - [🌉 Web Gateways now support _redirects files](#-web-gateways-now-support-_redirects-files)
      - [😻 Add files to MFS with ipfs add --to-files](#-add-files-to-mfs-with-ipfs-add---to-files)
    - [Changelog](#changelog)
    - [Contributors](#contributors)


### 🔦 Highlights

<!-- TODO -->

#### 🛣️ More configurable delegated routing system

Since Kubo v0.14.0 [Reframe protocol](https://github.com/ipfs/specs/tree/main/reframe#readme) has been supported as a new routing system.

Now, we allow to configure several routers working together, so you can have several `reframe` and `dht` routers making queries. You can use the special `parallel` and `sequential` routers to fill your needs.

Example configuration usage using the [Filecoin Network Indexer](https://docs.cid.contact/filecoin-network-indexer/overview) and the DHT, making first a query to the indexer, and timing out after 3 seconds.

```console
$ ipfs config Routing.Type --json '"custom"'

$ ipfs config Routing.Routers.CidContact --json '{
  "Type": "reframe",
  "Parameters": {
    "Endpoint": "https://cid.contact/reframe"
  }
}'

$ ipfs config Routing.Routers.WanDHT --json '{
  "Type": "dht",
  "Parameters": {
    "Mode": "auto",
    "PublicIPNetwork": true,
    "AcceleratedDHTClient": false
  }
}'

$ ipfs config Routing.Routers.ParallelHelper --json '{
  "Type": "parallel",
  "Parameters": {
    "Routers": [
        {
        "RouterName" : "CidContact",
        "IgnoreErrors" : true,
        "Timeout": "3s"
        },
        {
        "RouterName" : "WanDHT",
        "IgnoreErrors" : false,
        "Timeout": "5m",
        "ExecuteAfter": "2s"
        }
    ]
  }
}'

$ ipfs config Routing.Methods --json '{
      "find-peers": {
        "RouterName": "ParallelHelper"
      },
      "find-providers": {
        "RouterName": "ParallelHelper"
      },
      "get-ipns": {
        "RouterName": "ParallelHelper"
      },
      "provide": {
        "RouterName": "WanDHT"
      },
      "put-ipns": {
        "RouterName": "ParallelHelper"
      }
    }'

```

### 🌍 WebTransport new experimental Transport

A new feature of [`go-libp2p`](https://github.com/libp2p/go-libp2p/releases/tag/v0.23.0) is [WebTransport](https://github.com/libp2p/go-libp2p/issues/1717).

For now it is **disabled by default** and considered **experimental**.
If you find issues running it please [report them to us](https://github.com/ipfs/kubo/issues/new).

In the future Kubo will listen on WebTransport by default for anyone already listening on QUIC addresses.

WebTransport is a new transport protocol currently under development by the [IETF](https://datatracker.ietf.org/wg/webtrans/about/) and the [W3C](https://www.w3.org/TR/webtransport/), and [already implemented by Chrome](https://caniuse.com/webtransport).
Conceptually, it’s like WebSocket run over QUIC instead of TCP. Most importantly, it allows browsers to establish (secure!) connections to WebTransport servers without the need for CA-signed certificates,
thereby enabling any js-libp2p node running in a browser to connect to any kubo node, with zero manual configuration involved.

The previous alternative is websocket secure, which require installing a reverse proxy and TLS certificates manually.

#### How to enable WebTransport

Thoses steps are temporary and wont be needed once we make it enabled by default.

1. Enable the WebTransport transport:
   `ipfs config Swarm.Transports.Network.WebTransport --json true`
1. Add a listener address for WebTransport to your `Addresses.Swarm` key, for example:
   ```json
   [
     "/ip4/0.0.0.0/tcp/4001",
     "/ip4/0.0.0.0/udp/4001/quic",
     "/ip4/0.0.0.0/udp/4002/quic/webtransport"
   ]
   ```
1. Restart your daemon to apply the config changes.

### 🗃️ Hardened IPNS record verification

Records that do not have a valid IPNS V2 signature, or exceed the max size
limit, will no longer pass verification, and will be ignored by Kubo when
resolving `/ipns/{libp2p-key}` content paths.

Kubo continues publishing backward-compatible V1+V2 records that can be
resolved by V1-only (go-ipfs <0.9.0) clients.

More details can be found in _Backward Compatibility_, _Record Creation_, and
_Record Verification_ sections of the [updated IPNS
specification](https://github.com/ipfs/specs/pull/319/files).

### 🌉 Web Gateways now support `_redirects` files

This feature enables support for redirects, single-page applications (SPA),
custom 404 pages, and moving to IPFS-backed website hosting
[without breaking existing HTTP links](https://www.w3.org/Provider/Style/URI).

It is limited to websites hosted in web contexts with unique
[Origins](https://en.wikipedia.org/wiki/Same-origin_policy), such as
[subdomain](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway) and
[DNSLink](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#dnslink-gateway) gateways.
Redirect logic is evaluated only if the requested path is not in the DAG.

See more details and usage examples see
[docs.ipfs.tech: _Redirects, custom 404s, and SPA support_](https://docs.ipfs.tech/how-to/websites-on-ipfs/redirects-and-custom-404s/).

### 😻 Add files to MFS with `ipfs add --to-files`

Users no longer need to  call `ipfs files cp` after `ipfs add` to create a
reference in [MFS](https://docs.ipfs.tech/concepts/glossary/#mfs), or deal with
low level pins if they do not wish to do so. It is now possible to pass MFS
path in an optional `--to-files` to add data directly to MFS, without creating
a low level pin.

Before (Kubo <0.16.0):


```console
$ ipfs add cat.jpg
QmCID
$ ipfs files cp /ipfs/QmCID /mfs-cats/cat.jpg
$ ipfs pin rm QmCID # removing low level pin, since MFS is protecting from gc
```

Kubo 0.16.0 collapses the above steps into one:

```console
$ ipfs add --pin=false cat.jpg --to-files /mfs-cats/
```

A recursive add to MFS works too (below line will create `/lots-of-cats/` directory in MFS):

```console
$ ipfs add -r ./lots-of-cats/ --to-files /
```

For more information, see `ipfs add --help` and `ipfs files --help`.

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - fix: Set default Methods value to nil
  - docs: document remaining 0.16.0 features
  - docs: add WebTransport docs ([ipfs/kubo#9308](https://github.com/ipfs/kubo/pull/9308))
  - chore: bump version to 0.16.0-rc1
  - fix: ensure hasher is registered when using a hashing function
  - feat: add webtransport as an optin transport ([ipfs/kubo#9293](https://github.com/ipfs/kubo/pull/9293))
  - feat(gateway): _redirects file support (#8890) ([ipfs/kubo#8890](https://github.com/ipfs/kubo/pull/8890))
  - docs: fix typo in changelog-v0.16.0.md
  - Readme: Rewrite introduction and featureset (#9211) ([ipfs/kubo#9211](https://github.com/ipfs/kubo/pull/9211))
  - feat: Delegated routing with custom configuration. (#9274) ([ipfs/kubo#9274](https://github.com/ipfs/kubo/pull/9274))
  - Add <protocols> to `ipfs id -h` options (#9229) ([ipfs/kubo#9229](https://github.com/ipfs/kubo/pull/9229))
  - chore: bump go-libp2p v0.23.1 ([ipfs/kubo#9285](https://github.com/ipfs/kubo/pull/9285))
  - feat(cmds/add): --to-files option automates files cp (#8927) ([ipfs/kubo#8927](https://github.com/ipfs/kubo/pull/8927))
  - docs: fix broken ENS DoH example (#9281) ([ipfs/kubo#9281](https://github.com/ipfs/kubo/pull/9281))
  -  ([ipfs/kubo#9258](https://github.com/ipfs/kubo/pull/9258))
  -  ([ipfs/kubo#9213](https://github.com/ipfs/kubo/pull/9213))
  - docs: small typo in Dockerfile
  - feat: ipfs-webui v2.18.1
  - feat: ipfs-webui v2.18.0 (#9262) ([ipfs/kubo#9262](https://github.com/ipfs/kubo/pull/9262))
  - bump go-libp2p v0.22.0 & go1.18&go1.19 ([ipfs/kubo#9244](https://github.com/ipfs/kubo/pull/9244))
  - docs: change windows choco install command to point to go-ipfs
  - fix: pass the repo directory into the ignored_commit function
  - docs(cmds): daemon: update DHTClient description
  - fix(gw): send 200 for empty files
  - docs(readme): official vs unofficial packages
  - chore: remove Gateway.PathPrefixes
  - docs(readme): update Docker section
  - docs: fix markdown syntax typo in v0.15's changelog
  - chore: Release v0.15.0 ([ipfs/kubo#9236](https://github.com/ipfs/kubo/pull/9236))
  - chore: fix undiallable api and gateway files ([ipfs/kubo#9233](https://github.com/ipfs/kubo/pull/9233))
  - chore: Bump version to 0.16.0-dev
- github.com/ipfs/go-bitswap (v0.9.0 -> v0.10.2):
  - chore: release v0.10.2
  - fix: create a copy of the protocol slice in network.processSettings
  - chore: release v0.10.1
  - fix: incorrect type in the WithTracer polyfill option
  - chore: fix incorrect log message when a bad option is passed
  - chore: release v0.10.0
  - chore: update go-libp2p v0.22.0
- github.com/ipfs/go-cid (v0.2.0 -> v0.3.2):
  - chore: release v0.3.2
  - Revert "fix: bring back, but deprecate CodecToStr and Codecs"
  - chore: release v0.2.1
  - fix: bring back, but deprecate CodecToStr and Codecs
  - run gofmt -s
  - bump go.mod to Go 1.18 and run go fix
  - chore: release v0.3.0
  - fix: return nil Bytes() if the Cid in undef
  - Add MustParse ([ipfs/go-cid#139](https://github.com/ipfs/go-cid/pull/139))
- github.com/ipfs/go-datastore (v0.5.1 -> v0.6.0):
  - Release v0.6.0 (#194) ([ipfs/go-datastore#194](https://github.com/ipfs/go-datastore/pull/194))
  - feat: add Features + datastore scoping
  - chore: Fix comment typo (#191) ([ipfs/go-datastore#191](https://github.com/ipfs/go-datastore/pull/191))
- github.com/ipfs/go-delegated-routing (v0.3.0 -> v0.6.0):
  -  ([ipfs/go-delegated-routing#53](https://github.com/ipfs/go-delegated-routing/pull/53))
  -  ([ipfs/go-delegated-routing#52](https://github.com/ipfs/go-delegated-routing/pull/52))
  - Release v0.5.2 (#50) ([ipfs/go-delegated-routing#50](https://github.com/ipfs/go-delegated-routing/pull/50))
  - Fixed serialisation issue with multiadds (#49) ([ipfs/go-delegated-routing#49](https://github.com/ipfs/go-delegated-routing/pull/49))
  - Upgrade to IPLD `0.18.0`
  - Release v0.5.0 (#47) ([ipfs/go-delegated-routing#47](https://github.com/ipfs/go-delegated-routing/pull/47))
  - feat: use GET for FindProviders (#46) ([ipfs/go-delegated-routing#46](https://github.com/ipfs/go-delegated-routing/pull/46))
  - Update provide to take an array of keys, per spec (#45) ([ipfs/go-delegated-routing#45](https://github.com/ipfs/go-delegated-routing/pull/45))
  -  ([ipfs/go-delegated-routing#44](https://github.com/ipfs/go-delegated-routing/pull/44))
  - fix: upgrade edelweiss and rerun 'go generate' (#42) ([ipfs/go-delegated-routing#42](https://github.com/ipfs/go-delegated-routing/pull/42))
  - ci: add check to ensure generated files are up-to-date (#41) ([ipfs/go-delegated-routing#41](https://github.com/ipfs/go-delegated-routing/pull/41))
  - Add Provide RPC  (#37) ([ipfs/go-delegated-routing#37](https://github.com/ipfs/go-delegated-routing/pull/37))
  - upgrade to go-log/v2 (#34) ([ipfs/go-delegated-routing#34](https://github.com/ipfs/go-delegated-routing/pull/34))
- github.com/ipfs/go-ipns (v0.1.2 -> v0.3.0):
  - fix: require V2 signatures ([ipfs/go-ipns#41](https://github.com/ipfs/go-ipns/pull/41))
  - update go-libp2p to v0.22.0, release v0.2.0 (#39) ([ipfs/go-ipns#39](https://github.com/ipfs/go-ipns/pull/39))
  - use peer.IDFromBytes instead of peer.IDFromString (#38) ([ipfs/go-ipns#38](https://github.com/ipfs/go-ipns/pull/38))
  - sync: update CI config files (#34) ([ipfs/go-ipns#34](https://github.com/ipfs/go-ipns/pull/34))
- github.com/ipfs/go-pinning-service-http-client (v0.1.1 -> v0.1.2):
  - chore: release v0.1.2
  - fix: send up to nanosecond precision
  - refactor: cleanup Sprintf for Bearer token
  - sync: update CI config files ([ipfs/go-pinning-service-http-client#21](https://github.com/ipfs/go-pinning-service-http-client/pull/21))
- github.com/ipld/edelweiss (v0.1.4 -> v0.2.0):
  - Release v0.2.0 (#60) ([ipld/edelweiss#60](https://github.com/ipld/edelweiss/pull/60))
  - feat: add cachable modifier to methods (#48) ([ipld/edelweiss#48](https://github.com/ipld/edelweiss/pull/48))
  - adding licenses (#52) ([ipld/edelweiss#52](https://github.com/ipld/edelweiss/pull/52))
  - sync: update CI config files ([ipld/edelweiss#56](https://github.com/ipld/edelweiss/pull/56))
  - chore: replace deprecated ioutil with io/os ([ipld/edelweiss#59](https://github.com/ipld/edelweiss/pull/59))
  - Release v0.1.6
  - fix: iterate over BlueMap in deterministic order (#57) ([ipld/edelweiss#57](https://github.com/ipld/edelweiss/pull/57))
  - fix: wrap DAG-JSON serialization error (#55) ([ipld/edelweiss#55](https://github.com/ipld/edelweiss/pull/55))
  - update examples and harness
  - upgrade to go-log/v2 (#53) ([ipld/edelweiss#53](https://github.com/ipld/edelweiss/pull/53))
- github.com/ipld/go-ipld-prime (v0.17.0 -> v0.18.0):
  - Prepare v0.18.0
  - feat(bindnode): add a BindnodeRegistry utility (#437) ([ipld/go-ipld-prime#437](https://github.com/ipld/go-ipld-prime/pull/437))
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
  - feat: add release checklist (#442) ([ipld/go-ipld-prime#442](https://github.com/ipld/go-ipld-prime/pull/442))
- github.com/libp2p/go-flow-metrics (v0.0.3 -> v0.1.0):
  - introduce an API to set a mock clock (#20) ([libp2p/go-flow-metrics#20](https://github.com/libp2p/go-flow-metrics/pull/20))
  - chore: skip slow tests when short testing is specified ([libp2p/go-flow-metrics#16](https://github.com/libp2p/go-flow-metrics/pull/16))
- github.com/libp2p/go-libp2p (v0.21.0 -> v0.23.2):
  - release v0.23.2 (#1781) ([libp2p/go-libp2p#1781](https://github.com/libp2p/go-libp2p/pull/1781))
  - webtransport: return error before wrapping opened / accepted streams (#1775) ([libp2p/go-libp2p#1775](https://github.com/libp2p/go-libp2p/pull/1775))
  - release v0.23.1 (#1773) ([libp2p/go-libp2p#1773](https://github.com/libp2p/go-libp2p/pull/1773))
  - websocket: fix nil pointer in tlsClientConf (#1770) ([libp2p/go-libp2p#1770](https://github.com/libp2p/go-libp2p/pull/1770))
  - release v0.23.0 (#1764) ([libp2p/go-libp2p#1764](https://github.com/libp2p/go-libp2p/pull/1764))
  - noise: switch to proto2, use the new NoiseExtensions protobuf ([libp2p/go-libp2p#1762](https://github.com/libp2p/go-libp2p/pull/1762))
  - webtransport: add custom resolver to add SNI (#1761) ([libp2p/go-libp2p#1761](https://github.com/libp2p/go-libp2p/pull/1761))
  - swarm: skip dialing WebTransport addresses when we have QUIC addresses (#1756) ([libp2p/go-libp2p#1756](https://github.com/libp2p/go-libp2p/pull/1756))
  - webtransport: have the server send the certificates (#1757) ([libp2p/go-libp2p#1757](https://github.com/libp2p/go-libp2p/pull/1757))
  - noise: make it possible for the server to send early data (#1750) ([libp2p/go-libp2p#1750](https://github.com/libp2p/go-libp2p/pull/1750))
  - swarm: fix selection of transport for dialing (#1653) ([libp2p/go-libp2p#1653](https://github.com/libp2p/go-libp2p/pull/1653))
  - autorelay: Add a context.Context to WithPeerSource callback (#1736) ([libp2p/go-libp2p#1736](https://github.com/libp2p/go-libp2p/pull/1736))
  - webtransport: add and check the ?type=noise URL parameter (#1749) ([libp2p/go-libp2p#1749](https://github.com/libp2p/go-libp2p/pull/1749))
  - webtransport: disable HTTP origin check (#1752) ([libp2p/go-libp2p#1752](https://github.com/libp2p/go-libp2p/pull/1752))
  - noise: don't fail handshake when early data is received without handler (#1746) ([libp2p/go-libp2p#1746](https://github.com/libp2p/go-libp2p/pull/1746))
  - Add Resolver interface to transport (#1719) ([libp2p/go-libp2p#1719](https://github.com/libp2p/go-libp2p/pull/1719))
  - use new /libp2p/go-libp2p/core  pkg (#1745) ([libp2p/go-libp2p#1745](https://github.com/libp2p/go-libp2p/pull/1745))
  - yamux: pass constructors for peer resource scopes to session constructor (#1739) ([libp2p/go-libp2p#1739](https://github.com/libp2p/go-libp2p/pull/1739))
  - tcp: add an option to enable metrics (disabled by default) (#1734) ([libp2p/go-libp2p#1734](https://github.com/libp2p/go-libp2p/pull/1734))
  - move go-libp2p-webtransport to p2p/transport/webtransport ([libp2p/go-libp2p#1737](https://github.com/libp2p/go-libp2p/pull/1737))
  - autorelay: fix race condition in TestBackoff (#1731) ([libp2p/go-libp2p#1731](https://github.com/libp2p/go-libp2p/pull/1731))
  - rcmgr: increase default connection memory limit to 32 MB (#1740) ([libp2p/go-libp2p#1740](https://github.com/libp2p/go-libp2p/pull/1740))
  - quic: update quic-go to v0.29.0 (#1723) ([libp2p/go-libp2p#1723](https://github.com/libp2p/go-libp2p/pull/1723))
  - noise: implement an API to send and receive early data ([libp2p/go-libp2p#1728](https://github.com/libp2p/go-libp2p/pull/1728))
  - identify: make the protocol version configurable (#1724) ([libp2p/go-libp2p#1724](https://github.com/libp2p/go-libp2p/pull/1724))
  - Fix threshold calculation (#1722) ([libp2p/go-libp2p#1722](https://github.com/libp2p/go-libp2p/pull/1722))
  - connmgr: use clock interface (#1720) ([libp2p/go-libp2p#1720](https://github.com/libp2p/go-libp2p/pull/1720))
  - quic: increase the buffer size used for encoding qlogs (#1715) ([libp2p/go-libp2p#1715](https://github.com/libp2p/go-libp2p/pull/1715))
  - quic: add a WithMetrics option (#1716) ([libp2p/go-libp2p#1716](https://github.com/libp2p/go-libp2p/pull/1716))
  - add default listen addresses for QUIC (#1615) ([libp2p/go-libp2p#1615](https://github.com/libp2p/go-libp2p/pull/1615))
  - feat: inject DNS resolver (#1607) ([libp2p/go-libp2p#1607](https://github.com/libp2p/go-libp2p/pull/1607))
  - connmgr: prefer peers with no streams when closing connections (#1675) ([libp2p/go-libp2p#1675](https://github.com/libp2p/go-libp2p/pull/1675))
  - quic: add DisableReuseport option (#1476) ([libp2p/go-libp2p#1476](https://github.com/libp2p/go-libp2p/pull/1476))
  - release v0.22.0 ([libp2p/go-libp2p#1688](https://github.com/libp2p/go-libp2p/pull/1688))
  - fix: don't prefer local ports from other addresses when dialing (#1673) ([libp2p/go-libp2p#1673](https://github.com/libp2p/go-libp2p/pull/1673))
  - crypto: add better support for alternative backends (#1686) ([libp2p/go-libp2p#1686](https://github.com/libp2p/go-libp2p/pull/1686))
  - crypto/secp256k1: Remove btcsuite intermediary. (#1689) ([libp2p/go-libp2p#1689](https://github.com/libp2p/go-libp2p/pull/1689))
  - Update resource manager README (#1684) ([libp2p/go-libp2p#1684](https://github.com/libp2p/go-libp2p/pull/1684))
  - move go-libp2p-core here ([libp2p/go-libp2p#1683](https://github.com/libp2p/go-libp2p/pull/1683))
  - rcmgr: make scaling changes more intuitive (#1685) ([libp2p/go-libp2p#1685](https://github.com/libp2p/go-libp2p/pull/1685))
  - move go-eventbus here ([libp2p/go-libp2p#1681](https://github.com/libp2p/go-libp2p/pull/1681))
  - basichost: remove usage of MultistreamServerMatcher in test (#1680) ([libp2p/go-libp2p#1680](https://github.com/libp2p/go-libp2p/pull/1680))
  - sync: update CI config files (#1678) ([libp2p/go-libp2p#1678](https://github.com/libp2p/go-libp2p/pull/1678))
  - move go-libp2p-resource-manager to p2p/host/resource-manager ([libp2p/go-libp2p#1677](https://github.com/libp2p/go-libp2p/pull/1677))
  - chore: preallocate slices with known final size (#1679) ([libp2p/go-libp2p#1679](https://github.com/libp2p/go-libp2p/pull/1679))
  - autorelay: fix flaky TestMaxAge (#1676) ([libp2p/go-libp2p#1676](https://github.com/libp2p/go-libp2p/pull/1676))
  - move go-libp2p-peerstore to p2p/host/peerstore ([libp2p/go-libp2p#1667](https://github.com/libp2p/go-libp2p/pull/1667))
  - examples: remove ipfs components from echo (#1672) ([libp2p/go-libp2p#1672](https://github.com/libp2p/go-libp2p/pull/1672))
  - chore: update libp2p to v0.21 in examples (#1674) ([libp2p/go-libp2p#1674](https://github.com/libp2p/go-libp2p/pull/1674))
  - change the default key type to Ed25519 (#1576) ([libp2p/go-libp2p#1576](https://github.com/libp2p/go-libp2p/pull/1576))
  - autorelay: poll for new candidates when needed ([libp2p/go-libp2p#1587](https://github.com/libp2p/go-libp2p/pull/1587))
  - examples: fix unresponsive pubsub chat example (#1652) ([libp2p/go-libp2p#1652](https://github.com/libp2p/go-libp2p/pull/1652))
  - routed: respect force direct dial context (#1665) ([libp2p/go-libp2p#1665](https://github.com/libp2p/go-libp2p/pull/1665))
  - pstoremanager: fix flaky TestClose (#1649) ([libp2p/go-libp2p#1649](https://github.com/libp2p/go-libp2p/pull/1649))
  - Allow adding prologue to noise connections (#1663) ([libp2p/go-libp2p#1663](https://github.com/libp2p/go-libp2p/pull/1663))
  - connmgr: add nowatchdog go build tag (#1666) ([libp2p/go-libp2p#1666](https://github.com/libp2p/go-libp2p/pull/1666))
  - mdns: don't discover ourselves (#1661) ([libp2p/go-libp2p#1661](https://github.com/libp2p/go-libp2p/pull/1661))
  - Support generating custom x509 certificates (#1481) ([libp2p/go-libp2p#1481](https://github.com/libp2p/go-libp2p/pull/1481))
- github.com/libp2p/go-libp2p-core (v0.19.1 -> v0.20.1):
  - chore: release v0.20.1
  - feat: forward crypto/pb
  - release v0.20.0
  - deprecate this repo
  - stop using the deprecated io/ioutil package (#279) ([libp2p/go-libp2p-core#279](https://github.com/libp2p/go-libp2p-core/pull/279))
  - use a mock clock in bandwidth tests (#276) ([libp2p/go-libp2p-core#276](https://github.com/libp2p/go-libp2p-core/pull/276))
  - remove unused MultistreamSemverMatcher (#277) ([libp2p/go-libp2p-core#277](https://github.com/libp2p/go-libp2p-core/pull/277))
  - remove peer.IDFromString (#274) ([libp2p/go-libp2p-core#274](https://github.com/libp2p/go-libp2p-core/pull/274))
  - deprecate peer.Encode in favor of peer.ID.String (#275) ([libp2p/go-libp2p-core#275](https://github.com/libp2p/go-libp2p-core/pull/275))
  - deprecate peer.ID.Pretty (#273) ([libp2p/go-libp2p-core#273](https://github.com/libp2p/go-libp2p-core/pull/273))
- github.com/libp2p/go-libp2p-kad-dht (v0.17.0 -> v0.18.0):
  - update go-libp2p to v0.22.0, release v0.18.0 ([libp2p/go-libp2p-kad-dht#788](https://github.com/libp2p/go-libp2p-kad-dht/pull/788))
  - sync: update CI config files (#789) ([libp2p/go-libp2p-kad-dht#789](https://github.com/libp2p/go-libp2p-kad-dht/pull/789))
- github.com/libp2p/go-libp2p-peerstore (v0.7.1 -> v0.8.0):
  - release v0.8.0
  - deprecate this repo
  - fix flaky TestGCDelay (#206) ([libp2p/go-libp2p-peerstore#206](https://github.com/libp2p/go-libp2p-peerstore/pull/206))
  - fix flaky EWMA test (#205) ([libp2p/go-libp2p-peerstore#205](https://github.com/libp2p/go-libp2p-peerstore/pull/205))
- github.com/libp2p/go-libp2p-record (v0.1.3 -> v0.2.0):
  - update go-libp2p to v0.22.0, release v0.2.0 ([libp2p/go-libp2p-record#50](https://github.com/libp2p/go-libp2p-record/pull/50))
  - sync: update CI config files (#47) ([libp2p/go-libp2p-record#47](https://github.com/libp2p/go-libp2p-record/pull/47))
  - increase RSA key sizes in tests ([libp2p/go-libp2p-record#44](https://github.com/libp2p/go-libp2p-record/pull/44))
  - cleanup: fix staticcheck failures ([libp2p/go-libp2p-record#43](https://github.com/libp2p/go-libp2p-record/pull/43))
- github.com/libp2p/go-libp2p-routing-helpers (v0.2.3 -> v0.4.0):
  -  ([libp2p/go-libp2p-routing-helpers#62](https://github.com/libp2p/go-libp2p-routing-helpers/pull/62))
  -  ([libp2p/go-libp2p-routing-helpers#58](https://github.com/libp2p/go-libp2p-routing-helpers/pull/58))
  - Update version.json ([libp2p/go-libp2p-routing-helpers#60](https://github.com/libp2p/go-libp2p-routing-helpers/pull/60))
  - update go-libp2p to v0.22.0 ([libp2p/go-libp2p-routing-helpers#59](https://github.com/libp2p/go-libp2p-routing-helpers/pull/59))
  - sync: update CI config files (#53) ([libp2p/go-libp2p-routing-helpers#53](https://github.com/libp2p/go-libp2p-routing-helpers/pull/53))
  - fix staticcheck ([libp2p/go-libp2p-routing-helpers#49](https://github.com/libp2p/go-libp2p-routing-helpers/pull/49))
  - fix error handling in Parallel.search ([libp2p/go-libp2p-routing-helpers#48](https://github.com/libp2p/go-libp2p-routing-helpers/pull/48))
- github.com/libp2p/go-libp2p-testing (v0.11.0 -> v0.12.0):
  - release v0.12.0 (#67) ([libp2p/go-libp2p-testing#67](https://github.com/libp2p/go-libp2p-testing/pull/67))
  - chore: update to go-libp2p v0.22.0 (#66) ([libp2p/go-libp2p-testing#66](https://github.com/libp2p/go-libp2p-testing/pull/66))
  - remove the resource manager mocks (#65) ([libp2p/go-libp2p-testing#65](https://github.com/libp2p/go-libp2p-testing/pull/65))
- github.com/libp2p/go-openssl (v0.0.7 -> v0.1.0):
  - release v0.1.0 (#31) ([libp2p/go-openssl#31](https://github.com/libp2p/go-openssl/pull/31))
  - Fix build with OpenSSL 3.0 (#25) ([libp2p/go-openssl#25](https://github.com/libp2p/go-openssl/pull/25))
  - sync: update CI config files ([libp2p/go-openssl#24](https://github.com/libp2p/go-openssl/pull/24))
  - Add openssl.DialTimeout(network, addr, timeout, ctx, flags) call ([libp2p/go-openssl#26](https://github.com/libp2p/go-openssl/pull/26))
  - Add Ctx.SetMinProtoVersion and Ctx.SetMaxProtoVersion wrappers ([libp2p/go-openssl#27](https://github.com/libp2p/go-openssl/pull/27))
  - sync: update CI config files ([libp2p/go-openssl#17](https://github.com/libp2p/go-openssl/pull/17))
  - fix: unsafe pointer passing ([libp2p/go-openssl#18](https://github.com/libp2p/go-openssl/pull/18))
  - Update test RSA cert ([libp2p/go-openssl#15](https://github.com/libp2p/go-openssl/pull/15))
  - Fix tests ([libp2p/go-openssl#16](https://github.com/libp2p/go-openssl/pull/16))
  - Address `staticcheck` issues ([libp2p/go-openssl#14](https://github.com/libp2p/go-openssl/pull/14))
  - Enabled PEM files with CRLF line endings to be used (#10) ([libp2p/go-openssl#11](https://github.com/libp2p/go-openssl/pull/11))
- github.com/libp2p/zeroconf/v2 (v2.1.1 -> v2.2.0):
  - Fix windows libp2p (#29) ([libp2p/zeroconf#29](https://github.com/libp2p/zeroconf/pull/29))
  - Fix compatibility with some IoT devices using avahi 0.8-rc1 (#27) ([libp2p/zeroconf#27](https://github.com/libp2p/zeroconf/pull/27))
  - Add TTL server option (#23) ([libp2p/zeroconf#23](https://github.com/libp2p/zeroconf/pull/23))
- github.com/lucas-clemente/quic-go (v0.28.0 -> v0.29.1):
  - http3: fix double close of chan when using DontCloseRequestStream
  - add a logging.NullTracer and logging.NullConnectionTracer ([lucas-clemente/quic-go#3512](https://github.com/lucas-clemente/quic-go/pull/3512))
  - add support for providing a custom Connection ID generator via Config (#3452) ([lucas-clemente/quic-go#3452](https://github.com/lucas-clemente/quic-go/pull/3452))
  - fix typo in README
  - fix datagram support detection (#3511) ([lucas-clemente/quic-go#3511](https://github.com/lucas-clemente/quic-go/pull/3511))
  - use a single Go routine to send copies of CONNECTION_CLOSE packets ([lucas-clemente/quic-go#3514](https://github.com/lucas-clemente/quic-go/pull/3514))
  - add YoMo to list of projects in README (#3513) ([lucas-clemente/quic-go#3513](https://github.com/lucas-clemente/quic-go/pull/3513))
  - http3: fix listening on both QUIC and TCP (#3465) ([lucas-clemente/quic-go#3465](https://github.com/lucas-clemente/quic-go/pull/3465))
  - Disable anti-amplification limit by address validation token (#3326) ([lucas-clemente/quic-go#3326](https://github.com/lucas-clemente/quic-go/pull/3326))
  - fix typo in README
  - implement a new API to let servers control client address verification ([lucas-clemente/quic-go#3501](https://github.com/lucas-clemente/quic-go/pull/3501))
  - use a generic streams map for incoming streams ([lucas-clemente/quic-go#3489](https://github.com/lucas-clemente/quic-go/pull/3489))
  - fix unreachable code after log.Fatal in fuzzing corpus generator (#3496) ([lucas-clemente/quic-go#3496](https://github.com/lucas-clemente/quic-go/pull/3496))
  - use generic Min and Max functions ([lucas-clemente/quic-go#3483](https://github.com/lucas-clemente/quic-go/pull/3483))
  - add QPACK (RFC 9204) to the list of supported RFCs (#3485) ([lucas-clemente/quic-go#3485](https://github.com/lucas-clemente/quic-go/pull/3485))
  - add a function to distinguish between long and short header packets (#3498) ([lucas-clemente/quic-go#3498](https://github.com/lucas-clemente/quic-go/pull/3498))
  - use a generic streams map for outgoing streams (#3488) ([lucas-clemente/quic-go#3488](https://github.com/lucas-clemente/quic-go/pull/3488))
  - update golangci-lint action to v3, golangci-lint to v1.48.0 (#3499) ([lucas-clemente/quic-go#3499](https://github.com/lucas-clemente/quic-go/pull/3499))
  - use a generic linked list (#3487) ([lucas-clemente/quic-go#3487](https://github.com/lucas-clemente/quic-go/pull/3487))
  - drop support for Go 1.16 and 1.17 (#3482) ([lucas-clemente/quic-go#3482](https://github.com/lucas-clemente/quic-go/pull/3482))
  - optimize FirstOutstanding in the sent packet history (#3467) ([lucas-clemente/quic-go#3467](https://github.com/lucas-clemente/quic-go/pull/3467))
  - update supported RFCs in README (#3456) ([lucas-clemente/quic-go#3456](https://github.com/lucas-clemente/quic-go/pull/3456))
  - http3: ignore context after response when using DontCloseRequestStream (#3473) ([lucas-clemente/quic-go#3473](https://github.com/lucas-clemente/quic-go/pull/3473))
- github.com/marten-seemann/webtransport-go (null -> v0.1.1):
  - release v0.1.1 (#31) ([marten-seemann/webtransport-go#31](https://github.com/marten-seemann/webtransport-go/pull/31))
  - fix double close of chan when using DontCloseRequestStream
- github.com/multiformats/go-base32 (v0.0.4 -> v0.1.0):
  - chore: bump version to 0.1.0
  - fix: fix staticcheck complaints
  - run gofmt -s
  - sync: update CI config files (#5) ([multiformats/go-base32#5](https://github.com/multiformats/go-base32/pull/5))
- github.com/multiformats/go-multiaddr (v0.6.0 -> v0.7.0):
  - Release v0.7.0 ([multiformats/go-multiaddr#183](https://github.com/multiformats/go-multiaddr/pull/183))
  - use decimal numbers for multicodecs ([multiformats/go-multiaddr#184](https://github.com/multiformats/go-multiaddr/pull/184))
  - Fix comment on Decapsulate ([multiformats/go-multiaddr#181](https://github.com/multiformats/go-multiaddr/pull/181))
  -  ([multiformats/go-multiaddr#182](https://github.com/multiformats/go-multiaddr/pull/182))
  - sync: update CI config files (#180) ([multiformats/go-multiaddr#180](https://github.com/multiformats/go-multiaddr/pull/180))
  - Add webrtc (#179) ([multiformats/go-multiaddr#179](https://github.com/multiformats/go-multiaddr/pull/179))
- github.com/multiformats/go-multicodec (v0.5.0 -> v0.6.0):
  - chore: version bump 0.6.0
  - fix: replace io/ioutil with io
  - bump go.mod to Go 1.18 and run go fix

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 236 | +12637/-24326 | 1152 |
| Raúl Kripalani | 118 | +11626/-4136 | 422 |
| vyzo | 144 | +10129/-3665 | 230 |
| galargh | 9 | +5293/-5298 | 26 |
| Marco Munizaga | 83 | +7502/-3080 | 147 |
| Antonio Navarro Perez | 33 | +4074/-1240 | 78 |
| Steven Allen | 98 | +1974/-1693 | 202 |
| Cole Brown | 57 | +2169/-1338 | 95 |
| Rod Vagg | 21 | +2588/-768 | 56 |
| Gus Eggert | 16 | +2011/-1226 | 36 |
| Yusef Napora | 6 | +2738/-187 | 43 |
| Raúl Kripalani | 2 | +1000/-889 | 18 |
| Łukasz Magiera | 26 | +1312/-500 | 54 |
| Will | 2 | +1593/-200 | 18 |
| Jorropo | 31 | +924/-712 | 204 |
| Juan Batiz-Benet | 2 | +1531/-9 | 21 |
| Jeromy | 14 | +691/-468 | 51 |
| Petar Maymounkov | 4 | +469/-285 | 25 |
| Jeromy Johnson | 24 | +474/-204 | 116 |
| Justin Johnson | 1 | +582/-93 | 7 |
| Aarsh Shah | 24 | +377/-105 | 34 |
| web3-bot | 18 | +246/-228 | 93 |
| Masih H. Derkani | 2 | +197/-213 | 21 |
| Marcin Rataj | 9 | +211/-176 | 16 |
| adam | 4 | +235/-49 | 9 |
| Jakub Sztandera | 9 | +203/-73 | 13 |
| Guilhem Fanton | 1 | +216/-48 | 5 |
| Lucas Molas | 1 | +219/-9 | 3 |
| Peter Argue | 1 | +166/-36 | 3 |
| Vibhav Pant | 4 | +186/-12 | 7 |
| Adrian Lanzafame | 3 | +180/-16 | 5 |
| Lars Gierth | 5 | +151/-41 | 25 |
| João Oliveirinha | 1 | +124/-38 | 11 |
| dignifiedquire | 3 | +122/-33 | 6 |
| Chinmay Kousik | 2 | +128/-4 | 7 |
| Toby | 1 | +89/-36 | 4 |
| Oleg Jukovec | 3 | +111/-14 | 8 |
| Whyrusleeping | 2 | +120/-0 | 6 |
| KevinZønda | 1 | +81/-20 | 2 |
| wzp | 2 | +86/-3 | 2 |
| Benedikt Spies | 1 | +75/-12 | 8 |
| nisainan | 1 | +33/-43 | 12 |
| Tshaka Eric Lekholoane | 1 | +57/-19 | 6 |
| cpuchip | 1 | +65/-6 | 2 |
| Roman Proskuryakov | 2 | +69/-0 | 2 |
| Arceliar | 2 | +36/-28 | 2 |
| Maxim Merzhanov | 1 | +29/-24 | 1 |
| Richard Ramos | 1 | +51/-0 | 2 |
| Dave Collins | 1 | +25/-25 | 4 |
| Leo Balduf | 2 | +37/-10 | 3 |
| David Aronchick | 1 | +42/-0 | 3 |
| Didrik Nordström | 1 | +35/-6 | 1 |
| Vasco Santos | 1 | +20/-20 | 7 |
| Jesse Bouwman | 1 | +19/-21 | 1 |
| Ivan Schasny | 2 | +22/-14 | 4 |
| MGMCN | 1 | +9/-24 | 2 |
| Brian Meek | 1 | +14/-17 | 4 |
| Ian Davis | 3 | +21/-9 | 5 |
| Mars Zuo | 1 | +7/-18 | 1 |
| RubenKelevra | 1 | +10/-10 | 1 |
| mojatter | 1 | +9/-8 | 1 |
| Cory Schwartz | 1 | +0/-17 | 1 |
| Steve Loeppky | 6 | +7/-6 | 6 |
| Matt Joiner | 2 | +10/-3 | 2 |
| Winterhuman | 2 | +7/-5 | 2 |
| Dmitry Yu Okunev | 1 | +5/-7 | 5 |
| corverroos | 1 | +7/-4 | 2 |
| Marcel Gregoriadis | 1 | +9/-0 | 1 |
| Ignacio Hagopian | 2 | +7/-2 | 2 |
| Julien Muret | 1 | +4/-4 | 2 |
| Eclésio Junior | 1 | +8/-0 | 1 |
| Stephan Eberle | 1 | +4/-3 | 1 |
| muXxer | 1 | +3/-3 | 1 |
| eth-limo | 1 | +3/-3 | 2 |
| Russell Dempsey | 2 | +4/-2 | 2 |
| Sergey | 1 | +1/-3 | 1 |
| Jun10ng | 2 | +2/-2 | 2 |
| Jorik Schellekens | 1 | +2/-2 | 1 |
| Eli Wang | 1 | +2/-2 | 1 |
| Andreas Linde | 1 | +4/-0 | 1 |
| whyrusleeping | 1 | +2/-1 | 1 |
| xiabin | 1 | +1/-1 | 1 |
| star | 1 | +0/-2 | 1 |
| fanweixiao | 1 | +1/-1 | 1 |
| dbadoy4874 | 1 | +1/-1 | 1 |
| bigs | 1 | +1/-1 | 1 |
| Tarun Bansal | 1 | +1/-1 | 1 |
| Mikerah | 1 | +1/-1 | 1 |
| Mike Goelzer | 1 | +2/-0 | 1 |
| Max Inden | 1 | +1/-1 | 1 |
| Kevin Mai-Husan Chia | 1 | +1/-1 | 1 |
| John B Nelson | 1 | +1/-1 | 1 |
| Eli Bailey | 1 | +1/-1 | 1 |
| Bryan Stenson | 1 | +1/-1 | 1 |
| Alex Stokes | 1 | +1/-1 | 1 |
| Abirdcfly | 1 | +1/-1 | 1 |


# Kubo changelog v0.17

## v0.17.0

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

- [Kubo changelog v0.17](#kubo-changelog-v017)
  - [v0.17.0](#v0170)
    - [Overview](#overview)
    - [🔦 Highlights](#-highlights)
      - [libp2p resource management enabled by default](#libp2p-resource-management-enabled-by-default)
      - [Implicit connection manager limits](#implicit-connection-manager-limits)
      - [TAR Response Format on Gateways](#tar-response-format-on-gateways)
      - [Dialling `/wss` peer behind a reverse proxy](#dialling-wss-peer-behind-a-reverse-proxy)
    - [Changelog](#changelog)
    - [Contributors](#contributors)

### 🔦 Highlights

<!-- TODO -->

#### libp2p resource management enabled by default

To help protect nodes from DoS (resource exhaustion) and eclipse attacks, 
go-libp2p released a [Network Resource Manager](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager) with a host of improvements throughout 2022.  

Kubo first [exposed this functionality in Kubo 0.13](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.13.md#-libp2p-network-resource-manager-swarmresourcemgr), 
but it was disabled by default.

The resource manager is now enabled by default to protect nodes.  
The defaults balance providing protection from various attacks while still enabling normal usecases to work as expected.

If you want to adjust the defaults, then you can:
1. bound the amount of memory and file descriptors that libp2p will use with [Swarm.ResourceMgr.MaxMemory](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmresourcemgrmaxmemory) 
and [Swarm.ResourceMgr.MaxFileDescriptors](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmresourcemgrmaxfiledescriptors) and/or
2. override any specific resource scopes/limits with [Swarm.ResourceMgr.Limits](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmresourcemgrlimits)

See [Swarm.ResourceMgr](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmresourcemgr) for
1. what limits are set by default,
2. example override configuration, 
3. how to access Prometheus metrics and view Grafana dashboards of resource usage, and 
4. how to set explicit "allow lists" to protect against eclipse attacks. 

#### Implicit connection manager limits

Starting with this release, `ipfs init` will no longer store the default
[Connection Manager](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmconnmgr)
limits in the user config under `Swarm.ConnMgr`.

Users are still free to use this setting to set custom values, but for most use
cases, the defaults provided with the latest Kubo release should be sufficient.

To remove any custom limits and switch to the implicit defaults managed by Kubo:

```console
$ ipfs config --json Swarm.ConnMgr '{}'
```

We will be adjusting defaults in the future releases.

#### TAR Response Format on Gateways

Implemented [IPIP-288](https://github.com/ipfs/specs/pull/288) which adds
support for requesting deserialized UnixFS directory as a TAR stream.

HTTP clients can request TAR response by passing the `?format=tar` URL
parameter, or setting `Accept: application/x-tar` HTTP header:

```console
$ export DIR_CID=bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq
$ curl -H "Accept: application/x-tar" "http://127.0.0.1:8080/ipfs/$DIR_CID" > dir.tar
$ curl "http://127.0.0.1:8080/ipfs/$DIR_CID?format=tar" | tar xv
bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq
bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq/1 - Barrel - Part 1 - alt.txt
bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq/1 - Barrel - Part 1 - transcript.txt
bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq/1 - Barrel - Part 1.png
```

#### Dialling `/wss` peer behind a reverse proxy

This release resolves a regression introduced in Kubo 0.16, making it possible
again to connect to a peer over a WebSockets endpoint (`/wss`) that is
deployed behind a reverse proxy.

More details in [go-libp2p release notes](https://github.com/libp2p/go-libp2p/releases/tag/v0.23.3).

### Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - chore: bump version to v0.17.0 ([ipfs/kubo#9427](https://github.com/ipfs/kubo/pull/9427))
  - chore: bump version to v0.17.0-rc2 ([ipfs/kubo#9414](https://github.com/ipfs/kubo/pull/9414))
  - Doc improvements and changelog for resource manager (#9413) ([ipfs/kubo#9413](https://github.com/ipfs/kubo/pull/9413))
  - fix(docs): typo
  - docs: document /wss fixes in 0.17
  - refactor(config): remove Swarm.ConnMgr defaults
  - fix(config): skip nulls in ResourceMgr
  - Apply go fmt
  - Update core/node/libp2p/rcmgr_defaults.go
  - Remove limitation by HighWater param.
  - Fix RM errors when acceleratedDHT is active
  - docs: Deprecate Reframe on docs. (#9401) ([ipfs/kubo#9401](https://github.com/ipfs/kubo/pull/9401))
  - chore: bump version to v0.17.0-rc1 ([ipfs/kubo#9394](https://github.com/ipfs/kubo/pull/9394))
  - feat: Improve ResourceManager UX (#9338) ([ipfs/kubo#9338](https://github.com/ipfs/kubo/pull/9338))
  - feat: ipfs-webui 2.20.0
  - docs: note log tail is broken (#9383) ([ipfs/kubo#9383](https://github.com/ipfs/kubo/pull/9383))
  - feat(gateway): TAR response format (#9029) ([ipfs/kubo#9029](https://github.com/ipfs/kubo/pull/9029))
  - fix: error when using huge json limit file
  - chore: go-multicodec v0.7.0
  - fix: remove old unused buggy coredag code
  - feat: Add command line completion for fish
  - chore: delete snap configuration ([ipfs/kubo#9352](https://github.com/ipfs/kubo/pull/9352))
  - docs: update scoop package
  - docs: init release issue template improvement process v0.16.0 ([ipfs/kubo#9283](https://github.com/ipfs/kubo/pull/9283))
  - feat: add delegated routing metrics (#9354) ([ipfs/kubo#9354](https://github.com/ipfs/kubo/pull/9354))
  - chore: create v0.17.md changelog ([ipfs/kubo#9353](https://github.com/ipfs/kubo/pull/9353))
  - docs: pin remote arg
  - feat: webui@v2.19.0
  - test(car): export/import of (dag-)cbor/json codecs
  - add refs local alias repo ls (#9320) ([ipfs/kubo#9320](https://github.com/ipfs/kubo/pull/9320))
  - docs(cmds): Clarify block fetching of refs endpoint.
  - chore(cmds): dag import: use ipld legacy decode ([ipfs/kubo#9219](https://github.com/ipfs/kubo/pull/9219))
  - fix ipfs swarm peering crash in offline mode (#9261) ([ipfs/kubo#9261](https://github.com/ipfs/kubo/pull/9261))
  - feat: remove provider delay interval in bitswap (#9053) ([ipfs/kubo#9053](https://github.com/ipfs/kubo/pull/9053))
  - feat: --reset flag on swarm limit command (#9310) ([ipfs/kubo#9310](https://github.com/ipfs/kubo/pull/9310))
  - fix: add InlineDNSLink flag to PublicGateways config (#9328) ([ipfs/kubo#9328](https://github.com/ipfs/kubo/pull/9328))
  - docs: Fix typo and grammar in README
  - ci: add stylecheck to golangci-lint (#9334) ([ipfs/kubo#9334](https://github.com/ipfs/kubo/pull/9334))
  - Fix: `swarm stats all` command
  - Merge release v0.16.0 back into master ([ipfs/kubo#9324](https://github.com/ipfs/kubo/pull/9324))
  - fix: Set default Methods value to nil
  - docs: add WebTransport docs ([ipfs/kubo#9314](https://github.com/ipfs/kubo/pull/9314))
  - chore: bump version to 0.17.0-dev
- github.com/ipfs/go-delegated-routing (v0.6.0 -> v0.7.0):
  - Release v0.7.0
  - feat: add latency & count metrics for content routing client (#59) ([ipfs/go-delegated-routing#59](https://github.com/ipfs/go-delegated-routing/pull/59))
  - docs: add basic readme ([ipfs/go-delegated-routing#57](https://github.com/ipfs/go-delegated-routing/pull/57))
  - sync: update CI config files ([ipfs/go-delegated-routing#40](https://github.com/ipfs/go-delegated-routing/pull/40))
  - added link to reframe blog post (#54) ([ipfs/go-delegated-routing#54](https://github.com/ipfs/go-delegated-routing/pull/54))
- github.com/ipfs/go-ipfs-files (v0.1.1 -> v0.2.0):
  - Release v0.2.0
  - fix: error when TAR has files outside of root (#56) ([ipfs/go-ipfs-files#56](https://github.com/ipfs/go-ipfs-files/pull/56))
  - sync: update CI config files ([ipfs/go-ipfs-files#55](https://github.com/ipfs/go-ipfs-files/pull/55))
  - chore(Directory): add DirIterator API restriction: iterate only once
- github.com/ipfs/go-unixfs (v0.4.0 -> v0.4.1):
  - Update version.json
  - Fix: panic when childer is nil (#127) ([ipfs/go-unixfs#127](https://github.com/ipfs/go-unixfs/pull/127))
  - sync: update CI config files (#125) ([ipfs/go-unixfs#125](https://github.com/ipfs/go-unixfs/pull/125))
- github.com/ipld/go-ipld-prime (v0.18.0 -> v0.19.0):
  - Prepare v0.19.0
  - fix: correct json codec links & bytes handling
  - test(basicnode): increase test coverage for int and map types (#454) ([ipld/go-ipld-prime#454](https://github.com/ipld/go-ipld-prime/pull/454))
  - fix: remove reliance on ioutil
  - run gofmt -s
  - bump go.mod to Go 1.18 and run go fix
  - feat: add kinded union to gendemo
- github.com/libp2p/go-libp2p (v0.23.2 -> v0.23.4):
  - Release v0.23.4 (#1864) ([libp2p/go-libp2p#1864](https://github.com/libp2p/go-libp2p/pull/1864))
  - release v0.23.3
  - websocket: set the HTTP host header in WSS
- github.com/libp2p/go-netroute (v0.2.0 -> v0.2.1):
  - v0.2.1 ([libp2p/go-netroute#27](https://github.com/libp2p/go-netroute/pull/27))
  - fix(phys-addr-length): fix physical address length mismatch ([libp2p/go-netroute#29](https://github.com/libp2p/go-netroute/pull/29))
  - compare priority if route rule's dst mask is same size
  - compare priority if route rule's dst mask is same size
  - sync: update CI config files (#24) ([libp2p/go-netroute#24](https://github.com/libp2p/go-netroute/pull/24))
- github.com/marten-seemann/qpack (v0.2.1 -> v0.3.0):
  - update to Ginkgo v2 (#30) ([marten-seemann/qpack#30](https://github.com/marten-seemann/qpack/pull/30))
  - return write error when encoding header fields (#28) ([marten-seemann/qpack#28](https://github.com/marten-seemann/qpack/pull/28))
  - update Go versions (#29) ([marten-seemann/qpack#29](https://github.com/marten-seemann/qpack/pull/29))
  - remove CircleCI build status from README
  - add link to QPACK RFC to README
  - remove build constraint from fuzzer ([marten-seemann/qpack#24](https://github.com/marten-seemann/qpack/pull/24))
- github.com/multiformats/go-multicodec (v0.6.0 -> v0.7.0):
  - feat: update ./multicodec/table.csv ([multiformats/go-multicodec#71](https://github.com/multiformats/go-multicodec/pull/71))

</details>

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Antonio Navarro Perez | 11 | +780/-987 | 31 |
| Marcin Rataj | 14 | +791/-543 | 26 |
| web3-bot | 7 | +393/-427 | 71 |
| galargh | 20 | +309/-277 | 21 |
| Gus Eggert | 5 | +358/-222 | 58 |
| Henrique Dias | 3 | +409/-30 | 13 |
| Dustin Long | 1 | +314/-0 | 2 |
| Marco Munizaga | 2 | +211/-46 | 11 |
| Rod Vagg | 4 | +188/-62 | 13 |
| Jorropo | 2 | +4/-219 | 5 |
| Steve Loeppky | 1 | +115/-72 | 4 |
| Andreas Källberg | 1 | +145/-5 | 5 |
| Lucas Molas | 3 | +76/-53 | 9 |
| snyh | 2 | +36/-18 | 2 |
| Piotr Galar | 2 | +31/-4 | 2 |
| Ondrej Kokes | 1 | +25/-4 | 2 |
| Marten Seemann | 6 | +14/-14 | 14 |
| Yann Autissier | 1 | +14/-4 | 1 |
| maxos | 1 | +8/-1 | 2 |
| reidlw | 1 | +1/-4 | 1 |
| Russell Dempsey | 2 | +4/-1 | 2 |
| Ian Davis | 1 | +4/-0 | 2 |
| Daniel Norman | 1 | +3/-1 | 1 |
| Will Scott | 1 | +1/-1 | 1 |
| Nikhilesh Susarla | 1 | +2/-0 | 2 |
| Jamie Wilkinson | 1 | +1/-1 | 1 |
| Will | 1 | +0/-1 | 1 |


# Kubo changelog v0.18

## v0.18.1

This release includes improvements around Pubsub message deduplication, libp2p resource management, and more.

<!-- TOC depthfrom:3 -->

- [Overview](#overview)
- [🔦 Highlights](#-highlights)
    - [New default Pubsub.SeenMessagesStrategy](#new-default-pubsubseenmessagesstrategy)
    - [Improving libp2p resource management integration](#improving-libp2p-resource-management-integration)
- [📝 Changelog](#-changelog)
- [👨‍👩‍👧‍👦 Contributors](#-contributors)

<!-- /TOC -->

### 🔦 Highlights

#### New default `Pubsub.SeenMessagesStrategy`

A new optional [`Pubsub.SeenMessagesStrategy`](../config.md#pubsubseenmessagesstrategy) configuration option has been added.

This option allows you to choose between two different strategies for
deduplicating messages: `first-seen` and `last-seen`.

When unset, the default strategy is `last-seen`, which calculates the
time-to-live (TTL) countdown based on the last time a message is seen. This
means that if a message is received and then seen again within the specified
TTL window based on the last time it was seen, it won't be emitted.

If you prefer the old behavior, which calculates the TTL countdown based on the
first time a message is seen, you can set `Pubsub.SeenMessagesStrategy` to
`first-seen`.

#### Improving libp2p resource management integration

TL;DR: limit autoscaling improved, most users should start with default settings.
If you have old configuration, switch to implicit defaults:

```
ipfs config --json -- Swarm.ResourceMgr '{}'
ipfs config --json -- Swarm.ConnMgr '{}'
```

IF you run a server and want to utilize more than half of memory and file descriptors to p2p work, adjust [`Swarm.ResourceMgr.MaxMemory`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmresourcemgrmaxmemory) and [`Swarm.ResourceMgr.MaxFileDescriptors`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmresourcemgrmaxfiledescriptors).

The 0.18.1 builds on the default protection nodes get against DoS (resource exhaustion) and eclipse attacks
with the [go-libp2p Network Resource Manager/Accountant](https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md)
that was fine-tuned in [Kubo 0.18](https://github.com/ipfs/kubo/blob/biglep/resource-manager-example-of-what-want/docs/changelogs/v0.18.md#improving-libp2p-resource-management-integration).

Adding default hard-limits from the Resource Manager/Accountant after the fact is tricky,
and some additional improvements have been made to improve the [computed defaults](https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md#computed-default-limits).

As much as possible, the aim is for a user to only think about how much memory they want to bound libp2p to, 
and not need to think about translating that to hard numbers for connections, streams, etc.
More updates are likely in future Kubo releases, but with this release: 
1. ``System.StreamsInbound`` is no longer bounded directly
2. ``System.ConnsInbound``, ``Transient.Memory``, ``Transiet.ConnsInbound`` have higher default computed values.

### 📝 Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
  - Add overview section
  - Adjust inbound connection limits depending on memory.
  - feat: Pubsub.SeenMessagesStrategy (#9543) ([ipfs/kubo#9543](https://github.com/ipfs/kubo/pull/9543))
  - chore: update version
- github.com/libp2p/go-libp2p-pubsub (v0.8.2 -> v0.8.3):
  - feat: expire messages from the cache based on last seen time (#513) ([libp2p/go-libp2p-pubsub#513](https://github.com/libp2p/go-libp2p-pubsub/pull/513))

</details>

### 👨‍👩‍👧‍👦 Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Mohsin Zaidi | 2 | +511/-55 | 12 |
| Antonio Navarro Perez | 2 | +57/-57 | 5 |
| galargh | 1 | +1/-1 | 1 |

## v0.18.0

### Overview

Below is an outline of all that is in this release, so you get a sense of all that's included.

<!-- TOC depthfrom:3 -->

- [Overview](#overview)
- [🔦 Highlights](#-highlights)
  - [Content routing](#content-routing)
    - [Default InterPlanetary Network Indexer](#default-interplanetary-network-indexer)
    - [Increase provider record republish interval and expiration](#increase-provider-record-republish-interval-and-expiration)
  - [Gateways](#gateways)
    - [(DAG-)JSON and (DAG-)CBOR response formats](#dag-json-and-dag-cbor-response-formats)
    - [🐎 Fast directory listings with DAG sizes](#-fast-directory-listings-with-dag-sizes)
  - [QUIC and WebTransport](#quic-and-webtransport)
    - [WebTransport enabled by default](#webtransport-enabled-by-default)
    - [QUIC and WebTransport share a single port](#quic-and-webtransport-share-a-single-port)
    - [Differentiating QUIC versions](#differentiating-quic-versions)
    - [QUICv1 and WebTransport config migration](#quicv1-and-webtransport-config-migration)
  - [Improving libp2p resource management integration](#improving-libp2p-resource-management-integration)
- [📝 Changelog](#-changelog)
- [👨‍👩‍👧‍👦 Contributors](#-contributors)

<!-- /TOC -->

### 🔦 Highlights

#### Content routing

##### Default InterPlanetary Network Indexer

Content routing is the process of discovering which peers provide a piece of content. Kubo has traditionally only supported [libp2p's implementation of Kademlia DHT](https://github.com/libp2p/specs/tree/master/kad-dht) for content routing.

Kubo can now bridge networks by including support for the [delegated routing HTTP API](https://github.com/ipfs/specs/pull/337). Users can compose content routers using the `Routing.Routers` config to pick content routers with different tradeoffs than a Kademlia DHT (e.g., high-performance and high-capacity centralized endpoints, dedicated Kademlia DHT nodes, routers with unique provider records, privacy-focused content routers).

One example is [InterPlanetary Network Indexers](https://github.com/ipni/specs/blob/main/IPNI.md#readme), which are HTTP endpoints that cache records from both the IPFS network and other sources such as web3.storage and Filecoin. This improves not only content availability by enabling Kubo to transparently fetch content directly from Filecoin storage providers, but also improves IPFS content routing latency by an order of magnitude and decreases resource consumption.

> *Note:* it's possible to retrieve content stored by Filecoin Storage Providers (SPs) from Kubo if the SPs service Bitswap requests.  As of this release, some SPs are advertising Bitswap.  You can follow the roadmap progress for IPNIs and Bitswap in SPs [here](https://www.starmaps.app/roadmap/github.com/protocol/bedrock/issues/1).

In this release, the default content router is changed from `dht` to `auto`. The `auto` router includes the IPFS DHT in addition to the [cid.contact](https://cid.contact) IPNI instance. In future releases, we plan to expand the functionality of `auto` to encompass automatic discovery of content routers, which will improve performance and content availability (for example, see [IPIP-342](https://github.com/ipfs/specs/pull/342)).

Previous behavior can be restored by setting `Routing.Type` to `dht`.

Alternative routing rules, including alternative IPNI endpoints, can be configured in `Routing.Routers` after setting `Routing.Type` to `custom`.

Learn more in the [`Routing` docs](https://github.com/ipfs/kubo/blob/master/docs/config.md#routing).

##### Increase provider record republish interval and expiration

Default `Reprovider.Interval` changed from 12h to 22h to match new defaults for the Provider Record Expiration (48h) in [go-libp2p-kad-dht v0.20.0](https://github.com/libp2p/go-libp2p-kad-dht/releases/tag/v0.20.0).

The rationale for increasing this can be found in
[RFM 17: Provider Record Liveness Report](https://github.com/protocol/network-measurements/blob/master/results/rfm17-provider-record-liveness.md),
[kubo#9326](https://github.com/ipfs/kubo/pull/9326),
and the upstream DHT specifications at [libp2p/specs#451](https://github.com/libp2p/specs/pull/451).

Learn more in the [`Reprovider` config](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#reprovider).

#### Gateways

##### (DAG-)JSON and (DAG-)CBOR response formats

The IPFS project has reserved the corresponding media types at IANA:
- [`application/vnd.ipld.dag-json`](https://www.iana.org/assignments/media-types/application/vnd.ipld.dag-json)
- [`application/vnd.ipld.dag-cbor`](https://www.iana.org/assignments/media-types/application/vnd.ipld.dag-cbor)

This release implements them as part of [IPIP-328](https://github.com/ipfs/specs/pull/328)
and adds Gateway support for CIDs with `json` (0x0200), `cbor` (0x51),
[`dag-json`](https://ipld.io/specs/codecs/dag-json/) (0x0129)
and [`dag-cbor`](https://ipld.io/specs/codecs/dag-cbor/spec/) (0x71) codecs.

To specify the response `Content-Type` explicitly, the HTTP client can override
the codec present in the CID by using the `format` parameter
or setting the `Accept` HTTP header:

- Plain JSON: `?format=json` or `Accept: application/json`
- Plain CBOR: `?format=cbor` or `Accept: application/cbor`
- DAG-JSON: `?format=dag-json` or `Accept: application/vnd.ipld.dag-json`
- DAG-CBOR: `?format=dag-cbor` or `Accept: application/vnd.ipld.dag-cbor`

In addition, when DAG-JSON or DAG-CBOR is requested with the `Accept` header
set to `text/html`, the Gateway will return a basic HTML page with download
options, improving the user experience in web browsers.

###### Example 1: DAG-CBOR and DAG-JSON Conversion on Gateway

The Gateway supports conversion between DAG-CBOR and DAG-JSON for efficient
end-to-end data structure management: author in CBOR or JSON, store as binary
CBOR and retrieve as JSON via HTTP:

```console
$ echo '{"test": "json"}' | ipfs dag put # implicit --input-codec dag-json --store-codec dag-cbor
bafyreico7mjtqtqhvawro3yud5uqn6sc33nzqb7b5j2d7pdmzer5nab4t4

$ ipfs block get bafyreico7mjtqtqhvawro3yud5uqn6sc33nzqb7b5j2d7pdmzer5nab4t4 | xxd
00000000: a164 7465 7374 646a 736f 6e              .dtestdjson

$ ipfs dag get bafyreico7mjtqtqhvawro3yud5uqn6sc33nzqb7b5j2d7pdmzer5nab4t4 # implicit --output-codec dag-json
{"test":"json"}

$ curl "http://127.0.0.1:8080/ipfs/bafyreico7mjtqtqhvawro3yud5uqn6sc33nzqb7b5j2d7pdmzer5nab4t4?format=dag-json"
{"test":"json"}
```

###### Example 2: Traversing CBOR DAGs

Placing a CID in [CBOR Tag 42](https://github.com/ipld/cid-cbor/) enables the
creation of arbitrary DAGs. The equivalent DAG-JSON notation for linking
to different blocks is represented by `{ "/": "cid" }`.

The Gateway supports traversing these links, enabling access to data
referenced by structures other than regular UnixFS directories:

```console
$ echo '{"test.jpg": {"/": "bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi"}}' | ipfs dag put
bafyreihspwy3zlkzgphmec5d3xb5g5njrqwotd46lyubnelbzktnmsxkq4 # dag-cbor document linking to unixfs file

$ ipfs resolve /ipfs/bafyreihspwy3zlkzgphmec5d3xb5g5njrqwotd46lyubnelbzktnmsxkq4/test.jpg
/ipfs/bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi

$ ipfs dag stat bafyreihspwy3zlkzgphmec5d3xb5g5njrqwotd46lyubnelbzktnmsxkq4
Size: 119827, NumBlocks: 2

$ curl "http://127.0.0.1:8080/ipfs/bafyreihspwy3zlkzgphmec5d3xb5g5njrqwotd46lyubnelbzktnmsxkq4/test.jpg" > test.jpg
```

###### Example 3: UnixFS directory listing as JSON

Finally, Gateway now supports the same [logical format projection](https://ipld.io/specs/codecs/dag-pb/spec/#logical-format) from
DAG-PB to DAG-JSON as the `ipfs dag get` command, enabling the retrieval of directory listings as JSON instead of HTML:

```console
$ export DIR_CID=bafybeigccimv3zqm5g4jt363faybagywkvqbrismoquogimy7kvz2sj7sq
$ curl -H "Accept: application/vnd.ipld.dag-json" "http://127.0.0.1:8080/ipfs/$DIR_CID" | jq
$ curl "http://127.0.0.1:8080/ipfs/$DIR_CID?format=dag-json" | jq
{
  "Data": {
    "/": {
      "bytes": "CAE"
    }
  },
  "Links": [
    {
      "Hash": {
        "/": "Qmc3zqKcwzbbvw3MQm3hXdg8BQoFjGdZiGdAfXAyAGGdLi"
      },
      "Name": "1 - Barrel - Part 1 - alt.txt",
      "Tsize": 21
    },
    {
      "Hash": {
        "/": "QmdMxMx29KVYhHnaCc1icWYxQqXwUNCae6t1wS2NqruiHd"
      },
      "Name": "1 - Barrel - Part 1 - transcript.txt",
      "Tsize": 195
    },
    {
      "Hash": {
        "/": "QmawceGscqN4o8Y8Fv26UUmB454kn2bnkXV5tEQYc4jBd6"
      },
      "Name": "1 - Barrel - Part 1.png",
      "Tsize": 24862
    }
  ]
}
$ ipfs dag get $DIR_CID
{"Data":{"/":{"bytes":"CAE"}},"Links":[{"Hash":{"/":"Qmc3zqKcwzbbvw3MQm3hXdg8BQoFjGdZiGdAfXAyAGGdLi"},"Name":"1 - Barrel - Part 1 - alt.txt","Tsize":21},{"Hash":{"/":"QmdMxMx29KVYhHnaCc1icWYxQqXwUNCae6t1wS2NqruiHd"},"Name":"1 - Barrel - Part 1 - transcript.txt","Tsize":195},{"Hash":{"/":"QmawceGscqN4o8Y8Fv26UUmB454kn2bnkXV5tEQYc4jBd6"},"Name":"1 - Barrel - Part 1.png","Tsize":24862}]}
```

##### 🐎 Fast directory listings with DAG sizes

Fast listings are now enabled for _all_ UnixFS directories: big and small.
There is no linear slowdown caused by reading size metadata from child nodes,
and the size of DAG representing child items is always present.

As an example, the CID
`bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm` represents a UnixFS
directory with over 10k files. Listing big directories was fast
since Kubo 0.13, but in this release it will also include the size column.

#### QUIC and WebTransport

##### WebTransport enabled by default
[WebTransport](https://docs.libp2p.io/concepts/transports/webtransport/) is a new libp2p transport that [was introduced in v0.16](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.16.md#-webtransport-new-experimental-transport) that is based on top of QUIC and HTTP3.

This allows browser-based nodes to contact Kubo nodes, so now instead of just serving requests for other system-level application nodes, you can also serve requests directly to a node running inside a browser page.

For the full story see [connectivity.libp2p.io](https://connectivity.libp2p.io/).

##### QUIC and WebTransport share a single port
WebTransport is enabled by default in part because [go-libp2p now supports running WebTransport and QUIC transports on the same QUIC listener](https://github.com/libp2p/go-libp2p/issues/1759).  No additional port needs to be opened.

To use this feature, register two listen addresses on the same `/ipX/.../udp/XXX` prefix.

##### Differentiating QUIC versions
go-libp2p now differentiates the first version of QUIC that was originally implemented, `Draft-29`, from the ratified protocol in [RFC9000](https://www.rfc-editor.org/rfc/rfc9000.html), `QUICv1`.
This was done for performance (time to first byte) reasons as [outlined here](https://github.com/multiformats/multiaddr/issues/145).

This manifests as two different multiaddr components `/quic` (old Draft-29) and `/quic-v1`.
go-libp2p do supports listening with both QUIC versions on one single listener.
WebTransport has only supported QUICv1.
`/webtransport` now needs to be prefixed by a `/quic-v1` component instead of a `/quic` component.

Support for QUIC Draft-29 will be removed at some point in 2023 ([tracking issue](https://github.com/ipfs/kubo/issues/9496)).  As a result, new deployments should use `/quic-v1` instead of `/quic`.

##### QUICv1 and WebTransport config migration
To support QUICv1 and WebTransport by default a new config migration (`v13`) is run which automatically adds entries in addresses-related fields:
- Replace all `/quic/webtransport` to `/quic-v1/webtransport`.
- For all `/quic` listeners, keep the Draft-29 listener, and on the same ip and port, add `/quic-v1` and `/quic-v1/webtransport` listeners.

#### Improving libp2p resource management integration
To help protect nodes from DoS (resource exhaustion) and eclipse attacks,
Kubo enabled the [go-libp2p Network Resource Manager](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager)
by default in [Kubo 0.17](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.17.md#libp2p-resource-management-enabled-by-default).

Introducing limits like this by default after the fact is tricky,
and various improvements have been made to improve the UX including:
1. [Dedicated docs concerning the resource manager integration](https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md).  This is a great place to go to learn more or get your FAQs answered.
2. Increasing the default limits for the resource manager.
3. Enabling the [`Swarm.ConnMgr`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmconnmgr) by default and reducing it thresholds so it can intelligently prune connections in many cases before the indiscriminate resource manager kicks in.
4. Adjusted log messages and levels to make clear that the resource manager is likely doing your node a favor by bounding resources.
5. [Other miscellaneous config and command bugs reported by users](https://github.com/ipfs/kubo/issues/9442).

### 📝 Changelog

<details><summary>Full Changelog</summary>

- github.com/ipfs/kubo:
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
  - chore: update version.go
  - fix(test): stabilize flaky provider tests
  - feat: port pins CLI test
  - Removing QRI from early tester ([ipfs/kubo#9503](https://github.com/ipfs/kubo/pull/9503))
  - fix: disable provide over HTTP with Routing.Type=auto (#9511) ([ipfs/kubo#9511](https://github.com/ipfs/kubo/pull/9511))
  - Update version.go
  - 'chore: update version.go'
  - Clened up 0.18 changelog for release ([ipfs/kubo#9497](https://github.com/ipfs/kubo/pull/9497))
  - feat: turn on WebTransport by default ([ipfs/kubo#9492](https://github.com/ipfs/kubo/pull/9492))
  - feat: fast directory listings with DAG Size column (#9481) ([ipfs/kubo#9481](https://github.com/ipfs/kubo/pull/9481))
  - feat: add basic CLI tests using Go Test
  - Changelog: v0.18.0 ([ipfs/kubo#9485](https://github.com/ipfs/kubo/pull/9485))
  - feat: go-libp2p-kad-dht with expiration 48h
  - chore: update go-libp2p to v0.24.1
  - fix: remove the imports work-around
  - fix: replace quic to quic-v1 for webtransport sharness
  - fix: silence staticcheck warning for fx.Extract usage
  - update go-libp2p to v0.24.0
  - stop using the deprecated go-libp2p-loggables package
  - docs(readme): update package managers section (#9488) ([ipfs/kubo#9488](https://github.com/ipfs/kubo/pull/9488))
  - fix: support /quic-v1 in webui v0.21
  - feat: Routing.Type=auto (DHT+IPNI) (#9475) ([ipfs/kubo#9475](https://github.com/ipfs/kubo/pull/9475))
  - feat: adjust ConnMgr target to 32-96 (#9483) ([ipfs/kubo#9483](https://github.com/ipfs/kubo/pull/9483))
  - feat: increase default Reprovider.Interval (#9326) ([ipfs/kubo#9326](https://github.com/ipfs/kubo/pull/9326))
  - feat: add response body limiter to routing HTTP client (#9478) ([ipfs/kubo#9478](https://github.com/ipfs/kubo/pull/9478))
  - docs: libp2p resource management (#9468) ([ipfs/kubo#9468](https://github.com/ipfs/kubo/pull/9468))
  - chore: upgrade libipfs for routing HTTP API schema changes (#9477) ([ipfs/kubo#9477](https://github.com/ipfs/kubo/pull/9477))
  - feat: lower connection pool
  - Add missing &&
  - Fix sharness test
  - Added a message when RM is disabled.
  - Requested changes.
  - Fix sharness checking daemon output
  - Update test/sharness/t0060-daemon.sh
  - Try to fix sharness test.
  - Fix: RM: Improve init RM message and fix final memory value.
  - Fix: Resource Manager: Filter stats correctly by %
  - Apply suggestions from code review
  - Increase MaxMemory param to use half of total memory.
  - Update libipfs dependency.
  - Add sharness tests and documentation
  - Fix variable name
  - feature: delegated-routing: Add HTTP delegated routing.
  - Fix: Change RM log output to WARN level
  - Fix: RM: Set no-limit value to 1e9 (1000000000).
  - Partial Revert "Revert "fix: ensure hasher is registered when using a hashing function""
  - Add logs to the routing system
  - fix: apply agent-version-suffix to libp2p identify
  - chore: migrate ipfs/tar-utils to libipfs
  - feat(gateway): JSON and CBOR response formats (IPIP-328) (#9335) ([ipfs/kubo#9335](https://github.com/ipfs/kubo/pull/9335))
  -  ([ipfs/kubo#9318](https://github.com/ipfs/kubo/pull/9318))
  - docs: release process updates from v0.17.0 ([ipfs/kubo#9391](https://github.com/ipfs/kubo/pull/9391))
  - fix(rcmgr): improve error phrasing
  - docs: Update CHANGELOG.md adding 0.17 link
  - feat(config): Pubsub.SeenMessagesTTL (#9372) ([ipfs/kubo#9372](https://github.com/ipfs/kubo/pull/9372))
  - docs: remove snap and chocolatey packages
  - Merge release v0.17.0 ([ipfs/kubo#9431](https://github.com/ipfs/kubo/pull/9431))
  - docs: ipfs-http-client -> kubo-rpc-client (#9331) ([ipfs/kubo#9331](https://github.com/ipfs/kubo/pull/9331))
  - docs(readme): improve tldr
  - Update config.md for resource management limits (#9421) ([ipfs/kubo#9421](https://github.com/ipfs/kubo/pull/9421))
  - Doc improvements and changelog for resource manager (#9413) ([ipfs/kubo#9413](https://github.com/ipfs/kubo/pull/9413))
  - Revert "Doc improvements for rcmgr"
  - Doc improvements for rcmgr
  - docs: document /wss fixes in 0.17
  - refactor(config): remove Swarm.ConnMgr defaults
  - fix(config): skip nulls in ResourceMgr
  - docs: replace tabcat with aphelionz in EARLY_TESTERS.md (#9404) ([ipfs/kubo#9404](https://github.com/ipfs/kubo/pull/9404))
  - docs: fix spoiler for 0.13.1 changelog
  - fix(docs): typo
  - chore: bump version to v0.18.0-dev ([ipfs/kubo#9393](https://github.com/ipfs/kubo/pull/9393))
- github.com/ipfs/go-bitswap (v0.10.2 -> v0.11.0):
  - chore: release v0.11.0
- github.com/ipfs/go-blockservice (v0.4.0 -> v0.5.0):
  - chore: release v0.5.0
- github.com/ipfs/go-graphsync (v0.13.1 -> v0.14.1):
  - chore: version 0.14.1 (#400) ([ipfs/go-graphsync#400](https://github.com/ipfs/go-graphsync/pull/400))
  - chore: migrate files (#399) ([ipfs/go-graphsync#399](https://github.com/ipfs/go-graphsync/pull/399))
  - docs(CHANGELOG): update for v0.14.0 release
  - updates for libp2p v0.22 (#392) ([ipfs/go-graphsync#392](https://github.com/ipfs/go-graphsync/pull/392))
  - feat(ipld): use bindnode/registry (#386) ([ipfs/go-graphsync#386](https://github.com/ipfs/go-graphsync/pull/386))
  - Accept/Reject requests up front (#384) ([ipfs/go-graphsync#384](https://github.com/ipfs/go-graphsync/pull/384))
  - Remove protobuf protocol (#385) ([ipfs/go-graphsync#385](https://github.com/ipfs/go-graphsync/pull/385))
  - docs(CHANGELOG): update for v0.13.2
  - chore(deps): upgrade libp2p & ipld-prime (#389) ([ipfs/go-graphsync#389](https://github.com/ipfs/go-graphsync/pull/389))
  - chore(ipld): switch to using top-level ipld-prime codec helpers (#383) ([ipfs/go-graphsync#383](https://github.com/ipfs/go-graphsync/pull/383))
  - feat(requestmanager): read request from context (#381) ([ipfs/go-graphsync#381](https://github.com/ipfs/go-graphsync/pull/381))
  - fix: minor typo in error msg
  - fix(panics): lift panic recovery up to top of network handling
  - feat: expand use of panic handler to cover network and codec interaction
  - feat(panics): capture panics from selector execution
- github.com/ipfs/go-ipfs-cmds (v0.8.1 -> v0.8.2):
  - chore: version v0.8.2 (#235) ([ipfs/go-ipfs-cmds#235](https://github.com/ipfs/go-ipfs-cmds/pull/235))
  - chore: migrate files (#233) ([ipfs/go-ipfs-cmds#233](https://github.com/ipfs/go-ipfs-cmds/pull/233))
  - sync: update CI config files (#229) ([ipfs/go-ipfs-cmds#229](https://github.com/ipfs/go-ipfs-cmds/pull/229))
- github.com/ipfs/go-ipfs-keystore (v0.0.2 -> v0.1.0):
  - chore: release v0.1.0
  - chore: update go-libp2p
  - sync: update CI config files ([ipfs/go-ipfs-keystore#10](https://github.com/ipfs/go-ipfs-keystore/pull/10))
  - sync: update CI config files ([ipfs/go-ipfs-keystore#8](https://github.com/ipfs/go-ipfs-keystore/pull/8))
  - Add link to pkg.go.dev
  - README: this module does not use Gx
- github.com/ipfs/go-ipfs-provider (v0.7.1 -> v0.8.1):
  - chore: release v0.8.1
  - fix: make queue 64bits on 32bits platforms too
  - sync: update CI config files ([ipfs/go-ipfs-provider#36](https://github.com/ipfs/go-ipfs-provider/pull/36))
  - chore: update go-lib2p, avoid depending on go-libp2p-core, bump go.mod version
- github.com/ipfs/go-ipfs-routing (v0.2.1 -> v0.3.0):
  - release v0.3.0 ([ipfs/go-ipfs-routing#36](https://github.com/ipfs/go-ipfs-routing/pull/36))
  - chore: update go-libp2p to v0.22.0 ([ipfs/go-ipfs-routing#35](https://github.com/ipfs/go-ipfs-routing/pull/35))
  - sync: update CI config files (#34) ([ipfs/go-ipfs-routing#34](https://github.com/ipfs/go-ipfs-routing/pull/34))
- github.com/ipfs/go-ipld-cbor (v0.0.5 -> v0.0.6):
  - Add contexts to IpldBlockstore ([ipfs/go-ipld-cbor#82](https://github.com/ipfs/go-ipld-cbor/pull/82))
  - sync: update CI config files (#81) ([ipfs/go-ipld-cbor#81](https://github.com/ipfs/go-ipld-cbor/pull/81))
  - sync: update CI config files ([ipfs/go-ipld-cbor#79](https://github.com/ipfs/go-ipld-cbor/pull/79))
  - Fix lint errors ([ipfs/go-ipld-cbor#78](https://github.com/ipfs/go-ipld-cbor/pull/78))
  - Add notice directing new projects to codec/dagcbor ([ipfs/go-ipld-cbor#77](https://github.com/ipfs/go-ipld-cbor/pull/77))
- github.com/ipfs/go-merkledag (v0.6.0 -> v0.9.0):
  - chore: bump version to 0.9.0
  - chore: bump version to 0.8.1
  - feat: remove panic() from non-error methods
  - feat: improve broken cid.Builder testing for CidBuilder
  - chore: bump version to 0.8.0
  - doc: document potential panics and how to avoid them
  - fix: simplify Cid generation cache & usage
  - feat: check links on setting and sanitise on encoding
  - feat: check that the CidBuilder hasher is usable
  - chore: bump version to 0.7.0
  - fix: remove use of ioutil
  - run gofmt -s
  - fix!: keep deserialised state stable until explicit mutation
  - sync: update CI config files ([ipfs/go-merkledag#84](https://github.com/ipfs/go-merkledag/pull/84))
- github.com/ipfs/go-namesys (v0.5.0 -> v0.6.0):
  - chore: release v0.6.0
  - chore: update go-libp2p to v0.23.4, update go.mod version to 1.18
  - stop using the deprecated io/ioutil package
- github.com/ipfs/go-peertaskqueue (v0.7.1 -> v0.8.0):
  - Release v0.8.0 ([ipfs/go-peertaskqueue#25](https://github.com/ipfs/go-peertaskqueue/pull/25))
  - chore: update go-libp2p to v0.22.0 ([ipfs/go-peertaskqueue#24](https://github.com/ipfs/go-peertaskqueue/pull/24))
  - sync: update CI config files (#23) ([ipfs/go-peertaskqueue#23](https://github.com/ipfs/go-peertaskqueue/pull/23))
  - sync: update CI config files (#21) ([ipfs/go-peertaskqueue#21](https://github.com/ipfs/go-peertaskqueue/pull/21))
- github.com/ipfs/go-unixfs (v0.4.1 -> v0.4.2):
  - chore: version 0.4.2 (#136) ([ipfs/go-unixfs#136](https://github.com/ipfs/go-unixfs/pull/136))
  - chore: migrate files (#134) ([ipfs/go-unixfs#134](https://github.com/ipfs/go-unixfs/pull/134))
  -  ([ipfs/go-unixfs#128](https://github.com/ipfs/go-unixfs/pull/128))
- github.com/ipfs/go-unixfsnode (v1.4.0 -> v1.5.1):
  - v1.5.1
  - fix a possible `index out of range` crash ([ipfs/go-unixfsnode#39](https://github.com/ipfs/go-unixfsnode/pull/39))
  - add an ADL to preload hamt loading ([ipfs/go-unixfsnode#38](https://github.com/ipfs/go-unixfsnode/pull/38))
  - chore: bump version to 1.5.0
  - fix: remove use of ioutil
  - run gofmt -s
  - bump go.mod to Go 1.18 and run go fix
  - test for reader / sizing behavior on large files ([ipfs/go-unixfsnode#34](https://github.com/ipfs/go-unixfsnode/pull/34))
  - add helper to approximate test creation patter from ipfs-files ([ipfs/go-unixfsnode#32](https://github.com/ipfs/go-unixfsnode/pull/32))
  - chore: remove Stebalien/go-bitfield in favour of ipfs/go-bitfield
- github.com/ipfs/interface-go-ipfs-core (v0.7.0 -> v0.8.2):
  - chore: version 0.8.2 (#100) ([ipfs/interface-go-ipfs-core#100](https://github.com/ipfs/interface-go-ipfs-core/pull/100))
  - chore: migrate files (#97) ([ipfs/interface-go-ipfs-core#97](https://github.com/ipfs/interface-go-ipfs-core/pull/97))
  - chore: release v0.8.1
  - feat: add UseCumulativeSize UnixfsLs option (#95) ([ipfs/interface-go-ipfs-core#95](https://github.com/ipfs/interface-go-ipfs-core/pull/95))
  - chore: release v0.8.0
  - chore: update go-libp2p to v0.23.4
  - sync: update CI config files (#87) ([ipfs/interface-go-ipfs-core#87](https://github.com/ipfs/interface-go-ipfs-core/pull/87))
- github.com/ipld/go-car/v2 (v2.4.0 -> v2.5.1):
  - add a `SkipNext` method on block reader (#338) ([ipld/go-car#338](https://github.com/ipld/go-car/pull/338))
  - feat: Has() and Get() will respect StoreIdentityCIDs option
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
- github.com/ipld/go-codec-dagpb (v1.4.1 -> v1.5.0):
  - chore: version bump to 1.5.0
  - fix: replace io/ioutil with io
  - bump go.mod to Go 1.18 and run go fix
  - v1.4.2 bump
- github.com/libp2p/go-libp2p (v0.23.4 -> v0.24.2):
  - release v0.24.2 (#1969) ([libp2p/go-libp2p#1969](https://github.com/libp2p/go-libp2p/pull/1969))
  - webtransport: initialize a NullResourceManager if none is provided (#1962) ([libp2p/go-libp2p#1962](https://github.com/libp2p/go-libp2p/pull/1962))
  - release v0.24.1 (#1945) ([libp2p/go-libp2p#1945](https://github.com/libp2p/go-libp2p/pull/1945))
  - routed host: return Connect error if FindPeer doesn't yield new addresses (#1946) ([libp2p/go-libp2p#1946](https://github.com/libp2p/go-libp2p/pull/1946))
  - chore: update examples to v0.24.0 (#1936) ([libp2p/go-libp2p#1936](https://github.com/libp2p/go-libp2p/pull/1936))
  - webtransport: fix flaky accept queue test (#1938) ([libp2p/go-libp2p#1938](https://github.com/libp2p/go-libp2p/pull/1938))
  - quic: fix race condition in TestClientCanDialDifferentQUICVersions (#1937) ([libp2p/go-libp2p#1937](https://github.com/libp2p/go-libp2p/pull/1937))
  - quic: update quic-go to v0.31.1 (#1942) ([libp2p/go-libp2p#1942](https://github.com/libp2p/go-libp2p/pull/1942))
  - release v0.24.0 (#1934) ([libp2p/go-libp2p#1934](https://github.com/libp2p/go-libp2p/pull/1934))
  - Disable support for signed/static TLS certificates in WebTransport (#1927) ([libp2p/go-libp2p#1927](https://github.com/libp2p/go-libp2p/pull/1927))
  - webtransport: add PSK to constructor, and fail if it is used (#1929) ([libp2p/go-libp2p#1929](https://github.com/libp2p/go-libp2p/pull/1929))
  - use a different set of default transports when PSK is enabled (#1921) ([libp2p/go-libp2p#1921](https://github.com/libp2p/go-libp2p/pull/1921))
  - transport.Listener,quic: Support multiple QUIC versions with the same Listener. Only return a single multiaddr per listener. (#1923) ([libp2p/go-libp2p#1923](https://github.com/libp2p/go-libp2p/pull/1923))
  - quic / webtransport: make it possible to listen on the same address / port (#1905) ([libp2p/go-libp2p#1905](https://github.com/libp2p/go-libp2p/pull/1905))
  - autorelay: fix flaky TestReconnectToStaticRelays (#1903) ([libp2p/go-libp2p#1903](https://github.com/libp2p/go-libp2p/pull/1903))
  - swarm / rcmgr: synchronize the concurrent outbound dials with limits (#1898) ([libp2p/go-libp2p#1898](https://github.com/libp2p/go-libp2p/pull/1898))
  - add QUIC v1 addresses to the default listen addresses (#1914) ([libp2p/go-libp2p#1914](https://github.com/libp2p/go-libp2p/pull/1914))
  - webtransport: update webtransport-go to v0.3.0 (#1895) ([libp2p/go-libp2p#1895](https://github.com/libp2p/go-libp2p/pull/1895))
  - tls: fix flaky TestHandshakeConnectionCancellations test (#1896) ([libp2p/go-libp2p#1896](https://github.com/libp2p/go-libp2p/pull/1896))
  - holepunch: disable the resource manager in tests (#1897) ([libp2p/go-libp2p#1897](https://github.com/libp2p/go-libp2p/pull/1897))
  - transports: expose the name of the transport in the ConnectionState (#1911) ([libp2p/go-libp2p#1911](https://github.com/libp2p/go-libp2p/pull/1911))
  - respect the user's security protocol preference order ([libp2p/go-libp2p#1912](https://github.com/libp2p/go-libp2p/pull/1912))
  - circuitv2: disable the resource manager in tests (#1899) ([libp2p/go-libp2p#1899](https://github.com/libp2p/go-libp2p/pull/1899))
  - expose the security protocol on the ConnectionState ([libp2p/go-libp2p#1907](https://github.com/libp2p/go-libp2p/pull/1907))
  - fix: autorelay: treat static relays as just another peer source (#1875) ([libp2p/go-libp2p#1875](https://github.com/libp2p/go-libp2p/pull/1875))
  - feat: quic,webtransport: enable both quic-draft29 and quic-v1 addrs on quic. only quic-v1 on webtransport (#1881) ([libp2p/go-libp2p#1881](https://github.com/libp2p/go-libp2p/pull/1881))
  - holepunch: add multiaddress filter (#1839) ([libp2p/go-libp2p#1839](https://github.com/libp2p/go-libp2p/pull/1839))
  - README: remove broken links from table of contents (#1893) ([libp2p/go-libp2p#1893](https://github.com/libp2p/go-libp2p/pull/1893))
  - quic: update quic-go to v0.31.0 (#1882) ([libp2p/go-libp2p#1882](https://github.com/libp2p/go-libp2p/pull/1882))
  - add an integration test for muxer selection ([libp2p/go-libp2p#1887](https://github.com/libp2p/go-libp2p/pull/1887))
  - core/network: fix typo
  - tls / noise: prefer the client's muxer preferences ([libp2p/go-libp2p#1888](https://github.com/libp2p/go-libp2p/pull/1888))
  - upgrader: absorb the muxer_multistream.Transport into the upgrader (#1885) ([libp2p/go-libp2p#1885](https://github.com/libp2p/go-libp2p/pull/1885))
  - Apply service peer default (#1878) ([libp2p/go-libp2p#1878](https://github.com/libp2p/go-libp2p/pull/1878))
  - webtransport: use deterministic TLS certificates (#1833) ([libp2p/go-libp2p#1833](https://github.com/libp2p/go-libp2p/pull/1833))
  - remove deprecated StaticRelays option (#1868) ([libp2p/go-libp2p#1868](https://github.com/libp2p/go-libp2p/pull/1868))
  - autorelay: remove the default static relay option (#1867) ([libp2p/go-libp2p#1867](https://github.com/libp2p/go-libp2p/pull/1867))
  - core/protocol: remove deprecated Negotiator.NegotiateLazy (#1869) ([libp2p/go-libp2p#1869](https://github.com/libp2p/go-libp2p/pull/1869))
  - config: use fx dependency injection to construct transports ([libp2p/go-libp2p#1858](https://github.com/libp2p/go-libp2p/pull/1858))
  - noise: add an option to allow unknown peer ID in SecureOutbound  (#1823) ([libp2p/go-libp2p#1823](https://github.com/libp2p/go-libp2p/pull/1823))
  - Add some guard rails and docs (#1863) ([libp2p/go-libp2p#1863](https://github.com/libp2p/go-libp2p/pull/1863))
  - Fix concurrent map access in connmgr (#1860) ([libp2p/go-libp2p#1860](https://github.com/libp2p/go-libp2p/pull/1860))
  - fix: return filtered addrs (#1855) ([libp2p/go-libp2p#1855](https://github.com/libp2p/go-libp2p/pull/1855))
  - chore: preallocate slices (#1842) ([libp2p/go-libp2p#1842](https://github.com/libp2p/go-libp2p/pull/1842))
  - Close ping stream when we exit the loop (#1853) ([libp2p/go-libp2p#1853](https://github.com/libp2p/go-libp2p/pull/1853))
  - tls: don't set the deprecated tls.Config.PreferServerCipherSuites field (#1845) ([libp2p/go-libp2p#1845](https://github.com/libp2p/go-libp2p/pull/1845))
  - routed host: search for new multi addresses upon connect failure (#1835) ([libp2p/go-libp2p#1835](https://github.com/libp2p/go-libp2p/pull/1835))
  - core/peerstore: removed unused provider addr ttl constant (#1848) ([libp2p/go-libp2p#1848](https://github.com/libp2p/go-libp2p/pull/1848))
  - basichost: improve protocol negotiation debug message (#1846) ([libp2p/go-libp2p#1846](https://github.com/libp2p/go-libp2p/pull/1846))
  - noise: use Noise Extension to negotiate the muxer during the handshake (#1813) ([libp2p/go-libp2p#1813](https://github.com/libp2p/go-libp2p/pull/1813))
  - webtransport: use the rcmgr to control flow control window increases ([libp2p/go-libp2p#1832](https://github.com/libp2p/go-libp2p/pull/1832))
  - chore: update quic-go to v0.30.0 (#1838) ([libp2p/go-libp2p#1838](https://github.com/libp2p/go-libp2p/pull/1838))
  - roadmap: reorder priority, reorganize sections (#1831) ([libp2p/go-libp2p#1831](https://github.com/libp2p/go-libp2p/pull/1831))
  - websocket: set the HTTP host header in WSS(#1834) ([libp2p/go-libp2p#1834](https://github.com/libp2p/go-libp2p/pull/1834))
  - webtransport: make it possible to record qlogs (controlled by QLOGDIR env) ([libp2p/go-libp2p#1828](https://github.com/libp2p/go-libp2p/pull/1828))
  - ipfs /api/v0/id is post (#1819) ([libp2p/go-libp2p#1819](https://github.com/libp2p/go-libp2p/pull/1819))
  - examples: connect to all peers in example mdns chat app (#1798) ([libp2p/go-libp2p#1798](https://github.com/libp2p/go-libp2p/pull/1798))
  - roadmap: fix header level on "Mid Q4" (#1818) ([libp2p/go-libp2p#1818](https://github.com/libp2p/go-libp2p/pull/1818))
  - examples: use circuitv2 in relay example (#1795) ([libp2p/go-libp2p#1795](https://github.com/libp2p/go-libp2p/pull/1795))
  - add a roadmap for the next 6 months (#1784) ([libp2p/go-libp2p#1784](https://github.com/libp2p/go-libp2p/pull/1784))
  - tls: use ALPN to negotiate the stream multiplexer (#1772) ([libp2p/go-libp2p#1772](https://github.com/libp2p/go-libp2p/pull/1772))
  - tls: add tests for test vector from the spec (#1788) ([libp2p/go-libp2p#1788](https://github.com/libp2p/go-libp2p/pull/1788))
  - examples: update go-libp2p to v0.23.x (#1803) ([libp2p/go-libp2p#1803](https://github.com/libp2p/go-libp2p/pull/1803))
  - Try increasing timeouts if we're in CI for this test (#1796) ([libp2p/go-libp2p#1796](https://github.com/libp2p/go-libp2p/pull/1796))
  - Don't use rcmgr in this test (#1799) ([libp2p/go-libp2p#1799](https://github.com/libp2p/go-libp2p/pull/1799))
  - Bump timeout in CI for flaky test (#1800) ([libp2p/go-libp2p#1800](https://github.com/libp2p/go-libp2p/pull/1800))
  - Bump timeout in CI for flaky test (#1801) ([libp2p/go-libp2p#1801](https://github.com/libp2p/go-libp2p/pull/1801))
  - Fix comment in webtransport client auth handshake (#1793) ([libp2p/go-libp2p#1793](https://github.com/libp2p/go-libp2p/pull/1793))
  - examples: add basic pubsub-with-rendezvous example (#1738) ([libp2p/go-libp2p#1738](https://github.com/libp2p/go-libp2p/pull/1738))
  - quic: speed up the stateless reset test case (#1778) ([libp2p/go-libp2p#1778](https://github.com/libp2p/go-libp2p/pull/1778))
  - tls: fix flaky handshake cancellation test (#1779) ([libp2p/go-libp2p#1779](https://github.com/libp2p/go-libp2p/pull/1779))
- github.com/libp2p/go-libp2p-gostream (v0.3.0 -> v0.5.0):
  - release v0.5.0 (#74) ([libp2p/go-libp2p-gostream#74](https://github.com/libp2p/go-libp2p-gostream/pull/74))
  - update go-libp2p to v0.22.0 (#73) ([libp2p/go-libp2p-gostream#73](https://github.com/libp2p/go-libp2p-gostream/pull/73))
  - Expose some read-only methods on the underlying Stream interface (#67) ([libp2p/go-libp2p-gostream#67](https://github.com/libp2p/go-libp2p-gostream/pull/67))
  - Update libp2p ([libp2p/go-libp2p-gostream#69](https://github.com/libp2p/go-libp2p-gostream/pull/69))
  - sync: update CI config files (#65) ([libp2p/go-libp2p-gostream#65](https://github.com/libp2p/go-libp2p-gostream/pull/65))
  - sync: update CI config files ([libp2p/go-libp2p-gostream#62](https://github.com/libp2p/go-libp2p-gostream/pull/62))
  - fix staticcheck ([libp2p/go-libp2p-gostream#61](https://github.com/libp2p/go-libp2p-gostream/pull/61))
- github.com/libp2p/go-libp2p-http (v0.2.1 -> v0.4.0):
  - release v0.4.0 ([libp2p/go-libp2p-http#81](https://github.com/libp2p/go-libp2p-http/pull/81))
  - sync: update CI config files ([libp2p/go-libp2p-http#79](https://github.com/libp2p/go-libp2p-http/pull/79))
  - Update to latest go-libp2p ([libp2p/go-libp2p-http#80](https://github.com/libp2p/go-libp2p-http/pull/80))
  - Update to latest go-libp2p ([libp2p/go-libp2p-http#78](https://github.com/libp2p/go-libp2p-http/pull/78))
  - sync: update CI config files (#73) ([libp2p/go-libp2p-http#73](https://github.com/libp2p/go-libp2p-http/pull/73))
- github.com/libp2p/go-libp2p-kad-dht (v0.18.0 -> v0.20.0):
  - release v0.20.0 (#803) ([libp2p/go-libp2p-kad-dht#803](https://github.com/libp2p/go-libp2p-kad-dht/pull/803))
  - feat: increase the max record age to 48h (PUT_VALUE, RFM17) (#794) ([libp2p/go-libp2p-kad-dht#794](https://github.com/libp2p/go-libp2p-kad-dht/pull/794))
  - feat: increase expiration time for Provider Records to 48h (RFM17)
  - release v0.19.0 (#801) ([libp2p/go-libp2p-kad-dht#801](https://github.com/libp2p/go-libp2p-kad-dht/pull/801))
  - define the ProviderAddrTTL in this repo (#797) ([libp2p/go-libp2p-kad-dht#797](https://github.com/libp2p/go-libp2p-kad-dht/pull/797))
- github.com/libp2p/go-libp2p-kbucket (v0.4.7 -> v0.5.0):
  - chore: release 0.5.0 (#111) ([libp2p/go-libp2p-kbucket#111](https://github.com/libp2p/go-libp2p-kbucket/pull/111))
  - deprecate go-libp2p-core and use go-libp2p instead (#109) ([libp2p/go-libp2p-kbucket#109](https://github.com/libp2p/go-libp2p-kbucket/pull/109))
  - sync: update CI config files (#108) ([libp2p/go-libp2p-kbucket#108](https://github.com/libp2p/go-libp2p-kbucket/pull/108))
  - sync: update CI config files ([libp2p/go-libp2p-kbucket#107](https://github.com/libp2p/go-libp2p-kbucket/pull/107))
  - sync: update CI config files (#104) ([libp2p/go-libp2p-kbucket#104](https://github.com/libp2p/go-libp2p-kbucket/pull/104))
  - sync: update CI config files ([libp2p/go-libp2p-kbucket#101](https://github.com/libp2p/go-libp2p-kbucket/pull/101))
  - sync: update CI config files ([libp2p/go-libp2p-kbucket#99](https://github.com/libp2p/go-libp2p-kbucket/pull/99))
  - fix staticcheck ([libp2p/go-libp2p-kbucket#98](https://github.com/libp2p/go-libp2p-kbucket/pull/98))
- github.com/libp2p/go-libp2p-pubsub (v0.6.1 -> v0.8.2):
  - Add docstring for WithAppSpecificRPCInspector (#510) ([libp2p/go-libp2p-pubsub#510](https://github.com/libp2p/go-libp2p-pubsub/pull/510))
  - Adds Application Specific RPC Inspector (#509) ([libp2p/go-libp2p-pubsub#509](https://github.com/libp2p/go-libp2p-pubsub/pull/509))
  - chore: ignore signing keys during WithLocalPublication publishing (#497) ([libp2p/go-libp2p-pubsub#497](https://github.com/libp2p/go-libp2p-pubsub/pull/497))
  - improve handling of dead peers (#508) ([libp2p/go-libp2p-pubsub#508](https://github.com/libp2p/go-libp2p-pubsub/pull/508))
  - perf: use pooled buffers for message writes (#507) ([libp2p/go-libp2p-pubsub#507](https://github.com/libp2p/go-libp2p-pubsub/pull/507))
  - perf: use msgio pooled buffers for received msgs (#500) ([libp2p/go-libp2p-pubsub#500](https://github.com/libp2p/go-libp2p-pubsub/pull/500))
  - Enables injectable GossipSub router (#503) ([libp2p/go-libp2p-pubsub#503](https://github.com/libp2p/go-libp2p-pubsub/pull/503))
  - Enables non-atomic validation for peer scoring parameters (#499) ([libp2p/go-libp2p-pubsub#499](https://github.com/libp2p/go-libp2p-pubsub/pull/499))
  - update go-libp2p to v0.22.0 (#498) ([libp2p/go-libp2p-pubsub#498](https://github.com/libp2p/go-libp2p-pubsub/pull/498))
  - fix handling of dead peers (#492) ([libp2p/go-libp2p-pubsub#492](https://github.com/libp2p/go-libp2p-pubsub/pull/492))
  - feat: WithLocalPublication option to enable local only publishing on a topic (#481) ([libp2p/go-libp2p-pubsub#481](https://github.com/libp2p/go-libp2p-pubsub/pull/481))
  - update pubsub deps (#491) ([libp2p/go-libp2p-pubsub#491](https://github.com/libp2p/go-libp2p-pubsub/pull/491))
  - Gossipsub: Unsubscribe backoff (#488) ([libp2p/go-libp2p-pubsub#488](https://github.com/libp2p/go-libp2p-pubsub/pull/488))
  - Adds exponential backoff to re-spawing new streams for supposedly dead peers (#483) ([libp2p/go-libp2p-pubsub#483](https://github.com/libp2p/go-libp2p-pubsub/pull/483))
  - Publishing option for signing a message with a custom private key (#486) ([libp2p/go-libp2p-pubsub#486](https://github.com/libp2p/go-libp2p-pubsub/pull/486))
  - fix unused GossipSubHistoryGossip, make seenMessages ttl configurable, make score params SeenMsgTTL configurable
  - Update README.md
  - Add in Backoff Check
  - Modify comment
  - Add Backoff For Pruned Peers
  - tests: new test for WithTopicMsgIdFunction
  - chore: better name
  - feat: detach WithMsgIdFunction
  - fix: use RawID in traceRPCMeta to avoid allocations
  - feat: extract RawID from ID
  - chore: hello mister mutex hat
  - chore: go fmt and return timecache named import
  - feat: new WithMsgIdFunction topic option to enable topics to have own msg id generation rules
  - feat: integrate msgIdGenerator
  - feat: introduce msgIdGenerator and add ID field to Message wrapper
- github.com/libp2p/go-libp2p-pubsub-router (v0.5.0 -> v0.6.0):
  - release v0.6.0 (#99) ([libp2p/go-libp2p-pubsub-router#99](https://github.com/libp2p/go-libp2p-pubsub-router/pull/99))
  - sync: update CI config files (#93) ([libp2p/go-libp2p-pubsub-router#93](https://github.com/libp2p/go-libp2p-pubsub-router/pull/93))
- github.com/libp2p/go-libp2p-routing-helpers (v0.4.0 -> v0.6.0):
  - Update version.json
  - Change interface name
  - Add tests
  - Feat: retrieve routers from composable routers
  - Update version.json
  - Update version.json
  - chore: add regression test for compparallel deadlock
  - Add logs to composable parallel
  - Bump version to v0.4.1
  -  ([libp2p/go-libp2p-routing-helpers#64](https://github.com/libp2p/go-libp2p-routing-helpers/pull/64))
- github.com/lucas-clemente/quic-go (v0.29.1 -> v0.31.1):
  - qerr: include role (remote / local) in error string representations (#3629) ([lucas-clemente/quic-go#3629](https://github.com/lucas-clemente/quic-go/pull/3629))
  - introduce a type for the stateless reset key (#3621) ([lucas-clemente/quic-go#3621](https://github.com/lucas-clemente/quic-go/pull/3621))
  - limit the exponential PTO backoff to 60s (#3595) ([lucas-clemente/quic-go#3595](https://github.com/lucas-clemente/quic-go/pull/3595))
  - expose the QUIC version of a connection (#3620) ([lucas-clemente/quic-go#3620](https://github.com/lucas-clemente/quic-go/pull/3620))
  - expose function to convert byte slice to a connection ID (#3614) ([lucas-clemente/quic-go#3614](https://github.com/lucas-clemente/quic-go/pull/3614))
  - use `go run` for mockgen, goimports and ginkgo (#3616) ([lucas-clemente/quic-go#3616](https://github.com/lucas-clemente/quic-go/pull/3616))
  - fix client SNI handling (#3613) ([lucas-clemente/quic-go#3613](https://github.com/lucas-clemente/quic-go/pull/3613))
  - chore: fix multiple typos in comments (#3612) ([lucas-clemente/quic-go#3612](https://github.com/lucas-clemente/quic-go/pull/3612))
  - use the new zero-allocation control message parsing function from x/sys (#3609) ([lucas-clemente/quic-go#3609](https://github.com/lucas-clemente/quic-go/pull/3609))
  - http3: add support for parsing and writing HTTP/3 capsules (#3607) ([lucas-clemente/quic-go#3607](https://github.com/lucas-clemente/quic-go/pull/3607))
  - http3: add request to response (#3608) ([lucas-clemente/quic-go#3608](https://github.com/lucas-clemente/quic-go/pull/3608))
  - fix availability signaling of the send queue (#3597) ([lucas-clemente/quic-go#3597](https://github.com/lucas-clemente/quic-go/pull/3597))
  - http3: add a ConnectionState method to the StreamCreator interface (#3600) ([lucas-clemente/quic-go#3600](https://github.com/lucas-clemente/quic-go/pull/3600))
  - http3: add a Context method to the StreamCreator interface (#3601) ([lucas-clemente/quic-go#3601](https://github.com/lucas-clemente/quic-go/pull/3601))
  - rename the variable in quic.Config.AllowConnectionWindowIncrease (#3602) ([lucas-clemente/quic-go#3602](https://github.com/lucas-clemente/quic-go/pull/3602))
  - migrate to Ginkgo v2, remove benchmark test ([lucas-clemente/quic-go#3589](https://github.com/lucas-clemente/quic-go/pull/3589))
  - don't drop more than 10 consecutive packets in drop test (#3584) ([lucas-clemente/quic-go#3584](https://github.com/lucas-clemente/quic-go/pull/3584))
  - use a monotonous timer for the connection (#3570) ([lucas-clemente/quic-go#3570](https://github.com/lucas-clemente/quic-go/pull/3570))
  - http3: add http3.Server.ServeQUICConn to serve a single QUIC connection (#3587) ([lucas-clemente/quic-go#3587](https://github.com/lucas-clemente/quic-go/pull/3587))
  - http3: expose ALPN values (#3580) ([lucas-clemente/quic-go#3580](https://github.com/lucas-clemente/quic-go/pull/3580))
  - log the size of buffered packets (#3571) ([lucas-clemente/quic-go#3571](https://github.com/lucas-clemente/quic-go/pull/3571))
  - ackhandler: reject duplicate packets in ReceivedPacket (#3568) ([lucas-clemente/quic-go#3568](https://github.com/lucas-clemente/quic-go/pull/3568))
  - reduce max DATAGRAM frame size, so that DATAGRAMs fit in IPv6 packets (#3581) ([lucas-clemente/quic-go#3581](https://github.com/lucas-clemente/quic-go/pull/3581))
  - use a Peek / Pop API for the datagram queue (#3582) ([lucas-clemente/quic-go#3582](https://github.com/lucas-clemente/quic-go/pull/3582))
  - http3: handle ErrAbortHandler when the handler panics (#3575) ([lucas-clemente/quic-go#3575](https://github.com/lucas-clemente/quic-go/pull/3575))
  - http3: fix double close of chan when using DontCloseRequestStream (#3561) ([lucas-clemente/quic-go#3561](https://github.com/lucas-clemente/quic-go/pull/3561))
  - qlog: rename key_retired to key_discarded (#3463) ([lucas-clemente/quic-go#3463](https://github.com/lucas-clemente/quic-go/pull/3463))
  - simplify packing of ACK-only packets ([lucas-clemente/quic-go#3545](https://github.com/lucas-clemente/quic-go/pull/3545))
  - use a sync.Pool for ACK frames ([lucas-clemente/quic-go#3547](https://github.com/lucas-clemente/quic-go/pull/3547))
  - prioritize sending ACKs over sending new DATAGRAM frames (#3544) ([lucas-clemente/quic-go#3544](https://github.com/lucas-clemente/quic-go/pull/3544))
  - http3: reduce usage of bytes.Buffer (#3539) ([lucas-clemente/quic-go#3539](https://github.com/lucas-clemente/quic-go/pull/3539))
  - use a single bytes.Reader for frame parsing (#3536) ([lucas-clemente/quic-go#3536](https://github.com/lucas-clemente/quic-go/pull/3536))
  - split code paths for packing 0-RTT and 1-RTT packets in packet packer (#3540) ([lucas-clemente/quic-go#3540](https://github.com/lucas-clemente/quic-go/pull/3540))
  - remove the wire.ShortHeader in favor of more return values (#3535) ([lucas-clemente/quic-go#3535](https://github.com/lucas-clemente/quic-go/pull/3535))
  - preallocate the message buffers of the ipv4.Message passed to ReadBatch (#3541) ([lucas-clemente/quic-go#3541](https://github.com/lucas-clemente/quic-go/pull/3541))
  - introduce a separate code paths for Short Header packet handling ([lucas-clemente/quic-go#3534](https://github.com/lucas-clemente/quic-go/pull/3534))
  - fix usage of ackhandler.Packet pool for non-ack-eliciting packets (#3538) ([lucas-clemente/quic-go#3538](https://github.com/lucas-clemente/quic-go/pull/3538))
  - return an error when parsing a too long connection ID from a header (#3533) ([lucas-clemente/quic-go#3533](https://github.com/lucas-clemente/quic-go/pull/3533))
  - speed up marshaling of transport parameters (#3531) ([lucas-clemente/quic-go#3531](https://github.com/lucas-clemente/quic-go/pull/3531))
  - use a struct containing an array to represent Connection IDs ([lucas-clemente/quic-go#3529](https://github.com/lucas-clemente/quic-go/pull/3529))
  - reduce allocations of ackhandler.Packet ([lucas-clemente/quic-go#3525](https://github.com/lucas-clemente/quic-go/pull/3525))
  - serialize frames by appending to a byte slice, not to a bytes.Buffer ([lucas-clemente/quic-go#3530](https://github.com/lucas-clemente/quic-go/pull/3530))
  - fix datagram RFC number in documentation for quic.Config (#3523) ([lucas-clemente/quic-go#3523](https://github.com/lucas-clemente/quic-go/pull/3523))
  - add DPLPMTUD (RFC 8899) to list of supported RFCs in README (#3520) ([lucas-clemente/quic-go#3520](https://github.com/lucas-clemente/quic-go/pull/3520))
  - use the null tracers in the tracer integration tests (#3528) ([lucas-clemente/quic-go#3528](https://github.com/lucas-clemente/quic-go/pull/3528))
- github.com/marten-seemann/webtransport-go (v0.1.1 -> v0.4.3):
  - release v0.4.3 (#57) ([marten-seemann/webtransport-go#57](https://github.com/marten-seemann/webtransport-go/pull/57))
  - return the correct error from OpenStreamSync when context is canceled (#55) ([marten-seemann/webtransport-go#55](https://github.com/marten-seemann/webtransport-go/pull/55))
  - release v0.4.2 (#54) ([marten-seemann/webtransport-go#54](https://github.com/marten-seemann/webtransport-go/pull/54))
  - use a buffered channel in the acceptQueue (#53) ([marten-seemann/webtransport-go#53](https://github.com/marten-seemann/webtransport-go/pull/53))
  - add a comment why using (a blocking) Read in the StreamHijacker is fine
  - release v0.4.1 (#52) ([marten-seemann/webtransport-go#52](https://github.com/marten-seemann/webtransport-go/pull/52))
  - release session mutex when an error occurs when closing (#51) ([marten-seemann/webtransport-go#51](https://github.com/marten-seemann/webtransport-go/pull/51))
  - release v0.4.0 (#48) ([marten-seemann/webtransport-go#48](https://github.com/marten-seemann/webtransport-go/pull/48))
  - add a Server.ServeQUICConn method (#47) ([marten-seemann/webtransport-go#47](https://github.com/marten-seemann/webtransport-go/pull/47))
  - release v0.3.0 (#46) ([marten-seemann/webtransport-go#46](https://github.com/marten-seemann/webtransport-go/pull/46))
  - read and write CLOSE_WEBTRANSPORT_SESSION capsules ([marten-seemann/webtransport-go#40](https://github.com/marten-seemann/webtransport-go/pull/40))
  - implement the SetDeadline method on the stream (#44) ([marten-seemann/webtransport-go#44](https://github.com/marten-seemann/webtransport-go/pull/44))
  - expose the QUIC stream ID on the stream interfaces (#43) ([marten-seemann/webtransport-go#43](https://github.com/marten-seemann/webtransport-go/pull/43))
  - release v0.2.0 (#38) ([marten-seemann/webtransport-go#38](https://github.com/marten-seemann/webtransport-go/pull/38))
  - expose quic-go's connection tracing ID on the Session.Context (#35) ([marten-seemann/webtransport-go#35](https://github.com/marten-seemann/webtransport-go/pull/35))
  - add a ConnectionState method to the Session (#33) ([marten-seemann/webtransport-go#33](https://github.com/marten-seemann/webtransport-go/pull/33))
  - chore: update quic-go to v0.30.0 (#36) ([marten-seemann/webtransport-go#36](https://github.com/marten-seemann/webtransport-go/pull/36))
  - fix interop build (#37) ([marten-seemann/webtransport-go#37](https://github.com/marten-seemann/webtransport-go/pull/37))
  - rename session receiver variable (#34) ([marten-seemann/webtransport-go#34](https://github.com/marten-seemann/webtransport-go/pull/34))
  - fix double close of chan when using DontCloseRequestStream (#30) ([marten-seemann/webtransport-go#30](https://github.com/marten-seemann/webtransport-go/pull/30))
  - add a simple integration test using Selenium and a headless Chrome (#28) ([marten-seemann/webtransport-go#28](https://github.com/marten-seemann/webtransport-go/pull/28))
  - use a generic accept queue for uni- and bidirectional streams (#26) ([marten-seemann/webtransport-go#26](https://github.com/marten-seemann/webtransport-go/pull/26))
- github.com/multiformats/go-base36 (v0.1.0 -> v0.2.0):
  - v0.2.0
  - sync: update CI config files (#11) ([multiformats/go-base36#11](https://github.com/multiformats/go-base36/pull/11))
  - fix link to documentation (#9) ([multiformats/go-base36#9](https://github.com/multiformats/go-base36/pull/9))
  - sync: update CI config files (#8) ([multiformats/go-base36#8](https://github.com/multiformats/go-base36/pull/8))
  - Address `staticcheck` issue ([multiformats/go-base36#5](https://github.com/multiformats/go-base36/pull/5))
  - Feat/fasterer ([multiformats/go-base36#4](https://github.com/multiformats/go-base36/pull/4))
- github.com/multiformats/go-multiaddr (v0.7.0 -> v0.8.0):
  - release v0.8.0 ([multiformats/go-multiaddr#187](https://github.com/multiformats/go-multiaddr/pull/187))
  - Add quic-v1 component ([multiformats/go-multiaddr#186](https://github.com/multiformats/go-multiaddr/pull/186))
- github.com/multiformats/go-varint (v0.0.6 -> v0.0.7):
  - v0.0.7 ([multiformats/go-varint#18](https://github.com/multiformats/go-varint/pull/18))
  - feat: optimize decoding (#15) ([multiformats/go-varint#15](https://github.com/multiformats/go-varint/pull/15))
  - sync: update CI config files (#13) ([multiformats/go-varint#13](https://github.com/multiformats/go-varint/pull/13))
  - fix staticcheck ([multiformats/go-varint#9](https://github.com/multiformats/go-varint/pull/9))
  - tests for unbounded uvarint streams. ([multiformats/go-varint#7](https://github.com/multiformats/go-varint/pull/7))
- github.com/whyrusleeping/cbor-gen (v0.0.0-20210219115102-f37d292932f2 -> v0.0.0-20221220214510-0333c149dec0):
  - fix bug in consts code
  - Allow 'const' fields to be declared ([whyrusleeping/cbor-gen#78](https://github.com/whyrusleeping/cbor-gen/pull/78))
  - support omitting empty fields on map encoders ([whyrusleeping/cbor-gen#77](https://github.com/whyrusleeping/cbor-gen/pull/77))
  - support string pointers
  - feat: add the ability to encode a byte array (#76) ([whyrusleeping/cbor-gen#76](https://github.com/whyrusleeping/cbor-gen/pull/76))
  - support string slices (#73) ([whyrusleeping/cbor-gen#73](https://github.com/whyrusleeping/cbor-gen/pull/73))
  - Feat/size types ([whyrusleeping/cbor-gen#69](https://github.com/whyrusleeping/cbor-gen/pull/69))
  - Add CborReader and CborWriter (#67) ([whyrusleeping/cbor-gen#67](https://github.com/whyrusleeping/cbor-gen/pull/67))
  - Fix broken TestTimeIsh (#66) ([whyrusleeping/cbor-gen#66](https://github.com/whyrusleeping/cbor-gen/pull/66))
  - Return EOF and ErrUnexpectedEOF correctly (#64) ([whyrusleeping/cbor-gen#64](https://github.com/whyrusleeping/cbor-gen/pull/64))
  - fix: don't fail if we try to discard nothing at the end of an object ([whyrusleeping/cbor-gen#63](https://github.com/whyrusleeping/cbor-gen/pull/63))
  - Make peeker.ReadByte follow buffer.ReadByte semantics ([whyrusleeping/cbor-gen#61](https://github.com/whyrusleeping/cbor-gen/pull/61))
  - Fix read bug in readByteBuf ([whyrusleeping/cbor-gen#60](https://github.com/whyrusleeping/cbor-gen/pull/60))
  - support for cborgen struct field tags ([whyrusleeping/cbor-gen#58](https://github.com/whyrusleeping/cbor-gen/pull/58))
  - feat: take cbor adapters by-value when encoding ([whyrusleeping/cbor-gen#55](https://github.com/whyrusleeping/cbor-gen/pull/55))
  - fix: import "math" in generated code for uint8 unmarshalling ([whyrusleeping/cbor-gen#53](https://github.com/whyrusleeping/cbor-gen/pull/53))
  - doc: document Write*EncodersToFile functions ([whyrusleeping/cbor-gen#52](https://github.com/whyrusleeping/cbor-gen/pull/52))

</details>

### 👨‍👩‍👧‍👦 Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 154 | +8826/-6369 | 911 |
| Gus Eggert | 7 | +2792/-1444 | 40 |
| Marco Munizaga | 26 | +2324/-752 | 101 |
| hannahhoward | 7 | +695/-1587 | 50 |
| Rod Vagg | 30 | +1508/-668 | 106 |
| Henrique Dias | 13 | +1321/-431 | 85 |
| Yahya Hassanzadeh | 4 | +984/-158 | 9 |
| galargh | 17 | +519/-520 | 20 |
| Steve Loeppky | 11 | +612/-418 | 25 |
| Antonio Navarro Perez | 30 | +742/-88 | 47 |
| Marcin Rataj | 19 | +377/-407 | 52 |
| Ian Davis | 2 | +419/-307 | 7 |
| whyrusleeping | 5 | +670/-28 | 17 |
| Piotr Galar | 8 | +211/-417 | 25 |
| web3-bot | 28 | +282/-264 | 75 |
| Will Scott | 10 | +428/-103 | 19 |
| julian88110 | 2 | +367/-55 | 27 |
| Will | 5 | +282/-131 | 65 |
| Jorropo | 25 | +263/-94 | 38 |
| Wondertan | 10 | +203/-87 | 24 |
| Mohsin Zaidi | 1 | +269/-0 | 4 |
| Dennis Trautwein | 3 | +230/-21 | 7 |
| Prithvi Shahi | 1 | +116/-77 | 1 |
| Masih H. Derkani | 5 | +130/-37 | 11 |
| Iulian Pascalau | 1 | +151/-16 | 2 |
| Scott Martin | 1 | +166/-0 | 3 |
| Daniel Vernall | 1 | +92/-45 | 2 |
| Steven Allen | 7 | +114/-15 | 11 |
| Hlib Kanunnikov | 4 | +100/-28 | 6 |
| Peter Rabbitson | 4 | +59/-65 | 5 |
| Lucas Molas | 1 | +60/-57 | 7 |
| nisdas | 3 | +107/-6 | 5 |
| why | 2 | +80/-20 | 5 |
| ShengTao | 2 | +46/-45 | 16 |
| nisainan | 2 | +40/-50 | 12 |
| Mikel Cortes | 3 | +44/-36 | 10 |
| Chinmay Kousik | 1 | +64/-14 | 6 |
| ZenGround0 | 2 | +62/-15 | 6 |
| Antonio Navarro | 3 | +58/-3 | 8 |
| Michael Muré | 2 | +49/-2 | 2 |
| Dirk McCormick | 1 | +3/-42 | 1 |
| kixelated | 1 | +20/-20 | 4 |
| Russell Dempsey | 1 | +19/-17 | 3 |
| Karthik Nallabolu | 1 | +17/-17 | 1 |
| protolambda | 1 | +26/-4 | 4 |
| cliffc-spirent | 1 | +25/-5 | 2 |
| Raúl Kripalani | 1 | +29/-0 | 1 |
| Håvard Anda Estensen | 1 | +9/-19 | 6 |
| vyzo | 1 | +11/-12 | 1 |
| anorth | 1 | +15/-8 | 3 |
| shade34321 | 1 | +21/-1 | 2 |
| Toby | 2 | +9/-13 | 6 |
| Nishant Das | 1 | +9/-9 | 5 |
| Jeromy Johnson | 1 | +17/-0 | 3 |
| Oleg | 1 | +14/-1 | 1 |
| Hector Sanjuan | 6 | +4/-11 | 6 |
| Łukasz Magiera | 2 | +10/-4 | 2 |
| Aayush Rajasekaran | 1 | +7/-7 | 1 |
| Adin Schmahmann | 1 | +4/-3 | 1 |
| Stojan Dimitrovski | 1 | +6/-0 | 1 |
| Mathew Jacob | 1 | +3/-3 | 1 |
| Vladimir Ivanov | 1 | +2/-2 | 1 |
| Nex Zhu | 1 | +4/-0 | 2 |
| Michele Mastrogiovanni | 1 | +2/-2 | 1 |
| Louis Thibault | 1 | +4/-0 | 1 |
| Eric Myhre | 1 | +3/-1 | 1 |
| Kubo Mage | 2 | +2/-1 | 2 |
| tabcat | 1 | +1/-1 | 1 |
| Viacheslav | 1 | +1/-1 | 1 |
| Max Inden | 1 | +1/-1 | 1 |
| Manic Security | 1 | +1/-1 | 1 |
| Jc0803kevin | 1 | +1/-1 | 1 |
| David Brouwer | 1 | +2/-0 | 2 |
| Rong Zhou | 1 | +1/-0 | 1 |
| Neel Virdy | 1 | +1/-0 | 1 |
