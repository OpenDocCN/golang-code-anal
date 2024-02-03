# `kubo\repo\common\common_test.go`

```go
package common

import (
    "testing"

    "github.com/ipfs/kubo/thirdparty/assert"
)

func TestMapMergeDeepReturnsNew(t *testing.T) {
    leftMap := make(map[string]interface{})
    leftMap["A"] = "Hello World"

    rightMap := make(map[string]interface{})
    rightMap["A"] = "Foo"

    MapMergeDeep(leftMap, rightMap)

    assert.True(leftMap["A"] == "Hello World", t, "MapMergeDeep should return a new map instance")
}

func TestMapMergeDeepNewKey(t *testing.T) {
    leftMap := make(map[string]interface{})
    leftMap["A"] = "Hello World"
    /*
        leftMap
        {
            A: "Hello World"
        }
    */

    rightMap := make(map[string]interface{})
    rightMap["B"] = "Bar"
    /*
        rightMap
        {
            B: "Bar"
        }
    */

    result := MapMergeDeep(leftMap, rightMap)
    /*
        expected
        {
            A: "Hello World"
            B: "Bar"
        }
    */

    assert.True(result["B"] == "Bar", t, "New keys in right map should exist in resulting map")
}

func TestMapMergeDeepRecursesOnMaps(t *testing.T) {
    leftMapA := make(map[string]interface{})
    leftMapA["B"] = "A value!"
    leftMapA["C"] = "Another value!"

    leftMap := make(map[string]interface{})
    leftMap["A"] = leftMapA
    /*
        leftMap
        {
            A: {
                B: "A value!"
                C: "Another value!"
            }
        }
    */

    rightMapA := make(map[string]interface{})
    rightMapA["C"] = "A different value!"

    rightMap := make(map[string]interface{})
    rightMap["A"] = rightMapA
    /*
        rightMap
        {
            A: {
                C: "A different value!"
            }
        }
    */

    result := MapMergeDeep(leftMap, rightMap)
    /*
        expected
        {
            A: {
                B: "A value!"
                C: "A different value!"
            }
        }
    */

    resultA := result["A"].(map[string]interface{})
    // 获取合并后的结果中 A 键对应的值，并断言其类型为 map[string]interface{}
    # 断言resultA字典中键为"B"的值是否等于"A value!"，如果不等则抛出异常
    assert.True(resultA["B"] == "A value!", t, "Unaltered values should not change")
    # 断言resultA字典中键为"C"的值是否等于"A different value!"，如果不等则抛出异常
    assert.True(resultA["C"] == "A different value!", t, "Nested values should be altered")
func TestMapMergeDeepRightNotAMap(t *testing.T) {
    // 创建一个空的 map 类型为 string 到 interface{} 的映射
    leftMapA := make(map[string]interface{})
    // 在 leftMapA 中添加键值对 "B" 到 "A value!"
    leftMapA["B"] = "A value!"

    // 创建一个空的 map 类型为 string 到 interface{} 的映射
    leftMap := make(map[string]interface{})
    // 在 leftMap 中添加键值对 "A" 到 leftMapA
    leftMap["A"] = leftMapA
    /*
        origMap
        {
            A: {
                B: "A value!"
            }
        }
    */

    // 创建一个空的 map 类型为 string 到 interface{} 的映射
    rightMap := make(map[string]interface{})
    // 在 rightMap 中添加键值对 "A" 到 "Not a map!"
    rightMap["A"] = "Not a map!"
    /*
        newMap
        {
            A: "Not a map!"
        }
    */

    // 调用 MapMergeDeep 函数，将 leftMap 和 rightMap 合并
    result := MapMergeDeep(leftMap, rightMap)
    /*
        expected
        {
            A: "Not a map!"
        }
    */

    // 断言结果中键 "A" 的值是否等于 "Not a map!"，如果不等则输出错误信息
    assert.True(result["A"] == "Not a map!", t, "Right values that are not a map should be set on the result")
}
```