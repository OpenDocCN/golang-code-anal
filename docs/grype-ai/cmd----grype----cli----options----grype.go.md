# `grype\cmd\grype\cli\options\grype.go`

```
package options

import (
	"fmt"  // 导入 fmt 包，用于格式化输出

	"github.com/anchore/clio"  // 导入 anchore/clio 包
	"github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
	"github.com/anchore/grype/internal/format"  // 导入 anchore/grype/internal/format 包
	"github.com/anchore/syft/syft/source"  // 导入 anchore/syft/syft/source 包
)

type Grype struct {
	File                   string             `yaml:"file" json:"file" mapstructure:"file"`  // 定义 Grype 结构体的属性 File，并指定其在 yaml、json、mapstructure 中的名称
	Distro                 string             `yaml:"distro" json:"distro" mapstructure:"distro"`  // 定义 Grype 结构体的属性 Distro，并指定其在 yaml、json、mapstructure 中的名称
	OutputTemplateFile     string             `yaml:"output-template-file" json:"output-template-file" mapstructure:"output-template-file"`  // 定义 Grype 结构体的属性 OutputTemplateFile，并指定其在 yaml、json、mapstructure 中的名称
	OnlyFixed              bool               `yaml:"only-fixed" json:"only-fixed" mapstructure:"only-fixed"`  // 定义 Grype 结构体的属性 OnlyFixed，并指定其在 yaml、json、mapstructure 中的名称
	OnlyNotFixed           bool               `yaml:"only-notfixed" json:"only-notfixed" mapstructure:"only-notfixed"`  // 定义 Grype 结构体的属性 OnlyNotFixed，并指定其在 yaml、json、mapstructure 中的名称
	Search                 search             `yaml:"search" json:"search" mapstructure:"search"`  // 定义 Grype 结构体的属性 Search，并指定其在 yaml、json、mapstructure 中的名称
	Ignore                 []match.IgnoreRule `yaml:"ignore" json:"ignore" mapstructure:"ignore"`  // 定义 Grype 结构体的属性 Ignore，并指定其在 yaml、json、mapstructure 中的名称
```
	Exclusions             []string           `yaml:"exclude" json:"exclude" mapstructure:"exclude"` 
	# 定义一个字符串数组，用于存储需要排除的内容，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	DB                     Database           `yaml:"db" json:"db" mapstructure:"db"`
	# 定义一个Database类型的变量，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	ExternalSources        externalSources    `yaml:"external-sources" json:"externalSources" mapstructure:"external-sources"`
	# 定义一个externalSources类型的变量，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	Match                  matchConfig        `yaml:"match" json:"match" mapstructure:"match"`
	# 定义一个matchConfig类型的变量，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	FailOn                 string             `yaml:"fail-on-severity" json:"fail-on-severity" mapstructure:"fail-on-severity"`
	# 定义一个字符串变量，用于存储失败的严重程度，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	Registry               registry           `yaml:"registry" json:"registry" mapstructure:"registry"`
	# 定义一个registry类型的变量，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	ShowSuppressed         bool               `yaml:"show-suppressed" json:"show-suppressed" mapstructure:"show-suppressed"`
	# 定义一个布尔变量，用于表示是否显示被抑制的内容，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	Name                   string             `yaml:"name" json:"name" mapstructure:"name"`
	# 定义一个字符串变量，用于存储名称，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	DefaultImagePullSource string             `yaml:"default-image-pull-source" json:"default-image-pull-source" mapstructure:"default-image-pull-source"`
	# 定义一个字符串变量，用于存储默认的镜像拉取源，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	VexDocuments           []string           `yaml:"vex-documents" json:"vex-documents" mapstructure:"vex-documents"`
	# 定义一个字符串数组，用于存储vex文档，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名

	VexAdd                 []string           `yaml:"vex-add" json:"vex-add" mapstructure:"vex-add"` // GRYPE_VEX_ADD
	# 定义一个字符串数组，通过yaml、json和mapstructure标签指定对应的配置文件中的字段名，同时注释中提到了一个环境变量GRYPE_VEX_ADD

}

var _ interface {
	clio.FlagAdder
	clio.PostLoader
} = (*Grype)(nil)
# 定义一个接口类型的变量，实现了clio.FlagAdder和clio.PostLoader接口，赋值为Grype类型的空指针

func DefaultGrype(id clio.Identification) *Grype {
	# 定义一个函数，返回一个Grype类型的指针，参数为clio.Identification类型的id
	return &Grype{
		# 返回一个Grype类型的结构体，字段值为默认值
	}
// 使用默认搜索范围创建搜索对象
Search:            defaultSearch(source.SquashedScope),
// 使用默认数据库 ID 创建数据库对象
DB:                DefaultDatabase(id),
// 使用默认匹配配置创建匹配对象
Match:             defaultMatchConfig(),
// 使用默认外部数据源创建外部数据源对象
ExternalSources:   defaultExternalSources(),
// 设置是否检查应用程序更新为 true
CheckForAppUpdate: true,
// 初始化 VexAdd 为空字符串数组
VexAdd:            []string{},
}

