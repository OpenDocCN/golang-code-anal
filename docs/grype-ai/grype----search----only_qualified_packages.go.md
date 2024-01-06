# `grype\grype\search\only_qualified_packages.go`

```
package search

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"github.com/anchore/grype/grype/distro"  // 导入 distro 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
)

func onlyQualifiedPackages(d *distro.Distro, p pkg.Package, allVulns []vulnerability.Vulnerability) ([]vulnerability.Vulnerability, error) {
	var vulns []vulnerability.Vulnerability  // 定义一个空的漏洞切片

	for _, vuln := range allVulns {  // 遍历所有漏洞
		isVulnerable := true  // 初始化漏洞状态为 true

		for _, q := range vuln.PackageQualifiers {  // 遍历漏洞的包限定符
			v, err := q.Satisfied(d, p)  // 判断包是否满足漏洞的限定条件

			if err != nil {  // 如果出现错误
# 返回一个空值和一个错误，错误信息包括包限定符、发行版和包的信息，以及错误本身
return nil, fmt.Errorf("failed to check package qualifier=%q for distro=%q package=%q: %w", q, d, p, err)

# 将变量 isVulnerable 设置为 v 的值
isVulnerable = v

# 如果不是易受攻击的，则跳出循环
if !isVulnerable:
    break

# 如果不是易受攻击的，则继续下一次循环
if !isVulnerable:
    continue

# 将 vuln 添加到 vulns 列表中
vulns = append(vulns, vuln)

# 返回 vulns 列表和一个空值
return vulns, nil
```