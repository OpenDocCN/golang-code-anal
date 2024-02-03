# `grype\grype\search\only_vulnerable_versions.go`

```go
package search

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"     // 导入 fmt 包，用于格式化输出

    "github.com/anchore/grype/grype/version"  // 导入版本相关的包
    "github.com/anchore/grype/grype/vulnerability"  // 导入漏洞相关的包
    "github.com/anchore/grype/internal/log"  // 导入日志相关的包
)

func onlyVulnerableVersions(verObj *version.Version, allVulns []vulnerability.Vulnerability) ([]vulnerability.Vulnerability, error) {
    var vulns []vulnerability.Vulnerability  // 定义漏洞数组

    for _, vuln := range allVulns {  // 遍历所有漏洞
        isPackageVulnerable, err := vuln.Constraint.Satisfied(verObj)  // 检查漏洞是否满足版本约束
        if err != nil {  // 如果发生错误
            var e *version.NonFatalConstraintError  // 定义非致命约束错误
            if errors.As(err, &e) {  // 如果错误是非致命约束错误
                log.Warn(e)  // 记录警告日志
            } else {
                return nil, fmt.Errorf("failed to check constraint=%q version=%q: %w", vuln.Constraint, verObj, err)  // 返回错误信息
            }
        }

        if !isPackageVulnerable {  // 如果软件包不受影响
            continue  // 继续下一次循环
        }

        vulns = append(vulns, vuln)  // 将受影响的漏洞添加到漏洞数组中
    }

    return vulns, nil  // 返回漏洞数组和空错误
}
```