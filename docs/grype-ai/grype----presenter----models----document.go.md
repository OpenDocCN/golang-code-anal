# `grype\grype\presenter\models\document.go`

```
package models

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"sort"  // 导入 sort 包，用于对数据进行排序
	"time"  // 导入 time 包，用于处理时间相关操作

	"github.com/anchore/clio"  // 导入 anchore/clio 包
	"github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
)

// Document represents the JSON document to be presented
type Document struct {
	Matches        []Match        `json:"matches"`  // 表示匹配项的数组
	IgnoredMatches []IgnoredMatch `json:"ignoredMatches,omitempty"`  // 表示被忽略的匹配项的数组
	Source         *source        `json:"source"`  // 表示源信息的指针
	Distro         distribution   `json:"distro"`  // 表示发行版信息
	Descriptor     descriptor     `json:"descriptor"`  // 表示描述信息
```

注释：以上代码是一个 Go 语言的结构体定义，包含了对不同包的引用和对结构体字段的定义。
// NewDocument创建并填充一个新的Document结构，表示填充的JSON文档。
// 获取本地时间戳并转换为字节流，如果出现错误则返回空的Document结构和错误信息
timestamp, timestampErr := time.Now().Local().MarshalText()
if timestampErr != nil {
    return Document{}, timestampErr
}

// 我们必须预先分配findings以确保JSON文档在找不到匹配项时不显示"null"
var findings = make([]Match, 0)
// 遍历排序后的匹配项，查找对应的包并创建匹配模型
for _, m := range matches.Sorted() {
    p := pkg.ByID(m.Package.ID, packages)
    if p == nil {
        return Document{}, fmt.Errorf("unable to find package in collection: %+v", p)
    }

    matchModel, err := newMatch(m, *p, metadataProvider)
    if err != nil {
        return Document{}, err
    }
}
	# 将匹配结果追加到findings切片中
	findings = append(findings, *matchModel)
	# 对findings切片进行排序
	sort.Sort(MatchSort(findings))

	# 声明一个指向source结构体的指针变量src
	var src *source
	# 如果上下文中存在源，则创建一个新的source结构体，并将其赋值给theSrc变量
	if context.Source != nil {
		theSrc, err := newSource(*context.Source)
		# 如果创建source结构体时发生错误，则返回空的Document结构体和错误信息
		if err != nil {
			return Document{}, err
		}
		# 将theSrc的地址赋值给src指针变量
		src = &theSrc
	}

	# 声明一个空的ignoredMatchModels切片
	var ignoredMatchModels []IgnoredMatch
	# 遍历ignoredMatches切片中的每个元素m
	for _, m := range ignoredMatches {
		# 根据m.Package.ID和packages查找对应的包p
		p := pkg.ByID(m.Package.ID, packages)
		# 如果找不到对应的包，则返回空的Document结构体和包查找错误的信息
		if p == nil {
			return Document{}, fmt.Errorf("unable to find package in collection: %+v", p)
		}

		// 根据匹配结果创建匹配模型
		matchModel, err := newMatch(m.Match, *p, metadataProvider)
		// 如果创建匹配模型出错，则返回错误
		if err != nil {
			return Document{}, err
		}

		// 创建忽略的匹配对象
		ignoredMatch := IgnoredMatch{
			Match:              *matchModel,
			AppliedIgnoreRules: mapIgnoreRules(m.AppliedIgnoreRules),
		}
		// 将忽略的匹配对象添加到忽略匹配模型列表中
		ignoredMatchModels = append(ignoredMatchModels, ignoredMatch)
	}

	// 返回文档对象，包括匹配结果、忽略的匹配模型、源、分布和描述符
	return Document{
		Matches:        findings,
		IgnoredMatches: ignoredMatchModels,
		Source:         src,
		Distro:         newDistribution(context.Distro),
		Descriptor: descriptor{
# 设置名称字段为id.Name
Name: id.Name,
# 设置版本字段为id.Version
Version: id.Version,
# 设置配置字段为appConfig
Configuration: appConfig,
# 设置漏洞数据库状态字段为dbStatus
VulnerabilityDBStatus: dbStatus,
# 设置时间戳字段为timestamp的字符串形式
Timestamp: string(timestamp),
```