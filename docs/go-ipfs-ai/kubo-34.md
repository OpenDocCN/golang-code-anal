# go-ipfs 源码解析 34

# Transferring a file with ipfs
This document is a guide to help troubleshoot transferring a file between two
machines using ipfs.

## A file transfer
To start, make sure that ipfs is running on both machines. To verify, run `ipfs
id` on each machine and check if the `Addresses` field has anything in it. If
it says `null`, then your node is not online and you will need to run `ipfs
daemon`.

Now, lets call the node with the file you want to transfer node 'A' and the
node you want to get the file to node 'B'. On node A, add the file to ipfs
using the `ipfs add` command. This will print out the multihash of the content
you added. Now, on node B, you can fetch the content using `ipfs get <hash>`.

```
# On A
> ipfs add myfile.txt
added QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye myfile.txt

# On B
> ipfs get QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye
Saving file(s) to QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye
 13 B / 13 B [=====================================================] 100.00% 1s
 ```

If that worked, and downloaded the file, then congratulations! You just used
ipfs to move files across the internet! But, if that `ipfs get` command is
hanging, with no output, read onwards.

## Troubleshooting

So your ipfs file transfer appears to not be working. The primary reason this
happens is because node B cannot figure out how to connect to node A, or node B
doesn't even know it has to connect to node A. 

### Checking for existing connections 

The first thing to do is to double check that both nodes are in fact running
and online. To do this, run `ipfs id` on each machine. If both nodes show some
addresses (like the example below), then your nodes are online.

```json
{
        "ID": "QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
        "PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDZb6znj3LQZKP1+X81exf+vbnqNCMtHjZ5RKTCm7Fytnfe+AI1fhs9YbZdkgFkM1HLxmIOLQj2bMXPIGxUM+EnewN8tWurx4B3+lR/LWNwNYcCFL+jF2ltc6SE6BC8kMLEZd4zidOLPZ8lIRpd0x3qmsjhGefuRwrKeKlR4tQ3C76ziOms47uLdiVVkl5LyJ5+mn4rXOjNKt/oy2O4m1St7X7/yNt8qQgYsPfe/hCOywxCEIHEkqmil+vn7bu4RpAtsUzCcBDoLUIWuU3i6qfytD05hP8Clo+at+l//ctjMxylf3IQ5qyP+yfvazk+WHcsB0tWueEmiU5P2nfUUIR3AgMBAAE=",
        "Addresses": [
                "/ip4/127.0.0.1/tcp/4001/p2p/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
                "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
                "/ip4/192.168.2.131/tcp/4001/p2p/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
                "/ip4/192.168.2.131/udp/4001/quic-v1/p2p/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
        ],
        "AgentVersion": "go-ipfs/0.4.11-dev/",
        "ProtocolVersion": "ipfs/0.1.0"
}
```

Next, check to see if the nodes have a connection to each other. You can do this
by running `ipfs swarm peers` on one node, and checking for the other nodes
peer ID in the output. If the two nodes *are* connected, and the `ipfs get`
command is still hanging, then something unexpected is going on, and I
recommend filing an issue about it. If they are not connected, then let's try
and debug why. (Note: you can skip to 'Manually connecting node A to node B' if
you just want things to work. Going through the debugging process and reporting
what happened to the ipfs team on IRC is helpful to us to understand common
pitfalls that people run into)

### Checking providers
When requesting content on ipfs, nodes search the DHT for 'provider records' to
see who has what content. Let's manually do that on node B to make sure that
node B is able to determine that node A has the data. Run `ipfs dht findprovs
<hash>`. We expect to see the peer ID of node A printed out. If this command
returns nothing (or returns IDs that are not node A), then no record of A
having the data exists on the network. This can happen if the data is added
while node A does not have a daemon running. If this happens, you can run `ipfs
dht provide <hash>` on node A to announce to the network that you have that
hash. Then if you restart the `ipfs get` command, node B should now be able
to tell that node A has the content it wants. If node A's peer ID showed up in
the initial `findprovs` call, or manually providing the hash didn't resolve the
problem, then it's likely that node B is unable to make a connection to node A.

### Checking addresses

In the case where node B simply cannot form a connection to node A, despite
knowing that it needs to, the likely culprit is a bad NAT. When node B learns
that it needs to connect to node A, it checks the DHT for addresses for node A,
and then starts trying to connect to them. We can check those addresses by
running `ipfs dht findpeer <node A peerID>` on node B. This command should
return a list of addresses for node A. If it doesn't return any addresses, then
you should try running the manual providing command from the previous steps. 
Example output of addresses might look something like this:

```
/ip4/127.0.0.1/tcp/4001
/ip4/127.0.0.1/udp/4001/quic-v1
/ip4/192.168.2.133/tcp/4001
/ip4/192.168.2.133/udp/4001/quic-v1
/ip4/88.157.217.196/tcp/63674
/ip4/88.157.217.196/udp/63674/quic-v1
```

In this case, we can see a localhost (127.0.0.1) address, a LAN address (the
192.168.*.* one) and another address. If this third address matches your
external IP, then the network knows a valid external address for your node. At
this point, its safe to assume that your node has a difficult to traverse NAT
situation. If this is the case, you can try to enable UPnP or NAT-PMP on the
router of node A and retry the process. Otherwise, you can try manually
connecting node A to node B.

### Manually connecting node A to B 

On node B run `ipfs id` and take one of the multiaddrs that contains its public
ip address, and then on node A run `ipfs swarm connect <multiaddr>`.  You can
also try using a relayed connection, for more information [read this
doc](./experimental-features.md#circuit-relay). If that *still* doesn't work,
then you should either join IRC and ask for help there, or file an issue on
github.

If this manual step *did* work, then you likely have an issue with NAT
traversal, and ipfs cannot figure out how to make it through. Please report
situations like this to us so we can work on fixing them.


# FUSE

**EXPERIMENTAL:** FUSE support is limited, YMMV.

Kubo makes it possible to mount `/ipfs` and `/ipns` namespaces in your OS,
allowing arbitrary apps access to IPFS.

## Install FUSE

You will need to install and configure fuse before you can mount IPFS

#### Linux

Note: while this guide should work for most distributions, you may need to refer
to your distribution manual to get things working.

Install `fuse` with your favorite package manager:
```
sudo apt-get install fuse
```

On some older Linux distributions, you may need to add yourself to the `fuse` group.  
(If no such group exists, you can probably skip this step)
```sh
sudo usermod -a -G fuse <username>
```

Restart user session, if active, for the change to apply, either by restarting
ssh connection or by re-logging to the system.

#### Mac OSX -- OSXFUSE

It has been discovered that versions of `osxfuse` prior to `2.7.0` will cause a
kernel panic. For everyone's sake, please upgrade (latest at time of writing is
`2.7.4`). The installer can be found at https://osxfuse.github.io/. There is
also a homebrew formula (`brew cask install osxfuse`) but users report best results
installing from the official OSXFUSE installer package.

Note that `ipfs` attempts an automatic version check on `osxfuse` to prevent you
from shooting yourself in the foot if you have pre `2.7.0`. Since checking the
OSXFUSE version [is more complicated than it should be], running `ipfs mount`
may require you to install another binary:

```sh
go get github.com/jbenet/go-fuse-version/fuse-version
```

If you run into any problems installing FUSE or mounting IPFS, hop on IRC and
speak with us, or if you figure something new out, please add to this document!

## Prepare mountpoints

By default ipfs uses `/ipfs` and `/ipns` directories for mounting, this can be
changed in config. You will have to create the `/ipfs` and `/ipns` directories
explicitly. Note that modifying root requires sudo permissions.

```sh
# make the directories
sudo mkdir /ipfs
sudo mkdir /ipns

# chown them so ipfs can use them without root permissions
sudo chown <username> /ipfs
sudo chown <username> /ipns
```

Depending on whether you are using OSX or Linux, follow the proceeding instructions. 

## Make sure IPFS daemon is not running

You'll need to stop the IPFS daemon if you have it started, otherwise the mount will complain. 

```
# Check to see if IPFS daemon is running
ps aux | grep ipfs

# Kill the IPFS daemon 
pkill -f ipfs

# Verify that it has been killed
```

## Mounting IPFS

```sh
ipfs daemon --mount
```

If you wish to allow other users to use the mount points, edit `/etc/fuse.conf`
to enable non-root users, i.e.:
```sh
# /etc/fuse.conf - Configuration file for Filesystem in Userspace (FUSE)

# Set the maximum number of FUSE mounts allowed to non-root users.
# The default is 1000.
#mount_max = 1000

# Allow non-root users to specify the allow_other or allow_root mount options.
user_allow_other
```

Next set `Mounts.FuseAllowOther` config option to `true`:
```sh
ipfs config --json Mounts.FuseAllowOther true
ipfs daemon --mount
```

## Troubleshooting

#### `Permission denied` or `fusermount: user has no write access to mountpoint` error in Linux

Verify that the config file can be read by your user:
```sh
sudo ls -l /etc/fuse.conf
-rw-r----- 1 root fuse 216 Jan  2  2013 /etc/fuse.conf
```
In most distributions, the group named `fuse` will be created during fuse
installation. You can check this with:

```sh
sudo grep -q fuse /etc/group && echo fuse_group_present || echo fuse_group_missing
```

If the group is present, just add your regular user to the `fuse` group:
```sh
sudo usermod -G fuse -a <username>
```

