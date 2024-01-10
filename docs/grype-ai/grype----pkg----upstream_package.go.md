# `grype\grype\pkg\upstream_package.go`

```
package pkg

import (
    "strings" // 导入字符串操作包
    "github.com/scylladb/go-set/strset" // 导入集合操作包
    "github.com/anchore/syft/syft/cpe" // 导入CPE包
)

type UpstreamPackage struct {
    Name    string // 包名
    Version string // 包的版本
}

func UpstreamPackages(p Package) (pkgs []Package) {
    original := p // 保存原始包信息
    for _, u := range p.Upstreams { // 遍历上游包列表
        tmp := original // 保存原始包信息的副本

        if u.Name == "" { // 如果上游包名为空，则跳过
            continue
        }

        tmp.Name = u.Name // 更新包名为上游包名
        if u.Version != "" { // 如果上游包版本不为空
            tmp.Version = u.Version // 更新包版本为上游包版本
        }
        tmp.Upstreams = nil // 清空上游包列表

        // 遍历每个CPE，将包名替换为原始包名并添加到集合中
        cpeStrings := strset.New()
        for _, c := range tmp.CPEs {
            if u.Version != "" { // 如果上游包版本不为空
                c.Version = u.Version // 更新CPE版本为上游包版本
            }

            updatedCPEString := strings.ReplaceAll(c.BindToFmtString(), p.Name, u.Name) // 替换包名并生成更新后的CPE字符串

            cpeStrings.Add(updatedCPEString) // 将更新后的CPE字符串添加到集合中
        }

        // 遍历集合中的每个条目，将字符串转换为CPE并更新新的CPE列表
        var updatedCPEs []cpe.CPE
        for _, cpeString := range cpeStrings.List() {
            updatedCPE, _ := cpe.New(cpeString) // 将字符串转换为CPE对象
            updatedCPEs = append(updatedCPEs, updatedCPE) // 将新的CPE对象添加到列表中
        }
        tmp.CPEs = updatedCPEs // 更新包的CPE列表为新的CPE列表

        pkgs = append(pkgs, tmp) // 将更新后的包信息添加到结果列表中
    }
    return pkgs // 返回更新后的包列表
}
```