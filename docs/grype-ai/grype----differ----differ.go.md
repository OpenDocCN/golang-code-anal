# `grype\grype\differ\differ.go`

```
// 定义一个名为differ的包
package differ

// 导入所需的包
import (
	"encoding/json"  // 导入json包，用于处理json数据
	"fmt"  // 导入fmt包，用于格式化输入输出
	"io"  // 导入io包，用于进行输入输出操作
	"net/url"  // 导入net/url包，用于URL解析
	"path"  // 导入path包，用于处理文件路径
	"github.com/olekukonko/tablewriter"  // 导入tablewriter包，用于生成表格
	"github.com/wagoodman/go-partybus"  // 导入go-partybus包，用于事件总线
	"github.com/wagoodman/go-progress"  // 导入go-progress包，用于显示进度条

	"github.com/anchore/grype/grype/db"  // 导入grype/db包
	v5 "github.com/anchore/grype/grype/db/v5"  // 导入v5包
	"github.com/anchore/grype/grype/event"  // 导入event包
	"github.com/anchore/grype/internal/bus"  // 导入bus包
)

// 定义一个名为Differ的结构体
type Differ struct {
# 定义了两个数据库管理员对象，用于比较两个数据库之间的差异
baseCurator   db.Curator
targetCurator db.Curator

# 创建一个新的差异比较器对象
func NewDiffer(config db.Config) (*Differ, error) {
    # 创建一个基准数据库管理员对象
    baseCurator, err := db.NewCurator(db.Config{
        DBRootDir:           path.Join(config.DBRootDir, "diff", "base"),
        ListingURL:          config.ListingURL,
        CACert:              config.CACert,
        ValidateByHashOnGet: config.ValidateByHashOnGet,
    })
    # 如果创建过程中出现错误，则返回错误信息
    if err != nil {
        return nil, err
    }

    # 创建一个目标数据库管理员对象
    targetCurator, err := db.NewCurator(db.Config{
        DBRootDir:           path.Join(config.DBRootDir, "diff", "target"),
        ListingURL:          config.ListingURL,
        CACert:              config.CACert,
        ValidateByHashOnGet: config.ValidateByHashOnGet,
    })
	})
	// 如果发生错误，返回空指针和错误信息
	if err != nil {
		return nil, err
	}

	// 返回一个新的 Differ 对象，包含 baseCurator 和 targetCurator
	return &Differ{
		baseCurator:   baseCurator,
		targetCurator: targetCurator,
	}, nil
}

// 设置基础数据库
func (d *Differ) SetBaseDB(base string) error {
	// 调用 setOrDownload 方法设置或下载基础数据库
	return d.setOrDownload(&d.baseCurator, base)
}

// 设置目标数据库
func (d *Differ) SetTargetDB(target string) error {
	// 调用 setOrDownload 方法设置或下载目标数据库
	return d.setOrDownload(&d.targetCurator, target)
}

// 设置或下载数据库
func (d *Differ) setOrDownload(curator *db.Curator, filenameOrURL string) error {
// 解析文件名或 URL，返回解析结果和可能的错误
u, err := url.ParseRequestURI(filenameOrURL)

// 如果解析出错或者没有指定协议，创建一个新的 Curator 对象并初始化
if err != nil || u.Scheme == "" {
    *curator, err = db.NewCurator(db.Config{
        DBRootDir: filenameOrURL,
    })
    // 如果初始化出错，返回错误
    if err != nil {
        return err
    }
} else {
    // 从 URL 获取列表信息
    listings, err := d.baseCurator.ListingFromURL()
    // 如果获取列表信息出错，返回错误
    if err != nil {
        return err
    }

    // 获取可用的列表信息和对应的数据库
    available := listings.Available
    dbs := available[v5.SchemaVersion]

    var listing *db.ListingEntry
}
		// 遍历 dbs 列表中的每个数据库
		for _, d := range dbs {
			// 将当前数据库赋值给变量 database
			database := d
			// 如果当前数据库的 URL 与给定的 URL 相同
			if *d.URL == *u {
				// 将当前数据库赋值给 listing
				listing = &database
			}
		}

		// 如果未找到匹配的数据库
		if listing == nil {
			// 返回错误信息，指示无法找到对应 URL 的数据库
			return fmt.Errorf("unable to find listing for url: %s", filenameOrURL)
		}

		// 调用 download 函数下载漏洞数据库
		if err := download(curator, listing); err != nil {
			// 如果下载出现错误，返回错误信息
			return fmt.Errorf("unable to download vulnerability database: %+v", err)
		}
	}

	// 如果没有出现错误，返回 nil
	return nil
}

// download 函数用于下载漏洞数据库
func download(curator *db.Curator, listing *db.ListingEntry) error {
// 创建一个手动进度条，用于通知消费者监控事件（下载和导入阶段）
importProgress := progress.NewManual(1)

// 创建一个原子阶段，用于表示“检查可用数据库”阶段
stage := progress.NewAtomicStage("checking available databases")

// 创建一个手动进度条，用于表示下载阶段的进度
downloadProgress := progress.NewManual(1)

// 创建一个进度聚合器，用于将下载和导入阶段的进度进行聚合
aggregateProgress := progress.NewAggregator(progress.DefaultStrategy, downloadProgress, importProgress)

// 发布一个事件，通知消费者更新漏洞数据库的进度
bus.Publish(partybus.Event{
    Type: event.UpdateVulnerabilityDatabase,
    Value: progress.StagedProgressable(&struct {
        progress.Stager
        progress.Progressable
    }{
        Stager:       progress.Stager(stage),
        Progressable: progress.Progressable(aggregateProgress),
    }),
})

// 在函数结束时，标记下载阶段为已完成
defer downloadProgress.SetCompleted()

// 在函数结束时，标记导入阶段为已完成
defer importProgress.SetCompleted()
# 调用curator.UpdateTo方法，传入参数listing, downloadProgress, importProgress, stage，并返回结果
return curator.UpdateTo(listing, downloadProgress, importProgress, stage)
}

# DiffDatabases方法用于比较数据库的差异，返回差异结果和可能的错误
func (d *Differ) DiffDatabases() (*[]v5.Diff, error) {
    # 通过baseCurator获取存储，返回存储对象baseStore、关闭存储的函数baseDBCloser和可能的错误
    baseStore, baseDBCloser, err := d.baseCurator.GetStore()
    if err != nil {
        return nil, err
    }

    # 延迟执行关闭存储的函数baseDBCloser
    defer baseDBCloser.Close()

    # 通过targetCurator获取存储，返回存储对象targetStore、关闭存储的函数targetDBCloser和可能的错误
    targetStore, targetDBCloser, err := d.targetCurator.GetStore()
    if err != nil {
        return nil, err
    }

    # 延迟执行关闭存储的函数targetDBCloser
    defer targetDBCloser.Close()

    # 调用baseStore的DiffStore方法，传入targetStore作为参数，返回比较结果和可能的错误
    return baseStore.DiffStore(targetStore)
}
# 删除数据库操作，首先删除基准数据库，如果失败则返回错误信息
func (d *Differ) DeleteDatabases() error {
	if err := d.baseCurator.Delete(); err != nil {
		return fmt.Errorf("unable to delete vulnerability database: %+v", err)
	}
	# 删除目标数据库，如果失败则返回错误信息
	if err := d.targetCurator.Delete(); err != nil {
		return fmt.Errorf("unable to delete vulnerability database: %+v", err)
	}
	# 操作成功，返回空值
	return nil
}

# 根据指定的输出格式和差异数据，将差异数据输出到指定的输出流中
func (d *Differ) Present(outputFormat string, diff *[]v5.Diff, output io.Writer) error {
	# 如果差异数据为空，则直接返回空值
	if diff == nil {
		return nil
	}

	# 根据输出格式进行不同的处理
	switch outputFormat {
	case "table":
		# 初始化一个二维字符串数组用于存储表格数据
		rows := [][]string{}
		# 遍历差异数据，处理每一条差异数据
		for _, d := range *diff {
# 将包含数据的切片添加到rows切片中
rows = append(rows, []string{d.ID, d.Namespace, d.Reason})

# 创建一个新的表格写入器，并将输出作为参数
table = tablewriter.NewWriter(output)

# 定义表格的列名
columns = []string{"ID", "Namespace", "Reason"}

# 设置表头
table.SetHeader(columns)

# 设置自动换行文本为false
table.SetAutoWrapText(false)

# 设置表头对齐方式为左对齐
table.SetHeaderAlignment(tablewriter.ALIGN_LEFT)

# 设置表格内容对齐方式为左对齐
table.SetAlignment(tablewriter.ALIGN_LEFT)

# 设置表头线为false
table.SetHeaderLine(false)

# 设置表格边框为false
table.SetBorder(false)

# 设置自动格式化表头为true
table.SetAutoFormatHeaders(true)

# 设置中心分隔符为空
table.SetCenterSeparator("")

# 设置列分隔符为空
table.SetColumnSeparator("")

# 设置行分隔符为空
table.SetRowSeparator("")

# 设置表格填充为两个空格
table.SetTablePadding("  ")

# 设置无空白为true
table.SetNoWhiteSpace(true)
		// 将数据批量添加到表格中
		table.AppendBulk(rows)
		// 渲染表格
		table.Render()
	case "json":
		// 创建一个 JSON 编码器，并设置不转义 HTML，以及缩进格式
		enc := json.NewEncoder(output)
		enc.SetEscapeHTML(false)
		enc.SetIndent("", " ")
		// 将差异信息编码为 JSON 格式，并输出到指定的输出流
		if err := enc.Encode(*diff); err != nil {
			return fmt.Errorf("failed to encode diff information: %+v", err)
		}
	default:
		// 如果输出格式不是表格或 JSON，则返回不支持的输出格式错误
		return fmt.Errorf("unsupported output format: %s", outputFormat)
	}
	// 返回空错误，表示操作成功
	return nil
}
```