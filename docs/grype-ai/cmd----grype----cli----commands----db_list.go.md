# `grype\cmd\grype\cli\commands\db_list.go`

```
// 导入必要的包
package commands

import (
	"encoding/json" // 导入 JSON 编解码包
	"fmt" // 导入格式化输出包
	"os" // 导入操作系统包

	"github.com/spf13/cobra" // 导入命令行解析包

	"github.com/anchore/clio" // 导入 anchore/clio 包
	"github.com/anchore/grype/grype/db" // 导入 anchore/grype/grype/db 包
)

// 定义 dbListOptions 结构体
type dbListOptions struct {
	Output    string `yaml:"output" json:"output" mapstructure:"output"` // 定义输出选项
	DBOptions `yaml:",inline" mapstructure:",squash"` // 引用 DBOptions 结构体
}

// 实现 FlagAdder 接口
var _ clio.FlagAdder = (*dbListOptions)(nil)
// AddFlags 方法用于向给定的 FlagSet 中添加标志，这里是为 dbListOptions 结构体中的 Output 字段添加标志
func (d *dbListOptions) AddFlags(flags clio.FlagSet) {
    // 使用 flags.StringVarP 方法为 Output 字段添加标志，设置了标志的名称、简写、默认值和说明
    flags.StringVarP(&d.Output, "output", "o", "format to display results (available=[text, raw, json])")
}

// DBList 函数用于创建一个 Cobra 命令，用于列出所有可用的数据库
func DBList(app clio.Application) *cobra.Command {
    // 创建 dbListOptions 结构体的实例 opts，并设置了 Output 字段的默认值和 DBOptions 字段的值
    opts := &dbListOptions{
        Output:    "text",
        DBOptions: *dbOptionsDefault(app.ID()),
    }

    // 设置命令的基本信息、参数要求和运行时的操作
    return app.SetupCommand(&cobra.Command{
        Use:   "list",
        Short: "list all DBs available according to the listing URL",
        Args:  cobra.ExactArgs(0),
        RunE: func(cmd *cobra.Command, args []string) error {
            // 运行 runDBList 函数，并传入 opts 结构体作为参数
            return runDBList(opts)
        },
    }, opts)
}
# 运行数据库列表操作，接受一个数据库列表选项参数，返回错误信息
func runDBList(opts *dbListOptions) error {
    # 根据数据库列表选项参数创建数据库管理员对象
    dbCurator, err := db.NewCurator(opts.DB.ToCuratorConfig())
    if err != nil {
        return err
    }

    # 从数据库管理员对象获取数据库列表
    listing, err := dbCurator.ListingFromURL()
    if err != nil {
        return err
    }

    # 获取数据库管理员支持的数据库模式
    supportedSchema := dbCurator.SupportedSchema()
    # 检查支持的数据库模式是否在列表中
    available, exists := listing.Available[supportedSchema]

    # 如果可用数据库列表为空或者支持的数据库模式不存在，则返回错误信息
    if len(available) == 0 || !exists {
        return stderrPrintLnf("No databases available for the current schema (%d)", supportedSchema)
    }

    # 根据输出选项进行不同的操作
    switch opts.Output {
    case "text":
// 遍历可用数据库列表，打印每个数据库的构建时间、URL和校验和
for _, l := range available {
    fmt.Printf("Built:    %s\n", l.Built)
    fmt.Printf("URL:      %s\n", l.URL)
    fmt.Printf("Checksum: %s\n\n", l.Checksum)
}

// 打印当前模式下可用数据库的数量
fmt.Printf("%d databases available for schema %d\n", len(available), supportedSchema)

// 如果命令行参数为 "json"，则以 JSON 格式输出当前模式下的数据库列表
enc := json.NewEncoder(os.Stdout)
enc.SetEscapeHTML(false)
enc.SetIndent("", " ")
if err := enc.Encode(&available); err != nil {
    return fmt.Errorf("failed to db listing information: %+v", err)
}

// 如果命令行参数为 "raw"，则以 JSON 格式输出整个数据库列表文件
enc := json.NewEncoder(os.Stdout)
enc.SetEscapeHTML(false)
		// 设置编码器的缩进格式为空格
		enc.SetIndent("", " ")
		// 使用编码器将列表信息编码并写入输出流，如果出现错误则返回错误信息
		if err := enc.Encode(&listing); err != nil {
			return fmt.Errorf("failed to db listing information: %+v", err)
		}
	// 如果不是默认输出格式，则返回不支持的输出格式错误
	default:
		return fmt.Errorf("unsupported output format: %s", opts.Output)
	}

	// 返回空值
	return nil
}
```