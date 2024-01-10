# `kubo\plugin\plugins\levelds\levelds.go`

```
package levelds

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "path/filepath"  // 导入 filepath 包，用于处理文件路径

    "github.com/ipfs/kubo/plugin"  // 导入 kubo 插件包
    "github.com/ipfs/kubo/repo"  // 导入 kubo repo 包
    "github.com/ipfs/kubo/repo/fsrepo"  // 导入 kubo repo fsrepo 包

    levelds "github.com/ipfs/go-ds-leveldb"  // 导入 levelds 包
    ldbopts "github.com/syndtr/goleveldb/leveldb/opt"  // 导入 ldbopts 包
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{  // 定义插件列表
    &leveldsPlugin{},  // levelds 插件
}

type leveldsPlugin struct{}  // levelds 插件结构体

var _ plugin.PluginDatastore = (*leveldsPlugin)(nil)  // 插件数据存储接口

func (*leveldsPlugin) Name() string {  // 获取插件名称
    return "ds-level"
}

func (*leveldsPlugin) Version() string {  // 获取插件版本
    return "0.1.0"
}

func (*leveldsPlugin) Init(_ *plugin.Environment) error {  // 初始化插件
    return nil
}

func (*leveldsPlugin) DatastoreTypeName() string {  // 获取数据存储类型名称
    return "levelds"
}

type datastoreConfig struct {  // 数据存储配置结构体
    path        string  // 存储路径
    compression ldbopts.Compression  // 压缩方式
}

// BadgerdsDatastoreConfig returns a configuration stub for a badger datastore
// from the given parameters.
func (*leveldsPlugin) DatastoreConfigParser() fsrepo.ConfigFromMap {  // 数据存储配置解析器
    return func(params map[string]interface{}) (fsrepo.DatastoreConfig, error) {  // 返回数据存储配置和错误
        var c datastoreConfig  // 定义数据存储配置
        var ok bool  // 定义布尔值

        c.path, ok = params["path"].(string)  // 获取存储路径
        if !ok {  // 如果获取失败
            return nil, fmt.Errorf("'path' field is missing or not string")  // 返回错误信息
        }

        switch cm := params["compression"]; cm {  // 根据压缩方式进行判断
        case "none":  // 如果是无压缩
            c.compression = ldbopts.NoCompression  // 设置为无压缩
        case "snappy":  // 如果是 snappy 压缩
            c.compression = ldbopts.SnappyCompression  // 设置为 snappy 压缩
        case "", nil:  // 如果为空或为 nil
            c.compression = ldbopts.DefaultCompression  // 设置为默认压缩
        default:  // 默认情况
            return nil, fmt.Errorf("unrecognized value for compression: %s", cm)  // 返回错误信息
        }

        return &c, nil  // 返回数据存储配置和 nil
    }
}

func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec {  // 获取磁盘规格
    return map[string]interface{}{  // 返回磁盘规格
        "type": "levelds",  // 存储类型为 levelds
        "path": c.path,  // 存储路径为 c.path
    }
}

func (c *datastoreConfig) Create(path string) (repo.Datastore, error) {  // 创建数据存储
    p := c.path  // 获取存储路径
    # 如果路径不是绝对路径，则将其与给定的路径拼接成一个新的路径
    if !filepath.IsAbs(p):
        p = filepath.Join(path, p)
    
    # 使用给定的路径创建一个新的 levelds 数据存储对象，设置压缩选项为指定的压缩方式
    return levelds.NewDatastore(p, &levelds.Options{
        Compression: c.compression,
    })
# 闭合前面的函数定义
```