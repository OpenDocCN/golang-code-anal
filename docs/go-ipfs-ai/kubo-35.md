# go-ipfs 源码解析 35

# go-ipfs changelog v0.10

## v0.10.0 2021-09-30

We're happy to announce go-ipfs 0.10.0. This release brings some big changes to the IPLD internals of go-ipfs that make working with non-UnixFS DAGs easier than ever. There are also a variety of new commands and configuration options available.

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [release process](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

### 🛠 TLDR: BREAKING CHANGES

- `ipfs dag get`
  - default output changed to [`dag-json`](https://ipld.io/specs/codecs/dag-json/spec/)
  - dag-pb (e.g. unixfs) field names changed - impacts userland code that works with `dag-pb` objects returned by `dag get`
  - no longer emits an additional new-line character at the end of the data output
- `ipfs dag put`
  - defaults changed to reduce ambiguity and surprises: input is now assumed to be [`dag-json`](https://ipld.io/specs/codecs/dag-json/spec/), and data is serialized to [`dag-cbor`](https://ipld.io/specs/codecs/dag-cbor/spec/) at rest.
  - `--format` and `--input-enc` were removed and replaced with `--store-codec` and `--input-codec`
  - codec names now match the ones defined in the [multicodec table](https://github.com/multiformats/multicodec/blob/master/table.csv)
  - dag-pb (e.g. unixfs) field names changed - impacts userland code that works with `dag-pb` objects stored via `dag put`

Keep reading to learn more details.

### 🔦 Highlights

#### 🌲 IPLD Levels Up

The handling of data serialization as well as many aspects of DAG traversal and pathing have been migrated from older libraries, including [go-merkledag](https://github.com/ipfs/go-merkledag) and [go-ipld-format](https://github.com/ipfs/go-ipld-format) to the new **[go-ipld-prime](https://github.com/ipld/go-ipld-prime)** library and its components. This allows us to use many of the newer tools afforded by go-ipld-prime, stricter and more uniform codec implementations, support for additional (pluggable) codecs, and some minor performance improvements.

This is significant refactor of a core component that touches many parts of IPFS, and does come with some **breaking changes**:

* **IPLD plugins**:
  * The `PluginIPLD` interface has been changed to utilize go-ipld-prime. There is a demonstration of the change in the [bundled git plugin](./plugin/plugins/git/).
* **The semantics of `dag put` and `dag get` change**:
  * `dag get` now takes the `output-codec` option which accepts a [multicodec](https://docs.ipfs.tech/concepts/glossary/#multicodec) name used to encode the output. By default this is `dag-json`, which is  a strict and deterministic subset of JSON created by the IPLD team. Users may notice differences from the previously plain Go JSON output, particularly where bytes are concerned which are now encoded using a form similar to CIDs: `{"/":{"bytes":"unpadded-base64-bytes"}}` rather than the previously Go-specific plain padded base64 string. See the [dag-json specification](https://ipld.io/specs/codecs/dag-json/spec/) for an explanation of these forms.
  * `dag get` no longer prints an additional new-line character at the end of the encoded block output. This means that the output as presented by `dag get` are the exact bytes of the requested node. A round-trip of such bytes back in through `dag put` using the same codec should result in the same CID.
  * `dag put` uses the `input-codec` option to specify the multicodec name of the format data is being provided in, and the `store-codec` option to specify the multicodec name of the format the data should be stored in at rest. These formerly defaulted to `json` and `cbor` respectively. They now default to `dag-json` and `dag-cbor` respectively but may be changed to any supported codec (bundled or loaded via plugin) by its [multicodec name](https://github.com/multiformats/multicodec/blob/master/table.csv).
  * The `json` and `cbor` multicodec names (as used by `input-enc` and `format` options) are now no longer aliases for `dag-json` and `dag-cbor` respectively. Instead, they now refer to their proper [multicodec](https://github.com/multiformats/multicodec/blob/master/table.csv) types. `cbor` refers to a plain CBOR format, which will not encode CIDs and does not have strict deterministic encoding rules. `json` is a plain JSON format, which also won't encode CIDs and will encode bytes in the Go-specific padded base64 string format rather than the dag-json method of byte encoding. See https://ipld.io/specs/codecs/ for more information on IPLD codecs.
  * `protobuf` is no longer used as the codec name for `dag-pb`
  * The codec name `raw` is used to mean Bytes in the [IPLD Data Model](https://github.com/ipld/specs/blob/master/data-model-layer/data-model.md#bytes-kind)
* **UnixFS refactor**. The **dag-pb codec**, which is used to encode UnixFS data for IPFS, is now represented through the `dag` API in a form that mirrors the protobuf schema used to define the binary format. This unifies the implementations and specification of dag-pb across the IPLD and IPFS stacks. Previously, additional layers of code for file and directory handling within IPFS between protobuf serialization and UnixFS obscured the protobuf representation. Much of this code has now been replaced and there are fewer layers of transformation. This means that interacting with dag-pb data via the `dag` API will use different forms:
  * Previously, using `dag get` on a dag-pb block would present the block serialized as JSON as `{"data":"padded-base64-bytes","links":[{"Name":"foo","Size":100,"Cid":{"/":"Qm..."}},...]}`.
  * Now,  the dag-pb data with dag-json codec for output will be serialized using the data model from the [dag-pb specification](https://ipld.io/specs/codecs/dag-pb/spec/): `{"Data":{"/":{"bytes":"unpadded-base64-bytes"}},"Links":[{"Name":"foo","Tsize":100,"Hash":{"/":"Qm..."}},...]}`. Aside from the change in byte formatting, most field names have changed: `data` → `Data`, `links` → `Links`, `Size` → `Tsize`, `Cid` → `Hash`. Note that this output can be changed now using the `output-codec` option to specify an alternative codec.
  * Similarly, using `dag put` and a `store-codec` option of `dag-pb` now requires that the input conform to this dag-pb specified form. Previously, input using `{"data":"...","links":[...]}` was accepted, now it must be `{"Data":"...","Links":[...]}`.
  * Previously it was not possible to use paths to navigate to any of these properties of a dag-pb node, the only possible paths were named links, e.g. `dag get QmFoo/NamedLink` where `NamedLink` was one of the links whose name was `NamedLink`. This functionality remains the same, but by prefixing the path with `/ipld/` we enter data model pathing semantics and can `dag get /ipld/QmFoo/Links/0/Hash` to navigate to links or `/ipld/QmFoo/Data` to simply retrieve the data section of the node, for example.
  * ℹ See the [dag-pb specification](https://ipld.io/specs/codecs/dag-pb/) for details on the codec and its data model representation.
  * ℹ See this [detailed write-up](https://github.com/ipld/ipld/blob/master/design/tricky-choices/dag-pb-forms-impl-and-use.md) for further background on these changes.

#### Ⓜ Multibase Command

go-ipfs now provides utility commands for working with [multibase](https://docs.ipfs.tech/concepts/glossary/#multibase):

```goconsole
$ echo -n hello | ipfs multibase encode -b base16 > file-mbase16
$ cat file-mbase16
f68656c6c6f

$ ipfs multibase decode file-mbase16
hello

$ cat file-mbase16 | ipfs multibase decode
hello

$ ipfs multibase transcode -b base2 file-mbase16
00110100001100101011011000110110001101111
```

See `ipfs multibase --help` for more examples.

#### 🔨 Bitswap now supports greater configurability

This release adds an [`Internal` section](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#internal) to the configuration file that is designed to help advanced users optimize their setups without needing a custom binary. The `Internal` section is not guaranteed to be the same from release to release and may not be covered by migrations. If you use the `Internal` section you should be making sure to check the config documentation between releases for any changes.

#### 🐚 Programmatic shell completions command

`ipfs commands completion bash` will generate a bash completion script for go-ipfs commands

#### 📜 Profile collection command

Performance profiles can now be collected using `ipfs diag profile`. If you need to do some debugging or have an issue to submit the collected profiles are very useful to have around.

#### 🍎 Mac OS notarized binaries

The go-ipfs and related migration binaries (for both Intel and Apple Sillicon) are now signed and notarized to make Mac OS installation easier.

#### 👨‍👩‍👦 Improved MDNS

There is a completed implementation of the revised libp2p MDNS spec. This should result in better MDNS discovery and better local/offline operation as a result.

#### 🚗 CAR import statistics

`dag import` command now  supports `--stats` option which will include the number of imported blocks and their total size in the output.

#### 🕸 Peering command

This release adds `swarm peering`  command for easy management of  the peering subsystem. Peer in the peering subsystem is maintained to be connected at all times, and gets reconnected on disconnect with a back-off.

See `ipfs swarm peering --help` for more details.

### Changelog

- github.com/ipfs/go-ipfs:
  - fuse: load unixfs adls as their dagpb substrates
  - enable the legacy mDNS implementation
  - test: add dag get --ouput-codec test
  - change ipfs dag get flag name from format to output-codec
  - test: check behavior of loading UnixFS sharded directories with missing shards
  - remove dag put option shortcuts
  - change names of ipfs dag put flags to make changes clearer
  - feat: dag import --stats (#8237) ([ipfs/go-ipfs#8237](https://github.com/ipfs/go-ipfs/pull/8237))
  - feat: ipfs-webui v2.13.0 (#8430) ([ipfs/go-ipfs#8430](https://github.com/ipfs/go-ipfs/pull/8430))
  - feat(cli): add daemon option --agent-version-suffix (#8419) ([ipfs/go-ipfs#8419](https://github.com/ipfs/go-ipfs/pull/8419))
  - feat: multibase transcode command (#8403) ([ipfs/go-ipfs#8403](https://github.com/ipfs/go-ipfs/pull/8403))
  - fix: take the lock while listing peers
  - feature: 'ipfs swarm peering' command (#8147) ([ipfs/go-ipfs#8147](https://github.com/ipfs/go-ipfs/pull/8147))
  - fix(sharness): add extra check in flush=false in files write
  - chore: update IPFS Desktop testing steps (#8393) ([ipfs/go-ipfs#8393](https://github.com/ipfs/go-ipfs/pull/8393))
  - add more buttons; remove some sections covered in the docs; general cleanup
  - Cosmetic fixups in examples (#8325) ([ipfs/go-ipfs#8325](https://github.com/ipfs/go-ipfs/pull/8325))
  - perf: use performance-enhancing FUSE mount options
  - ci: publish Docker images for bifrost-* branches
  - chore: add comments to peerlog plugin about being unsupported
  - test: add unit tests for peerlog config parsing
  - ci: preload peerlog plugin, disable by default
  - fix(mkreleaselog): specify the parent commit when diffing
  - update go-libp2p to v0.15.0-rc.1 ([ipfs/go-ipfs#8354](https://github.com/ipfs/go-ipfs/pull/8354))
  - feat: add 'ipfs multibase' commands (#8180) ([ipfs/go-ipfs#8180](https://github.com/ipfs/go-ipfs/pull/8180))
  - support bitswap configurability (#8268) ([ipfs/go-ipfs#8268](https://github.com/ipfs/go-ipfs/pull/8268))
  - IPLD Prime In IPFS: Target Merge Branch (#7976) ([ipfs/go-ipfs#7976](https://github.com/ipfs/go-ipfs/pull/7976))
  - ci: upgrade to Go 1.16.7 on CI ([ipfs/go-ipfs#8324](https://github.com/ipfs/go-ipfs/pull/8324))
  - Add flag to create parent directories in files cp command ([ipfs/go-ipfs#8340](https://github.com/ipfs/go-ipfs/pull/8340))
  - fix: avoid out of bounds error when rendering short hashes ([ipfs/go-ipfs#8318](https://github.com/ipfs/go-ipfs/pull/8318))
  - fix: remove some deprecated calls ([ipfs/go-ipfs#8296](https://github.com/ipfs/go-ipfs/pull/8296))
  - perf: set an appropriate capacity ([ipfs/go-ipfs#8244](https://github.com/ipfs/go-ipfs/pull/8244))
  - Fix: Use a pointer type on IpfsNode.Peering ([ipfs/go-ipfs#8331](https://github.com/ipfs/go-ipfs/pull/8331))
  - fix: macos notarized fs-repo-migrations (#8333) ([ipfs/go-ipfs#8333](https://github.com/ipfs/go-ipfs/pull/8333))
  - README.md: Add MacPorts to install section ([ipfs/go-ipfs#8220](https://github.com/ipfs/go-ipfs/pull/8220))
  - feat: register first block metric by default ([ipfs/go-ipfs#8332](https://github.com/ipfs/go-ipfs/pull/8332))
  - Build a go-ipfs:extras docker image ([ipfs/go-ipfs#8142](https://github.com/ipfs/go-ipfs/pull/8142))
  - fix/go-ipfs-as-a-library ([ipfs/go-ipfs#8266](https://github.com/ipfs/go-ipfs/pull/8266))
  - Expose additional migration APIs (#8153) ([ipfs/go-ipfs#8153](https://github.com/ipfs/go-ipfs/pull/8153))
  - point ipfs to pinner that syncs on every pin (#8231) ([ipfs/go-ipfs#8231](https://github.com/ipfs/go-ipfs/pull/8231))
  - docs: chocolatey package name
  - Disambiguate online/offline naming in sharness tests ([ipfs/go-ipfs#8254](https://github.com/ipfs/go-ipfs/pull/8254))
  - Rename DOCKER_HOST to TEST_DOCKER_HOST to avoid conflicts ([ipfs/go-ipfs#8283](https://github.com/ipfs/go-ipfs/pull/8283))
  - feat: add an "ipfs diag profile" command ([ipfs/go-ipfs#8291](https://github.com/ipfs/go-ipfs/pull/8291))
  - Merge branch 'release'
  - feat: improve mkreleaslog ([ipfs/go-ipfs#8290](https://github.com/ipfs/go-ipfs/pull/8290))
  - Add test with expected failure for #3503 ([ipfs/go-ipfs#8280](https://github.com/ipfs/go-ipfs/pull/8280))
  - Create PATCH_RELEASE_TEMPLATE.md
  - fix document error ([ipfs/go-ipfs#8271](https://github.com/ipfs/go-ipfs/pull/8271))
  - feat: webui v2.12.4
  - programmatic shell completions ([ipfs/go-ipfs#8043](https://github.com/ipfs/go-ipfs/pull/8043))
  - test: gateway response for bafkqaaa
  - doc(README): update chat links (and misc fixes) ([ipfs/go-ipfs#8222](https://github.com/ipfs/go-ipfs/pull/8222))
  - link to the actual doc (#8126) ([ipfs/go-ipfs#8126](https://github.com/ipfs/go-ipfs/pull/8126))
  - Improve peer hints for pin remote add (#8143) ([ipfs/go-ipfs#8143](https://github.com/ipfs/go-ipfs/pull/8143))
  - fix(mkreleaselog): support multiple commit authors ([ipfs/go-ipfs#8214](https://github.com/ipfs/go-ipfs/pull/8214))
  - fix(mkreleaselog): handle commit 0 ([ipfs/go-ipfs#8121](https://github.com/ipfs/go-ipfs/pull/8121))
  - bump snap to build with Go 1.16
  - chore: update CHANGELOG
  - chore: switch tar-utils dep to ipfs org
  - feat: print error on bootstrap failure ([ipfs/go-ipfs#8166](https://github.com/ipfs/go-ipfs/pull/8166))
  - fix: typo in migration error
  - refactor: improved humanNumber and humanSI
  - feat: humanized durations in stat provide
  - feat: humanized numbers in stat provide
  - feat: add a text output encoding for the stats provide command
  - fix: webui-2.12.3
  - refactor(pinmfs): log error if pre-existing pin failed (#8056) ([ipfs/go-ipfs#8056](https://github.com/ipfs/go-ipfs/pull/8056))
  - config.md: fix typos/improve wording ([ipfs/go-ipfs#8031](https://github.com/ipfs/go-ipfs/pull/8031))
  - fix(peering_test) : Fix the peering_test to check the connection explicitly added ([ipfs/go-ipfs#8140](https://github.com/ipfs/go-ipfs/pull/8140))
  - build: ignore generated files in changelog ([ipfs/go-ipfs#7712](https://github.com/ipfs/go-ipfs/pull/7712))
  - update version to 0.10.0-dev ([ipfs/go-ipfs#8136](https://github.com/ipfs/go-ipfs/pull/8136))
- github.com/ipfs/go-bitswap (v0.3.4 -> v0.4.0):
  - More stats, knobs and tunings (#514) ([ipfs/go-bitswap#514](https://github.com/ipfs/go-bitswap/pull/514))
  - fix: fix a map access race condition in the want index ([ipfs/go-bitswap#523](https://github.com/ipfs/go-bitswap/pull/523))
  - fix: make blockstore cancel test less timing dependent ([ipfs/go-bitswap#507](https://github.com/ipfs/go-bitswap/pull/507))
  - fix(decision): fix a datarace on disconnect ([ipfs/go-bitswap#508](https://github.com/ipfs/go-bitswap/pull/508))
  - optimize the lookup which peers are waiting for a given block ([ipfs/go-bitswap#486](https://github.com/ipfs/go-bitswap/pull/486))
  - fix: hold the task worker lock when starting task workers ([ipfs/go-bitswap#504](https://github.com/ipfs/go-bitswap/pull/504))
  - fix: Nil dereference while using SetSendDontHaves ([ipfs/go-bitswap#488](https://github.com/ipfs/go-bitswap/pull/488))
  - Fix flaky tests in message queue ([ipfs/go-bitswap#497](https://github.com/ipfs/go-bitswap/pull/497))
  - Fix flaky DontHaveTimeoutManger tests ([ipfs/go-bitswap#495](https://github.com/ipfs/go-bitswap/pull/495))
  - sync: update CI config files ([ipfs/go-bitswap#485](https://github.com/ipfs/go-bitswap/pull/485))
- github.com/ipfs/go-blockservice (v0.1.4 -> v0.1.7):
  - update go-bitswap to v0.3.4 ([ipfs/go-blockservice#78](https://github.com/ipfs/go-blockservice/pull/78))
  - fix staticcheck ([ipfs/go-blockservice#75](https://github.com/ipfs/go-blockservice/pull/75))
  - fix: handle missing session exchange in Session ([ipfs/go-blockservice#73](https://github.com/ipfs/go-blockservice/pull/73))
- github.com/ipfs/go-datastore (v0.4.5 -> v0.4.6):
  - sync: update CI config files ([ipfs/go-datastore#175](https://github.com/ipfs/go-datastore/pull/175))
  - speedup tests ([ipfs/go-datastore#177](https://github.com/ipfs/go-datastore/pull/177))
  - test: reduce element count when the race detector is enabled ([ipfs/go-datastore#176](https://github.com/ipfs/go-datastore/pull/176))
  - fix staticcheck ([ipfs/go-datastore#173](https://github.com/ipfs/go-datastore/pull/173))
  - remove Makefile ([ipfs/go-datastore#172](https://github.com/ipfs/go-datastore/pull/172))
- github.com/ipfs/go-ds-badger (v0.2.6 -> v0.2.7):
  - Log start and end of GC rounds ([ipfs/go-ds-badger#115](https://github.com/ipfs/go-ds-badger/pull/115))
- github.com/ipfs/go-fs-lock (v0.0.6 -> v0.0.7):
  - chore: update log ([ipfs/go-fs-lock#24](https://github.com/ipfs/go-fs-lock/pull/24))
  - sync: update CI config files ([ipfs/go-fs-lock#21](https://github.com/ipfs/go-fs-lock/pull/21))
  - fix TestLockedByOthers on Windows ([ipfs/go-fs-lock#19](https://github.com/ipfs/go-fs-lock/pull/19))
- github.com/ipfs/go-ipfs-config (v0.14.0 -> v0.16.0):
  - feat: add Internal and Internal.Bitswap config options
  - feat: add an OptionalInteger type
  - fix: make sure the Priority type properly implements the JSON marshal/unmarshal interfaces
  - fix: remove deprecated calls ([ipfs/go-ipfs-config#138](https://github.com/ipfs/go-ipfs-config/pull/138))
  - sync: update CI config files ([ipfs/go-ipfs-config#132](https://github.com/ipfs/go-ipfs-config/pull/132))
  - remove period, fix staticcheck ([ipfs/go-ipfs-config#131](https://github.com/ipfs/go-ipfs-config/pull/131))
- github.com/ipfs/go-ipfs-pinner (v0.1.1 -> v0.1.2):
  - Fix/minimize rebuild (#15) ([ipfs/go-ipfs-pinner#15](https://github.com/ipfs/go-ipfs-pinner/pull/15))
  - Define ErrNotPinned alongside the Pinner interface
  - fix staticcheck ([ipfs/go-ipfs-pinner#11](https://github.com/ipfs/go-ipfs-pinner/pull/11))
  - fix: remove the rest of the pb backed pinner ([ipfs/go-ipfs-pinner#9](https://github.com/ipfs/go-ipfs-pinner/pull/9))
  - Remove old ipldpinner that has been replaced by dspinner ([ipfs/go-ipfs-pinner#7](https://github.com/ipfs/go-ipfs-pinner/pull/7))
  - optimize CheckIfPinned ([ipfs/go-ipfs-pinner#6](https://github.com/ipfs/go-ipfs-pinner/pull/6))
- github.com/ipfs/go-ipfs-provider (v0.5.1 -> v0.6.1):
  - Update to IPLD Prime (#32) ([ipfs/go-ipfs-provider#32](https://github.com/ipfs/go-ipfs-provider/pull/32))
- github.com/ipfs/go-ipld-git (v0.0.4 -> v0.1.1):
  - return ErrUnexpectedEOF when Decode input is too short
  - Update go-ipld-git to a go-ipld-prime codec (#46) ([ipfs/go-ipld-git#46](https://github.com/ipfs/go-ipld-git/pull/46))
  - fix staticcheck ([ipfs/go-ipld-git#49](https://github.com/ipfs/go-ipld-git/pull/49))
  - change WriteTo to the standard signature ([ipfs/go-ipld-git#47](https://github.com/ipfs/go-ipld-git/pull/47))
  - don't copy mutexes ([ipfs/go-ipld-git#48](https://github.com/ipfs/go-ipld-git/pull/48))
- github.com/ipfs/go-ipns (v0.1.0 -> v0.1.2):
  - fix: remove deprecated calls ([ipfs/go-ipns#30](https://github.com/ipfs/go-ipns/pull/30))
  - remove Makefile ([ipfs/go-ipns#27](https://github.com/ipfs/go-ipns/pull/27))
- github.com/ipfs/go-log/v2 (v2.1.3 -> v2.3.0):
  - Stop defaulting to color output on non-TTY ([ipfs/go-log#116](https://github.com/ipfs/go-log/pull/116))
  - feat: add ability to use custom zap core ([ipfs/go-log#114](https://github.com/ipfs/go-log/pull/114))
  - fix staticcheck ([ipfs/go-log#112](https://github.com/ipfs/go-log/pull/112))
  - test: fix flaky label test ([ipfs/go-log#111](https://github.com/ipfs/go-log/pull/111))
  - per-subsystem log-levels ([ipfs/go-log#109](https://github.com/ipfs/go-log/pull/109))
  - fix: don't panic on invalid log labels ([ipfs/go-log#110](https://github.com/ipfs/go-log/pull/110))
- github.com/ipfs/go-merkledag (v0.3.2 -> v0.4.0):
  - Use IPLD-prime: target merge branch ([ipfs/go-merkledag#67](https://github.com/ipfs/go-merkledag/pull/67))
  - sync: update CI config files ([ipfs/go-merkledag#70](https://github.com/ipfs/go-merkledag/pull/70))
  - staticcheck ([ipfs/go-merkledag#69](https://github.com/ipfs/go-merkledag/pull/69))
  - Fix bug in dagutils MergeDiffs. (#59) ([ipfs/go-merkledag#59](https://github.com/ipfs/go-merkledag/pull/59))
  - chore: add tests to verify allowable data layouts ([ipfs/go-merkledag#58](https://github.com/ipfs/go-merkledag/pull/58))
- github.com/ipfs/go-namesys (v0.3.0 -> v0.3.1):
  - fix: remove deprecated call to pk.Bytes ([ipfs/go-namesys#19](https://github.com/ipfs/go-namesys/pull/19))
- github.com/ipfs/go-path (v0.0.9 -> v0.1.2):
  - fix: give one minute timeouts to function calls instead of block retrievals ([ipfs/go-path#44](https://github.com/ipfs/go-path/pull/44))
  - IPLD Prime In IPFS: Target Merge Branch (#36) ([ipfs/go-path#36](https://github.com/ipfs/go-path/pull/36))
  - remove Makefile ([ipfs/go-path#40](https://github.com/ipfs/go-path/pull/40))
  - sync: update CI config files ([ipfs/go-path#39](https://github.com/ipfs/go-path/pull/39))
- github.com/ipfs/go-peertaskqueue (v0.2.0 -> v0.4.0):
  - add stats
  - Have a configurable maximum active work per peer ([ipfs/go-peertaskqueue#10](https://github.com/ipfs/go-peertaskqueue/pull/10))
  - sync: update CI config files ([ipfs/go-peertaskqueue#13](https://github.com/ipfs/go-peertaskqueue/pull/13))
  - fix staticcheck ([ipfs/go-peertaskqueue#12](https://github.com/ipfs/go-peertaskqueue/pull/12))
  - fix go vet ([ipfs/go-peertaskqueue#11](https://github.com/ipfs/go-peertaskqueue/pull/11))
- github.com/ipfs/go-unixfsnode (null -> v1.1.3):
  - make UnixFSHAMTShard implement the ADL interface (#11) ([ipfs/go-unixfsnode#11](https://github.com/ipfs/go-unixfsnode/pull/11))
- github.com/ipfs/interface-go-ipfs-core (v0.4.0 -> v0.5.1):
  - IPLD In IPFS: Target Merge Branch (#67) ([ipfs/interface-go-ipfs-core#67](https://github.com/ipfs/interface-go-ipfs-core/pull/67))
  - fix staticcheck ([ipfs/interface-go-ipfs-core#72](https://github.com/ipfs/interface-go-ipfs-core/pull/72))
  - remove Makefile ([ipfs/interface-go-ipfs-core#70](https://github.com/ipfs/interface-go-ipfs-core/pull/70))
- github.com/ipld/go-codec-dagpb (v1.2.0 -> v1.3.0):
  - fix staticcheck warnings ([ipld/go-codec-dagpb#29](https://github.com/ipld/go-codec-dagpb/pull/29))
  - update go-ipld-prime, use go:generate
  - allow decoding PBNode fields in any order
  - expose APIs without Reader/Writer overhead
  - preallocate 1KiB on the stack for marshals
  - encode directly with a []byte
  - decode directly with a []byte
  - remove unnecessary xerrors dep
- github.com/ipld/go-ipld-prime (v0.9.1-0.20210324083106-dc342a9917db -> v0.12.2):
  - Printer feature ([ipld/go-ipld-prime#238](https://github.com/ipld/go-ipld-prime/pull/238))
  - schema: keep TypeSystem names ordered
  - schema/dmt: redesign with bindnode and add Compile
  - codec: make cbor and json codecs use ErrUnexpectedEOF
  - bindnode: fix for stringjoin struct emission when first field is the empty string ([ipld/go-ipld-prime#239](https://github.com/ipld/go-ipld-prime/pull/239))
  - schema: typekind names are not capitalized.
  - Bindnode fixes continued ([ipld/go-ipld-prime#233](https://github.com/ipld/go-ipld-prime/pull/233))
  - helper methods for encoding and decoding ([ipld/go-ipld-prime#232](https://github.com/ipld/go-ipld-prime/pull/232))
  - mark v0.12.0
  - Major refactor: extract datamodel package.
    ([ipld/go-ipld-prime#228](https://github.com/ipld/go-ipld-prime/pull/228))
  - Fix ExploreRecursive stopAt condition, add tests, add error return to Explore (#229) ([ipld/go-ipld-prime#229](https://github.com/ipld/go-ipld-prime/pull/229))
  - selector: add tests which are driven by language-agnostic spec fixtures. ([ipld/go-ipld-prime#231](https://github.com/ipld/go-ipld-prime/pull/231))
  - selector: Improve docs for implementers. (#227) ([ipld/go-ipld-prime#227](https://github.com/ipld/go-ipld-prime/pull/227))
  - Bindnode fixes of opportunity ([ipld/go-ipld-prime#226](https://github.com/ipld/go-ipld-prime/pull/226))
  - node/bindnode: redesign the shape of unions in Go ([ipld/go-ipld-prime#223](https://github.com/ipld/go-ipld-prime/pull/223))
  - summary of the v0.11.0 changelog should holler even more about how cool bindnode is.
  - mark v0.11.0
  - node/bindnode: mark as experimental in its godoc.
  - codecs: more docs, a terminology guide, consistency in options. ([ipld/go-ipld-prime#221](https://github.com/ipld/go-ipld-prime/pull/221))
  - Changelog backfill.
  - selectors: docs enhancements, new construction helpers. ([ipld/go-ipld-prime#199](https://github.com/ipld/go-ipld-prime/pull/199))
  - Changelog backfill.
  - Allow parsing of single Null tokens from refmt
  - Add link conditions for 'stop-at' expression in ExploreRecursive selector ([ipld/go-ipld-prime#214](https://github.com/ipld/go-ipld-prime/pull/214))
  - Remove base64 padding for dag-json bytes as per spec
  - node/bindnode: temporarily skip Links schema test
  - test: add test for traversal of typed node links
  - fix: typed links LinkTargetNodePrototype should return ReferencedType
  - Make `go vet` happy
  - Add MapSortMode to MarshalOptions
  - Add {Unm,M}arshalOptions for explicit mode switching for cbor vs dagcbor
  - Sort map entries marshalling dag-cbor
  - node/bindnode: first pass at inferring IPLD schemas
  - Add {Unm,M}arshalOptions for explicit mode switching for json vs dagjson
  - Make tests pass with sorted dag-json output
  - Sort map entries marshalling dag-json
  - Simplify refmt usage
  - Fix failing test using dagjson encoding
  - Fix some failing tests using dagjson
  - Remove pretty-printing
  - Update readme linking to specs and meta repo.
  - Fix example names so they render on go.pkg.dev.
  - fluent/quip: remove in favor of qp
  - node/basic: add Chooser
  - schema: add TypedPrototype
  - node/bindnode: rethink and better document APIs
  - node/tests: cover yet more interface methods
  - node/tests: cover more error cases for scalar kinds
  - node/tests: add more extensive scalar kind tests
  - node/bindnode: start running all schema tests
  - mark v0.10.0
  - More changelog grooming.
  - Changelog grooming.
  - node/tests: put most of the schema test cases here
  - Add more explicit discussion of indicies to ListIterator.
  - node/bindnode: start of a reflect-based Node implementation
  - add DeepEqual and start using it in tests
  - Add enumerate methods to the multicodec registries. ([ipld/go-ipld-prime#176](https://github.com/ipld/go-ipld-prime/pull/176))
  - Make a multicodec.Registry type available. ([ipld/go-ipld-prime#172](https://github.com/ipld/go-ipld-prime/pull/172))
  - fluent/qp: don't panic on string panics
  - Allow emitting & parsing of bytes per dagjson codec spec ([ipld/go-ipld-prime#166](https://github.com/ipld/go-ipld-prime/pull/166))
  - Package docs for dag-cbor.
  - Update package docs.
  - schema/gen/go: apply gofmt automatically ([ipld/go-ipld-prime#163](https://github.com/ipld/go-ipld-prime/pull/163))
  - schema/gen/go: fix remaining vet warnings on generated code
  - schema/gen/go: batch file writes via a bytes.Buffer ([ipld/go-ipld-prime#161](https://github.com/ipld/go-ipld-prime/pull/161))
  - schema/gen/go: avoid Maybe pointers for small types
  - fix readme formatting typo
  - feat(linksystem): add reification to LinkSystem ([ipld/go-ipld-prime#158](https://github.com/ipld/go-ipld-prime/pull/158))
- github.com/libp2p/go-addr-util (v0.0.2 -> v0.1.0):
  - stop using the deprecated go-multiaddr-net package ([libp2p/go-addr-util#34](https://github.com/libp2p/go-addr-util/pull/34))
  - Remove `IsFDCostlyTransport` ([libp2p/go-addr-util#31](https://github.com/libp2p/go-addr-util/pull/31))
- github.com/libp2p/go-libp2p (v0.14.3 -> v0.15.0):
  - chore: update go-tcp-transport to v0.2.8
  - implement the new mDNS spec, move the old mDNS implementation (#1161) ([libp2p/go-libp2p#1161](https://github.com/libp2p/go-libp2p/pull/1161))
  - remove deprecated basichost.New constructor ([libp2p/go-libp2p#1156](https://github.com/libp2p/go-libp2p/pull/1156))
  - Make BasicHost.evtLocalAddrsUpdated event emitter stateful. ([libp2p/go-libp2p#1147](https://github.com/libp2p/go-libp2p/pull/1147))
  - fix: deflake multipro echo test ([libp2p/go-libp2p#1149](https://github.com/libp2p/go-libp2p/pull/1149))
  - fix(basic_host): stream not closed when context done ([libp2p/go-libp2p#1148](https://github.com/libp2p/go-libp2p/pull/1148))
  - chore: update deps ([libp2p/go-libp2p#1141](https://github.com/libp2p/go-libp2p/pull/1141))
  - remove secio from examples ([libp2p/go-libp2p#1143](https://github.com/libp2p/go-libp2p/pull/1143))
  - remove deprecated Filter option ([libp2p/go-libp2p#1132](https://github.com/libp2p/go-libp2p/pull/1132))
  - fix: remove deprecated call ([libp2p/go-libp2p#1136](https://github.com/libp2p/go-libp2p/pull/1136))
  - test: fix flaky example test ([libp2p/go-libp2p#1135](https://github.com/libp2p/go-libp2p/pull/1135))
  - remove deprecated identify.ClientVersion ([libp2p/go-libp2p#1133](https://github.com/libp2p/go-libp2p/pull/1133))
  - remove Go version requirement and note about Go modules from README ([libp2p/go-libp2p#1126](https://github.com/libp2p/go-libp2p/pull/1126))
  - Error assignment fix ([libp2p/go-libp2p#1124](https://github.com/libp2p/go-libp2p/pull/1124))
  - perf/basic_host: Don't handle address change if we hasn't anyone ([libp2p/go-libp2p#1115](https://github.com/libp2p/go-libp2p/pull/1115))
- github.com/libp2p/go-libp2p-core (v0.8.5 -> v0.9.0):
  - feat: remove unused metrics (#208) ([libp2p/go-libp2p-core#208](https://github.com/libp2p/go-libp2p-core/pull/208))
  - feat: keep addresses for longer (#207) ([libp2p/go-libp2p-core#207](https://github.com/libp2p/go-libp2p-core/pull/207))
  - remove deprecated key stretching struct / function (#203) ([libp2p/go-libp2p-core#203](https://github.com/libp2p/go-libp2p-core/pull/203))
  - remove deprecated Bytes method from the Key interface (#204) ([libp2p/go-libp2p-core#204](https://github.com/libp2p/go-libp2p-core/pull/204))
  - remove deprecated functions in the peer package (#205) ([libp2p/go-libp2p-core#205](https://github.com/libp2p/go-libp2p-core/pull/205))
  - remove deprecated constructor for the insecure transport (#206) ([libp2p/go-libp2p-core#206](https://github.com/libp2p/go-libp2p-core/pull/206))
  - feat: add helper functions for working with addr infos (#202) ([libp2p/go-libp2p-core#202](https://github.com/libp2p/go-libp2p-core/pull/202))
  - fix: make timestamps strictly increasing (#201) ([libp2p/go-libp2p-core#201](https://github.com/libp2p/go-libp2p-core/pull/201))
  - ci: use github-actions for compatibility testing (#200) ([libp2p/go-libp2p-core#200](https://github.com/libp2p/go-libp2p-core/pull/200))
  - sync: update CI config files (#189) ([libp2p/go-libp2p-core#189](https://github.com/libp2p/go-libp2p-core/pull/189))
  - remove minimum Go version from README (#199) ([libp2p/go-libp2p-core#199](https://github.com/libp2p/go-libp2p-core/pull/199))
  - remove flaky tests (#194) ([libp2p/go-libp2p-core#194](https://github.com/libp2p/go-libp2p-core/pull/194))
  - reduce default timeouts to 15s (#192) ([libp2p/go-libp2p-core#192](https://github.com/libp2p/go-libp2p-core/pull/192))
  - fix benchmark of key verifications (#190) ([libp2p/go-libp2p-core#190](https://github.com/libp2p/go-libp2p-core/pull/190))
  - fix staticcheck errors (#191) ([libp2p/go-libp2p-core#191](https://github.com/libp2p/go-libp2p-core/pull/191))
  - doc: document Close on Transport (#188) ([libp2p/go-libp2p-core#188](https://github.com/libp2p/go-libp2p-core/pull/188))
  - add a helper function to go directly from a string to an AddrInfo (#184) ([libp2p/go-libp2p-core#184](https://github.com/libp2p/go-libp2p-core/pull/184))
- github.com/libp2p/go-libp2p-http (v0.2.0 -> v0.2.1):
  - remove Makefile ([libp2p/go-libp2p-http#70](https://github.com/libp2p/go-libp2p-http/pull/70))
  - fix staticcheck ([libp2p/go-libp2p-http#67](https://github.com/libp2p/go-libp2p-http/pull/67))
  - Revert "increase buffer size"
  - Increase read buffer size to reduce poll system calls ([libp2p/go-libp2p-http#66](https://github.com/libp2p/go-libp2p-http/pull/66))
- github.com/libp2p/go-libp2p-kad-dht (v0.12.2 -> v0.13.1):
  - Extract validation from ProtocolMessenger ([libp2p/go-libp2p-kad-dht#741](https://github.com/libp2p/go-libp2p-kad-dht/pull/741))
  - remove codecov.yml ([libp2p/go-libp2p-kad-dht#742](https://github.com/libp2p/go-libp2p-kad-dht/pull/742))
  - integrate some basic opentelemetry tracing ([libp2p/go-libp2p-kad-dht#734](https://github.com/libp2p/go-libp2p-kad-dht/pull/734))
  - feat: delete GetValues ([libp2p/go-libp2p-kad-dht#728](https://github.com/libp2p/go-libp2p-kad-dht/pull/728))
  - chore: skip flaky test when race detector is enabled ([libp2p/go-libp2p-kad-dht#731](https://github.com/libp2p/go-libp2p-kad-dht/pull/731))
  - Dont count connection times in usefulness ([libp2p/go-libp2p-kad-dht#660](https://github.com/libp2p/go-libp2p-kad-dht/pull/660))
  - Routing table refresh should NOT block ([libp2p/go-libp2p-kad-dht#705](https://github.com/libp2p/go-libp2p-kad-dht/pull/705))
  - update bootstrapPeers to be func() []peer.AddrInfo (#716) ([libp2p/go-libp2p-kad-dht#716](https://github.com/libp2p/go-libp2p-kad-dht/pull/716))
- github.com/libp2p/go-libp2p-noise (v0.2.0 -> v0.2.2):
  - remove note about go modules in README ([libp2p/go-libp2p-noise#100](https://github.com/libp2p/go-libp2p-noise/pull/100))
  - fix: remove deprecated call to pk.Bytes ([libp2p/go-libp2p-noise#99](https://github.com/libp2p/go-libp2p-noise/pull/99))
- github.com/libp2p/go-libp2p-peerstore (v0.2.7 -> v0.2.8):
  - Fix perfomance issue in updating addr book ([libp2p/go-libp2p-peerstore#141](https://github.com/libp2p/go-libp2p-peerstore/pull/141))
  - Fix test flakes ([libp2p/go-libp2p-peerstore#164](https://github.com/libp2p/go-libp2p-peerstore/pull/164))
  - Only remove records during GC ([libp2p/go-libp2p-peerstore#135](https://github.com/libp2p/go-libp2p-peerstore/pull/135))
  - sync: update CI config files ([libp2p/go-libp2p-peerstore#160](https://github.com/libp2p/go-libp2p-peerstore/pull/160))
  - fix: fix some race conditions in the ds address book ([libp2p/go-libp2p-peerstore#161](https://github.com/libp2p/go-libp2p-peerstore/pull/161))
  - address lints and test failures ([libp2p/go-libp2p-peerstore#159](https://github.com/libp2p/go-libp2p-peerstore/pull/159))
  - stop using the deprecated go-multiaddr-net package ([libp2p/go-libp2p-peerstore#158](https://github.com/libp2p/go-libp2p-peerstore/pull/158))
- github.com/libp2p/go-libp2p-pubsub (v0.4.2 -> v0.5.4):
  - make slowness a warning, with a user configurable threshold
  - reduce log spam from empty heartbeat messages
  - fix: code review
  - add support for custom protocol matching function
  - fix: remove deprecated Bytes call (#436) ([libp2p/go-libp2p-pubsub#436](https://github.com/libp2p/go-libp2p-pubsub/pull/436))
  - cleanup: fix vet and staticcheck failures (#435) ([libp2p/go-libp2p-pubsub#435](https://github.com/libp2p/go-libp2p-pubsub/pull/435))
  - Revert noisy newline changes
  - fix: avoid panic when peer is blacklisted after connection
  - release priority locks early when handling batches
  - don't respawn writer if we fail to open a stream; declare it a peer error
  - batch process dead peer notifications
  - use a priority lock instead of a semaphore
  - do the notification in a goroutine
  - emit new peer notification without holding the semaphore
  - use a semaphore for new peer notifications so that we don't block the event loop
  - don't accumulate pending goroutines from new connections
  - rename RawTracer's DroppedInSubscribe into UndeliverableMessage
  - add a new RawTracer event to track messages dropped in Subscribe
  - add an option to configure the Subscription output queue length
  - fix some comments
  - expose more events for RawTracer
  - Make close concurrent safe
  - Fix close of closed channel
  - Update README to point to correct example directory (#424) ([libp2p/go-libp2p-pubsub#424](https://github.com/libp2p/go-libp2p-pubsub/pull/424))
  - fix: remove deprecated and never used topic descriptors (#423) ([libp2p/go-libp2p-pubsub#423](https://github.com/libp2p/go-libp2p-pubsub/pull/423))
  - Refactor Gossipsub Parameters To Make Them More Configurable (#421) ([libp2p/go-libp2p-pubsub#421](https://github.com/libp2p/go-libp2p-pubsub/pull/421))
  - add tests for gs features and custom protocols
  - add support for custom gossipsub protocols and feature tests
  - RIP travis, Long Live CircleCI (#414) ([libp2p/go-libp2p-pubsub#414](https://github.com/libp2p/go-libp2p-pubsub/pull/414))
  - Ignore transient connections (#412) ([libp2p/go-libp2p-pubsub#412](https://github.com/libp2p/go-libp2p-pubsub/pull/412))
  - demote log spam to debug
  - fix bug
  - add last amount of validation
  - add threshold validation
  - strengthen validation
  - rename checkSignature to checkSigningPolicy
  - rename validation.Publish to PushLocal
  - fix TestValidate, add TestValidate2
  - skip flaky test until we can fix it
  - implement synchronous validation for locally published messages
  - expose internalTracer as RawTracer
  - export rejection named string constants
  - more intelligent handling of ip whitelist check
  - remove obsolete explicit IP whitelisting in favor of subnets
  - add subnet whitelisting for IPColocation
- github.com/libp2p/go-libp2p-quic-transport (v0.11.2 -> v0.12.0):
  - sync: update CI config files (#228) ([libp2p/go-libp2p-quic-transport#228](https://github.com/libp2p/go-libp2p-quic-transport/pull/228))
  - fix closing of streams in example ([libp2p/go-libp2p-quic-transport#221](https://github.com/libp2p/go-libp2p-quic-transport/pull/221))
  - close all UDP connections when the reuse is closed ([libp2p/go-libp2p-quic-transport#216](https://github.com/libp2p/go-libp2p-quic-transport/pull/216))
  - fix staticcheck ([libp2p/go-libp2p-quic-transport#217](https://github.com/libp2p/go-libp2p-quic-transport/pull/217))
  - sync: update CI config files (#214) ([libp2p/go-libp2p-quic-transport#214](https://github.com/libp2p/go-libp2p-quic-transport/pull/214))
  - implement a Transport.Close that waits for the reuse's GC to finish ([libp2p/go-libp2p-quic-transport#211](https://github.com/libp2p/go-libp2p-quic-transport/pull/211))
  - don't compare peer IDs when hole punching ([libp2p/go-libp2p-quic-transport#210](https://github.com/libp2p/go-libp2p-quic-transport/pull/210))
  - add hole punching support (#194) ([libp2p/go-libp2p-quic-transport#194](https://github.com/libp2p/go-libp2p-quic-transport/pull/194))
- github.com/libp2p/go-libp2p-swarm (v0.5.0 -> v0.5.3):
  - sync: update CI config files ([libp2p/go-libp2p-swarm#263](https://github.com/libp2p/go-libp2p-swarm/pull/263))
  - remove incorrect call to InterceptAddrDial ([libp2p/go-libp2p-swarm#260](https://github.com/libp2p/go-libp2p-swarm/pull/260))
  - speed up the TestFDLimitUnderflow test ([libp2p/go-libp2p-swarm#262](https://github.com/libp2p/go-libp2p-swarm/pull/262))
  - sync: update CI config files (#248) ([libp2p/go-libp2p-swarm#248](https://github.com/libp2p/go-libp2p-swarm/pull/248))
- github.com/libp2p/go-libp2p-testing (v0.4.0 -> v0.4.2):
  - fix deadlock in the transport's serve function ([libp2p/go-libp2p-testing#35](https://github.com/libp2p/go-libp2p-testing/pull/35))
  - fix: cleanup transport suite ([libp2p/go-libp2p-testing#34](https://github.com/libp2p/go-libp2p-testing/pull/34))
  - Address `go vet` and `saticcheck` issues ([libp2p/go-libp2p-testing#33](https://github.com/libp2p/go-libp2p-testing/pull/33))
  - Defer closing stream for reading ([libp2p/go-libp2p-testing#32](https://github.com/libp2p/go-libp2p-testing/pull/32))
- github.com/libp2p/go-libp2p-tls (v0.1.3 -> v0.2.0):
  - fix: don't fail the handshake when the libp2p extension is critical ([libp2p/go-libp2p-tls#88](https://github.com/libp2p/go-libp2p-tls/pull/88))
  - fix deprecated call to key.Bytes ([libp2p/go-libp2p-tls#86](https://github.com/libp2p/go-libp2p-tls/pull/86))
  - fix usage of deprecated peer.IDB58Decode ([libp2p/go-libp2p-tls#77](https://github.com/libp2p/go-libp2p-tls/pull/77))
  - remove setting of the TLS 1.3 GODEBUG flag ([libp2p/go-libp2p-tls#68](https://github.com/libp2p/go-libp2p-tls/pull/68))
  - improve the error message returned when peer verification fails ([libp2p/go-libp2p-tls#57](https://github.com/libp2p/go-libp2p-tls/pull/57))
  - update to Go 1.14 ([libp2p/go-libp2p-tls#54](https://github.com/libp2p/go-libp2p-tls/pull/54))
  - Update deps and fix tests ([libp2p/go-libp2p-tls#43](https://github.com/libp2p/go-libp2p-tls/pull/43))
- github.com/libp2p/go-libp2p-transport-upgrader (v0.4.2 -> v0.4.6):
  - chore: update deps ([libp2p/go-libp2p-transport-upgrader#78](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/78))
  - fix typo in error message ([libp2p/go-libp2p-transport-upgrader#77](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/77))
  - fix staticcheck ([libp2p/go-libp2p-transport-upgrader#74](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/74))
  - don't listen on all interfaces in tests ([libp2p/go-libp2p-transport-upgrader#73](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/73))
  - stop using the deprecated go-multiaddr-net ([libp2p/go-libp2p-transport-upgrader#72](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/72))
- github.com/libp2p/go-libp2p-xor (v0.0.0-20200501025846-71e284145d58 -> v0.0.0-20210714161855-5c005aca55db):
  - Add immutable remove operation ([libp2p/go-libp2p-xor#14](https://github.com/libp2p/go-libp2p-xor/pull/14))
  - fix go vet and staticcheck ([libp2p/go-libp2p-xor#11](https://github.com/libp2p/go-libp2p-xor/pull/11))
- github.com/libp2p/go-reuseport-transport (v0.0.4 -> v0.0.5):
  - remove note about Go modules in README ([libp2p/go-reuseport-transport#32](https://github.com/libp2p/go-reuseport-transport/pull/32))
  - stop using the deprecated go-multiaddr-net package ([libp2p/go-reuseport-transport#30](https://github.com/libp2p/go-reuseport-transport/pull/30))
- github.com/libp2p/go-socket-activation (v0.0.2 -> v0.1.0):
  - chore: stop using the deprecated go-multiaddr-net package ([libp2p/go-socket-activation#16](https://github.com/libp2p/go-socket-activation/pull/16))
  - fix staticcheck ([libp2p/go-socket-activation#13](https://github.com/libp2p/go-socket-activation/pull/13))
- github.com/libp2p/go-tcp-transport (v0.2.4 -> v0.2.8):
  - disable metrics collection on Windows ([libp2p/go-tcp-transport#93](https://github.com/libp2p/go-tcp-transport/pull/93))
  - sync: update CI config files (#90) ([libp2p/go-tcp-transport#90](https://github.com/libp2p/go-tcp-transport/pull/90))
  - chore: update go-libp2p-transport-upgrader and go-reuseport-transport ([libp2p/go-tcp-transport#84](https://github.com/libp2p/go-tcp-transport/pull/84))
- github.com/libp2p/go-ws-transport (v0.4.0 -> v0.5.0):
  - chore: update go-libp2p-transport-upgrader and go-libp2p-core ([libp2p/go-ws-transport#103](https://github.com/libp2p/go-ws-transport/pull/103))
  - remove deprecated type ([libp2p/go-ws-transport#102](https://github.com/libp2p/go-ws-transport/pull/102))
  - sync: update CI config files ([libp2p/go-ws-transport#101](https://github.com/libp2p/go-ws-transport/pull/101))
  - chore: various cleanups required to get vet/staticcheck/test to pass ([libp2p/go-ws-transport#100](https://github.com/libp2p/go-ws-transport/pull/100))
- github.com/lucas-clemente/quic-go (v0.21.2 -> v0.23.0):
  - update to Go 1.17.x ([lucas-clemente/quic-go#3258](https://github.com/lucas-clemente/quic-go/pull/3258))
  - quicvarint: export Min and Max (#3253) ([lucas-clemente/quic-go#3253](https://github.com/lucas-clemente/quic-go/pull/3253))
  - drop support for Go 1.15 ([lucas-clemente/quic-go#3247](https://github.com/lucas-clemente/quic-go/pull/3247))
  - quicvarint: add Reader and Writer interfaces (#3233) ([lucas-clemente/quic-go#3233](https://github.com/lucas-clemente/quic-go/pull/3233))
  - fix race when stream.Read and CancelRead are called concurrently ([lucas-clemente/quic-go#3241](https://github.com/lucas-clemente/quic-go/pull/3241))
  - also count coalesced 0-RTT packets in the integration tests ([lucas-clemente/quic-go#3251](https://github.com/lucas-clemente/quic-go/pull/3251))
  - remove draft versions 32 and 34 from README (#3244) ([lucas-clemente/quic-go#3244](https://github.com/lucas-clemente/quic-go/pull/3244))
  - update Changelog ([lucas-clemente/quic-go#3245](https://github.com/lucas-clemente/quic-go/pull/3245))
  - optimize hasOutstandingCryptoPackets in sentPacketHandler ([lucas-clemente/quic-go#3230](https://github.com/lucas-clemente/quic-go/pull/3230))
  - permit underlying conn to implement batch interface directly ([lucas-clemente/quic-go#3237](https://github.com/lucas-clemente/quic-go/pull/3237))
  - cancel the PTO timer when all Handshake packets are acknowledged ([lucas-clemente/quic-go#3231](https://github.com/lucas-clemente/quic-go/pull/3231))
  - fix flaky INVALID_TOKEN server test ([lucas-clemente/quic-go#3223](https://github.com/lucas-clemente/quic-go/pull/3223))
  - drop support for QUIC draft version 32 and 34 ([lucas-clemente/quic-go#3217](https://github.com/lucas-clemente/quic-go/pull/3217))
  - fix flaky 0-RTT integration test ([lucas-clemente/quic-go#3224](https://github.com/lucas-clemente/quic-go/pull/3224))
  - use batched reads ([lucas-clemente/quic-go#3142](https://github.com/lucas-clemente/quic-go/pull/3142))
  - add a config option to disable sending of Version Negotiation packets ([lucas-clemente/quic-go#3216](https://github.com/lucas-clemente/quic-go/pull/3216))
  - remove the RetireBugBackwardsCompatibilityMode ([lucas-clemente/quic-go#3213](https://github.com/lucas-clemente/quic-go/pull/3213))
  - remove outdated ackhandler test case ([lucas-clemente/quic-go#3212](https://github.com/lucas-clemente/quic-go/pull/3212))
  - remove unused StripGreasedVersions function ([lucas-clemente/quic-go#3214](https://github.com/lucas-clemente/quic-go/pull/3214))
  - fix incorrect usage of errors.Is ([lucas-clemente/quic-go#3215](https://github.com/lucas-clemente/quic-go/pull/3215))
  - return error on SendMessage when session is closed ([lucas-clemente/quic-go#3218](https://github.com/lucas-clemente/quic-go/pull/3218))
  - remove a redundant error check ([lucas-clemente/quic-go#3210](https://github.com/lucas-clemente/quic-go/pull/3210))
  - update golangci-lint to v1.41.1 ([lucas-clemente/quic-go#3205](https://github.com/lucas-clemente/quic-go/pull/3205))
  - Update doc for dialer in http3.RoundTripper ([lucas-clemente/quic-go#3208](https://github.com/lucas-clemente/quic-go/pull/3208))
- github.com/multiformats/go-multiaddr (v0.3.3 -> v0.4.0):
  - remove forced dependency on deprecated go-maddr-filter ([multiformats/go-multiaddr#162](https://github.com/multiformats/go-multiaddr/pull/162))
  - remove deprecated SwapToP2pMultiaddrs ([multiformats/go-multiaddr#161](https://github.com/multiformats/go-multiaddr/pull/161))
  - remove Makefile ([multiformats/go-multiaddr#163](https://github.com/multiformats/go-multiaddr/pull/163))
  - remove deprecated filter functions ([multiformats/go-multiaddr#157](https://github.com/multiformats/go-multiaddr/pull/157))
  - remove deprecated NetCodec ([multiformats/go-multiaddr#159](https://github.com/multiformats/go-multiaddr/pull/159))
  - add Noise ([multiformats/go-multiaddr#156](https://github.com/multiformats/go-multiaddr/pull/156))
  - Add TLS protocol ([multiformats/go-multiaddr#153](https://github.com/multiformats/go-multiaddr/pull/153))
- github.com/multiformats/go-multicodec (v0.2.0 -> v0.3.0):
  - Export reserved range constants (#53) ([multiformats/go-multicodec#53](https://github.com/multiformats/go-multicodec/pull/53))
  - make Code.Set accept valid code numbers
  - replace Of with Code.Set, implementing flag.Value
  - add multiformats/multicodec as a git submodule
  - update the generator with the "status" CSV column
  - Run `go generate` to generate the latest codecs
  - Add lookup for multicodec code by string name ([multiformats/go-multicodec#40](https://github.com/multiformats/go-multicodec/pull/40))

### Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Daniel Martí | 42 | +8549/-6587 | 170 |
| Eric Myhre | 55 | +5883/-6715 | 395 |
| Marten Seemann | 100 | +1814/-2028 | 275 |
| Steven Allen | 80 | +1573/-1998 | 127 |
| hannahhoward | 18 | +1721/-671 | 53 |
| Will | 2 | +1114/-1217 | 18 |
| Andrew Gillis | 2 | +1220/-720 | 14 |
| gammazero | 3 | +43/-1856 | 10 |
| Masih H. Derkani | 3 | +960/-896 | 8 |
| Adin Schmahmann | 25 | +1458/-313 | 44 |
| vyzo | 27 | +986/-353 | 60 |
| Will Scott | 6 | +852/-424 | 16 |
| Rod Vagg | 19 | +983/-255 | 66 |
| Petar Maymounkov | 6 | +463/-179 | 22 |
| web3-bot | 10 | +211/-195 | 24 |
| adlrocha | 1 | +330/-75 | 15 |
| RubenKelevra | 2 | +128/-210 | 2 |
| Ian Davis | 3 | +200/-109 | 17 |
| Cory Schwartz | 3 | +231/-33 | 7 |
| Keenan Nemetz | 1 | +184/-71 | 2 |
| Randy Reddig | 2 | +187/-53 | 8 |
| Takashi Matsuda | 3 | +201/-2 | 7 |
| guseggert | 4 | +161/-20 | 9 |
| Lucas Molas | 5 | +114/-47 | 27 |
| nisdas | 4 | +115/-45 | 7 |
| Michael Muré | 6 | +107/-33 | 24 |
| Richard Ramos | 2 | +113/-9 | 3 |
| Marcin Rataj | 12 | +88/-24 | 13 |
| Ondrej Prazak | 2 | +104/-6 | 4 |
| Michal Dobaczewski | 2 | +77/-28 | 3 |
| Jorropo | 3 | +9/-75 | 4 |
| Andey Robins | 1 | +70/-3 | 3 |
| Gus Eggert | 10 | +34/-31 | 12 |
| noot | 1 | +54/-9 | 5 |
| Maxim Merzhanov | 1 | +29/-24 | 1 |
| Adrian Lanzafame | 1 | +30/-13 | 2 |
| Bogdan Stirbat | 1 | +22/-16 | 2 |
| Shad Sterling | 1 | +28/-3 | 1 |
| Jesse Bouwman | 5 | +30/-0 | 5 |
| Pavel Karpy | 1 | +19/-7 | 2 |
| lasiar | 5 | +14/-10 | 5 |
| Dennis Trautwein | 1 | +20/-4 | 2 |
| Louis Thibault | 1 | +22/-1 | 2 |
| whyrusleeping | 2 | +21/-1 | 2 |
| aarshkshah1992 | 3 | +12/-8 | 3 |
| Peter Rabbitson | 2 | +20/-0 | 2 |
| bt90 | 2 | +17/-2 | 2 |
| Dominic Della Valle | 1 | +13/-1 | 2 |
| Audrius Butkevicius | 1 | +12/-1 | 1 |
| Brian Strauch | 1 | +9/-3 | 1 |
| Aarsh Shah | 2 | +1/-11 | 2 |
| Whyrusleeping | 1 | +11/-0 | 1 |
| Max | 1 | +7/-3 | 1 |
| vallder | 1 | +3/-5 | 1 |
| Michael Burns | 3 | +2/-6 | 3 |
| Lasse Johnsen | 1 | +4/-4 | 2 |
| snyh | 1 | +5/-2 | 1 |
| Hector Sanjuan | 2 | +3/-2 | 2 |
| 市川恭佑 (ebi) | 1 | +1/-3 | 1 |
| godcong | 2 | +2/-1 | 2 |
| Mathis Engelbart | 1 | +1/-2 | 1 |
| folbrich | 1 | +1/-1 | 1 |
| Med Mouine | 1 | +1/-1 | 1 |


# go-ipfs changelog v0.11

## v0.11.1 2022-04-08

This patch release fixes a security issue wherein traversing some malformed DAGs can cause the node to panic.

See also the security advisory: https://github.com/ipfs/go-ipfs/security/advisories/GHSA-mcq2-w56r-5w2w

Note: the v0.11.1 patch release contains the Docker compose fix from v0.12.1 as well

### Changelog

<details>
<summary>Full Changelog</summary>
- github.com/ipld/go-codec-dagpb (v1.3.0 -> v1.3.2):
  - fix: use protowire for Links bytes decoding
</details>

### ❤ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Rod Vagg | 1 | +34/-19 | 2 |

## v0.11.0 2021-12-08

We're happy to announce go-ipfs 0.11.0. This release comes with improvements to the UnixFS Sharding and PubSub experiments as well as support for Circuit-Relay v2 which sets the network up for decentralized hole punching support.

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [release process](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

### 🛠 BREAKING CHANGES

-  UnixFS sharding is now automatic and enabled by default
   - HAMT-based sharding is applied to large directories (i.e. those that would serialize into [block](https://docs.ipfs.tech/concepts/glossary/#block) larger than ~256KiB)s. This means importing data via commands like `ipfs add -r <directory>` may result in different [CID](https://docs.ipfs.tech/concepts/glossary/#cid)s due to the different [DAG](https://docs.ipfs.tech/concepts/glossary/#dag) representations.
   - Support for `Experimental.ShardingEnabled` is removed.
- go-ipfs can no longer act as a [Circuit Relay](https://docs.ipfs.tech/concepts/glossary/#circuit-relay) v1
  - Node will refuse to start if `Swarm.EnableRelayHop` is set to `true`
  -  If you depend on v1 relay service provider, see "Removal of v1 relay service" section for available migration options.
- HTTP RPC wire format for experimental commands at  `/api/v0/pubsub` changed.
  - If you use [js-ipfs-http-client](https://www.npmjs.com/package/ipfs-http-client) or [go-ipfs-http-client](https://github.com/ipfs/go-ipfs-http-client), just update to their latest version.
  - If you use something else, see "Multibase in PubSub" section below for migration details.

Keep reading to learn more details.

### 🔦 Highlights

#### 🗃 Automatic UnixFS sharding

Truly big directories can have so many items, that the root block with all of their names is too big to be exchanged with other peers. This was partially solved by [HAMT-sharding](https://docs.ipfs.tech/concepts/glossary/#hamt-sharding), which was introduced a while ago as opt-in. The main downside of the implementation was that it was a global flag that sharded all imported directories (big and small).

This release solves that inconvenience by making UnixFS sharding smarter and applies it only to larger directories (i.e. directories that would be at least ~256KiB). This is now the default behavior in `ipfs add` and `ipfs files` commands, where UnixFS sharding works out-of-the-box.

#### 🔁 Circuit Relay v2

This release adds support for the [circuit relay v2](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md) protocol based on the reference implementation from [go-libp2p 0.16](https://github.com/libp2p/go-libp2p/releases/tag/v0.16.0).

This is the cornerstone for maximizing p2p connections between IPFS peers. Every publicly dialable peer can now act as a limited relay v2, which can be used for [hole punching](https://docs.ipfs.tech/concepts/glossary/#hole-punching) and other decentralized signaling protocols.

##### Limited relay v2 configuration options

go-ipfs can now be configured to act as a [`RelayClient`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayclient)  that uses other peers for autorelay functionality when behind a NAT, or provide a limited [`RelayService`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmrelayservice) to other peers on the network.

Starting with go-ipfs v0.11 every publicly dialable go-ipfs (based on AutoNAT determination) will start a limited `RelayService`.  `RelayClient` remains disabled by default for now, as we want the network to update and get enough v2 service providers first.

Note: the limited Circuit Relay v2 provided with this release only allows low-bandwidth protocols (identify, ping, holepunch) over transient connections. If you want to relay things like bitswap sessions, you need to set up a v1 relay by some other means. See details below.

##### Removal of unlimited v1 relay service provider

Switching to v2 of the relay spec means removal or deprecation of configuration keys that were specific to v1.

- Relay transport and client support circuit-relay v2:
  - `Swarm.EnableAutoRelay` was replaced by `Swarm.RelayClient.Enabled`.
  - `Swarm.DisableRelay` is deprecated, relay transport can be now disabled globally (both client and service) by setting `Swarm.Transports.Network.Relay` to `false`
- Relay v1 service provider was replaced by v2:
  - `Swarm.EnableRelayHop` no longer starts an unlimited v1 relay. If you have it set to `true` the node will refuse to start and display an error message.
  - Existing users who choose to continue running a v1 relay should migrate their setups to relay v1 based on js-ipfs running in node, or the standalone [libp2p-relay-daemon](https://dist.ipfs.tech/#libp2p-relay-daemon) [configured](https://github.com/libp2p/go-libp2p-relay-daemon/#configuration) with `RelayV1.Enabled` set to `true`. Be mindful that v1 relays are unlimited, and one may want to set up some ACL based either on PeerIDs or Subnets.

#### 🕳 Decentralized Hole Punching (DCUtR protocol client)

We are working towards enabling hole punching for NAT traversal when port forwarding is not possible.

[go-libp2p 0.16](https://github.com/libp2p/go-libp2p/releases/tag/v0.16.0) provides an implementation of the [DCUtR (decentralized hole punching)](https://github.com/libp2p/specs/blob/master/relay/DCUtR.md) protocol. It is hidden behind the `Swarm.EnableHolePunching` configuration flag.

When enabled, go-ipfs will coordinate with the counterparty using a [relayed v2 connection](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md), to [upgrade to a direct connection](https://github.com/libp2p/specs/blob/master/relay/DCUtR.md) through a NAT/firewall whenever possible.

This feature is disabled by default in this release, but we hope to enable it by default as soon the network updates to go-ipfs v0.11 and gains a healthy set of limited v2 relays.

#### 💬 Multibase in PubSub HTTP RPC API

This release fixed some edge cases that were reported by users of the PubSub experiment, getting it closer to becoming a stable feature of go-ipfs. Some PubSub users will notice that the plaintext limitation is lifted: one can now use line breaks in messages published to non-ascii topic names, or even publish arbitrary bytes to arbitrary topics.  It required a change to the wire format used when pubsub commands are executed over the HTTP RPC API at `/api/v0/pubsub/*`, and also modified the behavior of the `ipfs pubsub pub` command, which now is publishing only a single pubsub message with data read from a file or stdin.

##### PubSub client migration tips

If you use the HTTP RPC API with the [go-ipfs-http-client](https://github.com/ipfs/go-ipfs-http-client) library, make sure to update to the latest version. The next version of [js-ipfs-http-client](https://www.npmjs.com/package/ipfs-http-client) will use the new wire format as well, so you don't need to do anything.

If you use `/api/v0/pubsub/*` directly or maintain your own client library, you must adjust your HTTP client code. Byte fields and URL args are now encoded in `base64url` [Multibase](https://docs.ipfs.tech/concepts/glossary/#multibase). Encode/decode bytes using the `ipfs multibase --help` commands, or use the multiformats libraries ([js-multiformats](https://github.com/multiformats/js-multiformats#readme), [go-multibase](https://github.com/multiformats/go-multibase)).

Low level changes:
- `topic` passed as URL `arg` in requests to `/api/v0/pubsub/*` must be encoded in URL-safe multibase (`base64url`)
- `data`, `from`, `seqno` and `topicIDs` returned in JSON responses are now encoded in multibase
- Peer IDs returned in `from` now use the same default text representation from go-libp2p and peerid encoder/decoder from libp2p. This means the same text representation as in as in `swarm peers`, which makes it possible to compare them without decoding multibase.
-  `/api/v0/pubsub/pub`  no longer accepts `data` to be passed as URL, it has to be sent as `multipart/form-data`. This removes size limitations based on URL length, and enables regular HTTP clients to publish data to PubSub topics. For example, to publish `some-file` to topic named `test-topic` using vanilla `curl`, one would execute: `curl -X POST -v -F "stdin=@some-file" 'http://127.0.0.1:5001/api/v0/pubsub/pub?arg=$(echo -n test-topic | ipfs multibase encode -b base64url)'`
- `ipfs pubsub pub` on the command line no longer accepts variadic `data` arguments. Instead, it expects a single file input or stream of bytes from stdin. This ensures arbitrary stream of bytes can be published, removing limitation around messages that include `\n` or `\r\n`.

#### ⚙ New configuration flags

- [`Addresses.AppendAnnounce`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#addressesappendannounce)  is an array of multiaddrs, similar to  `Addresses.Announce`, except it does not override inferred swarm addresses, but appends custom ones to the list.
- Pubsub experiments can now  be enabled via config, removing the need for CLI flag to be passed every time daemon starts:
  - [`Pubsub.Enabled`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#pubsubenabled) enables the pubsub system.
  - [`Ipns.UsePubsub`](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#ipnsusepubsub) enables IPFS over pubsub experiment for publishing IPNS records in real time.

#### 🔐 Support for DAG-JOSE IPLD codec

JOSE is a [standard](https://datatracker.ietf.org/wg/jose/documents/) for signing and encrypting JSON objects. [DAG-JOSE](https://ipld.io/specs/codecs/dag-jose/spec/) is an IPLD codec based on JOSE and represented in CBOR. Upon encountering the `dag-jose` multicodec indicator, implementations can expect that the block contains dag-cbor encoded data which matches the IPLD schema from the [DAG-JOSE spec](https://ipld.io/specs/codecs/dag-jose/spec/).

This work was [contributed](https://github.com/ipfs/go-ipfs/pull/8569) by [Ceramic](https://ceramic.network/) and acts as a template for future IPFS improvements driven by the real world needs of the IPFS community.

### Changelog

- github.com/ipfs/go-ipfs:
  - docs: update changelog for v0.11.0
  - Release v0.11.0-rc2
  - fix(corehttp): adjust peer counting metrics (#8577) ([ipfs/go-ipfs#8577](https://github.com/ipfs/go-ipfs/pull/8577))
  - Release v0.11.0-rc1
  - feat: Swarm.EnableHolePunching flag (#8562) ([ipfs/go-ipfs#8562](https://github.com/ipfs/go-ipfs/pull/8562))
  - feat: enabling pubsub and ipns-pubsub via config flags (#8510) ([ipfs/go-ipfs#8510](https://github.com/ipfs/go-ipfs/pull/8510))
  - Integrate go-dag-jose plugin (#8569) ([ipfs/go-ipfs#8569](https://github.com/ipfs/go-ipfs/pull/8569))
  - feat: Addresses.AppendAnnounce (#8177) ([ipfs/go-ipfs#8177](https://github.com/ipfs/go-ipfs/pull/8177))
  - fix: multibase in pubsub http rpc (#8183) ([ipfs/go-ipfs#8183](https://github.com/ipfs/go-ipfs/pull/8183))
  - refactor: remove dir-index-html submodule   ([ipfs/go-ipfs#8555](https://github.com/ipfs/go-ipfs/pull/8555))
  - feat: hard deprecation of IPFS_REUSEPORT
  - feat: go-libp2p 0.16, UnixFS autosharding and go-datastore with contexts (#8563) ([ipfs/go-ipfs#8563](https://github.com/ipfs/go-ipfs/pull/8563))
  - chore: fix link in README.md (#8551) ([ipfs/go-ipfs#8551](https://github.com/ipfs/go-ipfs/pull/8551))
  - Updating release template based off some 0.10 learnings (#8491) ([ipfs/go-ipfs#8491](https://github.com/ipfs/go-ipfs/pull/8491))
  - fix: multiple subdomain gateways on same domain (#8556) ([ipfs/go-ipfs#8556](https://github.com/ipfs/go-ipfs/pull/8556))
  - Fix typos (#8548) ([ipfs/go-ipfs#8548](https://github.com/ipfs/go-ipfs/pull/8548))
  - Add support for multiple files to `ipfs files rm`.
  - add a docker-compose file (#8387) ([ipfs/go-ipfs#8387](https://github.com/ipfs/go-ipfs/pull/8387))
  - fix(sharness): use -Q option instead of pipe to tail cmd
  - Add Homebrew installation method. ([ipfs/go-ipfs#8545](https://github.com/ipfs/go-ipfs/pull/8545))
  - docs: fix ipfs files cp examples (#8533) ([ipfs/go-ipfs#8533](https://github.com/ipfs/go-ipfs/pull/8533))
  - fix(unixfs): check for errors before dereferencing the link ([ipfs/go-ipfs#8508](https://github.com/ipfs/go-ipfs/pull/8508))
  - chore: replace go-merkledag walk with go-ipld-prime traversal for dag export (#8506) ([ipfs/go-ipfs#8506](https://github.com/ipfs/go-ipfs/pull/8506))
  - test: add sharness test for reading ADLs with FUSE
  - fix: allow the levelds compression level to be unspecified
  -  ([ipfs/go-ipfs#8457](https://github.com/ipfs/go-ipfs/pull/8457))
  -  ([ipfs/go-ipfs#8482](https://github.com/ipfs/go-ipfs/pull/8482))
  - Added the missing heathcheck for the container (#8429) ([ipfs/go-ipfs#8429](https://github.com/ipfs/go-ipfs/pull/8429))
  - chore: update dir-index-html to v1.2.2
  - Update RELEASE_ISSUE_TEMPLATE.md
  - Update RELEASE_ISSUE_TEMPLATE.md
  - add more logging to flaky TestPeersTotal
  - Update RELEASE_ISSUE_TEMPLATE.md
  - Update RELEASE_ISSUE_TEMPLATE.md
  - Updating chocolatey to reference go-ipfs
  - chore: update changelog for v0.10.0
  - add testground plans to bitswap on CI
  - ci: move Docker image build to Actions (#8467) ([ipfs/go-ipfs#8467](https://github.com/ipfs/go-ipfs/pull/8467))
  - fix(cli): object add-link: do not allow blocks over BS limit (#8414) ([ipfs/go-ipfs#8414](https://github.com/ipfs/go-ipfs/pull/8414))
  - fuse: load unixfs adls as their dagpb substrates
  - enable the legacy mDNS implementation
  - change ipfs dag get flag name from format to output-codec ([ipfs/go-ipfs#8440](https://github.com/ipfs/go-ipfs/pull/8440))
  - change names of ipfs dag put flags to make changes clearer ([ipfs/go-ipfs#8439](https://github.com/ipfs/go-ipfs/pull/8439))
  - test: check behavior of loading UnixFS sharded directories with missing shards
  -  ([ipfs/go-ipfs#8432](https://github.com/ipfs/go-ipfs/pull/8432))
  - feat: dag import --stats (#8237) ([ipfs/go-ipfs#8237](https://github.com/ipfs/go-ipfs/pull/8237))
  - feat: ipfs-webui v2.13.0 (#8430) ([ipfs/go-ipfs#8430](https://github.com/ipfs/go-ipfs/pull/8430))
  - feat(cli): add daemon option --agent-version-suffix (#8419) ([ipfs/go-ipfs#8419](https://github.com/ipfs/go-ipfs/pull/8419))
  - feat: multibase transcode command (#8403) ([ipfs/go-ipfs#8403](https://github.com/ipfs/go-ipfs/pull/8403))
  - fix: take the lock while listing peers
  - feature: 'ipfs swarm peering' command (#8147) ([ipfs/go-ipfs#8147](https://github.com/ipfs/go-ipfs/pull/8147))
  - chore: update IPFS Desktop testing steps (#8393) ([ipfs/go-ipfs#8393](https://github.com/ipfs/go-ipfs/pull/8393))
  - add more buttons; remove some sections covered in the docs; general cleanup ([ipfs/go-ipfs#8274](https://github.com/ipfs/go-ipfs/pull/8274))
  - Cosmetic fixups in examples (#8325) ([ipfs/go-ipfs#8325](https://github.com/ipfs/go-ipfs/pull/8325))
  - perf: use performance-enhancing FUSE mount options
  - ci: publish Docker images for bifrost-* branches
  - chore: add comments to peerlog plugin about being unsupported
  - test: add unit tests for peerlog config parsing
  - ci: preload peerlog plugin, disable by default
  - fix(mkreleaselog): specify the parent commit when diffing
  - update version to v0.11.0-dev
- github.com/ipfs/go-bitswap (v0.4.0 -> v0.5.1):
  - Version 0.5.1
  - Change incorrect function name in README (#541) ([ipfs/go-bitswap#541](https://github.com/ipfs/go-bitswap/pull/541))
  - Version 0.5.0 (#540) ([ipfs/go-bitswap#540](https://github.com/ipfs/go-bitswap/pull/540))
  - feat: plumb through contexts (#539) ([ipfs/go-bitswap#539](https://github.com/ipfs/go-bitswap/pull/539))
  - sync: update CI config files (#538) ([ipfs/go-bitswap#538](https://github.com/ipfs/go-bitswap/pull/538))
  - fix: optimize handling for peers with lots of tasks ([ipfs/go-bitswap#537](https://github.com/ipfs/go-bitswap/pull/537))
  - Enable custom task prioritization logic ([ipfs/go-bitswap#535](https://github.com/ipfs/go-bitswap/pull/535))
  - feat: cache the materialized wantlist ([ipfs/go-bitswap#530](https://github.com/ipfs/go-bitswap/pull/530))
  - fix: reduce receive contention ([ipfs/go-bitswap#536](https://github.com/ipfs/go-bitswap/pull/536))
  - Fix ProviderQueryManager test timings ([ipfs/go-bitswap#534](https://github.com/ipfs/go-bitswap/pull/534))
  - fix: rename wiretap to tracer ([ipfs/go-bitswap#531](https://github.com/ipfs/go-bitswap/pull/531))
  - fix: fix race on "responsive" check ([ipfs/go-bitswap#528](https://github.com/ipfs/go-bitswap/pull/528))
  - fix: reduce log verbosity
- github.com/ipfs/go-blockservice (v0.1.7 -> v0.2.1):
  - Version 0.2.1
  - Version 0.2.0 (#87) ([ipfs/go-blockservice#87](https://github.com/ipfs/go-blockservice/pull/87))
  - feat: add context to interfaces (#86) ([ipfs/go-blockservice#86](https://github.com/ipfs/go-blockservice/pull/86))
  - sync: update CI config files (#85) ([ipfs/go-blockservice#85](https://github.com/ipfs/go-blockservice/pull/85))
  - chore: update log ([ipfs/go-blockservice#84](https://github.com/ipfs/go-blockservice/pull/84))
- github.com/ipfs/go-cid (v0.0.7 -> v0.1.0):
  - amend the CidFromReader slice extension math
  - implement CidFromReader
  - chore: fixups from running go vet, go fmt and staticcheck ([ipfs/go-cid#122](https://github.com/ipfs/go-cid/pull/122))
  - s/characters/bytes
  - Fix inaccurate comment for uvarint
  - coverage: more tests for cid
  - coverage: more tests for varint
  - coverage: more tests for builder
  - fix: make tests run with Go 1.15
  - Add the dagjose multiformat
- github.com/ipfs/go-datastore (v0.4.6 -> v0.5.1):
  - Release v0.5.1
  - chore: add lots of interface assertions
  - fix: make NullDatastore satisfy the Batching interface again
  - Update version.json (#183) ([ipfs/go-datastore#183](https://github.com/ipfs/go-datastore/pull/183))
  - feat: add context to interfaces (#181) ([ipfs/go-datastore#181](https://github.com/ipfs/go-datastore/pull/181))
  - sync: update CI config files ([ipfs/go-datastore#182](https://github.com/ipfs/go-datastore/pull/182))
- github.com/ipfs/go-ds-badger (v0.2.7 -> v0.3.0):
  - feat: plumb through contexts (#119) ([ipfs/go-ds-badger#119](https://github.com/ipfs/go-ds-badger/pull/119))
- github.com/ipfs/go-ds-flatfs (v0.4.5 -> v0.5.1):
  - Update version.json
  - fix: add context to DiskUsage()
  - Version 0.5.0 (#99) ([ipfs/go-ds-flatfs#99](https://github.com/ipfs/go-ds-flatfs/pull/99))
  - feat: add contexts on datastore methods (#98) ([ipfs/go-ds-flatfs#98](https://github.com/ipfs/go-ds-flatfs/pull/98))
  - sync: update CI config files (#97) ([ipfs/go-ds-flatfs#97](https://github.com/ipfs/go-ds-flatfs/pull/97))
  - sync: update CI config files ([ipfs/go-ds-flatfs#96](https://github.com/ipfs/go-ds-flatfs/pull/96))
  - fix staticcheck ([ipfs/go-ds-flatfs#92](https://github.com/ipfs/go-ds-flatfs/pull/92))
  - fix typo in readme.go ([ipfs/go-ds-flatfs#89](https://github.com/ipfs/go-ds-flatfs/pull/89))
- github.com/ipfs/go-ds-leveldb (v0.4.2 -> v0.5.0):
  - Version 0.5.0 (#58) ([ipfs/go-ds-leveldb#58](https://github.com/ipfs/go-ds-leveldb/pull/58))
  - feat: plumb through contexts (#57) ([ipfs/go-ds-leveldb#57](https://github.com/ipfs/go-ds-leveldb/pull/57))
  - sync: update CI config files (#56) ([ipfs/go-ds-leveldb#56](https://github.com/ipfs/go-ds-leveldb/pull/56))
  - fix closing of datastore in tests ([ipfs/go-ds-leveldb#52](https://github.com/ipfs/go-ds-leveldb/pull/52))
  - fix staticcheck ([ipfs/go-ds-leveldb#49](https://github.com/ipfs/go-ds-leveldb/pull/49))
  - fix typo in function documentation ([ipfs/go-ds-leveldb#46](https://github.com/ipfs/go-ds-leveldb/pull/46))
- github.com/ipfs/go-ds-measure (v0.1.0 -> v0.2.0):
  - Version 0.2.0 (#39) ([ipfs/go-ds-measure#39](https://github.com/ipfs/go-ds-measure/pull/39))
  - feat: add contexts on datastore methods (#38) ([ipfs/go-ds-measure#38](https://github.com/ipfs/go-ds-measure/pull/38))
  - sync: update CI config files (#37) ([ipfs/go-ds-measure#37](https://github.com/ipfs/go-ds-measure/pull/37))
- github.com/ipfs/go-fetcher (v1.5.0 -> v1.6.1):
  - Version 1.6.1
  - Version 1.6.0 (#29) ([ipfs/go-fetcher#29](https://github.com/ipfs/go-fetcher/pull/29))
  - feat: plumb through context changes (#28) ([ipfs/go-fetcher#28](https://github.com/ipfs/go-fetcher/pull/28))
  - sync: update CI config files (#27) ([ipfs/go-fetcher#27](https://github.com/ipfs/go-fetcher/pull/27))
  - add a fetcher constructor for the case where we already have a session ([ipfs/go-fetcher#26](https://github.com/ipfs/go-fetcher/pull/26))
- github.com/ipfs/go-filestore (v0.0.3 -> v0.1.0):
  - feat: plumb through context changes (#56) ([ipfs/go-filestore#56](https://github.com/ipfs/go-filestore/pull/56))
- github.com/ipfs/go-graphsync (v0.8.0 -> v0.11.0):
  - docs(CHANGELOG): update for v0.11.0 release
  - Merge branch 'release/v0.10.6'
  - update to context datastores (#275) ([ipfs/go-graphsync#275](https://github.com/ipfs/go-graphsync/pull/275))
  - feat!(requestmanager): remove request allocation backpressure (#272) ([ipfs/go-graphsync#272](https://github.com/ipfs/go-graphsync/pull/272))
  - message/pb: stop using gogo/protobuf (#277) ([ipfs/go-graphsync#277](https://github.com/ipfs/go-graphsync/pull/277))
  - mark all test helper funcs via t.Helper (#276) ([ipfs/go-graphsync#276](https://github.com/ipfs/go-graphsync/pull/276))
  - chore(queryexecutor): remove unused RunTraversal
  - chore(responsemanager): remove unused workSignal
  - chore(queryexecutor): fix tests for runtraversal refactor + clean up
  - feat(queryexecutor): merge RunTraversal into QueryExecutor
  - feat(responsemanager): QueryExecutor to separate module - use TaskQueue, add tests
  - Merge branch 'release/v0.10.5'
  - fix(responseassembler): don't hold block data reference in passed on subscribed block link (#268) ([ipfs/go-graphsync#268](https://github.com/ipfs/go-graphsync/pull/268))
  - sync: update CI config files (#266) ([ipfs/go-graphsync#266](https://github.com/ipfs/go-graphsync/pull/266))
  - Check IPLD context cancellation error type instead of string comparison
  - Use `context.CancelFunc` instead of `func()` (#257) ([ipfs/go-graphsync#257](https://github.com/ipfs/go-graphsync/pull/257))
  - fix: bail properly when budget exceeded
  - feat(requestmanager): report inProgressRequestCount on OutgoingRequests event
  - fix(requestmanager): remove failing racy test select block
  - feat(requestmanager): add OutgoingRequeustProcessingListener
  - Merge branch 'release/v0.10.4'
  - fix(allocator): prevent buffer overflow (#248) ([ipfs/go-graphsync#248](https://github.com/ipfs/go-graphsync/pull/248))
  - Merge branch 'release/v0.10.3'
  - Configure message parameters (#247) ([ipfs/go-graphsync#247](https://github.com/ipfs/go-graphsync/pull/247))
  - Stats! (#246) ([ipfs/go-graphsync#246](https://github.com/ipfs/go-graphsync/pull/246))
  - Limit simultaneous incoming requests on a per peer basis (#245) ([ipfs/go-graphsync#245](https://github.com/ipfs/go-graphsync/pull/245))
  - sync: update CI config files (#191) ([ipfs/go-graphsync#191](https://github.com/ipfs/go-graphsync/pull/191))
  - Merge branch 'release/v0.10.2'
  - test(responsemanager): fix flakiness TestCancellationViaCommand (#243) ([ipfs/go-graphsync#243](https://github.com/ipfs/go-graphsync/pull/243))
  - Fix deadlock on notifications (#242) ([ipfs/go-graphsync#242](https://github.com/ipfs/go-graphsync/pull/242))
  - Merge branch 'release/v0.10.1'
  - Free memory on request finish (#240) ([ipfs/go-graphsync#240](https://github.com/ipfs/go-graphsync/pull/240))
  - release: v1.10.0 ([ipfs/go-graphsync#238](https://github.com/ipfs/go-graphsync/pull/238))
  - Add support for IPLD prime's budgets feature in selectors (#235) ([ipfs/go-graphsync#235](https://github.com/ipfs/go-graphsync/pull/235))
  - feat(graphsync): add an index for blocks in the on new block hook (#234) ([ipfs/go-graphsync#234](https://github.com/ipfs/go-graphsync/pull/234))
  - Do not send first blocks extension (#230) ([ipfs/go-graphsync#230](https://github.com/ipfs/go-graphsync/pull/230))
  - Protect Libp2p Connections (#229) ([ipfs/go-graphsync#229](https://github.com/ipfs/go-graphsync/pull/229))
  - test(responsemanager): remove check (#228) ([ipfs/go-graphsync#228](https://github.com/ipfs/go-graphsync/pull/228))
  - feat(graphsync): give missing blocks a named error (#227) ([ipfs/go-graphsync#227](https://github.com/ipfs/go-graphsync/pull/227))
  - Add request limits (#224) ([ipfs/go-graphsync#224](https://github.com/ipfs/go-graphsync/pull/224))
  - Tech Debt Cleanup and Docs Update (#219) ([ipfs/go-graphsync#219](https://github.com/ipfs/go-graphsync/pull/219))
  - Release/v0.9.3 ([ipfs/go-graphsync#218](https://github.com/ipfs/go-graphsync/pull/218))
  - 0.9.2 release ([ipfs/go-graphsync#217](https://github.com/ipfs/go-graphsync/pull/217))
  - fix(requestmanager): remove main thread block on allocation (#216) ([ipfs/go-graphsync#216](https://github.com/ipfs/go-graphsync/pull/216))
  - feat(allocator): add debug logging (#213) ([ipfs/go-graphsync#213](https://github.com/ipfs/go-graphsync/pull/213))
  - fix: spurious warn log (#210) ([ipfs/go-graphsync#210](https://github.com/ipfs/go-graphsync/pull/210))
  - docs(CHANGELOG): update for v0.9.1 release (#212) ([ipfs/go-graphsync#212](https://github.com/ipfs/go-graphsync/pull/212))
  - fix(message): fix dropping of response extensions (#211) ([ipfs/go-graphsync#211](https://github.com/ipfs/go-graphsync/pull/211))
  - docs(CHANGELOG): update change log ([ipfs/go-graphsync#208](https://github.com/ipfs/go-graphsync/pull/208))
  - docs(README): add notice about branch rename
  - fix(graphsync): make sure linkcontext is passed (#207) ([ipfs/go-graphsync#207](https://github.com/ipfs/go-graphsync/pull/207))
  - Merge final v0.6.x commit history, and 0.8.0 changelog (#205) ([ipfs/go-graphsync#205](https://github.com/ipfs/go-graphsync/pull/205))
  - Fix broken link to IPLD selector documentation (#189) ([ipfs/go-graphsync#189](https://github.com/ipfs/go-graphsync/pull/189))
  - fix: check errors before defering a close (#200) ([ipfs/go-graphsync#200](https://github.com/ipfs/go-graphsync/pull/200))
  - chore: fix checks (#197) ([ipfs/go-graphsync#197](https://github.com/ipfs/go-graphsync/pull/197))
  - Merge the v0.6.x commit history (#190) ([ipfs/go-graphsync#190](https://github.com/ipfs/go-graphsync/pull/190))
  - Ready for universal CI (#187) ([ipfs/go-graphsync#187](https://github.com/ipfs/go-graphsync/pull/187))
  - fix(requestmanager): pass through linksystem (#166) ([ipfs/go-graphsync#166](https://github.com/ipfs/go-graphsync/pull/166))
  - fix missing word in section title (#179) ([ipfs/go-graphsync#179](https://github.com/ipfs/go-graphsync/pull/179))
- github.com/ipfs/go-ipfs-blockstore (v0.1.6 -> v0.2.1):
  - fix: revert back to go-ipfs-ds-help@v0.1.1 (#92) ([ipfs/go-ipfs-blockstore#92](https://github.com/ipfs/go-ipfs-blockstore/pull/92))
  - feat: add context to interfaces & plumb through datastore contexts (#89) ([ipfs/go-ipfs-blockstore#89](https://github.com/ipfs/go-ipfs-blockstore/pull/89))
- github.com/ipfs/go-ipfs-config (v0.16.0 -> v0.18.0):
  - Release v0.18.0 (#159) ([ipfs/go-ipfs-config#159](https://github.com/ipfs/go-ipfs-config/pull/159))
  - feat: add Addresses.AppendAnnounce (#135) ([ipfs/go-ipfs-config#135](https://github.com/ipfs/go-ipfs-config/pull/135))
  - feat: omitempty Swarm.EnableRelayHop for circuit v1 migration (#157) ([ipfs/go-ipfs-config#157](https://github.com/ipfs/go-ipfs-config/pull/157))
  - chore: omitempty Experimental.ShardingEnabled (#158) ([ipfs/go-ipfs-config#158](https://github.com/ipfs/go-ipfs-config/pull/158))
  - chore: update comment to match struct
  - Release v0.17.0 (#156) ([ipfs/go-ipfs-config#156](https://github.com/ipfs/go-ipfs-config/pull/156))
  - feat: add a flag to enable the hole punching service (#155) ([ipfs/go-ipfs-config#155](https://github.com/ipfs/go-ipfs-config/pull/155))
  - improve AutoRelay configuration, add config option for static relays ([ipfs/go-ipfs-config#154](https://github.com/ipfs/go-ipfs-config/pull/154))
  - feat: Swarm.RelayService (circuit v2) (#146) ([ipfs/go-ipfs-config#146](https://github.com/ipfs/go-ipfs-config/pull/146))
  - fix: String method on the OptionalString (#153) ([ipfs/go-ipfs-config#153](https://github.com/ipfs/go-ipfs-config/pull/153))
  - sync: update CI config files (#152) ([ipfs/go-ipfs-config#152](https://github.com/ipfs/go-ipfs-config/pull/152))
  - feat: OptionalString type and UnixFSShardingSizeThreshold (#149) ([ipfs/go-ipfs-config#149](https://github.com/ipfs/go-ipfs-config/pull/149))
  - feat: pubsub and ipns pubsub flags (#145) ([ipfs/go-ipfs-config#145](https://github.com/ipfs/go-ipfs-config/pull/145))
  - feat: add an OptionalDuration type (#148) ([ipfs/go-ipfs-config#148](https://github.com/ipfs/go-ipfs-config/pull/148))
- github.com/ipfs/go-ipfs-exchange-interface (v0.0.1 -> v0.1.0):
  - Update version.json (#20) ([ipfs/go-ipfs-exchange-interface#20](https://github.com/ipfs/go-ipfs-exchange-interface/pull/20))
  - sync: update CI config files (#19) ([ipfs/go-ipfs-exchange-interface#19](https://github.com/ipfs/go-ipfs-exchange-interface/pull/19))
  - feat: add context to interface (#18) ([ipfs/go-ipfs-exchange-interface#18](https://github.com/ipfs/go-ipfs-exchange-interface/pull/18))
  - doc: add a lead maintainer
- github.com/ipfs/go-ipfs-exchange-offline (v0.0.1 -> v0.1.1):
  - Version 0.1.1
  - Version 0.1.0 (#43) ([ipfs/go-ipfs-exchange-offline#43](https://github.com/ipfs/go-ipfs-exchange-offline/pull/43))
  - feat: plumb through contexts (#42) ([ipfs/go-ipfs-exchange-offline#42](https://github.com/ipfs/go-ipfs-exchange-offline/pull/42))
  - sync: update CI config files (#41) ([ipfs/go-ipfs-exchange-offline#41](https://github.com/ipfs/go-ipfs-exchange-offline/pull/41))
  - fix staticcheck ([ipfs/go-ipfs-exchange-offline#35](https://github.com/ipfs/go-ipfs-exchange-offline/pull/35))
  - chore(gx): remove gx
- github.com/ipfs/go-ipfs-files (v0.0.8 -> v0.0.9):
  - sync: update CI config files ([ipfs/go-ipfs-files#40](https://github.com/ipfs/go-ipfs-files/pull/40))
  - fix: manually parse the content disposition to preserve directories ([ipfs/go-ipfs-files#42](https://github.com/ipfs/go-ipfs-files/pull/42))
  - fix: round timestamps down by truncating them to seconds ([ipfs/go-ipfs-files#41](https://github.com/ipfs/go-ipfs-files/pull/41))
  - sync: update CI config files ([ipfs/go-ipfs-files#34](https://github.com/ipfs/go-ipfs-files/pull/34))
  - Fix test failure on Windows caused by nil `sys` in mock `FileInfo` ([ipfs/go-ipfs-files#39](https://github.com/ipfs/go-ipfs-files/pull/39))
  - fix staticcheck ([ipfs/go-ipfs-files#35](https://github.com/ipfs/go-ipfs-files/pull/35))
  - fix linters ([ipfs/go-ipfs-files#33](https://github.com/ipfs/go-ipfs-files/pull/33))
- github.com/ipfs/go-ipfs-pinner (v0.1.2 -> v0.2.1):
  - feat: plumb through context changes (#18) ([ipfs/go-ipfs-pinner#18](https://github.com/ipfs/go-ipfs-pinner/pull/18))
- github.com/ipfs/go-ipfs-provider (v0.6.1 -> v0.7.1):
  - Fix go vet and staticcheck ([ipfs/go-ipfs-provider#40](https://github.com/ipfs/go-ipfs-provider/pull/40))
  - feat: plumb through datastore contexts (#39) ([ipfs/go-ipfs-provider#39](https://github.com/ipfs/go-ipfs-provider/pull/39))
- github.com/ipfs/go-ipfs-routing (v0.1.0 -> v0.2.1):
  - Version 0.2.1
  - Bump version to 0.2.0 (#29) ([ipfs/go-ipfs-routing#29](https://github.com/ipfs/go-ipfs-routing/pull/29))
  - feat: plumb through context changes (#28) ([ipfs/go-ipfs-routing#28](https://github.com/ipfs/go-ipfs-routing/pull/28))
  - sync: update CI config files (#27) ([ipfs/go-ipfs-routing#27](https://github.com/ipfs/go-ipfs-routing/pull/27))
  - fix staticcheck ([ipfs/go-ipfs-routing#24](https://github.com/ipfs/go-ipfs-routing/pull/24))
- github.com/ipfs/go-merkledag (v0.4.0 -> v0.5.1):
  - Version 0.5.1
  - Version 0.5.0 (#79) ([ipfs/go-merkledag#79](https://github.com/ipfs/go-merkledag/pull/79))
  - feat: plumb through contexts (#78) ([ipfs/go-merkledag#78](https://github.com/ipfs/go-merkledag/pull/78))
  - sync: update CI config files (#77) ([ipfs/go-merkledag#77](https://github.com/ipfs/go-merkledag/pull/77))
  - expose session construction to other callers
  - fix RawNode incomplete stats
- github.com/ipfs/go-mfs (v0.1.2 -> v0.2.1):
  - Version 0.2.1
  - Version 0.2.0 (#96) ([ipfs/go-mfs#96](https://github.com/ipfs/go-mfs/pull/96))
  - support threshold based automatic sharding and unsharding of directories (#88) ([ipfs/go-mfs#88](https://github.com/ipfs/go-mfs/pull/88))
  - sync: update CI config files (#94) ([ipfs/go-mfs#94](https://github.com/ipfs/go-mfs/pull/94))
  - Fix lint errors ([ipfs/go-mfs#90](https://github.com/ipfs/go-mfs/pull/90))
  - remove Makefile ([ipfs/go-mfs#89](https://github.com/ipfs/go-mfs/pull/89))
- github.com/ipfs/go-namesys (v0.3.1 -> v0.4.0):
  - Release v0.4.0
  - feat: plumb through datastore contexts
  - sync: update CI config files (#23) ([ipfs/go-namesys#23](https://github.com/ipfs/go-namesys/pull/23))
- github.com/ipfs/go-path (v0.1.2 -> v0.2.1):
  - Version 0.2.1
  - Version 0.2.0 (#48) ([ipfs/go-path#48](https://github.com/ipfs/go-path/pull/48))
  - feat: plumb through context changes (#47) ([ipfs/go-path#47](https://github.com/ipfs/go-path/pull/47))
  - sync: update CI config files (#46) ([ipfs/go-path#46](https://github.com/ipfs/go-path/pull/46))
  - Revert "feat: plumb through context changes"
  - feat: plumb through context changes
- github.com/ipfs/go-peertaskqueue (v0.4.0 -> v0.7.0):
  - feat: optimize checking if a new task is "better" ([ipfs/go-peertaskqueue#19](https://github.com/ipfs/go-peertaskqueue/pull/19))
  - Adds customizable prioritization logic for peertracker and peertaskqueue ([ipfs/go-peertaskqueue#17](https://github.com/ipfs/go-peertaskqueue/pull/17))
  - When priority is equal, use FIFO ([ipfs/go-peertaskqueue#16](https://github.com/ipfs/go-peertaskqueue/pull/16))
- github.com/ipfs/go-unixfs (v0.2.5 -> v0.3.1):
  - Version 0.3.1
  - Version 0.3.0 (#114) ([ipfs/go-unixfs#114](https://github.com/ipfs/go-unixfs/pull/114))
  - feat: plumb through datastore context changes
  - Size-based unsharding (#94) ([ipfs/go-unixfs#94](https://github.com/ipfs/go-unixfs/pull/94))
  - sync: update CI config files (#112) ([ipfs/go-unixfs#112](https://github.com/ipfs/go-unixfs/pull/112))
  - chore(deps): move bitfield to ipfs org ([ipfs/go-unixfs#98](https://github.com/ipfs/go-unixfs/pull/98))
  - fix staticcheck ([ipfs/go-unixfs#95](https://github.com/ipfs/go-unixfs/pull/95))
  - fix(directory): initialize size when computing it ([ipfs/go-unixfs#93](https://github.com/ipfs/go-unixfs/pull/93))
  - fix: always return upgradeable instead of basic dir (#92) ([ipfs/go-unixfs#92](https://github.com/ipfs/go-unixfs/pull/92))
  - feat: switch to HAMT based on size (#91) ([ipfs/go-unixfs#91](https://github.com/ipfs/go-unixfs/pull/91))
  - go fmt
  - fix: add pointer receiver
  - add test
  - feat: add UpgradeableDirectory
- github.com/ipfs/interface-go-ipfs-core (v0.5.1 -> v0.5.2):
  - fix: check errors by string ([ipfs/interface-go-ipfs-core#76](https://github.com/ipfs/interface-go-ipfs-core/pull/76))
- github.com/ipfs/tar-utils (v0.0.1 -> v0.0.2):
  - Release v0.0.2 (#8) ([ipfs/tar-utils#8](https://github.com/ipfs/tar-utils/pull/8))
  - sync: update CI config files ([ipfs/tar-utils#7](https://github.com/ipfs/tar-utils/pull/7))
  - sync: update CI config files (#6) ([ipfs/tar-utils#6](https://github.com/ipfs/tar-utils/pull/6))
  - allow .. in file and directory names ([ipfs/tar-utils#5](https://github.com/ipfs/tar-utils/pull/5))
- github.com/ipld/go-car (v0.3.1 -> v0.3.2):
  - Expose selector traversal options for SelectiveCar ([ipld/go-car#251](https://github.com/ipld/go-car/pull/251))
  - Implement API to allow replacing root CIDs in a CARv1 or CARv2
  - blockstore: OpenReadWrite should not modify if it refuses to resume
  - clarify the relation between StoreIdentityCIDs and SetFullyIndexed
  - Implement options to handle `IDENTITY` CIDs gracefully
  - Combine API options for simplicity and logical coherence
  - Add test script for car verify (#236) ([ipld/go-car#236](https://github.com/ipld/go-car/pull/236))
  - cmd/car: add first testscript tests
  - integrate `car/` cli into `cmd/car` (#233) ([ipld/go-car#233](https://github.com/ipld/go-car/pull/233))
  - Add `car get-dag` command (#232) ([ipld/go-car#232](https://github.com/ipld/go-car/pull/232))
  - Separate CLI to separate module (#231) ([ipld/go-car#231](https://github.com/ipld/go-car/pull/231))
  - add `get block` to car cli (#230) ([ipld/go-car#230](https://github.com/ipld/go-car/pull/230))
  - use file size when loading from v1 car (#229) ([ipld/go-car#229](https://github.com/ipld/go-car/pull/229))
  - add interface describing iteration (#228) ([ipld/go-car#228](https://github.com/ipld/go-car/pull/228))
  - Add `list` and `filter` commands (#227) ([ipld/go-car#227](https://github.com/ipld/go-car/pull/227))
  - Add `car split` command (#226) ([ipld/go-car#226](https://github.com/ipld/go-car/pull/226))
  - Make `MultihashIndexSorted` the default index codec for CARv2
  - Add carve utility for updating the index of a car{v1,v2} file (#219) ([ipld/go-car#219](https://github.com/ipld/go-car/pull/219))
  - Ignore records with `IDENTITY` CID in `IndexSorted`
  - Fix index GetAll infinite loop if function always returns `true`
  - Expose the ability to iterate over records in `MultihasIndexSorted`
  - avoid another alloc per read byte
  - avoid allocating on every byte read
  - Implement new index type that also includes mutltihash code
  - Return `nil` as Index reader when reading indexless CARv2
  - Assert `OpenReader` from file does not panic after closure
  - Document performance caveats of `ExtractV1File` and address comments
  - Implement utility to extract CARv1 from a CARv2
  - v2/blockstore: add ReadWrite.Discard
  - update LICENSE files to point to the new gateway
  - re-add root LICENSE file
  - v2: stop using a symlink for LICENSE.md
  - Update the readme with link to examples
  - update package godocs and root level README for v2
  - blockstore: stop embedding ReadOnly in ReadWrite
  - Implement version agnostic streaming CAR block iterator
  - blockstore: use errors when API contracts are broken
  - add the first read-only benchmarks
  - Implement reader block iterator over CARv1 or CARv2
  - Propagate async `blockstore.AllKeysChan` errors via context
  - Add zero-length sections as EOF option to internal CARv1 reader
  - Improve error handing in tests
  - Allow `ReadOption`s to be set when getting or generating index
  - Use `ioutil.TempFile` to simplify file creation in index example
  - Avoid writing to files in testdata
  - blockstore: implement UseWholeCIDs
  - Merge wip-v2 into master (#178) ([ipld/go-car#178](https://github.com/ipld/go-car/pull/178))
- github.com/ipld/go-ipld-prime (v0.12.2 -> v0.14.2):
  - dagcbor: coerce undef to null. ([ipld/go-ipld-prime#308](https://github.com/ipld/go-ipld-prime/pull/308))
  - fluent: add toInterface (#304) ([ipld/go-ipld-prime#304](https://github.com/ipld/go-ipld-prime/pull/304))
  - traversal: s/Walk/WalkLocal/
  - traversal: add a primitive walk function.
  - Remove dependency to `go-wish`
  - mark v0.14.0
  -  ([ipld/go-ipld-prime#279](https://github.com/ipld/go-ipld-prime/pull/279))
  - Port `traversal` package tests to quicktest
  - Port `codec` package tests to quicktest
  - changelog: backfill.
  - Gracefully handle `TypedNode` with `nil` type of kind `Map`
  - Gracefully print typed nodes with `nil` type
  - Implement handling of `Link` and `[]byte` in `printer` (#294) ([ipld/go-ipld-prime#294](https://github.com/ipld/go-ipld-prime/pull/294))
  - changelog: backfill for the v0.12.x series.
  - readme: introduce a migration guide.
  - Port `fluent` package tests to quicktest
  - Port `datamodel` package tests to quicktest
  - Port `adl` package tests to quicktest
  - Port `node` package tests to quicktest
  - node/bindnode: support links in ProduceGoTypes
  - bump CI to Go 1.16 and 1.17
  - node/bindnode: support links in schema-type verification
  - node/bindnode: export ProduceGoTypes
  - all: fix "an" typos after the ipld->datamodel refactor
  - node/bindnode: fix test code after two PR merges
  - add LoadSchema APIs to the root package
  - storage: add 'Has' feature. ([ipld/go-ipld-prime#276](https://github.com/ipld/go-ipld-prime/pull/276))
  - node/bindnode: start verifying schema compatibility
  - linking: add LoadRaw and LoadPlusRaw functions to LinkSystem. ([ipld/go-ipld-prime#267](https://github.com/ipld/go-ipld-prime/pull/267))
  - node/bindnode: add support for lists behind kinded unions
  - node/bindnode: also run TestPrototype with just schemas
  - node/bindnode: polish a few TODO panics away
  - node/bindnode: add support for all scalars behind kinded unions
  - node/bindnode: get closer to passing the Links schema tests
  - start using Rod's schema tests from ipld/ipld
  - fully support parsing, encoding, and decoding the schema-schema
  - node/bindnode: add native support for cid.Cid
  - A more Featureful Approach to Storage APIs ([ipld/go-ipld-prime#265](https://github.com/ipld/go-ipld-prime/pull/265))
  - Add a cidlink.Memory storage option (#266) ([ipld/go-ipld-prime#266](https://github.com/ipld/go-ipld-prime/pull/266))
  - Improve docs for AssignNode; and datamodel.Copy function. ([ipld/go-ipld-prime#264](https://github.com/ipld/go-ipld-prime/pull/264))
  - schemadsl: assign the struct representation.
  - schema,tests,gen/go: more tests, gen union fixes. ([ipld/go-ipld-prime#257](https://github.com/ipld/go-ipld-prime/pull/257))
  - fix: deal with LinkRevisit->LinkVisitOnlyOnce change
  - traversal: the link-visit-only-once behavior should require opt-in, rather than defaulting to on.
  - chore: add LinkRevisit:false traversal test
  - traversal: track seen links, and revisit only if configured to do so.
  - fix: use datamodel.Node selectors
  - Revert encode round-trip to leave unencoded node test intact
  - Add more walk tests, including tests for use of SkipMe
  - Round-trip test nodes through custom codec to ensure stability
  - Don't abort block processing when encountering SkipMe
  - traversal: implement monotonically decrementing budgets. ([ipld/go-ipld-prime#260](https://github.com/ipld/go-ipld-prime/pull/260))
  - Use datamodel.Node for "Common" selector variants
  - schema/dmt: first pass at a parser ([ipld/go-ipld-prime#253](https://github.com/ipld/go-ipld-prime/pull/253))
  - drop codectools.
  - drop jst codec.  It lives in https://github.com/warpfork/go-jst/ now.
  - drop dagjson2.
  - fix(traversal): properly wrap errors
  - printer: empty maps and lists and structs should stay on one line.
  - schema: turn TypeName into an alias
  - schema/dmt: sync with schema-schema changes, finish Compile
  - schema: add ways to set and access the ImplicitValue for a struct field.
  - schema: accessor for TypeEnum.Representation.
  - schema: finish minimum viable support for describing enum types.
- github.com/libp2p/go-conn-security-multistream (v0.2.1 -> v0.3.0):
  - use the new SecureTransport and SecureMuxer interfaces (#36) ([libp2p/go-conn-security-multistream#36](https://github.com/libp2p/go-conn-security-multistream/pull/36))
  - fix go vet and staticcheck ([libp2p/go-conn-security-multistream#33](https://github.com/libp2p/go-conn-security-multistream/pull/33))
- github.com/libp2p/go-libp2p (v0.15.0 -> v0.16.0):
  - release v0.16.0 ([libp2p/go-libp2p#1246](https://github.com/libp2p/go-libp2p/pull/1246))
  - allow the ping protocol on transient connections ([libp2p/go-libp2p#1244](https://github.com/libp2p/go-libp2p/pull/1244))
  - make the Type field required in the HolePunch protobuf ([libp2p/go-libp2p#1241](https://github.com/libp2p/go-libp2p/pull/1241))
  - reject hole punching attempts when we don't have any public addresses ([libp2p/go-libp2p#1214](https://github.com/libp2p/go-libp2p/pull/1214))
  - refactor the AutoRelay code ([libp2p/go-libp2p#1240](https://github.com/libp2p/go-libp2p/pull/1240))
  - remove dead API link in README ([libp2p/go-libp2p#1233](https://github.com/libp2p/go-libp2p/pull/1233))
  - pass static relays to EnableAutoRelay, deprecate libp2p.StaticRelays and libp2p.DefaultStaticRelays ([libp2p/go-libp2p#1239](https://github.com/libp2p/go-libp2p/pull/1239))
  - feat: plumb through peerstore context changes (#1237) ([libp2p/go-libp2p#1237](https://github.com/libp2p/go-libp2p/pull/1237))
  - emit the EvtPeerConnectednessChanged event ([libp2p/go-libp2p#1230](https://github.com/libp2p/go-libp2p/pull/1230))
  - update go-libp2p-swarm to v0.7.0 ([libp2p/go-libp2p#1226](https://github.com/libp2p/go-libp2p/pull/1226))
  - sync: update CI config files (#1225) ([libp2p/go-libp2p#1225](https://github.com/libp2p/go-libp2p/pull/1225))
  - simplify circuitv2 package structure ([libp2p/go-libp2p#1224](https://github.com/libp2p/go-libp2p/pull/1224))
  - use a random string for the mDNS peer-name ([libp2p/go-libp2p#1222](https://github.com/libp2p/go-libp2p/pull/1222))
  - remove {Un}RegisterNotifee functions from mDNS service ([libp2p/go-libp2p#1220](https://github.com/libp2p/go-libp2p/pull/1220))
  - fix structured logging in holepunch coordination ([libp2p/go-libp2p#1213](https://github.com/libp2p/go-libp2p/pull/1213))
  - fix flaky TestStBackpressureStreamWrite test ([libp2p/go-libp2p#1212](https://github.com/libp2p/go-libp2p/pull/1212))
  - properly close hosts in mDNS tests ([libp2p/go-libp2p#1216](https://github.com/libp2p/go-libp2p/pull/1216))
  - close the ObserverAddrManager when the ID service is closed ([libp2p/go-libp2p#1218](https://github.com/libp2p/go-libp2p/pull/1218))
  - make it possible to pass options to a transport constructor ([libp2p/go-libp2p#1205](https://github.com/libp2p/go-libp2p/pull/1205))
  - remove goprocess from the NATManager ([libp2p/go-libp2p#1193](https://github.com/libp2p/go-libp2p/pull/1193))
  - add an option to start the relay v2 ([libp2p/go-libp2p#1197](https://github.com/libp2p/go-libp2p/pull/1197))
  - fix flaky TestFastDisconnect identify test ([libp2p/go-libp2p#1200](https://github.com/libp2p/go-libp2p/pull/1200))
  - chore: update go-tcp-transport to v0.3.0 ([libp2p/go-libp2p#1203](https://github.com/libp2p/go-libp2p/pull/1203))
  - fix: skip variadic params in constructors ([libp2p/go-libp2p#1204](https://github.com/libp2p/go-libp2p/pull/1204))
  - fix flaky BasicHost tests ([libp2p/go-libp2p#1202](https://github.com/libp2p/go-libp2p/pull/1202))
  - remove dependency on github.com/ipfs/go-detect-race ([libp2p/go-libp2p#1201](https://github.com/libp2p/go-libp2p/pull/1201))
  - fix flaky TestEndToEndSimConnect holepunching test ([libp2p/go-libp2p#1191](https://github.com/libp2p/go-libp2p/pull/1191))
  - autorelay support for circuitv2 relays (#1198) ([libp2p/go-libp2p#1198](https://github.com/libp2p/go-libp2p/pull/1198))
  - reject circuitv2 reservations with nonsensical expiration times ([libp2p/go-libp2p#1199](https://github.com/libp2p/go-libp2p/pull/1199))
  - Tag relay hops in relay implementations ([libp2p/go-libp2p#1188](https://github.com/libp2p/go-libp2p/pull/1188))
  - Add standalone implementation of v1 Relay (#1186) ([libp2p/go-libp2p#1186](https://github.com/libp2p/go-libp2p/pull/1186))
  - remove the context from the libp2p and the Host constructor ([libp2p/go-libp2p#1190](https://github.com/libp2p/go-libp2p/pull/1190))
  - don't use a context to shut down the circuitv2 ([libp2p/go-libp2p#1185](https://github.com/libp2p/go-libp2p/pull/1185))
  - fix: remove v1 go-log dep ([libp2p/go-libp2p#1189](https://github.com/libp2p/go-libp2p/pull/1189))
  - don't use the context to shut down the relay ([libp2p/go-libp2p#1184](https://github.com/libp2p/go-libp2p/pull/1184))
  - Use circuitv2 code (#1183) ([libp2p/go-libp2p#1183](https://github.com/libp2p/go-libp2p/pull/1183))
  - clean up badges in README ([libp2p/go-libp2p#1179](https://github.com/libp2p/go-libp2p/pull/1179))
  - remove recommendation about Go module proxy from README ([libp2p/go-libp2p#1180](https://github.com/libp2p/go-libp2p/pull/1180))
  - merge branch 'hole-punching'
  - don't use a context for closing the ObservedAddrManager ([libp2p/go-libp2p#1175](https://github.com/libp2p/go-libp2p/pull/1175))
  - move the circuit v2 code here ([libp2p/go-libp2p#1174](https://github.com/libp2p/go-libp2p/pull/1174))
  - make QUIC a default transport ([libp2p/go-libp2p#1128](https://github.com/libp2p/go-libp2p/pull/1128))
  - stop using jbenet/go-cienv ([libp2p/go-libp2p#1176](https://github.com/libp2p/go-libp2p/pull/1176))
  - fix flaky TestObsAddrSet test ([libp2p/go-libp2p#1172](https://github.com/libp2p/go-libp2p/pull/1172))
  - clean up messy defer logic in IDService.sendIdentifyResp ([libp2p/go-libp2p#1169](https://github.com/libp2p/go-libp2p/pull/1169))
  - remove secio from README, add noise ([libp2p/go-libp2p#1165](https://github.com/libp2p/go-libp2p/pull/1165))
- github.com/libp2p/go-libp2p-asn-util (v0.0.0-20200825225859-85005c6cf052 -> v0.1.0):
  - Update from upstream and make regeneration easier (#17) ([libp2p/go-libp2p-asn-util#17](https://github.com/libp2p/go-libp2p-asn-util/pull/17))
  - add license file so it can be found by go-licenses ([libp2p/go-libp2p-asn-util#10](https://github.com/libp2p/go-libp2p-asn-util/pull/10))
  - refactor: rename ASN table files ([libp2p/go-libp2p-asn-util#9](https://github.com/libp2p/go-libp2p-asn-util/pull/9))
  - Library for IP -> ASN mapping ([libp2p/go-libp2p-asn-util#1](https://github.com/libp2p/go-libp2p-asn-util/pull/1))
- github.com/libp2p/go-libp2p-autonat (v0.4.2 -> v0.6.0):
  - Version 0.6.0 (#112) ([libp2p/go-libp2p-autonat#112](https://github.com/libp2p/go-libp2p-autonat/pull/112))
  - feat: plumb through contexts from peerstore (#111) ([libp2p/go-libp2p-autonat#111](https://github.com/libp2p/go-libp2p-autonat/pull/111))
  - sync: update CI config files (#110) ([libp2p/go-libp2p-autonat#110](https://github.com/libp2p/go-libp2p-autonat/pull/110))
  - remove context from constructor, implement a proper Close method ([libp2p/go-libp2p-autonat#109](https://github.com/libp2p/go-libp2p-autonat/pull/109))
  - fix stream deadlines ([libp2p/go-libp2p-autonat#107](https://github.com/libp2p/go-libp2p-autonat/pull/107))
  - disable failing integration test ([libp2p/go-libp2p-autonat#108](https://github.com/libp2p/go-libp2p-autonat/pull/108))
  - fix staticcheck ([libp2p/go-libp2p-autonat#103](https://github.com/libp2p/go-libp2p-autonat/pull/103))
- github.com/libp2p/go-libp2p-core (v0.9.0 -> v0.11.0):
  - release v0.11.0 (#217) ([libp2p/go-libp2p-core#217](https://github.com/libp2p/go-libp2p-core/pull/217))
  - remove the ConnHandler (#214) ([libp2p/go-libp2p-core#214](https://github.com/libp2p/go-libp2p-core/pull/214))
  - sync: update CI config files (#216) ([libp2p/go-libp2p-core#216](https://github.com/libp2p/go-libp2p-core/pull/216))
  - remove the Process from the Network interface (#212) ([libp2p/go-libp2p-core#212](https://github.com/libp2p/go-libp2p-core/pull/212))
  - pass the peer ID to SecureInbound in the SecureTransport and SecureMuxer (#211) ([libp2p/go-libp2p-core#211](https://github.com/libp2p/go-libp2p-core/pull/211))
  - save the role (client, server) in the simultaneous connect context (#210) ([libp2p/go-libp2p-core#210](https://github.com/libp2p/go-libp2p-core/pull/210))
  - sync: update CI config files (#209) ([libp2p/go-libp2p-core#209](https://github.com/libp2p/go-libp2p-core/pull/209))
- github.com/libp2p/go-libp2p-discovery (v0.5.1 -> v0.6.0):
  - feat: plumb peerstore contexts changes through (#75) ([libp2p/go-libp2p-discovery#75](https://github.com/libp2p/go-libp2p-discovery/pull/75))
  - remove deprecated types ([libp2p/go-libp2p-discovery#73](https://github.com/libp2p/go-libp2p-discovery/pull/73))
- github.com/libp2p/go-libp2p-kad-dht (v0.13.1 -> v0.15.0):
  - Bump version to 0.15.0 (#755) ([libp2p/go-libp2p-kad-dht#755](https://github.com/libp2p/go-libp2p-kad-dht/pull/755))
  - sync: update CI config files (#754) ([libp2p/go-libp2p-kad-dht#754](https://github.com/libp2p/go-libp2p-kad-dht/pull/754))
  - feat: plumb through datastore contexts (#753) ([libp2p/go-libp2p-kad-dht#753](https://github.com/libp2p/go-libp2p-kad-dht/pull/753))
  - custom ProviderManager that brokers AddrInfos (#751) ([libp2p/go-libp2p-kad-dht#751](https://github.com/libp2p/go-libp2p-kad-dht/pull/751))
  - feat: make compatible with go-libp2p 0.15 ([libp2p/go-libp2p-kad-dht#747](https://github.com/libp2p/go-libp2p-kad-dht/pull/747))
  - sync: update CI config files ([libp2p/go-libp2p-kad-dht#743](https://github.com/libp2p/go-libp2p-kad-dht/pull/743))
  - Disallow GetPublicKey when DisableValues is passed ([libp2p/go-libp2p-kad-dht#604](https://github.com/libp2p/go-libp2p-kad-dht/pull/604))
- github.com/libp2p/go-libp2p-nat (v0.0.6 -> v0.1.0):
  - remove Codecov config file ([libp2p/go-libp2p-nat#39](https://github.com/libp2p/go-libp2p-nat/pull/39))
  - stop using goprocess for shutdown ([libp2p/go-libp2p-nat#38](https://github.com/libp2p/go-libp2p-nat/pull/38))
  - chore: update go-log ([libp2p/go-libp2p-nat#37](https://github.com/libp2p/go-libp2p-nat/pull/37))
  - remove unused field permanent from mapping ([libp2p/go-libp2p-nat#33](https://github.com/libp2p/go-libp2p-nat/pull/33))
- github.com/libp2p/go-libp2p-noise (v0.2.2 -> v0.3.0):
  - add the peer ID to SecureInbound ([libp2p/go-libp2p-noise#104](https://github.com/libp2p/go-libp2p-noise/pull/104))
  - update go-libp2p-core, remove integration test ([libp2p/go-libp2p-noise#102](https://github.com/libp2p/go-libp2p-noise/pull/102))
- github.com/libp2p/go-libp2p-peerstore (v0.2.8 -> v0.4.0):
  - Update version.json (#178) ([libp2p/go-libp2p-peerstore#178](https://github.com/libp2p/go-libp2p-peerstore/pull/178))
  - limit the number of protocols we store per peer ([libp2p/go-libp2p-peerstore#172](https://github.com/libp2p/go-libp2p-peerstore/pull/172))
  - sync: update CI config files (#177) ([libp2p/go-libp2p-peerstore#177](https://github.com/libp2p/go-libp2p-peerstore/pull/177))
  - feat: plumb through datastore contexts (#176) ([libp2p/go-libp2p-peerstore#176](https://github.com/libp2p/go-libp2p-peerstore/pull/176))
  - remove leftover peerstore implementation in the root package ([libp2p/go-libp2p-peerstore#173](https://github.com/libp2p/go-libp2p-peerstore/pull/173))
  - fix: replace deprecated call ([libp2p/go-libp2p-peerstore#168](https://github.com/libp2p/go-libp2p-peerstore/pull/168))
  - feat: remove queue ([libp2p/go-libp2p-peerstore#166](https://github.com/libp2p/go-libp2p-peerstore/pull/166))
  - remove deprecated types ([libp2p/go-libp2p-peerstore#165](https://github.com/libp2p/go-libp2p-peerstore/pull/165))
- github.com/libp2p/go-libp2p-pubsub (v0.5.4 -> v0.6.0):
  - feat: plumb through context changes (#459) ([libp2p/go-libp2p-pubsub#459](https://github.com/libp2p/go-libp2p-pubsub/pull/459))
  - support MinTopicSize without a discovery mechanism
  - clear peerPromises map when fulfilling a promise
  - README: remove obsolete notice, fix example code for tracing.
  - remove peer filter check from subscriptions (#453) ([libp2p/go-libp2p-pubsub#453](https://github.com/libp2p/go-libp2p-pubsub/pull/453))
  - Create peer filter option
- github.com/libp2p/go-libp2p-pubsub-router (v0.4.0 -> v0.5.0):
  - Version 0.5.0
  - feat: plumb through datastore contexts
  - sync: update CI config files (#86) ([libp2p/go-libp2p-pubsub-router#86](https://github.com/libp2p/go-libp2p-pubsub-router/pull/86))
  - Remove arbitrary sleeps from tests ([libp2p/go-libp2p-pubsub-router#87](https://github.com/libp2p/go-libp2p-pubsub-router/pull/87))
  - cleanup: fix staticcheck failures ([libp2p/go-libp2p-pubsub-router#84](https://github.com/libp2p/go-libp2p-pubsub-router/pull/84))
  - Add WithDatastore option. ([libp2p/go-libp2p-pubsub-router#82](https://github.com/libp2p/go-libp2p-pubsub-router/pull/82))
- github.com/libp2p/go-libp2p-quic-transport (v0.12.0 -> v0.15.0):
  - release v0.15.0 (#241) ([libp2p/go-libp2p-quic-transport#241](https://github.com/libp2p/go-libp2p-quic-transport/pull/241))
  - reuse the same router until we change listeners ([libp2p/go-libp2p-quic-transport#240](https://github.com/libp2p/go-libp2p-quic-transport/pull/240))
  - release v0.14.0 ([libp2p/go-libp2p-quic-transport#237](https://github.com/libp2p/go-libp2p-quic-transport/pull/237))
  - fix error assertions in the tracer ([libp2p/go-libp2p-quic-transport#234](https://github.com/libp2p/go-libp2p-quic-transport/pull/234))
  - sync: update CI config files (#235) ([libp2p/go-libp2p-quic-transport#235](https://github.com/libp2p/go-libp2p-quic-transport/pull/235))
  - read the client option from the simultaneous connect context ([libp2p/go-libp2p-quic-transport#230](https://github.com/libp2p/go-libp2p-quic-transport/pull/230))
- github.com/libp2p/go-libp2p-swarm (v0.5.3 -> v0.8.0):
  - Version 0.8.0 (#292) ([libp2p/go-libp2p-swarm#292](https://github.com/libp2p/go-libp2p-swarm/pull/292))
  - feat: plumb contexts through from peerstore (#290) ([libp2p/go-libp2p-swarm#290](https://github.com/libp2p/go-libp2p-swarm/pull/290))
  - release v0.7.0 ([libp2p/go-libp2p-swarm#289](https://github.com/libp2p/go-libp2p-swarm/pull/289))
  - update go-tcp-transport to v0.4.0 ([libp2p/go-libp2p-swarm#287](https://github.com/libp2p/go-libp2p-swarm/pull/287))
  - remove the ConnHandler ([libp2p/go-libp2p-swarm#286](https://github.com/libp2p/go-libp2p-swarm/pull/286))
  - sync: update CI config files (#288) ([libp2p/go-libp2p-swarm#288](https://github.com/libp2p/go-libp2p-swarm/pull/288))
  - remove a lot of incorrect statements from the README ([libp2p/go-libp2p-swarm#284](https://github.com/libp2p/go-libp2p-swarm/pull/284))
  - unexport the DialSync ([libp2p/go-libp2p-swarm#281](https://github.com/libp2p/go-libp2p-swarm/pull/281))
  - add an error return value to the constructor ([libp2p/go-libp2p-swarm#280](https://github.com/libp2p/go-libp2p-swarm/pull/280))
  - use functional options to configure the swarm ([libp2p/go-libp2p-swarm#279](https://github.com/libp2p/go-libp2p-swarm/pull/279))
  - stop using goprocess to control teardown ([libp2p/go-libp2p-swarm#278](https://github.com/libp2p/go-libp2p-swarm/pull/278))
  - read and use the direction from the simultaneous connect context ([libp2p/go-libp2p-swarm#277](https://github.com/libp2p/go-libp2p-swarm/pull/277))
  - simplify the DialSync code ([libp2p/go-libp2p-swarm#272](https://github.com/libp2p/go-libp2p-swarm/pull/272))
  - remove redundant self-dialing check, simplify starting of dialWorkerLoop ([libp2p/go-libp2p-swarm#273](https://github.com/libp2p/go-libp2p-swarm/pull/273))
  - add a test case for the testing package ([libp2p/go-libp2p-swarm#276](https://github.com/libp2p/go-libp2p-swarm/pull/276))
  - simplify limiter by removing the injected isFdConsumingFnc ([libp2p/go-libp2p-swarm#274](https://github.com/libp2p/go-libp2p-swarm/pull/274))
  - update badges ([libp2p/go-libp2p-swarm#271](https://github.com/libp2p/go-libp2p-swarm/pull/271))
  - remove unused context in Swarm.dialWorkerLoop ([libp2p/go-libp2p-swarm#268](https://github.com/libp2p/go-libp2p-swarm/pull/268))
  - remove Codecov config ([libp2p/go-libp2p-swarm#270](https://github.com/libp2p/go-libp2p-swarm/pull/270))
  - fix race condition in TestFailFirst ([libp2p/go-libp2p-swarm#269](https://github.com/libp2p/go-libp2p-swarm/pull/269))
- github.com/libp2p/go-libp2p-testing (v0.4.2 -> v0.5.0):
  - chore: update go-libp2p-core to v0.10.0 ([libp2p/go-libp2p-testing#38](https://github.com/libp2p/go-libp2p-testing/pull/38))
  - sync: update CI config files (#37) ([libp2p/go-libp2p-testing#37](https://github.com/libp2p/go-libp2p-testing/pull/37))
- github.com/libp2p/go-libp2p-tls (v0.2.0 -> v0.3.1):
  - release v0.3.1 ([libp2p/go-libp2p-tls#101](https://github.com/libp2p/go-libp2p-tls/pull/101))
  - set a random certificate subject ([libp2p/go-libp2p-tls#100](https://github.com/libp2p/go-libp2p-tls/pull/100))
  - sync: update CI config files (#96) ([libp2p/go-libp2p-tls#96](https://github.com/libp2p/go-libp2p-tls/pull/96))
  - add the peer ID to SecureInbound ([libp2p/go-libp2p-tls#94](https://github.com/libp2p/go-libp2p-tls/pull/94))
  - sync: update CI config files ([libp2p/go-libp2p-tls#91](https://github.com/libp2p/go-libp2p-tls/pull/91))
- github.com/libp2p/go-libp2p-transport-upgrader (v0.4.6 -> v0.5.0):
  - increase timeout in TestConnectionsClosedIfNotAccepted on CI ([libp2p/go-libp2p-transport-upgrader#85](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/85))
  - add the peer ID to SecureInbound ([libp2p/go-libp2p-transport-upgrader#83](https://github.com/libp2p/go-libp2p-transport-upgrader/pull/83))
- github.com/libp2p/go-msgio (v0.0.6 -> v0.1.0):
  - sync: update CI config files (#27) ([libp2p/go-msgio#27](https://github.com/libp2p/go-msgio/pull/27))
  - remove .gxignore file ([libp2p/go-msgio#24](https://github.com/libp2p/go-msgio/pull/24))
  - remove Codecov config ([libp2p/go-msgio#26](https://github.com/libp2p/go-msgio/pull/26))
  - remove "Chan" type ([libp2p/go-msgio#23](https://github.com/libp2p/go-msgio/pull/23))
- github.com/libp2p/go-nat (v0.0.5 -> v0.1.0):
  - pass a context to DiscoverGateway ([libp2p/go-nat#23](https://github.com/libp2p/go-nat/pull/23))
- github.com/libp2p/go-reuseport (v0.0.2 -> v0.1.0):
  - stop using github.com/pkg/errors ([libp2p/go-reuseport#85](https://github.com/libp2p/go-reuseport/pull/85))
  - sync: update CI config files (#84) ([libp2p/go-reuseport#84](https://github.com/libp2p/go-reuseport/pull/84))
- github.com/libp2p/go-reuseport-transport (v0.0.5 -> v0.1.0):
  - remove Codecov config file ([libp2p/go-reuseport-transport#36](https://github.com/libp2p/go-reuseport-transport/pull/36))
  - chore: update go-log to v2 ([libp2p/go-reuseport-transport#35](https://github.com/libp2p/go-reuseport-transport/pull/35))
  - sync: update CI config files ([libp2p/go-reuseport-transport#31](https://github.com/libp2p/go-reuseport-transport/pull/31))
- github.com/libp2p/go-tcp-transport (v0.2.8 -> v0.4.0):
  - release v0.4.0 ([libp2p/go-tcp-transport#108](https://github.com/libp2p/go-tcp-transport/pull/108))
  - sync: update CI config files (#107) ([libp2p/go-tcp-transport#107](https://github.com/libp2p/go-tcp-transport/pull/107))
  - remove the deprecated IPFS_REUSEPORT command line flag ([libp2p/go-tcp-transport#104](https://github.com/libp2p/go-tcp-transport/pull/104))
  - add options to the constructor ([libp2p/go-tcp-transport#99](https://github.com/libp2p/go-tcp-transport/pull/99))
  - remove the context from the libp2p constructor in README ([libp2p/go-tcp-transport#101](https://github.com/libp2p/go-tcp-transport/pull/101))
  - don't use libp2p.ChainOption in README ([libp2p/go-tcp-transport#102](https://github.com/libp2p/go-tcp-transport/pull/102))
  - remove incorrect statement about dns addresses in README ([libp2p/go-tcp-transport#100](https://github.com/libp2p/go-tcp-transport/pull/100))
  - use the assigned role when upgrading a sim open connection ([libp2p/go-tcp-transport#95](https://github.com/libp2p/go-tcp-transport/pull/95))
  - chore: update go-log to v2 ([libp2p/go-tcp-transport#97](https://github.com/libp2p/go-tcp-transport/pull/97))
  - simplify dial timeout context ([libp2p/go-tcp-transport#94](https://github.com/libp2p/go-tcp-transport/pull/94))
- github.com/libp2p/go-yamux/v2 (v2.2.0 -> v2.3.0):
  - limit the number of concurrent incoming streams ([libp2p/go-yamux#66](https://github.com/libp2p/go-yamux/pull/66))
  - drastically reduce allocations in ring buffer implementation (#64) ([libp2p/go-yamux#64](https://github.com/libp2p/go-yamux/pull/64))
  - sync: update CI config files (#63) ([libp2p/go-yamux#63](https://github.com/libp2p/go-yamux/pull/63))
  - remove call to asyncNotify in Stream.Read
- github.com/libp2p/zeroconf/v2 (v2.0.0 -> v2.1.1):
  - fix flaky TTL test ([libp2p/zeroconf#18](https://github.com/libp2p/zeroconf/pull/18))
  - implement a clean shutdown of the probe method ([libp2p/zeroconf#16](https://github.com/libp2p/zeroconf/pull/16))
  - remove dependency on the backoff library ([libp2p/zeroconf#17](https://github.com/libp2p/zeroconf/pull/17))
  - Don't stop browsing after ~15min ([libp2p/zeroconf#13](https://github.com/libp2p/zeroconf/pull/13))
  - fix delays when sending initial probe packets ([libp2p/zeroconf#14](https://github.com/libp2p/zeroconf/pull/14))
  - improve starting of mDNS service in tests, stop using pkg/errors ([libp2p/zeroconf#15](https://github.com/libp2p/zeroconf/pull/15))
  - update import path to include v2 in README ([libp2p/zeroconf#11](https://github.com/libp2p/zeroconf/pull/11))
- github.com/lucas-clemente/quic-go (v0.23.0 -> v0.24.0):
  - don't unlock the receive stream mutex for copying from STREAM frames ([lucas-clemente/quic-go#3290](https://github.com/lucas-clemente/quic-go/pull/3290))
  - List projects using quic-go ([lucas-clemente/quic-go#3266](https://github.com/lucas-clemente/quic-go/pull/3266))
  - disable Path MTU Discovery on Windows ([lucas-clemente/quic-go#3276](https://github.com/lucas-clemente/quic-go/pull/3276))
  - enter the regular run loop if no undecryptable packet was processed ([lucas-clemente/quic-go#3268](https://github.com/lucas-clemente/quic-go/pull/3268))
  - Allow use of custom port value in Alt-Svc header. ([lucas-clemente/quic-go#3272](https://github.com/lucas-clemente/quic-go/pull/3272))
  - disable the goconst linter ([lucas-clemente/quic-go#3286](https://github.com/lucas-clemente/quic-go/pull/3286))
  - use x/net/ipv{4,6} to construct oob info when writing packets (#3278) ([lucas-clemente/quic-go#3278](https://github.com/lucas-clemente/quic-go/pull/3278))
  - run gofmt to add the new go:build tags ([lucas-clemente/quic-go#3277](https://github.com/lucas-clemente/quic-go/pull/3277))
  - fix log string in client example ([lucas-clemente/quic-go#3264](https://github.com/lucas-clemente/quic-go/pull/3264))
- github.com/multiformats/go-multiaddr (v0.4.0 -> v0.4.1):
  - add the plaintextv2 protocol ([multiformats/go-multiaddr#165](https://github.com/multiformats/go-multiaddr/pull/165))
- github.com/multiformats/go-multihash (v0.0.15 -> v0.1.0):
  - bump version to v0.1.0 ([multiformats/go-multihash#151](https://github.com/multiformats/go-multihash/pull/151))
  - add version.json per tooling convention.
  - murmur3 support (#150) ([multiformats/go-multihash#150](https://github.com/multiformats/go-multihash/pull/150))
  - Add variations of sha2 ([multiformats/go-multihash#149](https://github.com/multiformats/go-multihash/pull/149))
  - don't use pointers for Multihash.String
  - Add blake3 hash and sharness tests ([multiformats/go-multihash#147](https://github.com/multiformats/go-multihash/pull/147))
  - remove Makefile ([multiformats/go-multihash#142](https://github.com/multiformats/go-multihash/pull/142))
  - fix staticcheck ([multiformats/go-multihash#141](https://github.com/multiformats/go-multihash/pull/141))
  - New SumStream function reads from io.Reader ([multiformats/go-multihash#138](https://github.com/multiformats/go-multihash/pull/138))
- github.com/warpfork/go-testmark (v0.3.0 -> v0.9.0):
  - testexec: will now always set up tmpdirs.
  - testexec: fix typo in error message.
  - testexec: subtest ("then-*") feature ([warpfork/go-testmark#7](https://github.com/warpfork/go-testmark/pull/7))
  - testexec: quote error from child; attribution better via more t.Helper.
  - Improve documentation of format.
  - Rename Hunk.BlockTag -> InfoString.
  - testexec: will now create tmpdirs and files for you if you have an 'fs' entry tree.
  - testexec: getting exit codes correctly. ([warpfork/go-testmark#6](https://github.com/warpfork/go-testmark/pull/6))
  - fix parsing CRLF files, part 3 ([warpfork/go-testmark#5](https://github.com/warpfork/go-testmark/pull/5))
  - fix parsing CRLF files, part 2 ([warpfork/go-testmark#4](https://github.com/warpfork/go-testmark/pull/4))
  - testexec: support both simple sequence and script mode.
  - Proper tests for read function.
  - avoid creeping extra linebreaks at the end of a patched document.
  - refrain from making double linebreaks when patching with content that ends in a linebreak.
  - Merge branch 'testexec'
  - add support for parsing CRLF line endings ([warpfork/go-testmark#3](https://github.com/warpfork/go-testmark/pull/3))
  - link to patch example code
  - More readme; and, parsing recommendations document.
  - Further improve readme.

### ❤️ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Will | 13 | +73226/-130481 | 43 |
| Masih H. Derkani | 99 | +10549/-5799 | 489 |
| hannahhoward | 43 | +5515/-3293 | 233 |
| Daniel Martí | 60 | +5312/-2883 | 208 |
| Marten Seemann | 175 | +4839/-3254 | 396 |
| Eric Myhre | 73 | +3924/-3328 | 175 |
| Jessica Schilling | 52 | +2709/-2386 | 75 |
| Rod Vagg | 30 | +2719/-1703 | 79 |
| vyzo | 10 | +3516/-177 | 87 |
| Gus Eggert | 64 | +1677/-1416 | 147 |
| Adin Schmahmann | 23 | +1708/-381 | 95 |
| Lucas Molas | 14 | +1557/-365 | 48 |
| Will Scott | 7 | +1846/-15 | 34 |
| Steven Allen | 32 | +537/-897 | 56 |
| Cory Schwartz | 3 | +614/-109 | 12 |
| rht | 3 | +576/-4 | 7 |
| Simon Zhu | 9 | +352/-51 | 16 |
| Petar Maymounkov | 7 | +173/-167 | 23 |
| RubenKelevra | 1 | +107/-188 | 1 |
| jwh | 2 | +212/-80 | 7 |
| longfeiW | 1 | +4/-249 | 10 |
| guseggert | 5 | +230/-21 | 11 |
| Kevin Neaton | 8 | +137/-80 | 13 |
| Takashi Matsuda | 1 | +199/-0 | 5 |
| Andrey Kostakov | 1 | +107/-49 | 2 |
| Jesse Bouwman | 1 | +151/-0 | 7 |
| web3-bot | 39 | +136/-3 | 52 |
| Marcin Rataj | 16 | +62/-57 | 25 |
| Marco Munizaga | 1 | +118/-0 | 2 |
| Aaron Riekenberg | 4 | +64/-52 | 6 |
| Ian Davis | 4 | +81/-32 | 7 |
| Jorropo | 2 | +79/-19 | 6 |
| Mohsin Zaidi | 1 | +89/-1 | 20 |
| Andey Robins | 1 | +70/-3 | 3 |
| gammazero | 3 | +40/-25 | 4 |
| Steve Loeppky | 2 | +26/-27 | 3 |
| Dimitris Apostolou | 1 | +25/-25 | 15 |
| Sudarshan Reddy | 1 | +9/-40 | 1 |
| Richard Littauer | 2 | +42/-1 | 3 |
| pymq | 1 | +32/-8 | 2 |
| Dirk McCormick | 2 | +23/-1 | 2 |
| Nicholas Bollweg | 1 | +21/-0 | 1 |
| anorth | 1 | +14/-6 | 2 |
| Jack Loughran | 1 | +16/-0 | 2 |
| whyrusleeping | 2 | +11/-2 | 2 |
| bt90 | 1 | +13/-0 | 1 |
| Yi Cao | 1 | +10/-0 | 1 |
| Max | 1 | +7/-3 | 1 |
| Juan Batiz-Benet | 2 | +8/-2 | 2 |
| Keenan Nemetz | 1 | +8/-0 | 1 |
| muXxer | 1 | +3/-3 | 1 |
| galargh | 2 | +3/-3 | 3 |
| Didrik Nordström | 1 | +2/-4 | 1 |
| Ben Lubar | 1 | +3/-3 | 1 |
| arjunraghurama | 1 | +5/-0 | 1 |
| Whyrusleeping | 1 | +3/-2 | 1 |
| TUSF | 1 | +3/-2 | 3 |
| mathew-cf | 1 | +3/-1 | 2 |
| Stephen Whitmore | 1 | +2/-2 | 1 |
| Song Zhu | 1 | +2/-2 | 1 |
| Michael Muré | 1 | +4/-0 | 1 |
| Alex Good | 1 | +4/-0 | 2 |
| aarshkshah1992 | 1 | +2/-1 | 1 |
| susarlanikhilesh | 1 | +1/-1 | 1 |
| falstack | 1 | +1/-1 | 1 |
| Michael Vorburger ⛑️ | 1 | +1/-1 | 1 |
| Ismail Khoffi | 1 | +1/-1 | 1 |
| George Xie | 1 | +1/-1 | 1 |
| Bryan Stenson | 1 | +1/-1 | 1 |
| Lars Gierth | 1 | +1/-0 | 1 |


# go-ipfs changelog v0.12

## v0.12.2 2022-04-08

This patch release fixes a security issue wherein traversing some malformed DAGs can cause the node to panic.

See also the security advisory: https://github.com/ipfs/go-ipfs/security/advisories/GHSA-mcq2-w56r-5w2w

Note: the v0.11.1 patch release contains the Docker compose fix from v0.12.1 as well

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipld/go-codec-dagpb (v1.3.0 -> v1.3.2):
  - fix: use protowire for Links bytes decoding

</details>

### ❤ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Rod Vagg | 1 | +34/-19 | 2 |

## v0.12.1 2022-03-17

This patch release [fixes](https://github.com/ipfs/go-ipfs/commit/816a128aaf963d72c4930852ce32b9a4e31924a1) a security issue with the `docker-compose.yaml` file in which the IPFS daemon API listens on all interfaces instead of only the loopback interface, which could allow remote callers to control your IPFS daemon. If you use the included `docker-compose.yaml` file, it is recommended to upgrade.

See also the security advisory: https://github.com/ipfs/go-ipfs/security/advisories/GHSA-fx5p-f64h-93xc

Thanks to @LynHyper for finding and disclosing this.

### Changelog

<details>
<summary>Full Changelog</summary>

- github.com/ipfs/go-ipfs:
  -  fix: listen on loopback for API and gateway ports in docker-compose.yaml

</details>

### ❤ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| guseggert | 1 | +10/-3 | 1 |

## v0.12.0 2022-02-17

We're happy to announce go-ipfs 0.12.0. This release switches the storage of IPLD blocks to be keyed by multihash instead of CID.

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [release process](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

### 🛠 BREAKING CHANGES

- `ipfs refs local` will now list all blocks as if they were [raw]() CIDv1 instead of with whatever CID version and IPLD codecs they were stored with. All other functionality should remain the same.

Note: This change also effects [ipfs-update](https://github.com/ipfs/ipfs-update) so if you use that tool to mange your go-ipfs installation then grab ipfs-update v1.8.0 from [dist](https://dist.ipfs.tech/#ipfs-update).

Keep reading to learn more details.

#### 🔦 Highlights

There is only one change since 0.11:

##### Blockstore migration from full CID to Multihash keys

We are switching the default low level [datastore](https://docs.ipfs.tech/concepts/glossary/#datastore) to be keyed only by the [Multihash](https://docs.ipfs.tech/concepts/glossary/#multihash) part of the [CID](https://docs.ipfs.tech/concepts/glossary/#cid), and deduplicate some [blocks](https://docs.ipfs.tech/concepts/glossary/#block) in the process. The blockstore will become [codec](https://docs.ipfs.tech/concepts/glossary/#codec)-agnostic.

###### Rationale

The blockstore/datastore layers are not concerned with data interpretation, only with storage of binary blocks and verification that the Multihash they are addressed with (which comes from the CID), matches the block. In fact, different CIDs, with different codecs prefixes, may be carrying the same multihash, and referencing the same block. Carrying the CID abstraction so low on the stack means potentially fetching and storing the same blocks multiple times just because they are referenced by different CIDs. Prior to this change, a CIDv1 with a `dag-cbor` codec and a CIDv1 with a `raw` codec, both containing the same multihash, would result in two identical blocks stored. A CIDv0 and CIDv1 both being the same `dag-pb` block would also result in two copies.

###### How migration works

In order to perform the switch, and start referencing all blocks by their multihash, a migration will occur on update. This migration will take the repository version from 11 (current) to 12.

One thing to note is that any content addressed CIDv0 (all the hashes that start with `Qm...`, the current default in go-ipfs), does not need any migration, as CIDv0 are raw multihashes already. This means the migration will be very lightweight for the majority of users.

The migration process will take care of re-keying any CIDv1 block so that it is only addressed by its multihash. Large nodes with lots of CIDv1-addressed content will need to go through a heavier process as the migration happens. This is how the migration works:

1. Phase 1: The migration script will perform a pass for every block in the datastore and will add all CIDv1s found to a file named `11-to-12-cids.txt`, in the go-ipfs configuration folder. Nothing is written in this first phase and it only serves to identify keys that will be migrated in phase 2.
2. Phase 2: The migration script will perform a second pass where every CIDv1 block will be read and re-written with its raw-multihash as key. There is 1 worker performing this task, although more can be configured. Every 100MiB-worth of blocks (this is configurable), each worker will trigger a datastore "sync" (to ensure all written data is flushed to disk) and delete the CIDv1-addressed blocks that were just renamed. This provides a good compromise between speed and resources needed to run the migration.

At every sync, the migration emits a log message showing how many blocks need to be rewritten and how far the process is.

####### FlatFS specific migration

For those using a single FlatFS datastore as their backing blockstore (i.e. the default behavior), the migration (but not reversion) will take advantage of the ability to easily move/rename the blocks to improve migration performance.

Unfortunately, other common datastores do not support renames which is what makes this FlatFS specific. If you are running a large custom datastore that supports renames you may want to consider running a fork of [fs-repo-11-to-12](https://github.com/ipfs/fs-repo-migrations/tree/master/fs-repo-11-to-12) specific to your datastore.

If you want to disable this behavior, set the environment variable `IPFS_FS_MIGRATION_11_TO_12_ENABLE_FLATFS_FASTPATH` to `false`.

####### Migration configuration

For those who want to tune the migration more precisely for their setups, there are two environment variables to configure:

- `IPFS_FS_MIGRATION_11_TO_12_NWORKERS` : an integer describing the number of migration workers - defaults to 1
- `IPFS_FS_MIGRATION_11_TO_12_SYNC_SIZE_BYTES` : an integer describing the number of bytes after which migration workers will sync - defaults to 104857600 (i.e. 100MiB)

###### Migration caveats

Large repositories with very large numbers of CIDv1s should be mindful of the migration process:

* We recommend ensuring that IPFS runs with an appropriate (high) file-descriptor limit, particularly when Badger is use as datastore backend. Badger is known to open many tables when experiencing a high number of writes, which may trigger "too many files open" type of errors during the migrations. If this happens, the migration can be retried with a higher FD limit (see below).
* Migrations using the Badger datastore may not immediately reclaim the space freed by the deletion of migrated blocks, thus space requirements may grow considerably. A periodic Badger-GC is run every 2 minutes, which will reclaim space used by deleted and de-duplicated blocks. The last portion of the space will only be reclaimed after go-ipfs starts (the Badger-GC cycle will trigger after 15 minutes).
* While there is a revert process detailed below, we recommend keeping a backup of the repository, particularly for very large ones, in case an issue happens, so that the revert can happen immediately and cases of repository corruption due to crashes or unexpected circumstances are not catastrophic.

###### Migration interruptions and retries

If a problem occurs during the migration, it is be possible to simply re-start and retry it:

1. Phase 1 will never overwrite the `11-to-12-cids.txt` file, but only append to it (so that a list of things we were supposed to have migrated during our first attempt is not lost - this is important for reverts, see below).
2. Phase 2 will proceed to continue re-keying blocks that were not re-keyed during previous attempts.

###### Migration reverts

It is also possible to revert the migration after it has succeeded, for example to go to a previous go-ipfs version (<=[0.11](https://github.com/ipfs/go-ipfs/releases/tag/v0.11.0)), even after starting and using go-ipfs in the new version (>=0.12). The revert process works as follows:

1. The `11-to-12-cids.txt` file is read, which has the list of all the CIDv1s that had to be rewritten for the migration.
2. A CIDv1-addressed block is written for every item on the list. This work is performed by 1 worker (configurable), syncing every 100MiB (configurable).
3. It is ensured that every CIDv1 pin, and every CIDv1 reference in MFS, are also written as CIDV1-addressed blocks, regardless of whether they were part of the original migration or were added later.

The revert process does not delete any blocks--it only makes sure that blocks that were accessible with CIDv1s before the migration are again keyed with CIDv1s. This may result in a datastore becoming twice as large (i.e. if all the blocks were CIDv1-addressed before the migration). This is however done this way to cover corner cases: user can add CIDv1s after migration, which may reference blocks that existed as CIDv0 before migration. The revert aims to ensure that no data becomes unavailable on downgrade.

While go-ipfs will auto-run the migration for you, it will not run the reversion. To do so you can download the [latest migration binary](https://dist.ipfs.tech/fs-repo-11-to-12) or use [ipfs-update](https://dist.ipfs.tech/#ipfs-update).

###### Custom datastores

As with previous migrations if you work with custom datastores and want to leverage the migration you can run a fork of [fs-repo-11-to-12](https://github.com/ipfs/fs-repo-migrations/tree/master/fs-repo-11-to-12) specific to your datastore. The repo includes instructions on building for different datastores.

For this migration, if your datastore has fast renames you may want to consider writing some code to leverage the particular efficiencies of your datastore similar to what was done for FlatFS.

### Changelog

- github.com/ipfs/go-ipfs:
  - Release v0.12.0
  - docs: v0.12.0 release notes
  - chore: bump migrations dist.ipfs.tech CID to contain fs-repo-11-to-12 v1.0.2
  - feat: refactor Fetcher interface used for downloading migrations (#8728) ([ipfs/go-ipfs#8728](https://github.com/ipfs/go-ipfs/pull/8728))
  - feat: log multifetcher errors
  - Release v0.12.0-rc1
  - chore: bump Go version to 1.16.12
  - feat: switch to raw multihashes for blocks ([ipfs/go-ipfs#6816](https://github.com/ipfs/go-ipfs/pull/6816))
  - chore: add release template snippet for fetching artifact tarball
  - chore: bump Go version to 1.16.11
  - chore: add release steps for upgrading Go
  - Merge branch 'release'
  - fix(corehttp): adjust peer counting metrics (#8577) ([ipfs/go-ipfs#8577](https://github.com/ipfs/go-ipfs/pull/8577))
  - chore: update version to v0.12.0-dev
- github.com/ipfs/go-filestore (v0.1.0 -> v1.1.0):
  - Version 1.1.0
  - feat: plumb through context changes
  - sync: update CI config files (#54) ([ipfs/go-filestore#54](https://github.com/ipfs/go-filestore/pull/54))
  - fix staticcheck ([ipfs/go-filestore#48](https://github.com/ipfs/go-filestore/pull/48))
  - Work with Multihashes directly (#21) ([ipfs/go-filestore#21](https://github.com/ipfs/go-filestore/pull/21))
- github.com/ipfs/go-ipfs-blockstore (v0.2.1 -> v1.1.2):
  - Release v1.1.2
  - feat: per-cid locking
  - Version 1.1.1
  - fix: remove context from HashOnRead
  - Version 1.1.0 (#91) ([ipfs/go-ipfs-blockstore#91](https://github.com/ipfs/go-ipfs-blockstore/pull/91))
  - feat: add context to interfaces (#90) ([ipfs/go-ipfs-blockstore#90](https://github.com/ipfs/go-ipfs-blockstore/pull/90))
  - sync: update CI config files (#88) ([ipfs/go-ipfs-blockstore#88](https://github.com/ipfs/go-ipfs-blockstore/pull/88))
  - add constructor that doesn't mess with datastore keys ([ipfs/go-ipfs-blockstore#83](https://github.com/ipfs/go-ipfs-blockstore/pull/83))
  - Use bloom filter in GetSize
  - fix staticcheck ([ipfs/go-ipfs-blockstore#73](https://github.com/ipfs/go-ipfs-blockstore/pull/73))
  - add BenchmarkARCCacheConcurrentOps ([ipfs/go-ipfs-blockstore#70](https://github.com/ipfs/go-ipfs-blockstore/pull/70))
  - fix(arc): striped locking on last byte of CID ([ipfs/go-ipfs-blockstore#67](https://github.com/ipfs/go-ipfs-blockstore/pull/67))
  - make idstore implement io.Closer. (#60) ([ipfs/go-ipfs-blockstore#60](https://github.com/ipfs/go-ipfs-blockstore/pull/60))
  - add View() to all the various blockstores. (#59) ([ipfs/go-ipfs-blockstore#59](https://github.com/ipfs/go-ipfs-blockstore/pull/59))
  - Optimize id store ([ipfs/go-ipfs-blockstore#56](https://github.com/ipfs/go-ipfs-blockstore/pull/56))
  - add race fix for HashOnRead ([ipfs/go-ipfs-blockstore#50](https://github.com/ipfs/go-ipfs-blockstore/pull/50))
  - Add test to maintain Put contract of calling Has first ([ipfs/go-ipfs-blockstore#47](https://github.com/ipfs/go-ipfs-blockstore/pull/47))
  - Update readme and license ([ipfs/go-ipfs-blockstore#44](https://github.com/ipfs/go-ipfs-blockstore/pull/44))
  - feat: switch to raw multihashes for blocks ([ipfs/go-ipfs-blockstore#38](https://github.com/ipfs/go-ipfs-blockstore/pull/38))
- github.com/ipfs/go-ipfs-ds-help (v0.1.1 -> v1.1.0):
  - Update version.json (#38) ([ipfs/go-ipfs-ds-help#38](https://github.com/ipfs/go-ipfs-ds-help/pull/38))
  - sync: update CI config files (#37) ([ipfs/go-ipfs-ds-help#37](https://github.com/ipfs/go-ipfs-ds-help/pull/37))
  - feat: switch to raw multihashes for blocks ([ipfs/go-ipfs-ds-help#18](https://github.com/ipfs/go-ipfs-ds-help/pull/18))

### ❤️ Contributors

| Contributor | Commits | Lines ± | Files Changed |
|-------------|---------|---------|---------------|
| Gus Eggert | 10 | +333/-321 | 24 |
| Steven Allen | 7 | +289/-190 | 13 |
| Hector Sanjuan | 9 | +134/-109 | 18 |
| Adin Schmahmann | 11 | +179/-55 | 21 |
| Raúl Kripalani | 2 | +152/-42 | 5 |
| Daniel Martí | 1 | +120/-1 | 1 |
| frrist | 1 | +95/-13 | 2 |
| Alex Trottier | 2 | +22/-11 | 4 |
| Andrey Petrov | 1 | +32/-0 | 1 |
| Lucas Molas | 1 | +18/-7 | 2 |
| Marten Seemann | 2 | +11/-7 | 3 |
| whyrusleeping | 1 | +10/-0 | 1 |
| web3-bot | 3 | +9/-0 | 3 |
| postables | 1 | +5/-3 | 1 |
| Dr Ian Preston | 1 | +4/-0 | 1 |
