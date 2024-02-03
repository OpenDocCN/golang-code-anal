# `grype\cmd\grype\cli\commands\db_check.go`

```go
package commands

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，用于操作系统功能

    "github.com/spf13/cobra" // 导入 cobra 包，用于创建命令行应用

    "github.com/anchore/clio" // 导入 anchore/clio 包
    "github.com/anchore/grype/cmd/grype/cli/options" // 导入 anchore/grype/cmd/grype/cli/options 包
    "github.com/anchore/grype/grype/db" // 导入 anchore/grype/grype/db 包
)

const (
    exitCodeOnDBUpgradeAvailable = 100 // 定义常量，表示数据库可用更新时的退出码
)

func DBCheck(app clio.Application) *cobra.Command {
    opts := dbOptionsDefault(app.ID()) // 调用 dbOptionsDefault 函数，获取数据库选项

    return app.SetupCommand(&cobra.Command{ // 设置命令行应用
        Use:   "check", // 设置命令名称
        Short: "check to see if there is a database update available", // 设置命令的简短描述
        Args:  cobra.ExactArgs(0), // 设置命令的参数数量
        RunE: func(cmd *cobra.Command, args []string) error { // 设置命令的运行函数
            return runDBCheck(opts.DB) // 调用 runDBCheck 函数，执行数据库检查
        },
    }, opts)
}

func runDBCheck(opts options.Database) error {
    dbCurator, err := db.NewCurator(opts.ToCuratorConfig()) // 调用 NewCurator 函数，创建数据库管理员
    if err != nil {
        return err // 如果创建数据库管理员出错，返回错误
    }

    updateAvailable, currentDBMetadata, updateDBEntry, err := dbCurator.IsUpdateAvailable() // 调用 IsUpdateAvailable 函数，检查是否有更新
    if err != nil {
        return fmt.Errorf("unable to check for vulnerability database update: %+v", err) // 如果检查更新出错，返回错误
    }

    if !updateAvailable {
        return stderrPrintLnf("No update available") // 如果没有更新，输出提示信息
    }

    fmt.Println("Update available!") // 输出提示信息

    if currentDBMetadata != nil {
        fmt.Printf("Current DB version %d was built on %s\n", currentDBMetadata.Version, currentDBMetadata.Built.String()) // 输出当前数据库版本信息
    }

    fmt.Printf("Updated DB version %d was built on %s\n", updateDBEntry.Version, updateDBEntry.Built.String()) // 输出更新后数据库版本信息
    fmt.Printf("Updated DB URL: %s\n", updateDBEntry.URL.String()) // 输出更新后数据库的 URL
    fmt.Println("You can run 'grype db update' to update to the latest db") // 输出提示信息

    os.Exit(exitCodeOnDBUpgradeAvailable) //nolint:gocritic // 以指定的退出码退出程序

    return nil // 返回空值
}
```