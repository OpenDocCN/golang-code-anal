# `kubo\core\coreiface\tests\dag.go`

```
package tests

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "math"  // 导入数学包，用于处理最大值
    "strings"  // 导入字符串包，用于处理 JSON 字符串

    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理 IPFS 路径
    coreiface "github.com/ipfs/kubo/core/coreiface"  // 导入核心接口包，用于定义 IPFS 核心 API 接口

    ipldcbor "github.com/ipfs/go-ipld-cbor"  // 导入 CBOR 包，用于处理 IPLD 数据
    ipld "github.com/ipfs/go-ipld-format"  // 导入 IPLD 格式包，用于处理 IPLD 数据格式
    mh "github.com/multiformats/go-multihash"  // 导入多哈希包，用于处理多种哈希算法
)

func (tp *TestSuite) TestDag(t *testing.T) {
    tp.hasApi(t, func(api coreiface.CoreAPI) error {  // 调用测试套件的 hasApi 方法，传入测试对象和回调函数
        if api.Dag() == nil {  // 如果核心 API 中的 DAG 方法为空
            return errAPINotImplemented  // 返回未实现错误
        }
        return nil  // 返回空值
    })

    t.Run("TestPut", tp.TestPut)  // 运行测试用例 TestPut
    t.Run("TestPutWithHash", tp.TestPutWithHash)  // 运行测试用例 TestPutWithHash
    t.Run("TestPath", tp.TestDagPath)  // 运行测试用例 TestDagPath
    t.Run("TestTree", tp.TestTree)  // 运行测试用例 TestTree
    t.Run("TestBatch", tp.TestBatch)  // 运行测试用例 TestBatch
}

var treeExpected = map[string]struct{}{  // 定义预期的树结构
    "a":   {},  // 键为 "a"，值为空结构
    "b":   {},  // 键为 "b"，值为空结构
    "c":   {},  // 键为 "c"，值为空结构
    "c/d": {},  // 键为 "c/d"，值为空结构
    "c/e": {},  // 键为 "c/e"，值为空结构
}

func (tp *TestSuite) TestPut(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消功能的上下文
    defer cancel()  // 延迟调用取消函数
    api, err := tp.makeAPI(t, ctx)  // 调用测试套件的 makeAPI 方法，传入测试对象和上下文
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    nd, err := ipldcbor.FromJSON(strings.NewReader(`"Hello"`), math.MaxUint64, -1)  // 从 JSON 字符串创建 IPLD 数据节点
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    err = api.Dag().Add(ctx, nd)  // 调用核心 API 中的 DAG 方法，添加 IPLD 数据节点
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    if nd.Cid().String() != "bafyreicnga62zhxnmnlt6ymq5hcbsg7gdhqdu6z4ehu3wpjhvqnflfy6nm" {  // 如果节点的 CID 不符合预期
        t.Errorf("got wrong cid: %s", nd.Cid().String())  // 输出错误信息
    }
}

func (tp *TestSuite) TestPutWithHash(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建带有取消功能的上下文
    defer cancel()  // 延迟调用取消函数
    api, err := tp.makeAPI(t, ctx)  // 调用测试套件的 makeAPI 方法，传入测试对象和上下文
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    nd, err := ipldcbor.FromJSON(strings.NewReader(`"Hello"`), mh.SHA3_256, -1)  // 从 JSON 字符串创建 IPLD 数据节点，指定哈希算法
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    err = api.Dag().Add(ctx, nd)  // 调用核心 API 中的 DAG 方法，添加 IPLD 数据节点
    if err != nil {  // 如果发生错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    if nd.Cid().String() != "bafyrmifu7haikttpqqgc5ewvmp76z3z4ebp7h2ph4memw7dq4nt6btmxny" {  // 如果节点的 CID 不符合预期
        t.Errorf("got wrong cid: %s", nd.Cid().String())  // 输出错误信息
    }
}
func (tp *TestSuite) TestDagPath(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试套件的方法创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从 JSON 字符串创建 IPLD 数据
    snd, err := ipldcbor.FromJSON(strings.NewReader(`"foo"`), math.MaxUint64, -1)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据添加到 DAG 中
    err = api.Dag().Add(ctx, snd)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从 JSON 字符串创建 IPLD 数据
    nd, err := ipldcbor.FromJSON(strings.NewReader(`{"lnk": {"/": "`+snd.Cid().String()+`"}}`), math.MaxUint64, -1)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据添加到 DAG 中
    err = api.Dag().Add(ctx, nd)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 创建路径
    p, err := path.Join(path.FromCid(nd.Cid()), "lnk")
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 解析路径
    rp, _, err := api.ResolvePath(ctx, p)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从 DAG 中获取数据
    ndd, err := api.Dag().Get(ctx, rp.RootCid())
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查获取的数据是否与预期一致
    if ndd.Cid().String() != snd.Cid().String() {
        t.Errorf("got unexpected cid %s, expected %s", ndd.Cid().String(), snd.Cid().String())
    }
}

func (tp *TestSuite) TestTree(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 使用测试套件的方法创建 API 对象
    api, err := tp.makeAPI(t, ctx)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从 JSON 字符串创建 IPLD 数据
    nd, err := ipldcbor.FromJSON(strings.NewReader(`{"a": 123, "b": "foo", "c": {"d": 321, "e": 111}}`), math.MaxUint64, -1)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将数据添加到 DAG 中
    err = api.Dag().Add(ctx, nd)
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 从 DAG 中获取数据
    res, err := api.Dag().Get(ctx, nd.Cid())
    // 如果出现错误，记录错误并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 获取树形结构
    lst := res.Tree("", -1)
    // 检查树形结构的长度是否与预期一致
    if len(lst) != len(treeExpected) {
        t.Errorf("tree length of %d doesn't match expected %d", len(lst), len(treeExpected))
    }

    // 遍历树形结构，检查是否有意外的条目
    for _, ent := range lst {
        if _, ok := treeExpected[ent]; !ok {
            t.Errorf("unexpected tree entry %s", ent)
        }
    }
}
// 定义一个名为 TestBatch 的方法，参数为指向 TestSuite 结构体的指针和一个测试对象 t
func (tp *TestSuite) TestBatch(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 延迟调用取消函数
    defer cancel()
    // 调用 makeAPI 方法创建一个 API 对象，并检查是否有错误发生
    api, err := tp.makeAPI(t, ctx)
    if err != nil {
        t.Fatal(err)
    }

    // 将 JSON 字符串转换为 CBOR 格式的 IPLD 节点
    nd, err := ipldcbor.FromJSON(strings.NewReader(`"Hello"`), math.MaxUint64, -1)
    if err != nil {
        t.Fatal(err)
    }

    // 检查生成的 CID 是否符合预期值，如果不符合则输出错误信息
    if nd.Cid().String() != "bafyreicnga62zhxnmnlt6ymq5hcbsg7gdhqdu6z4ehu3wpjhvqnflfy6nm" {
        t.Errorf("got wrong cid: %s", nd.Cid().String())
    }

    // 尝试从 API 中获取指定 CID 的节点，如果返回错误为空或者错误信息中不包含 "not found" 则输出错误信息
    _, err = api.Dag().Get(ctx, nd.Cid())
    if err == nil || !strings.Contains(err.Error(), "not found") {
        t.Fatal(err)
    }

    // 将单个 IPLD 节点添加到 API 中，如果有错误发生则输出错误信息
    if err := api.Dag().AddMany(ctx, []ipld.Node{nd}); err != nil {
        t.Fatal(err)
    }

    // 再次尝试从 API 中获取指定 CID 的节点，如果返回错误则输出错误信息
    _, err = api.Dag().Get(ctx, nd.Cid())
    if err != nil {
        t.Fatal(err)
    }
}
```