# `v2ray-core\testing\scenarios\common.go`

```
package scenarios

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "crypto/rand" // 导入 crypto/rand 包，用于生成加密安全的随机数
    "fmt" // 导入 fmt 包，用于格式化输入输出
    "io" // 导入 io 包，提供了基本的接口和函数
    "io/ioutil" // 导入 ioutil 包，用于读取文件内容
    "os/exec" // 导入 exec 包，用于执行外部命令
    "path/filepath" // 导入 filepath 包，用于操作文件路径
    "runtime" // 导入 runtime 包，提供了与 Go 运行时环境交互的函数
    "sync" // 导入 sync 包，提供了基本的同步原语
    "syscall" // 导入 syscall 包，提供了操作系统底层接口
    "time" // 导入 time 包，提供了时间的测量和显示功能

    "github.com/golang/protobuf/proto" // 导入 proto 包，用于处理 Protocol Buffers 数据
    "v2ray.com/core" // 导入 core 包
    "v2ray.com/core/app/dispatcher" // 导入 dispatcher 包
    "v2ray.com/core/app/proxyman" // 导入 proxyman 包
    "v2ray.com/core/common" // 导入 common 包
    "v2ray.com/core/common/errors" // 导入 errors 包，用于处理错误
    "v2ray.com/core/common/log" // 导入 log 包，用于日志记录
    "v2ray.com/core/common/net" // 导入 net 包，用于网络操作
    "v2ray.com/core/common/retry" // 导入 retry 包，用于重试操作
    "v2ray.com/core/common/serial" // 导入 serial 包，用于序列化操作
)

func xor(b []byte) []byte {
    r := make([]byte, len(b)) // 创建一个与 b 长度相同的字节切片 r
    for i, v := range b {
        r[i] = v ^ 'c' // 对 b 中的每个字节进行异或运算
    }
    return r // 返回结果字节切片
}

func readFrom(conn net.Conn, timeout time.Duration, length int) []byte {
    b := make([]byte, length) // 创建一个指定长度的字节切片 b
    deadline := time.Now().Add(timeout) // 设置读取超时时间
    conn.SetReadDeadline(deadline) // 设置连接的读取截止时间
    n, err := io.ReadFull(conn, b[:length]) // 从连接中读取指定长度的数据
    if err != nil {
        fmt.Println("Unexpected error from readFrom:", err) // 如果发生错误，打印错误信息
    }
    return b[:n] // 返回实际读取的数据
}

func readFrom2(conn net.Conn, timeout time.Duration, length int) ([]byte, error) {
    b := make([]byte, length) // 创建一个指定长度的字节切片 b
    deadline := time.Now().Add(timeout) // 设置读取超时时间
    conn.SetReadDeadline(deadline) // 设置连接的读取截止时间
    n, err := io.ReadFull(conn, b[:length]) // 从连接中读取指定长度的数据
    if err != nil {
        return nil, err // 如果发生错误，返回 nil 和错误信息
    }
    return b[:n], nil // 返回实际读取的数据和 nil
}

func InitializeServerConfigs(configs ...*core.Config) ([]*exec.Cmd, error) {
    servers := make([]*exec.Cmd, 0, 10) // 创建一个长度为 0，容量为 10 的 exec.Cmd 切片 servers

    for _, config := range configs {
        server, err := InitializeServerConfig(config) // 初始化服务器配置
        if err != nil {
            CloseAllServers(servers) // 关闭所有服务器
            return nil, err // 返回 nil 和错误信息
        }
        servers = append(servers, server) // 将服务器添加到切片中
    }

    time.Sleep(time.Second * 2) // 休眠 2 秒

    return servers, nil // 返回服务器切片和 nil
}

func InitializeServerConfig(config *core.Config) (*exec.Cmd, error) {
    err := BuildV2Ray() // 构建 V2Ray
    if err != nil {
        return nil, err // 如果发生错误，返回 nil 和错误信息
    }

    config = withDefaultApps(config) // 使用默认应用程序配置
    configBytes, err := proto.Marshal(config) // 序列化配置
    if err != nil {
        return nil, err // 如果发生错误，返回 nil 和错误信息
    }
    # 运行 V2RayProtobuf，并将结果赋值给 proc
    proc := RunV2RayProtobuf(configBytes)

    # 如果启动过程中出现错误，返回空和错误信息
    if err := proc.Start(); err != nil {
        return nil, err
    }

    # 返回 proc 和空错误信息
    return proc, nil
// 定义全局变量 testBinaryPath 和 testBinaryPathGen
var (
    testBinaryPath    string
    testBinaryPathGen sync.Once
)

// 生成测试二进制文件路径
func genTestBinaryPath() {
    // 使用 sync.Once 确保只执行一次
    testBinaryPathGen.Do(func() {
        var tempDir string
        // 使用 common.Must(retry.Timed(5, 100).On(func() error {...})) 生成临时目录
        common.Must(retry.Timed(5, 100).On(func() error {
            dir, err := ioutil.TempDir("", "v2ray")
            if err != nil {
                return err
            }
            tempDir = dir
            return nil
        }))
        // 生成测试二进制文件路径
        file := filepath.Join(tempDir, "v2ray.test")
        if runtime.GOOS == "windows" {
            file += ".exe"
        }
        testBinaryPath = file
        fmt.Printf("Generated binary path: %s\n", file)
    })
}

// 获取源代码路径
func GetSourcePath() string {
    return filepath.Join("v2ray.com", "core", "main")
}

// 关闭所有服务器
func CloseAllServers(servers []*exec.Cmd) {
    // 记录通用消息
    log.Record(&log.GeneralMessage{
        Severity: log.Severity_Info,
        Content:  "Closing all servers.",
    })
    // 关闭所有服务器进程
    for _, server := range servers {
        if runtime.GOOS == "windows" {
            server.Process.Kill()
        } else {
            server.Process.Signal(syscall.SIGTERM)
        }
    }
    // 等待所有服务器进程结束
    for _, server := range servers {
        server.Process.Wait()
    }
    // 记录通用消息
    log.Record(&log.GeneralMessage{
        Severity: log.Severity_Info,
        Content:  "All server closed.",
    })
}

// 添加默认应用程序配置
func withDefaultApps(config *core.Config) *core.Config {
    // 添加默认应用程序配置
    config.App = append(config.App, serial.ToTypedMessage(&dispatcher.Config{}))
    config.App = append(config.App, serial.ToTypedMessage(&proxyman.InboundConfig{}))
    config.App = append(config.App, serial.ToTypedMessage(&proxyman.OutboundConfig{}))
    return config
}

// 测试 TCP 连接
func testTCPConn(port net.Port, payloadSize int, timeout time.Duration) func() error {
    return func() error {
        // 建立 TCP 连接
        conn, err := net.DialTCP("tcp", nil, &net.TCPAddr{
            IP:   []byte{127, 0, 0, 1},
            Port: int(port),
        })
        if err != nil {
            return err
        }
        defer conn.Close()

        // 调用 testTCPConn2 函数进行测试
        return testTCPConn2(conn, payloadSize, timeout)()
    }
}
    # 代码块结束
# 定义一个测试 UDP 连接的函数，接受端口号、数据包大小和超时时间作为参数
func testUDPConn(port net.Port, payloadSize int, timeout time.Duration) func() error {
    # 返回一个匿名函数，该函数返回一个错误
    return func() error {
        # 使用 UDP 协议创建连接
        conn, err := net.DialUDP("udp", nil, &net.UDPAddr{
            IP:   []byte{127, 0, 0, 1},  # 设置 IP 地址为本地回环地址
            Port: int(port),  # 设置端口号
        })
        # 如果连接出现错误，返回错误信息
        if err != nil {
            return err
        }
        # 延迟关闭连接
        defer conn.Close()

        # 调用 testTCPConn2 函数，传入连接、数据包大小和超时时间，并执行返回的函数
        return testTCPConn2(conn, payloadSize, timeout)()
    }
}

# 定义一个测试 TCP 连接的函数，接受连接、数据包大小和超时时间作为参数
func testTCPConn2(conn net.Conn, payloadSize int, timeout time.Duration) func() error {
    # 返回一个匿名函数，该函数返回一个错误
    return func() error {
        # 创建指定大小的数据包
        payload := make([]byte, payloadSize)
        # 生成随机数据填充数据包
        common.Must2(rand.Read(payload))

        # 将数据包写入连接
        nBytes, err := conn.Write(payload)
        # 如果写入出现错误，返回错误信息
        if err != nil {
            return err
        }
        # 如果写入的字节数不等于数据包大小，返回错误信息
        if nBytes != len(payload) {
            return errors.New("expect ", len(payload), " written, but actually ", nBytes)
        }

        # 从连接中读取响应数据
        response, err := readFrom2(conn, timeout, payloadSize)
        # 如果读取出现错误，返回错误信息
        if err != nil {
            return err
        }
        # 对响应数据和原始数据包进行异或操作，并比较结果
        if r := bytes.Compare(response, xor(payload)); r != 0 {
            return errors.New(r)
        }

        # 返回空值表示没有错误
        return nil
    }
}
```