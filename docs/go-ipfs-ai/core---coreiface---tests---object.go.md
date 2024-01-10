# `kubo\core\coreiface\tests\object.go`

```
package tests

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于管理上下文
    "encoding/hex"  // 导入 hex 包，用于进行十六进制编解码
    "io"  // 导入 io 包，用于进行 I/O 操作
    "strings"  // 导入 strings 包，用于操作字符串
    "testing"  // 导入 testing 包，用于编写测试函数

    iface "github.com/ipfs/kubo/core/coreiface"  // 导入 coreiface 包中的 iface 模块
    opt "github.com/ipfs/kubo/core/coreiface/options"  // 导入 coreiface 包中的 options 模块
)

func (tp *TestSuite) TestObject(t *testing.T) {  // 定义测试函数 TestObject
    tp.hasApi(t, func(api iface.CoreAPI) error {  // 调用 hasApi 方法，传入测试对象和回调函数
        if api.Object() == nil {  // 判断是否存在 Object 接口
            return errAPINotImplemented  // 返回错误信息
        }
        return nil  // 返回空值
    })

    t.Run("TestNew", tp.TestNew)  // 运行子测试函数 TestNew
    t.Run("TestObjectPut", tp.TestObjectPut)  // 运行子测试函数 TestObjectPut
    // ... 运行其他子测试函数
}

func (tp *TestSuite) TestNew(t *testing.T) {  // 定义测试函数 TestNew
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消函数的上下文
    defer cancel()  // 延迟调用取消函数
    api, err := tp.makeAPI(t, ctx)  // 调用 makeAPI 方法，传入测试对象和上下文，获取 API 对象和错误信息
    if err != nil {  // 判断是否存在错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    emptyNode, err := api.Object().New(ctx)  // 调用 API 对象的 New 方法，创建空节点
    if err != nil {  // 判断是否存在错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    dirNode, err := api.Object().New(ctx, opt.Object.Type("unixfs-dir"))  // 调用 API 对象的 New 方法，创建目录节点
    if err != nil {  // 判断是否存在错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    if emptyNode.String() != "QmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n" {  // 判断空节点的路径是否符合预期
        t.Errorf("Unexpected emptyNode path: %s", emptyNode.String())  // 输出错误信息
    }

    if dirNode.String() != "QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn" {  // 判断目录节点的路径是否符合预期
        t.Errorf("Unexpected dirNode path: %s", dirNode.String())  // 输出错误信息
    }
}

func (tp *TestSuite) TestObjectPut(t *testing.T) {  // 定义测试函数 TestObjectPut
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消函数的上下文
    defer cancel()  // 延迟调用取消函数
    api, err := tp.makeAPI(t, ctx)  // 调用 makeAPI 方法，传入测试对象和上下文，获取 API 对象和错误信息
    if err != nil {  // 判断是否存在错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    // 调用 api.Object().Put 方法将字符串 `{"Data":"foo"}` 存储到 IPFS，并返回存储路径 p1
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 api.Object().Put 方法将 base64 编码的字符串 `{"Data":"YmFy"}` 存储到 IPFS，并返回存储路径 p2
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"YmFy"}`), opt.Object.DataType("base64")) // bar
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将十六进制字符串 "0a0362617a" 解码为字节流，存储在 pbBytes 中
    pbBytes, err := hex.DecodeString("0a0362617a")
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 调用 api.Object().Put 方法将字节流 pbBytes 存储到 IPFS，并指定输入编码为 protobuf，返回存储路径 p3
    p3, err := api.Object().Put(ctx, bytes.NewReader(pbBytes), opt.Object.InputEnc("protobuf"))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查 p1 的存储路径是否符合预期，如果不符合，记录错误
    if p1.String() != "/ipfs/QmQeGyS87nyijii7kFt1zbe4n2PsXTFimzsdxyE9qh9TST" {
        t.Errorf("unexpected path: %s", p1.String())
    }

    // 检查 p2 的存储路径是否符合预期，如果不符合，记录错误
    if p2.String() != "/ipfs/QmNeYRbCibmaMMK6Du6ChfServcLqFvLJF76PzzF76SPrZ" {
        t.Errorf("unexpected path: %s", p2.String())
    }

    // 检查 p3 的存储路径是否符合预期，如果不符合，记录错误
    if p3.String() != "/ipfs/QmZreR7M2t7bFXAdb1V5FtQhjk4t36GnrvueLJowJbQM9m" {
        t.Errorf("unexpected path: %s", p3.String())
    }
