# go-ipfs 源码解析 41

# go-ipfs changelog v0.7

## v0.7.0 2020-09-22

### Highlights

#### Secio is now disabled by default

As part of deprecating and removing support for the Secio security transport, we have disabled it by default. TLS1.3 will remain the default security transport with fallback to Noise. You can read more about the deprecation in the blog post, https://blog.ipfs.io/2020-08-07-deprecating-secio/. If you're running IPFS older than 0.5, this may start to impact your performance on the public network.

#### Ed25519 keys are now used by default

Previously go-ipfs generated 2048 bit RSA keys for new nodes, but it will now use ed25519 keys by default. This will not affect any existing keys, but newly created keys will be ed25519 by default. The main benefit of using ed25519 keys over RSA is that ed25519 keys have an inline public key. This means that someone only needs your PeerId to verify things you've signed, which means we don't have to worry about storing those bulky RSA public keys.

##### Rotating keys

Along with switching the default, we've added support for rotating keys. If you would like to change the key type of your IPFS node, you can now do so with the rotate command. **NOTE: This will affect your Peer Id, so be sure you want to do this!** Your existing identity key will be backed up in the Keystore.

```bash
ipfs key rotate -o my-old-key -t ed25519
```

#### Key export/import

We've added commands to allow you to export and import keys from the IPFS Keystore to a local .key file. This does not apply to the IPFS identity key, `self`.

```bash
ipfs key gen mykey
ipfs key export -o mykey.key mykey # ./<name>.key is the default path
ipfs key import mykey mykey.key # on another node
```

#### IPNS paths now encode the key name as a base36 CIDv1 by default

