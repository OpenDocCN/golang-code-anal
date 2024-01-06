# `grype\grype\presenter\table\presenter.go`

```
package table

import (
	"fmt"  // 导入格式化输出包
	"io"  // 导入输入输出包
	"sort"  // 导入排序包
	"strings"  // 导入字符串处理包

	"github.com/charmbracelet/lipgloss"  // 导入 lipgloss 包
	"github.com/olekukonko/tablewriter"  // 导入 tablewriter 包

	grypeDb "github.com/anchore/grype/grype/db/v5"  // 导入 grypeDb 包
	"github.com/anchore/grype/grype/match"  // 导入 match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/presenter/models"  // 导入 presenter/models 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
)

const (
	appendSuppressed    = " (suppressed)"  // 定义常量 appendSuppressed 为 "(suppressed)"
// appendSuppressedVEX is a constant string used to indicate that a match is suppressed by VEX
appendSuppressedVEX = " (suppressed by VEX)"
)

// Presenter is a generic struct for holding fields needed for reporting
type Presenter struct {
	results          match.Matches // results field holds the matches found
	ignoredMatches   []match.IgnoredMatch // ignoredMatches field holds the ignored matches
	packages         []pkg.Package // packages field holds the packages
	metadataProvider vulnerability.MetadataProvider // metadataProvider field holds the metadata provider
	showSuppressed   bool // showSuppressed is a flag to indicate whether suppressed matches should be shown
	withColor        bool // withColor is a flag to indicate whether the output should be colored
}

// NewPresenter is a *Presenter constructor
func NewPresenter(pb models.PresenterConfig, showSuppressed bool) *Presenter {
	// returns a new instance of Presenter with the provided fields
	return &Presenter{
		results:          pb.Matches, // initializes results field with the matches from PresenterConfig
		ignoredMatches:   pb.IgnoredMatches, // initializes ignoredMatches field with the ignored matches from PresenterConfig
		packages:         pb.Packages, // initializes packages field with the packages from PresenterConfig
		metadataProvider: pb.MetadataProvider, // initializes metadataProvider field with the metadata provider from PresenterConfig
		showSuppressed:   showSuppressed, // 将 showSuppressed 变量添加到对象中
		withColor:        supportsColor(), // 将 supportsColor() 函数的返回值添加到对象中
	}
}

// Present creates a JSON-based reporting
func (pres *Presenter) Present(output io.Writer) error {
	rows := make([][]string, 0) // 创建一个空的二维字符串数组

	columns := []string{"Name", "Installed", "Fixed-In", "Type", "Vulnerability", "Severity"} // 创建列名数组

	// 为匹配的漏洞生成行
	for m := range pres.results.Enumerate() { // 遍历结果对象的枚举
		row, err := createRow(m, pres.metadataProvider, "") // 调用 createRow 函数生成行数据
		if err != nil { // 如果出现错误
			return err // 返回错误
		}
		rows = append(rows, row) // 将生成的行数据添加到行数组中
	}

	// 为被抑制的漏洞生成行
# 如果 showSuppressed 为 true，则遍历 ignoredMatches 切片
if pres.showSuppressed {
    for _, m := range pres.ignoredMatches {
        # 设置默认消息为 appendSuppressed
        msg := appendSuppressed
        # 如果存在应用的忽略规则
        if m.AppliedIgnoreRules != nil {
            # 遍历应用的忽略规则
            for i := range m.AppliedIgnoreRules {
                # 如果命名空间为 "vex"，则将消息设置为 appendSuppressedVEX
                if m.AppliedIgnoreRules[i].Namespace == "vex" {
                    msg = appendSuppressedVEX
                }
            }
        }
        # 创建行数据，使用匹配项、元数据提供者和消息
        row, err := createRow(m.Match, pres.metadataProvider, msg)

        # 如果创建行数据过程中出现错误，则返回错误
        if err != nil {
            return err
        }
        # 将行数据添加到 rows 切片中
        rows = append(rows, row)
    }
}

# 如果 rows 切片长度为 0
if len(rows) == 0 {
# 使用 io.WriteString 将字符串 "No vulnerabilities found\n" 写入到 output 中，返回可能的错误
_, err := io.WriteString(output, "No vulnerabilities found\n")
return err
# 如果有错误，直接返回

# 对行进行排序，并去除重复的行
rows = sortRows(removeDuplicateRows(rows))

# 创建一个新的表格写入器，并设置表头
table := tablewriter.NewWriter(output)
table.SetHeader(columns)
table.SetAutoWrapText(false)
table.SetHeaderAlignment(tablewriter.ALIGN_LEFT)
table.SetAlignment(tablewriter.ALIGN_LEFT)

# 设置表格的样式
table.SetHeaderLine(false)
table.SetBorder(false)
table.SetAutoFormatHeaders(true)
table.SetCenterSeparator("")
table.SetColumnSeparator("")
table.SetRowSeparator("")
table.SetTablePadding("  ")
table.SetNoWhiteSpace(true)
# 如果 pres.withColor 为真，则对每一行数据进行处理，根据最后一列的严重程度获取对应的颜色，并将颜色应用到表格中的相应位置
if pres.withColor {
    for _, row := range rows:
        # 获取严重程度对应的颜色
        severityColor := getSeverityColor(row[len(row)-1])
        # 在表格中应用颜色
        table.Rich(row, []tablewriter.Colors{{}, {}, {}, {}, {}, severityColor})
} else {
    # 如果 pres.withColor 为假，则直接将所有行数据批量添加到表格中
    table.AppendBulk(rows)
}

# 渲染表格
table.Render()

# 返回空值
return nil
}

# 检查终端是否支持颜色
func supportsColor() bool {
    # 创建一个新的样式，并尝试渲染颜色，如果成功渲染则表示终端支持颜色
    return lipgloss.NewStyle().Foreground(lipgloss.Color("5")).Render("") != ""
}

# 对行数据进行排序
func sortRows(rows [][]string) [][]string {
	// 对行进行稳定排序
	sort.SliceStable(rows, func(i, j int) bool {
		var (
			name        = 0  // 定义列索引
			ver         = 1  // 定义列索引
			packageType = 3  // 定义列索引
			vuln        = 4  // 定义列索引
			sev         = 5  // 定义列索引
		)
		// 按照名称、版本、类型、严重程度、漏洞的顺序进行排序
		// > 用于数字排序，如严重程度或漏洞数量
		// < 用于字母排序，如名称、版本、类型
		if rows[i][name] == rows[j][name] {
			if rows[i][ver] == rows[j][ver] {
				if rows[i][packageType] == rows[j][packageType] {
					if models.SeverityScore(rows[i][sev]) == models.SeverityScore(rows[j][sev]) {
						// 在这里使用 > 来获取最近提交的漏洞
						// 以便在严重程度最高的情况下显示在顶部
						return rows[i][vuln] > rows[j][vuln]
					}
// 根据严重程度对行进行排序，使用 SeverityScore 函数进行比较
return models.SeverityScore(rows[i][sev]) > models.SeverityScore(rows[j][sev])

// 如果严重程度相同，则按照包类型进行排序
return rows[i][packageType] < rows[j][packageType]

// 如果包类型相同，则按照版本号进行排序
return rows[i][ver] < rows[j][ver]

// 如果版本号相同，则按照名称进行排序
return rows[i][name] < rows[j][name]

// 将重复行从 items 中移除
func removeDuplicateRows(items [][]string) [][]string {
	// 使用 map 来存储已经出现过的行
	seen := map[string][]string{}
	// 存储最终结果的切片
	var result [][]string

	// 遍历 items 中的每一行
	for _, v := range items {
		// 将行转换成字符串作为 map 的 key
		key := strings.Join(v, "|")
		// 如果该行已经出现过，则为重复行
		if seen[key] != nil {
			// 重复行
// 继续执行下一次循环
continue
// 将键值对添加到seen map中
seen[key] = v
// 将v添加到result切片中
result = append(result, v)
// 返回result切片
return result
// 创建一行数据
func createRow(m match.Match, metadataProvider vulnerability.MetadataProvider, severitySuffix string) ([]string, error) {
// 定义severity变量
var severity string
// 获取漏洞的元数据
metadata, err := metadataProvider.GetMetadata(m.Vulnerability.ID, m.Vulnerability.Namespace)
// 如果获取元数据出错，返回错误信息
if err != nil {
    return nil, fmt.Errorf("unable to fetch vuln=%q metadata: %+v", m.Vulnerability.ID, err)
}
// 如果元数据不为空，将severity设置为元数据的严重程度加上severitySuffix
if metadata != nil {
    severity = metadata.Severity + severitySuffix
}
// 将修复版本列表转换为字符串
fixVersion := strings.Join(m.Vulnerability.Fix.Versions, ", ")

// 根据漏洞修复状态设置修复版本字符串
switch m.Vulnerability.Fix.State {
case grypeDb.WontFixState:
	fixVersion = "(won't fix)"
case grypeDb.UnknownFixState:
	fixVersion = ""
}

// 返回包含软件包名称、版本、修复版本、类型、漏洞ID和严重程度的字符串切片
return []string{m.Package.Name, m.Package.Version, fixVersion, string(m.Package.Type), m.Vulnerability.ID, severity}, nil
}

// 根据漏洞严重程度返回对应的颜色
func getSeverityColor(severity string) tablewriter.Colors {
	severityFontType, severityColor := tablewriter.Normal, tablewriter.Normal

// 根据漏洞严重程度设置字体类型和颜色
switch strings.ToLower(severity) {
case "critical":
	severityFontType = tablewriter.Bold
	severityColor = tablewriter.FgRedColor
case "high":
# 定义变量 severityColor，并初始化为 tablewriter.FgRedColor
severityColor = tablewriter.FgRedColor
# 根据不同的情况设置 severityColor 的值
case "medium":
    severityColor = tablewriter.FgYellowColor
case "low":
    severityColor = tablewriter.FgGreenColor
case "negligible":
    severityColor = tablewriter.FgBlueColor
# 返回包含字体类型和颜色的对象
return tablewriter.Colors{severityFontType, severityColor}
```