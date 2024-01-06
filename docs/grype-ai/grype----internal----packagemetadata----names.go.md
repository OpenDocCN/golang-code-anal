# `grype\grype\internal\packagemetadata\names.go`

```
package packagemetadata

import (
	"reflect"  // 导入 reflect 包，用于获取类型信息
	"sort"     // 导入 sort 包，用于对数据进行排序
	"strings"  // 导入 strings 包，用于处理字符串

	"github.com/anchore/grype/grype/pkg"  // 导入自定义包

)

// jsonNameFromType 是一个包含所有已知包元数据类型到它们当前的 JSON 名称和所有先前已知别名的映射
// TODO: 在将来，元数据类型名称应该与在 syft 中使用的方式相匹配。然而，由于数据形状不同，选择不同的名称可能很重要。这个设计决定已经被推迟，现在应该使用过去使用过的相同元数据类型。
var jsonNameFromType = map[reflect.Type][]string{
	reflect.TypeOf(pkg.ApkMetadata{}):       nameList("ApkMetadata"),  // 获取 ApkMetadata 类型的反射信息，并将其与名称 "ApkMetadata" 关联
	reflect.TypeOf(pkg.GolangBinMetadata{}): nameList("GolangBinMetadata"),  // 获取 GolangBinMetadata 类型的反射信息，并将其与名称 "GolangBinMetadata" 关联
	reflect.TypeOf(pkg.GolangModMetadata{}): nameList("GolangModMetadata"),  // 获取 GolangModMetadata 类型的反射信息，并将其与名称 "GolangModMetadata" 关联
	reflect.TypeOf(pkg.JavaMetadata{}):      nameList("JavaMetadata"),  // 获取 JavaMetadata 类型的反射信息，并将其与名称 "JavaMetadata" 关联
	reflect.TypeOf(pkg.RpmMetadata{}):       nameList("RpmMetadata"),  // 获取 RpmMetadata 类型的反射信息，并将其与名称 "RpmMetadata" 关联
// 结束当前函数的定义

// 根据给定的 id 和其他可变参数，返回一个包含所有参数的字符串切片
// nolint:unparam 表示忽略未使用的参数警告
func nameList(id string, others ...string) []string {
    // 创建一个包含 id 的字符串切片
    names := []string{id}
    // 遍历其他参数，将其扩展后添加到 names 中
    for _, o := range others {
        names = append(names, expandLegacyNameVariants(o)...)
    }
    // 返回包含所有参数的字符串切片
    return names
}

// 根据给定的名称，返回一个包含可能的名称变体的字符串切片
func expandLegacyNameVariants(name string) []string {
    // 创建一个包含原始名称的字符串切片
    candidates := []string{name}
    // 如果名称以 "MetadataType" 结尾，将去掉 "Type" 后的名称添加到 candidates 中
    if strings.HasSuffix(name, "MetadataType") {
        candidates = append(candidates, strings.TrimSuffix(name, "Type"))
    } 
    // 如果名称以 "Metadata" 结尾，将添加 "Type" 后的名称添加到 candidates 中
    else if strings.HasSuffix(name, "Metadata") {
        candidates = append(candidates, name+"Type")
    }
    // 返回可能的名称变体的字符串切片
    return candidates
}
# 返回所有类型的名称的切片
func AllTypeNames() []string {
    # 创建一个空的字符串切片
    names := make([]string, 0)
    # 遍历所有类型，并将类型名称添加到切片中
    for _, t := range AllTypes() {
        names = append(names, reflect.TypeOf(t).Name())
    }
    return names
}

# 根据元数据返回 JSON 名称
func JSONName(metadata any) string {
    # 如果元数据的类型在 jsonNameFromType 中存在，则返回对应的 JSON 名称
    if vs, exists := jsonNameFromType[reflect.TypeOf(metadata)]; exists {
        return vs[0]
    }
    # 如果不存在，则返回空字符串
    return ""
}

# 根据 JSON 名称返回反射类型
func ReflectTypeFromJSONName(name string) reflect.Type {
    # 将名称转换为小写
    name = strings.ToLower(name)
    # 遍历排序后的 jsonNameFromType 中的类型
    for _, t := range sortedTypes(jsonNameFromType) {
        vs := jsonNameFromType[t]
	// 遍历vs切片中的值
	for _, v := range vs {
		// 将v转换为小写并与name比较
		if strings.ToLower(v) == name {
			// 如果相等则返回t
			return t
		}
	}
	// 如果未找到匹配的值，则返回nil
	return nil
}

func sortedTypes(typeNameMapping map[reflect.Type][]string) []reflect.Type {
	// 创建一个空的reflect.Type切片
	types := make([]reflect.Type, 0)
	// 遍历typeNameMapping中的类型，并添加到types切片中
	for t := range typeNameMapping {
		types = append(types, t)
	}

	// 根据类型的第一个JSON名称对types切片进行排序
	sort.Slice(types, func(i, j int) bool {
		return typeNameMapping[types[i]][0] < typeNameMapping[types[j]][0]
	})
# 返回变量types的值。
```