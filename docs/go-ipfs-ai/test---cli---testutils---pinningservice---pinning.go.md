# `kubo\test\cli\testutils\pinningservice\pinning.go`

```go
package pinningservice

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "net/http"  // 导入 HTTP 包
    "reflect"  // 导入反射包
    "strconv"  // 导入字符串转换包
    "strings"  // 导入字符串处理包
    "sync"  // 导入同步包
    "time"  // 导入时间包

    "github.com/google/uuid"  // 导入 UUID 包
    "github.com/julienschmidt/httprouter"  // 导入路由包
)

// NewRouter creates a new HTTP router with the specified authentication token and PinningService
func NewRouter(authToken string, svc *PinningService) http.Handler {
    router := httprouter.New()  // 创建一个新的路由器
    router.GET("/api/v1/pins", svc.listPins)  // 注册 GET 请求处理函数
    router.POST("/api/v1/pins", svc.addPin)  // 注册 POST 请求处理函数
    router.GET("/api/v1/pins/:requestID", svc.getPin)  // 注册 GET 请求处理函数
    router.POST("/api/v1/pins/:requestID", svc.replacePin)  // 注册 POST 请求处理函数
    router.DELETE("/api/v1/pins/:requestID", svc.removePin)  // 注册 DELETE 请求处理函数

    handler := authHandler(authToken, router)  // 创建一个带有认证的处理器

    return handler  // 返回处理器
}

// authHandler is a middleware that performs authentication before delegating to the provided handler
func authHandler(authToken string, delegate http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authz := r.Header.Get("Authorization")  // 获取请求头中的 Authorization 字段
        if !strings.HasPrefix(authz, "Bearer ") {  // 检查 Authorization 字段是否以 "Bearer " 开头
            errResp(w, "invalid authorization token, must start with 'Bearer '", "", http.StatusBadRequest)  // 返回错误响应
            return
        }

        token := strings.TrimPrefix(authz, "Bearer ")  // 去掉 "Bearer " 前缀，获取 token
        if token != authToken {  // 检查 token 是否与预期的认证 token 相符
            errResp(w, "access denied", "", http.StatusUnauthorized)  // 返回拒绝访问的错误响应
            return
        }

        delegate.ServeHTTP(w, r)  // 调用委托的处理器
    })
}

// New creates a new PinningService instance with default settings
func New() *PinningService {
    return &PinningService{
        PinAdded: func(*AddPinRequest, *PinStatus) {},  // 设置默认的 PinAdded 回调函数
    }
}

// PinningService is a basic pinning service that implements the Remote Pinning API, for testing Kubo's integration with remote pinning services.
// Pins are not persisted, they are just kept in-memory, and this provides callbacks for controlling the behavior of the pinning service.
type PinningService struct {
    m sync.Mutex  // 互斥锁
    // PinAdded is a callback that is invoked after a new pin is added via the API.
    PinAdded func(*AddPinRequest, *PinStatus)  // 添加新 pin 时的回调函数
    pins     []*PinStatus  // pin 列表
}

type Pin struct {
    CID     string                 `json:"cid"`  // pin 的 CID
    Name    string                 `json:"name"`  // pin 的名称
}
    # 定义一个字符串数组类型的字段 Origins，用于存储来源信息，将被转换为 JSON 时使用 "origins" 作为键
    Origins []string               `json:"origins"`
    # 定义一个键为字符串类型，值为接口类型的 map 类型的字段 Meta，用于存储元数据信息，将被转换为 JSON 时使用 "meta" 作为键
    Meta    map[string]interface{} `json:"meta"`
}

type PinStatus struct {
    M         sync.Mutex
    RequestID string
    Status    string
    Created   time.Time
    Pin       Pin
    Delegates []string
    Info      map[string]interface{}
}

func (p *PinStatus) MarshalJSON() ([]byte, error) {
    type pinStatusJSON struct {
        RequestID string                 `json:"requestid"`  // 定义结构体字段的 JSON 标签
        Status    string                 `json:"status"`     // 定义结构体字段的 JSON 标签
        Created   time.Time              `json:"created"`    // 定义结构体字段的 JSON 标签
        Pin       Pin                    `json:"pin"`        // 定义结构体字段的 JSON 标签
        Delegates []string               `json:"delegates"`  // 定义结构体字段的 JSON 标签
        Info      map[string]interface{} `json:"info"`       // 定义结构体字段的 JSON 标签
    }
    // 在进行序列化之前，对 pin 进行加锁以防止数据竞争
    p.M.Lock()
    pinJSON := pinStatusJSON{
        RequestID: p.RequestID,
        Status:    p.Status,
        Created:   p.Created,
        Pin:       p.Pin,
        Delegates: p.Delegates,
        Info:      p.Info,
    }
    // 在序列化完成后解锁
    p.M.Unlock()
    return json.Marshal(pinJSON)
}

func (p *PinStatus) Clone() PinStatus {
    return PinStatus{
        RequestID: p.RequestID,
        Status:    p.Status,
        Created:   p.Created,
        Pin:       p.Pin,
        Delegates: p.Delegates,
        Info:      p.Info,
    }
}

const (
    matchExact    = "exact"    // 定义常量 matchExact
    matchIExact   = "iexact"   // 定义常量 matchIExact
    matchPartial  = "partial"  // 定义常量 matchPartial
    matchIPartial = "ipartial" // 定义常量 matchIPartial

    statusQueued  = "queued"   // 定义常量 statusQueued
    statusPinning = "pinning"  // 定义常量 statusPinning
    statusPinned  = "pinned"   // 定义常量 statusPinned
    statusFailed  = "failed"   // 定义常量 statusFailed

    timeLayout = "2006-01-02T15:04:05.999Z"  // 定义时间格式的常量
)

func errResp(w http.ResponseWriter, reason, details string, statusCode int) {
    type errorObj struct {
        Reason  string `json:"reason"`   // 定义结构体字段的 JSON 标签
        Details string `json:"details"`  // 定义结构体字段的 JSON 标签
    }
    type errorResp struct {
        Error errorObj `json:"error"`  // 定义结构体字段的 JSON 标签
    }
    resp := errorResp{
        Error: errorObj{
            Reason:  reason,
            Details: details,
        },
    }
    writeJSON(w, resp, statusCode)
}
# 写入 JSON 数据到 HTTP 响应中
func writeJSON(w http.ResponseWriter, val any, statusCode int) {
    # 将值转换为 JSON 格式的字节流
    b, err := json.Marshal(val)
    if err != nil {
        # 如果转换出错，设置响应头为纯文本类型，返回错误信息
        w.Header().Set("Content-Type", "text/plain")
        errResp(w, fmt.Sprintf("marshaling response: %s", err), "", http.StatusInternalServerError)
        return
    }
    # 设置响应头为 JSON 类型，设置状态码
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)
    # 将 JSON 数据写入响应
    _, _ = w.Write(b)
}

# 定义添加 Pin 请求的数据结构
type AddPinRequest struct {
    CID     string                 `json:"cid"`
    Name    string                 `json:"name"`
    Origins []string               `json:"origins"`
    Meta    map[string]interface{} `json:"meta"`
}

# 处理添加 Pin 请求的方法
func (p *PinningService) addPin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
    # 解析请求体中的 JSON 数据到 AddPinRequest 结构
    var addReq AddPinRequest
    err := json.NewDecoder(req.Body).Decode(&addReq)
    if err != nil {
        # 如果解析出错，返回错误信息
        errResp(writer, fmt.Sprintf("unmarshaling req: %s", err), "", http.StatusBadRequest)
        return
    }

    # 创建 PinStatus 对象
    pin := &PinStatus{
        RequestID: uuid.NewString(),
        Status:    statusQueued,
        Created:   time.Now(),
        Pin:       Pin(addReq),
    }

    # 加锁，向 pins 切片中添加 pin 对象，解锁
    p.m.Lock()
    p.pins = append(p.pins, pin)
    p.m.Unlock()

    # 将 pin 对象以 JSON 格式写入响应，设置状态码为已接受
    writeJSON(writer, &pin, http.StatusAccepted)
    # 触发 PinAdded 事件
    p.PinAdded(&addReq, pin)
}

# 定义列出 Pins 响应的数据结构
type ListPinsResponse struct {
    Count   int          `json:"count"`
    Results []*PinStatus `json:"results"`
}

# 处理列出 Pins 请求的方法
func (p *PinningService) listPins(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
    # 获取请求中的查询参数
    q := req.URL.Query()

    cidStr := q.Get("cid")
    name := q.Get("name")
    match := q.Get("match")
    status := q.Get("status")
    beforeStr := q.Get("before")
    afterStr := q.Get("after")
    limitStr := q.Get("limit")
    metaStr := q.Get("meta")

    # 如果 limit 参数为空，默认设置为 10
    if limitStr == "" {
        limitStr = "10"
    }
    # 将 limit 参数转换为整数
    limit, err := strconv.Atoi(limitStr)
    if err != nil {
        # 如果转换出错，返回错误信息
        errResp(writer, fmt.Sprintf("parsing limit: %s", err), "", http.StatusBadRequest)
        return
    }

    var cids []string
}
    # 如果 cidStr 不为空，则使用逗号分隔符将其拆分成字符串数组
    if cidStr != "" {
        cids = strings.Split(cidStr, ",")
    }

    # 初始化一个空的字符串数组statuses
    var statuses []string
    # 如果status不为空，则使用逗号分隔符将其拆分成字符串数组
    if status != "" {
        statuses = strings.Split(status, ",")
    }

    # 获取互斥锁
    p.m.Lock()
    # 延迟释放互斥锁
    defer p.m.Unlock()
    # 初始化一个空的PinStatus指针数组pins
    var pins []*PinStatus
    }

    # 构建ListPinsResponse结构体实例out
    out := ListPinsResponse{
        Count:   len(pins),  # 设置Count字段为pins数组的长度
        Results: pins,        # 设置Results字段为pins数组
    }
    # 将out以HTTP状态码http.StatusOK写入响应
    writeJSON(writer, out, http.StatusOK)
# 获取特定请求ID的固定信息
func (p *PinningService) getPin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
    # 从 URL 参数中获取请求ID
    requestID := params.ByName("requestID")
    # 加锁，确保并发安全
    p.m.Lock()
    # 延迟解锁，确保在函数返回时解锁
    defer p.m.Unlock()
    # 遍历固定信息列表，查找匹配请求ID的固定信息
    for _, pin := range p.pins {
        if pin.RequestID == requestID {
            # 将匹配的固定信息以 JSON 格式写入响应
            writeJSON(writer, pin, http.StatusOK)
            return
        }
    }
    # 如果未找到匹配的固定信息，返回 404 错误
    errResp(writer, "", "", http.StatusNotFound)
}

# 替换特定请求ID的固定信息
func (p *PinningService) replacePin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
    # 从 URL 参数中获取请求ID
    requestID := params.ByName("requestID")
    # 定义一个变量用于存储替换的固定信息
    var replaceReq Pin
    # 解析请求体中的 JSON 数据到 replaceReq 变量中
    err := json.NewDecoder(req.Body).Decode(&replaceReq)
    # 如果解析出错，返回 400 错误
    if err != nil {
        errResp(writer, fmt.Sprintf("decoding request: %s", err), "", http.StatusBadRequest)
        return
    }
    # 加锁，确保并发安全
    p.m.Lock()
    # 延迟解锁，确保在函数返回时解锁
    defer p.m.Unlock()
    # 遍历固定信息列表，查找匹配请求ID的固定信息
    for _, pin := range p.pins {
        if pin.RequestID == requestID {
            # 加锁，确保并发安全
            pin.M.Lock()
            # 替换匹配的固定信息为新的固定信息
            pin.Pin = replaceReq
            # 解锁
            pin.M.Unlock()
            # 返回 202 状态码，表示接受请求
            writer.WriteHeader(http.StatusAccepted)
            return
        }
    }
    # 如果未找到匹配的固定信息，返回 404 错误
    errResp(writer, "", "", http.StatusNotFound)
}

# 移除特定请求ID的固定信息
func (p *PinningService) removePin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
    # 从 URL 参数中获取请求ID
    requestID := params.ByName("requestID")
    # 加锁，确保并发安全
    p.m.Lock()
    # 延迟解锁，确保在函数返回时解锁
    defer p.m.Unlock()
    # 遍历固定信息列表，查找匹配请求ID的固定信息
    for i, pin := range p.pins {
        if pin.RequestID == requestID {
            # 从固定信息列表中移除匹配的固定信息
            p.pins = append(p.pins[0:i], p.pins[i+1:]...)
            # 返回 202 状态码，表示接受请求
            writer.WriteHeader(http.StatusAccepted)
            return
        }
    }
    # 如果未找到匹配的固定信息，返回 404 错误
    errResp(writer, "", "", http.StatusNotFound)
}
```