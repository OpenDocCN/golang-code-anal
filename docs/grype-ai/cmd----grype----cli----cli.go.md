# `grype\cmd\grype\cli\cli.go`

```
package cli

import (
    "os" // 导入操作系统相关的包
    "runtime/debug" // 导入运行时调试相关的包

    "github.com/spf13/cobra" // 导入命令行解析包

    "github.com/anchore/clio" // 导入 clio 包
    "github.com/anchore/grype/cmd/grype/cli/commands" // 导入 grype 命令行命令包
    grypeHandler "github.com/anchore/grype/cmd/grype/cli/ui" // 导入 grype 命令行 UI 包
    "github.com/anchore/grype/cmd/grype/internal/ui" // 导入 grype 内部 UI 包
    "github.com/anchore/grype/grype/vulnerability" // 导入 grype 漏洞包
    "github.com/anchore/grype/internal/bus" // 导入 grype 内部消息总线包
    "github.com/anchore/grype/internal/log" // 导入 grype 内部日志包
    "github.com/anchore/grype/internal/redact" // 导入 grype 内部数据脱敏包
    "github.com/anchore/stereoscope" // 导入 stereoscope 包
    syftHandler "github.com/anchore/syft/cmd/syft/cli/ui" // 导入 syft 命令行 UI 包
    "github.com/anchore/syft/syft" // 导入 syft 包
)

func Application(id clio.Identification) clio.Application {
    app, _ := create(id) // 调用 create 函数创建应用
    return app // 返回应用
}

func Command(id clio.Identification) *cobra.Command {
    _, cmd := create(id) // 调用 create 函数创建命令
    return cmd // 返回命令
}

func create(id clio.Identification) (clio.Application, *cobra.Command) {
    // 创建一个新的 CLI 配置对象，并设置应用程序的 ID
    clioCfg := clio.NewSetupConfig(id).
        // 添加持久的 -c <path> 标志，用于从中读取应用程序配置
        WithGlobalConfigFlag().
        // 添加持久的 -v 和 -q 标志，与日志配置相关联
        WithGlobalLoggingFlags().
        // 在根命令上使用 --help，将完整的应用程序配置呈现在帮助文本中
        WithConfigInRootHelp().
        // 设置 UI 构造函数
        WithUIConstructor(
            // 根据日志配置和标准输入的状态（如果标准输入是 tty），选择一个 UI
            func(cfg clio.Config) ([]clio.UI, error) {
                noUI := ui.None(cfg.Log.Quiet)
                if !cfg.Log.AllowUI(os.Stdin) || cfg.Log.Quiet {
                    return []clio.UI{noUI}, nil
                }

                return []clio.UI{
                    // 创建新的 UI 对象
                    ui.New(cfg.Log.Quiet,
                        // 创建 grypeHandler 对象
                        grypeHandler.New(grypeHandler.DefaultHandlerConfig()),
                        // 创建 syftHandler 对象
                        syftHandler.New(syftHandler.DefaultHandlerConfig()),
                    ),
                    noUI,
                }, nil
            },
        ).
        // 设置初始化函数
        WithInitializers(
            func(state *clio.State) error {
                // clio 正在设置并提供总线、redact 存储和日志记录器给应用程序。一旦加载完成，我们可以将它们提升到内部包中进行全局使用。
                stereoscope.SetBus(state.Bus)
                syft.SetBus(state.Bus)
                bus.Set(state.Bus)

                redact.Set(state.RedactStore)

                log.Set(state.Logger)
                syft.SetLogger(state.Logger)
                stereoscope.SetLogger(state.Logger)

                return nil
            },
        ).
        // 设置后运行函数
        WithPostRuns(func(state *clio.State, err error) {
            // 清理 stereoscope
            stereoscope.Cleanup()
        })

    // 创建一个新的 CLI 应用程序对象
    app := clio.New(*clioCfg)

    // 创建根命令
    rootCmd := commands.Root(app)

    // 添加子命令
    # 将数据库命令、完成命令、解释命令和版本命令添加到根命令中
    rootCmd.AddCommand(
        commands.DB(app),  # 添加数据库命令
        commands.Completion(),  # 添加完成命令
        commands.Explain(app),  # 添加解释命令
        clio.VersionCommand(id, syftVersion, dbVersion),  # 添加版本命令
    )
    # 返回应用程序和根命令
    return app, rootCmd
# 返回 Syft 版本信息
func syftVersion() (string, any) {
    # 读取构建信息
    buildInfo, ok := debug.ReadBuildInfo()
    # 如果读取失败，则输出错误信息并返回空字符串
    if !ok {
        log.Debug("unable to find the buildinfo section of the binary (syft version is unknown)")
        return "", ""
    }

    # 遍历构建信息中的依赖项，查找 Syft 版本信息
    for _, d := range buildInfo.Deps {
        if d.Path == "github.com/anchore/syft" {
            return "Syft Version", d.Version
        }
    }

    # 如果未找到 Syft 版本信息，则输出错误信息并返回空字符串
    log.Debug("unable to find 'github.com/anchore/syft' from the buildinfo section of the binary")
    return "", ""
}

# 返回数据库模式版本信息
func dbVersion() (string, any) {
    return "Supported DB Schema", vulnerability.SchemaVersion
}
```