# `grype\cmd\grype\cli\commands\db_import.go`

```
package commands

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"github.com/spf13/cobra"  // 导入 cobra 包，用于创建命令行应用
	"github.com/anchore/clio"  // 导入 anchore/clio 包
	"github.com/anchore/grype/cmd/grype/cli/options"  // 导入 anchore/grype/cmd/grype/cli/options 包
	"github.com/anchore/grype/grype/db"  // 导入 anchore/grype/grype/db 包
	"github.com/anchore/grype/internal"  // 导入 anchore/grype/internal 包
)

// 定义一个函数，用于导入漏洞数据库
func DBImport(app clio.Application) *cobra.Command {
	// 设置数据库导入的默认选项
	opts := dbOptionsDefault(app.ID())

	// 设置命令行应用的命令
	return app.SetupCommand(&cobra.Command{
		Use:   "import FILE",  // 命令使用说明
		Short: "import a vulnerability database archive",  // 命令简短说明
		Long:  fmt.Sprintf("import a vulnerability database archive from a local FILE.\nDB archives can be obtained from %q.", internal.DBUpdateURL),  // 命令详细说明
```
		Args:  cobra.ExactArgs(1),  // 设置命令行参数为1个
		RunE: func(cmd *cobra.Command, args []string) error {  // 定义命令行执行函数
			return runDBImport(opts.DB, args[0])  // 调用runDBImport函数，传入数据库选项和数据库归档路径作为参数
		},
	}, opts)  // 传入命令选项

func runDBImport(opts options.Database, dbArchivePath string) error {  // 定义runDBImport函数，传入数据库选项和数据库归档路径作为参数
	dbCurator, err := db.NewCurator(opts.ToCuratorConfig())  // 根据数据库选项创建数据库管理员对象
	if err != nil {  // 如果出现错误
		return err  // 返回错误
	}

	if err := dbCurator.ImportFrom(dbArchivePath); err != nil {  // 调用数据库管理员对象的ImportFrom方法，传入数据库归档路径作为参数
		return fmt.Errorf("unable to import vulnerability database: %+v", err)  // 返回导入数据库时出现的错误
	}

	return stderrPrintLnf("Vulnerability database imported")  // 返回导入成功的消息
}
```