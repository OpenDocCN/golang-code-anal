# `kubo\plugin\plugins\badgerds\badgerds.go`

```go
package badgerds

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于操作系统功能
    "path/filepath"  // 导入 filepath 包，用于处理文件路径

    "github.com/ipfs/kubo/plugin"  // 导入 kubo 插件
    "github.com/ipfs/kubo/repo"  // 导入 kubo 存储库
    "github.com/ipfs/kubo/repo/fsrepo"  // 导入 kubo 存储库文件系统

    humanize "github.com/dustin/go-humanize"  // 导入 humanize 包，用于人性化显示数据大小
    badgerds "github.com/ipfs/go-ds-badger"  // 导入 badger 数据存储包
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{  // 定义插件列表
    &badgerdsPlugin{},  // 添加 badger 数据存储插件
}

type badgerdsPlugin struct{}  // 定义 badger 数据存储插件结构体

var _ plugin.PluginDatastore = (*badgerdsPlugin)(nil)  // 确保 badger 数据存储插件实现了 PluginDatastore 接口

func (*badgerdsPlugin) Name() string {  // 定义获取插件名称的方法
    return "ds-badgerds"  // 返回插件名称
}

func (*badgerdsPlugin) Version() string {  // 定义获取插件版本的方法
    return "0.1.0"  // 返回插件版本
}

func (*badgerdsPlugin) Init(_ *plugin.Environment) error {  // 定义初始化方法
    return nil  // 返回空错误，表示初始化成功
}

func (*badgerdsPlugin) DatastoreTypeName() string {  // 定义获取数据存储类型名称的方法
    return "badgerds"  // 返回数据存储类型名称
}

type datastoreConfig struct {  // 定义数据存储配置结构体
    path       string  // 数据存储路径
    syncWrites bool  // 是否同步写入
    truncate   bool  // 是否截断

    vlogFileSize int64  // vlog 文件大小
}

// BadgerdsDatastoreConfig returns a configuration stub for a badger datastore
// from the given parameters.
func (*badgerdsPlugin) DatastoreConfigParser() fsrepo.ConfigFromMap {  // 定义数据存储配置解析方法
    # 定义一个函数，接收一个参数 map[string]interface{}，返回 fsrepo.DatastoreConfig 和 error
    return func(params map[string]interface{}) (fsrepo.DatastoreConfig, error) {
        # 声明变量 c 和 ok
        var c datastoreConfig
        var ok bool
    
        # 从参数中获取 path 字段的值，并判断是否为字符串类型
        c.path, ok = params["path"].(string)
        if !ok:
            # 如果不是字符串类型，则返回错误信息
            return nil, fmt.Errorf("'path' field is missing or not string")
        }
    
        # 从参数中获取 syncWrites 字段的值，并判断是否存在
        sw, ok := params["syncWrites"]
        if !ok {
            # 如果不存在，则将 syncWrites 设置为 false
            c.syncWrites = false
        } else {
            # 如果存在，则判断其类型是否为布尔类型
            if swb, ok := sw.(bool); ok {
                c.syncWrites = swb
            } else {
                # 如果不是布尔类型，则返回错误信息
                return nil, fmt.Errorf("'syncWrites' field was not a boolean")
            }
        }
    
        # 从参数中获取 truncate 字段的值，并判断是否存在
        truncate, ok := params["truncate"]
        if !ok {
            # 如果不存在，则将 truncate 设置为 true
            c.truncate = true
        } else {
            # 如果存在，则判断其类型是否为布尔类型
            if truncate, ok := truncate.(bool); ok {
                c.truncate = truncate
            } else {
                # 如果不是布尔类型，则返回错误信息
                return nil, fmt.Errorf("'truncate' field was not a boolean")
            }
        }
    
        # 从参数中获取 vlogFileSize 字段的值，并判断是否存在
        vls, ok := params["vlogFileSize"]
        if !ok {
            # 如果不存在，则将 vlogFileSize 设置为默认值 1GiB
            c.vlogFileSize = badgerds.DefaultOptions.ValueLogFileSize
        } else {
            # 如果存在，则判断其类型是否为字符串类型
            if vlogSize, ok := vls.(string); ok {
                # 将字符串类型的值解析为字节大小
                s, err := humanize.ParseBytes(vlogSize)
                if err != nil {
                    # 如果解析出错，则返回错误信息
                    return nil, err
                }
                c.vlogFileSize = int64(s)
            } else {
                # 如果不是字符串类型，则返回错误信息
                return nil, fmt.Errorf("'vlogFileSize' field was not a string")
            }
        }
    
        # 返回配置对象 c 和 nil 错误
        return &c, nil
    }
# 返回数据存储的磁盘规格
func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec {
    return map[string]interface{}{
        "type": "badgerds",
        "path": c.path,
    }
}

# 创建数据存储
func (c *datastoreConfig) Create(path string) (repo.Datastore, error) {
    # 获取数据存储的路径
    p := c.path
    # 如果路径不是绝对路径，则拼接成绝对路径
    if !filepath.IsAbs(p) {
        p = filepath.Join(path, p)
    }

    # 创建所有必要的目录
    err := os.MkdirAll(p, 0o755)
    if err != nil {
        return nil, err
    }

    # 设置 Badger 数据存储的默认选项
    defopts := badgerds.DefaultOptions
    defopts.SyncWrites = c.syncWrites
    defopts.Truncate = c.truncate
    defopts.ValueLogFileSize = c.vlogFileSize

    # 使用指定路径和选项创建 Badger 数据存储
    return badgerds.NewDatastore(p, &defopts)
}
```