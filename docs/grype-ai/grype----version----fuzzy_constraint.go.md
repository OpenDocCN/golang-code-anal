# `grype\grype\version\fuzzy_constraint.go`

```go
package version

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "regexp"  // 导入 regexp 包，用于正则表达式匹配
    "strings"  // 导入 strings 包，用于字符串操作

    hashiVer "github.com/anchore/go-version"  // 导入自定义的 hashiVer 包
)

// 定义正则表达式，用于匹配语义版本号和部分版本号
var pseudoSemverPattern = regexp.MustCompile(`^(0|[1-9]\d*)(\.(0|[1-9]\d*))?(\.(0|[1-9]\d*))?(?:(-|alpha|beta|rc)((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$`)

// 定义模糊约束结构体
type fuzzyConstraint struct {
    rawPhrase          string
    phraseHint         string
    semanticConstraint *hashiVer.Constraints
    constraints        constraintExpression
}

// 创建新的模糊约束
func newFuzzyConstraint(phrase, hint string) (*fuzzyConstraint, error) {
    if phrase == "" {
        // 如果约束为空，则始终满足条件
        return &fuzzyConstraint{
            rawPhrase:  phrase,
            phraseHint: hint,
        }, nil
    }

    // 创建约束表达式
    constraints, err := newConstraintExpression(phrase, newFuzzyComparator)
    if err != nil {
        return nil, fmt.Errorf("could not create fuzzy constraint: %+v", err)
    }
    var semverConstraint *hashiVer.Constraints

    // 检查所有版本单元短语，以查看是否为有效的语义版本约束
    valid := true
check:
    for _, units := range constraints.units {
        for _, unit := range units {
            if !pseudoSemverPattern.MatchString(unit.version) {
                valid = false
                break check
            }
        }
    }

    // 如果是有效的语义版本约束，则创建语义约束
    if value, err := hashiVer.NewConstraint(phrase); err == nil && valid {
        semverConstraint = &value
    }

    return &fuzzyConstraint{
        rawPhrase:          phrase,
        phraseHint:         hint,
        constraints:        constraints,
        semanticConstraint: semverConstraint,
    }, nil
}

// 创建新的模糊比较器
func newFuzzyComparator(unit constraintUnit) (Comparator, error) {
    ver, err := newFuzzyVersion(unit.version)
    # 如果 err 不为空，表示出现错误
    if err != nil:
        # 返回空指针和格式化的错误信息，包括无法解析的约束版本和具体的错误信息
        return nil, fmt.Errorf("unable to parse constraint version (%s): %w", unit.version, err)
    # 如果没有错误，返回解析后的版本和空指针
    return &ver, nil
// 判断模糊约束是否满足给定的版本对象
func (f *fuzzyConstraint) Satisfied(verObj *Version) (bool, error) {
    // 如果原始短语为空且版本对象不为空，则始终满足约束
    if f.rawPhrase == "" && verObj != nil {
        return true, nil
    } else if verObj == nil {
        // 如果给定的版本对象为空且原始短语不为空，则始终不满足约束
        if f.rawPhrase != "" {
            return false, nil
        }
        return true, nil
    }

    version := verObj.Raw

    // 根据版本对象重新构建临时约束
    if verObj.Format != UnknownFormat {
        newConstaint, err := GetConstraint(f.rawPhrase, verObj.Format)
        // 检查约束是否不是模糊约束
        _, ok := newConstaint.(*fuzzyConstraint)
        if err == nil && !ok {
            satisfied, err := newConstaint.Satisfied(verObj)
            if err == nil {
                return satisfied, nil
            }
        }
    }

    // 首先尝试语义版本，然后回退到模糊部分匹配...
    if f.semanticConstraint != nil {
        if pseudoSemverPattern.MatchString(version) {
            if semver, err := newSemanticVersion(version); err == nil && semver != nil {
                return f.semanticConstraint.Check(semver.verObj), nil
            }
        }
    }
    // 如果语义版本不起作用，则改用模糊部分匹配...
    return f.constraints.satisfied(verObj)
}

// 返回模糊约束的字符串表示形式
func (f *fuzzyConstraint) String() string {
    if f.rawPhrase == "" {
        return "none (unknown)"
    }
    if f.phraseHint != "" {
        return fmt.Sprintf("%s (%s)", f.rawPhrase, f.phraseHint)
    }
    return fmt.Sprintf("%s (unknown)", f.rawPhrase)
}

// 注意：以下代码来自 https://github.com/facebookincubator/nvdtools/blob/688794c4d3a41929eeca89304e198578d4595d53/cvefeed/nvd/smartvercmp.go (apache V2)
// 我更喜欢导入这个功能而不是复制它，但是这些函数未从包中导出
// 模糊版本比较比较软件的字符串化版本。
// 模糊版本比较函数，用于比较两个版本号的大小
// 如果未指定版本类型，则假定 v1 和 v2 采用相同的版本约定
// 对于"95SE" vs "98SP1"或"16.3.2" vs. "3.7.0"等情况返回有意义的结果，但对于"2000" vs "11.7"则无法返回有意义的结果
// 如果 v1 < v2，则返回-1；如果 v1 > v2，则返回1；如果 v1 == v2，则返回0
func fuzzyVersionComparison(v1, v2 string) int {
    // 去除版本号中可能存在的前导字符"v"
    v1 = stripLeadingV(v1)
    v2 = stripLeadingV(v2)
    // 循环比较版本号的各个部分
    for s1, s2 := v1, v2; len(s1) > 0 && len(s2) > 0; {
        // 解析版本号的各个部分
        num1, cmpTo1, skip1 := parseVersionParts(s1)
        num2, cmpTo2, skip2 := parseVersionParts(s2)

        ns1 := s1[:cmpTo1]
        ns2 := s2[:cmpTo2]
        diff := num1 - num2
        // 根据数字部分的长度差异进行补齐
        switch {
        case diff > 0: // ns1 的数字部分更长
            ns2 = leftPad(ns2, diff)
        case diff < 0: // ns2 的数字部分更长
            ns1 = leftPad(ns1, -diff)
        }
        // 比较数字部分
        if cmp := strings.Compare(ns1, ns2); cmp != 0 {
            return cmp
        }
        // 更新版本号字符串，准备比较下一个部分
        s1 = s1[skip1:]
        s2 = s2[skip2:]
    }
    // 如果到目前为止所有部分都相等，则比较版本号长度
    if len(v1) > len(v2) {
        return 1
    }
    if len(v2) > len(v1) {
        return -1
    }
    return 0
}

// 解析版本号的各个部分，返回连续的数字部分的长度、最后一个非分隔符字符的位置以及版本部分（主要、次要等）的结束位置
// 例如，parseVersionParts("11.b4.16-New_Year_Edition") 将返回 (2, 3, 4)
func parseVersionParts(v string) (int, int, int) {
    var num int
    // 寻找连续的数字部分
    for num = 0; num < len(v); num++ {
        if v[num] < '0' || v[num] > '9' {
            break
        }
    }
    // 如果整个字符串都是数字，则返回字符串长度
    if num == len(v) {
        return num, num, num
    }
    // 任何标点符号都将分隔版本号的各个部分
    skip := strings.IndexFunc(v, func(b rune) bool {
        // 定义一个匿名函数，用于查找字符串中第一个标点符号的位置
        // 标点符号的 ASCII 码范围是 33-126，但排除了 48-57, 65-90 和 97-122 的间隙
        // 这种反向逻辑可以提前短路大部分字符，从而在基准测试中节省约20纳秒
        return b >= '!' && b <= '~' &&
            !(b > '/' && b < ':' ||
                b > '@' && b < '[' ||
                b > '`' && b < '{')
    })
    // 如果没有找到标点符号，则返回字符串长度
    if skip == -1 {
        return num, len(v), len(v)
    }
    // 如果找到标点符号，则返回标点符号前的字符数和标点符号的位置
    return num, skip, skip + 1
// leftPad函数用于在字符串s的左侧填充n个'0'
func leftPad(s string, n int) string {
    // 创建一个字符串构建器
    var sb strings.Builder
    // 循环n次，向构建器中添加'0'
    for i := 0; i < n; i++ {
        sb.WriteByte('0')
    }
    // 将字符串s添加到构建器末尾
    sb.WriteString(s)
    // 返回构建器中的字符串
    return sb.String()
}

// stripLeadingV函数用于去除字符串ver中的前缀"v"
func stripLeadingV(ver string) string {
    // 使用strings包中的TrimPrefix函数去除ver中的前缀"v"
    return strings.TrimPrefix(ver, "v")
}
```