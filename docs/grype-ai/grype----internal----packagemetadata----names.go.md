# `grype\grype\internal\packagemetadata\names.go`

```
package packagemetadata

import (
    "reflect"  // 导入反射包，用于获取类型信息
    "sort"     // 导入排序包，用于对字符串进行排序
    "strings"  // 导入字符串包，用于处理字符串

    "github.com/anchore/grype/grype/pkg"  // 导入自定义包
)

// jsonNameFromType is a map of all known package metadata types to their current JSON name and all previously known aliases.
// TODO: in the future the metadata type names should match how it is used in syft. However, since the data shapes are
// not the same it may be important to select different names. This design decision has been deferred, for now
// the same metadata types that have been used in the past should be used here.
var jsonNameFromType = map[reflect.Type][]string{
    reflect.TypeOf(pkg.ApkMetadata{}):       nameList("ApkMetadata"),  // 将类型和对应的 JSON 名称存储到映射中
    reflect.TypeOf(pkg.GolangBinMetadata{}): nameList("GolangBinMetadata"),
    reflect.TypeOf(pkg.GolangModMetadata{}): nameList("GolangModMetadata"),
    reflect.TypeOf(pkg.JavaMetadata{}):      nameList("JavaMetadata"),
    reflect.TypeOf(pkg.RpmMetadata{}):       nameList("RpmMetadata"),
}

//nolint:unparam
func nameList(id string, others ...string) []string {
    names := []string{id}  // 创建包含 id 的字符串切片
    for _, o := range others {
        names = append(names, expandLegacyNameVariants(o)...)  // 将其他字符串添加到切片中
    }
    return names  // 返回字符串切片
}

func expandLegacyNameVariants(name string) []string {
    candidates := []string{name}  // 创建包含 name 的字符串切片
    if strings.HasSuffix(name, "MetadataType") {  // 如果字符串以 "MetadataType" 结尾
        candidates = append(candidates, strings.TrimSuffix(name, "Type"))  // 将去除 "Type" 后的字符串添加到切片中
    } else if strings.HasSuffix(name, "Metadata") {  // 如果字符串以 "Metadata" 结尾
        candidates = append(candidates, name+"Type")  // 将在末尾添加 "Type" 后的字符串添加到切片中
    }
    return candidates  // 返回字符串切片
}

func AllTypeNames() []string {
    names := make([]string, 0)  // 创建空字符串切片
    for _, t := range AllTypes() {  // 遍历所有类型
        names = append(names, reflect.TypeOf(t).Name())  // 将类型的名称添加到切片中
    }
    return names  // 返回字符串切片
}

func JSONName(metadata any) string {
    if vs, exists := jsonNameFromType[reflect.TypeOf(metadata)]; exists {  // 如果映射中存在对应类型的 JSON 名称
        return vs[0]  // 返回第一个 JSON 名称
    }
    return ""  // 返回空字符串
}

func ReflectTypeFromJSONName(name string) reflect.Type {
    name = strings.ToLower(name)  // 将名称转换为小写
}
    # 遍历已排序的类型列表，使用下划线 _ 忽略索引值
    for _, t := range sortedTypes(jsonNameFromType) {
        # 获取类型 t 对应的 JSON 名称列表
        vs := jsonNameFromType[t]
        # 遍历 JSON 名称列表
        for _, v := range vs {
            # 如果 JSON 名称转换为小写后与给定名称相同，则返回类型 t
            if strings.ToLower(v) == name {
                return t
            }
        }
    }
    # 如果未找到匹配的类型，则返回空值
    return nil
// 根据类型名称映射表对类型进行排序
func sortedTypes(typeNameMapping map[reflect.Type][]string) []reflect.Type {
    // 创建一个空的类型切片
    types := make([]reflect.Type, 0)
    // 遍历类型名称映射表，将类型添加到切片中
    for t := range typeNameMapping {
        types = append(types, t)
    }

    // 根据类型的第一个 JSON 名称对类型进行排序
    sort.Slice(types, func(i, j int) bool {
        return typeNameMapping[types[i]][0] < typeNameMapping[types[j]][0]
    })

    // 返回排序后的类型切片
    return types
}
```