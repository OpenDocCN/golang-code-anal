# `kubo\plugin\plugins\flatfs\flatfs.go`

```go
package flatfs

import (
    "fmt"
    "path/filepath"

    "github.com/ipfs/kubo/plugin"
    "github.com/ipfs/kubo/repo"
    "github.com/ipfs/kubo/repo/fsrepo"

    flatfs "github.com/ipfs/go-ds-flatfs"
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{
    &flatfsPlugin{},
}

type flatfsPlugin struct{}

var _ plugin.PluginDatastore = (*flatfsPlugin)(nil)

func (*flatfsPlugin) Name() string {
    return "ds-flatfs"
}

func (*flatfsPlugin) Version() string {
    return "0.1.0"
}

func (*flatfsPlugin) Init(_ *plugin.Environment) error {
    return nil
}

func (*flatfsPlugin) DatastoreTypeName() string {
    return "flatfs"
}

type datastoreConfig struct {
    path      string
    shardFun  *flatfs.ShardIdV1
    syncField bool
}

// BadgerdsDatastoreConfig returns a configuration stub for a badger datastore
// from the given parameters.
func (*flatfsPlugin) DatastoreConfigParser() fsrepo.ConfigFromMap {
    return func(params map[string]interface{}) (fsrepo.DatastoreConfig, error) {
        var c datastoreConfig
        var ok bool
        var err error

        c.path, ok = params["path"].(string)  // 从参数中获取路径，并转换为字符串
        if !ok {
            return nil, fmt.Errorf("'path' field is missing or not boolean")  // 如果路径字段缺失或不是布尔值，则返回错误
        }

        sshardFun, ok := params["shardFunc"].(string)  // 从参数中获取分片函数，并转换为字符串
        if !ok {
            return nil, fmt.Errorf("'shardFunc' field is missing or not a string")  // 如果分片函数字段缺失或不是字符串，则返回错误
        }
        c.shardFun, err = flatfs.ParseShardFunc(sshardFun)  // 解析分片函数
        if err != nil {
            return nil, err  // 如果解析出错，则返回错误
        }

        c.syncField, ok = params["sync"].(bool)  // 从参数中获取同步字段，并转换为布尔值
        if !ok {
            return nil, fmt.Errorf("'sync' field is missing or not boolean")  // 如果同步字段缺失或不是布尔值，则返回错误
        }
        return &c, nil  // 返回数据存储配置和空错误
    }
}

func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec {
    return map[string]interface{}{
        "type":      "flatfs",
        "path":      c.path,
        "shardFunc": c.shardFun.String(),
    }
}
# 创建数据存储，接收路径参数并返回数据存储对象和错误信息
func (c *datastoreConfig) Create(path string) (repo.Datastore, error) {
    # 将配置中的路径赋值给变量 p
    p := c.path
    # 如果路径不是绝对路径，则将其与传入的路径拼接成新的路径
    if !filepath.IsAbs(p) {
        p = filepath.Join(path, p)
    }
    # 调用 flatfs 包中的 CreateOrOpen 方法，传入路径 p、分片函数和同步字段，返回数据存储对象和错误信息
    return flatfs.CreateOrOpen(p, c.shardFun, c.syncField)
}
```