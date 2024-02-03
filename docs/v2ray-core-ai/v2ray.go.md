# `v2ray-core\v2ray.go`

```go
// +build !confonly

package core

import (
    "context"  // 导入 context 包，用于处理上下文
    "reflect"  // 导入 reflect 包，用于实现反射功能
    "sync"  // 导入 sync 包，用于实现同步功能

    "v2ray.com/core/common"  // 导入 common 包
    "v2ray.com/core/common/serial"  // 导入 serial 包
    "v2ray.com/core/features"  // 导入 features 包
    "v2ray.com/core/features/dns"  // 导入 dns 包
    "v2ray.com/core/features/dns/localdns"  // 导入 localdns 包
    "v2ray.com/core/features/inbound"  // 导入 inbound 包
    "v2ray.com/core/features/outbound"  // 导入 outbound 包
    "v2ray.com/core/features/policy"  // 导入 policy 包
    "v2ray.com/core/features/routing"  // 导入 routing 包
    "v2ray.com/core/features/stats"  // 导入 stats 包
)

// Server is an instance of V2Ray. At any time, there must be at most one Server instance running.
type Server interface {
    common.Runnable  // 定义 Server 接口，包含 Runnable 方法
}

// ServerType returns the type of the server.
func ServerType() interface{} {
    return (*Instance)(nil)  // 返回 Server 实例的类型
}

type resolution struct {
    deps     []reflect.Type  // 定义 resolution 结构体，包含 deps 字段，类型为 reflect.Type 的切片
    callback interface{}  // 定义 resolution 结构体，包含 callback 字段，类型为 interface{}
}

func getFeature(allFeatures []features.Feature, t reflect.Type) features.Feature {
    for _, f := range allFeatures {
        if reflect.TypeOf(f.Type()) == t {  // 遍历 allFeatures，比较类型是否相等
            return f  // 返回匹配的 Feature
        }
    }
    return nil  // 如果没有匹配的 Feature，则返回 nil
}

func (r *resolution) resolve(allFeatures []features.Feature) (bool, error) {
    var fs []features.Feature  // 定义 fs 变量，类型为 Feature 的切片
    for _, d := range r.deps {  // 遍历 resolution 结构体的 deps 字段
        f := getFeature(allFeatures, d)  // 调用 getFeature 方法获取对应的 Feature
        if f == nil {  // 如果获取的 Feature 为 nil
            return false, nil  // 返回 false 和 nil
        }
        fs = append(fs, f)  // 将获取的 Feature 添加到 fs 切片中
    }

    callback := reflect.ValueOf(r.callback)  // 使用反射获取 callback 的值
    var input []reflect.Value  // 定义 input 变量，类型为 reflect.Value 的切片
    callbackType := callback.Type()  // 获取 callback 的类型
    for i := 0; i < callbackType.NumIn(); i++ {  // 遍历 callback 的输入参数
        pt := callbackType.In(i)  // 获取第 i 个输入参数的类型
        for _, f := range fs {  // 遍历 fs 切片
            if reflect.TypeOf(f).AssignableTo(pt) {  // 判断 f 是否可以赋值给 pt
                input = append(input, reflect.ValueOf(f))  // 如果可以赋值，则将 f 添加到 input 中
                break
            }
        }
    }

    if len(input) != callbackType.NumIn() {  // 如果 input 的长度不等于 callback 的输入参数个数
        panic("Can't get all input parameters")  // 抛出 panic 异常
    }

    var err error  // 定义 err 变量，类型为 error
    ret := callback.Call(input)  // 调用 callback 方法，并传入 input 参数，获取返回值
    errInterface := reflect.TypeOf((*error)(nil)).Elem()  // 获取 error 接口的类型
    # 从返回值列表的最后一个元素开始向前遍历
    for i := len(ret) - 1; i >= 0; i-- {
        # 检查当前元素的类型是否为错误接口类型
        if ret[i].Type() == errInterface {
            # 获取接口的值
            v := ret[i].Interface()
            # 如果接口的值不为空，则将其转换为错误类型，并赋值给err变量
            if v != nil {
                err = v.(error)
            }
            # 中断循环
            break
        }
    }

    # 返回true和err变量
    return true, err
// Instance 结构体包含了 V2Ray 中所有功能的组合。
type Instance struct {
    access             sync.Mutex  // 用于控制对 Instance 结构体的访问
    features           []features.Feature  // 存储功能列表
    featureResolutions []resolution  // 存储功能分辨率列表
    running            bool  // 标识实例是否正在运行

    ctx context.Context  // 上下文对象
}

// AddInboundHandler 向服务器实例中添加入站处理程序
func AddInboundHandler(server *Instance, config *InboundHandlerConfig) error {
    // 获取入站管理器
    inboundManager := server.GetFeature(inbound.ManagerType()).(inbound.Manager)
    // 创建原始处理程序对象
    rawHandler, err := CreateObject(server, config)
    if err != nil {
        return err
    }
    // 将原始处理程序对象转换为入站处理程序对象
    handler, ok := rawHandler.(inbound.Handler)
    if !ok {
        return newError("not an InboundHandler")
    }
    // 向入站管理器中添加处理程序
    if err := inboundManager.AddHandler(server.ctx, handler); err != nil {
        return err
    }
    return nil
}

// addInboundHandlers 向服务器实例中添加多个入站处理程序
func addInboundHandlers(server *Instance, configs []*InboundHandlerConfig) error {
    for _, inboundConfig := range configs {
        if err := AddInboundHandler(server, inboundConfig); err != nil {
            return err
        }
    }

    return nil
}

// AddOutboundHandler 向服务器实例中添加出站处理程序
func AddOutboundHandler(server *Instance, config *OutboundHandlerConfig) error {
    // 获取出站管理器
    outboundManager := server.GetFeature(outbound.ManagerType()).(outbound.Manager)
    // 创建原始处理程序对象
    rawHandler, err := CreateObject(server, config)
    if err != nil {
        return err
    }
    // 将原始处理程序对象转换为出站处理程序对象
    handler, ok := rawHandler.(outbound.Handler)
    if !ok {
        return newError("not an OutboundHandler")
    }
    // 向出站管理器中添加处理程序
    if err := outboundManager.AddHandler(server.ctx, handler); err != nil {
        return err
    }
    return nil
}

// addOutboundHandlers 向服务器实例中添加多个出站处理程序
func addOutboundHandlers(server *Instance, configs []*OutboundHandlerConfig) error {
    for _, outboundConfig := range configs {
        if err := AddOutboundHandler(server, outboundConfig); err != nil {
            return err
        }
    }

    return nil
}

// RequireFeatures 是一个辅助函数，用于在上下文中要求从 Instance 中获取功能。
// 有关更多信息，请参阅 Instance.RequireFeatures。
func RequireFeatures(ctx context.Context, callback interface{}) error {
    v := MustFromContext(ctx)  // 从上下文中获取实例
    # 调用对象 v 的 RequireFeatures 方法，并传入参数 callback，返回结果
    return v.RequireFeatures(callback)
// New函数基于给定的配置返回一个新的V2Ray实例。
// 此时实例尚未启动。
// 为确保V2Ray实例正常工作，配置必须包含一个Dispatcher、一个InboundHandlerManager和一个OutboundHandlerManager。其他功能是可选的。
func New(config *Config) (*Instance, error) {
    // 创建一个新的V2Ray实例，使用默认的上下文
    var server = &Instance{ctx: context.Background()}

    // 使用配置初始化V2Ray实例
    err, done := initInstanceWithConfig(config, server)
    // 如果初始化完成，则返回错误
    if done {
        return nil, err
    }

    // 返回初始化后的V2Ray实例
    return server, nil
}

// NewWithContext函数基于给定的配置和上下文返回一个新的V2Ray实例。
func NewWithContext(config *Config, ctx context.Context) (*Instance, error) {
    // 创建一个新的V2Ray实例，使用给定的上下文
    var server = &Instance{ctx: ctx}

    // 使用配置初始化V2Ray实例
    err, done := initInstanceWithConfig(config, server)
    // 如果初始化完成，则返回错误
    if done {
        return nil, err
    }

    // 返回初始化后的V2Ray实例
    return server, nil
}

// 使用配置初始化V2Ray实例
func initInstanceWithConfig(config *Config, server *Instance) (error, bool) {
    // 如果配置中包含传输设置，则打印已弃用的功能警告
    if config.Transport != nil {
        features.PrintDeprecatedFeatureWarning("global transport settings")
    }
    // 应用传输设置，如果出错则返回错误
    if err := config.Transport.Apply(); err != nil {
        return err, true
    }

    // 遍历配置中的应用设置
    for _, appSettings := range config.App {
        // 获取应用实例
        settings, err := appSettings.GetInstance()
        // 如果获取实例出错，则返回错误
        if err != nil {
            return err, true
        }
        // 创建对象并返回错误
        obj, err := CreateObject(server, settings)
        if err != nil {
            return err, true
        }
        // 如果对象是特性，则将其添加到V2Ray实例中
        if feature, ok := obj.(features.Feature); ok {
            if err := server.AddFeature(feature); err != nil {
                return err, true
            }
        }
    }

    // 初始化必要的特性
    essentialFeatures := []struct {
        Type     interface{}
        Instance features.Feature
    }{
        {dns.ClientType(), localdns.New()},
        {policy.ManagerType(), policy.DefaultManager{}},
        {routing.RouterType(), routing.DefaultRouter{}},
        {stats.ManagerType(), stats.NoopManager{}},
    }
    # 遍历必要特性列表
    for _, f := range essentialFeatures:
        # 如果服务器没有该特性，则添加该特性实例
        if server.GetFeature(f.Type) == nil:
            if err := server.AddFeature(f.Instance); err != nil:
                # 如果添加特性实例失败，则返回错误和 true
                return err, true

    # 如果服务器的特性解析不为空，则返回错误和 true
    if server.featureResolutions != nil:
        return newError("not all dependency are resolved."), true

    # 添加入站处理程序到服务器
    if err := addInboundHandlers(server, config.Inbound); err != nil:
        # 如果添加入站处理程序失败，则返回错误和 true
        return err, true

    # 添加出站处理程序到服务器
    if err := addOutboundHandlers(server, config.Outbound); err != nil:
        # 如果添加出站处理程序失败，则返回错误和 true
        return err, true

    # 如果以上都没有错误，则返回空和 false
    return nil, false
// Type 方法实现了 common.HasType 接口，返回 ServerType 类型
func (s *Instance) Type() interface{} {
    return ServerType()
}

// Close 方法关闭 V2Ray 实例
func (s *Instance) Close() error {
    // 加锁
    s.access.Lock()
    // 延迟解锁
    defer s.access.Unlock()

    // 将 running 状态设置为 false
    s.running = false

    // 初始化错误列表
    var errors []interface{}
    // 遍历 features 列表，关闭每个 feature，并记录错误
    for _, f := range s.features {
        if err := f.Close(); err != nil {
            errors = append(errors, err)
        }
    }
    // 如果有错误发生，返回包含错误信息的新错误
    if len(errors) > 0 {
        return newError("failed to close all features").Base(newError(serial.Concat(errors...)))
    }

    return nil
}

// RequireFeatures 方法注册一个回调函数，在所有依赖的 feature 都注册后调用
// 回调函数必须是 func() 类型，其参数必须是 features.Feature 类型
func (s *Instance) RequireFeatures(callback interface{}) error {
    // 获取回调函数的类型
    callbackType := reflect.TypeOf(callback)
    // 如果回调函数不是函数类型，抛出异常
    if callbackType.Kind() != reflect.Func {
        panic("not a function")
    }

    // 初始化 feature 类型列表
    var featureTypes []reflect.Type
    // 遍历回调函数的参数类型，将其指针类型添加到 featureTypes 中
    for i := 0; i < callbackType.NumIn(); i++ {
        featureTypes = append(featureTypes, reflect.PtrTo(callbackType.In(i)))
    }

    // 创建 resolution 对象，包含依赖的 feature 类型列表和回调函数
    r := resolution{
        deps:     featureTypes,
        callback: callback,
    }
    // 如果已经解析完成，返回错误
    if finished, err := r.resolve(s.features); finished {
        return err
    }
    // 将 resolution 对象添加到 featureResolutions 列表中
    s.featureResolutions = append(s.featureResolutions, r)
    return nil
}

// AddFeature 方法将一个 feature 注册到当前 Instance 中
func (s *Instance) AddFeature(feature features.Feature) error {
    // 将 feature 添加到 features 列表中
    s.features = append(s.features, feature)

    // 如果实例正在运行，启动 feature，并记录启动错误
    if s.running {
        if err := feature.Start(); err != nil {
            newError("failed to start feature").Base(err).WriteToLog()
        }
        return nil
    }

    // 如果 featureResolutions 为空，直接返回
    if s.featureResolutions == nil {
        return nil
    }

    // 初始化待处理的 resolution 列表
    var pendingResolutions []resolution
    # 遍历特征分辨率列表
    for _, r := range s.featureResolutions {
        # 解析特征，返回是否完成和错误信息
        finished, err := r.resolve(s.features)
        # 如果解析完成且有错误，则返回错误
        if finished && err != nil {
            return err
        }
        # 如果未完成，则将该解析添加到待处理解析列表中
        if !finished {
            pendingResolutions = append(pendingResolutions, r)
        }
    }
    # 如果待处理解析列表长度为0，则将特征分辨率列表置空
    if len(pendingResolutions) == 0 {
        s.featureResolutions = nil
    } 
    # 如果待处理解析列表长度小于特征分辨率列表长度，则更新特征分辨率列表为待处理解析列表
    else if len(pendingResolutions) < len(s.featureResolutions) {
        s.featureResolutions = pendingResolutions
    }

    # 返回空
    return nil
// GetFeature函数返回给定类型的特性，如果未注册该特性，则返回nil。
func (s *Instance) GetFeature(featureType interface{}) features.Feature {
    return getFeature(s.features, reflect.TypeOf(featureType))
}

// Start函数启动V2Ray实例，包括所有已注册的特性。当Start返回错误时，实例的状态是未知的。
// V2Ray实例只能启动一次。关闭后，不能保证实例能再次启动。
//
// v2ray:api:stable
func (s *Instance) Start() error {
    s.access.Lock()  // 获取实例的互斥锁
    defer s.access.Unlock()  // 在函数返回时释放互斥锁

    s.running = true  // 设置实例的运行状态为true
    for _, f := range s.features {  // 遍历实例的特性
        if err := f.Start(); err != nil {  // 如果特性启动失败
            return err  // 返回错误
        }
    }

    newError("V2Ray ", Version(), " started").AtWarning().WriteToLog()  // 记录V2Ray实例启动的日志

    return nil  // 返回nil表示启动成功
}
```