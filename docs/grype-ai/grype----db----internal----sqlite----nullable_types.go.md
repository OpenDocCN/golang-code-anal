# `grype\grype\db\internal\sqlite\nullable_types.go`

```
// 导入 sqlite 包和必要的依赖包
package sqlite

import (
	"database/sql" // 导入数据库操作包
	"encoding/json" // 导入 JSON 编解码包
)

// 定义 NullString 结构体，包含 sql.NullString 类型的字段
type NullString struct {
	sql.NullString
}

// 定义 NewNullString 函数，用于创建 NullString 对象
func NewNullString(s string, valid bool) NullString {
	// 返回一个 NullString 对象，包含一个初始化的 sql.NullString 对象
	return NullString{
		sql.NullString{
			String: s,  // 设置字符串值
			Valid:  valid,  // 设置有效性标志
		},
	}
}
# 将任意类型的值转换为 NullString 类型
func ToNullString(v any) NullString {
    # 创建一个空的 NullString 对象
    nullString := NullString{}
    # 将 Valid 属性设置为 false
    nullString.Valid = false

    # 如果值不为空
    if v != nil {
        # 声明一个字符串变量
        var stringValue string

        # 如果值是字符串类型
        if s, ok := v.(string); ok {
            # 将值转换为字符串赋值给 stringValue
            stringValue = s
        } else {
            # 如果值不是字符串类型，将其转换为 JSON 字符串
            vBytes, err := json.Marshal(v)
            # 如果转换过程中出现错误
            if err != nil {
                # 抛出错误
                panic(err)
            }
            # 将转换后的 JSON 字符串赋值给 stringValue
            stringValue = string(vBytes)
        }

        # 如果 stringValue 不等于 "null"
        if stringValue != "null" {
// 将字符串赋值给NullString的String字段
nullString.String = stringValue
// 设置NullString的Valid字段为true
nullString.Valid = true
```

```
// 将NullString转换为字节切片
func (v NullString) ToByteSlice() []byte {
	// 如果Valid为true，则返回String转换的字节切片
	if v.Valid {
		return []byte(v.String)
	}
	// 如果Valid为false，则返回字符串"null"的字节切片
	return []byte("null")
}
```

```
// 将NullString转换为JSON格式的字节切片
func (v NullString) MarshalJSON() ([]byte, error) {
	// 如果Valid为true，则使用json.Marshal将String转换为JSON格式的字节切片并返回
	if v.Valid {
		return json.Marshal(v.String)
	}
	// 如果Valid为false，则返回错误
}
# 返回一个表示空值的 JSON 对象
return json.Marshal(nil)
}

# 从 JSON 数据中解析出 NullString 对象
func (v *NullString) UnmarshalJSON(data []byte) error {
    # 如果数据不为空且不是 "null" 字符串，则将 Valid 设为 true，并将数据赋值给 String
    if data != nil && string(data) != "null" {
        v.Valid = true
        v.String = string(data)
    } else {
        # 否则将 Valid 设为 false
        v.Valid = false
    }
    # 返回空值
    return nil
}
```