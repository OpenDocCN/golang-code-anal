# `kubo\config\remotepin.go`

```
package config

// 远程服务路径
var (
    RemoteServicesPath     = "Pinning.RemoteServices"
    // 遮蔽选择器
    PinningConcealSelector = []string{"Pinning", "RemoteServices", "*", "API", "Key"}
)

// Pinning 结构体
type Pinning struct {
    RemoteServices map[string]RemotePinningService
}

// 远程固定服务结构体
type RemotePinningService struct {
    API      RemotePinningServiceAPI
    Policies RemotePinningServicePolicies
}

// 远程固定服务 API 结构体
type RemotePinningServiceAPI struct {
    Endpoint string
    Key      string
}

// 远程固定服务策略结构体
type RemotePinningServicePolicies struct {
    MFS RemotePinningServiceMFSPolicy
}

// 远程固定服务 MFS 策略结构体
type RemotePinningServiceMFSPolicy struct {
    // 启用 MFS 更改监视并在更改发生时重新固定 MFS 根 CID。
    Enable bool
    // MFS 的固定名称。
    PinName string
    // 当策略启用时，确定重新固定间隔。以 ns、us、ms、s、m、h 为单位。
    RepinInterval string
}
```