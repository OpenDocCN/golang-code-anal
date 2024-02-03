# `grype\cmd\grype\main.go`

```go
package main

import (
    _ "github.com/glebarez/sqlite"  // 导入 sqlite 包，但不直接使用它
    "github.com/anchore/clio"       // 导入 clio 包
    "github.com/anchore/grype/cmd/grype/cli"  // 导入 grype 包中的 cli 模块
    "github.com/anchore/grype/cmd/grype/internal"  // 导入 grype 包中的 internal 模块
)

// applicationName 是应用程序的非大写名称（请勿更改此值）
const applicationName = "grype"

// 所有变量都作为构建时参数提供，具有清晰的默认值
var (
    version        = internal.NotProvided  // 版本号，默认值为 NotProvided
    buildDate      = internal.NotProvided  // 构建日期，默认值为 NotProvided
    gitCommit      = internal.NotProvided  // Git 提交号，默认值为 NotProvided
    gitDescription = internal.NotProvided  // Git 描述，默认值为 NotProvided
)

func main() {
    // 创建一个应用程序实例
    app := cli.Application(
        clio.Identification{
            Name:           applicationName,  // 设置应用程序名称
            Version:        version,  // 设置应用程序版本号
            BuildDate:      buildDate,  // 设置应用程序构建日期
            GitCommit:      gitCommit,  // 设置 Git 提交号
            GitDescription: gitDescription,  // 设置 Git 描述
        },
    )

    // 运行应用程序
    app.Run()
}
```