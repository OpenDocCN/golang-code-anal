# `grype\grype\version\portage_version.go`

```
// 定义一个名为 version 的包
package version

// 引入需要使用的包
import (
	"fmt" // 格式化输入输出
	"math/big" // 大数运算
	"regexp" // 正则表达式
	"strings" // 字符串操作
)

// 定义一个名为 portageVersion 的结构体
type portageVersion struct {
	version string // 版本号
}

// 创建一个新的 portageVersion 实例
func newPortageVersion(raw string) portageVersion {
	return portageVersion{
		version: raw, // 使用传入的原始字符串作为版本号
	}
}

// 定义一个方法，用于比较版本号
func (v *portageVersion) Compare(other *Version) (int, error) {
// 如果给定的格式不是PortageFormat，则返回-1，并且返回一个错误信息
if other.Format != PortageFormat {
    return -1, fmt.Errorf("unable to compare portage to given format: %s", other.Format)
}

// 如果给定的portageVersion对象为空，则返回-1，并且返回一个错误信息
if other.rich.portVer == nil {
    return -1, fmt.Errorf("given empty portageVersion object")
}

// 调用portageVersion对象的compare方法，比较两个版本号的大小，并返回比较结果
return other.rich.portVer.compare(*v), nil
}

// compare方法用于比较两个portageVersion对象的版本号大小，返回0表示相等，-1表示小于，+1表示大于
func (v portageVersion) compare(v2 portageVersion) int {
    // 如果两个版本号相等，则返回0
    if v.version == v2.version {
        return 0
    }
    // 否则调用comparePortageVersions函数比较两个版本号的大小
    return comparePortageVersions(v.version, v2.version)
}

// 原始的Python实现请参考：
// https://github.com/gentoo/portage/blob/master/lib/portage/versions.py
// 定义正则表达式，用于匹配版本号的各个部分
var (
	versionRegexp = regexp.MustCompile(`(\d+)((\.\d+)*)([a-z]?)((_(pre|p|beta|alpha|rc)\d*)*)(-r(\d+))?`)
	suffixRegexp  = regexp.MustCompile(`^(alpha|beta|rc|pre|p)(\d*)$`)
	suffixValue   = map[string]int{"pre": -2, "p": 0, "alpha": -4, "beta": -3, "rc": -1}
)

// 定义比较 Portage 版本号的函数
//nolint:funlen,gocognit
func comparePortageVersions(a, b string) int {
	// 使用正则表达式匹配版本号的各个部分
	match1 := versionRegexp.FindStringSubmatch(a)
	match2 := versionRegexp.FindStringSubmatch(b)
	// 创建用于比较的大整数列表
	list1 := []*big.Int{big.NewInt(0)}
	list2 := []*big.Int{big.NewInt(0)}
	// 将版本号的主要部分转换为大整数
	list1[0].SetString(match1[1], 10)
	list2[0].SetString(match2[1], 10)
	// 将版本号的次要部分转换为字符串列表
	vlist1 := strings.Split(match1[2], ".")[1:]
	vlist2 := strings.Split(match2[2], ".")[1:]
	// 获取次要部分列表的最大长度
	vlistMaxLen := len(vlist1)
	if len(vlist2) > vlistMaxLen {
		vlistMaxLen = len(vlist2)
	}
	// 遍历 vlistMaxLen 次
	for index := 0; index < vlistMaxLen; index++ {
		// 根据不同情况进行处理
		switch {
		// 如果 vlist1 的长度小于等于 index，则在 list1 中添加一个值为 -1 的大整数
		case len(vlist1) <= index:
			list1 = append(list1, big.NewInt(-1))
			// 创建一个大整数 i，并将 vlist2[index] 转换为十进制后赋值给 i
			i := big.NewInt(0)
			i.SetString(vlist2[index], 10)
			// 在 list2 中添加 i
			list2 = append(list2, i)
		// 如果 vlist2 的长度小于等于 index，则在 list2 中添加一个值为 -1 的大整数
		case len(vlist2) <= index:
			list2 = append(list2, big.NewInt(-1))
			// 创建一个大整数 i，并将 vlist1[index] 转换为十进制后赋值给 i
			i := big.NewInt(0)
			i.SetString(vlist1[index], 10)
			// 在 list1 中添加 i
			list1 = append(list1, i)
		// 如果 vlist1[index] 和 vlist2[index] 都不以 "0" 开头
		case !strings.HasPrefix(vlist1[index], "0") && !strings.HasPrefix(vlist2[index], "0"):
			// 创建一个大整数 i，并将 vlist1[index] 转换为十进制后赋值给 i
			i := big.NewInt(0)
			i.SetString(vlist1[index], 10)
			// 在 list1 中添加 i
			list1 = append(list1, i)
			// 创建一个大整数 j，并将 vlist2[index] 转换为十进制后赋值给 j
			j := big.NewInt(0)
			j.SetString(vlist2[index], 10)
			// 在 list2 中添加 j
			list2 = append(list2, j)
		// 默认情况下，比较两个列表中每个元素的长度，并取最大长度
		maxLen := len(vlist1[index])
		// 如果第二个列表中的元素长度大于最大长度，则更新最大长度
		if len(vlist2[index]) > maxLen {
			maxLen = len(vlist2[index])
		}
		// 如果第一个列表中的元素长度小于最大长度，则在末尾补充0
		if len(vlist1[index]) < maxLen {
			vlist1[index] += strings.Repeat("0", maxLen-len(vlist1[index]))
		}
		// 如果第二个列表中的元素长度小于最大长度，则在末尾补充0
		if len(vlist2[index]) < maxLen {
			vlist2[index] += strings.Repeat("0", maxLen-len(vlist2[index]))
		}
		// 创建一个大整数对象，并将第一个列表中的元素转换为十进制数值
		i := big.NewInt(0)
		i.SetString(vlist1[index], 10)
		// 将转换后的数值添加到列表list1中
		list1 = append(list1, i)
		// 创建一个大整数对象，并将第二个列表中的元素转换为十进制数值
		j := big.NewInt(0)
		j.SetString(vlist2[index], 10)
		// 将转换后的数值添加到列表list2中
		list2 = append(list2, j)
	}
```
以上是对给定代码的注释。
# 如果匹配到的第一个字符串的第四个分组不为空
if len(match1[4]) != 0:
    # 将第四个分组转换为 Unicode 字符数组
    r := []rune(match1[4])
    # 将字符数组中的第一个字符转换为大整数
    i := big.NewInt(int64(r[0]))
    # 将大整数添加到列表1中
    list1 = append(list1, i)

