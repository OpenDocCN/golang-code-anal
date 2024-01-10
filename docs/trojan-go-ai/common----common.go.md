# `trojan-go\common\common.go`

```
package common

import (
    "crypto/sha256"  // 导入加密算法包
    "fmt"  // 导入格式化包
    "os"  // 导入操作系统功能包
    "path/filepath"  // 导入文件路径操作包

    "github.com/p4gefau1t/trojan-go/log"  // 导入日志包
)

type Runnable interface {  // 定义接口类型 Runnable
    Run() error  // 接口方法 Run，返回错误
    Close() error  // 接口方法 Close，返回错误
}

func SHA224String(password string) string {  // 计算 SHA224 哈希值的函数
    hash := sha256.New224()  // 创建 SHA224 哈希对象
    hash.Write([]byte(password))  // 写入密码数据
    val := hash.Sum(nil)  // 计算哈希值
    str := ""  // 初始化空字符串
    for _, v := range val {  // 遍历哈希值
        str += fmt.Sprintf("%02x", v)  // 格式化为十六进制字符串并拼接
    }
    return str  // 返回哈希值字符串
}

func GetProgramDir() string {  // 获取程序所在目录的函数
    dir, err := filepath.Abs(filepath.Dir(os.Args[0]))  // 获取程序所在目录的绝对路径
    if err != nil {  // 如果发生错误
        log.Fatal(err)  // 记录错误日志并退出程序
    }
    return dir  // 返回程序所在目录的绝对路径
}

func GetAssetLocation(file string) string {  // 获取资源文件位置的函数
    if filepath.IsAbs(file) {  // 如果文件路径是绝对路径
        return file  // 直接返回文件路径
    }
    if loc := os.Getenv("TROJAN_GO_LOCATION_ASSET"); loc != "" {  // 如果环境变量 TROJAN_GO_LOCATION_ASSET 存在
        absPath, err := filepath.Abs(loc)  // 获取环境变量指定路径的绝对路径
        if err != nil {  // 如果发生错误
            log.Fatal(err)  // 记录错误日志并退出程序
        }
        log.Debugf("env set: TROJAN_GO_LOCATION_ASSET=%s", absPath)  // 记录调试日志
        return filepath.Join(absPath, file)  // 返回环境变量指定路径和文件名拼接后的路径
    }
    return filepath.Join(GetProgramDir(), file)  // 返回程序所在目录和文件名拼接后的路径
}
```