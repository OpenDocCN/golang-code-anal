# `grype\cmd\grype\cli\commands\root.go`

```
package commands

import (
	"errors"  // 导入标准库中的 errors 包，用于处理错误
	"fmt"  // 导入标准库中的 fmt 包，用于格式化输出
	"strings"  // 导入标准库中的 strings 包，用于处理字符串

	"github.com/spf13/cobra"  // 导入第三方库 github.com/spf13/cobra，用于创建命令行应用
	"github.com/wagoodman/go-partybus"  // 导入第三方库 github.com/wagoodman/go-partybus，用于事件总线

	"github.com/anchore/clio"  // 导入自定义库 github.com/anchore/clio，用于命令行输入输出
	"github.com/anchore/grype/cmd/grype/cli/options"  // 导入自定义库 github.com/anchore/grype/cmd/grype/cli/options，用于命令行选项
	"github.com/anchore/grype/grype"  // 导入自定义库 github.com/anchore/grype/grype，用于漏洞扫描
	"github.com/anchore/grype/grype/db"  // 导入自定义库 github.com/anchore/grype/grype/db，用于数据库操作
	grypeDb "github.com/anchore/grype/grype/db/v5"  // 导入自定义库 github.com/anchore/grype/grype/db/v5，用于特定版本的数据库操作
	"github.com/anchore/grype/grype/event"  // 导入自定义库 github.com/anchore/grype/grype/event，用于事件处理
	"github.com/anchore/grype/grype/event/parsers"  // 导入自定义库 github.com/anchore/grype/grype/event/parsers，用于事件解析
	"github.com/anchore/grype/grype/grypeerr"  // 导入自定义库 github.com/anchore/grype/grype/grypeerr，用于自定义错误
	"github.com/anchore/grype/grype/match"  // 导入自定义库 github.com/anchore/grype/grype/match，用于匹配
	"github.com/anchore/grype/grype/matcher"  // 导入自定义库 github.com/anchore/grype/grype/matcher，用于匹配器
# 导入 dotnet 包的匹配器
"github.com/anchore/grype/grype/matcher/dotnet"
# 导入 golang 包的匹配器
"github.com/anchore/grype/grype/matcher/golang"
# 导入 java 包的匹配器
"github.com/anchore/grype/grype/matcher/java"
# 导入 javascript 包的匹配器
"github.com/anchore/grype/grype/matcher/javascript"
# 导入 python 包的匹配器
"github.com/anchore/grype/grype/matcher/python"
# 导入 ruby 包的匹配器
"github.com/anchore/grype/grype/matcher/ruby"
# 导入 stock 包的匹配器
"github.com/anchore/grype/grype/matcher/stock"
# 导入 pkg 包
"github.com/anchore/grype/grype/pkg"
# 导入 presenter/models 包
"github.com/anchore/grype/grype/presenter/models"
# 导入 store 包
"github.com/anchore/grype/grype/store"
# 导入 vex 包
"github.com/anchore/grype/grype/vex"
# 导入 internal 包
"github.com/anchore/grype/internal"
# 导入 bus 包
"github.com/anchore/grype/internal/bus"
# 导入 format 包
"github.com/anchore/grype/internal/format"
# 导入 log 包
"github.com/anchore/grype/internal/log"
# 导入 stringutil 包
"github.com/anchore/grype/internal/stringutil"
# 导入 syft/linux 包
"github.com/anchore/syft/syft/linux"
# 导入 syft/pkg 包
syftPkg "github.com/anchore/syft/syft/pkg"
# 导入 syft/sbom 包
"github.com/anchore/syft/syft/sbom"
// Root 函数用于创建根命令，接受一个应用程序对象作为参数
func Root(app clio.Application) *cobra.Command {
    // 使用默认的 Grype 选项初始化 opts 变量
    opts := options.DefaultGrype(app.ID())

    // 设置根命令的基本信息，包括使用说明和简要描述
    return app.SetupRootCommand(&cobra.Command{
        Use:   fmt.Sprintf("%s [IMAGE]", app.ID().Name),
        Short: "A vulnerability scanner for container images, filesystems, and SBOMs",
        Long: stringutil.Tprintf(`A vulnerability scanner for container images, filesystems, and SBOMs.

