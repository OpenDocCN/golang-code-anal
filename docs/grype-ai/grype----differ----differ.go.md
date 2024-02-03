# `grype\grype\differ\differ.go`

```go
package differ

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化输出包
    "io"  // 导入输入输出包
    "net/url"  // 导入 URL 处理包
    "path"  // 导入路径处理包

    "github.com/olekukonko/tablewriter"  // 导入表格输出包
    "github.com/wagoodman/go-partybus"  // 导入事件总线包
    "github.com/wagoodman/go-progress"  // 导入进度条包

    "github.com/anchore/grype/grype/db"  // 导入数据库包
    v5 "github.com/anchore/grype/grype/db/v5"  // 导入数据库 v5 版本包
    "github.com/anchore/grype/grype/event"  // 导入事件包
    "github.com/anchore/grype/internal/bus"  // 导入内部事件总线包
)

type Differ struct {
    baseCurator   db.Curator  // 定义基准数据库的 Curator
    targetCurator db.Curator  // 定义目标数据库的 Curator
}

func NewDiffer(config db.Config) (*Differ, error) {
    baseCurator, err := db.NewCurator(db.Config{  // 创建基准数据库的 Curator
        DBRootDir:           path.Join(config.DBRootDir, "diff", "base"),  // 设置数据库根目录
        ListingURL:          config.ListingURL,  // 设置数据库列表 URL
        CACert:              config.CACert,  // 设置 CA 证书
        ValidateByHashOnGet: config.ValidateByHashOnGet,  // 设置获取时的哈希验证
    })
    if err != nil {
        return nil, err
    }

    targetCurator, err := db.NewCurator(db.Config{  // 创建目标数据库的 Curator
        DBRootDir:           path.Join(config.DBRootDir, "diff", "target"),  // 设置数据库根目录
        ListingURL:          config.ListingURL,  // 设置数据库列表 URL
        CACert:              config.CACert,  // 设置 CA 证书
        ValidateByHashOnGet: config.ValidateByHashOnGet,  // 设置获取时的哈希验证
    })
    if err != nil {
        return nil, err
    }

    return &Differ{  // 返回 Differ 对象
        baseCurator:   baseCurator,  // 设置基准数据库的 Curator
        targetCurator: targetCurator,  // 设置目标数据库的 Curator
    }, nil
}

func (d *Differ) SetBaseDB(base string) error {
    return d.setOrDownload(&d.baseCurator, base)  // 设置或下载基准数据库
}

func (d *Differ) SetTargetDB(target string) error {
    return d.setOrDownload(&d.targetCurator, target)  // 设置或下载目标数据库
}

func (d *Differ) setOrDownload(curator *db.Curator, filenameOrURL string) error {
    u, err := url.ParseRequestURI(filenameOrURL)  // 解析文件名或 URL

    if err != nil || u.Scheme == "" {  // 如果解析出错或没有 Scheme
        *curator, err = db.NewCurator(db.Config{  // 创建新的 Curator
            DBRootDir: filenameOrURL,  // 设置数据库根目录
        })
        if err != nil {
            return err
        }
    } else {
        // 从基础策展人对象获取 URL 对应的列表
        listings, err := d.baseCurator.ListingFromURL()
        // 如果出现错误，返回错误信息
        if err != nil {
            return err
        }

        // 获取可用的数据库
        available := listings.Available
        dbs := available[v5.SchemaVersion]

        // 定义一个列表条目指针
        var listing *db.ListingEntry

        // 遍历数据库列表
        for _, d := range dbs {
            // 将数据库赋值给临时变量
            database := d
            // 如果数据库的 URL 与给定的 URL 相同，将列表条目指针指向该数据库
            if *d.URL == *u {
                listing = &database
            }
        }

        // 如果未找到列表条目，返回错误信息
        if listing == nil {
            return fmt.Errorf("unable to find listing for url: %s", filenameOrURL)
        }

        // 下载漏洞数据库
        if err := download(curator, listing); err != nil {
            return fmt.Errorf("unable to download vulnerability database: %+v", err)
        }
    }

    // 返回空值
    return nil
}

func download(curator *db.Curator, listing *db.ListingEntry) error {
    // 创建一个手动进度条，用于下载和导入阶段
    importProgress := progress.NewManual(1)
    // 创建一个原子阶段，用于检查可用数据库
    stage := progress.NewAtomicStage("checking available databases")
    // 创建一个手动进度条，用于下载阶段
    downloadProgress := progress.NewManual(1)
    // 创建一个进度聚合器，用于聚合下载和导入进度
    aggregateProgress := progress.NewAggregator(progress.DefaultStrategy, downloadProgress, importProgress)

    // 发布一个事件，通知消费者有一个可监控的事件（下载 + 导入阶段）
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

    // 延迟设置下载进度条为已完成
    defer downloadProgress.SetCompleted()
    // 延迟设置导入进度条为已完成
    defer importProgress.SetCompleted()

    // 调用 curator.UpdateTo 方法，传入下载进度条、导入进度条和阶段
    return curator.UpdateTo(listing, downloadProgress, importProgress, stage)
}

func (d *Differ) DiffDatabases() (*[]v5.Diff, error) {
    // 获取基础数据库存储、关闭函数和错误信息
    baseStore, baseDBCloser, err := d.baseCurator.GetStore()
    if err != nil {
        return nil, err
    }

    // 延迟关闭基础数据库存储
    defer baseDBCloser.Close()

    // 获取目标数据库存储、关闭函数和错误信息
    targetStore, targetDBCloser, err := d.targetCurator.GetStore()
    if err != nil {
        return nil, err
    }

    // 延迟关闭目标数据库存储
    defer targetDBCloser.Close()

    // 返回基础数据库存储和目标数据库存储的差异
    return baseStore.DiffStore(targetStore)
}

func (d *Differ) DeleteDatabases() error {
    // 删除基础数据库，如果出现错误则返回错误信息
    if err := d.baseCurator.Delete(); err != nil {
        return fmt.Errorf("unable to delete vulnerability database: %+v", err)
    }
    // 删除目标数据库，如果出现错误则返回错误信息
    if err := d.targetCurator.Delete(); err != nil {
        return fmt.Errorf("unable to delete vulnerability database: %+v", err)
    }
    // 返回空错误信息
    return nil
}

func (d *Differ) Present(outputFormat string, diff *[]v5.Diff, output io.Writer) error {
    // 如果差异为空，则返回空
    if diff == nil {
        return nil
    }

    // 根据输出格式进行处理
    switch outputFormat {
    # 根据输出格式进行不同的处理
    case "table":
        # 初始化一个二维字符串数组
        rows := [][]string{}
        # 遍历 diff 列表，将每个元素的属性组成一个字符串数组，并添加到 rows 中
        for _, d := range *diff {
            rows = append(rows, []string{d.ID, d.Namespace, d.Reason})
        }

        # 创建一个新的表格写入器，并指定输出目标
        table := tablewriter.NewWriter(output)
        # 定义表格的列名
        columns := []string{"ID", "Namespace", "Reason"}

        # 设置表头
        table.SetHeader(columns)
        # 设置自动换行文本
        table.SetAutoWrapText(false)
        # 设置表头对齐方式
        table.SetHeaderAlignment(tablewriter.ALIGN_LEFT)
        # 设置内容对齐方式
        table.SetAlignment(tablewriter.ALIGN_LEFT)

        # 设置表头线
        table.SetHeaderLine(false)
        # 设置边框
        table.SetBorder(false)
        # 设置自动格式化表头
        table.SetAutoFormatHeaders(true)
        # 设置中心分隔符
        table.SetCenterSeparator("")
        # 设置列分隔符
        table.SetColumnSeparator("")
        # 设置行分隔符
        table.SetRowSeparator("")
        # 设置表格填充
        table.SetTablePadding("  ")
        # 设置无空白
        table.SetNoWhiteSpace(true)

        # 批量添加行数据
        table.AppendBulk(rows)
        # 渲染表格
        table.Render()
    case "json":
        # 创建一个新的 JSON 编码器，并指定输出目标
        enc := json.NewEncoder(output)
        # 设置是否转义 HTML
        enc.SetEscapeHTML(false)
        # 设置缩进格式
        enc.SetIndent("", " ")
        # 将 diff 列表编码为 JSON 格式，并写入输出目标
        if err := enc.Encode(*diff); err != nil {
            return fmt.Errorf("failed to encode diff information: %+v", err)
        }
    default:
        # 返回不支持的输出格式错误
        return fmt.Errorf("unsupported output format: %s", outputFormat)
    }
    # 返回空值
    return nil
# 闭合前面的函数定义
```