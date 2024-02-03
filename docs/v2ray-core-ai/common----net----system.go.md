# `v2ray-core\common\net\system.go`

```go
// 导入 net 包，用于网络通信
import "net"

// DialTCP 是 net.DialTCP 的别名
var DialTCP = net.DialTCP
var DialUDP = net.DialUDP
var DialUnix = net.DialUnix
var Dial = net.Dial

// ListenConfig 是 net.ListenConfig 的别名
type ListenConfig = net.ListenConfig

var Listen = net.Listen
var ListenTCP = net.ListenTCP
var ListenUDP = net.ListenUDP
var ListenUnix = net.ListenUnix

var LookupIP = net.LookupIP

var FileConn = net.FileConn

// ParseIP 是 net.ParseIP 的别名
var ParseIP = net.ParseIP

var SplitHostPort = net.SplitHostPort

var CIDRMask = net.CIDRMask

// Addr 是 net.Addr 的别名
type Addr = net.Addr
// Conn 是 net.Conn 的别名
type Conn = net.Conn
// PacketConn 是 net.PacketConn 的别名
type PacketConn = net.PacketConn

// TCPAddr 是 net.TCPAddr 的别名
type TCPAddr = net.TCPAddr
// TCPConn 是 net.TCPConn 的别名
type TCPConn = net.TCPConn

// UDPAddr 是 net.UDPAddr 的别名
type UDPAddr = net.UDPAddr
// UDPConn 是 net.UDPConn 的别名
type UDPConn = net.UDPConn

// UnixAddr 是 net.UnixAddr 的别名
type UnixAddr = net.UnixAddr
// UnixConn 是 net.UnixConn 的别名
type UnixConn = net.UnixConn

// IP 是 net.IP 的别名
type IP = net.IP
// IPMask 是 net.IPMask 的别名
type IPMask = net.IPMask
// IPNet 是 net.IPNet 的别名
type IPNet = net.IPNet

// IPv4len 是 net.IPv4len 的常量
const IPv4len = net.IPv4len
// IPv6len 是 net.IPv6len 的常量
const IPv6len = net.IPv6len

// Error 是 net.Error 的别名
type Error = net.Error
// AddrError 是 net.AddrError 的别名
type AddrError = net.AddrError

// Dialer 是 net.Dialer 的别名
type Dialer = net.Dialer
// Listener 是 net.Listener 的别名
type Listener = net.Listener
// TCPListener 是 net.TCPListener 的别名
type TCPListener = net.TCPListener
// UnixListener 是 net.UnixListener 的别名
type UnixListener = net.UnixListener

var ResolveUnixAddr = net.ResolveUnixAddr
var ResolveUDPAddr = net.ResolveUDPAddr

// Resolver 是 net.Resolver 的别名
type Resolver = net.Resolver
```