If the group didn't exist, create `fuse` group (add your regular user to it) and
set necessary permissions, for example:
```sh
sudo chgrp fuse /etc/fuse.conf
sudo chmod g+r  /etc/fuse.conf
```
<!--
TODO: udev rules for /dev/fuse?
-->

Note that the use of `fuse` group is optional and may depend on your operating
system. It is okay to use a different group as long as proper permissions are
set for user running `ipfs mount` command.

#### Mount command crashes and mountpoint gets stuck

```
sudo umount /ipfs
sudo umount /ipns
```

If you manage to mount on other systems (or followed an alternative path to one
above), please contribute to these docs :D


# Gateway

An IPFS Gateway acts as a bridge between traditional web browsers and IPFS.
Through the gateway, users can browse files and websites stored in IPFS as if
they were stored in a traditional web server. 

[More about Gateways](https://docs.ipfs.tech/concepts/ipfs-gateway/) and [addressing IPFS on the web](https://docs.ipfs.tech/how-to/address-ipfs-on-web/).

Kubo's Gateway implementation follows [ipfs/specs: Specification for HTTP Gateways](https://github.com/ipfs/specs/tree/main/http-gateways#readme).

### Local gateway

By default, Kubo nodes run
a [path gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#path-gateway) at `http://127.0.0.1:8080/`
and a [subdomain gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway) at `http://localhost:8080/`

Additional listening addresses and gateway behaviors can be set in the [config](#configuration) file.

### Public gateways

Protocol Labs provides a public gateway at `https://ipfs.io` (path) and `https://dweb.link` (subdomain).
If you've ever seen a link in the form `https://ipfs.io/ipfs/Qm...`, that's being served from *our* gateway.

There is a list of third-party public gateways provided by the IPFS community at https://ipfs.github.io/public-gateway-checker/

## Configuration

The `Gateway.*` configuration options are (briefly) described in the
[config](https://github.com/ipfs/kubo/blob/master/docs/config.md#gateway)
documentation, including a list of common [gateway recipes](https://github.com/ipfs/kubo/blob/master/docs/config.md#gateway-recipes).

### Debug
The gateway's log level can be changed with this command:
```
> ipfs log level core/server debug
```

## Directories

For convenience, the gateway (mostly) acts like a normal web-server when serving
a directory:

1. If the directory contains an `index.html` file:
  1. If the path does not end in a `/`, append a `/` and redirect. This helps
     avoid serving duplicate content from different paths.<sup>&dagger;</sup>
  2. Otherwise, serve the `index.html` file.
2. Dynamically build and serve a listing of the contents of the directory.

<sub><sup>&dagger;</sup>This redirect is skipped if the query string contains a
`go-get=1` parameter. See [PR#3964](https://github.com/ipfs/kubo/pull/3963)
for details</sub>

## Static Websites

You can use an IPFS gateway to serve static websites at a custom domain using
[DNSLink](https://docs.ipfs.tech/concepts/glossary/#dnslink). See [Example: IPFS
Gateway](https://dnslink.dev/#example-ipfs-gateway) for instructions.

## Filenames

When downloading files, browsers will usually guess a file's filename by looking
at the last component of the path. Unfortunately, when linking *directly* to a
file (with no containing directory), the final component is just a CID
(`Qm...`). This isn't exactly user-friendly.

To work around this issue, you can add a `filename=some_filename` parameter to
your query string to explicitly specify the filename. For example:

> https://ipfs.io/ipfs/QmfM2r8seH2GiRaC4esTjeraXEachRt8ZsSeGaWTPLyMoG?filename=hello_world.txt

When you try to save above page, you browser will use passed `filename` instead of a CID.

## Downloads

It is possible to skip browser rendering of supported filetypes (plain text,
images, audio, video, PDF) and trigger immediate "save as" dialog by appending
`&download=true`:

> https://ipfs.io/ipfs/QmfM2r8seH2GiRaC4esTjeraXEachRt8ZsSeGaWTPLyMoG?filename=hello_world.txt&download=true

## Response Format

An explicit response format can be requested using `?format=raw|car|..` URL parameter,
or by sending `Accept: application/vnd.ipld.{format}` HTTP header with one of supported content types.

## Content-Types

### `application/vnd.ipld.raw`

Returns a byte array for a single `raw` block.

Sending such requests for `/ipfs/{cid}` allows for efficient fetch of blocks with data
encoded in custom format, without the need for deserialization and traversal on the gateway.

This is equivalent of `ipfs block get`.

### `application/vnd.ipld.car`

Returns a [CAR](https://ipld.io/specs/transport/car/) stream for specific DAG and selector.

Right now only 'full DAG' implicit selector is implemented.
Support for user-provided IPLD selectors is tracked in https://github.com/ipfs/kubo/issues/8769.

This is a rough equivalent of `ipfs dag export`.

## Deprecated Subset of RPC API

For legacy reasons, the gateway port exposes a small subset of RPC API under `/api/v0/`.
While this read-only API exposes a read-only, "safe" subset of the normal API,
it is deprecated and should not be used for greenfield projects.

Where possible, leverage `/ipfs/` and `/ipns/` endpoints.
along with `application/vnd.ipld.*` Content-Types instead.


# HTTP/RPC Clients

Kubo provides official HTTP RPC  (`/api/v0`) clients for selected languages:

| Language |     Package Name    | Github Repository                          |
|:--------:|:-------------------:|--------------------------------------------|
| JS       | kubo-rpc-client     | https://github.com/ipfs/js-kubo-rpc-client |
| Go       | `rpc`               | [`./client/rpc`](./client/rpc)             |


# IPFS API Implementation Doc

This short document aims to give a quick guide to anyone implementing API
bindings for IPFS implementations-- in particular kubo.

Sections:
- IPFS Types
- API Transports
- API Commands
- Implementing bindings for the HTTP API

## IPFS Types

IPFS uses a set of value type that is useful to enumerate up front:

- `<ipfs-path>` is unix-style path, beginning with `/ipfs/<cid>/...` or
  `/ipns/<hash>/...` or `/ipns/<domain>/...`.
- `<hash>` is a base58 encoded [multihash](https://github.com/multiformats/multihash)
- `cid` is a [multibase](https://github.com/multiformats/multibase) encoded
  [CID](https://github.com/ipld/cid) - a self-describing content-addressing identifier

A note on streams: IPFS is a streaming protocol. Everything about it can be
streamed. When importing files, API requests should aim to stream the data in,
and handle back-pressure correctly, so that the IPFS node can handle it
sequentially without too much memory pressure. (If using HTTP, this is typically
handled for you by writes to the request body blocking.)

## API Transports

Like with everything else, IPFS aims to be flexible regarding the API transports.
Currently, the [kubo](https://github.com/ipfs/kubo) implementation supports
both an in-process API and an HTTP API. More can be added easily, by mapping the
API functions over a transport. (This is similar to how gRPC is also _mapped on
top of transports_, like HTTP).

Mapping to a transport involves leveraging the transport's features to express
function calls. For example:

#### CLI API Transport

In the commandline, IPFS uses a traditional flag and arg-based mapping, where:
- the first arguments selects the command, as in git - e.g. `ipfs object get`
- the flags specify options - e.g. `--enc=protobuf -q`
- the rest are positional arguments - e.g.
  `ipfs object patch <hash1> add-linkfoo <hash2>`
- files are specified by filename, or through stdin

(NOTE: When kubo runs the daemon, the CLI API is actually converted to HTTP
calls. otherwise, they execute in the same process)

#### HTTP API Transport

In HTTP, our API layering uses a REST-like mapping, where:
- the URL path selects the command - e.g `/object/get`
- the URL query string implements option arguments - e.g. `&enc=protobuf&q=true`
- the URL query also implements positional arguments - e.g.
  `&arg=<hash1>&arg=add-link&arg=foo&arg=<hash2>`
- the request body streams file data - reads files or stdin
  - multiple streams are muxed with multipart (todo: add tar stream support)


## API Commands

There is a "standard IPFS API" which is currently defined as "all the commands
exposed by the kubo implementation". There are auto-generated [API Docs](https://ipfs.io/docs/api/).
You can Also see [a listing here](https://github.com/ipfs/kubo/blob/94b832df861728c65e912935641d08880c341e0a/core/commands/root.go#L96-L130), or get a list of
commands by running `ipfs commands` locally.

## Implementing bindings for the HTTP API

As mentioned above, the API commands map to HTTP with:
- the URL path selects the command - e.g `/object/get`
- the URL query string implements option arguments - e.g. `&enc=protobuf&q=true`
- the URL query also implements positional arguments - e.g.
  `&arg=<hash1>&arg=add-link&arg=foo&arg=<hash2>`
- the request body streams file data - reads files or stdin
  - multiple streams are muxed with multipart (todo: add tar stream support)

You can see the latest [list of our HTTP RPC clients here](http-rpc-clients.md)

The Go implementation is good to answer harder questions, like how is multipart
handled, or what headers should be set in edge conditions. But the javascript
implementation is very concise, and easy to follow.

## Note on multipart + inspecting requests

Despite all the generalization spoken about above, the IPFS API is actually very
simple. You can inspect all the requests made with `nc` and the `--api` option
(as of [this PR](https://github.com/ipfs/kubo/pull/1598), or `0.3.8`):

```
> nc -l 5002 &
> ipfs --api /ip4/127.0.0.1/tcp/5002 swarm addrs local --enc=json
POST /api/v0/version?enc=json&stream-channels=true HTTP/1.1
Host: 127.0.0.1:5002
User-Agent: /kubo/0.14.0/
Content-Length: 0
Content-Type: application/octet-stream
Accept-Encoding: gzip


```

The only hard part is getting the file streaming right. It is (now) fairly easy
to stream files to kubo using multipart. Basically, we end up with HTTP
requests like this:

```
> nc -l 5002 &
> ipfs --api /ip4/127.0.0.1/tcp/5002 add -r ~/demo/basic/test
POST /api/v0/add?encoding=json&progress=true&r=true&stream-channels=true HTTP/1.1
Host: 127.0.0.1:5002
User-Agent: /kubo/0.14.0/
Transfer-Encoding: chunked
Content-Disposition: form-data: name="files"
Content-Type: multipart/form-data; boundary=2186ef15d8f2c4f100af72d6d345afe36a4d17ef11264ec5b8ec4436447f
Accept-Encoding: gzip

1
-
e5
-2186ef15d8f2c4f100af72d6d345afe36a4d17ef11264ec5b8ec4436447f
Content-Disposition: form-data; name="file"; filename="test"
Content-Type: multipart/mixed; boundary=acdb172fe12f25e8ffae9981ce6f4580abdefb0cae3ceebe464d802866be


9c
--acdb172fe12f25e8ffae9981ce6f4580abdefb0cae3ceebe464d802866be
Content-Disposition: file; filename="test%2Fbar"
Content-Type: application/octet-stream


4
bar

dc

--acdb172fe12f25e8ffae9981ce6f4580abdefb0cae3ceebe464d802866be
Content-Disposition: file; filename="test%2Fbaz"
Content-Type: multipart/mixed; boundary=2799ac77a72ef7b8a0281945806b9f9a28f7681145aa8e91b052d599b2dd


a0
--2799ac77a72ef7b8a0281945806b9f9a28f7681145aa8e91b052d599b2dd
Content-Type: application/octet-stream
Content-Disposition: file; filename="test%2Fbaz%2Fb"


4
bar

a2

--2799ac77a72ef7b8a0281945806b9f9a28f7681145aa8e91b052d599b2dd
Content-Disposition: file; filename="test%2Fbaz%2Ff"
Content-Type: application/octet-stream


4
foo

44

--2799ac77a72ef7b8a0281945806b9f9a28f7681145aa8e91b052d599b2dd--

9e

--acdb172fe12f25e8ffae9981ce6f4580abdefb0cae3ceebe464d802866be
Content-Disposition: file; filename="test%2Ffoo"
Content-Type: application/octet-stream


4
foo

44

--acdb172fe12f25e8ffae9981ce6f4580abdefb0cae3ceebe464d802866be--

44

--2186ef15d8f2c4f100af72d6d345afe36a4d17ef11264ec5b8ec4436447f--

0

```

Which produces: http://gateway.ipfs.io/ipfs/QmNtpA5TBNqHrKf3cLQ1AiUKXiE4JmUodbG5gXrajg8wdv



<!-- omit in toc -->
# libp2p Network Resource Manager <small>(`Swarm.ResourceMgr`)</small>

## Purpose
The purpose of this document is to provide more information about the [libp2p Network Resource Manager](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#readme) and how it's integrated into Kubo so that Kubo users can understand and configure it appropriately.

## 🙋 Help!  The resource manager is protecting my node but I want to understand more
The resource manager is generally a *feature* to bound libp2p's resources, whether from bugs, unintentionally misbehaving peers, or intentional Denial of Service attacks.

Good places to start are:
1. Understand [how the resource manager is configured](#levels-of-configuration).
2. Understand [how to read the log message](#what-do-these-protected-from-exceeding-resource-limits-log-messages-mean)
3. Understand [how to inspect and change limits](#user-supplied-override-limits)

## Table of Contents

- [Purpose](#purpose)
- [🙋 Help!  The resource manager is protecting my node but I want to understand more](#-help--the-resource-manager-is-protecting-my-node-but-i-want-to-understand-more)
- [Table of Contents](#table-of-contents)
- [Levels of Configuration](#levels-of-configuration)
  - [Approach](#approach)
  - [Computed Default Limits](#computed-default-limits)
  - [User Supplied Override Limits](#user-supplied-override-limits)
- [FAQ](#faq)
  - [What do these "Protected from exceeding resource limits" log messages mean?](#what-do-these-protected-from-exceeding-resource-limits-log-messages-mean)
  - [How does one see the Active Limits?](#how-does-one-see-the-active-limits)
  - [How does one see the Computed Default Limits?](#how-does-one-see-the-computed-default-limits)
  - [How does one monitor libp2p resource usage?](#how-does-one-monitor-libp2p-resource-usage)
  - [How does the resource manager (ResourceMgr) relate to the connection manager (ConnMgr)?](#how-does-the-resource-manager-resourcemgr-relate-to-the-connection-manager-connmgr)
  - [What are the "Application error 0x0 (remote) ... cannot reserve ..." messages?](#what-are-the-application-error-0x0-remote--cannot-reserve--messages)
- [History](#history)

## Levels of Configuration

See also the [`Swarm.ResourceMgr` config docs](./config.md#swarmresourcemgr).

### Approach
libp2p's resource manager provides tremendous flexibility but also adds complexity.  There are these levels of limit configuration for resource management protection:

1. "The user who does nothing" - In this case Kubo attempts to give some sane defaults discussed below
   based on the amount of memory and file descriptors their system has.
   This should protect the node from many attacks.

2. "Slightly more advanced user" - Where the defaults aren't good enough, a good set of higher-level "knobs" are exposed to satisfy most use cases
   without requiring users to wade into all the intricacies of libp2p's resource manager.
   The "knobs"/inputs are `Swarm.ResourceMgr.MaxMemory` and `Swarm.ResourceMgr.MaxFileDescriptors` as described below.

3. "Power user" - They [specify override limits](#user-supplied-override-limits) and own their own destiny without Kubo getting in the way.

### Computed Default Limits
With the `Swarm.ResourceMgr.MaxMemory` and `Swarm.ResourceMgr.MaxFileDescriptors` inputs defined,
[resource manager limits](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#limits) are created at the
[system](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#the-system-scope),
[transient](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#the-transient-scope),
and [peer](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#peer-scopes) scopes.
Other scopes are ignored (by being set to "unlimited").

The reason these scopes are chosen is because:
- `system` - This gives us the coarse-grained control we want so we can reason about the system as a whole.
  It is the backstop, and allows us to reason about resource consumption more easily
  since don't have think about the interaction of many other scopes.
- `transient` - Limiting connections that are in process of being established provides backpressure so not too much work queues up.
- `peer` - The peer scope doesn't protect us against intentional DoS attacks.
  It's just as easy for an attacker to send 100 requests/second with 1 peerId vs. 10 requests/second with 10 peers.
  We are reliant on the system scope for protection here in the malicious case.
  The reason for having a peer scope is to protect against unintentional DoS attacks
  (e.g., bug in a peer which is causing it to "misbehave").
  In the unintentional case, we want to make sure a "misbehaving" node doesn't consume more resources than necessary.

Within these scopes, limits are set on:
1. [memory](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#memory)
2. [file descriptors (FD)](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#file-descriptors)
3. [*inbound* connections](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#connections).
Limits are set based on the `Swarm.ResourceMgr.MaxMemory` and `Swarm.ResourceMgr.MaxFileDescriptors` inputs above.

There are also some special cases where minimum values are enforced.
For example, Kubo maintainers have found in practice that it's a footgun to have too low of a value for `System.ConnsInbound` and a default minimum is used. (See [core/node/libp2p/rcmgr_defaults.go](https://github.com/ipfs/kubo/blob/master/core/node/libp2p/rcmgr_defaults.go) for specifics.)

We trust this node to behave properly and thus don't limit *outbound* connection/stream limits.
We apply any limits that libp2p has for its protocols/services
since we assume libp2p knows best here.

Source: [core/node/libp2p/rcmgr_defaults.go](https://github.com/ipfs/kubo/blob/master/core/node/libp2p/rcmgr_defaults.go)

### User Supplied Override Limits
A user who wants fine control over the limits used by the go-libp2p resource manager can specify overrides to the [computed default limits](#computed-default-limits).
This is done by defining limits in ``$IPFS_PATH/libp2p-resource-limit-overrides.json``.
These values trump anything else and are parsed directly by go-libp2p.
(See the [go-libp2p Resource Manager README](https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/README.md) for formatting.) 

## FAQ

### What do these "Protected from exceeding resource limits" log messages mean?
"Protected from exceeding resource limits" log messages denote that the resource manager is working and that it prevented additional resources being used beyond the set limits.  Per [libp2p code](https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/scope.go), these messages take the form of "$scope: cannot reserve $limitKey".  

As an example:

> Protected from exceeding resource limits 2 times: "system: cannot reserve inbound connection: resource limit exceeded"

This means that there were 2 recent occurrences where the libp2p resource manager prevented an inbound connection at the "system" [scope](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#resource-scopes).  
Specifically the ``System.ConnsInbound`` limit was hit.  

This can be analyzed by viewing the limit and current usage with `ipfs swarm resources`.
`System.ConnsInbound` is likely close or at the limit value.

The simplest way to identify all resources across all scopes that are close to exceeding their limit (>90% usage) is with a command like `ipfs swarm resources | egrep "9.\..%"` 

Sources:
* [kubo resource manager logging](https://github.com/ipfs/kubo/blob/master/core/node/libp2p/rcmgr_logging.go)
* [libp2p resource manager messages](https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/scope.go)

### How does one see the Active Limits?
A dump of what limits are actually being used by the resource manager ([Computed Default Limits](#computed-default-limits) + [User Supplied Override Limits](#user-supplied-override-limits))
can be obtained by `ipfs swarm resources`.

### How does one see the Computed Default Limits?
This can be observed [seeing the active limits](#how-does-one-see-the-active-limits) assuming one hasn't detoured into "power user" mode with [User Supplied Override Limits](#user-supplied-override-limits).

### How does one monitor libp2p resource usage?

For [monitoring libp2p resource usage](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#monitoring), 
various `*rcmgr_*` metrics can be accessed as the Prometheus endpoint at `{Addresses.API}/debug/metrics/prometheus` (default: `http://127.0.0.1:5001/debug/metrics/prometheus`).  
There are also [pre-built Grafana dashboards](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager/obs/grafana-dashboards) that can be added to a Grafana instance. 

A textual view of current resource usage and a list of services, protocols, and peers can be
obtained via `ipfs swarm stats --help`

### How does the resource manager (ResourceMgr) relate to the connection manager (ConnMgr)?
As discussed [here](https://github.com/libp2p/go-libp2p/tree/master/p2p/host/resource-manager#connmanager-vs-resource-manager)
these are separate systems in go-libp2p.
Kubo performs sanity checks to ensure that some of the hard limits of the ResourceMgr are sufficiently greater than the soft limits of the ConnMgr.

The soft limit of `Swarm.ConnMgr.HighWater` needs to be less than the resource manager hard limit `System.ConnsInbound` for the configuration to make sense.
This ensures the ConnMgr cleans up connections based on connection priorities before the hard limits of the ResourceMgr are applied.
If `Swarm.ConnMgr.HighWater` is greater than resource manager's `System.ConnsInbound`,
existing low priority idle connections can prevent new high priority connections from being established.
The ResourceMgr doesn't know that the new connection is high priority and simply blocks it because of the limit its enforcing.

To ensure the ConnMgr and ResourceMgr are congruent, the ResourceMgr [computed default limits](#computed-default-limits) are adjusted such that:
1. `System.ConnsInbound` >= `max(Swarm.ConnMgr.HighWater * 2, DefaultResourceMgrMinInboundConns)` AND
2. `System.StreamsInbound` is greater than any new/adjusted `Swarm.ResourceMgr.Limits.System.ConnsInbound` value so that there's enough streams per connection.

Source: [core/node/libp2p/rcmgr_defaults.go](https://github.com/ipfs/kubo/blob/master/core/node/libp2p/rcmgr_defaults.go)

### What are the "Application error 0x0 (remote) ... cannot reserve ..." messages?
These are messages coming from old (pre go-libp2p 0.26) *remote* go-libp2p peers (likely another older Kubo node) with the resource manager enabled on why it failed to establish a connection.  

This can be confusing, but these `Application error 0x0 (remote) ... cannot reserve ...` messages can occur even if your local node has the resource manager disabled.

You can distinguish resource manager messages originating from your local node if they're from the `resourcemanager` / `libp2p/rcmgr_logging.go` logger
or you see the string that is unique to Kubo (and not in go-libp2p): "Protected from exceeding resource limits".

See more info in this go-libp2p issue ([#1928](https://github.com/libp2p/go-libp2p/issues/1928)).  go-libp2p 0.26 / Kubo 0.19 onwards this confusing error message was removed.


## History
Kubo first [exposed this functionality in Kubo 0.13](./changelogs/v0.13.md#-libp2p-network-resource-manager-swarmresourcemgr), but it was disabled by default.  It was then enabled by default in [Kubo 0.17](./changelogs/v0.17.md#libp2p-resource-management-enabled-by-default).  Until that point, Kubo was vulnerable to unbound resource usage which could bring down nodes.  Introducing limits like this by default after the fact is tricky, which is why there have been changes and improvements afterwards.  The general trend since 0.17 with (0.18)[./changeloges/v0.18.md#improving-libp2p-resource-management-integration] and 0.19 has been to simplify and provide less options (and footguns!) for users and better documentation.


# Plugins

Since 0.4.11 Kubo has an experimental plugin system that allows augmenting
the daemons functionality without recompiling.

When an IPFS node is started, it will load plugins from the `$IPFS_PATH/plugins`
directory (by default `~/.ipfs/plugins`).

**Table of Contents**

- [Plugin Types](#plugin-types)
    - [IPLD](#ipld)
    - [Datastore](#datastore)
- [Available Plugins](#available-plugins)
- [Installing Plugins](#installing-plugins)
    - [External Plugin](#external-plugin)
        - [In-tree](#in-tree)
        - [Out-of-tree](#out-of-tree)
    - [Preloaded Plugins](#preloaded-plugins)
- [Creating A Plugin](#creating-a-plugin)

## Plugin Types

Plugins can implement one or more plugin types, defined in the
[plugin](https://godoc.org/github.com/ipfs/kubo/plugin) package.

### IPLD

IPLD plugins add support for additional formats to `ipfs dag` and other IPLD
related commands.

### Datastore

Datastore plugins add support for additional datastore backends.

### Tracer

(experimental)

Tracer plugins allow injecting an opentracing backend into Kubo.

### Daemon

Daemon plugins are started when the Kubo daemon is started and are given an
instance of the CoreAPI. This should make it possible to build an ipfs-based
application without IPC and without forking Kubo.

Note: We eventually plan to make Kubo usable as a library. However, this
plugin type is likely the best interim solution.

### fx (experimental)

Fx plugins let you customize the [fx](https://pkg.go.dev/go.uber.org/fx) dependency graph and configuration,
by customizing the`fx.Option`s that are passed to `fx` when the Kubo node is initialized.

For example, you can override an interface such as [exchange.Interface](https://github.com/ipfs/go-ipfs-exchange-interface)
or [pin.Pinner](https://github.com/ipfs/go-ipfs-pinner) with a custom implementation by appending an option like
`fx.Decorate(func() exchange.Interface { return customExchange })`.

Fx supports some advanced customization. Simple interface replacements like above are unlikely to break in the future, 
but the more invasive your changes, the more likely they are to break between releases. Kubo cannot guarantee backwards
compatibility for `fx` customizations.

Fx options are applied across every execution of the `ipfs` binary, including:

- Repo initialization
- Daemon
- Applying migrations
- etc.

So if you plug in a blockservice that disallows non-allowlisted CIDs, then this may break migrations
that fetch migration code over the IPFS network.

### Internal

(never stable)

Internal plugins are like daemon plugins _except_ that they can access, replace,
and modify all internal state. Use this plugin type to extend Kubo in
arbitrary ways. However, be aware that your plugin will likely break every time
Kubo updated.

## Configuration

Plugins can be configured in the `Plugins` section of the config file. Here,
plugins can be:

1. Passed an arbitrary config object via the `Config` field.
2. Disabled via the `Disabled` field.

Example:

```js
{
  // ...
  "Plugins": {
    "Plugins": {
      // plugin named "plugin-foo"
      "plugin-foo": {
        "Config": { /* arbitrary json */ }
      },
      // plugin named "plugin-bar"
      "plugin-bar": {
        "Disabled": true // "plugin-bar" will not be loaded
      }
    }
  }
}
```

## Available Plugins

| Name                                                                            | Type      | Preloaded | Description                                    |
|---------------------------------------------------------------------------------|-----------|-----------|------------------------------------------------|
| [git](https://github.com/ipfs/kubo/tree/master/plugin/plugins/git)           | IPLD      | x         | An IPLD format for git objects.                |
| [badgerds](https://github.com/ipfs/kubo/tree/master/plugin/plugins/badgerds) | Datastore | x         | A high performance but experimental datastore. |
| [flatfs](https://github.com/ipfs/kubo/tree/master/plugin/plugins/flatfs)     | Datastore | x         | A stable filesystem-based datastore.           |
| [levelds](https://github.com/ipfs/kubo/tree/master/plugin/plugins/levelds)   | Datastore | x         | A stable, flexible datastore backend.          |
| [jaeger](https://github.com/ipfs/go-jaeger-plugin)                              | Tracing   |           | An opentracing backend.                        |

* **Preloaded** plugins are built into the Kubo binary and do not need to be
  installed separately. At the moment, all in-tree plugins are preloaded.

## Installing Plugins

Kubo supports two types of plugins: External and Preloaded.

* External plugins must be installed in `$IPFS_PATH/plugins/` (usually
`~/.ipfs/plugins/`).
* Preloaded plugins are built-into the Kubo when it's compiled.

### External Plugin

The advantage of an external plugin is that it can be built, packaged, and
installed independently of Kubo. Unfortunately, this method is only supported
on Linux and MacOS at the moment. Users of other operating systems should follow
the instructions for preloaded plugins.

#### In-tree

To build plugins included in
[plugin/plugins](https://github.com/ipfs/kubo/tree/master/plugin/plugins),
run:

```bash
kubo$ make build_plugins
kubo$ ls plugin/plugins/*.so
```

To install, copy desired plugins to `$IPFS_PATH/plugins`. For example:

```bash
kubo$ mkdir -p ~/.ipfs/plugins/
kubo$ cp plugin/plugins/git.so ~/.ipfs/plugins/
kubo$ chmod +x ~/.ipfs/plugins/git.so # ensure plugin is executable
```

Finally, restart daemon if it is running.

#### Out-of-tree

To build out-of-tree plugins, use the plugin's Makefile if provided. Otherwise,
you can manually build the plugin by running:

```bash
myplugin$ go build -buildmode=plugin -o myplugin.so myplugin.go
```

Finally, as with in-tree plugins:

1. Install the plugin in `$IPFS_PATH/plugins`.
2. Mark the plugin as executable (`chmod +x $IPFS_PATH/plugins/myplugin.so`).
3. Restart your IPFS daemon (if running).

### Preloaded Plugins

The advantages of preloaded plugins are:

1. They're bundled with the Kubo binary.
2. They work on all platforms.

To preload a Kubo plugin:

1. Add the plugin to the preload list: `plugin/loader/preload_list`
2. Build ipfs
```bash
kubo$ make build
```

You can also preload an in-tree but disabled-by-default plugin by adding it to
the IPFS_PLUGINS variable. For example, to enable plugins foo, bar, and baz:

```bash
kubo$ make build IPFS_PLUGINS="foo bar baz"
```

## Creating A Plugin

To create your own out-of-tree plugin, use the [example
plugin](https://github.com/ipfs/go-ipfs-example-plugin/) as a starting point.
When you're ready, submit a PR adding it to the list of [available
plugins](#available-plugins).


# Developer Documentation and Guides

If you are looking for User Documentation & Guides, please visit [docs.ipfs.tech](https://docs.ipfs.tech/) or check [General Documentation](#general-documentation).

If you’re experiencing an issue with IPFS, **please follow [our issue guide](github-issue-guide.md) when filing an issue!**

Otherwise, check out the following guides to using and developing IPFS:

## General Documentation

- [Configuration reference](config.md)
    - [Datastore configuration](datastores.md)
    - [Experimental features](experimental-features.md)

## Developing `kubo`

- First, please read the Contributing Guidelines [for IPFS projects](https://github.com/ipfs/community/blob/master/CONTRIBUTING.md) and then the Contributing Guidelines for [Go code specifically](https://github.com/ipfs/community/blob/master/CONTRIBUTING_GO.md)
- Building on…
    - [Windows](windows.md)
- [Performance Debugging Guidelines](debug-guide.md)
- [Release Checklist](releases.md)

## Guides

- [How to Implement an API Client](implement-api-bindings.md)
- [Connecting with Websockets](transports.md) — if you want `js-ipfs` nodes in web browsers to connect to your `kubo` node, you will need to turn on websocket support in your `kubo` node.

## Advanced User Guides

- [Transferring a File Over IPFS](file-transfer.md)
- [Installing command completion](command-completion.md)
- [Mounting IPFS with FUSE](fuse.md)
- [Installing plugins](plugins.md)
- [Setting up an IPFS Gateway](https://github.com/ipfs/kubo/blob/master/docs/gateway.md)

## Other

- [Thanks to all our contributors ❤️](AUTHORS) (We use the `generate-authors.sh` script to regenerate this list.)
- [How to file a GitHub Issue](github-issue-guide.md)


# `kubo` Release Flow

# Table of Contents

- [`kubo` Release Flow](#kubo-release-flow)
- [Table of Contents](#table-of-contents)
  - [Release Philosophy](#release-philosophy)
  - [Release Flow](#release-flow)
    - [Stage 0 - Automated Testing](#stage-0---automated-testing)
    - [Stage 1 - Internal Testing](#stage-1---internal-testing)
    - [Stage 2 - Community Dev Testing](#stage-2---community-dev-testing)
    - [Stage 3 - Community Prod Testing](#stage-3---community-prod-testing)
    - [Stage 4 - Release](#stage-4---release)
  - [Release Cycle](#release-cycle)
    - [Patch Releases](#patch-releases)
  - [Security Fix Policy](#security-fix-policy)
  - [Performing a Release](#performing-a-release)
  - [Release Version Numbers (aka semver)](#release-version-numbers-aka-semver)
  - [_Footnotes_](#footnotes)

## Release Philosophy

`kubo` aims to have release every six weeks, two releases per quarter. During these 6 week releases, we go through 4 different stages that gives us the opportunity to test the new version against our test environments (unit, interop, integration), QA in our current production environment, IPFS apps (e.g. Desktop and WebUI) and with our community and _early testers_<sup>[1]</sup> that have IPFS running in production.

We might expand the six week release schedule in case of:

- No new updates to be added
- In case of a large community event that takes the core team availability away (e.g. IPFS Conf, Dev Meetings, IPFS Camp, etc.)

## Release Flow

`kubo` releases come in 5 stages designed to gradually roll out changes and reduce the impact of any regressions that may have been introduced. If we need to merge non-trivial<sup>[2]</sup> changes during the process, we start over at stage 0.

![kubo-release-process-illustration](https://user-images.githubusercontent.com/618519/62986422-653fee00-bdf0-11e9-8f61-197117b61da2.png)

### Stage 0 - Automated Testing

At this stage, we expect _all_ automated tests (interop, testlab, performance, etc.) to pass.

### Stage 1 - Internal Testing

At this stage, we'll:

1. Start a partial-rollout to our own infrastructure.
2. Test against ipfs and ipfs-shipyard applications.

**Goals:**

1. Make sure we haven't introduced any obvious regressions.
2. Test the release in an environment we can monitor and easily roll back (i.e. our own infra).

### Stage 2 - Community Dev Testing

At this stage, we'll announce the impending release to the community and ask for beta testers.

**Goal:**

Test the release in as many non-production environments as possible. This is relatively low-risk but gives us a _breadth_ of testing internal testing can't.

### Stage 3 - Community Prod Testing

At this stage, we consider the release to be "production ready" and will ask the community and our early testers to (partially) deploy the release to their production infrastructure.

**Goals:**

1. Test the release in some production environments with heavy workloads.
2. Partially roll-out an upgrade to see how it affects the network.
3. Retain the ability to ship last-minute fixes before the final release.

### Stage 4 - Release

At this stage, the release is "battle hardened" and ready for wide deployment.

## Release Cycle

A full release process should take about 3 weeks, a week per stage 1-3. We will start a new process every 6 weeks, regardless of when the previous release landed unless it's still ongoing.

### Patch Releases

If we encounter a serious bug in the stable latest release, we will create a patch release based on this release. For now, bug fixes will _not_ be backported to previous releases.

Patch releases will usually follow a compressed release cycle and should take 2-3 days. In a patch release:

1. Automated and internal testing (stage 0 and 1) will be compressed into a few hours - ideally less than a day.
2. Stage 2 will be skipped.
3. Community production testing will be shortened to 1-2 days of opt-in testing in production (early testers can choose to pass).

Some patch releases, especially ones fixing one or more complex bugs, may undergo the full release process.

## Security Fix Policy

Any release may contain security fixes. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP.

By policy, the team will usually wait until about 2 weeks after the final release to announce any fixed security issues. However, depending on the impact and ease of discovery of the issue, the team may wait more or less time. It is important to always update to the latest version ASAP and file issues if you're unable to update for some reason.

Finally, unless a security issue is actively being exploited or a significant number of users are unable to update to the latest version (e.g., due to a difficult migration, breaking changes, etc.), security fixes will _not_ be backported to previous releases.

## Performing a Release

The release is managed by the `Lead Maintainer` for `kubo`. It starts with the opening of an issue containing the content available on the [RELEASE_ISSUE_TEMPLATE](./RELEASE_ISSUE_TEMPLATE.md) not more than **48 hours** after the previous release.

This issue is pinned and labeled ["release"](https://github.com/ipfs/kubo/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3Arelease). When the cycle is due to begin the 5 stages will be followed until the release is done.

## Release Version Numbers (aka semver)

Until `kubo` 0.4.X, `kubo` was not using semver to communicate the type of release

Post `kubo` 0.5.X, `kubo` will use semver. This means that patch releases will not contain any breaking changes nor new features. Minor releases might contain breaking changes and always contain some new feature

Post `kubo` 1.X.X (future), `kubo` will use semver. This means that only major releases will contain breaking changes, minors will be reserved for new features and patches for bug fixes.

We do not yet retroactively apply fixes to older releases (no Long Term Support releases for now), which means that we always recommend users to update to the latest, whenever possible.

----------------------------

## _Footnotes_

- <sup>**[1]**</sup> - _early testers_ is an IPFS programme in which members of the community can self-volunteer to help test `kubo` Release Candidates. You find more info about it at [EARLY_TESTERS.md](./EARLY_TESTERS.md)
- <sup>**[2]**</sup> - A non-trivial change is any change that could potentially introduce an issue not trivially caught by automated testing. This is up to the discretion of the Lead Maintainer but the assumption is that every change is non-trivial unless proven otherwise.


# Testing Kubo releases with Thunderdome
This document is for running Thunderdome tests by release engineers as part of releasing Kubo.

We use Thunderdome to replay ipfs.io gateway traffic in a controlled environment against two different versions of Kubo, and we record metrics and compare them to look for logic or performance regressions before releasing a new Kubo version.

For background information about how Thunderdome works, see: https://github.com/ipfs-shipyard/thunderdome

## Prerequisites

* Ensure you have access to the "IPFS Stewards" vault in 1Password, which contains the requisite AWS Console and API credentials
* Ensure you have Docker and the Docker CLI installed
* Checkout the Thunderdome repo locally (or `git pull` to ensure it's up-to-date)
* Install AWS CLI v2: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* Configure the AWS CLI
  * Configure the credentials as described in the [Thunderdome documentation](https://github.com/ipfs-shipyard/thunderdome/blob/main/cmd/thunderdome/README.md#credentials), using the credentials from 1Password
* Make sure the `thunderdome` binary is up-to-date: `go build ./cmd/thunderdome`
  
## Add & run an experiment

Create a new release configuration JSON in the `experiments/` directory, based on the most recent `kubo-release` configuration, and tweak as necessary. Generally we setup the targets to run a commit against the tag of the last release, such as:

```json
	"targets": [
		{
			"name": "kubo190-4283b9",
			"description": "kubo 0.19.0-rc1",
			"build_from_git": {
				"repo": "https://github.com/ipfs/kubo.git",
				"commit":"4283b9d98f8438fc8751ccc840d8fc24eeae6f13"
			}
		},
		{
			"name": "kubo181",
			"description": "kubo 0.18.",
			"build_from_git": {
				"repo": "https://github.com/ipfs/kubo.git",
				"tag":"v0.18.1"
			}
		}
	]
```
  
Run the experiment (where `$EXPERIMENT_CONFIG_JSON` is a path to the config JSON created above):

```shell
AWS_PROFILE=thunderdome ./thunderdome deploy --verbose --duration 120 $EXPERIMENT_CONFIG_JSON
```

This will build the Docker images, upload them to ECR, and then launch the experiment in Thunderdome. Once the experiment starts, the CLI will exit and the experiment will continue to run for the duration.

## Analyze Results

Add a log entry in https://www.notion.so/pl-strflt/ce2d1bd56f3541028d960d3711465659 and link to it from the release issue, so that experiment results are publicly visible.

The `deploy` command will output a link to the Grafana dashboard for the experiment. We don't currently have rigorous acceptance criteria, so you should look for anomalies or changes in the metrics and make sure they are tolerable and explainable. Unexplainable anomalies should be noted in the log with a screenshot, and then root caused.


## Open a PR to merge the experiment config into Thunderdome

This is important for both posterity, and so that someone else can sanity-check the test parameters.


<!-- Last updated during [v0.18.0 release](https://github.com/ipfs/kubo/issues/9417) -->

# Items to do upon creating the release issue
- [ ] Fill in the Meta section
- [ ] Assign the issue to the release owner and reviewer.
- [ ] Name the issue "Release vX.Y.Z"
- [ ] Set the proper values for X.Y.Z
- [ ] Pin the issue

# Meta
* Release owner: @who
* Release reviewer: @who
* Expected RC date: week of YYYY-MM-DD
* 🚢 Expected final release date: YYYY-MM-DD
* Accompanying PR for improving the release process: (example: https://github.com/ipfs/kubo/pull/9391)

See the [Kubo release process](https://pl-strflt.notion.site/Kubo-Release-Process-5a5d066264704009a28a79cff93062c4) for more info.

# Kubo X.Y.Z Release

We're happy to announce Kubo X.Y.Z!

As usual, this release includes important fixes, some of which may be critical for security. Unless the fix addresses a bug being exploited in the wild, the fix will _not_ be called out in the release notes. Please make sure to update ASAP. See our [security fix policy](https://github.com/ipfs/go-ipfs/tree/master/docs/releases.md#security-fix-policy) for details.

## 🗺 What's left for release

<List of items with PRs and/or Issues to be considered for this release>

### Required

### Nice to have

## 🔦 Highlights

< top highlights for this release notes. For ANY version (final or RCs) >

## ✅ Release Checklist

### Labels

If an item should be executed for a specific release type, it should be labeled with one of the following labels:

- ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) execute **ONLY** when releasing a Release Candidate
- ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) execute **ONLY** when releasing a Final Release

Otherwise, it means it should be executed for **ALL** release types.

Patch releases should follow the same process as `.0` releases. If some item should **NOT** be executed for a Patch Release, it should be labeled with:

- ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) do **NOT** execute when releasing a Patch Release

### Before the release

This section covers tasks to be done ahead of the release.

- [ ] Verify you have access to all the services and tools required for the release
  - [ ] [GPG signature](https://docs.github.com/en/authentication/managing-commit-signature-verification) configured in local git and in GitHub
  - [ ] [admin access to IPFS Discourse](https://discuss.ipfs.tech/g/admins)
    - ask the previous release owner (or @2color) for an invite
  - [ ] ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) [access to #shared-pl-marketing-requests](https://filecoinproject.slack.com/archives/C018EJ8LWH1) channel in FIL Slack
    - ask the previous release owner for an invite
  - [ ] [access to IPFS network metrics](https://github.com/protocol/pldw/blob/624f47cf4ec14ad2cec6adf601a9f7b203ef770d/docs/sources/ipfs.md#ipfs-network-metrics) dashboards in Grafana
    - open an access request in the [pldw](https://github.com/protocol/pldw/issues/new/choose)
    - [example](https://github.com/protocol/pldw/issues/158)
  - [ ] [kuboreleaser](https://github.com/ipfs/kuboreleaser) checked out on your system (_only if you're using [kuboreleaser](https://github.com/ipfs/kuboreleaser)_)
  - [ ] [Thunderdome](https://github.com/ipfs-shipyard/thunderdome) checked out on your system and configured (see the [Thunderdome release docs](./releases_thunderdome.md) for setup)
  - [ ] [docker](https://docs.docker.com/get-docker/) installed on your system (_only if you're using [kuboreleaser](https://github.com/ipfs/kuboreleaser)_)
  - [ ] [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) installed on your system (_only if you're **NOT** using [kuboreleaser](https://github.com/ipfs/kuboreleaser)_)
  - [ ] [zsh](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH#install-and-set-up-zsh-as-default) installed on your system
  - [ ] [kubo](https://github.com/ipfs/kubo) checked out under `$(go env GOPATH)/src/github.com/ipfs/kubo`
    - you can also symlink your clone to the expected location by running `mkdir -p $(go env GOPATH)/src/github.com/ipfs && ln -s $(pwd) $(go env GOPATH)/src/github.com/ipfs/kubo`
  - [ ] ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) [Reddit](https://www.reddit.com) account
- ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Upgrade Go used in CI to the latest patch release available in [CircleCI](https://hub.docker.com/r/cimg/go/tags) in:
  - [ ] ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) [ipfs/distributions](https://github.com/ipfs/distributions)
    - [example](https://github.com/ipfs/distributions/pull/756)
  - [ ] ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) [ipfs/ipfs-docs](https://github.com/ipfs/ipfs-docs)
    - [example](https://github.com/ipfs/ipfs-docs/pull/1298)
- [ ] Verify there is nothing [left for release](-what-s-left-for-release)
- [ ] Create a release process improvement PR
  - [ ] update the [release issue template](docs/RELEASE_ISSUE_TEMPLATE.md) as you go
  - [ ] link it in the [Meta](#meta) section

### The release

This section covers tasks to be done during each release.

- [ ] Prepare the release branch and update version numbers accordingly <details><summary>using `./kuboreleaser --skip-check-before release --version vX.Y.Z(-rcN) prepare-branch` or ...</summary>
  - [ ] create a new branch `release-vX.Y.Z`
    - use `master` as base if `Z == 0`
    - use `release` as base if `Z > 0`
  - [ ] ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) update the `CurrentVersionNumber` in [version.go](version.go) in the `master` branch to `vX.Y+1.0-dev`
    - [example](https://github.com/ipfs/kubo/pull/9305)
  - [ ] update the `CurrentVersionNumber` in [version.go](version.go) in the `release-vX.Y` branch to `vX.Y.Z(-RCN)`
    - [example](https://github.com/ipfs/kubo/pull/9394)
  - [ ] create a draft PR from `release-vX.Y` to `release`
    - [example](https://github.com/ipfs/kubo/pull/9306)
  - [ ] Cherry-pick commits from `master` to the `release-vX.Y.Z` using `git cherry-pick -x <commit>`
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Add full changelog and contributors to the [changelog](docs/changelogs/vX.Y.md)
    - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Replace the `Changelog` and `Contributors` sections of the [changelog](docs/changelogs/vX.Y.md) with the stdout of `./bin/mkreleaselog`
      - do **NOT** copy the stderr
  - [ ] verify all CI checks on the PR from `release-vX.Y` to `release` are passing
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Merge the PR from `release-vX.Y` to `release` using the `Create a merge commit`
    - do **NOT** use `Squash and merge` nor `Rebase and merge` because we need to be able to sign the merge commit
    - do **NOT** delete the `release-vX.Y` branch
  </details>
- [ ] Run Thunderdome testing, see the [Thunderdome release docs](./releases_thunderdome.md) for details
  - [ ] create a PR and merge the experiment config into Thunderdome
- [ ] Create the release tag <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) tag` or ...</summary>
  - This is a dangerous operation! Go and Docker publishing are difficult to reverse! Have the release reviewer verify all the commands marked with ⚠️!
  - [ ] ⚠️ ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) tag the HEAD commit using `git tag -s vX.Y.Z(-RCN) -m 'Prerelease X.Y.Z(-RCN)'`
  - [ ] ⚠️ ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) tag the HEAD commit of the `release` branch using `git tag -s vX.Y.Z(-RCN) -m 'Release X.Y.Z(-RCN)'`
  - [ ] ⚠️ verify the tag is signed and tied to the correct commit using `git show vX.Y.Z(-RCN)`
  - [ ] ⚠️ push the tag to GitHub using `git push origin vX.Y.Z(-RCN)`
    - do **NOT** use `git push --tags` because it pushes all your local tags
  </details>
- [ ] Publish the release to [DockerHub](https://hub.docker.com/r/ipfs/kubo/) <details><summary>using `./kuboreleaser --skip-check-before --skip-run release --version vX.Y.Z(-rcN) publish-to-dockerhub` or ...</summary>
  - [ ] Wait for [Publish docker image](https://github.com/ipfs/kubo/actions/workflows/docker-image.yml) workflow run initiated by the tag push to finish
  - [ ] verify the image is available on [Docker Hub](https://hub.docker.com/r/ipfs/kubo/tags)
- [ ] Publish the release to [dist.ipfs.tech](https://dist.ipfs.tech) <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) publish-to-distributions` or ...</summary>
  - [ ] check out [ipfs/distributions](https://github.com/ipfs/distributions)
  - [ ] run `./dist.sh add-version kubo vX.Y.Z(-RCN)` to add the new version to the `versions` file
    - [usage](https://github.com/ipfs/distributions#usage)
  - [ ] create and merge the PR which updates `dists/kubo/versions` and `dists/go-ipfs/versions` (![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) and `dists/kubo/current_version` and `dists/go-ipfs/current_version`)
    - [example](https://github.com/ipfs/distributions/pull/760)
  - [ ] wait for the [CI](https://github.com/ipfs/distributions/actions/workflows/main.yml) workflow run initiated by the merge to master to finish
  - [ ] verify the release is available on [dist.ipfs.io](https://dist.ipfs.io/#kubo)
  </details>
- [ ] Publish the release to [NPM](https://www.npmjs.com/package/go-ipfs?activeTab=versions) <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) publish-to-npm` (⚠️ you might need to run the command a couple of times because GHA might not be able to see the new distribution straight away due to caching) or ...</summary>
  - [ ] run the [Release to npm](https://github.com/ipfs/npm-go-ipfs/actions/workflows/main.yml) workflow
  - [ ] check [Release to npm](https://github.com/ipfs/npm-go-ipfs/actions/workflows/main.yml) workflow run logs to verify it discovered the new release
  - [ ] verify the release is available on [NPM](https://www.npmjs.com/package/go-ipfs?activeTab=versions)
  </details>
- [ ] Publish the release to [GitHub](https://github.com/ipfs/kubo/releases) <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) publish-to-github` or ...</summary>
  - [ ] create a new release on [GitHub](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release)
    - [RC example](https://github.com/ipfs/kubo/releases/tag/v0.17.0-rc1)
    - [FINAL example](https://github.com/ipfs/kubo/releases/tag/v0.17.0)
    - [ ] use the `vX.Y.Z(-RCN)` tag
    - [ ] link to the release issue
    - [ ] ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) link to the changelog in the description
    - [ ] ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) check the `This is a pre-release` checkbox
    - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) copy the changelog (without the header) in the description
    - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) do **NOT** check the `This is a pre-release` checkbox
  - [ ] run the [sync-release-assets](https://github.com/ipfs/kubo/actions/workflows/sync-release-assets.yml) workflow
  - [ ] wait for the [sync-release-assets](https://github.com/ipfs/kubo/actions/workflows/sync-release-assets.yml) workflow run to finish
  - [ ] verify the release assets are present in the [GitHub release](https://github.com/ipfs/kubo/releases/tag/vX.Y.Z(-RCN))
  </details>
- [ ] Promote the release <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) promote` or ...</summary>
  - [ ] create an [IPFS Discourse](https://discuss.ipfs.tech) topic
    - [prerelease example](https://discuss.ipfs.tech/t/kubo-v0-16-0-rc1-release-candidate-is-out/15248)
    - [release example](https://discuss.ipfs.tech/t/kubo-v0-16-0-release-is-out/15249)
    - [ ] use `Kubo vX.Y.Z(-RCN) is out!` as the title
    - [ ] use `kubo` and `go-ipfs` as topics
    - [ ] repeat the title as a heading (`##`) in the description
    - [ ] link to the GitHub Release, binaries on IPNS, docker pull command and release notes in the description
  - [ ] pin the [IPFS Discourse](https://discuss.ipfs.tech) topic globally
    - you can make the topic a banner if there is no banner already
  - verify the [IPFS Discourse](https://discuss.ipfs.tech) topic was copied to:
    - [ ] [#ipfs-chatter](https://discord.com/channels/669268347736686612/669268347736686615) in IPFS Discord
    - [ ] [#ipfs-chatter](https://filecoinproject.slack.com/archives/C018EJ8LWH1) in FIL Slack
    - [ ] [#ipfs-chatter:ipfs.io](https://matrix.to/#/#ipfs-chatter:ipfs.io) in Matrix
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Add the link to the [IPFS Discourse](https://discuss.ipfs.tech) topic to the [GitHub Release](https://github.com/ipfs/kubo/releases/tag/vX.Y.Z(-RCN)) description
    - [example](https://github.com/ipfs/kubo/releases/tag/v0.17.0)
  - [ ] ![](https://img.shields.io/badge/only-RC-blue?style=flat-square) create an issue comment mentioning early testers on the release issue
    - [example](https://github.com/ipfs/kubo/issues/9319#issuecomment-1311002478)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) create an issue comment linking to the release on the release issue
    - [example](https://github.com/ipfs/kubo/issues/9417#issuecomment-1400740975)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) ask the marketing team to tweet about the release in [#shared-pl-marketing-requests](https://filecoinproject.slack.com/archives/C018EJ8LWH1) in FIL Slack
    - [example](https://filecoinproject.slack.com/archives/C018EJ8LWH1/p1664885305374900)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) post the link to the [GitHub Release](https://github.com/ipfs/kubo/releases/tag/vX.Y.Z(-RCN)) to [Reddit](https://reddit.com/r/ipfs)
    - [example](https://www.reddit.com/r/ipfs/comments/9x0q0k/kubo_v0160_release_is_out/)
  </details>
- [ ] Test the new version with `ipfs-companion` <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) test-ipfs-companion` or ...</summary>
  - [ ] run the [e2e](https://github.com/ipfs/ipfs-companion/actions/workflows/e2e.yml)
    - use `vX.Y.Z(-RCN)` as the Kubo image version
  - [ ] wait for the [e2e](https://github.com/ipfs/ipfs-companion/actions/workflows/e2e.yml) workflow run to finish
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Update Kubo in [ipfs-desktop](https://github.com/ipfs/ipfs-desktop) <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) update-ipfs-desktop` or ...</summary>
  - [ ] check out [ipfs/ipfs-desktop](https://github.com/ipfs/ipfs-desktop)
  - [ ] run `npm install`
  - [ ] create a PR which updates `package.json` and `package-lock.json`
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) add @SgtPooki and @whizzzkid as reviewers
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Update Kubo docs <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) update-ipfs-docs` or ...</summary>
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) run the [update-on-new-ipfs-tag.yml](https://github.com/ipfs/ipfs-docs/actions/workflows/update-on-new-ipfs-tag.yml) workflow
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) merge the PR created by the [update-on-new-ipfs-tag.yml](https://github.com/ipfs/ipfs-docs/actions/workflows/update-on-new-ipfs-tag.yml) workflow run
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Ask Brave to update Kubo in Brave Desktop
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) use [this link](https://github.com/brave/brave-browser/issues/new?assignees=&labels=OS%2FDesktop&projects=&template=desktop.md&title=) to create an issue for the new Kubo version
    - [basic example](https://github.com/brave/brave-browser/issues/31453), [example with additional notes](https://github.com/brave/brave-browser/issues/27965)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) post link to the issue in `#shared-pl-brave` for visibility
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Create a blog entry on [blog.ipfs.tech](https://blog.ipfs.tech) <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) update-ipfs-blog --date YYYY-MM-DD` or ...</summary>
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) create a PR which adds a release note for the new Kubo version
    - [example](https://github.com/ipfs/ipfs-blog/pull/529)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) merge the PR
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) verify the blog entry was published
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Merge the [release](https://github.com/ipfs/kubo/tree/release) branch back into [master](https://github.com/ipfs/kubo/tree/master), ignoring the changes to [version.go](version.go) (keep the `-dev`) version, <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) merge-branch` or ...</summary>
  - [ ] create a new branch `merge-release-vX.Y.Z` from `release`
  - [ ] create and merge a PR from `merge-release-vX.Y.Z` to `master`
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Prepare for the next release <details><summary>using `./kuboreleaser release --version vX.Y.Z(-rcN) prepare-next` or ...</summary>
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Create the next [changelog](https://github.com/ipfs/kubo/blob/master/docs/changelogs/vX.(Y+1).md)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Link to the new changelog in the [CHANGELOG.md](CHANGELOG.md) file
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Create the next release issue
  </details>
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) Create a dependency update PR
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) check out [ipfs/kubo](https://github.com/ipfs/kubo)
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) run `go get -u` in root directory
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) run `go mod tidy` in root directory
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) run `go mod tidy` in `docs/examples/kubo-as-a-library` directory
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) create a PR which updates `go.mod` and `go.sum`
  - [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) ![](https://img.shields.io/badge/not-PATCH-yellow?style=flat-square) add the PR to the next release milestone
- [ ] ![](https://img.shields.io/badge/only-FINAL-green?style=flat-square) Close the release issue

## How to contribute?

Would you like to contribute to the IPFS project and don't know how? Well, there are a few places you can get started:

- Check the issues with the `help wanted` label in the [ipfs/kubo repo](https://github.com/ipfs/kubo/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22)
- Join the discussion at [discuss.ipfs.tech](https://discuss.ipfs.tech/) and help users finding their answers.
- See other options at https://docs.ipfs.tech/community/


## /ws and /wss -- websockets

If you want browsers to connect to e.g. `/dns4/example.com/tcp/443/wss/p2p/QmFoo`

- [ ] An SSL cert matching the `/dns4` or `/dns6` name
- [ ] Kubo listening on `/ip4/127.0.0.1/tcp/8081/ws`
  - 8081 is just an example
  - note that it's `/ws` here, not `/wss` -- Kubo can't currently do SSL, see the next point
- [ ] nginx
  - configured with the SSL cert
  - listening on port 443
  - forwarding to 127.0.0.1:8081


# Building on Windows
![](https://ipfs.io/ipfs/QmccXW7JSZMVXidSc7tHsU6aktuaiV923q4yBGHUsdymYo/build.gif)

If you just want to install kubo, please download it from https://dist.ipfs.tech/#kubo. This document explains how to build it from source.

## Install Go
`kubo` is built on Golang and thus depends on it for all building methods.  
https://golang.org/doc/install  
The `GOPATH` environment variable must be set as well.  
https://golang.org/doc/code.html#GOPATH

## Choose the way you want to proceed
`kubo` utilizes `make` to automate builds and run tests, but can be built without it using only `git` and `go`.  
No matter which method you choose, if you encounter issues, please see the [Troubleshooting](#troubleshooting) section.  

**Using `make`:**  
MSYS2 and Cygwin provide the Unix tools we need to build `kubo`. You may use either, but if you don't already have one installed, we recommend MSYS2.  
[MSYS2→](#msys2)  
[Cygwin→](#cygwin)  

**Using build tools manually:**  
This section assumes you have a working version of `go` and `git` already setup. You may want to build this way if your environment restricts installing additional software, or if you're integrating IPFS into your own build system.  
[Minimal→](#minimal)  

## MSYS2
1. Install msys2 (http://www.msys2.org)  
2. Run the following inside a normal `cmd` prompt (Not the MSYS2 prompt, we only need MSYS2's tools).  
An explanation of this block is below.
```
SET PATH=%PATH%;\msys64\usr\bin
pacman --noconfirm -S  git make unzip
go get -u github.com/ipfs/kubo
cd %GOPATH%\src\github.com\ipfs\kubo
make install
%GOPATH%\bin\ipfs.exe version --all
```

If there were no errors, the final command should output version information similar to "`ipfs version 0.4.14-dev-XXXXXXX`" where "XXXXXXX" should match the current short-hash of the `kubo` repo. You can retrieve said hash via this command: `git rev-parse --short HEAD`.  
If `ipfs.exe` executes and the version string matches, then building was successful.

|Command|Explanation|
| ---: | :--- |
|`SET PATH=%PATH%;\msys64\usr\bin`         |Add msys2's tools to our [`PATH`](https://ss64.com/nt/path.html); Defaults to: (\msys64\usr\bin)|
|`pacman --noconfirm -S  git make unzip`   |Install `kubo` build dependencies|
|`go get -u github.com/ipfs/kubo`       |Fetch / Update `kubo` source|
|`cd %GOPATH%\src\github.com\ipfs\kubo` |Change to `kubo` source directory|
|`make install`                            |Build and install to `%GOPATH%\bin\ipfs.exe`|
|`%GOPATH%\bin\ipfs.exe version --all`     |Test the built binary|

To build again after making changes to the source, run:
```
SET PATH=%PATH%;\msys64\usr\bin
cd %GOPATH%\src\github.com\ipfs\kubo
make install
```

**Tip:** To avoid setting `PATH` every time (`SET PATH=%PATH%;\msys64\usr\bin`), you can lock it in permanently using `setx` after it's been set once:
```
SETX PATH %PATH%
```

## Cygwin
1. Install Cygwin (https://www.cygwin.com)  
2. During the install, select the following packages. (If you already have Cygwin installed, run the setup file again to install additional packages.) A fresh install should look something like [this reference image](https://ipfs.io/ipfs/QmaYFSQa4iHDafcebiKjm1WwuKhosoXr45HPpfaeMbCRpb/cygwin%20-%20install.png).
    - devel packages
        - `git`
        - `make`
    - archive packages
        - `unzip`
    - net packages
        - `curl`  
3. Run the following inside a normal `cmd` prompt (Not the Cygwin prompt, we only need Cygwin's tools)  
An explanation of this block is below.
```
SET PATH=%PATH%;\cygwin64\bin
mkdir %GOPATH%\src\github.com\ipfs
cd %GOPATH%\src\github.com\ipfs
git clone https://github.com/ipfs/kubo.git
cd %GOPATH%\src\github.com\ipfs\kubo
make install
%GOPATH%\bin\ipfs.exe version --all
```

If there were no errors, the final command should output version information similar  to "`ipfs version 0.4.14-dev-XXXXXXX`" where "XXXXXXX" should match the current short-hash of the `kubo` repo. You can retrieve said hash via this command: `git rev-parse --short HEAD`.  
If `ipfs.exe` executes and the version string matches, then building was successful.

|Command|Explanation|
| ---: | :--- |
|`SET PATH=%PATH%;\cygwin64\bin`           |Add Cygwin's tools to our [`PATH`](https://ss64.com/nt/path.html); Defaults to: (\cygwin64\bin)|
|`mkdir %GOPATH%\src\github.com\ipfs`<br/>`cd %GOPATH%\src\github.com\ipfs`<br/>`git clone https://github.com/ipfs/kubo.git`       |Fetch / Update `kubo` source|
|`cd %GOPATH%\src\github.com\ipfs\kubo` |Change to `kubo` source directory|
|`make install`                            |Build and install to `%GOPATH%\bin\ipfs.exe`|
|`%GOPATH%\bin\ipfs.exe version --all`     |Test the built binary|

To build again after making changes to the source, run:
```
SET PATH=%PATH%;\cygwin64\bin
cd %GOPATH%\src\github.com\ipfs\kubo
make install
```

**Tip:** To avoid setting `PATH` every time (`SET PATH=%PATH%;\cygwin64\bin`), you can lock it in permanently using `setx` after it's been set once:
```
SETX PATH %PATH%
```

## Minimal

While it's possible to build `kubo` with `go` alone, we'll be using `git` to fetch the source.

You can use whichever version of `git` you wish but we recommend the Windows builds at <https://git-scm.com>. `git` must be in your [`PATH`](https://ss64.com/nt/path.html) for `go get` to recognize and use it.

### kubo

Clone and change directory to the source code, if you haven't already:

CMD:
```bat
git clone https://github.com/ipfs/kubo %GOPATH%/src/github.com/ipfs/kubo
cd %GOPATH%/src/github.com/ipfs/kubo/cmd/ipfs
```

PowerShell:
```powershell
git clone https://github.com/ipfs/kubo $env:GOPATH/src/github.com/ipfs/kubo
cd $env:GOPATH/src/github.com/ipfs/kubo/cmd/ipfs
```

We need the `git` commit hash to be included in our build so that in the extremely rare event a bug is found, we have a reference point later for tracking it. We'll ask `git` for it and store it in a variable. The syntax for the next command is different depending on whether you're using the interactive command line or writing a batch file. Use the one that applies to you.  
- interactive: `FOR /F %V IN ('git rev-parse --short HEAD') do set SHA=%V`  
- interpreter: `FOR /F %%V IN ('git rev-parse --short HEAD') do set SHA=%%V`  

Finally, we'll build and test `ipfs` itself.

CMD:
```bat
go install -ldflags="-X "github.com/ipfs/kubo".CurrentCommit=%SHA%"
%GOPATH%\bin\ipfs.exe version --all
```

PowerShell:
```powershell
go install -ldflags="-X "github.com/ipfs/kubo".CurrentCommit=$env:SHA"
cp ./ipfs.exe $env:GOPATH/bin/ipfs.exe -force
. $env:GOPATH/bin/ipfs.exe version --all
```
You can check that the ipfs output versions match with `go version` and `git rev-parse --short HEAD`.  
If `ipfs.exe` executes and everything matches, then building was successful.

## Troubleshooting
- **Git auth**
If you get authentication problems with Git, you might want to take a look at https://help.github.com/articles/caching-your-github-password-in-git/ and use the suggested solution:  
`git config --global credential.helper wincred`

- **Anything else**  
Please search [https://discuss.ipfs.io](https://discuss.ipfs.io/search?q=windows%20category%3A13) for any additional issues you may encounter. If you can't find any existing resolution, feel free to post a question asking for help.

If you encounter a bug with `kubo` itself (not related to building) please use the [issue tracker](https://github.com/ipfs/kubo/issues) to report it.
