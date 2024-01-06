# `grype\grype\db\v5\namespace\index.go`

```
package namespace

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"regexp"  // 导入 regexp 包，用于正则表达式操作
	"strings"  // 导入 strings 包，用于字符串操作

	"github.com/anchore/grype/grype/db/v5/namespace/cpe"  // 导入 cpe 包
	"github.com/anchore/grype/grype/db/v5/namespace/distro"  // 导入 distro 包
	"github.com/anchore/grype/grype/db/v5/namespace/language"  // 导入 language 包
	grypeDistro "github.com/anchore/grype/grype/distro"  // 导入 grypeDistro 包，并重命名为 grypeDistro
	"github.com/anchore/grype/internal/log"  // 导入 log 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包，并重命名为 syftPkg
)

var alpineVersionRegularExpression = regexp.MustCompile(`^(\d+)\.(\d+)\.(\d+)$`)  // 定义 alpineVersionRegularExpression 变量，用于匹配 Alpine 版本号的正则表达式

type Index struct {
	all         []Namespace  // 定义 Index 结构体，包含 all 字段，类型为 Namespace 切片
	byLanguage  map[syftPkg.Language][]*language.Namespace  // 定义 Index 结构体，包含 byLanguage 字段，类型为语言到语言命名空间指针切片的映射
```

	// 创建一个映射，将字符串映射到一个包含多个 distro.Namespace 指针的切片
	byDistroKey map[string][]*distro.Namespace
	// 创建一个包含多个 cpe.Namespace 指针的切片
	cpe         []*cpe.Namespace
}

// 从字符串数组创建 Index 对象的函数
func FromStrings(namespaces []string) (*Index, error) {
	// 创建一个空的 Namespace 切片
	all := make([]Namespace, 0)
	// 创建一个映射，将 syftPkg.Language 映射到一个包含多个 language.Namespace 指针的切片
	byLanguage := make(map[syftPkg.Language][]*language.Namespace)
	// 创建一个映射，将字符串映射到一个包含多个 distro.Namespace 指针的切片
	byDistroKey := make(map[string][]*distro.Namespace)
	// 创建一个空的 cpe.Namespace 指针切片
	cpeNamespaces := make([]*cpe.Namespace, 0)

	// 遍历传入的 namespaces 切片
	for _, n := range namespaces {
		// 从字符串创建一个 Namespace 对象
		ns, err := FromString(n)

		// 如果出现错误
		if err != nil {
			// 记录警告日志
			log.Warnf("unable to create namespace object from namespace=%s: %+v", n, err)
			// 继续下一次循环
			continue
		}

		// 将创建的 Namespace 对象添加到 all 切片中
		all = append(all, ns)
		// 判断ns的类型并根据不同类型进行不同的处理
		switch nsObj := ns.(type) {
		// 如果是language.Namespace类型
		case *language.Namespace:
			// 获取语言
			l := nsObj.Language()
			// 如果byLanguage中没有该语言的键，则创建一个空的切片
			if _, ok := byLanguage[l]; !ok {
				byLanguage[l] = make([]*language.Namespace, 0)
			}
			// 将nsObj添加到byLanguage中对应语言的切片中
			byLanguage[l] = append(byLanguage[l], nsObj)
		// 如果是distro.Namespace类型
		case *distro.Namespace:
			// 获取distroKey
			distroKey := fmt.Sprintf("%s:%s", nsObj.DistroType(), nsObj.Version())
			// 如果byDistroKey中没有该distroKey的键，则创建一个空的切片
			if _, ok := byDistroKey[distroKey]; !ok {
				byDistroKey[distroKey] = make([]*distro.Namespace, 0)
			}
			// 将nsObj添加到byDistroKey中对应distroKey的切片中
			byDistroKey[distroKey] = append(byDistroKey[distroKey], nsObj)
		// 如果是cpe.Namespace类型
		case *cpe.Namespace:
			// 将nsObj添加到cpeNamespaces切片中
			cpeNamespaces = append(cpeNamespaces, nsObj)
		// 如果是其他类型
		default:
			// 输出警告信息
			log.Warnf("unable to index namespace=%s", n)
			// 继续下一次循环
			continue
// 返回一个指向Index结构的指针，包含all、byLanguage、byDistroKey和cpe字段的值，以及一个空的错误值
return &Index{
    all:         all,
    byLanguage:  byLanguage,
    byDistroKey: byDistroKey,
    cpe:         cpeNamespaces,
}, nil
}

// 返回指定语言的命名空间列表，如果找不到则返回空
func (i *Index) NamespacesForLanguage(l syftPkg.Language) []*language.Namespace {
    // 检查指定语言是否存在于byLanguage字段中，如果存在则返回对应的命名空间列表
    if _, ok := i.byLanguage[l]; ok {
        return i.byLanguage[l]
    }

    // 如果指定语言不存在，则返回空
    return nil
}

// 禁止对函数长度和认知复杂度进行lint检查
//nolint:funlen,gocognit
// 根据发行版获取命名空间列表
func (i *Index) NamespacesForDistro(d *grypeDistro.Distro) []*distro.Namespace {
	// 如果发行版为空，返回空列表
	if d == nil {
		return nil
	}

	// 如果发行版是滚动更新的
	if d.IsRolling() {
		// 构建发行版键值
		distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), "rolling")
		// 如果存在对应的命名空间列表，返回该列表
		if v, ok := i.byDistroKey[distroKey]; ok {
			return v
		}
	}

	// 初始化版本号段列表
	var versionSegments []int
	// 如果发行版有版本号，获取版本号段列表
	if d.Version != nil {
		versionSegments = d.Version.Segments()
	}

	// 如果版本号段列表不为空
	if len(versionSegments) > 0 {
		// Alpine 是一个特殊情况，因为我们只能匹配 x.y.z
		// 之后的 x.y 和 x 都是有效的命名空间选择
		// 如果发现是 Alpine 发行版，则尝试获取对应的命名空间
		if d.Type == grypeDistro.Alpine {
			if v := getAlpineNamespace(i, d, versionSegments); v != nil {
				return v
			}
		}

		// 尝试直接匹配发行版的完整名称和版本号
		distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), d.FullVersion())

		// 如果找到匹配的键值对，则返回对应的值
		if v, ok := i.byDistroKey[distroKey]; ok {
			return v
		}

		// 如果版本号由三个部分组成
		if len(versionSegments) == 3 {
			// 尝试只使用前两个版本组件进行匹配
			distroKey = fmt.Sprintf("%s:%d.%d", strings.ToLower(d.Type.String()), versionSegments[0], versionSegments[1])
			// 如果找到匹配的键值对，则返回对应的值
			if v, ok := i.byDistroKey[distroKey]; ok {
				return v
			}
		}
		// 尝试仅使用主要版本组件
		distroKey = fmt.Sprintf("%s:%d", strings.ToLower(d.Type.String()), versionSegments[0])
		// 检查是否存在以 distroKey 为键的值，如果存在则返回该值
		if v, ok := i.byDistroKey[distroKey]; ok {
			return v
		}
	}

	// 从派生的手动映射逻辑中回退
	// https://github.com/anchore/enterprise/blob/eb71bc6686b9f4c92347a4e95bec828cee879197/anchore_engine/services/policy_engine/__init__.py#L127-L140
	switch d.Type {
	case grypeDistro.CentOS, grypeDistro.RedHat, grypeDistro.Fedora, grypeDistro.RockyLinux, grypeDistro.AlmaLinux, grypeDistro.Gentoo:
		// TODO: 没有将 Fedora 版本映射到 RHEL 最新版本（只有名称）
		distroKey = fmt.Sprintf("%s:%d", strings.ToLower(string(grypeDistro.RedHat)), versionSegments[0])
		// 检查是否存在以 distroKey 为键的值，如果存在则返回该值
		if v, ok := i.byDistroKey[distroKey]; ok {
			return v
		}
	}

	// 如果没有找到版本段，回退到 alpine:edge
