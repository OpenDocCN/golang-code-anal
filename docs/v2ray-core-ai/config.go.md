# `v2ray-core\config.go`

```go
// +build !confonly

package core

import (
    "io"
    "strings"

    "github.com/golang/protobuf/proto"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/cmdarg"
    "v2ray.com/core/main/confloader"
)

// ConfigFormat is a configurable format of V2Ray config file.
type ConfigFormat struct {
    Name      string      // 配置文件格式的名称
    Extension []string    // 配置文件格式的扩展名
    Loader    ConfigLoader // 加载器函数
}

// ConfigLoader is a utility to load V2Ray config from external source.
type ConfigLoader func(input interface{}) (*Config, error) // 加载配置的函数类型

var (
    configLoaderByName = make(map[string]*ConfigFormat) // 根据名称注册的配置加载器
    configLoaderByExt  = make(map[string]*ConfigFormat) // 根据扩展名注册的配置加载器
)

// RegisterConfigLoader add a new ConfigLoader.
func RegisterConfigLoader(format *ConfigFormat) error {
    name := strings.ToLower(format.Name) // 将名称转换为小写
    if _, found := configLoaderByName[name]; found {
        return newError(format.Name, " already registered.") // 如果名称已经注册，则返回错误
    }
    configLoaderByName[name] = format // 将加载器注册到名称映射中

    for _, ext := range format.Extension {
        lext := strings.ToLower(ext) // 将扩展名转换为小写
        if f, found := configLoaderByExt[lext]; found {
            return newError(ext, " already registered to ", f.Name) // 如果扩展名已经注册，则返回错误
        }
        configLoaderByExt[lext] = format // 将加载器注册到扩展名映射中
    }

    return nil
}

func getExtension(filename string) string {
    idx := strings.LastIndexByte(filename, '.') // 获取文件名中最后一个'.'的索引
    if idx == -1 {
        return "" // 如果没有找到'.'，则返回空字符串
    }
    return filename[idx+1:] // 返回'.'后面的扩展名
}

// LoadConfig loads config with given format from given source.
// input accepts 2 different types:
// * []string slice of multiple filename/url(s) to open to read
// * io.Reader that reads a config content (the original way)
func LoadConfig(formatName string, filename string, input interface{}) (*Config, error) {
    ext := getExtension(filename) // 获取文件名的扩展名
    if len(ext) > 0 {
        if f, found := configLoaderByExt[ext]; found {
            return f.Loader(input) // 如果找到对应扩展名的加载器，则使用该加载器加载配置
        }
    }

    if f, found := configLoaderByName[formatName]; found {
        return f.Loader(input) // 如果找到对应名称的加载器，则使用该加载器加载配置
    }
}
    # 返回空值和新的错误对象，指示在特定格式名称中无法加载配置，并将其标记为警告
    return nil, newError("Unable to load config in ", formatName).AtWarning()
# 加载 Protobuf 格式的配置数据，返回 Config 对象和可能的错误
func loadProtobufConfig(data []byte) (*Config, error) {
    # 创建一个新的 Config 对象
    config := new(Config)
    # 使用 protobuf 解析传入的数据，如果出错则返回错误
    if err := proto.Unmarshal(data, config); err != nil {
        return nil, err
    }
    # 返回解析后的 Config 对象和 nil 错误
    return config, nil
}

# 初始化函数
func init() {
    # 注册配置加载器，使用 Protobuf 格式
    common.Must(RegisterConfigLoader(&ConfigFormat{
        Name:      "Protobuf",
        Extension: []string{"pb"},
        Loader: func(input interface{}) (*Config, error) {
            # 根据输入类型进行不同的处理
            switch v := input.(type) {
            # 如果输入是 cmdarg.Arg 类型
            case cmdarg.Arg:
                # 加载配置文件
                r, err := confloader.LoadConfig(v[0])
                common.Must(err)
                # 读取配置文件内容到字节流
                data, err := buf.ReadAllToBytes(r)
                common.Must(err)
                # 调用 loadProtobufConfig 函数解析配置数据
                return loadProtobufConfig(data)
            # 如果输入是 io.Reader 类型
            case io.Reader:
                # 读取输入流的内容到字节流
                data, err := buf.ReadAllToBytes(v)
                common.Must(err)
                # 调用 loadProtobufConfig 函数解析配置数据
                return loadProtobufConfig(data)
            # 如果输入类型未知
            default:
                # 返回 nil 和新的错误信息
                return nil, newError("unknow type")
            }
        },
    }))
}
```