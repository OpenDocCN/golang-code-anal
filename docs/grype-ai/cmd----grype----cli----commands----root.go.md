# `grype\cmd\grype\cli\commands\root.go`

```
package commands

import (
    "errors" // 导入 errors 包
    "fmt" // 导入 fmt 包
    "strings" // 导入 strings 包

    "github.com/spf13/cobra" // 导入 cobra 包
    "github.com/wagoodman/go-partybus" // 导入 go-partybus 包

    "github.com/anchore/clio" // 导入 clio 包
    "github.com/anchore/grype/cmd/grype/cli/options" // 导入 options 包
    "github.com/anchore/grype/grype" // 导入 grype 包
    "github.com/anchore/grype/grype/db" // 导入 db 包
    grypeDb "github.com/anchore/grype/grype/db/v5" // 导入 grype/db/v5 包
    "github.com/anchore/grype/grype/event" // 导入 event 包
    "github.com/anchore/grype/grype/event/parsers" // 导入 parsers 包
    "github.com/anchore/grype/grype/grypeerr" // 导入 grypeerr 包
    "github.com/anchore/grype/grype/match" // 导入 match 包
    "github.com/anchore/grype/grype/matcher" // 导入 matcher 包
    "github.com/anchore/grype/grype/matcher/dotnet" // 导入 dotnet 包
    "github.com/anchore/grype/grype/matcher/golang" // 导入 golang 包
    "github.com/anchore/grype/grype/matcher/java" // 导入 java 包
    "github.com/anchore/grype/grype/matcher/javascript" // 导入 javascript 包
    "github.com/anchore/grype/grype/matcher/python" // 导入 python 包
    "github.com/anchore/grype/grype/matcher/ruby" // 导入 ruby 包
    "github.com/anchore/grype/grype/matcher/stock" // 导入 stock 包
    "github.com/anchore/grype/grype/pkg" // 导入 pkg 包
    "github.com/anchore/grype/grype/presenter/models" // 导入 models 包
    "github.com/anchore/grype/grype/store" // 导入 store 包
    "github.com/anchore/grype/grype/vex" // 导入 vex 包
    "github.com/anchore/grype/internal" // 导入 internal 包
    "github.com/anchore/grype/internal/bus" // 导入 bus 包
    "github.com/anchore/grype/internal/format" // 导入 format 包
    "github.com/anchore/grype/internal/log" // 导入 log 包
    "github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
    "github.com/anchore/syft/syft/linux" // 导入 linux 包
    syftPkg "github.com/anchore/syft/syft/pkg" // 导入 syft/pkg 包
    "github.com/anchore/syft/syft/pkg/cataloger" // 导入 cataloger 包
    "github.com/anchore/syft/syft/sbom" // 导入 sbom 包
)

// 定义 Root 函数
func Root(app clio.Application) *cobra.Command {
    // 获取默认的 Grype 选项
    opts := options.DefaultGrype(app.ID())

    // 设置根命令
    return app.SetupRootCommand(&cobra.Command{
        Use:   fmt.Sprintf("%s [IMAGE]", app.ID().Name), // 设置命令使用说明
        Short: "A vulnerability scanner for container images, filesystems, and SBOMs", // 设置命令简短说明
        Long: stringutil.Tprintf(`A vulnerability scanner for container images, filesystems, and SBOMs. // 设置命令详细说明