func (tp *TestSuite) TestObjectGet(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据 {"Data":"foo"} 放入对象存储，并返回其标识符
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 根据标识符获取对象数据
    nd, err := api.Object().Get(ctx, p1)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查获取的数据是否以 "foo" 结尾，如果不是则记录错误并终止测试
    if string(nd.RawData()[len(nd.RawData())-3:]) != "foo" {
        t.Fatal("got non-matching data")
    }
}

func (tp *TestSuite) TestObjectData(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据 {"Data":"foo"} 放入对象存储，并返回其标识符
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 根据标识符获取对象数据的读取器
    r, err := api.Object().Data(ctx, p1)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 读取数据读取器中的所有数据
    data, err := io.ReadAll(r)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查读取的数据是否为 "foo"，如果不是则记录错误并终止测试
    if string(data) != "foo" {
        t.Fatal("got non-matching data")
    }
}

func (tp *TestSuite) TestObjectLinks(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据 {"Data":"foo"} 放入对象存储，并返回其标识符
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将包含链接的数据放入对象存储，并返回其标识符
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Links":[{"Name":"bar", "Hash":"`+p1.RootCid().String()+`"}]}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取对象存储中指定对象的链接
    links, err := api.Object().Links(ctx, p2)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查链接的数量是否为1，如果不是则记录错误
    if len(links) != 1 {
        t.Errorf("unexpected number of links: %d", len(links))
    }

    // 检查链接的标识符是否与预期的一致，如果不是则记录错误并终止测试
    if links[0].Cid.String() != p1.RootCid().String() {
        t.Fatal("cids didn't batch")
    }

    // 检查链接的名称是否为 "bar"，如果不是则记录错误并终止测试
    if links[0].Name != "bar" {
        t.Fatal("unexpected link name")
    }
}
func (tp *TestSuite) TestObjectStat(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用上下文创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储添加数据，并获取返回的对象
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储添加数据，并获取返回的对象
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"bazz", "Links":[{"Name":"bar", "Hash":"`+p1.RootCid().String()+`", "Size":3}]}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取对象的统计信息
    stat, err := api.Object().Stat(ctx, p2)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查对象的 CID 是否与预期的 CID 相符
    if stat.Cid.String() != p2.RootCid().String() {
        t.Error("unexpected stat.Cid")
    }

    // 检查对象的链接数是否符合预期
    if stat.NumLinks != 1 {
        t.Errorf("unexpected stat.NumLinks")
    }

    // 检查对象的块大小是否符合预期
    if stat.BlockSize != 51 {
        t.Error("unexpected stat.BlockSize")
    }

    // 检查对象的链接大小是否符合预期
    if stat.LinksSize != 47 {
        t.Errorf("unexpected stat.LinksSize: %d", stat.LinksSize)
    }

    // 检查对象的数据大小是否符合预期
    if stat.DataSize != 4 {
        t.Error("unexpected stat.DataSize")
    }

    // 检查对象的累积大小是否符合预期
    if stat.CumulativeSize != 54 {
        t.Error("unexpected stat.DataSize")
    }
}

func (tp *TestSuite) TestObjectAddLink(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用上下文创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储添加数据，并获取返回的对象
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储添加数据，并获取返回的对象
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"bazz", "Links":[{"Name":"bar", "Hash":"`+p1.RootCid().String()+`", "Size":3}]}`))
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象添加链接，并获取返回的对象
    p3, err := api.Object().AddLink(ctx, p2, "abc", p2)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取对象的链接列表
    links, err := api.Object().Links(ctx, p3)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查链接列表的长度是否符合预期
    if len(links) != 2 {
        t.Errorf("unexpected number of links: %d", len(links))
    }
}
    # 如果第一个链接的名称不是"abc"，则输出错误信息
    if links[0].Name != "abc" {
        t.Errorf("unexpected link 0 name: %s", links[0].Name)
    }

    # 如果第二个链接的名称不是"bar"，则输出错误信息
    if links[1].Name != "bar" {
        t.Errorf("unexpected link 1 name: %s", links[1].Name)
    }
func (tp *TestSuite) TestObjectAddLinkCreate(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试套件的 makeAPI 方法创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储中添加一个对象，并返回对象信息
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果添加对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储中添加一个对象，并返回对象信息
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"bazz", "Links":[{"Name":"bar", "Hash":"`+p1.RootCid().String()+`", "Size":3}]}`))
    // 如果添加对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储中的对象添加一个链接
    _, err = api.Object().AddLink(ctx, p2, "abc/d", p2)
    // 如果添加链接时未发生错误，则输出预期错误信息并终止测试
    if err == nil {
        t.Fatal("expected an error")
    }
    // 如果错误信息不包含指定内容，则输出错误信息并终止测试
    if !strings.Contains(err.Error(), "no link by that name") {
        t.Fatalf("unexpected error: %s", err.Error())
    }

    // 向对象存储中的对象添加一个链接，并创建新对象
    p3, err := api.Object().AddLink(ctx, p2, "abc/d", p2, opt.Object.Create(true))
    // 如果添加链接时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取对象存储中对象的链接信息
    links, err := api.Object().Links(ctx, p3)
    // 如果获取链接信息时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 如果链接数量不等于2，则输出错误信息
    if len(links) != 2 {
        t.Errorf("unexpected number of links: %d", len(links))
    }

    // 如果第一个链接的名称不等于"abc"，则输出错误信息
    if links[0].Name != "abc" {
        t.Errorf("unexpected link 0 name: %s", links[0].Name)
    }

    // 如果第二个链接的名称不等于"bar"，则输出错误信息
    if links[1].Name != "bar" {
        t.Errorf("unexpected link 1 name: %s", links[1].Name)
    }
}

