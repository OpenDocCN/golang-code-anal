# `trojan-go\test\util\util.go`

```
// CheckConn检查两个netConn是否连接并正常工作
func CheckConn(a net.Conn, b net.Conn) bool {
    // 创建用于发送和接收数据的缓冲区
    payload1 := make([]byte, 1024)
    payload2 := make([]byte, 1024)

    result1 := make([]byte, 1024)
    result2 := make([]byte, 1024)

    // 从随机源中读取随机数据填充缓冲区
    rand.Reader.Read(payload1)
    rand.Reader.Read(payload2)

    // 创建一个等待组，用于等待两个goroutine完成
    wg := sync.WaitGroup{}
    wg.Add(2)

    // 启动一个goroutine，向连接a写入payload1并从连接a读取数据到result2
    go func() {
        a.Write(payload1)
        a.Read(result2)
        wg.Done()
    }()

    // 启动一个goroutine，从连接b读取数据到result1并向连接b写入payload2
    go func() {
        b.Read(result1)
        b.Write(payload2)
        wg.Done()
    }()

    // 等待两个goroutine完成
    wg.Wait()

    // 返回两个payload和result是否相等的布尔值
    return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
}

// CheckPacketOverConn检查两个PacketConn通过连接流式传输是否正常工作
func CheckPacketOverConn(a, b net.PacketConn) bool {
    // 选择一个端口和地址
    port := common.PickPort("tcp", "127.0.0.1")
    addr := &net.UDPAddr{
        IP:   net.ParseIP("127.0.0.1"),
        Port: port,
    }

    // 创建用于发送和接收数据的缓冲区
    payload1 := make([]byte, 1024)
    payload2 := make([]byte, 1024)

    result1 := make([]byte, 1024)
    result2 := make([]byte, 1024)

    // 从随机源中读取随机数据填充缓冲区
    rand.Reader.Read(payload1)
    rand.Reader.Read(payload2)

    // 向连接a写入payload1并发送到指定地址，从连接b读取数据到result1
    common.Must2(a.WriteTo(payload1, addr))
    _, addr1, err := b.ReadFrom(result1)
    common.Must(err)
    // 检查地址是否匹配，不匹配则返回false
    if addr1.String() != addr.String() {
        return false
    }

    // 向连接a写入payload2并发送到指定地址，从连接b读取数据到result2
    common.Must2(a.WriteTo(payload2, addr))
    _, addr2, err := b.ReadFrom(result2)
    common.Must(err)
    // 检查地址是否匹配，不匹配则返回false
    if addr2.String() != addr.String() {
        return false
    }

    // 返回两个payload和result是否相等的布尔值
    return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
}

// CheckPacket检查两个PacketConn是否正常工作
func CheckPacket(a, b net.PacketConn) bool {
    // 创建用于发送和接收数据的缓冲区
    payload1 := make([]byte, 1024)
    payload2 := make([]byte, 1024)

    result1 := make([]byte, 1024)
    result2 := make([]byte, 1024)

    // 从随机源中读取随机数据填充缓冲区
    rand.Reader.Read(payload1)
    rand.Reader.Read(payload2)

    // 向连接a写入payload1并发送到连接b的本地地址
    _, err := a.WriteTo(payload1, b.LocalAddr())
    # 调用 common.Must 函数，如果 err 不为 nil，则触发 panic
    common.Must(err)
    # 从 result1 中读取数据到 b 中，返回读取的字节数和错误
    _, _, err = b.ReadFrom(result1)
    common.Must(err)

    # 将 b 中的数据写入 payload2，并发送到 a 的地址，返回写入的字节数和错误
    _, err = b.WriteTo(payload2, a.LocalAddr())
    common.Must(err)
    # 从 a 中读取数据到 result2 中，返回读取的字节数和错误
    _, _, err = a.ReadFrom(result2)
    common.Must(err)

    # 返回 payload1 和 result1 是否相等的布尔值，以及 payload2 和 result2 是否相等的布尔值的逻辑与结果
    return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
# 定义一个名为 GetTestAddr 的函数，用于获取测试地址
func GetTestAddr() string:
    # 调用 common 包中的 PickPort 函数，选择一个可用的端口号
    port := common.PickPort("tcp", "127.0.0.1")
    # 返回格式化后的测试地址，包括 IP 地址和端口号
    return fmt.Sprintf("127.0.0.1:%d", port)
```