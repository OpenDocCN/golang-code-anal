# `grype\grype\search\only_qualified_packages.go`

```go
package search

import (
    "fmt"

    "github.com/anchore/grype/grype/distro"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/vulnerability"
)

// onlyQualifiedPackages 函数用于过滤出符合条件的漏洞信息
func onlyQualifiedPackages(d *distro.Distro, p pkg.Package, allVulns []vulnerability.Vulnerability) ([]vulnerability.Vulnerability, error) {
    var vulns []vulnerability.Vulnerability

    // 遍历所有漏洞信息
    for _, vuln := range allVulns {
        isVulnerable := true

        // 遍历漏洞信息中的包限定条件
        for _, q := range vuln.PackageQualifiers {
            // 检查包限定条件是否满足
            v, err := q.Satisfied(d, p)

            // 如果出现错误，返回错误信息
            if err != nil {
                return nil, fmt.Errorf("failed to check package qualifier=%q for distro=%q package=%q: %w", q, d, p, err)
            }

            // 更新漏洞信息是否受影响的状态
            isVulnerable = v
            if !isVulnerable {
                break
            }
        }

        // 如果漏洞信息不受影响，则继续下一个漏洞信息
        if !isVulnerable {
            continue
        }

        // 将受影响的漏洞信息添加到结果列表中
        vulns = append(vulns, vuln)
    }

    // 返回符合条件的漏洞信息列表
    return vulns, nil
}
```