# `grype\grype\pkg\upstream_package.go`

```
package pkg

import (
	"strings" // 导入字符串操作的包
	"github.com/scylladb/go-set/strset" // 导入集合操作的包
	"github.com/anchore/syft/syft/cpe" // 导入CPE包

)

type UpstreamPackage struct {
	Name    string // 包的名称
	Version string // 包的版本
}

func UpstreamPackages(p Package) (pkgs []Package) {
	original := p // 保存原始包信息
	for _, u := range p.Upstreams { // 遍历包的上游依赖
		tmp := original // 保存原始包信息到临时变量
		// 如果名称为空，则跳过当前循环
		if u.Name == "" {
			continue
		}

		// 将临时变量的名称设置为u的名称
		tmp.Name = u.Name
		// 如果u的版本不为空，则将临时变量的版本设置为u的版本
		if u.Version != "" {
			tmp.Version = u.Version
		}
		// 将临时变量的上游设置为空
		tmp.Upstreams = nil

		// 对于每个cpe，用origin替换pkg名称并添加到集合中
		// 创建一个字符串集合来存储cpe的字符串表示
		cpeStrings := strset.New()
		// 遍历临时变量的CPEs
		for _, c := range tmp.CPEs {
			// 如果u的版本不为空，则将c的版本设置为u的版本
			if u.Version != "" {
				c.Version = u.Version
			}
			// 使用u的名称替换c的绑定格式字符串中的pkg名称，并将更新后的字符串添加到cpeStrings集合中
			updatedCPEString := strings.ReplaceAll(c.BindToFmtString(), p.Name, u.Name)
			cpeStrings.Add(updatedCPEString)
		}
		}

		// 遍历集合中的每个条目，将字符串转换为CPE并更新新的CPEs
		var updatedCPEs []cpe.CPE
		for _, cpeString := range cpeStrings.List() {
			// 将字符串转换为CPE对象
			updatedCPE, _ := cpe.New(cpeString)
			// 将更新后的CPE对象添加到更新后的CPEs列表中
			updatedCPEs = append(updatedCPEs, updatedCPE)
		}
		// 将更新后的CPEs列表赋值给tmp结构体的CPEs字段
		tmp.CPEs = updatedCPEs

		// 将tmp结构体添加到pkgs切片中
		pkgs = append(pkgs, tmp)
	}
	// 返回pkgs切片
	return pkgs
}
```