func (tp *TestSuite) TestObjectRmLink(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试套件的 makeAPI 方法创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储中添加一个对象，并返回对象信息
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果添加对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象存储中添加一个对象，并返回对象信息
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"bazz", "Links":[{"Name":"bar", "Hash":"`+p1.RootCid().String()+`", "Size":3}]}`))
    // 如果添加对象时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从对象存储中的对象中移除一个链接
    p3, err := api.Object().RmLink(ctx, p2, "bar")
    // 如果移除链接时发生错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取对象存储中对象的链接信息
    links, err := api.Object().Links(ctx, p3)
    # 如果错误不为空，测试失败并输出错误信息
    if err != nil:
        t.Fatal(err)
    
    # 如果链接列表的长度不为0，输出链接数量不符合预期的错误信息
    if len(links) != 0:
        t.Errorf("unexpected number of links: %d", len(links))
func (tp *TestSuite) TestObjectAddData(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象添加数据
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果添加数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象追加数据
    p2, err := api.Object().AppendData(ctx, p1, strings.NewReader("bar"))
    // 如果追加数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 读取对象数据
    r, err := api.Object().Data(ctx, p2)
    // 如果读取数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 读取数据流中的所有数据
    data, err := io.ReadAll(r)
    // 如果读取数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查读取的数据是否符合预期
    if string(data) != "foobar" {
        t.Error("unexpected data")
    }
}

func (tp *TestSuite) TestObjectSetData(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象设置数据
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果设置数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 设置对象的数据
    p2, err := api.Object().SetData(ctx, p1, strings.NewReader("bar"))
    // 如果设置数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 读取对象数据
    r, err := api.Object().Data(ctx, p2)
    // 如果读取数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 读取数据流中的所有数据
    data, err := io.ReadAll(r)
    // 如果读取数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查读取的数据是否符合预期
    if string(data) != "bar" {
        t.Error("unexpected data")
    }
}

func (tp *TestSuite) TestDiffTest(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 函数创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果创建 API 对象出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象设置数据
    p1, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"foo"}`))
    // 如果设置数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 向对象设置数据
    p2, err := api.Object().Put(ctx, strings.NewReader(`{"Data":"bar"}`))
    // 如果设置数据出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 比较两个对象的数据差异
    changes, err := api.Object().Diff(ctx, p1, p2)
    // 如果比较数据差异出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
}
    # 检查 changes 的长度是否为1，如果不是则输出错误信息
    if len(changes) != 1 {
        t.Fatal("unexpected changes len")
    }

    # 检查 changes[0] 的类型是否为 DiffMod，如果不是则输出错误信息
    if changes[0].Type != iface.DiffMod {
        t.Fatal("unexpected change type")
    }

    # 检查 changes[0] 的 Before 字段是否与 p1 相同，如果不是则输出错误信息
    if changes[0].Before.String() != p1.String() {
        t.Fatal("unexpected before path")
    }

    # 检查 changes[0] 的 After 字段是否与 p2 相同，如果不是则输出错误信息
    if changes[0].After.String() != p2.String() {
        t.Fatal("unexpected before path")
    }
# 闭合前面的函数定义
```