# 如果匹配到的第二个字符串的第四个分组不为空
if len(match2[4]) != 0:
    # 将第四个分组转换为 Unicode 字符数组
    r := []rune(match2[4])
    # 将字符数组中的第一个字符转换为大整数
    i := big.NewInt(int64(r[0]))
    # 将大整数添加到列表2中
    list2 = append(list2, i)

# 计算列表1的长度
maxLen := len(list1)
# 如果列表2的长度大于maxLen，则更新maxLen为列表2的长度
if len(list2) > maxLen:
    maxLen = len(list2)

# 遍历maxLen次
for index := 0; index < maxLen; index++:
    # 如果列表1的长度小于等于index，返回-1
    if len(list1) <= index:
        return -1
		# 如果 list2 的长度小于等于 index，则返回 1
		if len(list2) <= index {
			return 1
		}
		# 比较 list1 和 list2 在 index 处的值
		c := list1[index].Cmp(list2[index])
		# 如果不相等，则返回比较结果
		if c != 0 {
			return c
		}
	}

	# 根据 "_" 分割 match1[5] 字符串，并取第二个元素之后的部分
	slist1 := strings.Split(match1[5], "_")[1:]
	# 根据 "_" 分割 match2[5] 字符串，并取第二个元素之后的部分
	slist2 := strings.Split(match2[5], "_")[1:]
	# 初始化 maxLen 为 slist1 的长度
	maxLen = len(slist1)
	# 如果 slist2 的长度大于 maxLen，则更新 maxLen 为 slist2 的长度
	if len(slist2) > maxLen {
		maxLen = len(slist2)
	}
	# 遍历 slist1 和 slist2，比较它们的元素
	for index := 0; index < maxLen; index++ {
		# 初始化 s1 和 s2 为默认值
		s1 := []string{"p", "-1"}
		s2 := []string{"p", "-1"}
		# 如果 slist1 的长度大于 index，则根据正则表达式匹配 slist1[index] 的值
		if len(slist1) > index {
			s1 = suffixRegexp.FindStringSubmatch(slist1[index])[1:]
		# 如果 s1 的第二个元素为空字符串，则将其赋值为 "0"
		if s1[1] == "" {
			s1[1] = "0"
		}
	}
	# 如果 slist2 的长度大于 index，则执行以下操作
	if len(slist2) > index {
		# 从 slist2 中提取匹配后缀的字符串，并去除第一个元素
		s2 = suffixRegexp.FindStringSubmatch(slist2[index])[1:]
		# 如果 s2 的第二个元素为空字符串，则将其赋值为 "0"
		if s2[1] == "" {
			s2[1] = "0"
		}
	}
	# 如果 s1 的第一个元素不等于 s2 的第一个元素，则执行以下操作
	if s1[0] != s2[0] {
		# 获取 s1 和 s2 的后缀值
		v1 := suffixValue[s1[0]]
		v2 := suffixValue[s2[0]]
		# 如果 v1 大于 v2，则返回 1
		if v1 > v2 {
			return 1
		}
		# 否则返回 -1
		return -1
	}
	# 如果 s1 的第二个元素不等于 s2 的第二个元素，则执行以下操作
	if s1[1] != s2[1] {
# 创建一个新的大整数对象 i，并将字符串 s1[1] 转换为十进制的大整数赋值给 i
i := big.NewInt(0)
i.SetString(s1[1], 10)

# 创建一个新的大整数对象 j，并将字符串 s2[1] 转换为十进制的大整数赋值给 j
j := big.NewInt(0)
j.SetString(s2[1], 10)

# 比较两个大整数对象 i 和 j 的值，将比较结果赋值给变量 c
c := i.Cmp(j)

# 如果比较结果不等于 0，则返回比较结果
if c != 0 {
    return c
}

# 创建一个新的大整数对象 r1，并将字符串 match1[9] 转换为十进制的大整数赋值给 r1
r1 := big.NewInt(0)
if match1[9] != "" {
    r1.SetString(match1[9], 10)
}

# 创建一个新的大整数对象 r2，并将字符串 match2[9] 转换为十进制的大整数赋值给 r2
r2 := big.NewInt(0)
if match2[9] != "" {
    r2.SetString(match2[9], 10)
}
# 比较 r1 和 r2 的值
return r1.Cmp(r2)
```