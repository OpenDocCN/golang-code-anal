# `grype\grype\match\explicit_ignores.go`

```
package match

import (
	"github.com/anchore/grype/internal/log"
)

// 定义全局变量，用于存储显式忽略规则
var explicitIgnoreRules []IgnoreRule

// 初始化函数，在包被加载时执行
func init() {
	// 定义结构体，用于存储忽略规则的类型、漏洞和软件包信息
	type ignoreValues struct {
		typ             string
		vulnerabilities []string
		packages        []string
	}

	// 显式忽略规则列表
	var explicitIgnores = []ignoreValues{
		// 基于 https://github.com/anchore/grype/issues/552 的内容，包括对 https://github.com/mergebase/log4j-samples 集合的引用，我们要显式过滤这些内容:
		{
			typ:             "java-archive",
// 定义一个结构体，包含漏洞和软件包信息
{
    typ:             "maven",
    vulnerabilities: []string{"CVE-2021-44228", "CVE-2021-45046", "GHSA-jfh8-c2jp-5v3q", "GHSA-7rjr-3q55-vv33", "CVE-2020-9493", "CVE-2022-23307", "CVE-2023-26464"},
    packages:        []string{"log4j-api", "log4j-slf4j-impl", "log4j-to-slf4j", "log4j-1.2-api", "log4j-detector", "log4j-over-slf4j", "slf4j-log4j12"},
},
// 基于 https://github.com/anchore/grype/issues/558:
{
    typ:             "go-module",
    vulnerabilities: []string{"CVE-2015-5237", "CVE-2021-22570"},
    packages:        []string{"google.golang.org/protobuf"},
},
// 影响 Squiz Matrix，与 matrix ruby gem 没有任何关联
{
    typ:             "gem",
    vulnerabilities: []string{"CVE-2017-14196", "CVE-2017-14197", "CVE-2017-14198", "CVE-2019-19373", "CVE-2019-19374"},
    packages:        []string{"matrix"},
},
// 影响 DeleGate 代理服务器，与 delegate ruby gem 没有任何关联
{
    typ:             "gem",
    vulnerabilities: []string{"CVE-1999-1338", "CVE-2001-1202", "CVE-2002-1781", "CVE-2004-0789", "CVE-2004-2003", "CVE-2005-0036", "CVE-2005-0861", "CVE-2006-2072", "CVE-2015-7556"},
    packages:        []string{"delegate"},
}
		},
		// 影响 Observer 自动发现 PHP/MySQL/SNMP/CDP 基于网络管理系统，与 observer ruby gem 无关
		{
			typ:             "gem",
			vulnerabilities: []string{"CVE-2008-4318"},
			packages:        []string{"observer"},
		},
		// 影响 WeeChat 日志记录插件，与 logger ruby gem 无关
		{
			typ:             "gem",
			vulnerabilities: []string{"CVE-2017-14727"},
			packages:        []string{"logger"},
		},
	}

	for _, ignore := range explicitIgnores {
		for _, vulnerability := range ignore.vulnerabilities {
			for _, packageName := range ignore.packages {
				// 将忽略规则添加到 explicitIgnoreRules 中
				explicitIgnoreRules = append(explicitIgnoreRules, IgnoreRule{
					Vulnerability: vulnerability,
// 定义一个 IgnoreRulePackage 结构体，并设置其属性 Name 和 Type
Package: IgnoreRulePackage{
    Name: packageName,
    Type: ignore.typ,
},

// 调用 ApplyExplicitIgnoreRules 函数，该函数用于过滤符合指定条件的匹配项和 grype 数据库中的匹配项
func ApplyExplicitIgnoreRules(provider ExclusionProvider, matches Matches) (Matches, []IgnoredMatch) {
    // 定义一个 ignoreRules 切片用于存储忽略规则
    var ignoreRules []IgnoreRule
    // 将 explicitIgnoreRules 切片中的元素追加到 ignoreRules 切片中
    ignoreRules = append(ignoreRules, explicitIgnoreRules...)

    // 遍历 matches.Sorted() 中的每个元素
    for _, m := range matches.Sorted() {
        // 从 provider 中获取与漏洞 ID 对应的规则
        r, err := provider.GetRules(m.Vulnerability.ID)

        // 如果获取规则时发生错误，则记录警告日志
        if err != nil {
            log.Warnf("unable to get ignore rules for vuln id=%s", m.Vulnerability.ID)
# 继续执行下一次循环
continue

# 将 r 切片追加到 ignoreRules 切片中
ignoreRules = append(ignoreRules, r...)

# 返回应用了 ignoreRules 的匹配结果
return ApplyIgnoreRules(matches, ignoreRules)
```