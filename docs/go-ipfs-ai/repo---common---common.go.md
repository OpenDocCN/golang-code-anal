# `kubo\repo\common\common.go`

```
package common

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "strings" // 导入 strings 包，用于字符串操作
)

// 从 map 中获取指定 key 的 value，支持多层级 key，返回值和可能的错误
func MapGetKV(v map[string]interface{}, key string) (interface{}, error) {
    var ok bool // 声明变量 ok，用于判断类型断言是否成功
    var mcursor map[string]interface{} // 声明变量 mcursor，用于存储 map 类型的值
    var cursor interface{} = v // 声明变量 cursor，初始化为参数 v

    parts := strings.Split(key, ".") // 使用 "." 分割 key，得到多层级的 key
    for i, part := range parts { // 遍历分割后的 key
        sofar := strings.Join(parts[:i], ".") // 获取当前遍历的 key 的前缀

        mcursor, ok = cursor.(map[string]interface{}) // 尝试将 cursor 转换为 map 类型
        if !ok { // 如果转换失败
            return nil, fmt.Errorf("%s key is not a map", sofar) // 返回错误信息
        }

        cursor, ok = mcursor[part] // 获取当前 key 对应的 value
        if !ok { // 如果获取失败
            // 构造当前遍历路径，用于打印友好的错误信息
            var path string
            if len(sofar) > 0 {
                path += sofar + "."
            }
            path += part
            return nil, fmt.Errorf("%s not found", path) // 返回错误信息
        }
    }
    return cursor, nil // 返回获取到的 value 和 nil 错误
}

// 向 map 中设置指定 key 的 value，支持多层级 key，返回可能的错误
func MapSetKV(v map[string]interface{}, key string, value interface{}) error {
    var ok bool // 声明变量 ok，用于判断类型断言是否成功
    var mcursor map[string]interface{} // 声明变量 mcursor，用于存储 map 类型的值
    var cursor interface{} = v // 声明变量 cursor，初始化为参数 v

    parts := strings.Split(key, ".") // 使用 "." 分割 key，得到多层级的 key
    for i, part := range parts { // 遍历分割后的 key
        mcursor, ok = cursor.(map[string]interface{}) // 尝试将 cursor 转换为 map 类型
        if !ok { // 如果转换失败
            sofar := strings.Join(parts[:i], ".") // 获取当前遍历的 key 的前缀
            return fmt.Errorf("%s key is not a map", sofar) // 返回错误信息
        }

        // 最后一部分？在这里设置值
        if i == (len(parts) - 1) { // 如果是最后一部分
            mcursor[part] = value // 设置对应 key 的值为 value
            break // 跳出循环
        }

        cursor, ok = mcursor[part] // 获取当前 key 对应的 value
        if !ok || cursor == nil { // 如果获取失败或者值为空
            mcursor[part] = map[string]interface{}{} // 创建一个新的 map
            cursor = mcursor[part] // 更新 cursor 为新创建的 map
        }
    }
    return nil // 返回 nil 错误
}

// 将右侧 map 递归合并到左侧 map 中，直到找到非 map 类型的值
func MapMergeDeep(left, right map[string]interface{}) map[string]interface{} {
    // 我们想要修改 map 的副本，而不是原始的 map
    result := make(map[string]interface{}) // 创建一个新的 map
    for k, v := range left { // 遍历左侧 map
        result[k] = v // 将左侧 map 的值复制到新的 map 中
    }
    // 遍历右边的 map，获取键值对
    for key, rightVal := range right {
        // 如果右边的值是一个 map
        if rightMap, ok := rightVal.(map[string]interface{}); ok {
            // 如果键在左边也存在
            if leftVal, found := result[key]; found {
                // 如果左边的值也是一个 map
                if leftMap, ok := leftVal.(map[string]interface{}); ok {
                    // 合并嵌套的 map
                    result[key] = MapMergeDeep(leftMap, rightMap)
                    continue
                }
            }
        }

        // 否则将新值设置到结果中
        result[key] = rightVal
    }

    // 返回结果
    return result
# 闭合前面的函数定义
```