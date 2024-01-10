# `kubo\repo\fsrepo\fsrepo_test.go`

```
package fsrepo

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "context" // 导入 context 包，用于控制goroutine的生命周期
    "os" // 导入 os 包，提供操作系统函数
    "path/filepath" // 导入 filepath 包，用于操作文件路径
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/thirdparty/assert" // 导入第三方断言库

    datastore "github.com/ipfs/go-datastore" // 导入数据存储包
    config "github.com/ipfs/kubo/config" // 导入配置包
)

func TestInitIdempotence(t *testing.T) {
    t.Parallel() // 标记测试函数可以并行执行
    path := t.TempDir() // 创建临时目录作为仓库路径
    for i := 0; i < 10; i++ { // 循环10次
        assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "multiple calls to init should succeed") // 断言初始化函数调用成功
    }
}

func Remove(repoPath string) error {
    repoPath = filepath.Clean(repoPath) // 清理仓库路径
    return os.RemoveAll(repoPath) // 删除指定路径下的所有文件和目录
}

func TestCanManageReposIndependently(t *testing.T) {
    t.Parallel() // 标记测试函数可以并行执行
    pathA := t.TempDir() // 创建临时目录作为仓库A路径
    pathB := t.TempDir() // 创建临时目录作为仓库B路径

    t.Log("initialize two repos") // 打印日志
    assert.Nil(Init(pathA, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "a", "should initialize successfully") // 断言初始化仓库A成功
    assert.Nil(Init(pathB, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "b", "should initialize successfully") // 断言初始化仓库B成功

    t.Log("ensure repos initialized") // 打印日志
    assert.True(IsInitialized(pathA), t, "a should be initialized") // 断言仓库A已初始化
    assert.True(IsInitialized(pathB), t, "b should be initialized") // 断言仓库B已初始化

    t.Log("open the two repos") // 打印日志
    repoA, err := Open(pathA) // 打开仓库A
    assert.Nil(err, t, "a") // 断言打开仓库A成功
    repoB, err := Open(pathB) // 打开仓库B
    assert.Nil(err, t, "b") // 断言打开仓库B成功

    t.Log("close and remove b while a is open") // 打印日志
    assert.Nil(repoB.Close(), t, "close b") // 断言关闭仓库B成功
    assert.Nil(Remove(pathB), t, "remove b") // 删除仓库B

    t.Log("close and remove a") // 打印日志
    assert.Nil(repoA.Close(), t) // 断言关闭仓库A成功
    assert.Nil(Remove(pathA), t) // 删除仓库A
}

func TestDatastoreGetNotAllowedAfterClose(t *testing.T) {
    t.Parallel() // 标记测试函数可以并行执行
    path := t.TempDir() // 创建临时目录作为仓库路径

    assert.True(!IsInitialized(path), t, "should NOT be initialized") // 断言仓库未初始化
    assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "should initialize successfully") // 断言初始化仓库成功
    r, err := Open(path) // 打开仓库
    assert.Nil(err, t, "should open successfully") // 断言打开仓库成功

    k := "key" // 设置键
    data := []byte(k) // 设置数据
    # 使用断言检查数据存储是否成功，如果成功则不会有返回值，否则会输出错误信息
    assert.Nil(r.Datastore().Put(context.Background(), datastore.NewKey(k), data), t, "Put should be successful")

    # 使用断言检查资源关闭是否成功，如果成功则不会有返回值，否则会输出错误信息
    assert.Nil(r.Close(), t)
    # 使用断言检查数据存储获取是否失败，如果失败则会输出错误信息
    _, err = r.Datastore().Get(context.Background(), datastore.NewKey(k))
    assert.Err(err, t, "after closer, Get should be fail")
// 测试数据存储库从一个存储库到另一个存储库的持久性
func TestDatastorePersistsFromRepoToRepo(t *testing.T) {
    t.Parallel() // 并行测试
    path := t.TempDir() // 创建临时目录

    assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t) // 初始化存储库并断言是否成功
    r1, err := Open(path) // 打开存储库
    assert.Nil(err, t) // 断言是否成功打开存储库

    k := "key" // 设置键
    expected := []byte(k) // 设置预期值
    assert.Nil(r1.Datastore().Put(context.Background(), datastore.NewKey(k), expected), t, "using first repo, Put should be successful") // 断言使用第一个存储库时 Put 操作是否成功
    assert.Nil(r1.Close(), t) // 断言关闭存储库是否成功

    r2, err := Open(path) // 再次打开存储库
    assert.Nil(err, t) // 断言是否成功打开存储库
    actual, err := r2.Datastore().Get(context.Background(), datastore.NewKey(k)) // 获取存储库中的数据
    assert.Nil(err, t, "using second repo, Get should be successful") // 断言使用第二个存储库时 Get 操作是否成功
    assert.Nil(r2.Close(), t) // 断言关闭存储库是否成功
    assert.True(bytes.Equal(expected, actual), t, "data should match") // 断言数据是否匹配
}

// 在同一进程中多次打开存储库的测试
func TestOpenMoreThanOnceInSameProcess(t *testing.T) {
    t.Parallel() // 并行测试
    path := t.TempDir() // 创建临时目录
    assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t) // 初始化存储库并断言是否成功

    r1, err := Open(path) // 打开存储库
    assert.Nil(err, t, "first repo should open successfully") // 断言第一个存储库是否成功打开
    r2, err := Open(path) // 再次打开存储库
    assert.Nil(err, t, "second repo should open successfully") // 断言第二个存储库是否成功打开
    assert.True(r1 == r2, t, "second open returns same value") // 断言第二次打开存储库是否返回相同的值

    assert.Nil(r1.Close(), t) // 断言关闭第一个存储库是否成功
    assert.Nil(r2.Close(), t) // 断言关闭第二个存储库是否成功
}
```