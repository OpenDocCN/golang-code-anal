# `grype\grype\version\rpm_version.go`

```
// 导入所需的包
package version

import (
	"fmt" // 格式化输出
	"math" // 数学函数
	"reflect" // 反射
	"regexp" // 正则表达式
	"strconv" // 字符串转换为数字
	"strings" // 字符串操作
	"unicode" // Unicode 字符
)

// 定义 rpmVersion 结构体
type rpmVersion struct {
	epoch   *int // 版本的 epoch
	version string // 版本号
	release string // 发行版号
}

// 解析原始字符串创建 rpmVersion 对象
func newRpmVersion(raw string) (rpmVersion, error) {
	// 从原始字符串中分离出 epoch 和剩余的版本信息
	epoch, remainingVersion, err := splitEpochFromVersion(raw)
# 如果发生错误，则返回空的 rpmVersion 对象和错误信息
if err != nil:
    return rpmVersion{}, err

# 使用 "-" 分割剩余版本字符串，得到版本号和剩余部分
fields := strings.SplitN(remainingVersion, "-", 2)
version := fields[0]

# 定义 release 变量
var release string
# 如果剩余部分长度大于 1，则说明有 release
if len(fields) > 1:
    # 存储 release
    release = fields[1]

# 返回 rpmVersion 对象和空的错误信息
return rpmVersion{
    epoch:   epoch,
    version: version,
    release: release,
}, nil
// 从原始版本字符串中分离出 epoch 和版本号，并返回指向 epoch 的指针、版本号和可能的错误
func splitEpochFromVersion(rawVersion string) (*int, string, error) {
    // 使用冒号分割原始版本字符串，最多分割成两部分
    fields := strings.SplitN(rawVersion, ":", 2)

    // 当 epoch 未包含在内时，在比较时应该被视为 0（参见 https://github.com/rpm-software-management/rpm/issues/450）。
    // 但是，通常在漏洞数据库或源 RPM 文件名中包含的 epoch 不是一致的，因此将缺失的 epoch 表示为 nil。这允许比较逻辑本身确定是否应该使用零或其他值，从而支持更灵活的比较选项，因为版本创建不是有损的
    if len(fields) == 1 {
        return nil, rawVersion, nil
    }

    // 存在 epoch
    epochStr := strings.TrimLeft(fields[0], " ")

    // 将 epoch 字符串转换为整数
    epoch, err := strconv.Atoi(epochStr)
    if err != nil {
// 返回一个空值、空字符串和一个带有错误信息的错误对象，用于表示无法解析的时代
return nil, "", fmt.Errorf("unable to parse epoch (%s): %w", epochStr, err)
// 返回一个带有时代和字段信息的错误对象
return &epoch, fields[1], nil
// 比较两个版本对象的方法，返回比较结果和可能的错误
func (v *rpmVersion) Compare(other *Version) (int, error) {
// 如果给定的版本对象格式不是 RPM 格式，则返回错误
if other.Format != RpmFormat {
    return -1, fmt.Errorf("unable to compare rpm to given format: %s", other.Format)
}
// 如果给定的版本对象中的 RPM 版本为空，则返回错误
if other.rich.rpmVer == nil {
    return -1, fmt.Errorf("given empty rpmVersion object")
}

// 调用另一个方法来比较两个 RPM 版本对象，返回比较结果和可能的错误
return other.rich.rpmVer.compare(*v), nil
}

// Compare 方法返回 0 表示 v == v2，返回 -1 表示 v < v2，返回 +1 表示 v > v2。
// 这是对于在漏洞扫描中遇到的混乱数据进行比较的一种实用的适应方法。如果时代不是存在并且是显式的
// 定义一个比较函数，用于比较两个rpmVersion类型的对象
func (v rpmVersion) compare(v2 rpmVersion) int {
    // 如果两个对象相等，则返回0
    if reflect.DeepEqual(v, v2) {
        return 0
    }

    // 只有在两个对象都有epoch并且是显式指定的情况下才比较epoch值
    // 这与RedHat的规定有所不同，RedHat规定如果epoch缺失，则假定为0
    // 但是，由于我们可能处理来自上游数据源的数据，其中包含一个包的epoch值，但该值被剥离了
    // 我们最好只比较不带epoch值的版本值
    if epochIsPresent(v.epoch) && epochIsPresent(v2.epoch) {
        // 比较epoch值
        epochResult := compareEpochs(*v.epoch, *v2.epoch)
        // 如果epoch值不同，则返回比较结果
        if epochResult != 0 {
            return epochResult
        }
    }

    // 比较版本值
    ret := compareRpmVersions(v.version, v2.version)
```

