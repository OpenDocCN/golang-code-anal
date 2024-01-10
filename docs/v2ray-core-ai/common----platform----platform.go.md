# `v2ray-core\common\platform\platform.go`

```
// 导入所需的包
package platform // import "v2ray.com/core/common/platform"

import (
    "os" // 导入操作系统相关的包
    "path/filepath" // 导入处理文件路径的包
    "strconv" // 导入字符串转换相关的包
    "strings" // 导入处理字符串相关的包
)

// 定义 EnvFlag 结构体
type EnvFlag struct {
    Name    string // 环境变量名称
    AltName string // 备用环境变量名称
}

// 创建新的 EnvFlag 对象
func NewEnvFlag(name string) EnvFlag {
    return EnvFlag{
        Name:    name,
        AltName: NormalizeEnvName(name), // 调用 NormalizeEnvName 函数处理备用环境变量名称
    }
}

// 获取环境变量的值
func (f EnvFlag) GetValue(defaultValue func() string) string {
    if v, found := os.LookupEnv(f.Name); found { // 查找环境变量的值
        return v
    }
    if len(f.AltName) > 0 { // 如果备用环境变量名称不为空
        if v, found := os.LookupEnv(f.AltName); found { // 查找备用环境变量的值
            return v
        }
    }

    return defaultValue() // 返回默认值
}

// 获取环境变量的值并转换为整数
func (f EnvFlag) GetValueAsInt(defaultValue int) int {
    useDefaultValue := false
    s := f.GetValue(func() string {
        useDefaultValue = true
        return ""
    })
    if useDefaultValue {
        return defaultValue
    }
    v, err := strconv.ParseInt(s, 10, 32) // 将字符串转换为整数
    if err != nil {
        return defaultValue
    }
    return int(v)
}

// 标准化环境变量名称
func NormalizeEnvName(name string) string {
    return strings.Replace(strings.ToUpper(strings.TrimSpace(name)), ".", "_", -1) // 将环境变量名称标准化
}

// 获取可执行文件的目录
func getExecutableDir() string {
    exec, err := os.Executable() // 获取可执行文件的路径
    if err != nil {
        return ""
    }
    return filepath.Dir(exec) // 返回可执行文件的目录
}

// 获取可执行文件的子目录
func getExecutableSubDir(dir string) func() string {
    return func() string {
        return filepath.Join(getExecutableDir(), dir) // 返回可执行文件的子目录
    }
}

// 获取插件目录
func GetPluginDirectory() string {
    const name = "v2ray.location.plugin"
    pluginDir := NewEnvFlag(name).GetValue(getExecutableSubDir("plugins")) // 获取插件目录
    return pluginDir
}

// 获取配置文件路径
func GetConfigurationPath() string {
    const name = "v2ray.location.config"
    configPath := NewEnvFlag(name).GetValue(getExecutableDir) // 获取配置文件路径
    return filepath.Join(configPath, "config.json") // 返回配置文件的完整路径
}

// 读取 "v2ray.location.confdir" 环境变量
func GetConfDirPath() string {
    const name = "v2ray.location.confdir"
    configPath := NewEnvFlag(name).GetValue(func() string { return "" }) // 读取 "v2ray.location.confdir" 环境变量
    return configPath // 返回配置目录路径
}
```