// 如果版本段为空并且类型为Alpine，则将alpine:edge标记为alpine-x.x_alphaYYYYMMDD
if versionSegments == nil && d.Type == grypeDistro.Alpine {
    // 构建Alpine的distroKey
    distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), "edge")
    // 如果存在对应的distroKey，则返回对应的值
    if v, ok := i.byDistroKey[distroKey]; ok {
        return v
    }
}

// 如果版本段为空并且类型为Debian并且RawVersion为unstable
if versionSegments == nil && d.Type == grypeDistro.Debian && d.RawVersion == "unstable" {
    // 构建Debian的distroKey
    distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), "unstable")
    // 如果存在对应的distroKey，则返回对应的值
    if v, ok := i.byDistroKey[distroKey]; ok {
        return v
    }
}

// 默认返回空值
return nil
}

// 获取Alpine的命名空间
func getAlpineNamespace(i *Index, d *grypeDistro.Distro, versionSegments []int) []*distro.Namespace {
    // 检查distro版本是否匹配x.y.z
# 如果 alpineVersionRegularExpression 匹配了 d.RawVersion
if alpineVersionRegularExpression.MatchString(d.RawVersion):
    # 获取版本号的前两个组件
    # TODO: 我们是否应该在数据库生成时更新命名空间，以匹配这里的 x.y.z？
    distroKey := fmt.Sprintf("%s:%d.%d", strings.ToLower(d.Type.String()), versionSegments[0], versionSegments[1])
    # 如果 distroKey 在 i.byDistroKey 中存在，则返回对应的值
    if v, ok := i.byDistroKey[distroKey]; ok:
        return v

# 如果版本号不匹配 x.y.z，则认为是 edge 版本
# 在这种情况下，它可能会有 - 或 _ 以及 alpha、beta 等
# https://github.com/anchore/grype/issues/964#issuecomment-1290888755
distroKey := fmt.Sprintf("%s:%s", strings.ToLower(d.Type.String()), "edge")
# 如果 distroKey 在 i.byDistroKey 中存在，则返回对应的值
if v, ok := i.byDistroKey[distroKey]; ok:
    return v

# 如果以上条件都不满足，则返回空值
return nil
# 定义一个方法，用于返回Index结构体中的cpe命名空间列表
func (i *Index) CPENamespaces() []*cpe.Namespace {
    # 返回Index结构体中的cpe命名空间列表
    return i.cpe
}
```