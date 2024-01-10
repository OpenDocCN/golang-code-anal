# `v2ray-core\common\platform\platform_test.go`

```
package platform_test

import (
    "os"  // 导入操作系统相关的包
    "path/filepath"  // 导入处理文件路径的包
    "runtime"  // 导入运行时相关的包
    "testing"  // 导入测试相关的包

    "v2ray.com/core/common"  // 导入通用的包
    . "v2ray.com/core/common/platform"  // 导入平台相关的包
)

func TestNormalizeEnvName(t *testing.T) {
    cases := []struct {  // 定义测试用例
        input  string  // 输入字符串
        output string  // 期望输出字符串
    }{
        {
            input:  "a",  // 输入为 "a"
            output: "A",  // 期望输出为 "A"
        },
        {
            input:  "a.a",  // 输入为 "a.a"
            output: "A_A",  // 期望输出为 "A_A"
        },
        {
            input:  "A.A.B",  // 输入为 "A.A.B"
            output: "A_A_B",  // 期望输出为 "A_A_B"
        },
    }
    for _, test := range cases {  // 遍历测试用例
        if v := NormalizeEnvName(test.input); v != test.output {  // 调用函数并检查输出是否符合期望
            t.Error("unexpected output: ", v, " want ", test.output)  // 输出错误信息
        }
    }
}

func TestEnvFlag(t *testing.T) {
    if v := (EnvFlag{  // 调用 EnvFlag 结构体
        Name: "xxxxx.y",  // 设置 Name 字段
    }.GetValueAsInt(10)); v != 10 {  // 调用 GetValueAsInt 方法并检查输出是否符合期望
        t.Error("env value: ", v)  // 输出错误信息
    }
}

func TestGetAssetLocation(t *testing.T) {
    exec, err := os.Executable()  // 获取可执行文件路径
    common.Must(err)  // 检查错误

    loc := GetAssetLocation("t")  // 获取资源位置
    if filepath.Dir(loc) != filepath.Dir(exec) {  // 检查资源位置是否在可执行文件路径中
        t.Error("asset dir: ", loc, " not in ", exec)  // 输出错误信息
    }

    os.Setenv("v2ray.location.asset", "/v2ray")  // 设置环境变量

    if runtime.GOOS == "windows" {  // 如果是 Windows 系统
        if v := GetAssetLocation("t"); v != "\\v2ray\\t" {  // 获取资源位置并检查是否符合期望
            t.Error("asset loc: ", v)  // 输出错误信息
        }
    } else {  // 如果是其他系统
        if v := GetAssetLocation("t"); v != "/v2ray/t" {  // 获取资源位置并检查是否符合期望
            t.Error("asset loc: ", v)  // 输出错误信息
        }
    }
}
```