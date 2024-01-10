# `grype\grype\version\portage_version.go`

```
package version

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "math/big"  // 导入 math/big 包，用于处理大整数
    "regexp"  // 导入 regexp 包，用于正则表达式匹配
    "strings"  // 导入 strings 包，用于字符串操作
)

type portageVersion struct {
    version string  // 定义 portageVersion 结构体，包含 version 字段
}

func newPortageVersion(raw string) portageVersion {
    return portageVersion{  // 返回一个新的 portageVersion 对象
        version: raw,  // 将传入的 raw 字符串赋值给 version 字段
    }
}

func (v *portageVersion) Compare(other *Version) (int, error) {
    if other.Format != PortageFormat {  // 如果传入的 Version 对象的 Format 不是 PortageFormat
        return -1, fmt.Errorf("unable to compare portage to given format: %s", other.Format)  // 返回错误信息
    }
    if other.rich.portVer == nil {  // 如果传入的 portageVersion 对象为空
        return -1, fmt.Errorf("given empty portageVersion object")  // 返回错误信息
    }

    return other.rich.portVer.compare(*v), nil  // 调用 portageVersion 对象的 compare 方法进行比较
}

// Compare returns 0 if v == v2, -1 if v < v2, and +1 if v > v2.
func (v portageVersion) compare(v2 portageVersion) int {
    if v.version == v2.version {  // 如果两个 portageVersion 对象的 version 字段相等
        return 0  // 返回 0
    }
    return comparePortageVersions(v.version, v2.version)  // 调用 comparePortageVersions 函数进行比较
}

// For the original python implementation, see:
// https://github.com/gentoo/portage/blob/master/lib/portage/versions.py
var (
    versionRegexp = regexp.MustCompile(`(\d+)((\.\d+)*)([a-z]?)((_(pre|p|beta|alpha|rc)\d*)*)(-r(\d+))?`)  // 定义 versionRegexp 正则表达式
    suffixRegexp  = regexp.MustCompile(`^(alpha|beta|rc|pre|p)(\d*)$`)  // 定义 suffixRegexp 正则表达式
    suffixValue   = map[string]int{"pre": -2, "p": 0, "alpha": -4, "beta": -3, "rc": -1}  // 定义 suffixValue 映射
)

//nolint:funlen,gocognit
func comparePortageVersions(a, b string) int {
    match1 := versionRegexp.FindStringSubmatch(a)  // 使用 versionRegexp 正则表达式匹配字符串 a
    match2 := versionRegexp.FindStringSubmatch(b)  // 使用 versionRegexp 正则表达式匹配字符串 b
    list1 := []*big.Int{big.NewInt(0)}  // 创建 big.Int 类型的切片 list1，并初始化为 0
    list2 := []*big.Int{big.NewInt(0)}  // 创建 big.Int 类型的切片 list2，并初始化为 0
    list1[0].SetString(match1[1], 10)  // 将匹配结果中的第一个子串转换为十进制数，并赋值给 list1[0]
    list2[0].SetString(match2[1], 10)  // 将匹配结果中的第一个子串转换为十进制数，并赋值给 list2[0]
    vlist1 := strings.Split(match1[2], ".")[1:]  // 使用 "." 分割匹配结果中的第二个子串，并取第二个元素开始的子切片
    vlist2 := strings.Split(match2[2], ".")[1:]  // 使用 "." 分割匹配结果中的第二个子串，并取第二个元素开始的子切片
    vlistMaxLen := len(vlist1)  // 初始化 vlistMaxLen 为 vlist1 的长度
    if len(vlist2) > vlistMaxLen {  // 如果 vlist2 的长度大于 vlistMaxLen
        vlistMaxLen = len(vlist2)  // 更新 vlistMaxLen 为 vlist2 的长度
    }
    # 遍历索引从 0 到 vlistMaxLen-1
    for index := 0; index < vlistMaxLen; index++ {
        # 根据不同情况进行处理
        switch {
        # 如果 vlist1 的长度小于等于当前索引，则在 list1 中添加 big.NewInt(-1)
        case len(vlist1) <= index:
            list1 = append(list1, big.NewInt(-1))
            # 创建一个新的 big.Int 对象，并将 vlist2[index] 转换为十进制数值
            i := big.NewInt(0)
            i.SetString(vlist2[index], 10)
            # 在 list2 中添加新的 big.Int 对象
            list2 = append(list2, i)
        # 如果 vlist2 的长度小于等于当前索引，则在 list2 中添加 big.NewInt(-1)
        case len(vlist2) <= index:
            list2 = append(list2, big.NewInt(-1))
            # 创建一个新的 big.Int 对象，并将 vlist1[index] 转换为十进制数值
            i := big.NewInt(0)
            i.SetString(vlist1[index], 10)
            # 在 list1 中添加新的 big.Int 对象
            list1 = append(list1, i)
        # 如果 vlist1 和 vlist2 的当前索引对应的值都不以 "0" 开头
        case !strings.HasPrefix(vlist1[index], "0") && !strings.HasPrefix(vlist2[index], "0"):
            # 创建一个新的 big.Int 对象，并将 vlist1[index] 转换为十进制数值
            i := big.NewInt(0)
            i.SetString(vlist1[index], 10)
            # 在 list1 中添加新的 big.Int 对象
            list1 = append(list1, i)
            # 创建一个新的 big.Int 对象，并将 vlist2[index] 转换为十进制数值
            j := big.NewInt(0)
            j.SetString(vlist2[index], 10)
            # 在 list2 中添加新的 big.Int 对象
            list2 = append(list2, j)
        # 其他情况
        default:
            # 计算 vlist1[index] 和 vlist2[index] 的最大长度
            maxLen := len(vlist1[index])
            if len(vlist2[index]) > maxLen {
                maxLen = len(vlist2[index])
            }
            # 如果 vlist1[index] 的长度小于最大长度，则在其末尾添加 "0" 直到长度达到最大长度
            if len(vlist1[index]) < maxLen {
                vlist1[index] += strings.Repeat("0", maxLen-len(vlist1[index]))
            }
            # 如果 vlist2[index] 的长度小于最大长度，则在其末尾添加 "0" 直到长度达到最大长度
            if len(vlist2[index]) < maxLen {
                vlist2[index] += strings.Repeat("0", maxLen-len(vlist2[index]))
            }
            # 创建一个新的 big.Int 对象，并将 vlist1[index] 转换为十进制数值
            i := big.NewInt(0)
            i.SetString(vlist1[index], 10)
            # 在 list1 中添加新的 big.Int 对象
            list1 = append(list1, i)
            # 创建一个新的 big.Int 对象，并将 vlist2[index] 转换为十进制数值
            j := big.NewInt(0)
            j.SetString(vlist2[index], 10)
            # 在 list2 中添加新的 big.Int 对象
            list2 = append(list2, j)
        }
    }

    # 如果 match1[4] 的长度不为 0
    if len(match1[4]) != 0 {
        # 将 match1[4] 转换为 rune 数组
        r := []rune(match1[4])
        # 创建一个新的 big.Int 对象，并将 rune 数组的第一个元素转换为 int64 类型
        i := big.NewInt(int64(r[0]))
        # 在 list1 中添加新的 big.Int 对象
        list1 = append(list1, i)
    }

    # 如果 match2[4] 的长度不为 0
    if len(match2[4]) != 0 {
        # 将 match2[4] 转换为 rune 数组
        r := []rune(match2[4])
        # 创建一个新的 big.Int 对象，并将 rune 数组的第一个元素转换为 int64 类型
        i := big.NewInt(int64(r[0]))
        # 在 list2 中添加新的 big.Int 对象
        list2 = append(list2, i)
    }

    # 计算 list1 和 list2 的最大长度
    maxLen := len(list1)
    if len(list2) > maxLen {
        maxLen = len(list2)
    }
    # 遍历两个列表，比较它们的元素
    for index := 0; index < maxLen; index++ {
        # 如果 list1 的长度小于等于当前索引，返回 -1
        if len(list1) <= index {
            return -1
        }
        # 如果 list2 的长度小于等于当前索引，返回 1
        if len(list2) <= index {
            return 1
        }
        # 比较 list1 和 list2 当前索引处的元素
        c := list1[index].Cmp(list2[index])
        # 如果不相等，返回比较结果
        if c != 0 {
            return c
        }
    }

    # 根据下划线分割字符串，获取子列表
    slist1 := strings.Split(match1[5], "_")[1:]
    slist2 := strings.Split(match2[5], "_")[1:]
    # 获取子列表的最大长度
    maxLen = len(slist1)
    if len(slist2) > maxLen {
        maxLen = len(slist2)
    }
    # 遍历子列表，比较元素
    for index := 0; index < maxLen; index++ {
        # 初始化两个字符串列表
        s1 := []string{"p", "-1"}
        s2 := []string{"p", "-1"}
        # 如果 slist1 的长度大于当前索引，使用正则表达式匹配并获取子列表元素
        if len(slist1) > index {
            s1 = suffixRegexp.FindStringSubmatch(slist1[index])[1:]
            # 如果第二个元素为空，设置为 "0"
            if s1[1] == "" {
                s1[1] = "0"
            }
        }
        # 如果 slist2 的长度大于当前索引，使用正则表达式匹配并获取子列表元素
        if len(slist2) > index {
            s2 = suffixRegexp.FindStringSubmatch(slist2[index])[1:]
            # 如果第二个元素为空，设置为 "0"
            if s2[1] == "" {
                s2[1] = "0"
            }
        }
        # 比较两个子列表元素的第一个值
        if s1[0] != s2[0] {
            # 获取第一个值对应的数值
            v1 := suffixValue[s1[0]]
            v2 := suffixValue[s2[0]]
            # 如果 v1 大于 v2，返回 1
            if v1 > v2 {
                return 1
            }
            # 否则返回 -1
            return -1
        }
        # 如果两个子列表元素的第一个值相等，比较第二个值
        if s1[1] != s2[1] {
            # 将字符串转换为大整数
            i := big.NewInt(0)
            i.SetString(s1[1], 10)
            j := big.NewInt(0)
            j.SetString(s2[1], 10)
            # 比较两个大整数
            c := i.Cmp(j)
            # 如果不相等，返回比较结果
            if c != 0 {
                return c
            }
        }
    }

    # 将字符串转换为大整数
    r1 := big.NewInt(0)
    if match1[9] != "" {
        r1.SetString(match1[9], 10)
    }
    r2 := big.NewInt(0)
    if match2[9] != "" {
        r2.SetString(match2[9], 10)
    }

    # 比较两个大整数
    return r1.Cmp(r2)
# 闭合前面的函数定义
```