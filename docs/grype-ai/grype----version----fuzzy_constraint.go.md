# `grype\grype\version\fuzzy_constraint.go`

```
// 导入所需的包
package version

import (
	"fmt" // 导入格式化输出包
	"regexp" // 导入正则表达式包
	"strings" // 导入字符串处理包

	hashiVer "github.com/anchore/go-version" // 导入第三方版本控制包

)

// 定义模糊约束类型，包含原始短语、提示短语、语义约束和约束表达式
type fuzzyConstraint struct {
	rawPhrase          string // 原始短语
	phraseHint         string // 提示短语
	semanticConstraint *hashiVer.Constraints // 语义约束
	constraints        constraintExpression // 约束表达式
}

// 创建新的模糊约束对象
func newFuzzyConstraint(phrase, hint string) (*fuzzyConstraint, error) {
	if phrase == "" {
		// 如果输入的短语为空，则始终满足约束条件
		return &fuzzyConstraint{
			rawPhrase:  phrase,
			phraseHint: hint,
		}, nil
	}

	constraints, err := newConstraintExpression(phrase, newFuzzyComparator)
	if err != nil {
		// 如果无法创建模糊约束，则返回错误
		return nil, fmt.Errorf("could not create fuzzy constraint: %+v", err)
	}
	var semverConstraint *hashiVer.Constraints

	// 检查所有版本单元短语，以查看是否为有效的语义版本约束
	valid := true
check:
	for _, units := range constraints.units {
		for _, unit := range units {
			if !pseudoSemverPattern.MatchString(unit.version) {
				// 如果版本单元的版本号不符合伪语义版本模式，则标记为无效
				valid = false  # 将 valid 变量设置为 false
				break check  # 跳出 check 循环
			}
		}
	}

	if value, err := hashiVer.NewConstraint(phrase); err == nil && valid {  # 如果 hashiVer.NewConstraint(phrase) 没有错误并且 valid 为 true
		semverConstraint = &value  # 将 value 赋值给 semverConstraint
	}

	return &fuzzyConstraint{  # 返回 fuzzyConstraint 结构体指针
		rawPhrase:          phrase,  # 设置 rawPhrase 字段为 phrase
		phraseHint:         hint,  # 设置 phraseHint 字段为 hint
		constraints:        constraints,  # 设置 constraints 字段为 constraints
		semanticConstraint: semverConstraint,  # 设置 semanticConstraint 字段为 semverConstraint
	}, nil  # 返回空指针
}

func newFuzzyComparator(unit constraintUnit) (Comparator, error) {  # 定义一个名为 newFuzzyComparator 的函数，接受 constraintUnit 类型参数，返回 Comparator 和 error
	ver, err := newFuzzyVersion(unit.version)  # 调用 newFuzzyVersion 函数，将结果赋值给 ver 和 err
// 如果发生错误，则返回错误信息
if err != nil {
    return nil, fmt.Errorf("unable to parse constraint version (%s): %w", unit.version, err)
}
// 返回版本对象和空错误
return &ver, nil
}

// 判断模糊约束是否满足给定的版本对象
func (f *fuzzyConstraint) Satisfied(verObj *Version) (bool, error) {
    // 如果原始短语为空且版本对象不为空，则始终满足约束
    if f.rawPhrase == "" && verObj != nil {
        return true, nil
    } else if verObj == nil {
        // 如果原始短语不为空且没有给定版本，则始终失败
        if f.rawPhrase != "" {
            return false, nil
        }
        return true, nil
    }

    version := verObj.Raw
// 根据版本对象重建临时约束
if verObj.Format != UnknownFormat {
    // 根据版本对象的格式获取约束
    newConstaint, err := GetConstraint(f.rawPhrase, verObj.Format)
    // 检查约束是否不是模糊约束
    _, ok := newConstaint.(*fuzzyConstraint)
    if err == nil && !ok {
        // 检查约束是否满足版本对象
        satisfied, err := newConstaint.Satisfied(verObj)
        if err == nil {
            return satisfied, nil
        }
    }
}

// 首先尝试使用语义版本，然后回退到模糊部分匹配...
if f.semanticConstraint != nil {
    // 检查版本是否符合伪语义版本模式
    if pseudoSemverPattern.MatchString(version) {
        // 如果是，创建语义版本对象并检查约束是否满足
        if semver, err := newSemanticVersion(version); err == nil && semver != nil {
            return f.semanticConstraint.Check(semver.verObj), nil
        }
    }
}
// 结构体方法，用于检查给定的版本是否满足约束条件
func (f *fuzzyConstraint) satisfied(verObj *version) bool {
	// 如果约束条件为空，则返回 true
	if f.rawPhrase == "" {
		return true
	}
	// 使用模糊匹配来判断版本是否满足约束条件
	// 如果 semver 不起作用，则使用模糊匹配
	return f.constraints.satisfied(verObj)
}

// 结构体方法，用于返回模糊约束条件的字符串表示
func (f *fuzzyConstraint) String() string {
	// 如果原始约束条件为空，则返回 "none (unknown)"
	if f.rawPhrase == "" {
		return "none (unknown)"
	}
	// 如果有提示信息，则返回原始约束条件和提示信息的组合
	if f.phraseHint != "" {
		return fmt.Sprintf("%s (%s)", f.rawPhrase, f.phraseHint)
	}
	// 否则返回原始约束条件和 "unknown" 的组合
	return fmt.Sprintf("%s (unknown)", f.rawPhrase)
}

// 注意：以下代码来自 https://github.com/facebookincubator/nvdtools/blob/688794c4d3a41929eeca89304e198578d4595d53/cvefeed/nvd/smartvercmp.go (apache V2)
// 我更希望导入这个功能而不是复制它，但是这些函数并未从包中导出

// fuzzyVersionComparison 用于比较软件的版本号字符串
// 它尝试对任何未指定的版本类型进行正确处理
// 假设 v1 和 v2 遵循相同的版本约定。
// 对于 "95SE" vs "98SP1" 或者 "16.3.2" vs. "3.7.0"，将返回有意义的结果，
// 但对于 "2000" vs "11.7" 将不会返回有意义的结果。
// 如果 v1 < v2，则返回 -1；如果 v1 > v2，则返回 1；如果 v1 == v2，则返回 0。
func fuzzyVersionComparison(v1, v2 string) int {
    // 去除版本号前的 "v"，如果有的话
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
        switch {
            case diff > 0: // ns1 的数字部分更长
                ns2 = leftPad(ns2, diff)
            case diff < 0: // ns2 的数字部分更长
                ns1 = leftPad(ns1, -diff)
        }
// 使用 strings.Compare 比较两个字符串 ns1 和 ns2，如果不相等则返回比较结果
if cmp := strings.Compare(ns1, ns2); cmp != 0 {
    return cmp
}

// 更新字符串 s1 和 s2，跳过已经比较过的部分
s1 = s1[skip1:]
s2 = s2[skip2:]

// 如果所有部分都相等，返回长度较长的版本号
if len(v1) > len(v2) {
    return 1
}
if len(v2) > len(v1) {
    return -1
}
// 如果长度相等，返回 0
return 0
}

// parseVersionParts 函数返回字符串开头连续的数字的长度，最后一个非分隔符字符的索引（用于比较），以及版本号部分（主要、次要等）的结束索引
// 解析版本号的各个部分，返回它们的位置
// 例如，parseVersionParts("11.b4.16-New_Year_Edition") 将返回 (2, 3, 4)
func parseVersionParts(v string) (int, int, int) {
    var num int
    // 遍历字符串，找到第一个非数字字符的位置
    for num = 0; num < len(v); num++ {
        if v[num] < '0' || v[num] > '9' {
            break
        }
    }
    // 如果字符串全是数字，则返回字符串长度作为位置
    if num == len(v) {
        return num, num, num
    }
    // 任何标点符号都会分隔版本号的各个部分
    // 查找第一个标点符号的位置
    skip := strings.IndexFunc(v, func(b rune) bool {
        // 标点符号的 ASCII 码范围是 33-126，但排除了 48-57, 65-90 和 97-122 的间隙
        // 这种逆逻辑可以提前短路大部分字符，从而在基准测试中节省约 20 纳秒
        return b >= '!' && b <= '~' &&
            !(b > '/' && b < ':' ||
                b > '@' && b < '[' ||
// 如果 b 大于 '`' 并且小于 '{'，则返回 true，否则返回 false
	})
	// 如果 skip 等于 -1，则返回 num、v 的长度和 v 的长度
	// 否则返回 num、skip 和 skip+1
}

// leftPad 使用 n 个 '0' 对 s 进行左填充
func leftPad(s string, n int) string {
	// 创建一个 strings.Builder 对象
	var sb strings.Builder
	// 循环 n 次，向 sb 中写入 '0'
	for i := 0; i < n; i++ {
		sb.WriteByte('0')
	}
	// 将 s 写入 sb
	sb.WriteString(s)
	// 返回 sb 的字符串形式
	return sb.String()
}

// stripLeadingV 去除 ver 字符串的前缀 "v"
func stripLeadingV(ver string) string {
	// 返回去除 "v" 前缀后的字符串
	return strings.TrimPrefix(ver, "v")
这是一个代码块的结束标记，表示前面的函数或者循环的结束。
```