Supports the following image sources:
``` // 设置命令详细说明
    # 使用默认的 Docker 守护程序中的镜像
    {{.appName}} yourrepo/yourimage:tag
    
    # 使用指定路径中的 Docker tar、OCI tar、OCI 目录、SIF 容器或通用文件系统目录
    {{.appName}} path/to/yourproject
// 显示了一系列的命令示例，说明了如何显式指定要使用的方案
// 使用 Podman 守护进程
{{.appName}} podman:yourrepo/yourimage:tag
// 显式使用 Docker 守护进程
{{.appName}} docker:yourrepo/yourimage:tag
// 使用来自磁盘的 tarball 作为存档
{{.appName}} docker-archive:path/to/yourimage.tar
// 使用来自磁盘的 tarball 作为 OCI 存档（来自 Podman 或其他方式）
{{.appName}} oci-archive:path/to/yourimage.tar
// 直接从磁盘上的路径读取 OCI 布局目录（来自 Skopeo 或其他方式）
{{.appName}} oci-dir:path/to/yourimage
// 直接从磁盘上的 Singularity Image Format（SIF）容器读取
{{.appName}} singularity:path/to/yourimage.sif
// 直接从磁盘上的路径读取（任何目录）
{{.appName}} dir:path/to/yourproject
// 从磁盘上的路径读取 Syft JSON
{{.appName}} sbom:path/to/syft.json
// 直接从注册表中拉取镜像（无需容器运行时）
{{.appName}} registry:yourrepo/yourimage:tag
// 从磁盘上的路径读取 purl 文件
{{.appName}} purl:path/to/purl/file

// 也可以直接输入 Syft JSON
syft yourimage:tag -o json | {{.appName}}

// 创建了一个命令，包括了一些选项和参数
&cobra.Command{
    Use:           "grype [image]",
    Short:         "Scan an image for vulnerabilities",
    Long:          fmt.Sprintf(`Scan an image for vulnerabilities. If no image is provided, the default image will be used: %s`, defaultImage),
    Example:       fmt.Sprintf(`%s grype yourimage:tag`, app.ID().Name),
    Args:          validateRootArgs,
    SilenceUsage:  true,
    SilenceErrors: true,
    RunE: func(cmd *cobra.Command, args []string) error {
        userInput := ""
        if len(args) > 0 {
            userInput = args[0]
        }
        return runGrype(app, opts, userInput)
    },
    ValidArgsFunction: dockerImageValidArgsFunction,
}, opts)

// 定义了一个忽略非固定匹配的规则列表
var ignoreNonFixedMatches = []match.IgnoreRule{
    {FixState: string(grypeDb.NotFixedState)},
    {FixState: string(grypeDb.WontFixState)},
    {FixState: string(grypeDb.UnknownFixState)},
}

// 定义了一个忽略固定匹配的规则列表
var ignoreFixedMatches = []match.IgnoreRule{
    # 将FixState映射为grypeDb.FixedState的字符串形式，并作为字典的值
// 定义一个忽略规则的数组，包含VEX状态为NotAffected和Fixed的规则
var ignoreVEXFixedNotAffected = []match.IgnoreRule{
    {VexStatus: string(vex.StatusNotAffected)},
    {VexStatus: string(vex.StatusFixed)},
}

// 定义一个函数，用于运行Grype扫描
//nolint:funlen
func runGrype(app clio.Application, opts *options.Grype, userInput string) (errs error) {
    // 创建扫描结果写入器
    writer, err := format.MakeScanResultWriter(opts.Outputs, opts.File, format.PresentationConfig{
        TemplateFilePath: opts.OutputTemplateFile,
        ShowSuppressed:   opts.ShowSuppressed,
    })
    if err != nil {
        return err
    }

    // 初始化变量
    var str *store.Store
    var status *db.Status
    var dbCloser *db.Closer
    var packages []pkg.Package
    var s *sbom.SBOM
    var pkgContext pkg.Context

    // 如果只检查已修复的漏洞，则将忽略规则数组中的非修复规则添加到opts.Ignore中
    if opts.OnlyFixed {
        opts.Ignore = append(opts.Ignore, ignoreNonFixedMatches...)
    }

    // 如果只检查未修复的漏洞，则将忽略规则数组中的修复规则添加到opts.Ignore中
    if opts.OnlyNotFixed {
        opts.Ignore = append(opts.Ignore, ignoreFixedMatches...)
    }

    // 遍历用户输入的忽略状态，根据状态添加相应的忽略规则到opts.Ignore中
    for _, ignoreState := range stringutil.SplitCommaSeparatedString(opts.IgnoreStates) {
        switch grypeDb.FixState(ignoreState) {
        case grypeDb.UnknownFixState, grypeDb.FixedState, grypeDb.NotFixedState, grypeDb.WontFixState:
            opts.Ignore = append(opts.Ignore, match.IgnoreRule{FixState: ignoreState})
        default:
            return fmt.Errorf("unknown fix state %s was supplied for --ignore-states", ignoreState)
        }
    }
}
    # 并行执行多个函数，返回错误
    err = parallel(
        # 检查应用更新，并返回错误
        func() error {
            checkForAppUpdate(app.ID(), opts)
            return nil
        },
        # 加载漏洞数据库，并验证加载结果
        func() (err error) {
            log.Debug("loading DB")
            str, status, dbCloser, err = grype.LoadVulnerabilityDB(opts.DB.ToCuratorConfig(), opts.DB.AutoUpdate)
            return validateDBLoad(err, status)
        },
        # 收集软件包信息，并返回软件包、软件包上下文、状态和错误
        func() (err error) {
            log.Debugf("gathering packages")
            # 软件包是 grype.Package，而不是 syft.Package
            # SBOM 用于下游格式化问题
            # grype 使用 SBOM 与 syft 格式化程序结合，生成带有漏洞信息的 cycloneDX
            packages, pkgContext, s, err = pkg.Provide(userInput, getProviderConfig(opts))
            if err != nil:
                return fmt.Errorf("failed to catalog: %w", err)
            return nil
        },
    )

    # 如果有错误，则返回错误
    if err != nil:
        return err

    # 如果 dbCloser 不为空，则延迟关闭数据库连接
    if dbCloser != nil:
        defer dbCloser.Close()

    # 应用 vex 规则，如果有错误则返回错误
    if err = applyVexRules(opts); err != nil:
        return fmt.Errorf("applying vex rules: %w", err)

    # 应用发行版提示
    applyDistroHint(packages, &pkgContext, opts)

    # 创建漏洞匹配器
    vulnMatcher := grype.VulnerabilityMatcher{
        Store:          *str,
        IgnoreRules:    opts.Ignore,
        NormalizeByCVE: opts.ByCVE,
        FailSeverity:   opts.FailOnServerity(),
        Matchers:       getMatchers(opts),
        VexProcessor: vex.NewProcessor(vex.ProcessorOptions{
            Documents:   opts.VexDocuments,
            IgnoreRules: opts.Ignore,
        }),
    }

    # 查找软件包和软件包上下文的匹配漏洞
    remainingMatches, ignoredMatches, err := vulnMatcher.FindMatches(packages, pkgContext)
    if err != nil:
        # 如果错误不是 ErrAboveSeverityThreshold，则返回错误
        if !errors.Is(err, grypeerr.ErrAboveSeverityThreshold):
            return err
        # 将错误追加到错误列表
        errs = appendErrors(errs, err)
    }
    # 将 PresenterConfig 结构体的实例写入到 writer 中，并检查是否有错误发生
    if err = writer.Write(models.PresenterConfig{
        ID:               app.ID(),
        Matches:          *remainingMatches,
        IgnoredMatches:   ignoredMatches,
        Packages:         packages,
        Context:          pkgContext,
        MetadataProvider: str,
        SBOM:             s,
        AppConfig:        opts,
        DBStatus:         status,
    }); err != nil:
        # 如果有错误发生，则将错误添加到 errs 列表中
        errs = appendErrors(errs, err)
    # 返回错误列表
    return errs
# 应用发行版提示到上下文中的函数
func applyDistroHint(pkgs []pkg.Package, context *pkg.Context, opts *options.Grype) {
    # 如果选项中指定了发行版，则记录日志并解析发行版和版本
    if opts.Distro != "" {
        log.Infof("using distro: %s", opts.Distro)
        split := strings.Split(opts.Distro, ":")
        d := split[0]
        v := ""
        if len(split) > 1 {
            v = split[1]
        }
        # 将发行版和版本信息存储到上下文的发行版对象中
        context.Distro = &linux.Release{
            PrettyName: d,
            Name:       d,
            ID:         d,
            IDLike: []string{
                d,
            },
            Version:   v,
            VersionID: v,
        }
    }

    # 检查是否存在操作系统包
    hasOSPackage := false
    for _, p := range pkgs {
        # 根据包的类型判断是否为操作系统包
        switch p.Type {
        case syftPkg.AlpmPkg, syftPkg.DebPkg, syftPkg.RpmPkg, syftPkg.KbPkg:
            hasOSPackage = true
        }
    }

    # 如果无法确定操作系统发行版且存在操作系统包，则记录警告日志
    if context.Distro == nil && hasOSPackage {
        log.Warnf("Unable to determine the OS distribution. This may result in missing vulnerabilities. " +
            "You may specify a distro using: --distro <distro>:<version>")
    }
}

# 检查应用程序更新的函数
func checkForAppUpdate(id clio.Identification, opts *options.Grype) {
    # 如果未设置检查应用程序更新的选项，则直接返回
    if !opts.CheckForAppUpdate {
        return
    }

    # 获取应用程序的版本并检查是否有新版本可用
    version := id.Version
    isAvailable, newVersion, err := isUpdateAvailable(version)
    if err != nil {
        log.Errorf(err.Error())
    }
    # 如果有新版本可用，则记录日志并发布事件
    if isAvailable {
        log.Infof("new version of %s is available: %s (currently running: %s)", id.Name, newVersion, version)
        bus.Publish(partybus.Event{
            Type: event.CLIAppUpdateAvailable,
            Value: parsers.UpdateCheck{
                New:     newVersion,
                Current: id.Version,
            },
        })
    } else {
        # 如果没有新版本可用，则记录调试日志
        log.Debugf("no new %s update available", id.Name)
    }
}

# 获取匹配器的函数
func getMatchers(opts *options.Grype) []matcher.Matcher {
    # 返回一个包含默认匹配器的匹配器集合
    return matcher.NewDefaultMatchers(
        # 配置各种语言的匹配器
        matcher.Config{
            # 配置 Java 匹配器
            Java: java.MatcherConfig{
                # 配置外部搜索和使用 CPEs
                ExternalSearchConfig: opts.ExternalSources.ToJavaMatcherConfig(),
                UseCPEs:              opts.Match.Java.UseCPEs,
            },
            # 配置 Ruby 匹配器
            Ruby:       ruby.MatcherConfig(opts.Match.Ruby),
            # 配置 Python 匹配器
            Python:     python.MatcherConfig(opts.Match.Python),
            # 配置 Dotnet 匹配器
            Dotnet:     dotnet.MatcherConfig(opts.Match.Dotnet),
            # 配置 Javascript 匹配器
            Javascript: javascript.MatcherConfig(opts.Match.Javascript),
            # 配置 Golang 匹配器
            Golang: golang.MatcherConfig{
                UseCPEs:               opts.Match.Golang.UseCPEs,
                AlwaysUseCPEForStdlib: opts.Match.Golang.AlwaysUseCPEForStdlib,
            },
            # 配置 Stock 匹配器
            Stock: stock.MatcherConfig(opts.Match.Stock),
        },
    )
// 根据传入的选项获取供应商配置信息
func getProviderConfig(opts *options.Grype) pkg.ProviderConfig {
    // 使用默认配置创建目录管理器配置
    cfg := cataloger.DefaultConfig()
    // 将搜索选项转换为配置并赋值给目录管理器配置
    cfg.Search = opts.Search.ToConfig()

    // 返回供应商配置对象
    return pkg.ProviderConfig{
        // 赋值 Syft 供应商配置
        SyftProviderConfig: pkg.SyftProviderConfig{
            // 将注册表选项转换为配置
            RegistryOptions:        opts.Registry.ToOptions(),
            // 赋值排除项
            Exclusions:             opts.Exclusions,
            // 赋值目录管理器配置
            CatalogingOptions:      cfg,
            // 赋值平台
            Platform:               opts.Platform,
            // 赋值名称
            Name:                   opts.Name,
            // 赋值默认镜像拉取源
            DefaultImagePullSource: opts.DefaultImagePullSource,
        },
        // 赋值合成配置
        SynthesisConfig: pkg.SynthesisConfig{
            // 赋值生成缺失 CPEs
            GenerateMissingCPEs: opts.GenerateMissingCPEs,
        },
    }
}

// 验证数据库加载
func validateDBLoad(loadErr error, status *db.Status) error {
    // 如果加载错误不为空，返回加载错误
    if loadErr != nil {
        return fmt.Errorf("failed to load vulnerability db: %w", loadErr)
    }
    // 如果状态为空，返回无法确定漏洞数据库状态的错误
    if status == nil {
        return fmt.Errorf("unable to determine the status of the vulnerability db")
    }
    // 如果状态错误不为空，返回数据库无法加载的错误
    if status.Err != nil {
        return fmt.Errorf("db could not be loaded: %w", status.Err)
    }
    // 返回空
    return nil
}

// 验证根参数
func validateRootArgs(cmd *cobra.Command, args []string) error {
    // 判断是否为标准输入管道或重定向
    isStdinPipeOrRedirect, err := internal.IsStdinPipeOrRedirect()
    // 如果出现错误，记录警告并将 isStdinPipeOrRedirect 设置为 false
    if err != nil {
        log.Warnf("unable to determine if there is piped input: %+v", err)
        isStdinPipeOrRedirect = false
    }

    // 如果参数长度为 0 且没有标准输入管道或重定向，显示帮助文本并返回错误
    if len(args) == 0 && !isStdinPipeOrRedirect {
        if err := cmd.Help(); err != nil {
            return fmt.Errorf("unable to display help: %w", err)
        }
        return fmt.Errorf("an image/directory argument is required")
    }

    // 返回参数最大数量为 1 的验证结果
    return cobra.MaximumNArgs(1)(cmd, args)
}

// 应用 Vex 规则
func applyVexRules(opts *options.Grype) error {
    // 如果忽略项长度为 0 且 Vex 文档长度大于 0，将固定未受影响的 Vex 规则追加到忽略项中
    if len(opts.Ignore) == 0 && len(opts.VexDocuments) > 0 {
        opts.Ignore = append(opts.Ignore, ignoreVEXFixedNotAffected...)
    }
}
    # 遍历 opts.VexAdd 列表中的元素和索引
    for _, vexStatus := range opts.VexAdd {
        # 根据 vexStatus 的取值进行不同的操作
        switch vexStatus {
        # 当 vexStatus 为 "Affected" 时
        case string(vex.StatusAffected):
            # 将匹配的 IgnoreRule 添加到 opts.Ignore 列表中
            opts.Ignore = append(
                opts.Ignore, match.IgnoreRule{VexStatus: string(vex.StatusAffected)},
            )
        # 当 vexStatus 为 "UnderInvestigation" 时
        case string(vex.StatusUnderInvestigation):
            # 将匹配的 IgnoreRule 添加到 opts.Ignore 列表中
            opts.Ignore = append(
                opts.Ignore, match.IgnoreRule{VexStatus: string(vex.StatusUnderInvestigation)},
            )
        # 当 vexStatus 为其它值时
        default:
            # 返回错误信息
            return fmt.Errorf("invalid VEX status in vex-add setting: %s", vexStatus)
        }
    }

    # 没有错误发生，返回 nil
    return nil
# 闭合前面的函数定义
```