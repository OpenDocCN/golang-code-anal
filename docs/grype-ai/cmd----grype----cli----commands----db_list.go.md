# `grype\cmd\grype\cli\commands\db_list.go`

```go
package commands

import (
    "encoding/json"  // 导入编码/解码 JSON 数据的包
    "fmt"  // 导入格式化输出的包
    "os"  // 导入操作系统功能的包

    "github.com/spf13/cobra"  // 导入命令行应用程序的包

    "github.com/anchore/clio"  // 导入 anchore 应用程序的命令行输入/输出包
    "github.com/anchore/grype/grype/db"  // 导入 anchore 应用程序的数据库包
)

type dbListOptions struct {
    Output    string `yaml:"output" json:"output" mapstructure:"output"`  // 定义数据库列表选项结构体，包含输出格式
    DBOptions `yaml:",inline" mapstructure:",squash"`  // 内嵌 DBOptions 结构体
}

var _ clio.FlagAdder = (*dbListOptions)(nil)  // 实现 FlagAdder 接口

func (d *dbListOptions) AddFlags(flags clio.FlagSet) {
    flags.StringVarP(&d.Output, "output", "o", "format to display results (available=[text, raw, json])")  // 添加输出格式的命令行标志
}

func DBList(app clio.Application) *cobra.Command {
    opts := &dbListOptions{  // 创建数据库列表选项对象
        Output:    "text",  // 设置默认输出格式为文本
        DBOptions: *dbOptionsDefault(app.ID()),  // 使用默认的数据库选项
    }

    return app.SetupCommand(&cobra.Command{  // 设置命令行应用程序
        Use:   "list",  // 使用说明
        Short: "list all DBs available according to the listing URL",  // 简短说明
        Args:  cobra.ExactArgs(0),  // 精确的参数数量
        RunE: func(cmd *cobra.Command, args []string) error {  // 运行命令时执行的函数
            return runDBList(opts)  // 运行数据库列表函数
        },
    }, opts)
}

func runDBList(opts *dbListOptions) error {
    dbCurator, err := db.NewCurator(opts.DB.ToCuratorConfig())  // 创建数据库管理员对象
    if err != nil {
        return err  // 如果出现错误，返回错误信息
    }

    listing, err := dbCurator.ListingFromURL()  // 从 URL 获取数据库列表
    if err != nil {
        return err  // 如果出现错误，返回错误信息
    }

    supportedSchema := dbCurator.SupportedSchema()  // 获取支持的数据库模式
    available, exists := listing.Available[supportedSchema]  // 获取可用的数据库列表

    if len(available) == 0 || !exists {
        return stderrPrintLnf("No databases available for the current schema (%d)", supportedSchema)  // 如果没有可用的数据库，返回错误信息
    }

    switch opts.Output {
    case "text":
        // summarize each listing entry for the current DB schema
        for _, l := range available {
            fmt.Printf("Built:    %s\n", l.Built)  // 格式化输出构建信息
            fmt.Printf("URL:      %s\n", l.URL)  // 格式化输出 URL 信息
            fmt.Printf("Checksum: %s\n\n", l.Checksum)  // 格式化输出校验和信息
        }

        fmt.Printf("%d databases available for schema %d\n", len(available), supportedSchema)  // 格式化输出可用的数据库数量和模式
    # 如果输出格式为 JSON，则显示当前模式的条目
    enc := json.NewEncoder(os.Stdout)
    enc.SetEscapeHTML(false)
    enc.SetIndent("", " ")
    if err := enc.Encode(&available); err != nil {
        return fmt.Errorf("failed to db listing information: %+v", err)
    }
    # 如果输出格式为原始数据，则显示整个列表文件
    enc := json.NewEncoder(os.Stdout)
    enc.SetEscapeHTML(false)
    enc.SetIndent("", " ")
    if err := enc.Encode(&listing); err != nil {
        return fmt.Errorf("failed to db listing information: %+v", err)
    }
    # 如果输出格式不支持，则返回错误信息
    default:
        return fmt.Errorf("unsupported output format: %s", opts.Output)
    }

    return nil
# 闭合前面的函数定义
```