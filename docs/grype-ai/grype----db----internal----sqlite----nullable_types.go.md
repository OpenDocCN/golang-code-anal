# `grype\grype\db\internal\sqlite\nullable_types.go`

```go
package sqlite

import (
    "database/sql"  // 导入数据库操作相关的包
    "encoding/json"  // 导入 JSON 编解码相关的包
)

type NullString struct {  // 定义 NullString 结构体
    sql.NullString  // 嵌入 sql.NullString 结构体
}

func NewNullString(s string, valid bool) NullString {  // 定义函数，用于创建 NullString 对象
    return NullString{  // 返回一个 NullString 对象
        sql.NullString{  // 创建 sql.NullString 对象
            String: s,  // 设置字符串值
            Valid:  valid,  // 设置有效性标志
        },
    }
}

func ToNullString(v any) NullString {  // 定义函数，将任意类型转换为 NullString
    nullString := NullString{}  // 创建一个空的 NullString 对象
    nullString.Valid = false  // 设置有效性标志为 false

    if v != nil {  // 如果输入值不为空
        var stringValue string  // 声明一个字符串变量

        if s, ok := v.(string); ok {  // 判断输入值是否为字符串类型
            stringValue = s  // 如果是，直接赋值给 stringValue
        } else {
            vBytes, err := json.Marshal(v)  // 否则，将输入值转换为 JSON 字节流
            if err != nil {
                // TODO: just no  // 如果转换出错，抛出异常
                panic(err)
            }

            stringValue = string(vBytes)  // 将 JSON 字节流转换为字符串
        }

        if stringValue != "null" {  // 如果字符串值不为 "null"
            nullString.String = stringValue  // 设置 NullString 对象的字符串值
            nullString.Valid = true  // 设置有效性标志为 true
        }
    }

    return nullString  // 返回处理后的 NullString 对象
}

func (v NullString) ToByteSlice() []byte {  // 定义方法，将 NullString 转换为字节切片
    if v.Valid {  // 如果 NullString 有效
        return []byte(v.String)  // 返回字符串值的字节切片
    }

    return []byte("null")  // 否则返回 "null" 的字节切片
}

func (v NullString) MarshalJSON() ([]byte, error) {  // 定义方法，将 NullString 转换为 JSON 字节流
    if v.Valid {  // 如果 NullString 有效
        return json.Marshal(v.String)  // 返回字符串值的 JSON 字节流
    }

    return json.Marshal(nil)  // 否则返回 null 的 JSON 字节流
}

func (v *NullString) UnmarshalJSON(data []byte) error {  // 定义方法，从 JSON 字节流解析出 NullString
    if data != nil && string(data) != "null" {  // 如果 JSON 字节流不为空且不为 "null"
        v.Valid = true  // 设置 NullString 有效
        v.String = string(data)  // 设置字符串值为 JSON 字节流转换的字符串
    } else {
        v.Valid = false  // 否则设置 NullString 无效
    }
    return nil  // 返回空错误
}
```