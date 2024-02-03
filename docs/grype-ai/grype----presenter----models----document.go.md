# `grype\grype\presenter\models\document.go`

```go
package models

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "sort" // 导入 sort 包，用于对切片进行排序
    "time" // 导入 time 包，用于处理时间

    "github.com/anchore/clio" // 导入 anchore/clio 包
    "github.com/anchore/grype/grype/match" // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/pkg" // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/vulnerability" // 导入 anchore/grype/grype/vulnerability 包
)

// Document represents the JSON document to be presented
type Document struct {
    Matches        []Match        `json:"matches"` // 表示匹配的结果
    IgnoredMatches []IgnoredMatch `json:"ignoredMatches,omitempty"` // 表示被忽略的匹配结果
    Source         *source        `json:"source"` // 表示来源
    Distro         distribution   `json:"distro"` // 表示发行版
    Descriptor     descriptor     `json:"descriptor"` // 表示描述符
}

// NewDocument creates and populates a new Document struct, representing the populated JSON document.
func NewDocument(id clio.Identification, packages []pkg.Package, context pkg.Context, matches match.Matches, ignoredMatches []match.IgnoredMatch, metadataProvider vulnerability.MetadataProvider, appConfig interface{}, dbStatus interface{}) (Document, error) {
    timestamp, timestampErr := time.Now().Local().MarshalText() // 获取当前时间并转换为本地时间格式
    if timestampErr != nil {
        return Document{}, timestampErr // 如果获取时间出错，则返回错误
    }

    // we must preallocate the findings to ensure the JSON document does not show "null" when no matches are found
    var findings = make([]Match, 0) // 预先分配 findings 切片，以确保在没有匹配结果时 JSON 文档不显示 "null"
    for _, m := range matches.Sorted() { // 遍历排序后的匹配结果
        p := pkg.ByID(m.Package.ID, packages) // 根据 ID 在包集合中查找包
        if p == nil {
            return Document{}, fmt.Errorf("unable to find package in collection: %+v", p) // 如果找不到包，则返回错误
        }

        matchModel, err := newMatch(m, *p, metadataProvider) // 创建匹配模型
        if err != nil {
            return Document{}, err // 如果创建匹配模型出错，则返回错误
        }

        findings = append(findings, *matchModel) // 将匹配模型添加到 findings 切片中
    }

    sort.Sort(MatchSort(findings)) // 对 findings 切片进行排序

    var src *source // 声明 source 类型的变量
    if context.Source != nil { // 如果上下文中的来源不为空
        theSrc, err := newSource(*context.Source) // 创建新的来源
        if err != nil {
            return Document{}, err // 如果创建来源出错，则返回错误
        }
        src = &theSrc // 将创建的来源赋值给 src
    }

    var ignoredMatchModels []IgnoredMatch // 声明 ignoredMatchModels 切片
    # 遍历被忽略的匹配列表
    for _, m := range ignoredMatches:
        # 根据匹配的包ID在包集合中查找对应的包
        p := pkg.ByID(m.Package.ID, packages)
        # 如果找不到对应的包，则返回错误
        if p == nil:
            return Document{}, fmt.Errorf("unable to find package in collection: %+v", p)

        # 根据匹配、包和元数据提供者创建新的匹配模型
        matchModel, err := newMatch(m.Match, *p, metadataProvider)
        # 如果创建匹配模型时出现错误，则返回错误
        if err != nil:
            return Document{}, err

        # 创建被忽略的匹配对象
        ignoredMatch := IgnoredMatch{
            Match:              *matchModel,
            AppliedIgnoreRules: mapIgnoreRules(m.AppliedIgnoreRules),
        }
        # 将被忽略的匹配对象添加到被忽略的匹配模型列表中
        ignoredMatchModels = append(ignoredMatchModels, ignoredMatch)
    # 返回包含匹配、被忽略的匹配、源、发行版、描述符的文档对象和空错误
    return Document{
        Matches:        findings,
        IgnoredMatches: ignoredMatchModels,
        Source:         src,
        Distro:         newDistribution(context.Distro),
        Descriptor: descriptor{
            Name:                  id.Name,
            Version:               id.Version,
            Configuration:         appConfig,
            VulnerabilityDBStatus: dbStatus,
            Timestamp:             string(timestamp),
        },
    }, nil
# 闭合前面的函数定义
```