Previously go-ipfs encoded the key names for IPNS paths as base58btc multihashes (e.g. Qmabc...). We now encode them as base36 encoded CIDv1s as defined in the [peerID spec](https://github.com/libp2p/specs/blob/master/peer-ids/peer-ids.md#string-representation) (e.g. k51xyz...) which also deals with encoding of public keys. This is nice because it means that IPNS keys will by default be case-insensitive and that they will fit into DNS labels (e.g. k51xyz...ipns.localhost) and therefore that subdomain gateway redirections (e.g. from localhost:8080/ipns/{key} to {key}.ipns.localhost) will look better to users in the default case.

Many commands will accept a `--ipns-base` option that allows changing command outputs to use a particular encoding (i.e.  base58btc multihash, or CIDv1 encoded in any supported base)

#### Multiaddresses now accept PeerIDs encoded as CIDv1

In preparation for eventually changing the default PeerID representation multiaddresses can now contain strings like `/p2p/k51xyz...` in addition to the default `/p2p/Qmabc...`. There is a corresponding `--peerid-base` option to many functions that output peerIDs.

#### `dag stat`

Initial support has been added for the `ipfs dag stat` command. Running this command will traverse the DAG for the given root CID and report statistics. By default, progress will be shown as the DAG is traversed. Supported statistics currently include DAG size and number of blocks.

```bash
ipfs dag stat bafybeihpetclqvwb4qnmumvcn7nh4pxrtugrlpw4jgjpqicdxsv7opdm6e # the IPFS webui
Size: 30362191, NumBlocks: 346
```

#### Plugin build changes

We have changed the build flags used by the official binary distributions on dist.ipfs.tech (or `/ipns/dist.ipfs.tech`) to use the simpler and more reliable `-trimpath` flag instead of the more complicated and brittle `-asmflags=all=-trimpath="$(GOPATH)" -gcflags=all=-trimpath="$(GOPATH)"` flags, however the build flags used by default in go-ipfs remain the same.

The scripts in https://github.com/ipfs/go-ipfs-example-plugin have been updated to reflect this change. This is a breaking change to how people have been building plugins against the dist.ipfs.tech binary of go-ipfs and plugins should update their build processes accordingly see https://github.com/ipfs/go-ipfs-example-plugin/pull/9 for details.

### Changelog

- github.com/ipfs/go-ipfs:
  - chore: bump webui version
  - fix: remove the (empty) alias for --peerid-base
  - Release v0.7.0-rc2
  - fix: use override GOFLAGS changes from 480defab689610550ee3d346e31441a2bb881fcb but keep trimpath usage as is
  - Revert "fix: override GOFLAGS"
  - fix: remove the (empty) alias for --ipns-base
  - refactor: put all --ipns-base options in one place
  - docs: update config to indicate SECIO deprecation
  - fix: ipfs dht put/get commands now work on keys encoded as peerIDs and fail early for namespaces other than /pk or /ipns
  - Release v0.7.0-rc1
  - chore: cleanup ([ipfs/go-ipfs#7628](https://github.com/ipfs/go-ipfs/pull/7628))
  - namesys: fixed IPNS republisher to not overwrite IPNS record lifetimes ([ipfs/go-ipfs#7627](https://github.com/ipfs/go-ipfs/pull/7627))
  - Fix #7624: Do not fetch dag nodes when checking if a pin exists ([ipfs/go-ipfs#7625](https://github.com/ipfs/go-ipfs/pull/7625))
  - chore: update dependencies ([ipfs/go-ipfs#7610](https://github.com/ipfs/go-ipfs/pull/7610))
  - use t.Cleanup() to reduce the need to clean up servers in tests ([ipfs/go-ipfs#7550](https://github.com/ipfs/go-ipfs/pull/7550))
  - fix: ipfs pin ls - ignore pins that have errors ([ipfs/go-ipfs#7612](https://github.com/ipfs/go-ipfs/pull/7612))
  - docs(config): fix Peering header ([ipfs/go-ipfs#7623](https://github.com/ipfs/go-ipfs/pull/7623))
  - sharness: use dnsaddr example in ipfs p2p command tests ([ipfs/go-ipfs#7620](https://github.com/ipfs/go-ipfs/pull/7620))
  - fix(key): don't allow backup key to be named 'self' ([ipfs/go-ipfs#7615](https://github.com/ipfs/go-ipfs/pull/7615))
  - [BOUNTY] Directory page UI improvements ([ipfs/go-ipfs#7536](https://github.com/ipfs/go-ipfs/pull/7536))
  - fix: make assets deterministic ([ipfs/go-ipfs#7609](https://github.com/ipfs/go-ipfs/pull/7609))
  - use ed25519 keys by default ([ipfs/go-ipfs#7579](https://github.com/ipfs/go-ipfs/pull/7579))
  - feat: wildcard support for public gateways ([ipfs/go-ipfs#7319](https://github.com/ipfs/go-ipfs/pull/7319))
  - fix: fix go-bindata import path ([ipfs/go-ipfs#7605](https://github.com/ipfs/go-ipfs/pull/7605))
  - Upgrade graphsync deps ([ipfs/go-ipfs#7598](https://github.com/ipfs/go-ipfs/pull/7598))
  - Add --peerid-base to ipfs id command ([ipfs/go-ipfs#7591](https://github.com/ipfs/go-ipfs/pull/7591))
  - use b36 keys by default for keys and IPNS ([ipfs/go-ipfs#7582](https://github.com/ipfs/go-ipfs/pull/7582))
  - add ipfs dag stat command (#7553) ([ipfs/go-ipfs#7553](https://github.com/ipfs/go-ipfs/pull/7553))
  - Move key rotation command to ipfs key rotate ([ipfs/go-ipfs#7599](https://github.com/ipfs/go-ipfs/pull/7599))
  - Disable secio by default ([ipfs/go-ipfs#7600](https://github.com/ipfs/go-ipfs/pull/7600))
  - Stop searching for public keys before doing an IPNS Get (#7549) ([ipfs/go-ipfs#7549](https://github.com/ipfs/go-ipfs/pull/7549))
  - feat: return supported protocols in id output ([ipfs/go-ipfs#7409](https://github.com/ipfs/go-ipfs/pull/7409))
  - docs: fix typo in default swarm addrs config docs ([ipfs/go-ipfs#7585](https://github.com/ipfs/go-ipfs/pull/7585))
  - feat: nice errors when failing to load plugins ([ipfs/go-ipfs#7429](https://github.com/ipfs/go-ipfs/pull/7429))
  - doc: document reverse proxy bug ([ipfs/go-ipfs#7478](https://github.com/ipfs/go-ipfs/pull/7478))
  - fix: ipfs name resolve --dht-record-count flag uses correct type and now works
  - refactor: get rid of cmdDetails awkwardness
  - IPNS format keys in b36cid ([ipfs/go-ipfs#7554](https://github.com/ipfs/go-ipfs/pull/7554))
  - Key import and export cli commands ([ipfs/go-ipfs#7546](https://github.com/ipfs/go-ipfs/pull/7546))
  - feat: add snap package configuration ([ipfs/go-ipfs#7529](https://github.com/ipfs/go-ipfs/pull/7529))
  - chore: bump webui version
  - repeat gateway subdomain test for all key types (#7542) ([ipfs/go-ipfs#7542](https://github.com/ipfs/go-ipfs/pull/7542))
  - fix: override GOFLAGS
  - update QUIC, enable the RetireBugBackwardsCompatibilityMode
  - Document add behavior when the daemon is not running ([ipfs/go-ipfs#7514](https://github.com/ipfs/go-ipfs/pull/7514))
  -  ([ipfs/go-ipfs#7515](https://github.com/ipfs/go-ipfs/pull/7515))
  - Choose Key type at initialization ([ipfs/go-ipfs#7251](https://github.com/ipfs/go-ipfs/pull/7251))
  - feat: add flag to ipfs key and list to output keys in b36/CIDv1 (#7531) ([ipfs/go-ipfs#7531](https://github.com/ipfs/go-ipfs/pull/7531))
  - feat: support ED25519 libp2p-key in subdomains
  - chore: fix a typo
  - docs: document X-Forwarded-Host
  - feat: support X-Forwarded-Host when doing gateway redirect
  - chore: update test deps for graphsync
  - chore: bump test dependencies ([ipfs/go-ipfs#7524](https://github.com/ipfs/go-ipfs/pull/7524))
  - fix: use static binaries in docker container ([ipfs/go-ipfs#7505](https://github.com/ipfs/go-ipfs/pull/7505))
  - chore:bump webui version to 2.10.1 ([ipfs/go-ipfs#7504](https://github.com/ipfs/go-ipfs/pull/7504))
  - chore: bump webui version ([ipfs/go-ipfs#7501](https://github.com/ipfs/go-ipfs/pull/7501))
  - update version to 0.7.0-dev
  - Merge branch 'release' into master
  - systemd: specify repo path, to avoid unnecessary subdirectory ([ipfs/go-ipfs#7472](https://github.com/ipfs/go-ipfs/pull/7472))
  - doc(prod): start documenting production stuff ([ipfs/go-ipfs#7469](https://github.com/ipfs/go-ipfs/pull/7469))
  - Readme: Update link about init systems (and import old readme) ([ipfs/go-ipfs#7473](https://github.com/ipfs/go-ipfs/pull/7473))
  - doc(config): expand peering docs ([ipfs/go-ipfs#7466](https://github.com/ipfs/go-ipfs/pull/7466))
  - fix: Use the -p option in Dockerfile to make parents as needed ([ipfs/go-ipfs#7464](https://github.com/ipfs/go-ipfs/pull/7464))
  - systemd: enable systemd hardening features ([ipfs/go-ipfs#7286](https://github.com/ipfs/go-ipfs/pull/7286))
  - fix(migration): migrate /ipfs/ bootstrappers to /p2p/ ([ipfs/go-ipfs#7450](https://github.com/ipfs/go-ipfs/pull/7450))
  - readme: update go-version ([ipfs/go-ipfs#7447](https://github.com/ipfs/go-ipfs/pull/7447))
  - fix(migration): correctly migrate quic addresses ([ipfs/go-ipfs#7446](https://github.com/ipfs/go-ipfs/pull/7446))
  - chore: add migration to listen on QUIC by default ([ipfs/go-ipfs#7443](https://github.com/ipfs/go-ipfs/pull/7443))
  - go: bump minimal dependency to 1.14.4 ([ipfs/go-ipfs#7419](https://github.com/ipfs/go-ipfs/pull/7419))
  - fix: use bitswap sessions for ipfs refs ([ipfs/go-ipfs#7389](https://github.com/ipfs/go-ipfs/pull/7389))
  - fix(commands): print consistent addresses in ipfs id ([ipfs/go-ipfs#7397](https://github.com/ipfs/go-ipfs/pull/7397))
  - fix two pubsub issues. ([ipfs/go-ipfs#7394](https://github.com/ipfs/go-ipfs/pull/7394))
  - docs: add pacman.store (@RubenKelevra) to the early testers ([ipfs/go-ipfs#7368](https://github.com/ipfs/go-ipfs/pull/7368))
  - Update docs-beta links to final URLs ([ipfs/go-ipfs#7386](https://github.com/ipfs/go-ipfs/pull/7386))
  - feat: webui v2.9.0 ([ipfs/go-ipfs#7387](https://github.com/ipfs/go-ipfs/pull/7387))
  - chore: update WebUI to 2.8.0 ([ipfs/go-ipfs#7380](https://github.com/ipfs/go-ipfs/pull/7380))
  - mailmap support ([ipfs/go-ipfs#7375](https://github.com/ipfs/go-ipfs/pull/7375))
  - doc: update the release template for git flow changes ([ipfs/go-ipfs#7370](https://github.com/ipfs/go-ipfs/pull/7370))
  - chore: update deps ([ipfs/go-ipfs#7369](https://github.com/ipfs/go-ipfs/pull/7369))
- github.com/ipfs/go-bitswap (v0.2.19 -> v0.2.20):
  - fix: don't say we're sending a full wantlist unless we are (#429) ([ipfs/go-bitswap#429](https://github.com/ipfs/go-bitswap/pull/429))
- github.com/ipfs/go-cid (v0.0.6 -> v0.0.7):
  - feat: optimize cid.Prefix ([ipfs/go-cid#109](https://github.com/ipfs/go-cid/pull/109))
- github.com/ipfs/go-datastore (v0.4.4 -> v0.4.5):
  - Add test to ensure that Delete returns no error for missing keys ([ipfs/go-datastore#162](https://github.com/ipfs/go-datastore/pull/162))
  - Fix typo in sync/sync.go ([ipfs/go-datastore#159](https://github.com/ipfs/go-datastore/pull/159))
  - Add the generated flatfs stub, since it cannot be auto-generated ([ipfs/go-datastore#158](https://github.com/ipfs/go-datastore/pull/158))
  - support flatfs fuzzing ([ipfs/go-datastore#157](https://github.com/ipfs/go-datastore/pull/157))
  - fuzzing harness (#153) ([ipfs/go-datastore#153](https://github.com/ipfs/go-datastore/pull/153))
  - feat(mount): don't give up on error ([ipfs/go-datastore#146](https://github.com/ipfs/go-datastore/pull/146))
  - /test: fix bad ElemCount/10 lenght (should not be divided) ([ipfs/go-datastore#152](https://github.com/ipfs/go-datastore/pull/152))
- github.com/ipfs/go-ds-flatfs (v0.4.4 -> v0.4.5):
  - Add os.Rename wrapper for Plan 9 (#87) ([ipfs/go-ds-flatfs#87](https://github.com/ipfs/go-ds-flatfs/pull/87))
- github.com/ipfs/go-fs-lock (v0.0.5 -> v0.0.6):
  - Fix build on Plan 9 ([ipfs/go-fs-lock#17](https://github.com/ipfs/go-fs-lock/pull/17))
- github.com/ipfs/go-graphsync (v0.0.5 -> v0.1.1):
  - docs(CHANGELOG): update for v0.1.1
  - docs(CHANGELOG): update for v0.1.0 release ([ipfs/go-graphsync#84](https://github.com/ipfs/go-graphsync/pull/84))
  - Deduplicate by key extension (#83) ([ipfs/go-graphsync#83](https://github.com/ipfs/go-graphsync/pull/83))
  - Release infrastructure (#81) ([ipfs/go-graphsync#81](https://github.com/ipfs/go-graphsync/pull/81))
  - feat(persistenceoptions): add unregister ability (#80) ([ipfs/go-graphsync#80](https://github.com/ipfs/go-graphsync/pull/80))
  - fix(message): regen protobuf code (#79) ([ipfs/go-graphsync#79](https://github.com/ipfs/go-graphsync/pull/79))
  - feat(requestmanager): run response hooks on completed requests (#77) ([ipfs/go-graphsync#77](https://github.com/ipfs/go-graphsync/pull/77))
  - Revert "add extensions on complete (#76)"
  - add extensions on complete (#76) ([ipfs/go-graphsync#76](https://github.com/ipfs/go-graphsync/pull/76))
  - All changes to date including pause requests & start paused, along with new adds for cleanups and checking of execution (#75) ([ipfs/go-graphsync#75](https://github.com/ipfs/go-graphsync/pull/75))
  - More fine grained response controls (#71) ([ipfs/go-graphsync#71](https://github.com/ipfs/go-graphsync/pull/71))
  - Refactor request execution and use IPLD SkipMe functionality for proper partial results on a request (#70) ([ipfs/go-graphsync#70](https://github.com/ipfs/go-graphsync/pull/70))
  - feat(graphsync): implement do-no-send-cids extension (#69) ([ipfs/go-graphsync#69](https://github.com/ipfs/go-graphsync/pull/69))
  - Incoming Block Hooks (#68) ([ipfs/go-graphsync#68](https://github.com/ipfs/go-graphsync/pull/68))
  - fix(responsemanager): add nil check (#67) ([ipfs/go-graphsync#67](https://github.com/ipfs/go-graphsync/pull/67))
  - refactor(hooks): use external pubsub (#65) ([ipfs/go-graphsync#65](https://github.com/ipfs/go-graphsync/pull/65))
  - Update of IPLD Prime (#66) ([ipfs/go-graphsync#66](https://github.com/ipfs/go-graphsync/pull/66))
  - feat(responsemanager): add listener for completed responses (#64) ([ipfs/go-graphsync#64](https://github.com/ipfs/go-graphsync/pull/64))
  - Update Requests (#63) ([ipfs/go-graphsync#63](https://github.com/ipfs/go-graphsync/pull/63))
  - Add pausing and unpausing of requests (#62) ([ipfs/go-graphsync#62](https://github.com/ipfs/go-graphsync/pull/62))
  - Outgoing Request Hooks, swapping persistence layers (#61) ([ipfs/go-graphsync#61](https://github.com/ipfs/go-graphsync/pull/61))
  - Feat/request hook loader chooser (#60) ([ipfs/go-graphsync#60](https://github.com/ipfs/go-graphsync/pull/60))
  - Option to Reject requests by default (#58) ([ipfs/go-graphsync#58](https://github.com/ipfs/go-graphsync/pull/58))
  - Testify refactor (#56) ([ipfs/go-graphsync#56](https://github.com/ipfs/go-graphsync/pull/56))
  - Switch To Circle CI (#57) ([ipfs/go-graphsync#57](https://github.com/ipfs/go-graphsync/pull/57))
  - fix(deps): go mod tidy
  - docs(README): remove ipldbridge reference
  - Tech Debt: Remove IPLD Bridge ([ipfs/go-graphsync#55](https://github.com/ipfs/go-graphsync/pull/55))
- github.com/ipfs/go-ipfs-cmds (v0.2.9 -> v0.4.0):
  - fix: allow requests from electron renderer (#201) ([ipfs/go-ipfs-cmds#201](https://github.com/ipfs/go-ipfs-cmds/pull/201))
  - refactor: move external command checks into commands lib (#198) ([ipfs/go-ipfs-cmds#198](https://github.com/ipfs/go-ipfs-cmds/pull/198))
  - Fix build on Plan 9 ([ipfs/go-ipfs-cmds#199](https://github.com/ipfs/go-ipfs-cmds/pull/199))
- github.com/ipfs/go-ipfs-config (v0.8.0 -> v0.9.0):
  - error if bit size specified with ed25519 keys (#105) ([ipfs/go-ipfs-config#105](https://github.com/ipfs/go-ipfs-config/pull/105))
- github.com/ipfs/go-log/v2 (v2.0.8 -> v2.1.1):
  failed to fetch repo
- github.com/ipfs/go-path (v0.0.7 -> v0.0.8):
  - ResolveToLastNode no longer fetches nodes it does not need ([ipfs/go-path#30](https://github.com/ipfs/go-path/pull/30))
  - doc: add a lead maintainer
- github.com/ipfs/interface-go-ipfs-core (v0.3.0 -> v0.4.0):
  - Add ID formatting functions, used by various IPFS cli commands ([ipfs/interface-go-ipfs-core#65](https://github.com/ipfs/interface-go-ipfs-core/pull/65))
- github.com/ipld/go-car (v0.1.0 -> v0.1.1-0.20200429200904-c222d793c339):
  - Update go-ipld-prime to the era of NodeAssembler. ([ipld/go-car#31](https://github.com/ipld/go-car/pull/31))
  - fix: update the cli tool's car dep ([ipld/go-car#30](https://github.com/ipld/go-car/pull/30))
- github.com/ipld/go-ipld-prime (v0.0.2-0.20191108012745-28a82f04c785 -> v0.0.2-0.20200428162820-8b59dc292b8e):
  - Add two basic examples of usage, as go tests.
  - Fix marshalling error ([ipld/go-ipld-prime#53](https://github.com/ipld/go-ipld-prime/pull/53))
  - Add more test specs for list and map nesting.
  - traversal.SkipMe feature ([ipld/go-ipld-prime#51](https://github.com/ipld/go-ipld-prime/pull/51))
  - Improvements to traversal docs.
  - Drop code coverage bot config. ([ipld/go-ipld-prime#50](https://github.com/ipld/go-ipld-prime/pull/50))
  - Promote NodeAssembler/NodeStyle interface rework to core, and use improved basicnode implementation. ([ipld/go-ipld-prime#49](https://github.com/ipld/go-ipld-prime/pull/49))
  - Merge branch 'traversal-benchmarks'
  - Merge branch 'cycle-breaking-and-traversal-benchmarks'
  - Merge branch 'assembler-upgrade-to-codecs'
  - Path clarifications ([ipld/go-ipld-prime#47](https://github.com/ipld/go-ipld-prime/pull/47))
  - Merge branch 'research-admissions'
  - Add a typed link node to allow traversal with code generated builders across links ([ipld/go-ipld-prime#41](https://github.com/ipld/go-ipld-prime/pull/41))
  - Merge branch 'research-admissions'
  - Library updates.
  - Feat/add code gen disclaimer ([ipld/go-ipld-prime#39](https://github.com/ipld/go-ipld-prime/pull/39))
  - Readme and key Node interface docs improvements.
  - fix(schema/gen): return value not reference ([ipld/go-ipld-prime#38](https://github.com/ipld/go-ipld-prime/pull/38))
- github.com/ipld/go-ipld-prime-proto (v0.0.0-20191113031812-e32bd156a1e5 -> v0.0.0-20200428191222-c1ffdadc01e1):
  - feat(deps): upgrade to new IPLD prime ([ipld/go-ipld-prime-proto#1](https://github.com/ipld/go-ipld-prime-proto/pull/1))
  - Update to latest ipld before rework ([ipld/go-ipld-prime-proto#2](https://github.com/ipld/go-ipld-prime-proto/pull/2))
- github.com/libp2p/go-libp2p (v0.9.6 -> v0.11.0):
  - Added parsing of IPv6 addresses for incoming mDNS requests ([libp2p/go-libp2p#990](https://github.com/libp2p/go-libp2p/pull/990))
  - Switch from SECIO to Noise ([libp2p/go-libp2p#972](https://github.com/libp2p/go-libp2p/pull/972))
  - fix tests ([libp2p/go-libp2p#995](https://github.com/libp2p/go-libp2p/pull/995))
  - Bump Autonat version & validate fixed call loop in `.Addrs` (#988) ([libp2p/go-libp2p#988](https://github.com/libp2p/go-libp2p/pull/988))
  - fix: use the correct external address when NAT port-mapping ([libp2p/go-libp2p#987](https://github.com/libp2p/go-libp2p/pull/987))
  - upgrade deps + interoperable uvarint delimited writer/reader. (#985) ([libp2p/go-libp2p#985](https://github.com/libp2p/go-libp2p/pull/985))
  - fix host can be dialed by autonat public addr, but lost the public addr to announce ([libp2p/go-libp2p#983](https://github.com/libp2p/go-libp2p/pull/983))
  - Fix address advertisement bugs (#974) ([libp2p/go-libp2p#974](https://github.com/libp2p/go-libp2p/pull/974))
  - fix: avoid a close deadlock in the natmanager ([libp2p/go-libp2p#971](https://github.com/libp2p/go-libp2p/pull/971))
  - upgrade swarm; add ID() on mock conns and streams. (#970) ([libp2p/go-libp2p#970](https://github.com/libp2p/go-libp2p/pull/970))
- github.com/libp2p/go-libp2p-asn-util (null -> v0.0.0-20200825225859-85005c6cf052):
  - chore: go fmt
  - feat: use deferred initialization of the asnStore ([libp2p/go-libp2p-asn-util#3](https://github.com/libp2p/go-libp2p-asn-util/pull/3))
  - chore: switch to forked cidranger
  - fixed code
  - library for ASN mappings
- github.com/libp2p/go-libp2p-autonat (v0.2.3 -> v0.3.2):
  - static nat shouldn't call host.Addrs()
  - upgrade deps + interoperable uvarint delimited writer/reader. (#95) ([libp2p/go-libp2p-autonat#95](https://github.com/libp2p/go-libp2p-autonat/pull/95))
  - fix: a type switch nit ([libp2p/go-libp2p-autonat#83](https://github.com/libp2p/go-libp2p-autonat/pull/83))
- github.com/libp2p/go-libp2p-blankhost (v0.1.6 -> v0.2.0):
  - call reset where appropriate (and update deps) ([libp2p/go-libp2p-blankhost#52](https://github.com/libp2p/go-libp2p-blankhost/pull/52))
- github.com/libp2p/go-libp2p-circuit (v0.2.3 -> v0.3.1):
  - upgrade deps + interoperable uvarints. (#122) ([libp2p/go-libp2p-circuit#122](https://github.com/libp2p/go-libp2p-circuit/pull/122))
  - Fix/remove deprecated logging ([libp2p/go-libp2p-circuit#85](https://github.com/libp2p/go-libp2p-circuit/pull/85))
- github.com/libp2p/go-libp2p-core (v0.5.7 -> v0.6.1):
  - experimental introspection support (#159) ([libp2p/go-libp2p-core#159](https://github.com/libp2p/go-libp2p-core/pull/159))
- github.com/libp2p/go-libp2p-discovery (v0.4.0 -> v0.5.0):
  - Put period at end of sentence ([libp2p/go-libp2p-discovery#65](https://github.com/libp2p/go-libp2p-discovery/pull/65))
- github.com/libp2p/go-libp2p-kad-dht (v0.8.2 -> v0.9.0):
  - chore: update deps ([libp2p/go-libp2p-kad-dht#689](https://github.com/libp2p/go-libp2p-kad-dht/pull/689))
  - allow overwriting builtin dual DHT options ([libp2p/go-libp2p-kad-dht#688](https://github.com/libp2p/go-libp2p-kad-dht/pull/688))
  - Hardening Improvements: RT diversity and decreased RT churn ([libp2p/go-libp2p-kad-dht#687](https://github.com/libp2p/go-libp2p-kad-dht/pull/687))
  - Fix key log encoding ([libp2p/go-libp2p-kad-dht#682](https://github.com/libp2p/go-libp2p-kad-dht/pull/682))
  - upgrade deps + uvarint delimited writer/reader. (#684) ([libp2p/go-libp2p-kad-dht#684](https://github.com/libp2p/go-libp2p-kad-dht/pull/684))
  - periodicBootstrapInterval should be ticker? (#678) ([libp2p/go-libp2p-kad-dht#678](https://github.com/libp2p/go-libp2p-kad-dht/pull/678))
  - removes duplicate comment ([libp2p/go-libp2p-kad-dht#674](https://github.com/libp2p/go-libp2p-kad-dht/pull/674))
  - Revert "Peer Diversity in the Routing Table (#658)" ([libp2p/go-libp2p-kad-dht#670](https://github.com/libp2p/go-libp2p-kad-dht/pull/670))
  - Fixed problem with refresh logging ([libp2p/go-libp2p-kad-dht#667](https://github.com/libp2p/go-libp2p-kad-dht/pull/667))
  - feat: protect all peers in low buckets, tag everyone else with 5 ([libp2p/go-libp2p-kad-dht#666](https://github.com/libp2p/go-libp2p-kad-dht/pull/666))
  - Peer Diversity in the Routing Table (#658) ([libp2p/go-libp2p-kad-dht#658](https://github.com/libp2p/go-libp2p-kad-dht/pull/658))
- github.com/libp2p/go-libp2p-kbucket (v0.4.2 -> v0.4.7):
  - chore: switch from go-multiaddr-net to go-multiaddr/net
  - Use crypto/rand for generating random prefixes
  - feat: when using the diversity filter for ipv6 addresses if the ASN cannot be found for a particular address then fallback on using the /32 mask of the  address as the group name instead of simply rejecting the peer from routing table
  - simplify filter (#92) ([libp2p/go-libp2p-kbucket#92](https://github.com/libp2p/go-libp2p-kbucket/pull/92))
  - fix: switch to forked cid ranger dep ([libp2p/go-libp2p-kbucket#91](https://github.com/libp2p/go-libp2p-kbucket/pull/91))
  - Reduce Routing Table churn (#90) ([libp2p/go-libp2p-kbucket#90](https://github.com/libp2p/go-libp2p-kbucket/pull/90))
  - Peer Diversity for Routing Table and Querying (#88) ([libp2p/go-libp2p-kbucket#88](https://github.com/libp2p/go-libp2p-kbucket/pull/88))
  - fix bug in peer eviction (#87) ([libp2p/go-libp2p-kbucket#87](https://github.com/libp2p/go-libp2p-kbucket/pull/87))
  - feat: add an AddedAt timestamp (#84) ([libp2p/go-libp2p-kbucket#84](https://github.com/libp2p/go-libp2p-kbucket/pull/84))
- github.com/libp2p/go-libp2p-pubsub (v0.3.1 -> v0.3.5):
  - regenerate protobufs (#381) ([libp2p/go-libp2p-pubsub#381](https://github.com/libp2p/go-libp2p-pubsub/pull/381))
  - track validation time
  - fulfill promise as soon as a message begins validation
  - don't apply penalty in self origin rejections
  - add behaviour penalty threshold
  - Add String() method to Topic.
  - add regression test for issue 371
  - don't add direct peers to fanout
  - reference spec change in comment.
  - fix backoff slack time
  - use the heartbeat interval for slack time
  - add slack time to prune backoff clearance
  - fix: call the correct tracer function in FloodSubRouter.Leave (#373) ([libp2p/go-libp2p-pubsub#373](https://github.com/libp2p/go-libp2p-pubsub/pull/373))
  - downgrade trace buffer overflow log to debug
  - track topics in Reject/Duplicate/Deliver events
  - add topics to Reject/Duplicate/Deliver events
  - fix flaky test
  - refactor ip colocation factor computation that is common for score and inspection
  - better handling of intermediate topic score snapshots
  - disallow duplicate score inspectors
  - make peer score inspect function types aliases
  - extended peer score inspection
  - upgrade deps + interoperable uvarint delimited writer/reader.
  - Add warning about messageIDs
  - Signing policy + optional Signature, From and Seqno ([libp2p/go-libp2p-pubsub#359](https://github.com/libp2p/go-libp2p-pubsub/pull/359))
  - Update pubsub.go
  - Define a public error ErrSubscriptionCancelled.
  - only do PX on leave if PX was enabled in the node
  - drop warning about failure to open stream to a debug log
  - reinstate tagging (now protection) tests
  - disable tests for direct/mesh tags, we don't have an interface to query the connman yet
  - protect direct and mesh peers in the connection manager
  - feat: add direct connect ticks option
- github.com/libp2p/go-libp2p-pubsub-router (v0.3.0 -> v0.3.2):
  - upgrade deps + interoperable uvarint delimited writer/reader. (#79) ([libp2p/go-libp2p-pubsub-router#79](https://github.com/libp2p/go-libp2p-pubsub-router/pull/79))
- github.com/libp2p/go-libp2p-quic-transport (v0.6.0 -> v0.8.0):
  - update quic-go to v0.18.0 (#171) ([libp2p/go-libp2p-quic-transport#171](https://github.com/libp2p/go-libp2p-quic-transport/pull/171))
- github.com/libp2p/go-libp2p-swarm (v0.2.6 -> v0.2.8):
  - slim down dependencies ([libp2p/go-libp2p-swarm#225](https://github.com/libp2p/go-libp2p-swarm/pull/225))
  - `ID()` method on connections and streams + record opening time (#224) ([libp2p/go-libp2p-swarm#224](https://github.com/libp2p/go-libp2p-swarm/pull/224))
- github.com/libp2p/go-libp2p-testing (v0.1.1 -> v0.2.0):
  - Add net benchmark harness ([libp2p/go-libp2p-testing#21](https://github.com/libp2p/go-libp2p-testing/pull/21))
  - Update suite to check that streams respect mux.ErrReset. ([libp2p/go-libp2p-testing#16](https://github.com/libp2p/go-libp2p-testing/pull/16))
- github.com/libp2p/go-maddr-filter (v0.0.5 -> v0.1.0):
  - deprecate this package; moved to multiformats/go-multiaddr. (#23) ([libp2p/go-maddr-filter#23](https://github.com/libp2p/go-maddr-filter/pull/23))
  - chore(dep): update ([libp2p/go-maddr-filter#18](https://github.com/libp2p/go-maddr-filter/pull/18))
- github.com/libp2p/go-msgio (v0.0.4 -> v0.0.6):
  - interoperable uvarints. (#21) ([libp2p/go-msgio#21](https://github.com/libp2p/go-msgio/pull/21))
  - upgrade deps + interoperable uvarint delimited writer/reader. (#20) ([libp2p/go-msgio#20](https://github.com/libp2p/go-msgio/pull/20))
- github.com/libp2p/go-netroute (v0.1.2 -> v0.1.3):
  - add Plan 9 support
- github.com/libp2p/go-openssl (v0.0.5 -> v0.0.7):
  - make ed25519 less special ([libp2p/go-openssl#7](https://github.com/libp2p/go-openssl/pull/7))
  - Add required bindings to support openssl in libp2p-tls ([libp2p/go-openssl#6](https://github.com/libp2p/go-openssl/pull/6))
- github.com/libp2p/go-reuseport (v0.0.1 -> v0.0.2):
  - Fix build on Plan 9 ([libp2p/go-reuseport#79](https://github.com/libp2p/go-reuseport/pull/79))
  - farewell gx; thanks for serving us well.
  - update readme badges
  - remove Jenkinsfile.
- github.com/libp2p/go-reuseport-transport (v0.0.3 -> v0.0.4):
  - Update go-netroute and go-reuseport for Plan 9 support
  - Fix build on Plan 9
- github.com/lucas-clemente/quic-go (v0.16.2 -> v0.18.0):
  - create a milestone version for v0.18.x
  - add Changelog entries for v0.17 ([lucas-clemente/quic-go#2726](https://github.com/lucas-clemente/quic-go/pull/2726))
  - regenerate the testdata certificate with SAN instead of CommonName ([lucas-clemente/quic-go#2723](https://github.com/lucas-clemente/quic-go/pull/2723))
  - make it possible to use multiple qtls versions at the same time, add support for Go 1.15 ([lucas-clemente/quic-go#2720](https://github.com/lucas-clemente/quic-go/pull/2720))
  - add fuzzing for transport parameters ([lucas-clemente/quic-go#2713](https://github.com/lucas-clemente/quic-go/pull/2713))
  - run golangci-lint on GitHub Actions ([lucas-clemente/quic-go#2700](https://github.com/lucas-clemente/quic-go/pull/2700))
  - disallow values above 2^60 for Config.MaxIncoming{Uni}Streams ([lucas-clemente/quic-go#2711](https://github.com/lucas-clemente/quic-go/pull/2711))
  - never send a value larger than 2^60 in MAX_STREAMS frames ([lucas-clemente/quic-go#2710](https://github.com/lucas-clemente/quic-go/pull/2710))
  - run the check for go generated files on GitHub Actions instead of Travis ([lucas-clemente/quic-go#2703](https://github.com/lucas-clemente/quic-go/pull/2703))
  - update QUIC draft version information in README ([lucas-clemente/quic-go#2715](https://github.com/lucas-clemente/quic-go/pull/2715))
  - remove Fuzzit badge from README ([lucas-clemente/quic-go#2714](https://github.com/lucas-clemente/quic-go/pull/2714))
  - use the correct return values in Fuzz() functions ([lucas-clemente/quic-go#2705](https://github.com/lucas-clemente/quic-go/pull/2705))
  - simplify the connection, rename it to sendConn ([lucas-clemente/quic-go#2707](https://github.com/lucas-clemente/quic-go/pull/2707))
  - update qpack to v0.2.0 ([lucas-clemente/quic-go#2704](https://github.com/lucas-clemente/quic-go/pull/2704))
  - remove redundant error check in the stream ([lucas-clemente/quic-go#2718](https://github.com/lucas-clemente/quic-go/pull/2718))
  - put back the packet buffer when parsing the connection ID fails ([lucas-clemente/quic-go#2708](https://github.com/lucas-clemente/quic-go/pull/2708))
  - update fuzzing code for oss-fuzz ([lucas-clemente/quic-go#2702](https://github.com/lucas-clemente/quic-go/pull/2702))
  - fix travis script ([lucas-clemente/quic-go#2701](https://github.com/lucas-clemente/quic-go/pull/2701))
  - remove Fuzzit from Travis config ([lucas-clemente/quic-go#2699](https://github.com/lucas-clemente/quic-go/pull/2699))
  - add a script to check if go generated files are correct ([lucas-clemente/quic-go#2692](https://github.com/lucas-clemente/quic-go/pull/2692))
  - only arm the application data PTO timer after the handshake is confirmed ([lucas-clemente/quic-go#2689](https://github.com/lucas-clemente/quic-go/pull/2689))
  - fix tracing of congestion state updates ([lucas-clemente/quic-go#2691](https://github.com/lucas-clemente/quic-go/pull/2691))
  - fix reading of flag values in integration tests ([lucas-clemente/quic-go#2690](https://github.com/lucas-clemente/quic-go/pull/2690))
  - remove ACK decimation ([lucas-clemente/quic-go#2599](https://github.com/lucas-clemente/quic-go/pull/2599))
  - add a metric for PTOs ([lucas-clemente/quic-go#2686](https://github.com/lucas-clemente/quic-go/pull/2686))
  - remove the H3_EARLY_RESPONSE error ([lucas-clemente/quic-go#2687](https://github.com/lucas-clemente/quic-go/pull/2687))
  - implement tracing for congestion state changes ([lucas-clemente/quic-go#2684](https://github.com/lucas-clemente/quic-go/pull/2684))
  - remove the N connection simulation from the Reno code ([lucas-clemente/quic-go#2682](https://github.com/lucas-clemente/quic-go/pull/2682))
  - remove the SSLR (slow start large reduction) experiment ([lucas-clemente/quic-go#2680](https://github.com/lucas-clemente/quic-go/pull/2680))
  - remove unused connectionStats counters from the Reno implementation ([lucas-clemente/quic-go#2683](https://github.com/lucas-clemente/quic-go/pull/2683))
  - add an integration test that randomly sets tracers ([lucas-clemente/quic-go#2679](https://github.com/lucas-clemente/quic-go/pull/2679))
  - privatize some methods in the congestion controller package ([lucas-clemente/quic-go#2681](https://github.com/lucas-clemente/quic-go/pull/2681))
  - fix out-of-bounds read when creating a multiplexed tracer ([lucas-clemente/quic-go#2678](https://github.com/lucas-clemente/quic-go/pull/2678))
  - run integration tests with qlog and metrics on CircleCI ([lucas-clemente/quic-go#2677](https://github.com/lucas-clemente/quic-go/pull/2677))
  - add a metric for closed connections ([lucas-clemente/quic-go#2676](https://github.com/lucas-clemente/quic-go/pull/2676))
  - trace packets that are sent outside of a connection ([lucas-clemente/quic-go#2675](https://github.com/lucas-clemente/quic-go/pull/2675))
  - trace dropped packets that are dropped before they are passed to any session ([lucas-clemente/quic-go#2670](https://github.com/lucas-clemente/quic-go/pull/2670))
  - add a metric for sent packets ([lucas-clemente/quic-go#2673](https://github.com/lucas-clemente/quic-go/pull/2673))
  - add a metric for lost packets ([lucas-clemente/quic-go#2672](https://github.com/lucas-clemente/quic-go/pull/2672))
  - simplify the Tracer interface by combining the TracerFor... methods ([lucas-clemente/quic-go#2671](https://github.com/lucas-clemente/quic-go/pull/2671))
  - add a metrics package using OpenCensus, trace connections ([lucas-clemente/quic-go#2646](https://github.com/lucas-clemente/quic-go/pull/2646))
  - add a multiplexer for the tracer ([lucas-clemente/quic-go#2665](https://github.com/lucas-clemente/quic-go/pull/2665))
  - introduce a type for stateless reset tokens ([lucas-clemente/quic-go#2668](https://github.com/lucas-clemente/quic-go/pull/2668))
  - log all reasons why a connection is closed ([lucas-clemente/quic-go#2669](https://github.com/lucas-clemente/quic-go/pull/2669))
  - add integration tests using faulty packet conns ([lucas-clemente/quic-go#2663](https://github.com/lucas-clemente/quic-go/pull/2663))
  - don't block sendQueue.Send() if the runloop already exited. ([lucas-clemente/quic-go#2656](https://github.com/lucas-clemente/quic-go/pull/2656))
  - move the SupportedVersions slice out of the wire.Header ([lucas-clemente/quic-go#2664](https://github.com/lucas-clemente/quic-go/pull/2664))
  - add a flag to disable conn ID generation and the check for retired conn IDs ([lucas-clemente/quic-go#2660](https://github.com/lucas-clemente/quic-go/pull/2660))
  - put the session in the packet handler map directly (for client sessions) ([lucas-clemente/quic-go#2667](https://github.com/lucas-clemente/quic-go/pull/2667))
  - don't send write error in CONNECTION_CLOSE frames ([lucas-clemente/quic-go#2666](https://github.com/lucas-clemente/quic-go/pull/2666))
  - reset the PTO count before setting the timer when dropping a PN space ([lucas-clemente/quic-go#2657](https://github.com/lucas-clemente/quic-go/pull/2657))
  - enforce that a connection ID is not retired in a packet that uses that connection ID ([lucas-clemente/quic-go#2651](https://github.com/lucas-clemente/quic-go/pull/2651))
  - don't retire the conn ID that's in use when receiving a retransmission ([lucas-clemente/quic-go#2652](https://github.com/lucas-clemente/quic-go/pull/2652))
  - fix flaky cancelation integration test ([lucas-clemente/quic-go#2649](https://github.com/lucas-clemente/quic-go/pull/2649))
  - fix crash when the qlog callbacks returns a nil io.WriteCloser ([lucas-clemente/quic-go#2648](https://github.com/lucas-clemente/quic-go/pull/2648))
  - fix flaky server test on Travis ([lucas-clemente/quic-go#2645](https://github.com/lucas-clemente/quic-go/pull/2645))
  - fix a typo in the logging package test suite
  - introduce type aliases in the logging package ([lucas-clemente/quic-go#2643](https://github.com/lucas-clemente/quic-go/pull/2643))
  - rename frame fields to the names used in the draft ([lucas-clemente/quic-go#2644](https://github.com/lucas-clemente/quic-go/pull/2644))
  - split the qlog package into a logging and a qlog package, use a tracer interface in the quic.Config ([lucas-clemente/quic-go#2638](https://github.com/lucas-clemente/quic-go/pull/2638))
  - fix HTTP request writing if the Request.Body reads data and returns EOF ([lucas-clemente/quic-go#2642](https://github.com/lucas-clemente/quic-go/pull/2642))
  - handle Version Negotiation packets in the session ([lucas-clemente/quic-go#2640](https://github.com/lucas-clemente/quic-go/pull/2640))
  - increase the packet size of the client's Initial packet ([lucas-clemente/quic-go#2634](https://github.com/lucas-clemente/quic-go/pull/2634))
  - introduce an assertion in the server ([lucas-clemente/quic-go#2637](https://github.com/lucas-clemente/quic-go/pull/2637))
  - use the new qtls interface for (re)storing app data with a session state ([lucas-clemente/quic-go#2631](https://github.com/lucas-clemente/quic-go/pull/2631))
  - remove buffering of HTTP requests ([lucas-clemente/quic-go#2626](https://github.com/lucas-clemente/quic-go/pull/2626))
  - remove superfluous parameters logged when not doing 0-RTT ([lucas-clemente/quic-go#2632](https://github.com/lucas-clemente/quic-go/pull/2632))
  - return an infinite bandwidth if the RTT is zero ([lucas-clemente/quic-go#2636](https://github.com/lucas-clemente/quic-go/pull/2636))
  - drop support for Go 1.13 ([lucas-clemente/quic-go#2628](https://github.com/lucas-clemente/quic-go/pull/2628))
  - remove superfluos handleResetStreamFrame method on the stream ([lucas-clemente/quic-go#2623](https://github.com/lucas-clemente/quic-go/pull/2623))
  - implement a token-bucket pacing algorithm ([lucas-clemente/quic-go#2615](https://github.com/lucas-clemente/quic-go/pull/2615))
  - gracefully handle concurrent stream writes and cancellations ([lucas-clemente/quic-go#2624](https://github.com/lucas-clemente/quic-go/pull/2624))
  - log sent packets right before sending them out ([lucas-clemente/quic-go#2613](https://github.com/lucas-clemente/quic-go/pull/2613))
  - remove unused packet counter in the receivedPacketTracker ([lucas-clemente/quic-go#2611](https://github.com/lucas-clemente/quic-go/pull/2611))
  - rewrite the proxy to avoid packet reordering ([lucas-clemente/quic-go#2617](https://github.com/lucas-clemente/quic-go/pull/2617))
  - fix flaky INVALID_TOKEN integration test ([lucas-clemente/quic-go#2610](https://github.com/lucas-clemente/quic-go/pull/2610))
  - make DialEarly return EarlySession ([lucas-clemente/quic-go#2621](https://github.com/lucas-clemente/quic-go/pull/2621))
  - add debug logging to the packet handler map ([lucas-clemente/quic-go#2608](https://github.com/lucas-clemente/quic-go/pull/2608))
  - increase the minimum pacing delay to 1ms ([lucas-clemente/quic-go#2605](https://github.com/lucas-clemente/quic-go/pull/2605))
- github.com/marten-seemann/qpack (v0.1.0 -> v0.2.0):
  - don't reuse the encoder in the integration tests ([marten-seemann/qpack#18](https://github.com/marten-seemann/qpack/pull/18))
  - use Huffman encoding for field names and values ([marten-seemann/qpack#16](https://github.com/marten-seemann/qpack/pull/16))
  - add more tests for encoding using the static table ([marten-seemann/qpack#15](https://github.com/marten-seemann/qpack/pull/15))
  - Encoder uses the static table. ([marten-seemann/qpack#10](https://github.com/marten-seemann/qpack/pull/10))
  - add gofmt to golangci-lint
  - update qifs to the current version ([marten-seemann/qpack#14](https://github.com/marten-seemann/qpack/pull/14))
  - use golangci-lint for linting ([marten-seemann/qpack#12](https://github.com/marten-seemann/qpack/pull/12))
  - add fuzzing ([marten-seemann/qpack#9](https://github.com/marten-seemann/qpack/pull/9))
  - update qifs
  - use https protocol for submodule clone ([marten-seemann/qpack#7](https://github.com/marten-seemann/qpack/pull/7))
- github.com/marten-seemann/qtls (v0.9.1 -> v0.10.0):
  - add callbacks to store and restore app data along a session state
  - remove support for Go 1.13
- github.com/marten-seemann/qtls-go1-15 (null -> v0.1.0):
  - use a prefix for client session cache keys
  - add callbacks to store and restore app data along a session state
  - don't use TLS 1.3 compatibility mode when using alternative record layer
  - delete the session ticket after attempting 0-RTT
  - reject 0-RTT when a different ALPN is chosen
  - encode the ALPN into the session ticket
  - add a field to the ConnectionState to tell if 0-RTT was used
  - add a callback to tell the client about rejection of 0-RTT
  - don't offer 0-RTT after a HelloRetryRequest
  - add Accept0RTT to Config callback to decide if 0-RTT should be accepted
  - add the option to encode application data into the session ticket
  - export the 0-RTT write key
  - abuse the nonce field of ClientSessionState to save max_early_data_size
  - export the 0-RTT read key
  - close connection if client attempts 0-RTT, but ticket didn't allow it
  - encode the max early data size into the session ticket
  - implement parsing of the early_data extension in the EncryptedExtensions
  - add a tls.Config.MaxEarlyData option to enable 0-RTT
  - accept TLS 1.3 cipher suites in Config.CipherSuites
  - introduce a function on the connection to generate a session ticket
  - add a config option to enforce selection of an application protocol
  - export Conn.HandlePostHandshakeMessage
  - export Alert
  - reject Configs that set MaxVersion < 1.3 when using a record layer
  - enforce TLS 1.3 when using an alternative record layer
- github.com/multiformats/go-multiaddr (v0.2.2 -> v0.3.1):
  - dep: add "codependencies" for handling version conflicts ([multiformats/go-multiaddr#132](https://github.com/multiformats/go-multiaddr/pull/132))
  - Support /p2p addresses encoded as CIDs ([multiformats/go-multiaddr#130](https://github.com/multiformats/go-multiaddr/pull/130))
  - Merge go-multiaddr-net
- github.com/multiformats/go-multiaddr-net (v0.1.5 -> v0.2.0):
  - Deprecate ([multiformats/go-multiaddr-net#72](https://github.com/multiformats/go-multiaddr-net/pull/72))
- github.com/multiformats/go-multihash (v0.0.13 -> v0.0.14):
  - fix: only register one blake2s length ([multiformats/go-multihash#129](https://github.com/multiformats/go-multihash/pull/129))
  - feat: add two filecoin hashes, without Sum() implementations ([multiformats/go-multihash#128](https://github.com/multiformats/go-multihash/pull/128))
  - feat: reduce blake2b allocations by special-casing the 256/512 variants ([multiformats/go-multihash#126](https://github.com/multiformats/go-multihash/pull/126))
- github.com/multiformats/go-multistream (v0.1.1 -> v0.1.2):
  - upgrade deps + interoperable varints. (#51) ([multiformats/go-multistream#51](https://github.com/multiformats/go-multistream/pull/51))
- github.com/multiformats/go-varint (v0.0.5 -> v0.0.6):
  - fix minor interoperability issues. (#6) ([multiformats/go-varint#6](https://github.com/multiformats/go-varint/pull/6))
- github.com/warpfork/go-wish (v0.0.0-20190328234359-8b3e70f8e830 -> v0.0.0-20200122115046-b9ea61034e4a):
  - Add ShouldBeSameTypeAs checker.
  - Integration test update for go versions.
- github.com/whyrusleeping/cbor-gen (v0.0.0-20200123233031-1cdf64d27158 -> v0.0.0-20200402171437-3d27c146c105):
  - Handle Nil values for cbg.Deferred ([whyrusleeping/cbor-gen#14](https://github.com/whyrusleeping/cbor-gen/pull/14))
  - add name of struct field to error messages
  - Support uint64 pointers ([whyrusleeping/cbor-gen#13](https://github.com/whyrusleeping/cbor-gen/pull/13))
  - int64 support in map encoders ([whyrusleeping/cbor-gen#12](https://github.com/whyrusleeping/cbor-gen/pull/12))
  - Fix uint64 typed array gen ([whyrusleeping/cbor-gen#10](https://github.com/whyrusleeping/cbor-gen/pull/10))
  - Fix cbg self referencing import path ([whyrusleeping/cbor-gen#8](https://github.com/whyrusleeping/cbor-gen/pull/8))

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 156 | +16428/-42621 | 979 |
| hannahhoward | 42 | +15132/-9819 | 467 |
| Eric Myhre | 114 | +13709/-6898 | 586 |
| Steven Allen | 55 | +1211/-2714 | 95 |
| Adin Schmahmann | 54 | +1660/-783 | 117 |
| Petar Maymounkov | 23 | +1677/-671 | 75 |
| Aarsh Shah | 10 | +1926/-341 | 39 |
| Raúl Kripalani | 17 | +1134/-537 | 53 |
| Will | 1 | +841/-0 | 9 |
| rendaw | 3 | +425/-195 | 12 |
| Will Scott | 8 | +302/-229 | 15 |
| vyzo | 22 | +345/-166 | 23 |
| Fazlul Shahriar | 7 | +452/-44 | 19 |
| Peter Rabbitson | 1 | +353/-118 | 5 |
| Hector Sanjuan | 10 | +451/-3 | 14 |
| Marcin Rataj | 9 | +298/-106 | 16 |
| Łukasz Magiera | 4 | +329/-51 | 12 |
| RubenKelevra | 9 | +331/-7 | 12 |
| Michael Muré | 2 | +259/-69 | 6 |
| jstordeur | 1 | +252/-2 | 5 |
| Diederik Loerakker | 1 | +168/-35 | 7 |
| Tiger | 3 | +138/-52 | 8 |
| Kevin Neaton | 3 | +103/-21 | 9 |
| Rod Vagg | 1 | +50/-40 | 4 |
| Oli Evans | 4 | +60/-9 | 6 |
| achingbrain | 4 | +30/-30 | 5 |
| Cyril Fougeray | 2 | +34/-24 | 2 |
| Luke Tucker | 1 | +31/-1 | 2 |
| sandman | 2 | +23/-7 | 3 |
| Alan Shaw | 1 | +18/-9 | 2 |
| Jacob Heun | 4 | +13/-3 | 4 |
| Jessica Schilling | 3 | +7/-7 | 3 |
| Rafael Ramalho | 4 | +9/-4 | 4 |
| Jeromy Johnson | 2 | +6/-6 | 4 |
| Nick Cabatoff | 1 | +7/-2 | 1 |
| Stephen Solka | 1 | +1/-7 | 1 |
| Preston Van Loon | 2 | +6/-2 | 2 |
| Jakub Sztandera | 2 | +5/-2 | 2 |
| llx | 1 | +3/-3 | 1 |
| Adrian Lanzafame | 1 | +3/-3 | 1 |
| Yusef Napora | 1 | +3/-2 | 1 |
| Louis Thibault | 1 | +5/-0 | 1 |
| Martín Triay | 1 | +4/-0 | 1 |
| Hlib | 1 | +2/-2 | 1 |
| Shotaro Yamada | 1 | +2/-1 | 1 |
| phuslu | 1 | +1/-1 | 1 |
| Zero King | 1 | +1/-1 | 1 |
| Rüdiger Klaehn | 1 | +2/-0 | 1 |
| Nex | 1 | +1/-1 | 1 |
| Mark Gaiser | 1 | +1/-1 | 1 |
| Luflosi | 1 | +1/-1 | 1 |
| David Florness | 1 | +1/-1 | 1 |
| Dean Eigenmann | 1 | +0/-1 | 1 |


# go-ipfs changelog v0.8

## v0.8.0 2021-02-18

We're happy to announce go-ipfs 0.8.0! This is planned to be a fairly small release focused on integrating in the new pinning service/remote pinning [API](https://github.com/ipfs/pinning-services-api-spec) that makes the experience of managing pins across pinning services easier and more uniform.

### 🔦 Highlights

#### 🧷 Remote pinning services

There is now support for asking remote services to pin data for you. This means anyone can implement the [spec](https://ipfs.github.io/pinning-services-api-spec/) (developed in this [repo](https://github.com/ipfs/pinning-services-api-spec)) and allow for pin management.

All of the CLI (and corresponding HTTP API) commands are available under `ipfs pin remote`.

This remote pinning service comes with a redesign of how we're thinking about pinning and includes some commonly requested features such as:
- Pins can have names (and coming soon metadata)
- The same content can be pinned multiple times, but of course stored only once
  - This allows applications using the same pinning service to manage their own pins without worrying about removing content important to another application
- Data can be pinned in either the foreground or background


Examples include:
```
ipfs pin remote service add myservice https://myservice.tld:1234/api/path myaccess key

ipfs pin remote add /ipfs/bafymydata --service=myservice --name=myfile
ipfs pin remote ls --service=myservice --name=myfile
ipfs pin remote ls --service=myservice --cid=bafymydata
ipfs pin remote rm --serivce=myservice --name=myfile
```
A few notes:

Remote pinning services work with recursive pins. This means commands like `ipfs pin remote ls` will not list indirectly pinned CIDs.

While pinning service data is stored in the configuration file it cannot be edited directly via the `ipfs config` commands due to the sensitive nature of pinning service API keys. The `ipfs pin remote service` commands can be used for interacting with remote service settings.

#### 📌 Faster local pinning and unpinning

The pinning subsystem has been redesigned to be much faster and more flexible in how it tracks pins. For users who are working with many pins this will lead to a big speed increase in listing and modifying the set of pinned items as well as decreased memory usage.

Part of the redesign was setup to account for being able to interact with local pins the same way we can now interact with remote pins (e.g. names, being allowed to pin the same CID multiple times, etc.). Keep posted for more improvements to pinning.

#### DNSLink names on https:// subdomains

Previously DNSLink names would have trouble loading over subdomain gateways with HTTPS support since there is no way to get multilevel wildcard certificates (e.g. `en.wikipedia-on-ipfs.org.ipns.dweb.link` cannot be covered by `*.ipns.dweb.link`). Therefore, when trying to load DNSLink names over https:// subdomains go-ipfs we now forward to an encoded DNS name. Since DNS names cannot contain `.` in them they are escaped using `-`.

`/ipns/en.wikipedia-on-ipfs.org` →
`ipns://en.wikipedia-on-ipfs.org`  →
`https://dweb.link/ipns/en.wikipedia-on-ipfs.org`
`https://en-wikipedia--on--ipfs-org.ipns.dweb.link` :point_left: _a single DNS label, no TLS error_

#### QUIC update

QUIC support has received a number of upgrades, including the ability to take advantage of larger UDP receive buffers for increased performance.

Linux users may notice a logged error on daemon startup if your system needs extra configuration to allow IPFS increase the buffer size. A helpful link for resolving this is in the log message as well as [here](https://github.com/lucas-clemente/quic-go/wiki/UDP-Receive-Buffer-Size).

#### 👋 No more Darwin 386 builds

Go 1.15 (the latest version of Go) [no longer supports](https://github.com/golang/go/issues/34749) Darwin 386 and so we are dropping support as well.

### Changelog

- github.com/ipfs/go-ipfs:
  - Release v0.8.0
  - docs: RepinInterval
  - style: docs/config.md
  - style: improved MFS PinName example
  - docs: Pinning.RemoteServices.Policies
  - fix: decrease log level of opencensus initialization ([ipfs/go-ipfs#7815](https://github.com/ipfs/go-ipfs/pull/7815))
  - Register oc metrics ([ipfs/go-ipfs#7593](https://github.com/ipfs/go-ipfs/pull/7593))
  - add remote pinning to ipfs command (#7661) ([ipfs/go-ipfs#7661](https://github.com/ipfs/go-ipfs/pull/7661))
  - More p2p proxy checks ([ipfs/go-ipfs#7797](https://github.com/ipfs/go-ipfs/pull/7797))
  - Use datastore based pinning ([ipfs/go-ipfs#7750](https://github.com/ipfs/go-ipfs/pull/7750))
  - fix: return an error when an unknown object type is passed ([ipfs/go-ipfs#7795](https://github.com/ipfs/go-ipfs/pull/7795))
  - clarify why ipfs file ls is being deprecated ([ipfs/go-ipfs#7755](https://github.com/ipfs/go-ipfs/pull/7755))
  - fix: ipfs dag export uses the CoreAPI and respects the offline flag ([ipfs/go-ipfs#7753](https://github.com/ipfs/go-ipfs/pull/7753))
  - return an error when trying to download fs-repo-migrations for linux + musl ([ipfs/go-ipfs#7735](https://github.com/ipfs/go-ipfs/pull/7735))
  - fix: do not create a new (unused) peerID when initializing from config ([ipfs/go-ipfs#7730](https://github.com/ipfs/go-ipfs/pull/7730))
  - docs: Add a link in config.md ([ipfs/go-ipfs#7780](https://github.com/ipfs/go-ipfs/pull/7780))
  - update libp2p for stream closure refactor ([ipfs/go-ipfs#7747](https://github.com/ipfs/go-ipfs/pull/7747))
  - Fix typo in ipfs dag stat command ([ipfs/go-ipfs#7761](https://github.com/ipfs/go-ipfs/pull/7761))
  - docs(readme): key rotation in docker (#7721) ([ipfs/go-ipfs#7721](https://github.com/ipfs/go-ipfs/pull/7721))
  - fix(dnslink-gw): breadcrumbs and CID column when dir listing ([ipfs/go-ipfs#7699](https://github.com/ipfs/go-ipfs/pull/7699))
  - fix(gw): preserve query on website redirect ([ipfs/go-ipfs#7727](https://github.com/ipfs/go-ipfs/pull/7727))
  - feat: ipfs-webui v2.11.4 ([ipfs/go-ipfs#7716](https://github.com/ipfs/go-ipfs/pull/7716))
  - docs: how the ipfs snap is built and published ([ipfs/go-ipfs#7725](https://github.com/ipfs/go-ipfs/pull/7725))
  - fix: webui on ipv6 localhost ([ipfs/go-ipfs#7731](https://github.com/ipfs/go-ipfs/pull/7731))
  - Add missing plugin support on FreeBSD ([ipfs/go-ipfs#7722](https://github.com/ipfs/go-ipfs/pull/7722))
  - fix error when computing coverage ([ipfs/go-ipfs#7726](https://github.com/ipfs/go-ipfs/pull/7726))
  - docs(config): X-Forwarded-Host ([ipfs/go-ipfs#7651](https://github.com/ipfs/go-ipfs/pull/7651))
  - chore: webui v2.11.2 ([ipfs/go-ipfs#7703](https://github.com/ipfs/go-ipfs/pull/7703))
  - Add task for updating CLI docs right after updating the HTTP-api docs ([ipfs/go-ipfs#7711](https://github.com/ipfs/go-ipfs/pull/7711))
  - feat(gateway): Content-Disposition improvements ([ipfs/go-ipfs#7677](https://github.com/ipfs/go-ipfs/pull/7677))
  - fix build on Plan 9 ([ipfs/go-ipfs#7690](https://github.com/ipfs/go-ipfs/pull/7690))
  - docs: update changelog for v0.7.0
  - chore: bump webui version
  - fix: remove the (empty) alias for --peerid-base
  - fix: use override GOFLAGS changes from 480defab689610550ee3d346e31441a2bb881fcb but keep trimpath usage as is
  - Revert "fix: override GOFLAGS"
  - Fix --ipns-base alias ([ipfs/go-ipfs#7659](https://github.com/ipfs/go-ipfs/pull/7659))
  - docs: update config to indicate SECIO deprecation ([ipfs/go-ipfs#7630](https://github.com/ipfs/go-ipfs/pull/7630))
  - fix: ipfs dht put/get commands with peerIDs encoded as CIDs ([ipfs/go-ipfs#7633](https://github.com/ipfs/go-ipfs/pull/7633))
  - update version to 0.8.0-dev ([ipfs/go-ipfs#7629](https://github.com/ipfs/go-ipfs/pull/7629))
- github.com/ipfs/go-bitswap (v0.2.20 -> v0.3.3):
  - feat: configurable engine blockstore worker count (#449) ([ipfs/go-bitswap#449](https://github.com/ipfs/go-bitswap/pull/449))
  - fix: set the score ledger on start ([ipfs/go-bitswap#447](https://github.com/ipfs/go-bitswap/pull/447))
  - feat: update for go-libp2p-core 0.7.0 interface changes ([ipfs/go-bitswap#445](https://github.com/ipfs/go-bitswap/pull/445))
  - fix: guard access to the mock wiretap with a lock ([ipfs/go-bitswap#446](https://github.com/ipfs/go-bitswap/pull/446))
  - Add WireTap interface (#444) ([ipfs/go-bitswap#444](https://github.com/ipfs/go-bitswap/pull/444))
  - Fix: Increment stats.MessagesSent in msgToStream() function (#441) ([ipfs/go-bitswap#441](https://github.com/ipfs/go-bitswap/pull/441))
  - refactor: remove extraneous ledger field init (#437) ([ipfs/go-bitswap#437](https://github.com/ipfs/go-bitswap/pull/437))
  - Added `WithScoreLedger` Bitswap option  (#430) ([ipfs/go-bitswap#430](https://github.com/ipfs/go-bitswap/pull/430))
- github.com/ipfs/go-blockservice (v0.1.3 -> v0.1.4):
  - Avoid modifying passed in slice of cids ([ipfs/go-blockservice#65](https://github.com/ipfs/go-blockservice/pull/65))
- github.com/ipfs/go-ds-badger (v0.2.4 -> v0.2.6):
  - Log error if batch not committed or canceled ([ipfs/go-ds-badger#108](https://github.com/ipfs/go-ds-badger/pull/108))
  -  Add Cancel function; add finalizer to cleanup abandoned batch ([ipfs/go-ds-badger#105](https://github.com/ipfs/go-ds-badger/pull/105))
  - Do not implement batches using transactions ([ipfs/go-ds-badger#104](https://github.com/ipfs/go-ds-badger/pull/104))
  - readme: add information on Badger2 datastore ([ipfs/go-ds-badger#102](https://github.com/ipfs/go-ds-badger/pull/102))
  - update contributing link ([ipfs/go-ds-badger#91](https://github.com/ipfs/go-ds-badger/pull/91))
  - Use current go-log (#89) ([ipfs/go-ds-badger#89](https://github.com/ipfs/go-ds-badger/pull/89))
- github.com/ipfs/go-graphsync (v0.1.1 -> v0.6.0):
  - docs(CHANGELOG): revise for 0.6.0
  - Merge branch 'master' into release/v0.6.0
  - docs(CHANGELOG): update for 0.6.0 release
  - move block allocation into message queue (#140) ([ipfs/go-graphsync#140](https://github.com/ipfs/go-graphsync/pull/140))
  - Response Assembler Refactor (#138) ([ipfs/go-graphsync#138](https://github.com/ipfs/go-graphsync/pull/138))
  - Add error listener on receiver (#136) ([ipfs/go-graphsync#136](https://github.com/ipfs/go-graphsync/pull/136))
  - Run testplan on in CI (#137) ([ipfs/go-graphsync#137](https://github.com/ipfs/go-graphsync/pull/137))
  - fix(responsemanager): fix network error propagation (#133) ([ipfs/go-graphsync#133](https://github.com/ipfs/go-graphsync/pull/133))
  - testground test for graphsync (#132) ([ipfs/go-graphsync#132](https://github.com/ipfs/go-graphsync/pull/132))
  - docs(CHANGELOG): update for v0.5.2 ([ipfs/go-graphsync#130](https://github.com/ipfs/go-graphsync/pull/130))
  - RegisterNetworkErrorListener should fire when there's an error connecting to the peer (#127) ([ipfs/go-graphsync#127](https://github.com/ipfs/go-graphsync/pull/127))
  - Permit multiple data subscriptions per original topic (#128) ([ipfs/go-graphsync#128](https://github.com/ipfs/go-graphsync/pull/128))
  - release: v0.5.1 (#123) ([ipfs/go-graphsync#123](https://github.com/ipfs/go-graphsync/pull/123))
  - feat(responsemanager): allow configuration of max requests (#122) ([ipfs/go-graphsync#122](https://github.com/ipfs/go-graphsync/pull/122))
  - docs(CHANGELOG): update for 0.5.0 ([ipfs/go-graphsync#120](https://github.com/ipfs/go-graphsync/pull/120))
  - feat: use go-libp2p-core 0.7.0 stream interfaces (#116) ([ipfs/go-graphsync#116](https://github.com/ipfs/go-graphsync/pull/116))
  - Merge branch 'release/v0.4.3'
  - chore(benchmarks): remove extra files
  - fix(peerresponsemanager): avoid race condition that could result in NPE in link tracker (#118) ([ipfs/go-graphsync#118](https://github.com/ipfs/go-graphsync/pull/118))
  - docs(CHANGELOG): update for 0.4.2 ([ipfs/go-graphsync#117](https://github.com/ipfs/go-graphsync/pull/117))
  - feat(memory): improve memory usage (#110) ([ipfs/go-graphsync#110](https://github.com/ipfs/go-graphsync/pull/110))
  - fix(notifications): fix lock in close (#115) ([ipfs/go-graphsync#115](https://github.com/ipfs/go-graphsync/pull/115))
  - docs(CHANGELOG): update for v0.4.1 ([ipfs/go-graphsync#114](https://github.com/ipfs/go-graphsync/pull/114))
  - fix(allocator): remove peer from peer status list
  - docs(CHANGELOG): update for v0.4.0
  - docs(CHANGELOG): update for 0.3.1 ([ipfs/go-graphsync#112](https://github.com/ipfs/go-graphsync/pull/112))
  - Add allocator for memory backpressure (#108) ([ipfs/go-graphsync#108](https://github.com/ipfs/go-graphsync/pull/108))
  - Shutdown notifications go routines (#109) ([ipfs/go-graphsync#109](https://github.com/ipfs/go-graphsync/pull/109))
  - Switch to google protobuf generator (#105) ([ipfs/go-graphsync#105](https://github.com/ipfs/go-graphsync/pull/105))
  - feat(CHANGELOG): update for 0.3.0 ([ipfs/go-graphsync#104](https://github.com/ipfs/go-graphsync/pull/104))
  - docs(CHANGELOG): update for 0.2.1 ([ipfs/go-graphsync#103](https://github.com/ipfs/go-graphsync/pull/103))
  - Track actual network operations in a response (#102) ([ipfs/go-graphsync#102](https://github.com/ipfs/go-graphsync/pull/102))
  - feat(responsecache): prune blocks more intelligently (#101) ([ipfs/go-graphsync#101](https://github.com/ipfs/go-graphsync/pull/101))
  - Release/0.2.0 ([ipfs/go-graphsync#99](https://github.com/ipfs/go-graphsync/pull/99))
  - fix(metadata): fix cbor-gen (#98) ([ipfs/go-graphsync#98](https://github.com/ipfs/go-graphsync/pull/98))
  - fix(selectorvalidator): memory optimization (#97) ([ipfs/go-graphsync#97](https://github.com/ipfs/go-graphsync/pull/97))
  - Update go-ipld-prime@v0.5.0 (#92) ([ipfs/go-graphsync#92](https://github.com/ipfs/go-graphsync/pull/92))
  - refactor(metadata): use cbor-gen encoding (#96) ([ipfs/go-graphsync#96](https://github.com/ipfs/go-graphsync/pull/96))
  - Release/v0.1.2 ([ipfs/go-graphsync#95](https://github.com/ipfs/go-graphsync/pull/95))
  - Return Request context cancelled error (#93) ([ipfs/go-graphsync#93](https://github.com/ipfs/go-graphsync/pull/93))
  - feat(benchmarks): add p2p stress test (#91) ([ipfs/go-graphsync#91](https://github.com/ipfs/go-graphsync/pull/91))
  - Benchmark framework + First memory fixes (#89) ([ipfs/go-graphsync#89](https://github.com/ipfs/go-graphsync/pull/89))
  - docs(CHANGELOG): update for v0.1.1 ([ipfs/go-graphsync#85](https://github.com/ipfs/go-graphsync/pull/85))
- github.com/ipfs/go-ipfs-cmds (v0.4.0 -> v0.6.0):
  - Added DelimitedStringsOption for enabling delimited strings on the CLI ([ipfs/go-ipfs-cmds#204](https://github.com/ipfs/go-ipfs-cmds/pull/204))
  - feat: support strings option over HTTP API ([ipfs/go-ipfs-cmds#203](https://github.com/ipfs/go-ipfs-cmds/pull/203))
- github.com/ipfs/go-ipfs-config (v0.9.0 -> v0.12.0):
  - add support for pinning mfs (#116) ([ipfs/go-ipfs-config#116](https://github.com/ipfs/go-ipfs-config/pull/116))
  - add remote pinning services config ([ipfs/go-ipfs-config#113](https://github.com/ipfs/go-ipfs-config/pull/113))
  - Remove badger2 profile ([ipfs/go-ipfs-config#115](https://github.com/ipfs/go-ipfs-config/pull/115))
  - Add badger2 profile and config spec
- github.com/ipfs/go-ipfs-pinner (v0.0.4 -> v0.1.1):
  - Avoid loading all pins into memory during migration (#5) ([ipfs/go-ipfs-pinner#5](https://github.com/ipfs/go-ipfs-pinner/pull/5))
  - Datastore based pinner (#4) ([ipfs/go-ipfs-pinner#4](https://github.com/ipfs/go-ipfs-pinner/pull/4))
- github.com/ipfs/go-ipld-cbor (v0.0.4 -> v0.0.5):
  - add the ability to leverage zero-copy on blockstores. (#75) ([ipfs/go-ipld-cbor#75](https://github.com/ipfs/go-ipld-cbor/pull/75))
  - ipldstore: Also wrap Put serialization errors ([ipfs/go-ipld-cbor#74](https://github.com/ipfs/go-ipld-cbor/pull/74))
  - add helper constructor for inmem cbor store
  - docs: add comments describing methods & interfaces ([ipfs/go-ipld-cbor#71](https://github.com/ipfs/go-ipld-cbor/pull/71))
- github.com/ipfs/go-path (v0.0.8 -> v0.0.9):
  - fix: improved error message on broken CIDv0 ([ipfs/go-path#33](https://github.com/ipfs/go-path/pull/33))
- github.com/ipfs/go-pinning-service-http-client (null -> v0.1.0):
  - feat: LsBatchSync to fetch single batch of results ([ipfs/go-pinning-service-http-client#6](https://github.com/ipfs/go-pinning-service-http-client/pull/6))
  - Initial Implementation ([ipfs/go-pinning-service-http-client#1](https://github.com/ipfs/go-pinning-service-http-client/pull/1))
- github.com/ipld/go-car (v0.1.1-0.20200429200904-c222d793c339 -> v0.1.1-0.20201015032735-ff6ccdc46acc):
  - Update ipld libs ([ipld/go-car#35](https://github.com/ipld/go-car/pull/35))
- github.com/ipld/go-ipld-prime (v0.0.2-0.20200428162820-8b59dc292b8e -> v0.5.1-0.20201021195245-109253e8a018):
  - Merge branch 'codec-hardening'
  - Add fluent.MustReflect convenience method.
  - codegen: make error info available when tuples process data that is too long. ([ipld/go-ipld-prime#99](https://github.com/ipld/go-ipld-prime/pull/99))
  - Merge branch 'codegen-typofixes'
  - Implement resource budgets in dagcbor parsing. ([ipld/go-ipld-prime#85](https://github.com/ipld/go-ipld-prime/pull/85))
  - Codegen for links should emit the methods to conform to the schema.TypedLinkNode interface where applicable. ([ipld/go-ipld-prime#91](https://github.com/ipld/go-ipld-prime/pull/91))
  - Introduce fluent.Reflect convenience functions. ([ipld/go-ipld-prime#81](https://github.com/ipld/go-ipld-prime/pull/81))
  - schema/gen/go: make all top-level tests parallel
  - all: don't use buffers where readers suffice
  - fix typo in documentation
  - schema-schema codegen demo now includes unmarshal exercise ([ipld/go-ipld-prime#76](https://github.com/ipld/go-ipld-prime/pull/76))
  - Update tests for unions; several fixes ([ipld/go-ipld-prime#75](https://github.com/ipld/go-ipld-prime/pull/75))
  - New testcase system for exercising typed nodes; Revamp struct tests to use it. ([ipld/go-ipld-prime#66](https://github.com/ipld/go-ipld-prime/pull/66))
  - small docs fixes on an internal component.
  - Fix formatting in README.
  - fix(cidlink): check for byte buffer ([ipld/go-ipld-prime#70](https://github.com/ipld/go-ipld-prime/pull/70))
  - linking/cid: check a previously unused error ([ipld/go-ipld-prime#68](https://github.com/ipld/go-ipld-prime/pull/68))
  - all: make 'go test ./...' pass on Go 1.15 ([ipld/go-ipld-prime#67](https://github.com/ipld/go-ipld-prime/pull/67))
  - Merge branch 'kinded-union-gen'
  - Add traversal.Get function ([ipld/go-ipld-prime#65](https://github.com/ipld/go-ipld-prime/pull/65))
  - Kinded union gen ([ipld/go-ipld-prime#64](https://github.com/ipld/go-ipld-prime/pull/64))
  - Struct tuple representation codegen ([ipld/go-ipld-prime#63](https://github.com/ipld/go-ipld-prime/pull/63))
  - Merge branch 'moar-codegen'
  - Self-hosting gen of the schema-schema. ([ipld/go-ipld-prime#62](https://github.com/ipld/go-ipld-prime/pull/62))
  - Codegen: approaching self-host ([ipld/go-ipld-prime#61](https://github.com/ipld/go-ipld-prime/pull/61))
  - Codegen of unions, and their keyed representations ([ipld/go-ipld-prime#60](https://github.com/ipld/go-ipld-prime/pull/60))
  - mark v0.5
  - API updates for v0.5: the renamening ([ipld/go-ipld-prime#59](https://github.com/ipld/go-ipld-prime/pull/59))
  - mark v0.4
  - changelog: note the codegen work.
  - Codegen update -- Assemblers, and many new representations ([ipld/go-ipld-prime#52](https://github.com/ipld/go-ipld-prime/pull/52))
  - Merge branch 'json-tables-codec'
  - Merge branch 'docs-updates'
  - Introduce changelog!
  - Add examples of creating and loading links.
- github.com/ipld/go-ipld-prime-proto (v0.0.0-20200428191222-c1ffdadc01e1 -> v0.1.0):
  - Update go-ipld-prime ([ipld/go-ipld-prime-proto#6](https://github.com/ipld/go-ipld-prime-proto/pull/6))
  - feat(coding use -1 instead of 0):
  - Update ipld prime, use proper code-gen ([ipld/go-ipld-prime-proto#5](https://github.com/ipld/go-ipld-prime-proto/pull/5))
  - Updates to dependencies ([ipld/go-ipld-prime-proto#4](https://github.com/ipld/go-ipld-prime-proto/pull/4))
  - Check for byte buffer on decode ([ipld/go-ipld-prime-proto#3](https://github.com/ipld/go-ipld-prime-proto/pull/3))
- github.com/libp2p/go-libp2p (v0.11.0 -> v0.13.0):
  - use a context when opening streams ([libp2p/go-libp2p#1033](https://github.com/libp2p/go-libp2p/pull/1033))
  - fix: obey new stream timeout ([libp2p/go-libp2p#1029](https://github.com/libp2p/go-libp2p/pull/1029))
  - feat: update to go-libp2p-core 0.7.0 interface changes ([libp2p/go-libp2p#1001](https://github.com/libp2p/go-libp2p/pull/1001))
  - Basic Connection Gater Implementation ([libp2p/go-libp2p#1005](https://github.com/libp2p/go-libp2p/pull/1005))
  - Fixed bug for inbound connections gated by the deprecated filter option (#1004) ([libp2p/go-libp2p#1004](https://github.com/libp2p/go-libp2p/pull/1004))
- github.com/libp2p/go-libp2p-autonat (v0.3.2 -> v0.4.0):
  - feat: update to go-libp2p-core 0.7.0 ([libp2p/go-libp2p-autonat#97](https://github.com/libp2p/go-libp2p-autonat/pull/97))
- github.com/libp2p/go-libp2p-circuit (v0.3.1 -> v0.4.0):
  - feat: update to go-libp2p-core 0.7.0 ([libp2p/go-libp2p-circuit#123](https://github.com/libp2p/go-libp2p-circuit/pull/123))
- github.com/libp2p/go-libp2p-core (v0.6.1 -> v0.8.0):
  - add a context to OpenStream and NewStream (#172) ([libp2p/go-libp2p-core#172](https://github.com/libp2p/go-libp2p-core/pull/172))
  - sec/insecure/insecure.go: Fix typo (#167) ([libp2p/go-libp2p-core#167](https://github.com/libp2p/go-libp2p-core/pull/167))
  - add CloseRead/CloseWrite on streams (#166) ([libp2p/go-libp2p-core#166](https://github.com/libp2p/go-libp2p-core/pull/166))
  - Fix typo in docs (#163) ([libp2p/go-libp2p-core#163](https://github.com/libp2p/go-libp2p-core/pull/163))
- github.com/libp2p/go-libp2p-gostream (v0.2.1 -> v0.3.0):
  - feat: use go-libp2p-core 0.7.0 stream interfaces ([libp2p/go-libp2p-gostream#60](https://github.com/libp2p/go-libp2p-gostream/pull/60))
- github.com/libp2p/go-libp2p-http (v0.1.5 -> v0.2.0):
  - Fix var name in README ([libp2p/go-libp2p-http#63](https://github.com/libp2p/go-libp2p-http/pull/63))
  - Fix var name in doc ([libp2p/go-libp2p-http#62](https://github.com/libp2p/go-libp2p-http/pull/62))
- github.com/libp2p/go-libp2p-kad-dht (v0.9.0 -> v0.11.1):
  - Fix constructor ordering ([libp2p/go-libp2p-kad-dht#698](https://github.com/libp2p/go-libp2p-kad-dht/pull/698))
  - feat: update to go-libp2p-core 0.7.0 ([libp2p/go-libp2p-kad-dht#693](https://github.com/libp2p/go-libp2p-kad-dht/pull/693))
  - Run fixLowPeers on startup ([libp2p/go-libp2p-kad-dht#694](https://github.com/libp2p/go-libp2p-kad-dht/pull/694))
  - feat: add advanced V1ProtocolOverride option to be used by legacy networks
  - feat: remove dht v2 as it's not actually in use and could be confusing
- github.com/libp2p/go-libp2p-mplex (v0.2.4 -> v0.4.1):
  - update go-mplex, use the context passed to OpenStream ([libp2p/go-libp2p-mplex#23](https://github.com/libp2p/go-libp2p-mplex/pull/23))
  - change OpenStream to accept a context ([libp2p/go-libp2p-mplex#21](https://github.com/libp2p/go-libp2p-mplex/pull/21))
  - feat: update stream interfaces ([libp2p/go-libp2p-mplex#20](https://github.com/libp2p/go-libp2p-mplex/pull/20))
- github.com/libp2p/go-libp2p-noise (v0.1.1 -> v0.1.2):
  - optimize: reduce syscalls using a buffered reader.
- github.com/libp2p/go-libp2p-pubsub (v0.3.5 -> v0.4.1):
  - defer stream removal instead of doing it inline.
  - add test for inbound stream deduplication
  - deduplicate inbound streams
  - populate receivedFrom field in delivery trace
  - add receivedFrom field in delivery trace
  - fix: reduce log spam (#394) ([libp2p/go-libp2p-pubsub#394](https://github.com/libp2p/go-libp2p-pubsub/pull/394))
  - fix: treat peers already connected to the host before pubsub is initialized as valid potential pubsub peers
  - test: add test for if nodes are connected before pubsub is started
  - feat: update to go-libp2p-core 0.7.0
  - Add go-libp2p example in README.md (#392) ([libp2p/go-libp2p-pubsub#392](https://github.com/libp2p/go-libp2p-pubsub/pull/392))
  - subscription filters
  - remove multi-topic message support
  - satisfy race detector
  - clean up
  - copy string topic
  - add test for score adjustment from topis params reset
  - prettify things
  - add test for topic score parameter reset method
  - add test for topic score parameter reset
  - add api for dynamically setting and resetting topic score parameters
  - add support for priority topic delivery weights
  - tweak duplicate/reject weights
  - decay global counters after 2 min
  - decouple global counter decay from source counter decay
  - add warning for failure to parse IP out of remote multiaddr
  - more docs
  - configure the peer gater using a parameter object, docs and stuff
  - disable codecov annotations, makes things unreadable
  - further tweak gate threshold weights
  - fix test races
  - use IPs for peer gater stat tracking
  - mix total accounting components with different weights
  - count all rejections by default
  - fix non-determinism in test
  - tweak probability threshold
  - also account for duplicates in gating decisions
  - test throttle code path in gossip tracer
  - add test for peer gater
  - more efficient promise processing on throttling
  - trace throttle peers to avoid breaking promises unfairly
  - better log messages around gating
  - implement peer gater
  - peer gater scaffolding
  - rich router acceptance semantics
  - reduce log verbosity; debug mostly
- github.com/libp2p/go-libp2p-pubsub-router (v0.3.2 -> v0.4.0):
  - feat: use new stream interfaces from go-libp2p-core 0.7.0 ([libp2p/go-libp2p-pubsub-router#81](https://github.com/libp2p/go-libp2p-pubsub-router/pull/81))
- github.com/libp2p/go-libp2p-quic-transport (v0.8.0 -> v0.10.0):
  - change OpenStream to accept a context ([libp2p/go-libp2p-quic-transport#189](https://github.com/libp2p/go-libp2p-quic-transport/pull/189))
  - update quic-go to v0.19.1 ([libp2p/go-libp2p-quic-transport#182](https://github.com/libp2p/go-libp2p-quic-transport/pull/182))
  - pass a conn that can be type asserted to a net.UDPConn to quic-go ([libp2p/go-libp2p-quic-transport#180](https://github.com/libp2p/go-libp2p-quic-transport/pull/180))
  - add more integration tests ([libp2p/go-libp2p-quic-transport#181](https://github.com/libp2p/go-libp2p-quic-transport/pull/181))
  - always close the connection in the cmd client ([libp2p/go-libp2p-quic-transport#175](https://github.com/libp2p/go-libp2p-quic-transport/pull/175))
  - use GitHub Actions to test interopability of releases ([libp2p/go-libp2p-quic-transport#173](https://github.com/libp2p/go-libp2p-quic-transport/pull/173))
  - Implement CloseRead/CloseWrite ([libp2p/go-libp2p-quic-transport#174](https://github.com/libp2p/go-libp2p-quic-transport/pull/174))
  - enable quic-go metrics collection ([libp2p/go-libp2p-quic-transport#172](https://github.com/libp2p/go-libp2p-quic-transport/pull/172))
- github.com/libp2p/go-libp2p-swarm (v0.2.8 -> v0.4.0):
  - use a context for OpenStream and NewStream ([libp2p/go-libp2p-swarm#232](https://github.com/libp2p/go-libp2p-swarm/pull/232))
  - fix: handle case where swarm closes before stream ([libp2p/go-libp2p-swarm#229](https://github.com/libp2p/go-libp2p-swarm/pull/229))
  - feat: update to latest go-libp2p-core interfaces ([libp2p/go-libp2p-swarm#228](https://github.com/libp2p/go-libp2p-swarm/pull/228))
- github.com/libp2p/go-libp2p-testing (v0.2.0 -> v0.4.0):
  - pass contexts to OpenStream in tests ([libp2p/go-libp2p-testing#31](https://github.com/libp2p/go-libp2p-testing/pull/31))
  - chore: Adding LICENSE. ([libp2p/go-libp2p-testing#30](https://github.com/libp2p/go-libp2p-testing/pull/30))
  - feat: update to go-libp2p-core 0.7.0 ([libp2p/go-libp2p-testing#29](https://github.com/libp2p/go-libp2p-testing/pull/29))
- github.com/libp2p/go-libp2p-transport-upgrader (v0.3.0 -> v0.4.0):
  - pass contexts to OpenStream in tests ([libp2p/go-libp2p-transport-upgrader#70](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/70))
  - fix int to string conversion in tests, update Go version on CI ([libp2p/go-libp2p-transport-upgrader#69](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/69))
- github.com/libp2p/go-libp2p-yamux (v0.2.8 -> v0.5.1):
  - update go-yamux to v2.0.0, use context passed to OpenStream ([libp2p/go-libp2p-yamux#31](https://github.com/libp2p/go-libp2p-yamux/pull/31))
  - change OpenStream to accept a context ([libp2p/go-libp2p-yamux#29](https://github.com/libp2p/go-libp2p-yamux/pull/29))
  - feat: update to new stream interfaces ([libp2p/go-libp2p-yamux#27](https://github.com/libp2p/go-libp2p-yamux/pull/27))
- github.com/libp2p/go-mplex (v0.1.2 -> v0.3.0):
  - add a context to NewStream, remove the NewStreamTimeout ([libp2p/go-mplex#82](https://github.com/libp2p/go-mplex/pull/82))
  - Implement new CloseWrite/CloseRead interface ([libp2p/go-mplex#81](https://github.com/libp2p/go-mplex/pull/81))
  - Bump lodash from 4.17.15 to 4.17.19 in /interop/js ([libp2p/go-mplex#79](https://github.com/libp2p/go-mplex/pull/79))
  - upgrade deps + interoperable varints. (#80) ([libp2p/go-mplex#80](https://github.com/libp2p/go-mplex/pull/80))
  - write benchmarks (#77) ([libp2p/go-mplex#77](https://github.com/libp2p/go-mplex/pull/77))
- github.com/libp2p/go-ws-transport (v0.3.1 -> v0.4.0):
  - pass a context to OpenStream in tests ([libp2p/go-ws-transport#98](https://github.com/libp2p/go-ws-transport/pull/98))
  - Dependency: Remove deprecated multiaddr-net ([libp2p/go-ws-transport#97](https://github.com/libp2p/go-ws-transport/pull/97))
  - Update for go 1.14 Wasm changes ([libp2p/go-ws-transport#96](https://github.com/libp2p/go-ws-transport/pull/96))
- github.com/libp2p/go-yamux (v1.3.7 -> v1.4.1):
  - feat: improve ping accuracy ([libp2p/go-yamux#35](https://github.com/libp2p/go-yamux/pull/35))
  - implement CloseRead/CloseWrite ([libp2p/go-yamux#5](https://github.com/libp2p/go-yamux/pull/5))
  - fix space accounting in the receive buffer ([libp2p/go-yamux#33](https://github.com/libp2p/go-yamux/pull/33))
  - Limit pings ([libp2p/go-yamux#32](https://github.com/libp2p/go-yamux/pull/32))
  - fix: simplify inflight fix ([libp2p/go-yamux#31](https://github.com/libp2p/go-yamux/pull/31))
  - Clearing inflight along with streams to avoid memory leak ([libp2p/go-yamux#30](https://github.com/libp2p/go-yamux/pull/30))
- github.com/lucas-clemente/quic-go (v0.18.0 -> v0.19.3):
  - create a v0.19.x release
  - improve the warning about the UDP receive buffer size ([lucas-clemente/quic-go#2923](https://github.com/lucas-clemente/quic-go/pull/2923))
  - immediately remove reset tokens when retiring a connection ID ([lucas-clemente/quic-go#2897](https://github.com/lucas-clemente/quic-go/pull/2897))
  - add common temporary file patterns to .gitignore ([lucas-clemente/quic-go#2917](https://github.com/lucas-clemente/quic-go/pull/2917))
  - disable key updates when using HTTP/3 to avoid breaking Chrome 87 ([lucas-clemente/quic-go#2906](https://github.com/lucas-clemente/quic-go/pull/2906))
  - fix decoding of packet numbers in different packet number spaces ([lucas-clemente/quic-go#2903](https://github.com/lucas-clemente/quic-go/pull/2903))
  - log sent packet before logging its congestion / loss recovery effects ([lucas-clemente/quic-go#2912](https://github.com/lucas-clemente/quic-go/pull/2912))
  - fix a crash in the http3.Server when GetConfigForClient returns nil ([lucas-clemente/quic-go#2925](https://github.com/lucas-clemente/quic-go/pull/2925))
  - set the UDP receive buffer size on Windows ([lucas-clemente/quic-go#2896](https://github.com/lucas-clemente/quic-go/pull/2896))
  - remove superfluous sleep in packet handler map test ([lucas-clemente/quic-go#2894](https://github.com/lucas-clemente/quic-go/pull/2894))
  - fix setting of http.Handler in the example server ([lucas-clemente/quic-go#2900](https://github.com/lucas-clemente/quic-go/pull/2900))
  - remove stray print statement
  - remove unnecessary mutex locking in the stream flow controller ([lucas-clemente/quic-go#2869](https://github.com/lucas-clemente/quic-go/pull/2869))
  - only use syscalls on platforms that we're actually testing ([lucas-clemente/quic-go#2886](https://github.com/lucas-clemente/quic-go/pull/2886))
  - only write headers with a length that fits into 2 bytes in fuzz test ([lucas-clemente/quic-go#2884](https://github.com/lucas-clemente/quic-go/pull/2884))
  - fix packing of 1-RTT probe packets ([lucas-clemente/quic-go#2882](https://github.com/lucas-clemente/quic-go/pull/2882))
  - use PADDING frames to pad packets ([lucas-clemente/quic-go#2876](https://github.com/lucas-clemente/quic-go/pull/2876))
  - fix race condition when accepting streams ([lucas-clemente/quic-go#2874](https://github.com/lucas-clemente/quic-go/pull/2874))
  - only trace dropped 0-RTT packets when a tracer is set ([lucas-clemente/quic-go#2871](https://github.com/lucas-clemente/quic-go/pull/2871))
  - use consistent version numbers in client test ([lucas-clemente/quic-go#2870](https://github.com/lucas-clemente/quic-go/pull/2870))
  - replace the RWMutex with a Mutex in the flow controller ([lucas-clemente/quic-go#2865](https://github.com/lucas-clemente/quic-go/pull/2865))
  - replace the RWMutex with a Mutex in the packet handler map ([lucas-clemente/quic-go#2864](https://github.com/lucas-clemente/quic-go/pull/2864))
  - wait until the handshake is complete before updating the connection ID ([lucas-clemente/quic-go#2856](https://github.com/lucas-clemente/quic-go/pull/2856))
  - only check the SCID for Initial packets ([lucas-clemente/quic-go#2857](https://github.com/lucas-clemente/quic-go/pull/2857))
  - add the NO_VIABLE_PATH error ([lucas-clemente/quic-go#2861](https://github.com/lucas-clemente/quic-go/pull/2861))
  - implement qlogging of the preferred address in the transport parameters ([lucas-clemente/quic-go#2853](https://github.com/lucas-clemente/quic-go/pull/2853))
  - explicitly set the supported versions in the HTTP/3 server test ([lucas-clemente/quic-go#2854](https://github.com/lucas-clemente/quic-go/pull/2854))
  - allow an amplification factor of 3.x ([lucas-clemente/quic-go#2862](https://github.com/lucas-clemente/quic-go/pull/2862))
  - only allow the HTTP/3 client to dial with a single QUIC version ([lucas-clemente/quic-go#2848](https://github.com/lucas-clemente/quic-go/pull/2848))
  - send STREAMS_BLOCKED frame when MAX_STREAMS frame allows too few streams  ([lucas-clemente/quic-go#2828](https://github.com/lucas-clemente/quic-go/pull/2828))
  - set the ALPN based on the QUIC version in the HTTP3 server ([lucas-clemente/quic-go#2847](https://github.com/lucas-clemente/quic-go/pull/2847))
  - pad datagrams containing ack-eliciting Initial packets sent by the server ([lucas-clemente/quic-go#2841](https://github.com/lucas-clemente/quic-go/pull/2841))
  - fix OpenStreamSync busy looping ([lucas-clemente/quic-go#2827](https://github.com/lucas-clemente/quic-go/pull/2827))
  - fix deadlock when closing the server and the connection at the same time ([lucas-clemente/quic-go#2849](https://github.com/lucas-clemente/quic-go/pull/2849))
  - run gofumpt, enable the gofumpt linter ([lucas-clemente/quic-go#2839](https://github.com/lucas-clemente/quic-go/pull/2839))
  - prepare for draft-32 ([lucas-clemente/quic-go#2831](https://github.com/lucas-clemente/quic-go/pull/2831))
  - update the invalid packet limit for AES ([lucas-clemente/quic-go#2825](https://github.com/lucas-clemente/quic-go/pull/2825))
  - increase UDP receive buffer size ([lucas-clemente/quic-go#2791](https://github.com/lucas-clemente/quic-go/pull/2791))
  - listen on both IPv4 and IPv6 in the interop runner server ([lucas-clemente/quic-go#2822](https://github.com/lucas-clemente/quic-go/pull/2822))
  - only send Version Negotiation packets for packets larger than 1200 bytes ([lucas-clemente/quic-go#2820](https://github.com/lucas-clemente/quic-go/pull/2820))
  - don't send a version negotiation packet in response to a version negotiation packet ([lucas-clemente/quic-go#2818](https://github.com/lucas-clemente/quic-go/pull/2818))
  - client: Add DialEarlyContext and DialAddrEarlyContext API ([lucas-clemente/quic-go#2814](https://github.com/lucas-clemente/quic-go/pull/2814))
  - qlog the key phase bit ([lucas-clemente/quic-go#2817](https://github.com/lucas-clemente/quic-go/pull/2817))
  - only include quic-trace when the quictrace build flag is set ([lucas-clemente/quic-go#2799](https://github.com/lucas-clemente/quic-go/pull/2799))
  - fix error handling when receiving post handshake messages ([lucas-clemente/quic-go#2807](https://github.com/lucas-clemente/quic-go/pull/2807))
  - add support for the ChaCha20 test on the server side ([lucas-clemente/quic-go#2816](https://github.com/lucas-clemente/quic-go/pull/2816))
  - allow the first key update immediately after handshake confirmation ([lucas-clemente/quic-go#2811](https://github.com/lucas-clemente/quic-go/pull/2811))
  - ignore temporary errors when reading from the packet conn ([lucas-clemente/quic-go#2806](https://github.com/lucas-clemente/quic-go/pull/2806))
  - fix linting error on OSX ([lucas-clemente/quic-go#2813](https://github.com/lucas-clemente/quic-go/pull/2813))
  - add the exhaustive linter, replace panics by return values in logging stringers ([lucas-clemente/quic-go#2729](https://github.com/lucas-clemente/quic-go/pull/2729))
  - include the error code in the string for CRYPTO_ERRORs ([lucas-clemente/quic-go#2805](https://github.com/lucas-clemente/quic-go/pull/2805))
  - fail the handshake if the quic_transport_parameter extension is missing ([lucas-clemente/quic-go#2804](https://github.com/lucas-clemente/quic-go/pull/2804))
  - fix logging of received Retry packets ([lucas-clemente/quic-go#2803](https://github.com/lucas-clemente/quic-go/pull/2803))
  - fix deadlock in crypto setup when it is closed while handling a message ([lucas-clemente/quic-go#2802](https://github.com/lucas-clemente/quic-go/pull/2802))
  - make the key update integration test more rigorous ([lucas-clemente/quic-go#2760](https://github.com/lucas-clemente/quic-go/pull/2760))
  - add support for the new keyupdate interop runner test case ([lucas-clemente/quic-go#2782](https://github.com/lucas-clemente/quic-go/pull/2782))
  - remove unneeded mutex in the client ([lucas-clemente/quic-go#2798](https://github.com/lucas-clemente/quic-go/pull/2798))
  - correctly handle key updates within the 3 PTO period ([lucas-clemente/quic-go#2787](https://github.com/lucas-clemente/quic-go/pull/2787))
  - introduce an ECNCapablePacketConn interface to determine ECN support ([lucas-clemente/quic-go#2788](https://github.com/lucas-clemente/quic-go/pull/2788))
  - use certificates from /certs directory for the server ([lucas-clemente/quic-go#2794](https://github.com/lucas-clemente/quic-go/pull/2794))
  - remove support for the ECN test case ([lucas-clemente/quic-go#2793](https://github.com/lucas-clemente/quic-go/pull/2793))
  - check that the peer updated its keys when acknowledging a key update ([lucas-clemente/quic-go#2781](https://github.com/lucas-clemente/quic-go/pull/2781))
  - fix flaky packet number skipping test ([lucas-clemente/quic-go#2786](https://github.com/lucas-clemente/quic-go/pull/2786))
  - read ECN bits and send ECN counters in ACK frames ([lucas-clemente/quic-go#2741](https://github.com/lucas-clemente/quic-go/pull/2741))
  - implement the limit of unsuccessful decryptions for the AEADs ([lucas-clemente/quic-go#2771](https://github.com/lucas-clemente/quic-go/pull/2771))
  - use the KEY_UPDATE_ERROR ([lucas-clemente/quic-go#2770](https://github.com/lucas-clemente/quic-go/pull/2770))
  - fix dropping of key phase 0 ([lucas-clemente/quic-go#2769](https://github.com/lucas-clemente/quic-go/pull/2769))
  - reduce the handshake timeout to two minutes in the handshake drop tests ([lucas-clemente/quic-go#2768](https://github.com/lucas-clemente/quic-go/pull/2768))
  - fix handling of multiple handshake messages in the case of errors ([lucas-clemente/quic-go#2777](https://github.com/lucas-clemente/quic-go/pull/2777))
  - enable more linters, update golangci-lint to v1.31 ([lucas-clemente/quic-go#2775](https://github.com/lucas-clemente/quic-go/pull/2775))
  - increase the threshold for the receive stream deadline test ([lucas-clemente/quic-go#2774](https://github.com/lucas-clemente/quic-go/pull/2774))
  - add an assertion that bytes_in_flight never becomes negative ([lucas-clemente/quic-go#2779](https://github.com/lucas-clemente/quic-go/pull/2779))
  - fix race condition in handshake fuzz code ([lucas-clemente/quic-go#2778](https://github.com/lucas-clemente/quic-go/pull/2778))
  - use more tls.Config options in the handshake fuzzer ([lucas-clemente/quic-go#2746](https://github.com/lucas-clemente/quic-go/pull/2746))
  - run two handshakes in the handshake fuzzer ([lucas-clemente/quic-go#2743](https://github.com/lucas-clemente/quic-go/pull/2743))
  - send post-handshake message in the handshake fuzzer ([lucas-clemente/quic-go#2742](https://github.com/lucas-clemente/quic-go/pull/2742))
  - skip a packet number when sending a 1-RTT PTO packet ([lucas-clemente/quic-go#2754](https://github.com/lucas-clemente/quic-go/pull/2754))
  - save dummy packets in the packet history when skipping packet numbers ([lucas-clemente/quic-go#2753](https://github.com/lucas-clemente/quic-go/pull/2753))
  - delete unacknowledged packets from the packet history after 3 PTOs ([lucas-clemente/quic-go#2750](https://github.com/lucas-clemente/quic-go/pull/2750))
  - add support for the HTTP CONNECT method (#2761) ([lucas-clemente/quic-go#2761](https://github.com/lucas-clemente/quic-go/pull/2761))
  - don't drop keys for key phase N before receiving a N+1-protected packet ([lucas-clemente/quic-go#2762](https://github.com/lucas-clemente/quic-go/pull/2762))
  - close session on errors unpacking errors other than decryption errors ([lucas-clemente/quic-go#2756](https://github.com/lucas-clemente/quic-go/pull/2756))
  - log when an old 1-RTT key is retired ([lucas-clemente/quic-go#2765](https://github.com/lucas-clemente/quic-go/pull/2765))
  - only return an invalid first key phase error for decryptable packets ([lucas-clemente/quic-go#2757](https://github.com/lucas-clemente/quic-go/pull/2757))
  - fix logging of locally initiated key updates ([lucas-clemente/quic-go#2764](https://github.com/lucas-clemente/quic-go/pull/2764))
  - test that both endpoints time out in the timeout integration test ([lucas-clemente/quic-go#2744](https://github.com/lucas-clemente/quic-go/pull/2744))
  - refactor RTT measurements to simplify the sentPacketHistory ([lucas-clemente/quic-go#2747](https://github.com/lucas-clemente/quic-go/pull/2747))
  - fix dropping of 0-RTT packets ([lucas-clemente/quic-go#2752](https://github.com/lucas-clemente/quic-go/pull/2752))
  - always qlog the generation of 1-RTT key updates ([lucas-clemente/quic-go#2763](https://github.com/lucas-clemente/quic-go/pull/2763))
  - move the PacketHeader struct from logging to qlog package ([lucas-clemente/quic-go#2766](https://github.com/lucas-clemente/quic-go/pull/2766))
  - use a uint8 for the EncryptionLevel ([lucas-clemente/quic-go#2751](https://github.com/lucas-clemente/quic-go/pull/2751))
  - make sure to only pass handshake messages that keys are available for ([lucas-clemente/quic-go#2739](https://github.com/lucas-clemente/quic-go/pull/2739))
  - only close the handshake fuzz runner once ([lucas-clemente/quic-go#2740](https://github.com/lucas-clemente/quic-go/pull/2740))
  - generate a self-signed certificate for the handshake fuzzer ([lucas-clemente/quic-go#2738](https://github.com/lucas-clemente/quic-go/pull/2738))
  - use the os.ErrDeadlineExceeded for stream deadline errors on Go 1.15 ([lucas-clemente/quic-go#2734](https://github.com/lucas-clemente/quic-go/pull/2734))
  - use GitHub Actions to run unit tests ([lucas-clemente/quic-go#2732](https://github.com/lucas-clemente/quic-go/pull/2732))
  - add a basic fuzzer for the handshake ([lucas-clemente/quic-go#2733](https://github.com/lucas-clemente/quic-go/pull/2733))
  - export seed corpus files using the SHA1 of the content as the filename ([lucas-clemente/quic-go#2731](https://github.com/lucas-clemente/quic-go/pull/2731))
  - add a fuzz target for the token generator ([lucas-clemente/quic-go#2730](https://github.com/lucas-clemente/quic-go/pull/2730))
  - fix typo in error message in sent packet handler
  - fix missing OnLost callback for frames sent in 0-RTT packets ([lucas-clemente/quic-go#2728](https://github.com/lucas-clemente/quic-go/pull/2728))
  - fix overflow of the max_ack_delay when parsing transport parameters ([lucas-clemente/quic-go#2725](https://github.com/lucas-clemente/quic-go/pull/2725))
- github.com/marten-seemann/qpack (v0.2.0 -> v0.2.1):
  - run gofumpt, add a few more linters ([marten-seemann/qpack#21](https://github.com/marten-seemann/qpack/pull/21))
  - fix static table entry 80 ([marten-seemann/qpack#20](https://github.com/marten-seemann/qpack/pull/20))
- github.com/marten-seemann/qtls-go1-15 (v0.1.0 -> v0.1.1):
  - use a prefix for client session cache keys
  - add callbacks to store and restore app data along a session state
  - don't use TLS 1.3 compatibility mode when using alternative record layer
  - delete the session ticket after attempting 0-RTT
  - reject 0-RTT when a different ALPN is chosen
  - encode the ALPN into the session ticket
  - add a field to the ConnectionState to tell if 0-RTT was used
  - add a callback to tell the client about rejection of 0-RTT
  - don't offer 0-RTT after a HelloRetryRequest
  - add Accept0RTT to Config callback to decide if 0-RTT should be accepted
  - add the option to encode application data into the session ticket
  - export the 0-RTT write key
  - abuse the nonce field of ClientSessionState to save max_early_data_size
  - export the 0-RTT read key
  - close connection if client attempts 0-RTT, but ticket didn't allow it
  - encode the max early data size into the session ticket
  - implement parsing of the early_data extension in the EncryptedExtensions
  - add a tls.Config.MaxEarlyData option to enable 0-RTT
  - accept TLS 1.3 cipher suites in Config.CipherSuites
  - introduce a function on the connection to generate a session ticket
  - add a config option to enforce selection of an application protocol
  - export Conn.HandlePostHandshakeMessage
  - export Alert
  - reject Configs that set MaxVersion < 1.3 when using a record layer
  - enforce TLS 1.3 when using an alternative record layer
- github.com/multiformats/go-multistream (v0.1.2 -> v0.2.0):
  - improve negotiation flushing ([multiformats/go-multistream#52](https://github.com/multiformats/go-multistream/pull/52))
- github.com/whyrusleeping/cbor-gen (v0.0.0-20200402171437-3d27c146c105 -> v0.0.0-20200710004633-5379fc63235d):
  - correctly map typegen to cbg in all cases ([whyrusleeping/cbor-gen#26](https://github.com/whyrusleeping/cbor-gen/pull/26))
  - fix: clear struct state on unmarshal ([whyrusleeping/cbor-gen#22](https://github.com/whyrusleeping/cbor-gen/pull/22))
  - deferred: restrict max length ([whyrusleeping/cbor-gen#25](https://github.com/whyrusleeping/cbor-gen/pull/25))
  - reduce number of allocations in ScanForLinks ([whyrusleeping/cbor-gen#24](https://github.com/whyrusleeping/cbor-gen/pull/24))
  - attempt to allocate less by using shared buffers ([whyrusleeping/cbor-gen#18](https://github.com/whyrusleeping/cbor-gen/pull/18))
  - add benchmark
  - use new cid methods for less allocs ([whyrusleeping/cbor-gen#17](https://github.com/whyrusleeping/cbor-gen/pull/17))
  - properly handle roundtripping Deferred with 'null' value ([whyrusleeping/cbor-gen#16](https://github.com/whyrusleeping/cbor-gen/pull/16))
  - Support array types ([whyrusleeping/cbor-gen#15](https://github.com/whyrusleeping/cbor-gen/pull/15))
- github.com/whyrusleeping/tar-utils (v0.0.0-20180509141711-8c6c8ba81d5c -> v0.0.0-20201201191210-20a61371de5b):
  - more closely match default tar errors (GNU + BSD binaries)

Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Eric Myhre | 180 | +26453/-11032 | 883 |
| Marten Seemann | 212 | +14876/-9352 | 794 |
| hannahhoward | 41 | +9195/-3113 | 186 |
| Alex Cruikshank | 5 | +3323/-1895 | 58 |
| Andrew Gillis | 3 | +3792/-581 | 21 |
| vyzo | 49 | +2675/-949 | 95 |
| Adin Schmahmann | 57 | +1473/-837 | 90 |
| Steven Allen | 43 | +1252/-780 | 99 |
| Petar Maymounkov | 3 | +1755/-113 | 17 |
| Marcin Rataj | 35 | +979/-210 | 61 |
| Paul Wolneykien | 2 | +670/-338 | 9 |
| Jeromy Johnson | 9 | +525/-221 | 21 |
| gammazero | 11 | +366/-101 | 26 |
| Hector Sanjuan | 7 | +312/-0 | 11 |
| Dirk McCormick | 4 | +190/-90 | 15 |
| Will Scott | 1 | +252/-0 | 1 |
| Oli Evans | 1 | +201/-0 | 1 |
| Tomasz Zdybał | 2 | +182/-3 | 6 |
| Daniel Martí | 6 | +104/-66 | 35 |
| Sam | 3 | +76/-59 | 5 |
| Łukasz Magiera | 2 | +92/-3 | 5 |
| whyrusleeping | 3 | +77/-15 | 3 |
| nisdas | 3 | +76/-15 | 4 |
| Raúl Kripalani | 3 | +59/-31 | 5 |
| Lucas Molas | 1 | +66/-3 | 2 |
| Alex Towle | 1 | +52/-8 | 2 |
| Dennis Trautwein | 1 | +58/-0 | 2 |
| Adrian Lanzafame | 2 | +49/-7 | 4 |
| klzgrad | 1 | +49/-5 | 2 |
| Fazlul Shahriar | 1 | +35/-14 | 17 |
| Yingrong Zhao | 1 | +45/-2 | 2 |
| Jakub Sztandera | 2 | +22/-13 | 2 |
| Chaitanya | 8 | +16/-16 | 8 |
| Aarsh Shah | 1 | +27/-1 | 3 |
| Rod Vagg | 1 | +23/-4 | 2 |
| M. Hawn | 4 | +11/-11 | 8 |
| Will | 1 | +12/-2 | 1 |
| frrist | 1 | +7/-0 | 1 |
| Rafael Ramalho | 2 | +5/-2 | 2 |
| dependabot[bot] | 1 | +3/-3 | 1 |
| Zaurbek Zhakupov | 1 | +3/-3 | 1 |
| Tom Worrall | 1 | +4/-2 | 1 |
| Jorropo | 2 | +5/-1 | 2 |
| Chaitanya Raju | 1 | +3/-3 | 2 |
| Egon Elbre | 1 | +0/-5 | 1 |
| incognitomode | 1 | +2/-2 | 1 |
| achingbrain | 1 | +2/-2 | 1 |
| Michael Burns | 1 | +2/-2 | 1 |
| David Florness | 2 | +2/-2 | 2 |
| RubenKelevra | 1 | +2/-1 | 1 |
| Andrew Nesbitt | 2 | +2/-1 | 2 |
| Tarun Bansal | 1 | +1/-1 | 1 |
| Max Inden | 1 | +1/-1 | 1 |
| K | 1 | +2/-0 | 1 |
| Jacob Heun | 1 | +1/-1 | 1 |
| Henrique Dias | 1 | +1/-1 | 1 |
| Bryan White | 1 | +1/-1 | 1 |
| Bryan Stenson | 1 | +1/-1 | 1 |


# go-ipfs changelog v0.9

## v0.9.1 2021-07-20

This is a small bug fix release resolving the following issues:
1. A regression where the empty CID bafkqaaa could not resolve on gateways [#8230](https://github.com/ipfs/go-ipfs/issues/8230)
2. A panic on OpenBSD [#8211](https://github.com/ipfs/go-ipfs/issues/8211)
3. High CPU usage with QUIC [#8256](https://github.com/ipfs/go-ipfs/issues/8256)
4. High memory usage with TCP [#8219](https://github.com/ipfs/go-ipfs/issues/8219)
5. Some pubsub issues ([libp2p/go-libp2p-pubsub#427](https://github.com/libp2p/go-libp2p-pubsub/pull/427), [libp2p/go-libp2p-pubsub#430](https://github.com/libp2p/go-libp2p-pubsub/pull/430))
6. Updated WebUI to [v2.12.4](https://github.com/ipfs/ipfs-webui/releases/tag/v2.12.4)
7. Fixed the snap deployment [#8212](https://github.com/ipfs/go-ipfs/pull/8212)

### Changelog

- github.com/ipfs/go-ipfs:
  - chore: update deps
  - feat: webui v2.12.4
  - test: gateway response for bafkqaaa
  - fix: downgrade mimetype dependency
  - update go-libp2p to v0.14.3
  - bump snap to build with Go 1.16
- github.com/libp2p/go-libp2p (v0.14.2 -> v0.14.3):
  - update go-tcp-transport to v0.2.3 and go-multiaddr to v0.3.3 ([libp2p/go-libp2p#1121](https://github.com/libp2p/go-libp2p/pull/1121))
- github.com/libp2p/go-libp2p-pubsub (v0.4.1 -> v0.4.2):
  - release priority locks early when handling batches
  - don't respawn writer if we fail to open a stream; declare it a peer error
  - batch process dead peer notifications
  - use a priority lock instead of a semaphore
  - do the notification in a goroutine
  - emit new peer notification without holding the semaphore
  - use a semaphore for new peer notifications so that we don't block the event loop
  - don't accumulate pending goroutines from new connections
  - Make close concurrent safe
  - Fix close of closed channel
- github.com/libp2p/go-libp2p-quic-transport (v0.11.1 -> v0.11.2):
  - update quic-go to v0.21.2
- github.com/libp2p/go-tcp-transport (v0.2.2 -> v0.2.4):
  - collect metrics in a separate go routine ([libp2p/go-tcp-transport#82](https://github.com/libp2p/go-tcp-transport/pull/82))
  - fix: avoid logging "invalid argument" errors when setting keepalive ([libp2p/go-tcp-transport#83](https://github.com/libp2p/go-tcp-transport/pull/83))
  - Skip SetKeepAlivePeriod call on OpenBSD ([libp2p/go-tcp-transport#80](https://github.com/libp2p/go-tcp-transport/pull/80))
  - sync: update CI config files (#79) ([libp2p/go-tcp-transport#79](https://github.com/libp2p/go-tcp-transport/pull/79))
- github.com/lucas-clemente/quic-go (v0.21.1 -> v0.21.2):
  - update qtls to include the crypto/tls fix of Go 1.16.6 / 1.15.14
  - cancel the PTO timer when all Handshake packets are acknowledged
  - update to Go 1.17rc1
  - update Ginkgo to v1.16.4 and Gomega to v1.13.0 ([lucas-clemente/quic-go#3139](https://github.com/lucas-clemente/quic-go/pull/3139))
- github.com/multiformats/go-multiaddr (v0.3.2 -> v0.3.3):
  - guard against nil {Local,Remote}Addr() return values ([multiformats/go-multiaddr#155](https://github.com/multiformats/go-multiaddr/pull/155))
  - sync: update CI config files (#154) ([multiformats/go-multiaddr#154](https://github.com/multiformats/go-multiaddr/pull/154))

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| vyzo | 8 | +205/-141 | 12 |
| Marten Seemann | 7 | +127/-74 | 11 |
| gammazero | 2 | +43/-5 | 3 |
| Steven Allen | 1 | +13/-2 | 1 |
| Adin Schmahmann | 3 | +13/-2 | 3 |
| Marcin Rataj | 2 | +9/-1 | 2 |
| Aaron Bieber | 1 | +6/-2 | 1 |

## v0.9.0 2021-06-22

We're happy to announce go-ipfs 0.9.0. This release makes go-ipfs even more configurable with some fun experiments to boot. We're also deprecating or removing some uncommonly used features to make it easier for users to discover the easy ways to use go-ipfs safely and efficiently.

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [release process](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

### 🔦 Highlights

#### 📦 Exporting of DAGs via Gateways

Gateways now support downloading arbitrary IPLD graphs via the `/api/v0/dag/export` endpoint. This endpoint works in the same way as the `ipfs dag export` command.

One major thing this enables is ability to verify data downloaded from public gateways. If you go to `https://somegateway.example.net/ipfs/bafyexample` you are using the old school HTTP transport, and trusting that the gateway is being well behaved. However, if you download the graph as a [DAG archive](https://github.com/ipld/specs/blob/master/block-layer/content-addressable-archives.md) then it is possible to verify that the data you downloaded does in fact match `bafyexample`.

Additionally, it was previously quite painful to download things other than UnixFS (files + directories) using gateways. It is now possible to download arbitrary IPLD graphs from gateways, making them useful as a general-purpose alternative to p2p transports.

This opens exciting opportunities in areas like thin clients, mobile browsers and IoT devices, which now can delegate IPFS resolution to any public gateway, and have ability to verify that the data received matches the requested hash.

#### ☁ Custom DNS Resolvers

Resolution of DNS records for DNSLink and DNSAddrs means that names  are sent in cleartext between the operating system and the DNS server provided by an ISP. In the past, the only way to customize DNS resolution in IPFS stack was to set up own DNS proxy server.

There is now the ability to [customize DNS resolution](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#dns) and override the default resolver from the OS with [DNS over HTTPS](https://en.wikipedia.org/wiki/DNS_over_HTTPS) (DoH) one. We made it really flexible: override can be applied globally, or per specific [TLD](https://en.wikipedia.org/wiki/Top-level_domain)/[FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name). Examples can be found in the [documentation](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#dns).

#### 👪 Support for non-ICANN DNSLink names

Building off of the support for custom DNS resolvers it is now possible to create DNSLink names not handled by ICANN and choose how that domain name will be resolved. An example of this is how ENS is supported, despite `.eth` not being an ICANN TLD you can point `.eth` to any ENS resolver you want (including a local one).

While go-ipfs may have some DoH defaults for a few popular non-ICANN DNSLink names (e.g. ENS), you are free to use any protocol for a naming system and as long as it exposes a DNSLink record via a DNS endpoint you can make it work.

#### 🖥️ Updated to the latest WebUI

Our web interface now includes experimental support for pinning services, and various updates to _Files_ and _Peers_ screens.

Remote pinning services added via the `ipfs pin remote service add` command are already detected, one can also add one from _Settings_ screen, and it will appear in _Set pinning_ interface on the _Files_ screen.

Data presented on the _Peers_ screen can now be copied by simply clicking on a specific cell, and a list of open streams gives better insight into how a local node interacts with a specific peer.

See release notes for [ipfs-webui v2.12](https://github.com/ipfs/ipfs-webui/releases/tag/v2.12.0) for screenshots and more details.

#### 🔑 IPNS keys can now be exported via the CLI without stopping the daemon

`ipfs key export` no longer requires interrupting `ipfs daemon` ✨

#### 🕸 Experimental DHT Client and Provider System

An area of go-ipfs that has been historically tricky is how go-ipfs finds who has the data they are looking for. While the IPFS Public DHT is only one of the ways go-ipfs can find data it tends to be an important one. While since go-ipfs v0.5.0 the time to find content in the network has dropped significantly the time to put/get IPNS records or for a node to advertise the content it has still has much room for improvement.

We have been doing some experimenting and have an alternative DHT client that essentially trades off some resources and in return is much more performant. We have also included with the experimental DHT client a bulk provider system that takes advantage of the new client to more efficiently do many advertisements at a time

This work is quite new and still under development, however, the results so far have been promising especially for users with lots of data who have otherwise been having difficulty advertising their data into the IPFS Public DHT

As described in the experimental features [documentation](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#accelerated-dht-client) the experimental client can be enabled using the command below (or modifying the config file).

```
ipfs config --json Experimental.AcceleratedDHTClient true
```

A few things to take note of when `AcceleratedDHTClient` is enabled:
- go-ipfs will likely use more resources then previously
- DHT queries will not be usable (i.e. finding which peers have some data, finding where a particular peer is, etc.) for the first 5-10 minutes of operation depending on your network conditions
- There is an `ipfs stats provide` command that will help you track your provide/reprovide usage, if you are providing lots of data you may want to consider how to reduce the amount you are providing (e.g. [Reprovider Strategies](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#reproviderstrategy) and/or [Strategic Providing](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#strategic-providing))

See the [documentation](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#accelerated-dht-client) for more details.

#### 🚶‍♀️ Migrations

##### Migrations are now individually packaged

While previously the go-ipfs [repo migration](https://github.com/ipfs/fs-repo-migrations) binary was monolithic and contained all migrations from previous go-ipfs versions the binaries are now packaged individually. However, the fs-repo-migrations binary is still there to help those who manually upgrade their repos to download all the individual migrations.

This means faster download times for upgrades, a much easier time building migrations for those who make use of custom plugins, and an easier time developing new migrations going forward.

##### Configurable migration downloads enable downloading over IPFS

Previously the migration downloader built into go-ipfs downloaded the migrations from [dist.ipfs.tech](https://dist.ipfs.tech). While users could use tools like [ipfs-update](https://github.com/ipfs/ipfs-update) to download the migrations over IPFS or manually download the migrations (over IPFS or otherwise) themselves, this is now automated and configurable. Users can choose to download the migrations over IPFS or from any specified IPFS Gateway.

The configurable migration options are described in the config file [documentation](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#migration), although most users should not need to change the default settings.

The main benefit here is that users behind restrictive firewalls, or in offline/private deployments, won't have to run migrations manually, which is especially important for desktop use cases where go-ipfs is running inside of [IPFS Desktop](https://github.com/ipfs-shipyard/ipfs-desktop#readme) and [Brave](https://brave.com/ipfs-support/).

#### 🍎 Published builds for Apple M1 hardware

Go now supports building for Darwin ARM64, and we are now publishing those builds

#### 👋 Deprecations and Feature Removals

##### The `ipfs object` commands are now deprecated

In the last couple years most of the Object API's commands have become fulfillable using alternative APIs.

The utility of Object API's is limited to data in UnixFS-v1  (`dag-pb`) format. If you are still using it, it is highly recommended that you switch to the DAG `ipfs dag` (supports modern data types like `dag-cbor`) or Files `ipfs files` (more intuitive for working with `dag-pb`) APIs.

While the Object API and commands are still usable they are now marked as deprecated and hidden from users on the command line to discourage further use. We also updated their `--help` text to point at the modern replacements.


##### `X-Ipfs-Gateway-Prefix` is now deprecated

IPFS community moved towards dedicated Origins (DNSLink and [subdomain gateways](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway)) which are much easier to isolate and reason about.

Setting up `Gateway.PathPrefixes` and `X-Ipfs-Gateway-Prefix` is no longer necessary and support [will be removed in near future](https://github.com/ipfs/go-ipfs/issues/7702).

##### Proquints support removed

A little known feature that was not well used or documented and was more well known for the error message `Error: not a valid proquint string` users received when trying to download invalid IPNS or DNSLink names (e.g. `https://dweb.link/ipns/badname`). We have removed support for proquints as they were out of place and largely unused, however proquints are [valid multibases](https://github.com/multiformats/multibase/pull/78) so if there is renewed interest in them there is a way forward.

##### SECIO support removed

SECIO was deprecated and turned off by default given the prevalence of TLS and Noise support, SECIO support is now removed entirely.

### Changelog

- github.com/ipfs/go-ipfs:
  - chore: switch tar-utils dep to ipfs org
  - chore: update CHANGELOG
  - refactor: warning when bootstrap disabled by user
  - feat: print error on bootstrap failure
  - fix: typo in migration error
  - Release v0.9.0-rc2
  - refactor: improved humanNumber and humanSI
  - feat: humanized durations in stat provide
  - feat: humanized numbers in stat provide
  - feat: add a text output encoding for the stats provide command
  - fix: webui-2.12.3
  - refactor(pinmfs): log error if pre-existing pin failed (#8056) ([ipfs/go-ipfs#8056](https://github.com/ipfs/go-ipfs/pull/8056))
  - Release v0.9.0-rc1
  - Added support for an experimental DHT client and provider system via the Experiments.AcceleratedDHTClient config option ([ipfs/go-ipfs#8061](https://github.com/ipfs/go-ipfs/pull/8061))
  - feat: support DNSLink on non-ICANN DNS names ([ipfs/go-ipfs#8071](https://github.com/ipfs/go-ipfs/pull/8071))
  - update go-tcp-transport to v0.2.2 ([ipfs/go-ipfs#8129](https://github.com/ipfs/go-ipfs/pull/8129))
  - update quic-go to v0.21.0-rc.1 ([ipfs/go-ipfs#8125](https://github.com/ipfs/go-ipfs/pull/8125))
  - chore: bump minimum go version to 1.15 ([ipfs/go-ipfs#7944](https://github.com/ipfs/go-ipfs/pull/7944))
  - chore: update deps ([ipfs/go-ipfs#8128](https://github.com/ipfs/go-ipfs/pull/8128))
  - feat: allow key export in online mode ([ipfs/go-ipfs#8113](https://github.com/ipfs/go-ipfs/pull/8113))
  - Feat/migration ipfs download (#8064) ([ipfs/go-ipfs#8064](https://github.com/ipfs/go-ipfs/pull/8064))
  - feat: support custom DoH resolvers ([ipfs/go-ipfs#8068](https://github.com/ipfs/go-ipfs/pull/8068))
  - update go-libp2p to v0.14.0 ([ipfs/go-ipfs#8122](https://github.com/ipfs/go-ipfs/pull/8122))
  - feat(gw): expose /api/v0/dag/export on gateway port ([ipfs/go-ipfs#8111](https://github.com/ipfs/go-ipfs/pull/8111))
  - chore: update webui to 2.12.2 ([ipfs/go-ipfs#8097](https://github.com/ipfs/go-ipfs/pull/8097))
  - docs: deprecate object commands ([ipfs/go-ipfs#8098](https://github.com/ipfs/go-ipfs/pull/8098))
  - fix: omit empty pins slice when reporting pin progress ([ipfs/go-ipfs#8023](https://github.com/ipfs/go-ipfs/pull/8023))
  - Fix typo in comment ([ipfs/go-ipfs#8087](https://github.com/ipfs/go-ipfs/pull/8087))
  - fix: set systemd startup timeout to infinity ([ipfs/go-ipfs#8040](https://github.com/ipfs/go-ipfs/pull/8040))
  - fix(mkreleaselog): partially handle v2 modules ([ipfs/go-ipfs#8073](https://github.com/ipfs/go-ipfs/pull/8073))
  - Update migration sharness tests for new migrations (#8053) ([ipfs/go-ipfs#8053](https://github.com/ipfs/go-ipfs/pull/8053))
  - fix: make migrations log output to stdout ([ipfs/go-ipfs#8054](https://github.com/ipfs/go-ipfs/pull/8054))
  - fix(gw): remove hardcoded hostnames ([ipfs/go-ipfs#8069](https://github.com/ipfs/go-ipfs/pull/8069))
  - Fix transposed words in docs/config.md ([ipfs/go-ipfs#8051](https://github.com/ipfs/go-ipfs/pull/8051))
  - fix: update root help ([ipfs/go-ipfs#8052](https://github.com/ipfs/go-ipfs/pull/8052))
  - chore: don't docker tag rc as latest ([ipfs/go-ipfs#8055](https://github.com/ipfs/go-ipfs/pull/8055))
  - Add info to "pin rm" help about how to tell if pin is indirect ([ipfs/go-ipfs#8044](https://github.com/ipfs/go-ipfs/pull/8044))
  - build(deps): bump contrib.go.opencensus.io/exporter/prometheus from 0.2.0 to 0.3.0 ([ipfs/go-ipfs#8020](https://github.com/ipfs/go-ipfs/pull/8020))
  - fix(gw): remove use of Clear-Site-Data in subdomain router ([ipfs/go-ipfs#7890](https://github.com/ipfs/go-ipfs/pull/7890))
  -  ([ipfs/go-ipfs#7857](https://github.com/ipfs/go-ipfs/pull/7857))
  - docs: clarification of the Strategic Providing functionality ([ipfs/go-ipfs#8035](https://github.com/ipfs/go-ipfs/pull/8035))
  - docs: cosmetic fixes of help text ([ipfs/go-ipfs#8010](https://github.com/ipfs/go-ipfs/pull/8010))
  - chore: deprecate Gateway.PathPrefixes ([ipfs/go-ipfs#7994](https://github.com/ipfs/go-ipfs/pull/7994))
  - Fix text contrast for dark mode ([ipfs/go-ipfs#8027](https://github.com/ipfs/go-ipfs/pull/8027))
  - Do not fetch recursive pins from pinner unnecessarily ([ipfs/go-ipfs#7883](https://github.com/ipfs/go-ipfs/pull/7883))
  - test(sharness): verify the list of exported metrics ([ipfs/go-ipfs#7987](https://github.com/ipfs/go-ipfs/pull/7987))
  - fix: return an error if repo verify is canceled ([ipfs/go-ipfs#7973](https://github.com/ipfs/go-ipfs/pull/7973))
  - doc: document security fix policy ([ipfs/go-ipfs#7991](https://github.com/ipfs/go-ipfs/pull/7991))
  - feat(gw): /ipfs/ipfs/{cid} → /ipfs/{cid} ([ipfs/go-ipfs#7930](https://github.com/ipfs/go-ipfs/pull/7930))
  - Fix: inaccuracies in MFS command documentation. ([ipfs/go-ipfs#8001](https://github.com/ipfs/go-ipfs/pull/8001))
  - Feat: Re-import InitializeKeyspace code from go-namesys ([ipfs/go-ipfs#7984](https://github.com/ipfs/go-ipfs/pull/7984))
  - revert registration of metrics against unexposed prom registry ([ipfs/go-ipfs#7986](https://github.com/ipfs/go-ipfs/pull/7986))
  - Extract the namesys and the keystore submodules ([ipfs/go-ipfs#7925](https://github.com/ipfs/go-ipfs/pull/7925))
  - split core/commands/dag into individual files for different subcommands ([ipfs/go-ipfs#7970](https://github.com/ipfs/go-ipfs/pull/7970))
  - test(sharness): pass correct timeout format to go-timeout ([ipfs/go-ipfs#7971](https://github.com/ipfs/go-ipfs/pull/7971))
  - fix race condition when logging requests ([ipfs/go-ipfs#7953](https://github.com/ipfs/go-ipfs/pull/7953))
  - fix some sharness-in-CI issues ([ipfs/go-ipfs#7946](https://github.com/ipfs/go-ipfs/pull/7946))
  - chore: update deps ([ipfs/go-ipfs#7941](https://github.com/ipfs/go-ipfs/pull/7941))
  - fix: correctly return pin ls errors ([ipfs/go-ipfs#7942](https://github.com/ipfs/go-ipfs/pull/7942))
  - feat: remove secio support ([ipfs/go-ipfs#7943](https://github.com/ipfs/go-ipfs/pull/7943))
  - Set supported platforms by go-version ([ipfs/go-ipfs#7927](https://github.com/ipfs/go-ipfs/pull/7927))
  - docs: tips on debugging Policies.MFS (#7929) ([ipfs/go-ipfs#7929](https://github.com/ipfs/go-ipfs/pull/7929))
  - docs: fix DNSLink gw recipe ([ipfs/go-ipfs#7932](https://github.com/ipfs/go-ipfs/pull/7932))
  - Merge branch 'release'
  - docs: RepinInterval
  - style: docs/config.md
  - style: improved MFS PinName example
  - docs: Pinning.RemoteServices.Policies
  - peering: add logs before many-second waits ([ipfs/go-ipfs#7904](https://github.com/ipfs/go-ipfs/pull/7904))
  - all: gofmt -s ([ipfs/go-ipfs#7900](https://github.com/ipfs/go-ipfs/pull/7900))
- github.com/ipfs/go-bitswap (v0.3.3 -> v0.3.4):
  - remove Makefile ([ipfs/go-bitswap#483](https://github.com/ipfs/go-bitswap/pull/483))
  - test: deflake engine test ([ipfs/go-bitswap#480](https://github.com/ipfs/go-bitswap/pull/480))
  - test: deflake large-message test ([ipfs/go-bitswap#479](https://github.com/ipfs/go-bitswap/pull/479))
  - fix: fix alignment of stats struct in virtual network ([ipfs/go-bitswap#478](https://github.com/ipfs/go-bitswap/pull/478))
  - fix(network): impl: add timeout in newStreamToPeer call ([ipfs/go-bitswap#477](https://github.com/ipfs/go-bitswap/pull/477))
  - fix staticcheck ([ipfs/go-bitswap#474](https://github.com/ipfs/go-bitswap/pull/474))
  - ignore transient connections ([ipfs/go-bitswap#470](https://github.com/ipfs/go-bitswap/pull/470))
  - fix a startup race by creating the blockstoremanager process on init ([ipfs/go-bitswap#465](https://github.com/ipfs/go-bitswap/pull/465))
- github.com/ipfs/go-block-format (v0.0.2 -> v0.0.3):
  - doc: add a lead maintainer ([ipfs/go-block-format#16](https://github.com/ipfs/go-block-format/pull/16))
- github.com/ipfs/go-graphsync (v0.6.0 -> v0.8.0):
  - docs(CHANGELOG): update for v0.8.0
  - Update for LinkSystem (#161) ([ipfs/go-graphsync#161](https://github.com/ipfs/go-graphsync/pull/161))
  - Round out diagnostic parameters (#157) ([ipfs/go-graphsync#157](https://github.com/ipfs/go-graphsync/pull/157))
  - map response codes to names (#148) ([ipfs/go-graphsync#148](https://github.com/ipfs/go-graphsync/pull/148))
  - Discard http output (#156) ([ipfs/go-graphsync#156](https://github.com/ipfs/go-graphsync/pull/156))
  - Add debug logging (#121) ([ipfs/go-graphsync#121](https://github.com/ipfs/go-graphsync/pull/121))
  - Add optional HTTP comparison (#153) ([ipfs/go-graphsync#153](https://github.com/ipfs/go-graphsync/pull/153))
  - docs(architecture): update architecture docs (#154) ([ipfs/go-graphsync#154](https://github.com/ipfs/go-graphsync/pull/154))
  - release v0.7.0 ([ipfs/go-graphsync#152](https://github.com/ipfs/go-graphsync/pull/152))
  - chore: update deps (#151) ([ipfs/go-graphsync#151](https://github.com/ipfs/go-graphsync/pull/151))
  - Automatically record heap profiles in test plans (#147) ([ipfs/go-graphsync#147](https://github.com/ipfs/go-graphsync/pull/147))
  - feat(deps): update go-ipld-prime v0.7.0 (#145) ([ipfs/go-graphsync#145](https://github.com/ipfs/go-graphsync/pull/145))
  - Release/v0.6.0 ([ipfs/go-graphsync#144](https://github.com/ipfs/go-graphsync/pull/144))
- github.com/ipfs/go-ipfs-blockstore (v0.1.4 -> v0.1.6):
  - use bloom filter in GetSize
- github.com/ipfs/go-ipfs-config (v0.12.0 -> v0.14.0):
  - Added Experiments.AcceleratedDHTClient option ([ipfs/go-ipfs-config#125](https://github.com/ipfs/go-ipfs-config/pull/125))
  - Add config for downloading repo migrations ([ipfs/go-ipfs-config#128](https://github.com/ipfs/go-ipfs-config/pull/128))
  - remove duplicate entries in defaultServerFilters ([ipfs/go-ipfs-config#121](https://github.com/ipfs/go-ipfs-config/pull/121))
  - add custom DNS Resolver configuration ([ipfs/go-ipfs-config#126](https://github.com/ipfs/go-ipfs-config/pull/126))
- github.com/ipfs/go-ipfs-provider (v0.4.3 -> v0.5.1):
  - Fix batched providing of empty keys ([ipfs/go-ipfs-provider#37](https://github.com/ipfs/go-ipfs-provider/pull/37))
  - Bulk Provide/Reproviding System (#34) ([ipfs/go-ipfs-provider#34](https://github.com/ipfs/go-ipfs-provider/pull/34))
  - chore: update the Usage part of readme ([ipfs/go-ipfs-provider#33](https://github.com/ipfs/go-ipfs-provider/pull/33))
  - Retract and revert 1.0.0 ([ipfs/go-ipfs-provider#31](https://github.com/ipfs/go-ipfs-provider/pull/31))
  - replace go-merkledag with go-fetcher ([ipfs/go-ipfs-provider#30](https://github.com/ipfs/go-ipfs-provider/pull/30))
- github.com/ipfs/go-ipld-git (v0.0.3 -> v0.0.4):
  - add license file so it can be found by go-licenses ([ipfs/go-ipld-git#42](https://github.com/ipfs/go-ipld-git/pull/42))
- github.com/ipfs/go-ipns (v0.0.2 -> v0.1.0):
  - Add support for extensible records (and v2 signature)
- github.com/ipfs/go-log (v1.0.4 -> v1.0.5):
  - chore: update v1 deps ([ipfs/go-log#108](https://github.com/ipfs/go-log/pull/108))
- github.com/ipfs/go-log/v2 (v2.1.1 -> v2.1.3):
  - doc(README): use circle-ci badge ([ipfs/go-log#106](https://github.com/ipfs/go-log/pull/106))
  - feat: add ability to specify labels for all loggers ([ipfs/go-log#105](https://github.com/ipfs/go-log/pull/105))
  - Add an option to pass URL to zap ([ipfs/go-log#101](https://github.com/ipfs/go-log/pull/101))
  - enable configuring several log outputs ([ipfs/go-log#98](https://github.com/ipfs/go-log/pull/98))
  - Fix caller not being added ([ipfs/go-log#96](https://github.com/ipfs/go-log/pull/96))
- github.com/ipfs/go-unixfs (v0.2.4 -> v0.2.5):
  - correct file size for raw node ([ipfs/go-unixfs#88](https://github.com/ipfs/go-unixfs/pull/88))
- github.com/ipld/go-car (v0.1.1-0.20201015032735-ff6ccdc46acc -> v0.3.1):
  - chore: make sure we get an error where we expect one
  - chore: refactor header tests to iterate over a struct
  - chore: add header error tests
  - fix: lint errors
  - fix: go mod tidy
  - chore: update go.mod to 1.15
  - fix: ReadHeader return value mismatch
  - Updates for ipld linksystem branch ([ipld/go-car#56](https://github.com/ipld/go-car/pull/56))
  - replace go-ipld-prime-proto with go-codec-dagpb
  - fix staticcheck errors ([ipld/go-car#67](https://github.com/ipld/go-car/pull/67))
  - chore: switch to a single license file ([ipld/go-car#59](https://github.com/ipld/go-car/pull/59))
  - chore: remove LICENSE ([ipld/go-car#58](https://github.com/ipld/go-car/pull/58))
  - chore: relicense ([ipld/go-car#57](https://github.com/ipld/go-car/pull/57))
  - ci: remove travis support ([ipld/go-car#55](https://github.com/ipld/go-car/pull/55))
  - run gofmt -s
  - Allow user defined block hooks when using two step write for selective cars ([ipld/go-car#37](https://github.com/ipld/go-car/pull/37))
  - feat: handle mid-varint EOF case as UnexpectedEOF
  - fix: main NewReader call
- github.com/ipld/go-ipld-prime (v0.5.1-0.20201021195245-109253e8a018 -> v0.9.1-0.20210324083106-dc342a9917db):
  - Add option to tell link system storage is trusted and we can skip hash on read ([ipld/go-ipld-prime#149](https://github.com/ipld/go-ipld-prime/pull/149))
  - implement non-dag cbor codec ([ipld/go-ipld-prime#153](https://github.com/ipld/go-ipld-prime/pull/153))
  - add non-dag json codec ([ipld/go-ipld-prime#152](https://github.com/ipld/go-ipld-prime/pull/152))
  - typo fixes
  - mark v0.9.0
  - Changelog: more backfill :)
  - hackme: about merge strategies.
  - Dropping .gopath and other unmaintained scripts.
  - introduce LinkSystem ([ipld/go-ipld-prime#143](https://github.com/ipld/go-ipld-prime/pull/143))
  - Readme updates.
  - codec/raw: implement the raw codec
  - add an ADL interface type
  - schema/gen/go: cache genned code in os.TempDir
  - fluent/qp: finish writing all data model helpers
  - fluent: add qp, a different spin on quip
  - schema/gen/go: prevent some unkeyed literal vet errors
  - schema/gen/go: remove two common subtest levels
  - use %q in error strings
  - schema/gen/go: please vet a bit more
  - Introduce 'quip' data building helpers. ([ipld/go-ipld-prime#134](https://github.com/ipld/go-ipld-prime/pull/134))
  - gengo: support for unions with stringprefix representation. ([ipld/go-ipld-prime#133](https://github.com/ipld/go-ipld-prime/pull/133))
  - target of opporunity DRY improvement: use more shared templates for structs with stringjoin representations.
  - fix small consistency typo in gen function names.
  - drop old generation mechanisms that were already deprecated.
  - error type cleanup, and helpers.
  - v0.7.0 and changelog update
  - Revert "rename AssignNode to ConvertFrom"
  - Implement traversal.FocusedTransform. ([ipld/go-ipld-prime#130](https://github.com/ipld/go-ipld-prime/pull/130))
  - Update a few more lingering ReprKind references.
  - all: rename schema.Kind to TypeKind, ipld.ReprKind to Kind ([ipld/go-ipld-prime#127](https://github.com/ipld/go-ipld-prime/pull/127))
  - all: rename AssignNode to ConvertFrom
  - all: rewrite interfaces and APIs to support int64
  - mark v0.6.0
  - clean up node/gendemo regeneration ([ipld/go-ipld-prime#123](https://github.com/ipld/go-ipld-prime/pull/123))
  - cleanup: drop orphaned gitignore file.
  - Schema types rebased to use codegen types for the data ([ipld/go-ipld-prime#107](https://github.com/ipld/go-ipld-prime/pull/107))
  - codegen: assembler for struct with map representation validates all non-optional fields are present ([ipld/go-ipld-prime#121](https://github.com/ipld/go-ipld-prime/pull/121))
  - changelog: backfill.
  - fluent: finish out matrix of helper methods, and fix error handling of the non-Must methods.
  - all: fix a lot of "unkeyed literal" vet warnings
  - node/mixins: use simpler filenames
  - node/gendemo: use the new code generator
  - Merge pull request #96 , originally known as ipld/cidlink-only-usable-as-ptr
  - Codec revamp ([ipld/go-ipld-prime#112](https://github.com/ipld/go-ipld-prime/pull/112))
  - Allow overridden types (#116) ([ipld/go-ipld-prime#116](https://github.com/ipld/go-ipld-prime/pull/116))
  - add import to ipld in ipldsch_types.go ([ipld/go-ipld-prime#115](https://github.com/ipld/go-ipld-prime/pull/115))
  - Codegen output rearrange ([ipld/go-ipld-prime#105](https://github.com/ipld/go-ipld-prime/pull/105))
  - Validate struct builder sufficiency ([ipld/go-ipld-prime#111](https://github.com/ipld/go-ipld-prime/pull/111))
  - Fresh take on codec APIs, and some tokenization utilities. ([ipld/go-ipld-prime#101](https://github.com/ipld/go-ipld-prime/pull/101))
  - Add a demo ADL (rot13adl) ([ipld/go-ipld-prime#98](https://github.com/ipld/go-ipld-prime/pull/98))
  - Introduce traversal function that selects links out of a tree. ([ipld/go-ipld-prime#110](https://github.com/ipld/go-ipld-prime/pull/110))
  - Codegen various improvements ([ipld/go-ipld-prime#106](https://github.com/ipld/go-ipld-prime/pull/106))
- github.com/libp2p/go-conn-security-multistream (v0.2.0 -> v0.2.1):
  - Implement support for simultaneous open (#14) ([libp2p/go-conn-security-multistream#14](https://github.com/libp2p/go-conn-security-multistream/pull/14))
- github.com/libp2p/go-libp2p (v0.13.0 -> v0.14.2):
  - Fix race in adding connections to connsByPeer ([libp2p/go-libp2p#1116](https://github.com/libp2p/go-libp2p/pull/1116))
  - speed up the mock tests ([libp2p/go-libp2p#1103](https://github.com/libp2p/go-libp2p/pull/1103))
  - remove slow ObservedAddrManager test that doesn't test anything ([libp2p/go-libp2p#1104](https://github.com/libp2p/go-libp2p/pull/1104))
  - remove Codecov config ([libp2p/go-libp2p#1100](https://github.com/libp2p/go-libp2p/pull/1100))
  - doc: document standard connection manager ([libp2p/go-libp2p#1099](https://github.com/libp2p/go-libp2p/pull/1099))
  - run go mod tidy in the examples ([libp2p/go-libp2p#1098](https://github.com/libp2p/go-libp2p/pull/1098))
  - Cleanup some remaining examples nits ([libp2p/go-libp2p#1097](https://github.com/libp2p/go-libp2p/pull/1097))
  - chore: bring examples back into repository and add tests ([libp2p/go-libp2p#1092](https://github.com/libp2p/go-libp2p/pull/1092))
  - fix(mkreleasenotes): handle first commit ([libp2p/go-libp2p#1095](https://github.com/libp2p/go-libp2p/pull/1095))
  - doc: add a basic release process ([libp2p/go-libp2p#1080](https://github.com/libp2p/go-libp2p/pull/1080))
  - chore: update yamux ([libp2p/go-libp2p#1089](https://github.com/libp2p/go-libp2p/pull/1089))
  - fix: re-expose AutoNAT service on BasicHost ([libp2p/go-libp2p#1088](https://github.com/libp2p/go-libp2p/pull/1088))
  - remove NEWS.md ([libp2p/go-libp2p#1086](https://github.com/libp2p/go-libp2p/pull/1086))
  - test: deflake TestProtoDowngrade ([libp2p/go-libp2p#1084](https://github.com/libp2p/go-libp2p/pull/1084))
  - sync: update CI config files (and fix tests) ([libp2p/go-libp2p#1083](https://github.com/libp2p/go-libp2p/pull/1083))
  - static check fixes ([libp2p/go-libp2p#1076](https://github.com/libp2p/go-libp2p/pull/1076))
  - fix go vet ([libp2p/go-libp2p#1075](https://github.com/libp2p/go-libp2p/pull/1075))
  - option for custom dns resolver ([libp2p/go-libp2p#1073](https://github.com/libp2p/go-libp2p/pull/1073))
  - chore: update deps ([libp2p/go-libp2p#1066](https://github.com/libp2p/go-libp2p/pull/1066))
  - fix autonat race ([libp2p/go-libp2p#1062](https://github.com/libp2p/go-libp2p/pull/1062))
  - use transient connections in identify streams ([libp2p/go-libp2p#1061](https://github.com/libp2p/go-libp2p/pull/1061))
  - Emit event for User's NAT Type i.e. Hard NAT or Easy NAT (#1042) ([libp2p/go-libp2p#1042](https://github.com/libp2p/go-libp2p/pull/1042))
  - Finish and Test the simultaneous connect problem in libp2p peers (#1041) ([libp2p/go-libp2p#1041](https://github.com/libp2p/go-libp2p/pull/1041))
  - Close peerstore and document Host Close (#1037) ([libp2p/go-libp2p#1037](https://github.com/libp2p/go-libp2p/pull/1037))
  - Timeout all Identify stream reads (#1032) ([libp2p/go-libp2p#1032](https://github.com/libp2p/go-libp2p/pull/1032))
- github.com/libp2p/go-libp2p-autonat (v0.4.0 -> v0.4.2):
  - Fix: Stream read timeout ([libp2p/go-libp2p-autonat#99](https://github.com/libp2p/go-libp2p-autonat/pull/99))
  - fix: simplify address replacement ([libp2p/go-libp2p-autonat#102](https://github.com/libp2p/go-libp2p-autonat/pull/102))
  - replace the port number for double NAT mapping ([libp2p/go-libp2p-autonat#101](https://github.com/libp2p/go-libp2p-autonat/pull/101))
- github.com/libp2p/go-libp2p-core (v0.8.0 -> v0.8.5):
  - mind the dot.
  - context option for simultaneous connect
  - Event for user's NAT Device Type: Tell user if the node is behind an Easy or Hard NAT (#173) ([libp2p/go-libp2p-core#173](https://github.com/libp2p/go-libp2p-core/pull/173))
  - address aarshian nitpicks
  - make UseTransient context option take a reason argument, for consistency with other options
  - abstract Conn Stat interface for threading
  - Update network/context.go
  - add ErrTransientConn error
  - add support for transient connections
  - more docs for stream fncs (#183) ([libp2p/go-libp2p-core#183](https://github.com/libp2p/go-libp2p-core/pull/183))
  - refactor: use a helper type to decode AddrInfo from JSON (#178) ([libp2p/go-libp2p-core#178](https://github.com/libp2p/go-libp2p-core/pull/178))
  - fix stream docs (#182) ([libp2p/go-libp2p-core#182](https://github.com/libp2p/go-libp2p-core/pull/182))
  - context to force direct dial (#181) ([libp2p/go-libp2p-core#181](https://github.com/libp2p/go-libp2p-core/pull/181))
  - Secure Muxer Interface (#180) ([libp2p/go-libp2p-core#180](https://github.com/libp2p/go-libp2p-core/pull/180))
- github.com/libp2p/go-libp2p-discovery (v0.5.0 -> v0.5.1):
  - Fix hang in BackoffDiscovery.FindPeers when requesting limit lower than number of peers available ([libp2p/go-libp2p-discovery#69](https://github.com/libp2p/go-libp2p-discovery/pull/69))
  - fix staticcheck ([libp2p/go-libp2p-discovery#70](https://github.com/libp2p/go-libp2p-discovery/pull/70))
- github.com/libp2p/go-libp2p-kad-dht (v0.11.1 -> v0.12.2):
  - fullrt rework batching (#720) ([libp2p/go-libp2p-kad-dht#720](https://github.com/libp2p/go-libp2p-kad-dht/pull/720))
  - sync: update CI config files ([libp2p/go-libp2p-kad-dht#712](https://github.com/libp2p/go-libp2p-kad-dht/pull/712))
  - fix staticcheck ([libp2p/go-libp2p-kad-dht#721](https://github.com/libp2p/go-libp2p-kad-dht/pull/721))
  - fix: fullrt dht bug fixes ([libp2p/go-libp2p-kad-dht#719](https://github.com/libp2p/go-libp2p-kad-dht/pull/719))
  - Crawler based DHT client (#709) ([libp2p/go-libp2p-kad-dht#709](https://github.com/libp2p/go-libp2p-kad-dht/pull/709))
  - test: fix unique addr check ([libp2p/go-libp2p-kad-dht#714](https://github.com/libp2p/go-libp2p-kad-dht/pull/714))
  - chore: update deps ([libp2p/go-libp2p-kad-dht#713](https://github.com/libp2p/go-libp2p-kad-dht/pull/713))
  - Add basic crawler (#663) ([libp2p/go-libp2p-kad-dht#663](https://github.com/libp2p/go-libp2p-kad-dht/pull/663))
  - various staticcheck fixes ([libp2p/go-libp2p-kad-dht#710](https://github.com/libp2p/go-libp2p-kad-dht/pull/710))
  - findpeer should work even on peers that are not part of DHT queries ([libp2p/go-libp2p-kad-dht#711](https://github.com/libp2p/go-libp2p-kad-dht/pull/711))
  - Extract DHT message sender from the DHT ([libp2p/go-libp2p-kad-dht#659](https://github.com/libp2p/go-libp2p-kad-dht/pull/659))
- github.com/libp2p/go-libp2p-noise (v0.1.2 -> v0.2.0):
  - Update github.com/flynn/noise to address nonce handling security issues ([libp2p/go-libp2p-noise#95](https://github.com/libp2p/go-libp2p-noise/pull/95))
  - fix staticcheck ([libp2p/go-libp2p-noise#96](https://github.com/libp2p/go-libp2p-noise/pull/96))
  - chore: update deps ([libp2p/go-libp2p-noise#94](https://github.com/libp2p/go-libp2p-noise/pull/94))
  - chore: relicense MIT/Apache-2.0 ([libp2p/go-libp2p-noise#93](https://github.com/libp2p/go-libp2p-noise/pull/93))
- github.com/libp2p/go-libp2p-peerstore (v0.2.6 -> v0.2.7):
  - fix: delete addrs when "updating" them to zero ([libp2p/go-libp2p-peerstore#157](https://github.com/libp2p/go-libp2p-peerstore/pull/157))
- github.com/libp2p/go-libp2p-quic-transport (v0.10.0 -> v0.11.1):
  - update quic-go, enable QUIC v1 (RFC 9000) ([libp2p/go-libp2p-quic-transport#207](https://github.com/libp2p/go-libp2p-quic-transport/pull/207))
  - update quic-go to v0.21.0-rc2 ([libp2p/go-libp2p-quic-transport#206](https://github.com/libp2p/go-libp2p-quic-transport/pull/206))
  - increase test timeout to reduce flakiness of test on Windows ([libp2p/go-libp2p-quic-transport#204](https://github.com/libp2p/go-libp2p-quic-transport/pull/204))
  - correctly export version negotiation failures to Prometheus ([libp2p/go-libp2p-quic-transport#205](https://github.com/libp2p/go-libp2p-quic-transport/pull/205))
  - update quic-go to v0.20.1 ([libp2p/go-libp2p-quic-transport#201](https://github.com/libp2p/go-libp2p-quic-transport/pull/201))
  - expose some Prometheus metrics ([libp2p/go-libp2p-quic-transport#200](https://github.com/libp2p/go-libp2p-quic-transport/pull/200))
  - update quic-go to v0.20.0 ([libp2p/go-libp2p-quic-transport#198](https://github.com/libp2p/go-libp2p-quic-transport/pull/198))
  - reduce the zstd window size from 8 MB to 32 KB ([libp2p/go-libp2p-quic-transport#195](https://github.com/libp2p/go-libp2p-quic-transport/pull/195))
  - compress qlogs when the QUIC connection is closed ([libp2p/go-libp2p-quic-transport#193](https://github.com/libp2p/go-libp2p-quic-transport/pull/193))
  - switch from gzip to zstd for qlog compression ([libp2p/go-libp2p-quic-transport#190](https://github.com/libp2p/go-libp2p-quic-transport/pull/190))
- github.com/libp2p/go-libp2p-swarm (v0.4.0 -> v0.5.0):
  - run connection gating tests on both TCP and QUIC ([libp2p/go-libp2p-swarm#258](https://github.com/libp2p/go-libp2p-swarm/pull/258))
  - fix: avoid returning typed nils ([libp2p/go-libp2p-swarm#257](https://github.com/libp2p/go-libp2p-swarm/pull/257))
  - fix staticcheck ([libp2p/go-libp2p-swarm#255](https://github.com/libp2p/go-libp2p-swarm/pull/255))
  - fix go vet ([libp2p/go-libp2p-swarm#253](https://github.com/libp2p/go-libp2p-swarm/pull/253))
  - New Dialer ([libp2p/go-libp2p-swarm#243](https://github.com/libp2p/go-libp2p-swarm/pull/243))
  - fix: use 64bit stream/conn IDs ([libp2p/go-libp2p-swarm#247](https://github.com/libp2p/go-libp2p-swarm/pull/247))
  - feat: close transports that implement io.Closer ([libp2p/go-libp2p-swarm#227](https://github.com/libp2p/go-libp2p-swarm/pull/227))
  - fix swarm transient conn (#241) ([libp2p/go-libp2p-swarm#241](https://github.com/libp2p/go-libp2p-swarm/pull/241))
  - Support for Hole punching (#233) ([libp2p/go-libp2p-swarm#233](https://github.com/libp2p/go-libp2p-swarm/pull/233))
  - Treat transient connections as opt-in when opening new streams ([libp2p/go-libp2p-swarm#236](https://github.com/libp2p/go-libp2p-swarm/pull/236))
  - avoid assigning a function to a variable ([libp2p/go-libp2p-swarm#239](https://github.com/libp2p/go-libp2p-swarm/pull/239))
  - only listen on localhost in tests ([libp2p/go-libp2p-swarm#238](https://github.com/libp2p/go-libp2p-swarm/pull/238))
  - prevent dialing addresses that we're listening on ([libp2p/go-libp2p-swarm#237](https://github.com/libp2p/go-libp2p-swarm/pull/237))
  - Enable QUIC in Test Swarm (#235) ([libp2p/go-libp2p-swarm#235](https://github.com/libp2p/go-libp2p-swarm/pull/235))
- github.com/libp2p/go-libp2p-transport-upgrader (v0.4.0 -> v0.4.2):
  - Expose underlying transport connection stat where available ([libp2p/go-libp2p-transport-upgrader#71](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/71))
  - Implement support for simultaneous open (#25) ([libp2p/go-libp2p-transport-upgrader#25](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/25))
- github.com/libp2p/go-libp2p-yamux (v0.5.1 -> v0.5.4):
  - remove Makefile ([libp2p/go-libp2p-yamux#35](https://github.com/libp2p/go-libp2p-yamux/pull/35))
- github.com/libp2p/go-netroute (v0.1.3 -> v0.1.6):
  - add js stub impl
- github.com/libp2p/go-sockaddr (v0.0.2 -> v0.1.1):
  - fix: allocate "any" socket type then cast ([libp2p/go-sockaddr#20](https://github.com/libp2p/go-sockaddr/pull/20))
  - fix: remove CGO functions ([libp2p/go-sockaddr#18](https://github.com/libp2p/go-sockaddr/pull/18))
- github.com/libp2p/go-tcp-transport (v0.2.1 -> v0.2.2):
  - use log.Warn instead of log.Warning ([libp2p/go-tcp-transport#77](https://github.com/libp2p/go-tcp-transport/pull/77))
  - add bandwidth-related metrics (for Linux and OSX) ([libp2p/go-tcp-transport#76](https://github.com/libp2p/go-tcp-transport/pull/76))
  - expose some Prometheus metrics ([libp2p/go-tcp-transport#75](https://github.com/libp2p/go-tcp-transport/pull/75))
  - enable TCP keepalives ([libp2p/go-tcp-transport#73](https://github.com/libp2p/go-tcp-transport/pull/73))
  - stop using the deprecated go-multiaddr-net package ([libp2p/go-tcp-transport#72](https://github.com/libp2p/go-tcp-transport/pull/72))
- github.com/libp2p/go-yamux/v2 (v2.0.0 -> v2.2.0):
  - make the initial stream receive window configurable ([libp2p/go-yamux#59](https://github.com/libp2p/go-yamux/pull/59))
  - set initial window size to spec value (256 kB), remove config option ([libp2p/go-yamux#57](https://github.com/libp2p/go-yamux/pull/57))
  - fix: don't change the receive window if we're forcing an update ([libp2p/go-yamux#56](https://github.com/libp2p/go-yamux/pull/56))
  - sync: update CI config files ([libp2p/go-yamux#55](https://github.com/libp2p/go-yamux/pull/55))
  - increase the receive window size if we're sending updates to frequently ([libp2p/go-yamux#54](https://github.com/libp2p/go-yamux/pull/54))
  - remove unused Stream.Shrink() method ([libp2p/go-yamux#52](https://github.com/libp2p/go-yamux/pull/52))
  - remove misleading comment about the MaxMessageSize ([libp2p/go-yamux#50](https://github.com/libp2p/go-yamux/pull/50))
  - clean up the receive window check ([libp2p/go-yamux#49](https://github.com/libp2p/go-yamux/pull/49))
  - don't reslice byte slices taking from the buffer ([libp2p/go-yamux#48](https://github.com/libp2p/go-yamux/pull/48))
  - don't reimplement io.ReadFull ([libp2p/go-yamux#38](https://github.com/libp2p/go-yamux/pull/38))
  - remove the recvLock in the stream ([libp2p/go-yamux#42](https://github.com/libp2p/go-yamux/pull/42))
  - remove the sendLock in the stream ([libp2p/go-yamux#41](https://github.com/libp2p/go-yamux/pull/41))
  - remove misleading statement about NAT traversal ([libp2p/go-yamux#45](https://github.com/libp2p/go-yamux/pull/45))
  - remove .gx directory, add last gx version to README ([libp2p/go-yamux#43](https://github.com/libp2p/go-yamux/pull/43))
  - reduce usage of goto ([libp2p/go-yamux#40](https://github.com/libp2p/go-yamux/pull/40))
  - remove unused error return value in Stream.processFlags ([libp2p/go-yamux#39](https://github.com/libp2p/go-yamux/pull/39))
- github.com/lucas-clemente/quic-go (v0.19.3 -> v0.21.1):
  - add support for Go 1.17 Beta 1 ([lucas-clemente/quic-go#3203](https://github.com/lucas-clemente/quic-go/pull/3203))
  - add a CI test that go mod vendor works ([lucas-clemente/quic-go#3202](https://github.com/lucas-clemente/quic-go/pull/3202))
  - prevent go mod vendor from stumbling over the Go 1.18 file ([lucas-clemente/quic-go#3195](https://github.com/lucas-clemente/quic-go/pull/3195))
  - remove CipherSuiteName and HkdfExtract for Go 1.17 ([lucas-clemente/quic-go#3192](https://github.com/lucas-clemente/quic-go/pull/3192))
  - fix relocation target for cipherSuiteTLS13ByID in Go 1.17
  - use HkdfExtract from x/crypto ([lucas-clemente/quic-go#3173](https://github.com/lucas-clemente/quic-go/pull/3173))
  - add support for QUIC v1, RFC 9000 ([lucas-clemente/quic-go#3190](https://github.com/lucas-clemente/quic-go/pull/3190))
  - use tls.CipherSuiteName, instead of wrapping it in the qtls package ([lucas-clemente/quic-go#3174](https://github.com/lucas-clemente/quic-go/pull/3174))
  - use a pre-generated test vectors to test hkdfExpandLabel ([lucas-clemente/quic-go#3175](https://github.com/lucas-clemente/quic-go/pull/3175))
  - reduce flakiness of packet number generation test ([lucas-clemente/quic-go#3181](https://github.com/lucas-clemente/quic-go/pull/3181))
  - simplify the qtls tests ([lucas-clemente/quic-go#3185](https://github.com/lucas-clemente/quic-go/pull/3185))
  - add support for Go 1.17 (tip) ([lucas-clemente/quic-go#3182](https://github.com/lucas-clemente/quic-go/pull/3182))
  - prevent quic-go from building on Go 1.17 ([lucas-clemente/quic-go#3180](https://github.com/lucas-clemente/quic-go/pull/3180))
  - fix DONT_FRAGMENT error when using a IPv6 connection on Windows ([lucas-clemente/quic-go#3178](https://github.com/lucas-clemente/quic-go/pull/3178))
  - use net.ErrClosed (for Go 1.16) ([lucas-clemente/quic-go#3163](https://github.com/lucas-clemente/quic-go/pull/3163))
  - use the new error types to log the reason why a connection is closed ([lucas-clemente/quic-go#3166](https://github.com/lucas-clemente/quic-go/pull/3166))
  - fix race condition in deadline integration test ([lucas-clemente/quic-go#3165](https://github.com/lucas-clemente/quic-go/pull/3165))
  - add support for QUIC v1 ([lucas-clemente/quic-go#3160](https://github.com/lucas-clemente/quic-go/pull/3160))
  - rework error return values ([lucas-clemente/quic-go#3159](https://github.com/lucas-clemente/quic-go/pull/3159))
  - declare Path MTU probe packets lost with the early retransmit timer ([lucas-clemente/quic-go#3152](https://github.com/lucas-clemente/quic-go/pull/3152))
  - declare the handshake confirmed when receiving an ACK for a 1-RTT packet ([lucas-clemente/quic-go#3148](https://github.com/lucas-clemente/quic-go/pull/3148))
  - trace and qlog version selection / negotiation ([lucas-clemente/quic-go#3153](https://github.com/lucas-clemente/quic-go/pull/3153))
  - set the don't fragment (DF) bit on Windows (#3155) ([lucas-clemente/quic-go#3155](https://github.com/lucas-clemente/quic-go/pull/3155))
  - fix doc comment for Tracer.TracerForConnection ([lucas-clemente/quic-go#3154](https://github.com/lucas-clemente/quic-go/pull/3154))
  - make it possible to associate a ConnectionTracer with a Session ([lucas-clemente/quic-go#3146](https://github.com/lucas-clemente/quic-go/pull/3146))
  - remove the .editorconfig ([lucas-clemente/quic-go#3147](https://github.com/lucas-clemente/quic-go/pull/3147))
  - don't use a lower RTT than 5ms after receiving a Retry packet ([lucas-clemente/quic-go#3129](https://github.com/lucas-clemente/quic-go/pull/3129))
  - don't pass the QUIC version to the StartedConnection event ([lucas-clemente/quic-go#3109](https://github.com/lucas-clemente/quic-go/pull/3109))
  - update the packet numbers in decoding test to the ones from the draft ([lucas-clemente/quic-go#3137](https://github.com/lucas-clemente/quic-go/pull/3137))
  - various amplification limit fixes ([lucas-clemente/quic-go#3132](https://github.com/lucas-clemente/quic-go/pull/3132))
  - fix calculation of the handshake idle timeout ([lucas-clemente/quic-go#3120](https://github.com/lucas-clemente/quic-go/pull/3120))
  - only start PMTUD after handshake confirmation ([lucas-clemente/quic-go#3138](https://github.com/lucas-clemente/quic-go/pull/3138))
  - don't regard PMTU probe packets as outstanding ([lucas-clemente/quic-go#3126](https://github.com/lucas-clemente/quic-go/pull/3126))
  - expose the draft-34 version ([lucas-clemente/quic-go#3100](https://github.com/lucas-clemente/quic-go/pull/3100))
  - clean up the testutils ([lucas-clemente/quic-go#3104](https://github.com/lucas-clemente/quic-go/pull/3104))
  - initialize the congestion controller with the actual max datagram size ([lucas-clemente/quic-go#3107](https://github.com/lucas-clemente/quic-go/pull/3107))
  - make it possible to trace acknowledged packets ([lucas-clemente/quic-go#3134](https://github.com/lucas-clemente/quic-go/pull/3134))
  - avoid type confusion between protocol.PacketType and logging.PacketType ([lucas-clemente/quic-go#3108](https://github.com/lucas-clemente/quic-go/pull/3108))
  - fix duplicate logging of errors when the first error was a timeout error ([lucas-clemente/quic-go#3112](https://github.com/lucas-clemente/quic-go/pull/3112))
  - use a tracer to make the packetization test more useful ([lucas-clemente/quic-go#3136](https://github.com/lucas-clemente/quic-go/pull/3136))
  - improve string representation of timeout errors ([lucas-clemente/quic-go#3118](https://github.com/lucas-clemente/quic-go/pull/3118))
  - fix flaky timeout test ([lucas-clemente/quic-go#3105](https://github.com/lucas-clemente/quic-go/pull/3105))
  - fix calculation of the time for the next keep alive
  - add a 0-RTT test with different connection ID lengths ([lucas-clemente/quic-go#3098](https://github.com/lucas-clemente/quic-go/pull/3098))
  - only run Ginkgo focus detection in staged files in pre-commit hook ([lucas-clemente/quic-go#3099](https://github.com/lucas-clemente/quic-go/pull/3099))
  - allow 0-RTT when flow control windows are increased ([lucas-clemente/quic-go#3096](https://github.com/lucas-clemente/quic-go/pull/3096))
  - improve the 0-RTT rejection integration test ([lucas-clemente/quic-go#3097](https://github.com/lucas-clemente/quic-go/pull/3097))
  - rename config values for flow control limits ([lucas-clemente/quic-go#3089](https://github.com/lucas-clemente/quic-go/pull/3089))
  - allow 0-RTT resumption if the server's stream limit was increased ([lucas-clemente/quic-go#3086](https://github.com/lucas-clemente/quic-go/pull/3086))
  - cache the serialized OOB in the conn, not in the packet info  ([lucas-clemente/quic-go#3093](https://github.com/lucas-clemente/quic-go/pull/3093))
  - use code points from x/sys/unix for PKTINFO syscalls ([lucas-clemente/quic-go#3094](https://github.com/lucas-clemente/quic-go/pull/3094))
  - make it possible to detect version negotiation failures in logging, fix qlogging of those ([lucas-clemente/quic-go#3092](https://github.com/lucas-clemente/quic-go/pull/3092))
  - make the initial stream / connection flow control windows configurable ([lucas-clemente/quic-go#3083](https://github.com/lucas-clemente/quic-go/pull/3083))
  - only apply server's transport parameters after handshake completion ([lucas-clemente/quic-go#3085](https://github.com/lucas-clemente/quic-go/pull/3085))
  - fix documentation for baseFlowController.UpdateSendWindow ([lucas-clemente/quic-go#3087](https://github.com/lucas-clemente/quic-go/pull/3087))
  - set the Content-Length for HTTP/3 responses ([lucas-clemente/quic-go#3091](https://github.com/lucas-clemente/quic-go/pull/3091))
  - update the flow control windows of streams opened in 0-RTT ([lucas-clemente/quic-go#3088](https://github.com/lucas-clemente/quic-go/pull/3088))
  - Use the correct source IP when binding multiple IPs ([lucas-clemente/quic-go#3067](https://github.com/lucas-clemente/quic-go/pull/3067))
  - fix race condition when receiving 0-RTT packets ([lucas-clemente/quic-go#3074](https://github.com/lucas-clemente/quic-go/pull/3074))
  - require the application to handle 0-RTT rejection ([lucas-clemente/quic-go#3066](https://github.com/lucas-clemente/quic-go/pull/3066))
  - add an internal queue to signal that a datagram frame has been dequeued ([lucas-clemente/quic-go#3081](https://github.com/lucas-clemente/quic-go/pull/3081))
  - increase the maximum size of DATAGRAM frames ([lucas-clemente/quic-go#2966](https://github.com/lucas-clemente/quic-go/pull/2966))
  - remove non-functioning 0-RTT test with different conn ID lengths ([lucas-clemente/quic-go#3079](https://github.com/lucas-clemente/quic-go/pull/3079))
  - remove stray struct equality check ([lucas-clemente/quic-go#3078](https://github.com/lucas-clemente/quic-go/pull/3078))
  - fix issuing of connection IDs when dialing a 0-RTT connections ([lucas-clemente/quic-go#3058](https://github.com/lucas-clemente/quic-go/pull/3058))
  - only accept 0-RTT it the active_connection_id_limit didn't change ([lucas-clemente/quic-go#3060](https://github.com/lucas-clemente/quic-go/pull/3060))
  - remove unused error return value from HandleMaxStreamsFrame ([lucas-clemente/quic-go#3072](https://github.com/lucas-clemente/quic-go/pull/3072))
  - fix flaky accept queue integration test ([lucas-clemente/quic-go#3068](https://github.com/lucas-clemente/quic-go/pull/3068))
  - don't reset the QPACK encoder / decoder streams ([lucas-clemente/quic-go#3063](https://github.com/lucas-clemente/quic-go/pull/3063))
  - remove incorrect logging for client side retry packet ([lucas-clemente/quic-go#3071](https://github.com/lucas-clemente/quic-go/pull/3071))
  - allow sending 1xx responses (#3047) ([lucas-clemente/quic-go#3047](https://github.com/lucas-clemente/quic-go/pull/3047))
  - fix retry key and nonce for draft-34 ([lucas-clemente/quic-go#3062](https://github.com/lucas-clemente/quic-go/pull/3062))
  - implement DPLPMTUD ([lucas-clemente/quic-go#3028](https://github.com/lucas-clemente/quic-go/pull/3028))
  - only read multiple packets at a time after handshake completion ([lucas-clemente/quic-go#3041](https://github.com/lucas-clemente/quic-go/pull/3041))
  - make the certificate verificiation integration tests more explicit ([lucas-clemente/quic-go#3040](https://github.com/lucas-clemente/quic-go/pull/3040))
  - update gomock to v1.5.0, use mockgen source mode ([lucas-clemente/quic-go#3049](https://github.com/lucas-clemente/quic-go/pull/3049))
  - trace dropping of 0-RTT keys ([lucas-clemente/quic-go#3054](https://github.com/lucas-clemente/quic-go/pull/3054))
  - improve timeout measurement in the timeout test ([lucas-clemente/quic-go#3042](https://github.com/lucas-clemente/quic-go/pull/3042))
  - add a randomized test for the received_packet_history ([lucas-clemente/quic-go#3052](https://github.com/lucas-clemente/quic-go/pull/3052))
  - fix documentation of default values for MaxReceive{Stream, Connection}FlowControlWindow ([lucas-clemente/quic-go#3055](https://github.com/lucas-clemente/quic-go/pull/3055))
  - refactor merge packet number ranges ([lucas-clemente/quic-go#3051](https://github.com/lucas-clemente/quic-go/pull/3051))
  - add draft-34 to support versions in README
  - update README to reflect dropped Go 1.14 support
  - remove redundant nil-check in the packet packer  ([lucas-clemente/quic-go#3048](https://github.com/lucas-clemente/quic-go/pull/3048))
  - avoid using rand.Source ([lucas-clemente/quic-go#3046](https://github.com/lucas-clemente/quic-go/pull/3046))
  - update Go to 1.16, drop support for 1.14 ([lucas-clemente/quic-go#3045](https://github.com/lucas-clemente/quic-go/pull/3045))
  - fix error message when the UDP receive buffer size can't be increased ([lucas-clemente/quic-go#3039](https://github.com/lucas-clemente/quic-go/pull/3039))
  - add the time_format field to qlog common_fields ([lucas-clemente/quic-go#3038](https://github.com/lucas-clemente/quic-go/pull/3038))
  - log connection IDs without the 0x prefix ([lucas-clemente/quic-go#3036](https://github.com/lucas-clemente/quic-go/pull/3036))
  - add support for QUIC draft-34 ([lucas-clemente/quic-go#3031](https://github.com/lucas-clemente/quic-go/pull/3031))
  - fix qtls imports in mockgen generated mocks ([lucas-clemente/quic-go#3037](https://github.com/lucas-clemente/quic-go/pull/3037))
  - improve error message when the read buffer size can't be set ([lucas-clemente/quic-go#3030](https://github.com/lucas-clemente/quic-go/pull/3030))
  - qlog the quic-go version ([lucas-clemente/quic-go#3033](https://github.com/lucas-clemente/quic-go/pull/3033))
  - remove the metrics package ([lucas-clemente/quic-go#3032](https://github.com/lucas-clemente/quic-go/pull/3032))
  - expose the constructor for the qlog connection tracer ([lucas-clemente/quic-go#3034](https://github.com/lucas-clemente/quic-go/pull/3034))
  - expose the constructor for the multipexed connection tracer ([lucas-clemente/quic-go#3035](https://github.com/lucas-clemente/quic-go/pull/3035))
  - make sure the server is stopped before closing all server sessions ([lucas-clemente/quic-go#3020](https://github.com/lucas-clemente/quic-go/pull/3020))
  - increase the size of the send queue ([lucas-clemente/quic-go#3016](https://github.com/lucas-clemente/quic-go/pull/3016))
  - prioritize receiving packets over sending out more packets ([lucas-clemente/quic-go#3015](https://github.com/lucas-clemente/quic-go/pull/3015))
  - reenable key updates for HTTP/3 ([lucas-clemente/quic-go#3017](https://github.com/lucas-clemente/quic-go/pull/3017))
  - check for errors after handling each previously undecryptable packet ([lucas-clemente/quic-go#3011](https://github.com/lucas-clemente/quic-go/pull/3011))
  - fix flaky streams map test on Windows ([lucas-clemente/quic-go#3013](https://github.com/lucas-clemente/quic-go/pull/3013))
  - fix flaky stream cancelation integration test ([lucas-clemente/quic-go#3014](https://github.com/lucas-clemente/quic-go/pull/3014))
  - preallocate a slice of one frame when packing a packet ([lucas-clemente/quic-go#3018](https://github.com/lucas-clemente/quic-go/pull/3018))
  - allow sending of ACKs when pacing limited ([lucas-clemente/quic-go#3010](https://github.com/lucas-clemente/quic-go/pull/3010))
  - fix qlogging of the packet payload length ([lucas-clemente/quic-go#3004](https://github.com/lucas-clemente/quic-go/pull/3004))
  - corrupt more ACKs in the MITM test ([lucas-clemente/quic-go#3007](https://github.com/lucas-clemente/quic-go/pull/3007))
  - fix flaky key update integration test ([lucas-clemente/quic-go#3005](https://github.com/lucas-clemente/quic-go/pull/3005))
  - immediately complete streams that were canceled, drop retransmissions ([lucas-clemente/quic-go#3003](https://github.com/lucas-clemente/quic-go/pull/3003))
  - stop generating new packets when the send queue is full ([lucas-clemente/quic-go#2971](https://github.com/lucas-clemente/quic-go/pull/2971))
  - allow access to the underlying quic.Stream from a http.ResponseWriter ([lucas-clemente/quic-go#2993](https://github.com/lucas-clemente/quic-go/pull/2993))
  - remove stay print statement from session test
  - allow receiving of multiple packets before sending a packet ([lucas-clemente/quic-go#2984](https://github.com/lucas-clemente/quic-go/pull/2984))
  - use cryptographic random for determining skipped packet numbers ([lucas-clemente/quic-go#2940](https://github.com/lucas-clemente/quic-go/pull/2940))
  - fix interpretation of time.Time{} as a pacing deadline ([lucas-clemente/quic-go#2980](https://github.com/lucas-clemente/quic-go/pull/2980))
  - qlog restored transport parameters ([lucas-clemente/quic-go#2991](https://github.com/lucas-clemente/quic-go/pull/2991))
  - use a pkg.go.dev instead of a GoDoc badge ([lucas-clemente/quic-go#2982](https://github.com/lucas-clemente/quic-go/pull/2982))
  - introduce a separate queue for undecryptable packets ([lucas-clemente/quic-go#2988](https://github.com/lucas-clemente/quic-go/pull/2988))
  - improve 0-RTT queue ([lucas-clemente/quic-go#2990](https://github.com/lucas-clemente/quic-go/pull/2990))
  - simplify switch statement in the transport parameter parser ([lucas-clemente/quic-go#2995](https://github.com/lucas-clemente/quic-go/pull/2995))
  - remove unneeded overflow check when parsing the max_ack_delay ([lucas-clemente/quic-go#2996](https://github.com/lucas-clemente/quic-go/pull/2996))
  - remove unneeded check in receivedPacketHandler.IsPotentiallyDuplicate ([lucas-clemente/quic-go#2998](https://github.com/lucas-clemente/quic-go/pull/2998))
  - qlog the max_datagram_frame_size transport parameter ([lucas-clemente/quic-go#2997](https://github.com/lucas-clemente/quic-go/pull/2997))
  - qlog draft-02 fixes ([lucas-clemente/quic-go#2987](https://github.com/lucas-clemente/quic-go/pull/2987))
  - fix flaky qlog test ([lucas-clemente/quic-go#2981](https://github.com/lucas-clemente/quic-go/pull/2981))
  - only run gofumpt on .go files in pre-commit hook ([lucas-clemente/quic-go#2983](https://github.com/lucas-clemente/quic-go/pull/2983))
  - fix outdated comment for the http3.Server
  - make the OpenStreamSync cancelation test less flaky ([lucas-clemente/quic-go#2978](https://github.com/lucas-clemente/quic-go/pull/2978))
  - add some useful pre-commit hooks ([lucas-clemente/quic-go#2979](https://github.com/lucas-clemente/quic-go/pull/2979))
  - publicize QUIC varint reading and writing ([lucas-clemente/quic-go#2973](https://github.com/lucas-clemente/quic-go/pull/2973))
  - add a http3.RoundTripOpt to skip the request scheme check ([lucas-clemente/quic-go#2962](https://github.com/lucas-clemente/quic-go/pull/2962))
  - use the standard quic.Config in the deadline tests ([lucas-clemente/quic-go#2970](https://github.com/lucas-clemente/quic-go/pull/2970))
  - update golangci-lint to v1.34.1 ([lucas-clemente/quic-go#2964](https://github.com/lucas-clemente/quic-go/pull/2964))
  - update text about QUIC versions in the README ([lucas-clemente/quic-go#2975](https://github.com/lucas-clemente/quic-go/pull/2975))
  - remove stray TODO in the http3.Server
  - add support for Go 1.16 ([lucas-clemente/quic-go#2953](https://github.com/lucas-clemente/quic-go/pull/2953))
  - cancel reading on unidirectional streams when the stream type is unknown ([lucas-clemente/quic-go#2952](https://github.com/lucas-clemente/quic-go/pull/2952))
  - remove duplicate check of the URL scheme in the HTTP/3 client ([lucas-clemente/quic-go#2956](https://github.com/lucas-clemente/quic-go/pull/2956))
  - increase queueing duration in 0-RTT queue test to reduce flakiness ([lucas-clemente/quic-go#2954](https://github.com/lucas-clemente/quic-go/pull/2954))
  - implement the HTTP/3 Datagram negotiation ([lucas-clemente/quic-go#2951](https://github.com/lucas-clemente/quic-go/pull/2951))
  - implement HTTP/3 control stream handling ([lucas-clemente/quic-go#2949](https://github.com/lucas-clemente/quic-go/pull/2949))
  - fix flaky sentPacketHandler test ([lucas-clemente/quic-go#2950](https://github.com/lucas-clemente/quic-go/pull/2950))
  - don't retransmit PING frames added to ACK-only packets ([lucas-clemente/quic-go#2942](https://github.com/lucas-clemente/quic-go/pull/2942))
  - move the transport parameter stream limit check to the parser  ([lucas-clemente/quic-go#2944](https://github.com/lucas-clemente/quic-go/pull/2944))
  - remove unused initialVersion variable in session ([lucas-clemente/quic-go#2946](https://github.com/lucas-clemente/quic-go/pull/2946))
  - remove unneeded check for the peer's transport parameters ([lucas-clemente/quic-go#2945](https://github.com/lucas-clemente/quic-go/pull/2945))
  - add the H3_MESSAGE_ERROR ([lucas-clemente/quic-go#2947](https://github.com/lucas-clemente/quic-go/pull/2947))
  - simplify Read and Write mock calls in http3 tests ([lucas-clemente/quic-go#2948](https://github.com/lucas-clemente/quic-go/pull/2948))
  - implement the datagram draft ([lucas-clemente/quic-go#2162](https://github.com/lucas-clemente/quic-go/pull/2162))
  - fix logging of bytes_in_flight when receiving an ACK ([lucas-clemente/quic-go#2937](https://github.com/lucas-clemente/quic-go/pull/2937))
  - trace when a packet is dropped because the receivedPackets chan is full ([lucas-clemente/quic-go#2939](https://github.com/lucas-clemente/quic-go/pull/2939))
  - various improvements to the packet number generator ([lucas-clemente/quic-go#2905](https://github.com/lucas-clemente/quic-go/pull/2905))
  - introduce a quic.Config.HandshakeIdleTimeout, remove HandshakeTimeout ([lucas-clemente/quic-go#2930](https://github.com/lucas-clemente/quic-go/pull/2930))
  - allow up to 20 byte for the initial connection IDs ([lucas-clemente/quic-go#2936](https://github.com/lucas-clemente/quic-go/pull/2936))
  - reduce memory footprint of undecryptable packet handling ([lucas-clemente/quic-go#2932](https://github.com/lucas-clemente/quic-go/pull/2932))
  - use a buffer from the pool for composing Retry packets ([lucas-clemente/quic-go#2934](https://github.com/lucas-clemente/quic-go/pull/2934))
  - release the packet buffer after sending a CONNECTION_CLOSE in the server ([lucas-clemente/quic-go#2935](https://github.com/lucas-clemente/quic-go/pull/2935))
  - move integration tests to GitHub Actions, disable Travis ([lucas-clemente/quic-go#2891](https://github.com/lucas-clemente/quic-go/pull/2891))
  - use golang.org/x/sys/unix instead of syscall ([lucas-clemente/quic-go#2927](https://github.com/lucas-clemente/quic-go/pull/2927))
  - add support for the connection_closed qlog event ([lucas-clemente/quic-go#2921](https://github.com/lucas-clemente/quic-go/pull/2921))
  - qlog tokens in NEW_TOKEN frames, Retry packets and Initial packets ([lucas-clemente/quic-go#2863](https://github.com/lucas-clemente/quic-go/pull/2863))
  - qlog the packet_type as part of the packet header, not the event itself ([lucas-clemente/quic-go#2758](https://github.com/lucas-clemente/quic-go/pull/2758))
  - use the new, streaming-friendly NDJSON-based qlog encoding ([lucas-clemente/quic-go#2736](https://github.com/lucas-clemente/quic-go/pull/2736))
  - add a generic Debug() function to the connection tracer ([lucas-clemente/quic-go#2909](https://github.com/lucas-clemente/quic-go/pull/2909))
  - remove unnecessary call to time.Now() when sending a packet ([lucas-clemente/quic-go#2911](https://github.com/lucas-clemente/quic-go/pull/2911))
  - remove support for quic-trace ([lucas-clemente/quic-go#2913](https://github.com/lucas-clemente/quic-go/pull/2913))
  - reduce the maximum number of ACK ranges ([lucas-clemente/quic-go#2887](https://github.com/lucas-clemente/quic-go/pull/2887))
  - don't allocate for acked packets ([lucas-clemente/quic-go#2899](https://github.com/lucas-clemente/quic-go/pull/2899))
  - avoid allocating when detecting lost packets ([lucas-clemente/quic-go#2898](https://github.com/lucas-clemente/quic-go/pull/2898))
  - use the string optimization for map keys in the packet handler map ([lucas-clemente/quic-go#2892](https://github.com/lucas-clemente/quic-go/pull/2892))
  - use a single map in the incoming streams map ([lucas-clemente/quic-go#2890](https://github.com/lucas-clemente/quic-go/pull/2890))
- github.com/marten-seemann/qtls-go1-15 (v0.1.1 -> v0.1.4):
  - use a prefix for client session cache keys
  - add callbacks to store and restore app data along a session state
  - don't use TLS 1.3 compatibility mode when using alternative record layer
  - delete the session ticket after attempting 0-RTT
  - reject 0-RTT when a different ALPN is chosen
  - encode the ALPN into the session ticket
  - add a field to the ConnectionState to tell if 0-RTT was used
  - add a callback to tell the client about rejection of 0-RTT
  - don't offer 0-RTT after a HelloRetryRequest
  - add Accept0RTT to Config callback to decide if 0-RTT should be accepted
- github.com/marten-seemann/qtls-go1-16 (null -> v0.1.3):
  - use a prefix for client session cache keys
  - add callbacks to store and restore app data along a session state
  - don't use TLS 1.3 compatibility mode when using alternative record layer
  - delete the session ticket after attempting 0-RTT
  - reject 0-RTT when a different ALPN is chosen
- github.com/multiformats/go-multiaddr (v0.3.1 -> v0.3.2):
  - fix(net): export new net.Addr conversion registration functions ([multiformats/go-multiaddr#152](https://github.com/multiformats/go-multiaddr/pull/152))
  - sync: run go mod tidy (and set Go 1.15) and gofmt -s in copy workflow (#146) ([multiformats/go-multiaddr#146](https://github.com/multiformats/go-multiaddr/pull/146))
  - more linter fixes ([multiformats/go-multiaddr#145](https://github.com/multiformats/go-multiaddr/pull/145))
  - fix go vet and staticcheck failures ([multiformats/go-multiaddr#143](https://github.com/multiformats/go-multiaddr/pull/143))
  - don't listen on all interfaces in tests, unless on CI ([multiformats/go-multiaddr#136](https://github.com/multiformats/go-multiaddr/pull/136))
  - Fix Local Address on TCP connections ([multiformats/go-multiaddr#135](https://github.com/multiformats/go-multiaddr/pull/135))
- github.com/multiformats/go-multiaddr-dns (v0.2.0 -> v0.3.1):
  - Normalize domains to fqdn for resolver selection ([multiformats/go-multiaddr-dns#27](https://github.com/multiformats/go-multiaddr-dns/pull/27))
  - refactor Resolver to support custom per-TLD resolvers ([multiformats/go-multiaddr-dns#26](https://github.com/multiformats/go-multiaddr-dns/pull/26))
  - feat: exposes backend ([multiformats/go-multiaddr-dns#25](https://github.com/multiformats/go-multiaddr-dns/pull/25))
- github.com/multiformats/go-multihash (v0.0.14 -> v0.0.15):
  - Refactor registry system: no direct dependencies; expose standard hash.Hash; be a data carrier. ([multiformats/go-multihash#136](https://github.com/multiformats/go-multihash/pull/136))
- github.com/multiformats/go-multistream (v0.2.0 -> v0.2.2):
  - change the simultaneous open protocol to /libp2p/simultaneous-connect ([multiformats/go-multistream#66](https://github.com/multiformats/go-multistream/pull/66))
  - fix the lazy stress read test on Windows ([multiformats/go-multistream#61](https://github.com/multiformats/go-multistream/pull/61))
  - fix go vet and staticcheck errors ([multiformats/go-multistream#60](https://github.com/multiformats/go-multistream/pull/60))
  - Implement simultaneous open extension ([multiformats/go-multistream#42](https://github.com/multiformats/go-multistream/pull/42))
  - reduce the number of streams in the stress tests, fix error handling ([multiformats/go-multistream#54](https://github.com/multiformats/go-multistream/pull/54))
- github.com/whyrusleeping/cbor-gen (v0.0.0-20200710004633-5379fc63235d -> v0.0.0-20210219115102-f37d292932f2):
  - feat: allow unmarshalling of struct with more fields than marshaled struct ([whyrusleeping/cbor-gen#50](https://github.com/whyrusleeping/cbor-gen/pull/50))
  - chore: add a license file ([whyrusleeping/cbor-gen#49](https://github.com/whyrusleeping/cbor-gen/pull/49))
  - fix: enforce maxlen in ReadByteArray() ([whyrusleeping/cbor-gen#43](https://github.com/whyrusleeping/cbor-gen/pull/43))
  - use unix nanoseconds for encoding Cbortime ([whyrusleeping/cbor-gen#41](https://github.com/whyrusleeping/cbor-gen/pull/41))
  - add json marshalers to CborTime
  - add a helper for roundtripping time.time objects ([whyrusleeping/cbor-gen#40](https://github.com/whyrusleeping/cbor-gen/pull/40))
  - Add a validate function. ([whyrusleeping/cbor-gen#39](https://github.com/whyrusleeping/cbor-gen/pull/39))
  - Fix import handling ([whyrusleeping/cbor-gen#38](https://github.com/whyrusleeping/cbor-gen/pull/38))
  - Optimize discarding in ScanForLinks ([whyrusleeping/cbor-gen#36](https://github.com/whyrusleeping/cbor-gen/pull/36))
  - Always allocate scratch space when marshalling into a map. ([whyrusleeping/cbor-gen#37](https://github.com/whyrusleeping/cbor-gen/pull/37))
  - optimize byte reading ([whyrusleeping/cbor-gen#35](https://github.com/whyrusleeping/cbor-gen/pull/35))
  - Optimize decoding ([whyrusleeping/cbor-gen#34](https://github.com/whyrusleeping/cbor-gen/pull/34))
  - Fix named string issue ([whyrusleeping/cbor-gen#30](https://github.com/whyrusleeping/cbor-gen/pull/30))
  - Fix encoding/decoding fixed byte arrays ([whyrusleeping/cbor-gen#29](https://github.com/whyrusleeping/cbor-gen/pull/29))
  - fix overread on scanforlinks ([whyrusleeping/cbor-gen#28](https://github.com/whyrusleeping/cbor-gen/pull/28))

### ❤️ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Marten Seemann | 358 | +17444/-12000 | 1268 |
| Eric Myhre | 82 | +9672/-2459 | 328 |
| Ian Davis | 7 | +8421/-737 | 116 |
| Daniel Martí | 18 | +2733/-4377 | 313 |
| Adin Schmahmann | 46 | +5387/-1289 | 125 |
| Steven Allen | 95 | +3278/-1861 | 200 |
| hannahhoward | 14 | +1380/-3667 | 84 |
| gammazero | 29 | +2520/-1161 | 88 |
| Hector Sanjuan | 12 | +511/-3129 | 52 |
| vyzo | 77 | +2198/-940 | 117 |
| Will Scott | 12 | +912/-593 | 37 |
| Dirk McCormick | 3 | +1384/-63 | 14 |
| Andrew Gillis | 3 | +1231/-39 | 19 |
| Marcin Rataj | 37 | +549/-308 | 72 |
| Aarsh Shah | 13 | +668/-86 | 30 |
| Olivier Poitrey | 1 | +469/-182 | 15 |
| Rod Vagg | 9 | +364/-184 | 14 |
| whyrusleeping | 5 | +253/-32 | 11 |
| Cory Schwartz | 10 | +162/-115 | 37 |
| Adrian Lanzafame | 8 | +212/-60 | 11 |
| aarshkshah1992 | 7 | +102/-110 | 9 |
| Jakub Sztandera | 7 | +126/-75 | 16 |
| huoju | 4 | +127/-41 | 6 |
| acruikshank | 6 | +32/-24 | 7 |
| Toby | 1 | +41/-1 | 2 |
| Naveen | 1 | +40/-0 | 1 |
| Bogdan Stirbat | 1 | +22/-16 | 2 |
| Kévin Dunglas | 1 | +32/-2 | 2 |
| Nicholas Bollweg | 1 | +22/-0 | 1 |
| q191201771 | 2 | +4/-11 | 2 |
| Mathis Engelbart | 1 | +12/-2 | 1 |
| requilence | 1 | +13/-0 | 1 |
| divingpetrel | 1 | +7/-4 | 2 |
| Oli Evans | 2 | +9/-2 | 3 |
| Lucas Molas | 3 | +7/-3 | 3 |
| RubenKelevra | 3 | +2/-6 | 3 |
| Will | 1 | +1/-5 | 1 |
| Jorropo | 1 | +4/-2 | 1 |
| Ju Huo | 1 | +2/-2 | 1 |
| zhoujiajie | 1 | +1/-1 | 1 |
| Luflosi | 1 | +1/-1 | 1 |
| Jonathan Rudenberg | 1 | +1/-1 | 1 |
| David Pflug | 1 | +1/-1 | 1 |
| Ari Mattila | 1 | +1/-1 | 1 |
| Yingrong Zhao | 1 | +0/-1 | 1 |
