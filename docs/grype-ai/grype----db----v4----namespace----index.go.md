# `grype\grype\db\v4\namespace\index.go`

```
package namespace

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"strings"  // 导入 strings 包，用于处理字符串

	"github.com/anchore/grype/grype/db/v4/namespace/cpe"  // 导入 cpe 命名空间
	"github.com/anchore/grype/grype/db/v4/namespace/distro"  // 导入 distro 命名空间
	"github.com/anchore/grype/grype/db/v4/namespace/language"  // 导入 language 命名空间
	grypeDistro "github.com/anchore/grype/grype/distro"  // 导入 grypeDistro 命名空间
	"github.com/anchore/grype/internal/log"  // 导入 log 命名空间
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 命名空间
)

type Index struct {
	all         []Namespace  // 定义一个包含所有命名空间的切片
	byLanguage  map[syftPkg.Language][]*language.Namespace  // 定义一个以语言为键，命名空间切片为值的映射
	byDistroKey map[string][]*distro.Namespace  // 定义一个以发行版键为键，命名空间切片为值的映射
	cpe         []*cpe.Namespace  // 定义一个 cpe 命名空间切片
}
# 从字符串数组中创建索引对象，返回索引对象和错误信息
func FromStrings(namespaces []string) (*Index, error) {
    # 创建一个空的命名空间切片
    all := make([]Namespace, 0)
    # 创建一个按语言分组的命名空间映射
    byLanguage := make(map[syftPkg.Language][]*language.Namespace)
    # 创建一个按发行版密钥分组的命名空间映射
    byDistroKey := make(map[string][]*distro.Namespace)
    # 创建一个 CPE 命名空间切片
    cpeNamespaces := make([]*cpe.Namespace, 0)

    # 遍历命名空间数组
    for _, n := range namespaces {
        # 从字符串创建命名空间对象
        ns, err := FromString(n)

        # 如果出现错误，记录日志并继续下一个命名空间
        if err != nil {
            log.Warnf("unable to create namespace object from namespace=%s: %+v", n, err)
            continue
        }

        # 将命名空间对象添加到 all 切片中
        all = append(all, ns)

        # 根据命名空间对象的类型进行不同的处理
        switch nsObj := ns.(type) {
            case *language.Namespace:
                # 获取命名空间对象的语言信息
                l := nsObj.Language()
# 如果语言在byLanguage中不存在，则创建一个空的语言命名空间切片
if _, ok := byLanguage[l]; !ok {
    byLanguage[l] = make([]*language.Namespace, 0)
}

# 将当前命名空间对象添加到对应语言的命名空间切片中
byLanguage[l] = append(byLanguage[l], nsObj)

# 如果命名空间对象是distro.Namespace类型
case *distro.Namespace:
    # 根据DistroType和Version组成一个唯一的键
    distroKey := fmt.Sprintf("%s:%s", nsObj.DistroType(), nsObj.Version())
    # 如果该键在byDistroKey中不存在，则创建一个空的distro.Namespace切片
    if _, ok := byDistroKey[distroKey]; !ok {
        byDistroKey[distroKey] = make([]*distro.Namespace, 0)
    }
    # 将当前命名空间对象添加到对应键的distro.Namespace切片中
    byDistroKey[distroKey] = append(byDistroKey[distroKey], nsObj)

# 如果命名空间对象是cpe.Namespace类型
case *cpe.Namespace:
    # 将当前命名空间对象添加到cpeNamespaces切片中
    cpeNamespaces = append(cpeNamespaces, nsObj)

# 如果命名空间对象不是上述类型
default:
    # 记录警告日志，表示无法索引该命名空间
    log.Warnf("unable to index namespace=%s", n)
    # 继续处理下一个命名空间对象
    continue
}
# 返回一个包含所有索引信息的Index对象，以及一个空的错误对象
return &Index{
    all:         all,
    byLanguage:  byLanguage,
    byDistroKey: byDistroKey,
    cpe:         cpeNamespaces,
}, nil

# 根据语言返回对应的命名空间列表，如果不存在则返回空列表
func (i *Index) NamespacesForLanguage(l syftPkg.Language) []*language.Namespace {
    if _, ok := i.byLanguage[l]; ok {
        return i.byLanguage[l]
    }
    return nil
}

# 根据发行版返回对应的命名空间列表，如果发行版为空则返回空列表
func (i *Index) NamespacesForDistro(d *grypeDistro.Distro) []*distro.Namespace {
    if d == nil {
        return nil
    }
```

# 如果发行版正在滚动更新，则执行以下操作
if d.IsRolling():
    # 生成滚动更新的发行版键
    distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), "rolling")
    # 如果键存在于映射中，则返回对应的值
    if v, ok := i.byDistroKey[distroKey]; ok:
        return v

# 初始化版本段列表
var versionSegments []int
# 如果发行版版本不为空，则获取版本段列表
if d.Version != nil:
    versionSegments = d.Version.Segments()

# 如果版本段列表不为空
if len(versionSegments) > 0:
    # 尝试直接匹配发行版全名和版本
    distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), d.FullVersion())
    # 如果键存在于映射中，则返回对应的值
    if v, ok := i.byDistroKey[distroKey]; ok:
        return v
		if len(versionSegments) == 3 {
			// 如果版本号由3个部分组成
			// 尝试只使用前两个版本组件
			distroKey = fmt.Sprintf("%s:%d.%d", strings.ToLower(d.Type.String()), versionSegments[0], versionSegments[1])
			if v, ok := i.byDistroKey[distroKey]; ok {
				return v
			}

			// 尝试只使用主要版本组件
			distroKey = fmt.Sprintf("%s:%d", strings.ToLower(d.Type.String()), versionSegments[0])
			if v, ok := i.byDistroKey[distroKey]; ok {
				return v
			}
		}

		// 回退到从 https://github.com/anchore/enterprise/blob/eb71bc6686b9f4c92347a4e95bec828cee879197/anchore_engine/services/policy_engine/__init__.py#L127-L140 推导出的手动映射逻辑
		switch d.Type {
		case grypeDistro.CentOS, grypeDistro.RedHat, grypeDistro.Fedora, grypeDistro.RockyLinux, grypeDistro.AlmaLinux, grypeDistro.Gentoo:
			// TODO: 没有将 Fedora 版本映射到 RHEL 最新版本（只有名称）
# 根据格式化字符串创建 distroKey，格式为 distro:version
distroKey = fmt.Sprintf("%s:%d", strings.ToLower(string(grypeDistro.RedHat)), versionSegments[0])
# 检查 distroKey 是否存在于 byDistroKey 中，如果存在则返回对应的值
if v, ok := i.byDistroKey[distroKey]; ok:
    return v
# 遍历 CPENamespaces 返回 CPE 命名空间
return nil
}

# 返回 Index 对象的 CPE 命名空间
func (i *Index) CPENamespaces() []*cpe.Namespace:
    return i.cpe
```