# `v2ray-core\main\confloader\confloader.go`

```
package confloader

import (
    "io"  // 导入io包，提供了基本的输入输出功能
    "os"  // 导入os包，提供了操作系统功能
)

type configFileLoader func(string) (io.Reader, error)  // 定义了一个函数类型 configFileLoader，接受一个string参数，返回io.Reader和error
type extconfigLoader func([]string) (io.Reader, error)  // 定义了一个函数类型 extconfigLoader，接受一个[]string参数，返回io.Reader和error

var (
    EffectiveConfigFileLoader configFileLoader  // 定义了一个全局变量 EffectiveConfigFileLoader，类型为configFileLoader
    EffectiveExtConfigLoader  extconfigLoader  // 定义了一个全局变量 EffectiveExtConfigLoader，类型为extconfigLoader
)

// LoadConfig reads from a path/url/stdin
// actual work is in external module
func LoadConfig(file string) (io.Reader, error) {
    if EffectiveConfigFileLoader == nil {  // 如果EffectiveConfigFileLoader为空
        newError("external config module not loaded, reading from stdin").AtInfo().WriteToLog()  // 调用newError函数，记录日志信息
        return os.Stdin, nil  // 返回标准输入流和nil
    }
    return EffectiveConfigFileLoader(file)  // 调用EffectiveConfigFileLoader函数，传入file参数
}

// LoadExtConfig calls v2ctl to handle multiple config
// the actual work also in external module
func LoadExtConfig(files []string) (io.Reader, error) {
    if EffectiveExtConfigLoader == nil {  // 如果EffectiveExtConfigLoader为空
        return nil, newError("external config module not loaded").AtError()  // 返回nil和调用newError函数的结果
    }

    return EffectiveExtConfigLoader(files)  // 调用EffectiveExtConfigLoader函数，传入files参数
}
```