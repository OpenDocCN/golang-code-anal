# `grype\cmd\grype\cli\options\grype.go`

```
package options

import (
    "fmt"

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    "github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
    "github.com/anchore/grype/internal/format"  // 导入 anchore/grype/internal/format 包
    "github.com/anchore/syft/syft/source"  // 导入 anchore/syft/syft/source 包
)

type Grype struct {
    Outputs                []string           `yaml:"output" json:"output" mapstructure:"output"`  // 输出选项，用于指定报告格式和输出文件
    File                   string             `yaml:"file" json:"file" mapstructure:"file"`  // 文件选项，用于指定报告输出文件
    Distro                 string             `yaml:"distro" json:"distro" mapstructure:"distro"`  // 发行版选项，用于指定要显式使用的发行版
    GenerateMissingCPEs    bool               `yaml:"add-cpes-if-none" json:"add-cpes-if-none" mapstructure:"add-cpes-if-none"`  // 选项，用于在导入中自动生成 CPE（例如，来自第三方 SPDX 文档）
    OutputTemplateFile     string             `yaml:"output-template-file" json:"output-template-file" mapstructure:"output-template-file"`  // 模板文件选项，用于指定用于格式化最终报告的模板文件
    CheckForAppUpdate      bool               `yaml:"check-for-app-update" json:"check-for-app-update" mapstructure:"check-for-app-update"`  // 是否在启动时检查应用程序更新
    OnlyFixed              bool               `yaml:"only-fixed" json:"only-fixed" mapstructure:"only-fixed"`  // 只在检测到漏洞已修复时失败
    OnlyNotFixed           bool               `yaml:"only-notfixed" json:"only-notfixed" mapstructure:"only-notfixed"`  // 只在检测到漏洞未修复时失败
    # 定义一个字符串类型的变量 IgnoreStates，用于指定忽略的漏洞状态
    IgnoreStates           string             `yaml:"ignore-states" json:"ignore-wontfix" mapstructure:"ignore-wontfix"`                    // ignore detections for vulnerabilities matching these comma-separated fix states
    # 定义一个字符串类型的变量 Platform，用于指定目标平台的容器镜像
    Platform               string             `yaml:"platform" json:"platform" mapstructure:"platform"`                                     // --platform, override the target platform for a container image
    # 定义一个 search 结构体类型的变量 Search，用于指定搜索配置
    Search                 search             `yaml:"search" json:"search" mapstructure:"search"`
    # 定义一个 IgnoreRule 类型的切片变量 Ignore，用于指定忽略规则
    Ignore                 []match.IgnoreRule `yaml:"ignore" json:"ignore" mapstructure:"ignore"`
    # 定义一个字符串类型的切片变量 Exclusions，用于指定排除项
    Exclusions             []string           `yaml:"exclude" json:"exclude" mapstructure:"exclude"`
    # 定义一个 Database 类型的变量 DB，用于指定数据库配置
    DB                     Database           `yaml:"db" json:"db" mapstructure:"db"`
    # 定义一个 externalSources 结构体类型的变量 ExternalSources，用于指定外部数据源配置
    ExternalSources        externalSources    `yaml:"external-sources" json:"externalSources" mapstructure:"external-sources"`
    # 定义一个 matchConfig 结构体类型的变量 Match，用于指定匹配配置
    Match                  matchConfig        `yaml:"match" json:"match" mapstructure:"match"`
    # 定义一个字符串类型的变量 FailOn，用于指定严重程度失败的规则
    FailOn                 string             `yaml:"fail-on-severity" json:"fail-on-severity" mapstructure:"fail-on-severity"`
    # 定义一个 registry 结构体类型的变量 Registry，用于指定注册表配置
    Registry               registry           `yaml:"registry" json:"registry" mapstructure:"registry"`
    # 定义一个布尔类型的变量 ShowSuppressed，用于指定是否显示被抑制的漏洞
    ShowSuppressed         bool               `yaml:"show-suppressed" json:"show-suppressed" mapstructure:"show-suppressed"`
    # 定义一个布尔类型的变量 ByCVE，用于指定是否使用 CVE 代替原始匹配的漏洞 ID
    ByCVE                  bool               `yaml:"by-cve" json:"by-cve" mapstructure:"by-cve"` // --by-cve, indicates if the original match vulnerability IDs should be preserved or the CVE should be used instead
    # 定义一个字符串类型的变量 Name，用于指定名称
    Name                   string             `yaml:"name" json:"name" mapstructure:"name"`
    # 定义一个字符串类型的变量 DefaultImagePullSource，用于指定默认的镜像拉取源
    DefaultImagePullSource string             `yaml:"default-image-pull-source" json:"default-image-pull-source" mapstructure:"default-image-pull-source"`
    # 定义一个字符串类型的切片变量 VexDocuments，用于指定 Vex 文档
    VexDocuments           []string           `yaml:"vex-documents" json:"vex-documents" mapstructure:"vex-documents"`
    # 定义一个名为VexAdd的字段，类型为字符串切片，用于存储yaml文件中的vex-add字段的值
    VexAdd                 []string           `yaml:"vex-add" json:"vex-add" mapstructure:"vex-add"` // GRYPE_VEX_ADD
// 定义一个接口变量，包含FlagAdder和PostLoader两个接口
var _ interface {
    clio.FlagAdder
    clio.PostLoader
} = (*Grype)(nil)

// 创建一个默认的Grype对象，根据传入的Identification参数设置默认值
func DefaultGrype(id clio.Identification) *Grype {
    return &Grype{
        Search:            defaultSearch(source.SquashedScope), // 设置Search字段的默认值
        DB:                DefaultDatabase(id), // 设置DB字段的默认值
        Match:             defaultMatchConfig(), // 设置Match字段的默认值
        ExternalSources:   defaultExternalSources(), // 设置ExternalSources字段的默认值
        CheckForAppUpdate: true, // 设置CheckForAppUpdate字段的默认值
        VexAdd:            []string{}, // 设置VexAdd字段的默认值
    }
}

// nolint:funlen
// 为Grype对象添加FlagSet中的标志
func (o *Grype) AddFlags(flags clio.FlagSet) {
    flags.StringVarP(&o.Search.Scope, // 设置Search.Scope字段的值
        "scope", "s",
        fmt.Sprintf("selection of layers to analyze, options=%v", source.AllScopes), // 设置标志的说明信息
    )

    flags.StringArrayVarP(&o.Outputs, // 设置Outputs字段的值
        "output", "o",
        fmt.Sprintf("report output formatter, formats=%v, deprecated formats=%v", format.AvailableFormats, format.DeprecatedFormats), // 设置标志的说明信息
    )

    flags.StringVarP(&o.File, // 设置File字段的值
        "file", "",
        "file to write the default report output to (default is STDOUT)", // 设置标志的说明信息
    )

    flags.StringVarP(&o.Name, // 设置Name字段的值
        "name", "",
        "set the name of the target being analyzed", // 设置标志的说明信息
    )

    flags.StringVarP(&o.Distro, // 设置Distro字段的值
        "distro", "",
        "distro to match against in the format: <distro>:<version>", // 设置标志的说明信息
    )

    flags.BoolVarP(&o.GenerateMissingCPEs, // 设置GenerateMissingCPEs字段的值
        "add-cpes-if-none", "",
        "generate CPEs for packages with no CPE data", // 设置标志的说明信息
    )

    flags.StringVarP(&o.OutputTemplateFile, // 设置OutputTemplateFile字段的值
        "template", "t",
        "specify the path to a Go template file (requires 'template' output to be selected)", // 设置标志的说明信息
    )

    flags.StringVarP(&o.FailOn, // 设置FailOn字段的值
        "fail-on", "f",
        fmt.Sprintf("set the return code to 1 if a vulnerability is found with a severity >= the given severity, options=%v", vulnerability.AllSeverities()), // 设置标志的说明信息
    )

    flags.BoolVarP(&o.OnlyFixed, // 设置OnlyFixed字段的值
        "only-fixed", "",
        "ignore matches for vulnerabilities that are not fixed", // 设置标志的说明信息
    )
}
    # 创建一个布尔标志，用于指定是否仅忽略已修复的漏洞匹配
    flags.BoolVarP(&o.OnlyNotFixed,
        "only-notfixed", "",
        "ignore matches for vulnerabilities that are fixed",
    )

    # 创建一个字符串标志，用于指定要忽略的漏洞的修复状态
    flags.StringVarP(&o.IgnoreStates,
        "ignore-states", "",
        fmt.Sprintf("ignore matches for vulnerabilities with specified comma separated fix states, options=%v", vulnerability.AllFixStates()),
    )

    # 创建一个布尔标志，用于指定是否按CVE而不是原始漏洞ID来定位结果
    flags.BoolVarP(&o.ByCVE,
        "by-cve", "",
        "orient results by CVE instead of the original vulnerability ID when possible",
    )

    # 创建一个布尔标志，用于指定是否在输出中显示被抑制/忽略的漏洞（仅支持表格输出格式）
    flags.BoolVarP(&o.ShowSuppressed,
        "show-suppressed", "",
        "show suppressed/ignored vulnerabilities in the output (only supported with table output format)",
    )

    # 创建一个字符串数组标志，用于指定要从扫描中排除的路径
    flags.StringArrayVarP(&o.Exclusions,
        "exclude", "",
        "exclude paths from being scanned using a glob expression",
    )

    # 创建一个字符串标志，用于指定容器镜像源的可选平台说明符
    flags.StringVarP(&o.Platform,
        "platform", "",
        "an optional platform specifier for container image sources (e.g. 'linux/arm64', 'linux/arm64/v8', 'arm64', 'linux')",
    )

    # 创建一个字符串数组标志，用于指定在生成扫描结果时要考虑的VEX文档列表
    flags.StringArrayVarP(&o.VexDocuments,
        "vex", "",
        "a list of VEX documents to consider when producing scanning results",
    )
# 定义 Grype 结构体的 PostLoad 方法，用于在加载后执行操作
func (o *Grype) PostLoad() error {
    # 如果设置了 FailOn 属性
    if o.FailOn != "" {
        # 获取 FailOn 属性对应的漏洞严重程度
        failOnSeverity := *o.FailOnServerity()
        # 如果获取的漏洞严重程度为未知
        if failOnSeverity == vulnerability.UnknownSeverity {
            # 返回错误信息
            return fmt.Errorf("bad --fail-on severity value '%s'", o.FailOn)
        }
    }
    # 返回空值
    return nil
}

# 定义 Grype 结构体的 FailOnServerity 方法，用于获取 FailOn 属性对应的漏洞严重程度
func (o Grype) FailOnServerity() *vulnerability.Severity {
    # 解析 FailOn 属性对应的漏洞严重程度
    severity := vulnerability.ParseSeverity(o.FailOn)
    # 返回漏洞严重程度的指针
    return &severity
}
```