Supports the following image sources:
    {{.appName}} yourrepo/yourimage:tag             defaults to using images from a Docker daemon
    {{.appName}} path/to/yourproject                a Docker tar, OCI tar, OCI directory, SIF container, or generic filesystem directory

You can also explicitly specify the scheme to use:
    {{.appName}} podman:yourrepo/yourimage:tag          explicitly use the Podman daemon
    {{.appName}} docker:yourrepo/yourimage:tag          explicitly use the Docker daemon
    {{.appName}} docker-archive:path/to/yourimage.tar   use a tarball from disk for archives created from "docker save"
    {{.appName}} oci-archive:path/to/yourimage.tar      use a tarball from disk for OCI archives (from Podman or otherwise)
    {{.appName}} oci-dir:path/to/yourimage              read directly from a path on disk for OCI layout directories (from Skopeo or otherwise)
    {{.appName}} singularity:path/to/yourimage.sif      read directly from a Singularity Image Format (SIF) container on disk
```
    {{.appName}} dir:path/to/yourproject                从磁盘上的路径直接读取（任何目录）
    {{.appName}} sbom:path/to/syft.json                 从磁盘上的路径读取 Syft JSON
    {{.appName}} registry:yourrepo/yourimage:tag        直接从注册表中拉取镜像（不需要容器运行时）
    {{.appName}} purl:path/to/purl/file                 从磁盘上的路径读取一个以换行分隔的 purl 文件

您也可以直接输入 Syft JSON：
	syft yourimage:tag -o json | {{.appName}}

`, map[string]interface{}{
			"appName": app.ID().Name,
		}),
		Args:          validateRootArgs,  // 验证根参数
		SilenceUsage:  true,  // 静默使用
		SilenceErrors: true,  // 静默错误
		RunE: func(cmd *cobra.Command, args []string) error {  // 运行命令
			userInput := ""
			if len(args) > 0 {
				userInput = args[0]
			}
			return runGrype(app, opts, userInput)  // 运行 Grype
// 定义一个包含 dockerImageValidArgsFunction 函数的 ValidArgsFunction 变量
var ignoreNonFixedMatches = []match.IgnoreRule{
    // 创建一个 IgnoreRule 对象，FixState 属性为 NotFixedState
    {FixState: string(grypeDb.NotFixedState)},
    // 创建一个 IgnoreRule 对象，FixState 属性为 WontFixState
    {FixState: string(grypeDb.WontFixState)},
    // 创建一个 IgnoreRule 对象，FixState 属性为 UnknownFixState
    {FixState: string(grypeDb.UnknownFixState)},
}

// 定义一个包含 FixState 为 FixedState 的 IgnoreRule 对象的 ignoreFixedMatches 变量
var ignoreFixedMatches = []match.IgnoreRule{
    // 创建一个 IgnoreRule 对象，FixState 属性为 FixedState
    {FixState: string(grypeDb.FixedState)},
}

// 定义一个包含 VexStatus 为 NotAffected 和 Fixed 的 IgnoreRule 对象的 ignoreVEXFixedNotAffected 变量
var ignoreVEXFixedNotAffected = []match.IgnoreRule{
    // 创建一个 IgnoreRule 对象，VexStatus 属性为 StatusNotAffected
    {VexStatus: string(vex.StatusNotAffected)},
    // 创建一个 IgnoreRule 对象，VexStatus 属性为 StatusFixed
    {VexStatus: string(vex.StatusFixed)},
}
//nolint:funlen
// runGrype 函数用于运行 Grype 应用程序，接受应用程序、选项和用户输入作为参数，并返回错误信息
func runGrype(app clio.Application, opts *options.Grype, userInput string) (errs error) {
    // 创建扫描结果写入器，根据输出选项、文件和展示配置生成相应的写入器
    writer, err := format.MakeScanResultWriter(opts.Outputs, opts.File, format.PresentationConfig{
        TemplateFilePath: opts.OutputTemplateFile,
        ShowSuppressed:   opts.ShowSuppressed,
    })
    if err != nil {
        return err
    }

    var str *store.Store
    var status *db.Status
    var dbCloser *db.Closer
    var packages []pkg.Package
    var s *sbom.SBOM
    var pkgContext pkg.Context

    // 如果选项中指定仅显示已修复的漏洞，则将忽略列表中添加不需要修复的匹配项
    if opts.OnlyFixed {
        opts.Ignore = append(opts.Ignore, ignoreNonFixedMatches...)
    }
}
# 如果设置了 OnlyNotFixed 选项，则将 ignoreFixedMatches 添加到 Ignore 列表中
if opts.OnlyNotFixed {
    opts.Ignore = append(opts.Ignore, ignoreFixedMatches...)
}

# 遍历以逗号分隔的 IgnoreStates 字符串，将其转换为数组
for _, ignoreState := range stringutil.SplitCommaSeparatedString(opts.IgnoreStates) {
    # 根据 ignoreState 获取对应的 FixState
    switch grypeDb.FixState(ignoreState) {
        # 根据不同的 FixState 添加对应的 IgnoreRule 到 Ignore 列表中
        case grypeDb.UnknownFixState, grypeDb.FixedState, grypeDb.NotFixedState, grypeDb.WontFixState:
            opts.Ignore = append(opts.Ignore, match.IgnoreRule{FixState: ignoreState})
        # 如果 FixState 未知，则返回错误
        default:
            return fmt.Errorf("unknown fix state %s was supplied for --ignore-states", ignoreState)
    }
}

# 并行执行两个函数
err = parallel(
    # 检查应用程序更新
    func() error {
        checkForAppUpdate(app.ID(), opts)
        return nil
    },
    # 其他函数
    func() (err error) {
		// 输出调试信息，加载数据库
		log.Debug("loading DB")
		// 调用 grype.LoadVulnerabilityDB 方法加载漏洞数据库，返回数据库字符串、状态、数据库关闭函数和错误信息
		str, status, dbCloser, err = grype.LoadVulnerabilityDB(opts.DB.ToCuratorConfig(), opts.DB.AutoUpdate)
		// 调用 validateDBLoad 方法验证数据库加载结果，并返回错误信息
		return validateDBLoad(err, status)
	},
	func() (err error) {
		// 输出调试信息，收集软件包
		log.Debugf("gathering packages")
		// 软件包是 grype.Package，而不是 syft.Package
		// SBOM 用于下游格式化问题
		// grype 使用 SBOM 与 syft 格式化程序结合，生成带有漏洞信息的 cycloneDX
		// packages、pkgContext、s 为返回的软件包信息，err 为错误信息
		packages, pkgContext, s, err = pkg.Provide(userInput, getProviderConfig(opts))
		// 如果出现错误，返回格式化后的错误信息
		if err != nil {
			return fmt.Errorf("failed to catalog: %w", err)
		}
		// 返回空错误，表示软件包收集成功
		return nil
	},
)

// 如果出现错误，直接返回错误信息
if err != nil {
	return err
}
	}

	// 如果数据库关闭器不为空，则在函数返回前关闭数据库连接
	if dbCloser != nil {
		defer dbCloser.Close()
	}

	// 应用 Vex 规则，如果出现错误则返回错误信息
	if err = applyVexRules(opts); err != nil {
		return fmt.Errorf("applying vex rules: %w", err)
	}

	// 应用软件包的分发提示
	applyDistroHint(packages, &pkgContext, opts)

	// 创建漏洞匹配器对象
	vulnMatcher := grype.VulnerabilityMatcher{
		Store:          *str,
		IgnoreRules:    opts.Ignore,
		NormalizeByCVE: opts.ByCVE,
		FailSeverity:   opts.FailOnServerity(),
		Matchers:       getMatchers(opts),
		VexProcessor: vex.NewProcessor(vex.ProcessorOptions{
			Documents:   opts.VexDocuments,
```

		IgnoreRules: opts.Ignore,  # 设置忽略规则
	})

	# 创建漏洞匹配器
	vulnMatcher := matcher.NewVulnerabilityMatcher(matcher.Config{
		IgnoreRules: opts.Ignore,  # 设置忽略规则
	})

	# 查找匹配的漏洞
	remainingMatches, ignoredMatches, err := vulnMatcher.FindMatches(packages, pkgContext)
	if err != nil:
		# 如果错误不是严重程度超过阈值的错误，则返回错误
		if !errors.Is(err, grypeerr.ErrAboveSeverityThreshold):
			return err
		# 将错误添加到错误列表中
		errs = appendErrors(errs, err)

	# 将匹配结果写入输出
	if err = writer.Write(models.PresenterConfig{
		ID:               app.ID(),  # 设置 ID
		Matches:          *remainingMatches,  # 设置匹配结果
		IgnoredMatches:   ignoredMatches,  # 设置被忽略的匹配结果
		Packages:         packages,  # 设置包信息
		Context:          pkgContext,  # 设置上下文信息
		MetadataProvider: str,  # 设置元数据提供者
		SBOM:             s,  # 设置 SBOM
		# ...
	})  # 写入输出
		AppConfig:        opts,  // 将opts参数赋值给AppConfig字段
		DBStatus:         status,  // 将status参数赋值给DBStatus字段
	}); err != nil {  // 如果有错误发生
		errs = appendErrors(errs, err)  // 将错误信息添加到errs切片中
	}

	return errs  // 返回errs切片

}

func applyDistroHint(pkgs []pkg.Package, context *pkg.Context, opts *options.Grype) {
	if opts.Distro != "" {  // 如果opts的Distro字段不为空
		log.Infof("using distro: %s", opts.Distro)  // 打印使用的发行版信息

		split := strings.Split(opts.Distro, ":")  // 使用冒号分割opts的Distro字段
		d := split[0]  // 将分割后的第一个部分赋值给d变量
		v := ""  // 初始化v变量为空字符串
		if len(split) > 1 {  // 如果分割后的部分数量大于1
			v = split[1]  // 将第二部分赋值给v变量
		}
		context.Distro = &linux.Release{  // 将分割后的发行版信息赋值给context的Distro字段
# 设置 PrettyName、Name、ID、IDLike、Version、VersionID 的值为 d
PrettyName: d,
Name:       d,
ID:         d,
IDLike: []string{
    d,
},
Version:   v,
VersionID: v,
}

# 遍历 pkgs 切片，判断 p.Type 的值
hasOSPackage := false
for _, p := range pkgs:
    switch p.Type:
        case syftPkg.AlpmPkg, syftPkg.DebPkg, syftPkg.RpmPkg, syftPkg.KbPkg:
            # 如果 p.Type 的值为 AlpmPkg、DebPkg、RpmPkg、KbPkg 中的任意一个，将 hasOSPackage 设置为 true
            hasOSPackage = true

# 如果 context.Distro 为 nil 并且 hasOSPackage 为 true
if context.Distro == nil && hasOSPackage {
# 输出警告日志，提示无法确定操作系统发行版，可能导致漏报漏洞，建议使用 --distro <distro>:<version> 指定发行版
log.Warnf("Unable to determine the OS distribution. This may result in missing vulnerabilities. " +
	"You may specify a distro using: --distro <distro>:<version>")

# 检查是否需要应用程序更新
func checkForAppUpdate(id clio.Identification, opts *options.Grype) {
	# 如果不需要检查应用程序更新，则直接返回
	if !opts.CheckForAppUpdate {
		return
	}

	# 获取应用程序版本号
	version := id.Version
	# 检查是否有新版本可用
	isAvailable, newVersion, err := isUpdateAvailable(version)
	# 如果出现错误，输出错误日志
	if err != nil {
		log.Errorf(err.Error())
	}
	# 如果有新版本可用，输出提示日志
	if isAvailable {
		log.Infof("new version of %s is available: %s (currently running: %s)", id.Name, newVersion, version)

		# 发布应用程序更新事件
		bus.Publish(partybus.Event{
			Type: event.CLIAppUpdateAvailable,
// 创建一个UpdateCheck结构体的实例，包含新版本和当前版本信息
Value: parsers.UpdateCheck{
    New:     newVersion, // 新版本信息
    Current: id.Version, // 当前版本信息
},

// 如果没有新的更新可用，则记录日志
} else {
    log.Debugf("no new %s update available", id.Name) // 记录日志，指示没有新的更新可用
}

// 获取匹配器列表
func getMatchers(opts *options.Grype) []matcher.Matcher {
    // 返回默认的匹配器列表
    return matcher.NewDefaultMatchers(
        // 配置各种语言的匹配器
        matcher.Config{
            Java: java.MatcherConfig{
                ExternalSearchConfig: opts.ExternalSources.ToJavaMatcherConfig(), // 配置Java匹配器的外部搜索配置
                UseCPEs:              opts.Match.Java.UseCPEs, // 配置Java匹配器是否使用CPEs
            },
            Ruby:       ruby.MatcherConfig(opts.Match.Ruby), // 配置Ruby匹配器
            Python:     python.MatcherConfig(opts.Match.Python), // 配置Python匹配器
            Dotnet:     dotnet.MatcherConfig(opts.Match.Dotnet), // 配置Dotnet匹配器
// 创建 JavaScript 匹配器配置
Javascript: javascript.MatcherConfig(opts.Match.Javascript),
// 创建 Golang 匹配器配置
Golang: golang.MatcherConfig{
    UseCPEs:               opts.Match.Golang.UseCPEs,
    AlwaysUseCPEForStdlib: opts.Match.Golang.AlwaysUseCPEForStdlib,
},
// 创建 Stock 匹配器配置
Stock: stock.MatcherConfig(opts.Match.Stock),
// 获取供应商配置
func getProviderConfig(opts *options.Grype) pkg.ProviderConfig {
    return pkg.ProviderConfig{
        // 设置 Syft 供应商配置
        SyftProviderConfig: pkg.SyftProviderConfig{
            RegistryOptions:        opts.Registry.ToOptions(),
            Exclusions:             opts.Exclusions,
            CatalogingOptions:      opts.Search.ToConfig(),
            Platform:               opts.Platform,
            Name:                   opts.Name,
            DefaultImagePullSource: opts.DefaultImagePullSource,
        },
    }
}
// 创建一个 SynthesisConfig 结构体，并设置 GenerateMissingCPEs 字段的值为 opts.GenerateMissingCPEs
SynthesisConfig: pkg.SynthesisConfig{
    GenerateMissingCPEs: opts.GenerateMissingCPEs,
},

// 验证数据库加载是否出错，如果出错则返回错误信息
func validateDBLoad(loadErr error, status *db.Status) error {
    if loadErr != nil {
        return fmt.Errorf("failed to load vulnerability db: %w", loadErr)
    }
    // 如果 status 为 nil，则返回无法确定漏洞数据库状态的错误信息
    if status == nil {
        return fmt.Errorf("unable to determine the status of the vulnerability db")
    }
    // 如果 status 中的 Err 字段不为 nil，则返回数据库加载失败的错误信息
    if status.Err != nil {
        return fmt.Errorf("db could not be loaded: %w", status.Err)
    }
    // 如果以上情况都不满足，则返回 nil
    return nil
}

// 验证命令行参数是否正确
func validateRootArgs(cmd *cobra.Command, args []string) error {
// 检查标准输入是否为管道或重定向，如果出错则记录日志并将 isStdinPipeOrRedirect 设为 false
isStdinPipeOrRedirect, err := internal.IsStdinPipeOrRedirect()
if err != nil {
    log.Warnf("unable to determine if there is piped input: %+v", err)
    isStdinPipeOrRedirect = false
}

// 如果参数长度为 0 且没有标准输入管道或重定向，则显示帮助文本并返回非 0 返回码
if len(args) == 0 && !isStdinPipeOrRedirect {
    if err := cmd.Help(); err != nil {
        return fmt.Errorf("unable to display help: %w", err)
    }
    return fmt.Errorf("an image/directory argument is required")
}

// 应用 Vex 规则
func applyVexRules(opts *options.Grype) error {
    // 如果忽略列表长度为 0 且 Vex 文档长度大于 0，则将 ignoreVEXFixedNotAffected 追加到忽略列表中
    if len(opts.Ignore) == 0 && len(opts.VexDocuments) > 0 {
        opts.Ignore = append(opts.Ignore, ignoreVEXFixedNotAffected...)
	}

	// 遍历 vexAdd 切片中的每个 vex 状态
	for _, vexStatus := range opts.VexAdd {
		// 根据 vex 状态进行不同的处理
		switch vexStatus {
		// 如果 vex 状态为 "affected"，则将其添加到忽略规则中
		case string(vex.StatusAffected):
			opts.Ignore = append(
				opts.Ignore, match.IgnoreRule{VexStatus: string(vex.StatusAffected)},
			)
		// 如果 vex 状态为 "under investigation"，则将其添加到忽略规则中
		case string(vex.StatusUnderInvestigation):
			opts.Ignore = append(
				opts.Ignore, match.IgnoreRule{VexStatus: string(vex.StatusUnderInvestigation)},
			)
		// 如果 vex 状态不是以上两种情况，则返回错误信息
		default:
			return fmt.Errorf("invalid VEX status in vex-add setting: %s", vexStatus)
		}
	}

	// 返回空值表示处理成功
	return nil
}
```