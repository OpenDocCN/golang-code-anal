# `grype\grype\db\v5\pkg\qualifier\from_json.go`

```
package qualifier

import (
    "encoding/json"  // 导入 JSON 编码解码包

    "github.com/mitchellh/mapstructure"  // 导入 map 结构解析包

    "github.com/anchore/grype/grype/db/v5/pkg/qualifier/platformcpe"  // 导入平台 CPE 资格认证包
    "github.com/anchore/grype/grype/db/v5/pkg/qualifier/rpmmodularity"  // 导入 RPM 模块化包
    "github.com/anchore/grype/internal/log"  // 导入日志包
)

func FromJSON(data []byte) ([]Qualifier, error) {
    var records []map[string]interface{}  // 定义一个 map 切片
    if err := json.Unmarshal(data, &records); err != nil {  // 使用 JSON 解码将数据解析到 map 切片中
        return nil, err  // 如果解析出错，返回错误
    }

    var qualifiers []Qualifier  // 定义一个 Qualifier 切片

    for _, r := range records {  // 遍历 map 切片中的记录
        k, ok := r["kind"]  // 获取记录中的 "kind" 字段值

        if !ok {  // 如果 "kind" 字段不存在
            log.Warn("Skipping qualifier with no kind specified")  // 记录警告日志
            continue  // 继续下一次循环
        }

        // create the specific kind of Qualifier
        switch k {  // 根据 "kind" 字段值进行分支判断
        case "rpm-modularity":  // 如果 "kind" 字段值为 "rpm-modularity"
            var q rpmmodularity.Qualifier  // 定义一个 RPM 模块化 Qualifier
            if err := mapstructure.Decode(r, &q); err != nil {  // 使用 map 结构解析将记录解析到 RPM 模块化 Qualifier 中
                log.Warn("Error decoding rpm-modularity package qualifier:  (%v)", err)  // 记录警告日志
                continue  // 继续下一次循环
            }
            qualifiers = append(qualifiers, q)  // 将解析后的 Qualifier 添加到切片中
        case "platform-cpe":  // 如果 "kind" 字段值为 "platform-cpe"
            var q platformcpe.Qualifier  // 定义一个平台 CPE Qualifier
            if err := mapstructure.Decode(r, &q); err != nil {  // 使用 map 结构解析将记录解析到平台 CPE Qualifier 中
                log.Warn("Error decoding platform-cpe package qualifier:  (%v)", err)  // 记录警告日志
                continue  // 继续下一次循环
            }
            qualifiers = append(qualifiers, q)  // 将解析后的 Qualifier 添加到切片中
        default:  // 如果 "kind" 字段值为其它值
            log.Debug("Skipping unsupported package qualifier: %s", k)  // 记录调试日志
            continue  // 继续下一次循环
        }
    }

    return qualifiers, nil  // 返回解析后的 Qualifier 切片和空错误
}
```