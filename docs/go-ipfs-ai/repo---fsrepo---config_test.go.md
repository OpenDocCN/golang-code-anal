# `kubo\repo\fsrepo\config_test.go`

```
package fsrepo_test

import (
    "encoding/json"  // 导入 JSON 编解码包
    "reflect"  // 导入反射包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/plugin/loader"  // 导入插件加载包
    "github.com/ipfs/kubo/repo/fsrepo"  // 导入文件系统仓库包

    "github.com/ipfs/kubo/config"  // 导入配置包
)

// note: to test sorting of the mountpoints in the disk spec they are
// specified out of order in the test config.
var defaultConfig = []byte(`{  // 定义默认配置的 JSON 字节切片
    "StorageMax": "10GB",  // 存储最大容量
    "StorageGCWatermark": 90,  // 存储垃圾回收水印
    "GCPeriod": "1h",  // 垃圾回收周期
    "Spec": {  // 存储规格
      "mounts": [  // 挂载点列表
        {
          "child": {  // 子节点配置
            "compression": "none",  // 压缩方式
            "path": "datastore",  // 存储路径
            "type": "levelds"  // 存储类型
          },
          "mountpoint": "/",  // 挂载点
          "prefix": "leveldb.datastore",  // 前缀
          "type": "measure"  // 类型
        },
        {
          "child": {  // 子节点配置
            "path": "blocks",  // 存储路径
            "shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",  // 分片函数
            "sync": true,  // 同步标志
            "type": "flatfs"  // 存储类型
          },
          "mountpoint": "/blocks",  // 挂载点
          "prefix": "flatfs.datastore",  // 前缀
          "type": "measure"  // 类型
        }
      ],
      "type": "mount"  // 类型
    },
    "HashOnRead": false,  // 读取时哈希标志
    "BloomFilterSize": 0  // 布隆过滤器大小
}`)

var leveldbConfig = []byte(`{  // 定义 LevelDB 配置的 JSON 字节切片
            "compression": "none",  // 压缩方式
            "path": "datastore",  // 存储路径
            "type": "levelds"  // 存储类型
}`)

var flatfsConfig = []byte(`{  // 定义 FlatFS 配置的 JSON 字节切片
            "path": "blocks",  // 存储路径
            "shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",  // 分片函数
            "sync": true,  // 同步标志
            "type": "flatfs"  // 存储类型
}`)

var measureConfig = []byte(`{  // 定义测量配置的 JSON 字节切片
          "child": {  // 子节点配置
            "path": "blocks",  // 存储路径
            "shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",  // 分片函数
            "sync": true,  // 同步标志
            "type": "flatfs"  // 存储类型
          },
          "mountpoint": "/blocks",  // 挂载点
          "prefix": "flatfs.datastore",  // 前缀
          "type": "measure"  // 类型
}`)

func TestDefaultDatastoreConfig(t *testing.T) {
    loader, err := loader.NewPluginLoader("")  // 创建插件加载器
    if err != nil {  // 如果有错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    err = loader.Initialize()  // 初始化加载器
    if err != nil {  // 如果有错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    err = loader.Inject()  // 注入加载器
    # 如果发生错误，使用 t.Fatal() 终止测试并输出错误信息
    if err != nil:
        t.Fatal(err)

    # 创建临时目录
    dir := t.TempDir()

    # 创建一个 config.Datastore 对象，并将 defaultConfig 反序列化到该对象中
    config := new(config.Datastore)
    err = json.Unmarshal(defaultConfig, config)
    if err != nil:
        t.Fatal(err)

    # 根据 config.Spec 创建一个数据存储配置对象
    dsc, err := fsrepo.AnyDatastoreConfig(config.Spec)
    if err != nil:
        t.Fatal(err)

    # 预期的数据存储配置字符串
    expected := `{"mounts":[{"mountpoint":"/blocks","path":"blocks","shardFunc":"/repo/flatfs/shard/v1/next-to-last/2","type":"flatfs"},{"mountpoint":"/","path":"datastore","type":"levelds"}],"type":"mount"}`
    # 检查创建的数据存储配置对象的字符串表示是否符合预期
    if dsc.DiskSpec().String() != expected:
        t.Errorf("expected '%s' got '%s' as DiskId", expected, dsc.DiskSpec().String())

    # 根据数据存储配置对象在指定目录创建数据存储
    ds, err := dsc.Create(dir)
    if err != nil:
        t.Fatal(err)

    # 检查创建的数据存储对象类型是否符合预期
    if typ := reflect.TypeOf(ds).String(); typ != "*mount.Datastore" {
        t.Errorf("expected '*mount.Datastore' got '%s'", typ)
    }
# 测试 LevelDB 配置的函数
func TestLevelDbConfig(t *testing.T):
    # 创建一个新的 Datastore 配置对象
    config := new(config.Datastore)
    # 将默认配置解析为 Datastore 对象
    err := json.Unmarshal(defaultConfig, config)
    # 如果解析出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)
    # 创建临时目录
    dir := t.TempDir()

    # 创建一个空的 map 用于存储 LevelDB 配置
    spec := make(map[string]interface{})
    # 将 LevelDB 配置解析为 map
    err = json.Unmarshal(leveldbConfig, &spec)
    # 如果解析出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 根据配置创建任意类型的 Datastore 配置
    dsc, err := fsrepo.AnyDatastoreConfig(spec)
    # 如果创建出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 预期的 Datastore 配置字符串
    expected := `{"path":"datastore","type":"levelds"}`
    # 检查创建的 Datastore 配置字符串是否符合预期
    if dsc.DiskSpec().String() != expected:
        t.Errorf("expected '%s' got '%s' as DiskId", expected, dsc.DiskSpec().String())

    # 根据配置和目录创建 Datastore
    ds, err := dsc.Create(dir)
    # 如果创建出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 检查创建的 Datastore 类型是否为预期类型
    if typ := reflect.TypeOf(ds).String(); typ != "*leveldb.Datastore":
        t.Errorf("expected '*leveldb.datastore' got '%s'", typ)


# 测试 FlatFS 配置的函数
func TestFlatfsConfig(t *testing.T):
    # 创建一个新的 Datastore 配置对象
    config := new(config.Datastore)
    # 将默认配置解析为 Datastore 对象
    err := json.Unmarshal(defaultConfig, config)
    # 如果解析出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)
    # 创建临时目录
    dir := t.TempDir()

    # 创建一个空的 map 用于存储 FlatFS 配置
    spec := make(map[string]interface{})
    # 将 FlatFS 配置解析为 map
    err = json.Unmarshal(flatfsConfig, &spec)
    # 如果解析出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 根据配置创建任意类型的 Datastore 配置
    dsc, err := fsrepo.AnyDatastoreConfig(spec)
    # 如果创建出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 预期的 Datastore 配置字符串
    expected := `{"path":"blocks","shardFunc":"/repo/flatfs/shard/v1/next-to-last/2","type":"flatfs"}`
    # 检查创建的 Datastore 配置字符串是否符合预期
    if dsc.DiskSpec().String() != expected:
        t.Errorf("expected '%s' got '%s' as DiskId", expected, dsc.DiskSpec().String())

    # 根据配置和目录创建 Datastore
    ds, err := dsc.Create(dir)
    # 如果创建出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)

    # 检查创建的 Datastore 类型是否为预期类型
    if typ := reflect.TypeOf(ds).String(); typ != "*flatfs.Datastore":
        t.Errorf("expected '*flatfs.Datastore' got '%s'", typ)


# 测试 Measure 配置的函数
func TestMeasureConfig(t *testing.T):
    # 创建一个新的 Datastore 配置对象
    config := new(config.Datastore)
    # 将默认配置解析为 Datastore 对象
    err := json.Unmarshal(defaultConfig, config)
    # 如果解析出错，则输出错误信息并终止测试
    if err != nil:
        t.Fatal(err)
    # 创建临时目录
    dir := t.TempDir()

    # 创建一个空的 map 用于存储 Measure 配置
    spec := make(map[string]interface{})
    // 将 JSON 数据解析到 spec 变量中
    err = json.Unmarshal(measureConfig, &spec)
    // 如果解析出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 根据 spec 创建数据存储配置
    dsc, err := fsrepo.AnyDatastoreConfig(spec)
    // 如果创建配置出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 预期的数据存储配置字符串
    expected := `{"path":"blocks","shardFunc":"/repo/flatfs/shard/v1/next-to-last/2","type":"flatfs"}`
    // 检查创建的数据存储配置是否符合预期
    if dsc.DiskSpec().String() != expected {
        t.Errorf("expected '%s' got '%s' as DiskId", expected, dsc.DiskSpec().String())
    }

    // 根据配置创建数据存储
    ds, err := dsc.Create(dir)
    // 如果创建数据存储出错，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 检查创建的数据存储类型是否符合预期
    if typ := reflect.TypeOf(ds).String(); typ != "*measure.measure" {
        t.Errorf("expected '*measure.measure' got '%s'", typ)
    }
# 闭合前面的函数定义
```