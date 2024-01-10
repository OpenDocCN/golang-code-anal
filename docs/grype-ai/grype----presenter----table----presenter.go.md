# `grype\grype\presenter\table\presenter.go`

```
package table

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于输入输出操作
    "sort" // 导入 sort 包，用于排序
    "strings" // 导入 strings 包，用于处理字符串

    "github.com/charmbracelet/lipgloss" // 导入 lipgloss 包，用于生成终端美化输出
    "github.com/olekukonko/tablewriter" // 导入 tablewriter 包，用于生成终端表格输出

    grypeDb "github.com/anchore/grype/grype/db/v5" // 导入 grypeDb 包，用于与漏洞数据库交互
    "github.com/anchore/grype/grype/match" // 导入 match 包，用于匹配漏洞
    "github.com/anchore/grype/grype/pkg" // 导入 pkg 包，用于处理软件包信息
    "github.com/anchore/grype/grype/presenter/models" // 导入 models 包，用于展示模型
    "github.com/anchore/grype/grype/vulnerability" // 导入 vulnerability 包，用于处理漏洞信息
)

const (
    appendSuppressed    = " (suppressed)" // 定义常量，用于标记被抑制的漏洞
    appendSuppressedVEX = " (suppressed by VEX)" // 定义常量，用于标记被 VEX 抑制的漏洞
)

// Presenter is a generic struct for holding fields needed for reporting
type Presenter struct {
    results          match.Matches // 匹配结果
    ignoredMatches   []match.IgnoredMatch // 忽略的匹配结果
    packages         []pkg.Package // 软件包信息
    metadataProvider vulnerability.MetadataProvider // 漏洞元数据提供者
    showSuppressed   bool // 是否展示被抑制的漏洞
    withColor        bool // 是否支持颜色输出
}

// NewPresenter is a *Presenter constructor
func NewPresenter(pb models.PresenterConfig, showSuppressed bool) *Presenter {
    return &Presenter{
        results:          pb.Matches, // 初始化匹配结果
        ignoredMatches:   pb.IgnoredMatches, // 初始化忽略的匹配结果
        packages:         pb.Packages, // 初始化软件包信息
        metadataProvider: pb.MetadataProvider, // 初始化漏洞元数据提供者
        showSuppressed:   showSuppressed, // 初始化是否展示被抑制的漏洞
        withColor:        supportsColor(), // 初始化是否支持颜色输出
    }
}

// Present creates a JSON-based reporting
func (pres *Presenter) Present(output io.Writer) error {
    rows := make([][]string, 0) // 初始化行数据

    columns := []string{"Name", "Installed", "Fixed-In", "Type", "Vulnerability", "Severity"} // 定义表格列名
    // Generate rows for matching vulnerabilities
    for m := range pres.results.Enumerate() { // 遍历匹配结果
        row, err := createRow(m, pres.metadataProvider, "") // 调用 createRow 函数生成行数据
        if err != nil {
            return err // 如果生成行数据出错，返回错误
        }
        rows = append(rows, row) // 将生成的行数据添加到行数组中
    }

    // Generate rows for suppressed vulnerabilities
    # 如果需要展示被抑制的漏洞信息
    if pres.showSuppressed {
        # 遍历被忽略的匹配项
        for _, m := range pres.ignoredMatches {
            # 设置默认消息为追加被抑制的信息
            msg := appendSuppressed
            # 如果应用了忽略规则
            if m.AppliedIgnoreRules != nil {
                # 遍历应用的忽略规则
                for i := range m.AppliedIgnoreRules {
                    # 如果命名空间为 "vex"，则设置消息为追加被抑制的 VEX 信息
                    if m.AppliedIgnoreRules[i].Namespace == "vex" {
                        msg = appendSuppressedVEX
                    }
                }
            }
            # 创建行数据
            row, err := createRow(m.Match, pres.metadataProvider, msg)

            # 如果创建行数据出现错误，则返回错误
            if err != nil {
                return err
            }
            # 将行数据追加到行列表中
            rows = append(rows, row)
        }
    }

    # 如果行列表为空
    if len(rows) == 0 {
        # 向输出中写入消息 "No vulnerabilities found"，并返回可能出现的错误
        _, err := io.WriteString(output, "No vulnerabilities found\n")
        return err
    }

    # 对行列表进行去重和排序
    rows = sortRows(removeDuplicateRows(rows))

    # 创建新的表格写入器
    table := tablewriter.NewWriter(output)
    # 设置表头
    table.SetHeader(columns)
    # 设置自动换行文本为关闭状态
    table.SetAutoWrapText(false)
    # 设置表头对齐方式为左对齐
    table.SetHeaderAlignment(tablewriter.ALIGN_LEFT)
    # 设置内容对齐方式为左对齐
    table.SetAlignment(tablewriter.ALIGN_LEFT)

    # 设置表头线为关闭状态
    table.SetHeaderLine(false)
    # 设置边框为关闭状态
    table.SetBorder(false)
    # 设置自动格式化表头为开启状态
    table.SetAutoFormatHeaders(true)
    # 设置中心分隔符为空字符串
    table.SetCenterSeparator("")
    # 设置列分隔符为空字符串
    table.SetColumnSeparator("")
    # 设置行分隔符为空字符串
    table.SetRowSeparator("")
    # 设置表格填充为两个空格
    table.SetTablePadding("  ")
    # 设置无空白为开启状态
    table.SetNoWhiteSpace(true)

    # 如果需要使用颜色
    if pres.withColor {
        # 遍历行数据，获取严重程度颜色并设置到表格中
        for _, row := range rows {
            severityColor := getSeverityColor(row[len(row)-1])
            table.Rich(row, []tablewriter.Colors{{}, {}, {}, {}, {}, severityColor})
        }
    } else {
        # 将所有行数据一次性追加到表格中
        table.AppendBulk(rows)
    }

    # 渲染表格
    table.Render()

    # 返回空值
    return nil
// 检查终端是否支持颜色输出，返回布尔值
func supportsColor() bool {
    return lipgloss.NewStyle().Foreground(lipgloss.Color("5")).Render("") != ""
}

// 对二维字符串数组进行排序
func sortRows(rows [][]string) [][]string {
    // 使用稳定排序算法对行进行排序
    sort.SliceStable(rows, func(i, j int) bool {
        var (
            name        = 0
            ver         = 1
            packageType = 3
            vuln        = 4
            sev         = 5
        )
        // 根据不同字段进行排序，数字字段使用 >，字母字段使用 <
        if rows[i][name] == rows[j][name] {
            if rows[i][ver] == rows[j][ver] {
                if rows[i][packageType] == rows[j][packageType] {
                    if models.SeverityScore(rows[i][sev]) == models.SeverityScore(rows[j][sev]) {
                        // 使用 > 来获取最近提交的漏洞，显示在严重性顶部
                        return rows[i][vuln] > rows[j][vuln]
                    }
                    return models.SeverityScore(rows[i][sev]) > models.SeverityScore(rows[j][sev])
                }
                return rows[i][packageType] < rows[j][packageType]
            }
            return rows[i][ver] < rows[j][ver]
        }
        return rows[i][name] < rows[j][name]
    })

    return rows
}

// 移除二维字符串数组中的重复行
func removeDuplicateRows(items [][]string) [][]string {
    seen := map[string][]string{}
    var result [][]string

    for _, v := range items {
        key := strings.Join(v, "|")
        if seen[key] != nil {
            // 重复行，跳过
            continue
        }

        seen[key] = v
        result = append(result, v)
    }
    return result
}

// 创建一行数据
func createRow(m match.Match, metadataProvider vulnerability.MetadataProvider, severitySuffix string) ([]string, error) {
    var severity string

    // 获取漏洞的元数据
    metadata, err := metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
    # 如果存在错误，则返回空和错误信息
    if err != nil:
        return nil, fmt.Errorf("unable to fetch vuln=%q metadata: %+v", m.Vulnerability.ID, err)

    # 如果存在元数据，则将严重性设置为元数据的严重性加上后缀
    if metadata != nil:
        severity = metadata.Severity + severitySuffix

    # 将修复版本列表连接成字符串
    fixVersion := strings.Join(m.Vulnerability.Fix.Versions, ", ")

    # 根据漏洞修复状态进行处理
    switch m.Vulnerability.Fix.State:
        case grypeDb.WontFixState:
            fixVersion = "(won't fix)"
        case grypeDb.UnknownFixState:
            fixVersion = ""

    # 返回包含包名、版本、修复版本、包类型、漏洞ID和严重性的字符串列表
    return []string{m.Package.Name, m.Package.Version, fixVersion, string(m.Package.Type), m.Vulnerability.ID, severity}, nil
# 根据严重程度返回对应的颜色
func getSeverityColor(severity string) tablewriter.Colors {
    # 初始化字体类型和颜色
    severityFontType, severityColor := tablewriter.Normal, tablewriter.Normal

    # 根据严重程度设置字体类型和颜色
    switch strings.ToLower(severity) {
    case "critical":
        severityFontType = tablewriter.Bold
        severityColor = tablewriter.FgRedColor
    case "high":
        severityColor = tablewriter.FgRedColor
    case "medium":
        severityColor = tablewriter.FgYellowColor
    case "low":
        severityColor = tablewriter.FgGreenColor
    case "negligible":
        severityColor = tablewriter.FgBlueColor
    }

    # 返回设置好的字体类型和颜色
    return tablewriter.Colors{severityFontType, severityColor}
}
```