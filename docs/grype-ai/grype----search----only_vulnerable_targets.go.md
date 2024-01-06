# `grype\grype\search\only_vulnerable_targets.go`

```
package search

import (
	"github.com/facebookincubator/nvdtools/wfn"  // 导入 wfn 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func isOSPackage(p pkg.Package) bool {
	return p.Type == syftPkg.AlpmPkg || p.Type == syftPkg.ApkPkg || p.Type == syftPkg.DebPkg || p.Type == syftPkg.KbPkg || p.Type == syftPkg.PortagePkg || p.Type == syftPkg.RpmPkg
}

func isUnknownTarget(targetSW string) bool {
	if syftPkg.LanguageByName(targetSW) != syftPkg.UnknownLanguage {  // 判断目标软件是否为未知语言
		return false
	}

	// There are some common target software CPE components which are not currently
```

// known 是一个包含已知目标软件的映射，用于过滤虚假阳性
known := map[string]bool{
    "wordpress":  true,
    "wordpress_": true,
    "joomla":     true,
    "joomla\\!":  true,
    "drupal":     true,
}

// 如果目标软件在 known 中，则返回 false
if _, ok := known[targetSW]; ok {
    return false
}

// 否则返回 true
return true
}

// 根据漏洞的 cpes 的目标软件确定漏洞是否准确匹配
func onlyVulnerableTargets(p pkg.Package, allVulns []vulnerability.Vulnerability) []vulnerability.Vulnerability {
    var vulns []vulnerability.Vulnerability
// 排除操作系统包类型，因为它们可能包含任何类型的生态系统包
if isOSPackage(p) {
    return allVulns
}

// 在 Java 中有许多情况下，其他生态系统组件（特别是 JavaScript 包）直接嵌入在 jar 文件中，因此我们不能做出这种假设，因为这会导致放弃有效的漏洞，syft 有特定的逻辑来确保这些漏洞会被发现
if p.Language == syftPkg.Java {
    return allVulns
}

// 遍历所有漏洞
for _, vuln := range allVulns {
    // 判断包是否有漏洞
    isPackageVulnerable := len(vuln.CPEs) == 0
    // 遍历漏洞的 CPE
    for _, cpe := range vuln.CPEs {
        targetSW := cpe.TargetSW
        // 判断是否与未知语言不匹配
        mismatchWithUnknownLanguage := targetSW != string(p.Language) && isUnknownTarget(targetSW)
        // 判断是否匹配任何或未知目标软件，或者与包的语言不匹配
        if targetSW == wfn.Any || targetSW == wfn.NA || targetSW == string(p.Language) || mismatchWithUnknownLanguage {
# 初始化变量 isPackageVulnerable 为 true
isPackageVulnerable = true

# 遍历每个漏洞检测结果
for _, result := range results {
    # 如果检测结果中存在漏洞
    if result.Vulnerable {
        # 将 isPackageVulnerable 设置为 true
        isPackageVulnerable = true
    }
}

# 如果 isPackageVulnerable 为 false，则跳过当前循环
if !isPackageVulnerable {
    continue
}

# 将漏洞信息添加到 vulns 列表中
vulns = append(vulns, vuln)

# 返回漏洞列表
return vulns
```