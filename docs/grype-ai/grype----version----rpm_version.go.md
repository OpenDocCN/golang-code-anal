# `grype\grype\version\rpm_version.go`

```
package version

import (
    "fmt"       // 导入 fmt 包，用于格式化输出
    "math"      // 导入 math 包，提供数学函数和常量
    "reflect"   // 导入 reflect 包，用于反射操作
    "regexp"    // 导入 regexp 包，用于正则表达式
    "strconv"   // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings"   // 导入 strings 包，提供对字符串的操作
    "unicode"   // 导入 unicode 包，提供对 Unicode 字符的操作
)

type rpmVersion struct {
    epoch   *int    // RPM 版本的 epoch，可能为空
    version string  // RPM 版本号
    release string  // RPM 发行版号
}

func newRpmVersion(raw string) (rpmVersion, error) {
    epoch, remainingVersion, err := splitEpochFromVersion(raw)  // 调用 splitEpochFromVersion 函数，分割原始版本号
    if err != nil {
        return rpmVersion{}, err  // 如果出错，返回空的 rpmVersion 和错误
    }

    fields := strings.SplitN(remainingVersion, "-", 2)  // 使用 "-" 分割剩余的版本号和发行版号
    version := fields[0]  // 获取版本号

    var release string
    if len(fields) > 1 {
        // there is a release
        release = fields[1]  // 如果有发行版号，获取发行版号
    }

    return rpmVersion{
        epoch:   epoch,    // 返回创建的 rpmVersion 对象
        version: version,
        release: release,
    }, nil
}

func splitEpochFromVersion(rawVersion string) (*int, string, error) {
    fields := strings.SplitN(rawVersion, ":", 2)  // 使用 ":" 分割原始版本号

    if len(fields) == 1 {
        return nil, rawVersion, nil  // 如果没有 epoch，则返回 nil 和原始版本号
    }

    // there is an epoch
    epochStr := strings.TrimLeft(fields[0], " ")  // 去除 epoch 字段的空格

    epoch, err := strconv.Atoi(epochStr)  // 将 epoch 字符串转换为整数
    if err != nil {
        return nil, "", fmt.Errorf("unable to parse epoch (%s): %w", epochStr, err)  // 如果转换出错，返回错误
    }

    return &epoch, fields[1], nil  // 返回 epoch 和剩余的版本号
}

func (v *rpmVersion) Compare(other *Version) (int, error) {
    if other.Format != RpmFormat {
        return -1, fmt.Errorf("unable to compare rpm to given format: %s", other.Format)  // 如果格式不匹配，返回错误
    }
    if other.rich.rpmVer == nil {
        return -1, fmt.Errorf("given empty rpmVersion object")  // 如果给定的 rpmVersion 对象为空，返回错误
    }

    return other.rich.rpmVer.compare(*v), nil  // 调用 compare 方法比较两个 rpmVersion 对象
}
// Compare函数用于比较两个rpmVersion对象的版本号，返回值为0表示相等，-1表示小于，+1表示大于。
// 这是对杂乱数据进行比较的实用适应。如果版本中没有显式指定的epoch存在，比较时会忽略它们。
// 对于符合rpm规范的比较，请使用strictCompare()函数。
func (v rpmVersion) compare(v2 rpmVersion) int {
    // 如果两个版本号对象相等，则返回0
    if reflect.DeepEqual(v, v2) {
        return 0
    }

    // 只有当两个版本号对象的epoch都存在且显式指定时才进行比较。
    // 这在技术上违反了RedHat对于缺少epoch的处理方式（假设为0）。
    // 但是，由于我们可能处理的是上游数据源，其中包含一个包的epoch但值被剥离的情况，
    // 我们最好只比较不带epoch值的版本值。
    if epochIsPresent(v.epoch) && epochIsPresent(v2.epoch) {
        epochResult := compareEpochs(*v.epoch, *v2.epoch)
        if epochResult != 0 {
            return epochResult
        }
    }

    // 比较版本号值
    ret := compareRpmVersions(v.version, v2.version)
    if ret != 0 {
        return ret
    }

    // 比较发布号值
    return compareRpmVersions(v.release, v2.release)
}

// 判断epoch是否存在
func epochIsPresent(epoch *int) bool {
    return epoch != nil
}

// 比较epoch值的大小，标准的int比较用于排序
func compareEpochs(e1 int, e2 int) int {
    switch {
    case e1 > e2:
        return 1
    case e1 < e2:
        return -1
    default:
        return 0
    }
}

// String方法用于返回rpmVersion对象的字符串表示形式
func (v rpmVersion) String() string {
    version := ""
    if v.epoch != nil {
        version += fmt.Sprintf("%d:", *v.epoch)
    }
    version += v.version

    if v.release != "" {
        version += fmt.Sprintf("-%s", v.release)
    }
    return version
}

// compareRpmVersions函数用于比较两个不带epoch的版本或发布号字符串
// 来源：https://github.com/cavaliercoder/go-rpm/blob/master/version.go
//
// 原始的C语言实现，请参见：
// 定义正则表达式，用于匹配字母、数字和波浪线
var alphanumPattern = regexp.MustCompile("([a-zA-Z]+)|([0-9]+)|(~)")

//nolint:funlen,gocognit
func compareRpmVersions(a, b string) int {
    // 如果 a 和 b 相等，则返回 0
    if a == b {
        return 0
    }

    // 获取 a 和 b 的字母/数字段
    segsa := alphanumPattern.FindAllString(a, -1)
    segsb := alphanumPattern.FindAllString(b, -1)
    segs := int(math.Min(float64(len(segsa)), float64(len(segsb))))

    // 比较每个段
    for i := 0; i < segs; i++ {
        a := segsa[i]
        b := segsb[i]

        // 比较波浪线
        if []rune(a)[0] == '~' || []rune(b)[0] == '~' {
            if []rune(a)[0] != '~' {
                return 1
            }
            if []rune(b)[0] != '~' {
                return -1
            }
        }

        if unicode.IsNumber([]rune(a)[0]) {
            // 数字始终大于字母
            if !unicode.IsNumber([]rune(b)[0]) {
                // a 是数字，b 是字母
                return 1
            }

            // 去除前导零
            a = strings.TrimLeft(a, "0")
            b = strings.TrimLeft(b, "0")

            // 最长的字符串获胜，无需进一步比较
            if len(a) > len(b) {
                return 1
            } else if len(b) > len(a) {
                return -1
            }
        } else if unicode.IsNumber([]rune(b)[0]) {
            // a 是字母，b 是数字
            return -1
        }

        // 字符串比较
        if a < b {
            return -1
        } else if a > b {
            return 1
        }
    }

    // 段都相同，但分隔符可能不同
    if len(segsa) == len(segsb) {
        return 0
    }

    // 如果在最小段数之后的段中有波浪线，则找到它
    if len(segsa) > segs && []rune(segsa[segs])[0] == '~' {
        return -1
    // 如果第一个字符串的分段数大于第二个字符串的分段数，并且第二个字符串的第一个字符是波浪线，则返回1
    } else if len(segsb) > segs && []rune(segsb[segs])[0] == '~' {
        return 1
    }

    // 分段数多的字符串获胜
    if len(segsa) > len(segsb) {
        return 1
    }
    // 分段数少的字符串输
    return -1
# 闭合前面的函数定义
```