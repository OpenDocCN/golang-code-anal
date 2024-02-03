# `v2ray-core\infra\conf\serial\loader.go`

```go
package serial

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 编解码
    "io"  // 导入 io 包，用于输入输出操作

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common/errors"  // 导入 v2ray 核心错误处理包
    "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
    json_reader "v2ray.com/core/infra/conf/json"  // 导入 v2ray JSON 配置包
)

type offset struct {
    line int  // 定义偏移量结构体，包含行数
    char int  // 定义偏移量结构体，包含字符数
}

func findOffset(b []byte, o int) *offset {
    if o >= len(b) || o < 0 {  // 如果偏移量超出范围，返回空
        return nil
    }

    line := 1  // 初始化行数为 1
    char := 0  // 初始化字符数为 0
    for i, x := range b {  // 遍历字节流
        if i == o {  // 如果遍历到指定偏移量，结束循环
            break
        }
        if x == '\n' {  // 如果遇到换行符
            line++  // 行数加一
            char = 0  // 字符数重置为 0
        } else {
            char++  // 字符数加一
        }
    }

    return &offset{line: line, char: char}  // 返回偏移量结构体指针
}

// DecodeJSONConfig 从 reader 中读取并解码配置为 *conf.Config
// 可以检测到语法错误
func DecodeJSONConfig(reader io.Reader) (*conf.Config, error) {
    jsonConfig := &conf.Config{}  // 初始化配置对象

    jsonContent := bytes.NewBuffer(make([]byte, 0, 10240))  // 创建一个缓冲区，用于存储 JSON 内容
    jsonReader := io.TeeReader(&json_reader.Reader{  // 创建一个读取器，用于同时读取和存储 JSON 内容
        Reader: reader,  // 传入参数 reader
    }, jsonContent)
    decoder := json.NewDecoder(jsonReader)  // 创建 JSON 解码器

    if err := decoder.Decode(jsonConfig); err != nil {  // 如果解码出错
        var pos *offset  // 定义偏移量指针
        cause := errors.Cause(err)  // 获取错误原因
        switch tErr := cause.(type) {  // 根据错误类型进行处理
        case *json.SyntaxError:  // 如果是语法错误
            pos = findOffset(jsonContent.Bytes(), int(tErr.Offset))  // 获取错误位置
        case *json.UnmarshalTypeError:  // 如果是解码类型错误
            pos = findOffset(jsonContent.Bytes(), int(tErr.Offset))  // 获取错误位置
        }
        if pos != nil {  // 如果错误位置存在
            return nil, newError("failed to read config file at line ", pos.line, " char ", pos.char).Base(err)  // 返回错误信息和原始错误
        }
        return nil, newError("failed to read config file").Base(err)  // 返回错误信息和原始错误
    }

    return jsonConfig, nil  // 返回配置对象和空错误
}

func LoadJSONConfig(reader io.Reader) (*core.Config, error) {
    jsonConfig, err := DecodeJSONConfig(reader)  // 解码 JSON 配置
    if err != nil {  // 如果解码出错
        return nil, err  // 返回空对象和错误
    }

    pbConfig, err := jsonConfig.Build()  // 构建配置
    if err != nil {  // 如果构建出错
        return nil, newError("failed to parse json config").Base(err)  // 返回错误信息和原始错误
    }

    return pbConfig, nil  // 返回配置对象和空错误
}
```