# `kubo\repo\fsrepo\migrations\versions.go`

```
package migrations

import (
    "bufio"  // 导入用于缓冲读取的包
    "bytes"  // 导入用于操作字节切片的包
    "context"  // 导入用于控制goroutine的包
    "errors"  // 导入用于处理错误的包
    "fmt"  // 导入用于格式化输入输出的包
    "path"  // 导入用于处理文件路径的包
    "sort"  // 导入用于排序的包
    "strings"  // 导入用于处理字符串的包

    "github.com/blang/semver/v4"  // 导入用于处理语义化版本的包
)

const distVersions = "versions"  // 定义常量distVersions为"versions"

// LatestDistVersion返回指定分发的最新版本，该版本在分发站点上可用。
func LatestDistVersion(ctx context.Context, fetcher Fetcher, dist string, stableOnly bool) (string, error) {
    vs, err := DistVersions(ctx, fetcher, dist, false)  // 调用DistVersions函数获取指定分发的所有版本
    if err != nil {
        return "", err
    }

    for i := len(vs) - 1; i >= 0; i-- {  // 从最新版本开始遍历版本列表
        ver := vs[i]  // 获取当前版本
        if stableOnly && strings.Contains(ver, "-rc") {  // 如果只需要稳定版本且当前版本包含"-rc"，则跳过
            continue
        }
        if strings.Contains(ver, "-dev") {  // 如果当前版本包含"-dev"，则跳过
            continue
        }
        return ver, nil  // 返回当前版本
    }
    return "", errors.New("could not find a non dev version")  // 返回错误信息
}

// DistVersions返回指定分发的所有版本，这些版本在分发站点上可用。列表按升序排列，除非sortDesc为true。
func DistVersions(ctx context.Context, fetcher Fetcher, dist string, sortDesc bool) ([]string, error) {
    versionBytes, err := fetcher.Fetch(ctx, path.Join(dist, distVersions))  // 调用fetcher.Fetch函数获取指定分发的版本信息
    if err != nil {
        return nil, err
    }

    prefix := "v"  // 定义前缀为"v"
    var vers []semver.Version  // 定义semver.Version类型的切片

    scan := bufio.NewScanner(bytes.NewReader(versionBytes))  // 创建用于扫描的Scanner
    for scan.Scan() {  // 循环扫描版本信息
        ver, err := semver.Make(strings.TrimLeft(scan.Text(), prefix))  // 使用semver.Make函数解析版本信息
        if err != nil {
            continue
        }
        vers = append(vers, ver)  // 将解析后的版本信息添加到切片中
    }
    if scan.Err() != nil {
        return nil, fmt.Errorf("could not read versions: %w", scan.Err())  // 返回错误信息
    }

    if sortDesc {  // 如果需要降序排序
        sort.Sort(sort.Reverse(semver.Versions(vers)))  // 对版本切片进行降序排序
    } else {
        sort.Sort(semver.Versions(vers))  // 对版本切片进行升序排序
    }

    out := make([]string, len(vers))  // 创建与版本切片相同长度的字符串切片
    for i := range vers {  // 遍历版本切片
        out[i] = prefix + vers[i].String()  // 将版本信息添加到字符串切片中
    }

    return out, nil  // 返回版本信息字符串切片
}
```