# `v2ray-core\common\platform\others.go`

```go
// +build !windows
// 定义了一个构建标签，表示该文件不适用于 Windows 平台

package platform
// 声明了一个名为 platform 的包

import (
    "os" // 导入操作系统相关的包
    "path/filepath" // 导入处理文件路径的包
)

func ExpandEnv(s string) string {
    return os.ExpandEnv(s) // 使用操作系统的环境变量替换字符串中的 ${var} 或 $var
}

func LineSeparator() string {
    return "\n" // 返回换行符
}

func GetToolLocation(file string) string {
    const name = "v2ray.location.tool" // 定义常量 name
    toolPath := EnvFlag{Name: name, AltName: NormalizeEnvName(name)}.GetValue(getExecutableDir) // 获取工具路径
    return filepath.Join(toolPath, file) // 返回工具路径和文件名的拼接结果
}

// GetAssetLocation search for `file` in certain locations
// 在特定位置搜索文件
func GetAssetLocation(file string) string {
    const name = "v2ray.location.asset" // 定义常量 name
    assetPath := NewEnvFlag(name).GetValue(getExecutableDir) // 获取资源路径
    defPath := filepath.Join(assetPath, file) // 默认路径为资源路径和文件名的拼接结果
    for _, p := range []string{
        defPath, // 默认路径
        filepath.Join("/usr/local/share/v2ray/", file), // 拼接 /usr/local/share/v2ray/ 和文件名
        filepath.Join("/usr/share/v2ray/", file), // 拼接 /usr/share/v2ray/ 和文件名
    } {
        if _, err := os.Stat(p); os.IsNotExist(err) { // 判断路径是否存在
            continue // 如果路径不存在，继续下一个路径的判断
        }

        // asset found
        return p // 如果路径存在，返回该路径
    }

    // asset not found, let the caller throw out the error
    return defPath // 如果所有路径都不存在，返回默认路径
}
```