// nolint:funlen
// 添加 Grype 对象的命令行标志
func (o *Grype) AddFlags(flags clio.FlagSet) {
    // 添加 scope 标志，并设置默认值和说明
    flags.StringVarP(&o.Search.Scope,
        "scope", "s",
        fmt.Sprintf("selection of layers to analyze, options=%v", source.AllScopes),
    )

    // 添加 output 标志，并设置默认值和说明
    flags.StringArrayVarP(&o.Outputs,
        "output", "o",
        fmt.Sprintf("report output formatter, formats=%v, deprecated formats=%v", format.AvailableFormats, format.DeprecatedFormats),
    )
}
# 设置 o.File 变量的值，用于指定默认报告输出的文件名，默认为标准输出
flags.StringVarP(&o.File,
    "file", "",
    "file to write the default report output to (default is STDOUT)",
)

# 设置 o.Name 变量的值，用于指定被分析目标的名称
flags.StringVarP(&o.Name,
    "name", "",
    "set the name of the target being analyzed",
)

# 设置 o.Distro 变量的值，用于指定要匹配的发行版，格式为：<distro>:<version>
flags.StringVarP(&o.Distro,
    "distro", "",
    "distro to match against in the format: <distro>:<version>",
)

# 设置 o.GenerateMissingCPEs 变量的值，用于指定是否为没有 CPE 数据的软件包生成 CPE
flags.BoolVarP(&o.GenerateMissingCPEs,
    "add-cpes-if-none", "",
    "generate CPEs for packages with no CPE data",
)
# 设置命令行标志参数 o.OutputTemplateFile，用于指定输出模板文件的路径，需要选择 'template' 输出选项
flags.StringVarP(&o.OutputTemplateFile,
    "template", "t",
    "specify the path to a Go template file (requires 'template' output to be selected)")

# 设置命令行标志参数 o.FailOn，用于设置当发现严重程度大于等于给定严重程度的漏洞时返回代码为1，选项包括所有严重程度的选项
flags.StringVarP(&o.FailOn,
    "fail-on", "f",
    fmt.Sprintf("set the return code to 1 if a vulnerability is found with a severity >= the given severity, options=%v", vulnerability.AllSeverities()),
)

# 设置命令行标志参数 o.OnlyFixed，用于忽略未修复的漏洞匹配
flags.BoolVarP(&o.OnlyFixed,
    "only-fixed", "",
    "ignore matches for vulnerabilities that are not fixed",
)

# 设置命令行标志参数 o.OnlyNotFixed，用于忽略已修复的漏洞匹配
flags.BoolVarP(&o.OnlyNotFixed,
    "only-notfixed", "",
    "ignore matches for vulnerabilities that are fixed",
)
# 设置命令行参数 o.IgnoreStates，用于忽略指定修复状态的漏洞匹配
flags.StringVarP(&o.IgnoreStates,
    "ignore-states", "",
    fmt.Sprintf("ignore matches for vulnerabilities with specified comma separated fix states, options=%v", vulnerability.AllFixStates()),
)

# 设置命令行参数 o.ByCVE，用于在可能的情况下按CVE而不是原始漏洞ID对结果进行排序
flags.BoolVarP(&o.ByCVE,
    "by-cve", "",
    "orient results by CVE instead of the original vulnerability ID when possible",
)

# 设置命令行参数 o.ShowSuppressed，用于在输出中显示被抑制/忽略的漏洞（仅在表格输出格式中支持）
flags.BoolVarP(&o.ShowSuppressed,
    "show-suppressed", "",
    "show suppressed/ignored vulnerabilities in the output (only supported with table output format)",
)

# 设置命令行参数 o.Exclusions，用于使用通配符表达式排除被扫描的路径
flags.StringArrayVarP(&o.Exclusions,
    "exclude", "",
    "exclude paths from being scanned using a glob expression",
)
# 设置一个命令行标志，用于指定容器镜像源的可选平台说明符（例如'linux/arm64'，'linux/arm64/v8'，'arm64'，'linux'）
flags.StringVarP(&o.Platform,
	"platform", "",
	"an optional platform specifier for container image sources (e.g. 'linux/arm64', 'linux/arm64/v8', 'arm64', 'linux')",
)

# 设置一个命令行标志，用于指定在生成扫描结果时要考虑的VEX文档列表
flags.StringArrayVarP(&o.VexDocuments,
	"vex", "",
	"a list of VEX documents to consider when producing scanning results",
)

# 在加载后执行的函数，用于检查是否设置了--fail-on标志，并验证其值是否有效
func (o *Grype) PostLoad() error {
	if o.FailOn != "" {
		# 获取--fail-on标志的严重性级别，并检查是否为未知级别
		failOnSeverity := *o.FailOnServerity()
		if failOnSeverity == vulnerability.UnknownSeverity {
			return fmt.Errorf("bad --fail-on severity value '%s'", o.FailOn)
		}
	}
	return nil
}
# 定义一个名为 FailOnServerity 的方法，属于 Grype 结构体
func (o Grype) FailOnServerity() *vulnerability.Severity {
    # 调用 ParseSeverity 方法，将 o.FailOn 参数解析为漏洞严重程度
    severity := vulnerability.ParseSeverity(o.FailOn)
    # 返回漏洞严重程度的指针
    return &severity
}
```