// 如果返回值不等于0，则返回该值
	if ret != 0 {
		return ret
	}

	// 返回两个rpm版本的比较结果
	return compareRpmVersions(v.release, v2.release)
}

// 判断epoch是否存在
func epochIsPresent(epoch *int) bool {
	// 如果epoch不为空，则存在
	return epoch != nil
}

// 比较epoch，使用标准的整数比较进行排序
func compareEpochs(e1 int, e2 int) int {
	// 使用switch语句进行比较
	switch {
	// 如果e1大于e2，则返回1
	case e1 > e2:
		return 1
	// 如果e1小于e2，则返回-1
	case e1 < e2:
		return -1
	// 如果e1等于e2，则返回0
	default:
		return 0
// 定义 rpmVersion 结构体的 String 方法，用于返回版本号的字符串表示
func (v rpmVersion) String() string {
    version := ""
    // 如果存在 epoch，将其加入版本号字符串中
    if v.epoch != nil {
        version += fmt.Sprintf("%d:", *v.epoch)
    }
    // 将版本号加入版本号字符串中
    version += v.version

    // 如果存在 release，将其加入版本号字符串中
    if v.release != "" {
        version += fmt.Sprintf("-%s", v.release)
    }
    // 返回版本号字符串
    return version
}

// compareRpmVersions 比较两个版本或发布字符串，不考虑 epoch
// 来源：https://github.com/cavaliercoder/go-rpm/blob/master/version.go
//
// 对于原始的 C 语言实现，请参见：
// 创建一个正则表达式模式，用于匹配字符串中的字母和数字
var alphanumPattern = regexp.MustCompile("([a-zA-Z]+)|([0-9]+)|(~)")

//nolint:funlen,gocognit
// 比较两个 RPM 版本号的函数
func compareRpmVersions(a, b string) int {
	// 如果两个版本号相等，则返回 0
	if a == b {
		return 0
	}

	// 获取字符串中的字母/数字段
	segsa := alphanumPattern.FindAllString(a, -1)
	segsb := alphanumPattern.FindAllString(b, -1)
	// 取两个版本号中较短的段数
	segs := int(math.Min(float64(len(segsa)), float64(len(segsb))))

	// 逐个比较每个段
	for i := 0; i < segs; i++ {
		// 获取当前段的值
		a := segsa[i]
		b := segsb[i]
// 比较两个字符串是否以波浪号开头
if []rune(a)[0] == '~' || []rune(b)[0] == '~' {
    // 如果 a 字符串不是以波浪号开头，返回 1
    if []rune(a)[0] != '~' {
        return 1
    }
    // 如果 b 字符串不是以波浪号开头，返回 -1
    if []rune(b)[0] != '~' {
        return -1
    }
}

// 如果 a 字符串以数字开头
if unicode.IsNumber([]rune(a)[0]) {
    // 如果 b 字符串不是以数字开头，返回 1
    if !unicode.IsNumber([]rune(b)[0]) {
        // a 是数字，b 是字母
        return 1
    }

    // 去除 a 和 b 字符串开头的零
    a = strings.TrimLeft(a, "0")
    b = strings.TrimLeft(b, "0")
}
// 如果字符串a的长度大于字符串b的长度，则返回1，表示a排在b之前
if len(a) > len(b) {
    return 1
} else if len(b) > len(a) {
    return -1
}

// 如果字符串b的首字符是数字，则返回-1，表示a排在b之前
} else if unicode.IsNumber([]rune(b)[0]) {
    return -1
}

// 按照字符串的字典顺序进行比较
if a < b {
    return -1
} else if a > b {
    return 1
}
// 如果两个字符串数组的长度相等，说明它们的内容相同，返回0
if len(segsa) == len(segsb) {
    return 0
}

// 如果某个字符串数组的长度大于指定的最小段数，并且该段的第一个字符是波浪号"~"，则返回-1
if len(segsa) > segs && []rune(segsa[segs])[0] == '~' {
    return -1
} else if len(segsb) > segs && []rune(segsb[segs])[0] == '~' {
    return 1
}

// 如果第一个字符串数组的长度大于第二个字符串数组的长度，返回1；否则返回-1
if len(segsa) > len(segsb) {
    return 1
}
return -1
```