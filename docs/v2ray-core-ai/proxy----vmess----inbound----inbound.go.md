# `v2ray-core\proxy\vmess\inbound\inbound.go`

```
// +build !confonly

package inbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context" // 引入 context 包，用于控制程序的执行流程
    "io" // 引入 io 包，提供了基本的 I/O 接口
    "strings" // 引入 strings 包，提供了对字符串的操作
    "sync" // 引入 sync 包，提供了并发安全的锁和等待组

    "v2ray.com/core" // 引入 v2ray 核心包
    "v2ray.com/core/common" // 引入 v2ray 核心通用包
    "v2ray.com/core/common/buf" // 引入 v2ray 核心通用缓冲区包
    "v2ray.com/core/common/errors" // 引入 v2ray 核心通用错误处理包
    "v2ray.com/core/common/log" // 引入 v2ray 核心通用日志包
    "v2ray.com/core/common/net" // 引入 v2ray 核心通用网络包
    "v2ray.com/core/common/protocol" // 引入 v2ray 核心通用协议包
    "v2ray.com/core/common/session" // 引入 v2ray 核心通用会话包
    "v2ray.com/core/common/signal" // 引入 v2ray 核心通用信号包
    "v2ray.com/core/common/task" // 引入 v2ray 核心通用任务包
    "v2ray.com/core/common/uuid" // 引入 v2ray 核心通用 UUID 包
    feature_inbound "v2ray.com/core/features/inbound" // 引入 v2ray 核心入站特性包
    "v2ray.com/core/features/policy" // 引入 v2ray 核心策略包
    "v2ray.com/core/features/routing" // 引入 v2ray 核心路由包
    "v2ray.com/core/proxy/vmess" // 引入 v2ray VMess 代理包
    "v2ray.com/core/proxy/vmess/encoding" // 引入 v2ray VMess 编码包
    "v2ray.com/core/transport/internet" // 引入 v2ray 传输层网络包
)

// userByEmail 结构体，用于根据邮箱查找用户
type userByEmail struct {
    sync.Mutex // 互斥锁，用于保护并发访问
    cache           map[string]*protocol.MemoryUser // 缓存，存储邮箱和用户的映射关系
    defaultLevel    uint32 // 默认级别
    defaultAlterIDs uint16 // 默认 AlterID
}

// newUserByEmail 根据默认配置创建 userByEmail 对象
func newUserByEmail(config *DefaultConfig) *userByEmail {
    return &userByEmail{
        cache:           make(map[string]*protocol.MemoryUser), // 初始化缓存
        defaultLevel:    config.Level, // 设置默认级别
        defaultAlterIDs: uint16(config.AlterId), // 设置默认 AlterID
    }
}

// addNoLock 向缓存中添加用户，不加锁
func (v *userByEmail) addNoLock(u *protocol.MemoryUser) bool {
    email := strings.ToLower(u.Email) // 将邮箱转换为小写
    _, found := v.cache[email] // 检查缓存中是否已存在该邮箱
    if found {
        return false // 如果已存在，则返回 false
    }
    v.cache[email] = u // 否则将用户添加到缓存中
    return true // 返回 true
}

// Add 向缓存中添加用户
func (v *userByEmail) Add(u *protocol.MemoryUser) bool {
    v.Lock() // 加锁
    defer v.Unlock() // 延迟解锁

    return v.addNoLock(u) // 调用 addNoLock 方法添加用户
}

// Get 根据邮箱获取用户信息
func (v *userByEmail) Get(email string) (*protocol.MemoryUser, bool) {
    email = strings.ToLower(email) // 将邮箱转换为小写

    v.Lock() // 加锁
    defer v.Unlock() // 延迟解锁

    user, found := v.cache[email] // 从缓存中获取用户信息
    # 如果未找到用户
    if !found {
        # 生成一个新的 UUID 作为用户 ID
        id := uuid.New()
        # 创建一个 Vmess 账户对象，设置 ID 和 AlterId
        rawAccount := &vmess.Account{
            Id:      id.String(),
            AlterId: uint32(v.defaultAlterIDs),
        }
        # 将 Vmess 账户对象转换为通用账户对象
        account, err := rawAccount.AsAccount()
        # 必须处理错误
        common.Must(err)
        # 创建一个内存用户对象，设置用户等级、邮箱和账户信息
        user = &protocol.MemoryUser{
            Level:   v.defaultLevel,
            Email:   email,
            Account: account,
        }
        # 将用户信息缓存起来
        v.cache[email] = user
    }
    # 返回用户信息和是否找到用户的标志
    return user, found
// Remove removes the user with the specified email from the userByEmail cache
func (v *userByEmail) Remove(email string) bool {
    // Convert the email to lowercase
    email = strings.ToLower(email)

    // Lock the userByEmail cache to prevent concurrent access, and defer unlocking it until the function returns
    v.Lock()
    defer v.Unlock()

    // Check if the email exists in the cache, if not, return false
    if _, found := v.cache[email]; !found {
        return false
    }
    // Delete the user with the specified email from the cache and return true
    delete(v.cache, email)
    return true
}

// Handler is an inbound connection handler that handles messages in VMess protocol.
type Handler struct {
    policyManager         policy.Manager
    inboundHandlerManager feature_inbound.Manager
    clients               *vmess.TimedUserValidator
    usersByEmail          *userByEmail
    detours               *DetourConfig
    sessionHistory        *encoding.SessionHistory
    secure                bool
}

// New creates a new VMess inbound handler.
func New(ctx context.Context, config *Config) (*Handler, error) {
    // Get the core instance from the context
    v := core.MustFromContext(ctx)
    // Create a new Handler instance with the specified configuration
    handler := &Handler{
        policyManager:         v.GetFeature(policy.ManagerType()).(policy.Manager),
        inboundHandlerManager: v.GetFeature(feature_inbound.ManagerType()).(feature_inbound.Manager),
        clients:               vmess.NewTimedUserValidator(protocol.DefaultIDHash),
        detours:               config.Detour,
        usersByEmail:          newUserByEmail(config.GetDefaultValue()),
        sessionHistory:        encoding.NewSessionHistory(),
        secure:                config.SecureEncryptionOnly,
    }

    // Iterate through the users in the configuration and add them to the handler
    for _, user := range config.User {
        mUser, err := user.ToMemoryUser()
        if err != nil {
            return nil, newError("failed to get VMess user").Base(err)
        }

        if err := handler.AddUser(ctx, mUser); err != nil {
            return nil, newError("failed to initiate user").Base(err)
        }
    }

    return handler, nil
}

// Close implements common.Closable.
func (h *Handler) Close() error {
    // Close the clients, sessionHistory, and usersByEmail, and combine the errors if any
    return errors.Combine(
        h.clients.Close(),
        h.sessionHistory.Close(),
        common.Close(h.usersByEmail))
}

// Network implements proxy.Inbound.Network().
func (*Handler) Network() []net.Network {
    // Implement the Network method for the Handler type
}
    # 返回一个包含 net.Network_TCP 的网络列表
    return []net.Network{net.Network_TCP}
// 通过邮箱获取用户信息，如果用户存在则返回用户信息和 true，否则返回 nil 和 false
func (h *Handler) GetUser(email string) *protocol.MemoryUser {
    user, existing := h.usersByEmail.Get(email)
    // 如果用户不存在，则将用户添加到 clients 中
    if !existing {
        h.clients.Add(user)
    }
    // 返回用户信息
    return user
}

// 向用户列表中添加用户
func (h *Handler) AddUser(ctx context.Context, user *protocol.MemoryUser) error {
    // 如果用户的邮箱不为空且用户列表中已存在该用户，则返回错误信息
    if len(user.Email) > 0 && !h.usersByEmail.Add(user) {
        return newError("User ", user.Email, " already exists.")
    }
    // 将用户添加到 clients 中
    return h.clients.Add(user)
}

// 从用户列表中移除用户
func (h *Handler) RemoveUser(ctx context.Context, email string) error {
    // 如果邮箱为空，则返回错误信息
    if email == "" {
        return newError("Email must not be empty.")
    }
    // 如果用户列表中不存在该用户，则返回错误信息
    if !h.usersByEmail.Remove(email) {
        return newError("User ", email, " not found.")
    }
    // 从 clients 中移除用户
    h.clients.Remove(email)
    // 返回空值
    return nil
}

// 传输响应数据
func transferResponse(timer signal.ActivityUpdater, session *encoding.ServerSession, request *protocol.RequestHeader, response *protocol.ResponseHeader, input buf.Reader, output *buf.BufferedWriter) error {
    // 编码响应头部
    session.EncodeResponseHeader(response, output)

    // 编码响应体
    bodyWriter := session.EncodeResponseBody(request, output)

    {
        // 优化小型响应数据包
        data, err := input.ReadMultiBuffer()
        if err != nil {
            return err
        }

        // 写入响应体数据
        if err := bodyWriter.WriteMultiBuffer(data); err != nil {
            return err
        }
    }

    // 设置输出缓冲为非缓冲状态
    if err := output.SetBuffered(false); err != nil {
        return err
    }

    // 将输入数据拷贝到响应体中
    if err := buf.Copy(input, bodyWriter, buf.UpdateActivity(timer)); err != nil {
        return err
    }

    // 如果请求选项包含分块流，则写入空的多缓冲数据
    if request.Option.Has(protocol.RequestOptionChunkStream) {
        if err := bodyWriter.WriteMultiBuffer(buf.MultiBuffer{}); err != nil {
            return err
        }
    }

    // 返回空值
    return nil
}

// 判断加密类型是否为不安全的加密
func isInsecureEncryption(s protocol.SecurityType) bool {
    return s == protocol.SecurityType_NONE || s == protocol.SecurityType_LEGACY || s == protocol.SecurityType_UNKNOWN
}

// 实现 proxy.Inbound.Process() 方法
func (h *Handler) Process(ctx context.Context, network net.Network, connection internet.Connection, dispatcher routing.Dispatcher) error {
    // 获取会话策略
    sessionPolicy := h.policyManager.ForLevel(0)
    // 设置读取截止时间
    if err := connection.SetReadDeadline(time.Now().Add(sessionPolicy.Timeouts.Handshake)); err != nil {
        return newError("unable to set read deadline").Base(err).AtWarning()
    }

    // 创建读取器
    reader := &buf.BufferedReader{Reader: buf.NewReader(connection)}
    // 创建服务器会话
    svrSession := encoding.NewServerSession(h.clients, h.sessionHistory)
    // 解码请求头
    request, err := svrSession.DecodeRequestHeader(reader)
    if err != nil {
        // 处理解码错误
        if errors.Cause(err) != io.EOF {
            log.Record(&log.AccessMessage{
                From:   connection.RemoteAddr(),
                To:     "",
                Status: log.AccessRejected,
                Reason: err,
            })
            err = newError("invalid request from ", connection.RemoteAddr()).Base(err).AtInfo()
        }
        return err
    }

    // 检查是否使用不安全的加密
    if h.secure && isInsecureEncryption(request.Security) {
        log.Record(&log.AccessMessage{
            From:   connection.RemoteAddr(),
            To:     "",
            Status: log.AccessRejected,
            Reason: "Insecure encryption",
            Email:  request.User.Email,
        })
        return newError("client is using insecure encryption: ", request.Security)
    }

    // 检查请求命令是否为多路复用
    if request.Command != protocol.RequestCommandMux {
        // 更新上下文中的访问消息
        ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
            From:   connection.RemoteAddr(),
            To:     request.Destination(),
            Status: log.AccessAccepted,
            Reason: "",
            Email:  request.User.Email,
        })
    }

    // 记录接收到的请求
    newError("received request for ", request.Destination()).WriteToLog(session.ExportIDToError(ctx))

    // 设置读取截止时间为空
    if err := connection.SetReadDeadline(time.Time{}); err != nil {
        newError("unable to set back read deadline").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }
}
    # 从会话上下文中获取入站信息
    inbound := session.InboundFromContext(ctx)
    # 如果入站信息为空，则抛出异常
    if inbound == nil {
        panic("no inbound metadata")
    }
    # 将请求的用户信息赋给入站信息的用户字段
    inbound.User = request.User

    # 根据请求用户的级别获取会话策略
    sessionPolicy = h.policyManager.ForLevel(request.User.Level)

    # 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 根据会话策略中的连接空闲超时时间设置定时器
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)

    # 根据会话策略中的缓冲策略设置新的上下文
    ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)
    # 根据请求的目标地址分发请求，返回连接和可能的错误
    link, err := dispatcher.Dispatch(ctx, request.Destination())
    # 如果有错误发生，则返回包含错误信息的新错误
    if err != nil {
        return newError("failed to dispatch request to ", request.Destination()).Base(err)
    }

    # 定义处理请求完成的函数
    requestDone := func() error {
        # 在函数结束时设置定时器的下行数据传输超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        # 解码请求体并返回请求体的读取器
        bodyReader := svrSession.DecodeRequestBody(request, reader)
        # 将请求体数据从读取器复制到连接的写入器中，并更新定时器的活动时间
        if err := buf.Copy(bodyReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transfer request").Base(err)
        }
        return nil
    }

    # 定义处理响应完成的函数
    responseDone := func() error {
        # 在函数结束时设置定时器的上行数据传输超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        # 创建一个新的带缓冲的写入器
        writer := buf.NewBufferedWriter(buf.NewWriter(connection))
        defer writer.Flush()

        # 创建一个新的协议响应头，并返回传输响应的结果
        response := &protocol.ResponseHeader{
            Command: h.generateCommand(ctx, request),
        }
        return transferResponse(timer, svrSession, request, response, link.Reader, writer)
    }

    # 定义处理请求完成后的操作
    var requestDonePost = task.OnSuccess(requestDone, task.Close(link.Writer))
    # 运行请求完成后的操作和响应完成函数，并返回可能的错误
    if err := task.Run(ctx, requestDonePost, responseDone); err != nil {
        # 中断连接的读取和写入
        common.Interrupt(link.Reader)
        common.Interrupt(link.Writer)
        return newError("connection ends").Base(err)
    }

    # 返回空值
    return nil
}
// generateCommand 生成命令
func (h *Handler) generateCommand(ctx context.Context, request *protocol.RequestHeader) protocol.ResponseCommand {
    // 如果存在 detours
    if h.detours != nil {
        // 获取 detours 的标签
        tag := h.detours.To
        // 如果存在 inboundHandlerManager
        if h.inboundHandlerManager != nil {
            // 获取处理程序和错误
            handler, err := h.inboundHandlerManager.GetHandler(ctx, tag)
            // 如果出现错误
            if err != nil {
                // 记录错误日志并返回空
                newError("failed to get detour handler: ", tag).Base(err).AtWarning().WriteToLog(session.ExportIDToError(ctx))
                return nil
            }
            // 获取随机的入站代理处理程序、端口和可用时间
            proxyHandler, port, availableMin := handler.GetRandomInboundProxy()
            // 将代理处理程序转换为 Handler 类型
            inboundHandler, ok := proxyHandler.(*Handler)
            // 如果转换成功且 inboundHandler 不为空
            if ok && inboundHandler != nil {
                // 如果可用时间大于 255，则设置为 255
                if availableMin > 255 {
                    availableMin = 255
                }
                // 记录调试日志
                newError("pick detour handler for port ", port, " for ", availableMin, " minutes.").AtDebug().WriteToLog(session.ExportIDToError(ctx))
                // 获取用户
                user := inboundHandler.GetUser(request.User.Email)
                // 如果用户为空，返回空
                if user == nil {
                    return nil
                }
                // 获取用户账户
                account := user.Account.(*vmess.MemoryAccount)
                // 返回切换账户命令
                return &protocol.CommandSwitchAccount{
                    Port:     port,
                    ID:       account.ID.UUID(),
                    AlterIds: uint16(len(account.AlterIDs)),
                    Level:    user.Level,
                    ValidMin: byte(availableMin),
                }
            }
        }
    }
    // 返回空
    return nil
}

// 初始化函数
func init() {
    // 注册配置
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))
    }))
}
```