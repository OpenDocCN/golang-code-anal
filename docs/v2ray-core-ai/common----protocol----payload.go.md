# `v2ray-core\common\protocol\payload.go`

```
# 定义 TransferType 类型，表示传输类型，使用 byte 类型
type TransferType byte

# 定义 TransferType 类型的常量，表示流式传输
const (
    TransferTypeStream TransferType = 0
    # 定义 TransferType 类型的常量，表示分组传输
    TransferTypePacket TransferType = 1
)

# 定义 AddressType 类型，表示地址类型，使用 byte 类型
type AddressType byte

# 定义 AddressType 类型的常量，表示 IPv4 地址类型
const (
    AddressTypeIPv4   AddressType = 1
    # 定义 AddressType 类型的常量，表示域名地址类型
    AddressTypeDomain AddressType = 2
    # 定义 AddressType 类型的常量，表示 IPv6 地址类型
    AddressTypeIPv6   AddressType = 3
)
```