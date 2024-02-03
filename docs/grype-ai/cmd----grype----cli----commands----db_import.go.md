# `grype\cmd\grype\cli\commands\db_import.go`

```go
package commands

import (
    "fmt" // 导入 fmt 包

    "github.com/spf13/cobra" // 导入 cobra 包

    "github.com/anchore/clio" // 导入 clio 包
    "github.com/anchore/grype/cmd/grype/cli/options" // 导入 options 包
    "github.com/anchore/grype/grype/db" // 导入 db 包
    "github.com/anchore/grype/internal" // 导入 internal 包
)

func DBImport(app clio.Application) *cobra.Command {
    opts := dbOptionsDefault(app.ID()) // 使用 app 的 ID 创建默认的数据库选项

    return app.SetupCommand(&cobra.Command{ // 设置命令
        Use:   "import FILE", // 使用说明
        Short: "import a vulnerability database archive", // 简短说明
        Long:  fmt.Sprintf("import a vulnerability database archive from a local FILE.\nDB archives can be obtained from %q.", internal.DBUpdateURL), // 长说明
        Args:  cobra.ExactArgs(1), // 精确的参数数量
        RunE: func(cmd *cobra.Command, args []string) error { // 运行命令
            return runDBImport(opts.DB, args[0]) // 运行数据库导入函数
        },
    }, opts)
}

func runDBImport(opts options.Database, dbArchivePath string) error {
    dbCurator, err := db.NewCurator(opts.ToCuratorConfig()) // 创建数据库管理员
    if err != nil {
        return err // 如果出错，返回错误
    }

    if err := dbCurator.ImportFrom(dbArchivePath); err != nil { // 从指定路径导入数据库
        return fmt.Errorf("unable to import vulnerability database: %+v", err) // 返回导入数据库错误
    }

    return stderrPrintLnf("Vulnerability database imported") // 返回导入成功信息
}
```