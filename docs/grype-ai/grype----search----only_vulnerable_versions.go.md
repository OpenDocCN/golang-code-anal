# `grype\grype\search\only_vulnerable_versions.go`

```
package search

import (
	"errors"  // 导入标准库中的 errors 包，用于处理错误
	"fmt"      // 导入标准库中的 fmt 包，用于格式化输出

	"github.com/anchore/grype/grype/version"  // 导入版本相关的自定义包
	"github.com/anchore/grype/grype/vulnerability"  // 导入漏洞相关的自定义包
	"github.com/anchore/grype/internal/log"  // 导入日志相关的自定义包
)

func onlyVulnerableVersions(verObj *version.Version, allVulns []vulnerability.Vulnerability) ([]vulnerability.Vulnerability, error) {
	var vulns []vulnerability.Vulnerability  // 声明一个漏洞数组

	for _, vuln := range allVulns {  // 遍历所有漏洞
		isPackageVulnerable, err := vuln.Constraint.Satisfied(verObj)  // 检查给定版本是否满足漏洞的约束条件
		if err != nil {  // 如果发生错误
			var e *version.NonFatalConstraintError  // 声明一个非致命约束错误
			if errors.As(err, &e) {  // 如果错误是非致命约束错误
				log.Warn(e)  // 记录警告日志
		} else {
			// 如果检查约束失败，则返回错误信息
			return nil, fmt.Errorf("failed to check constraint=%q version=%q: %w", vuln.Constraint, verObj, err)
		}
	}

	// 如果软件包不受影响，则继续循环
	if !isPackageVulnerable {
		continue
	}

	// 将受影响的漏洞添加到漏洞列表中
	vulns = append(vulns, vuln)
}

// 返回漏洞列表和空错误
return vulns, nil
```