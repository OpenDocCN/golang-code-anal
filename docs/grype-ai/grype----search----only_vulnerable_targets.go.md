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
    return p.Type == syftPkg.AlpmPkg || p.Type == syftPkg.ApkPkg || p.Type == syftPkg.DebPkg || p.Type == syftPkg.KbPkg || p.Type == syftPkg.PortagePkg || p.Type == syftPkg.RpmPkg  // 判断包类型是否为操作系统包
}

func isUnknownTarget(targetSW string) bool {
    if syftPkg.LanguageByName(targetSW) != syftPkg.UnknownLanguage {  // 判断目标软件是否为未知语言
        return false
    }

    // There are some common target software CPE components which are not currently
    // supported by syft but are signifcant sources of false positives and should be
    // considered known for the purposes of filtering here
    known := map[string]bool{  // 定义包含已知目标软件的映射
        "wordpress":  true,
        "wordpress_": true,
        "joomla":     true,
        "joomla\\!":  true,
        "drupal":     true,
    }

    if _, ok := known[targetSW]; ok {  // 判断目标软件是否为已知目标软件
        return false
    }

    return true
}

// Determines if a vulnerability is an accurate match using the vulnerability's cpes' target software
func onlyVulnerableTargets(p pkg.Package, allVulns []vulnerability.Vulnerability) []vulnerability.Vulnerability {
    var vulns []vulnerability.Vulnerability  // 定义漏洞数组

    // Exclude OS package types from this logic, since they could be embedding any type of ecosystem package
    if isOSPackage(p) {  // 如果是操作系统包，则返回所有漏洞
        return allVulns
    }

    // There are quite a few cases within java where other ecosystem components (particularly javascript packages)
    // are embedded directly within jar files, so we can't yet make this assumption with java as it will cause dropping
    // of valid vulnerabilities that syft has specific logic https://github.com/anchore/syft/blob/main/syft/pkg/cataloger/common/cpe/candidate_by_package_type.go#L48-L75
    // to ensure will be surfaced
    if p.Language == syftPkg.Java {  // 如果是 Java 语言，则返回所有漏洞
        return allVulns
    }
    # 遍历所有漏洞
    for _, vuln := range allVulns {
        # 判断漏洞是否与软件包相关
        isPackageVulnerable := len(vuln.CPEs) == 0
        # 遍历漏洞的CPEs
        for _, cpe := range vuln.CPEs {
            # 获取目标软件
            targetSW := cpe.TargetSW
            # 判断是否与当前语言不匹配且为未知目标软件
            mismatchWithUnknownLanguage := targetSW != string(p.Language) && isUnknownTarget(targetSW)
            # 判断是否为任意软件、不适用软件、当前语言软件或与未知语言不匹配
            if targetSW == wfn.Any || targetSW == wfn.NA || targetSW == string(p.Language) || mismatchWithUnknownLanguage {
                isPackageVulnerable = true
            }
        }

        # 如果软件包不受影响，则继续下一个漏洞
        if !isPackageVulnerable {
            continue
        }

        # 将受影响的漏洞添加到结果列表中
        vulns = append(vulns, vuln)
    }

    # 返回受影响的漏洞列表
    return vulns
# 闭合前面的函数定义
```