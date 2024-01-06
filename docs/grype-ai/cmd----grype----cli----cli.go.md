# `grype\cmd\grype\cli\cli.go`

```
package cli  // 声明当前文件所属的包为 cli

import (
	"os"  // 导入 os 包，提供对操作系统功能的访问
	"runtime/debug"  // 导入 runtime/debug 包，提供对运行时调试信息的访问

	"github.com/spf13/cobra"  // 导入第三方包 github.com/spf13/cobra，用于创建命令行应用程序

	"github.com/anchore/clio"  // 导入 anchore/clio 包
	"github.com/anchore/grype/cmd/grype/cli/commands"  // 导入 anchore/grype/cmd/grype/cli/commands 包
	grypeHandler "github.com/anchore/grype/cmd/grype/cli/ui"  // 导入 anchore/grype/cmd/grype/cli/ui 包，并将其命名为 grypeHandler
	"github.com/anchore/grype/cmd/grype/internal/ui"  // 导入 anchore/grype/cmd/grype/internal/ui 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
	"github.com/anchore/grype/internal/bus"  // 导入 anchore/grype/internal/bus 包
	"github.com/anchore/grype/internal/log"  // 导入 anchore/grype/internal/log 包
	"github.com/anchore/grype/internal/redact"  // 导入 anchore/grype/internal/redact 包
	"github.com/anchore/stereoscope"  // 导入 anchore/stereoscope 包
	syftHandler "github.com/anchore/syft/cmd/syft/cli/ui"  // 导入 anchore/syft/cmd/syft/cli/ui 包，并将其命名为 syftHandler
	"github.com/anchore/syft/syft"  // 导入 anchore/syft/syft 包
)
# 创建一个应用程序对象，根据给定的标识
func Application(id clio.Identification) clio.Application {
    # 调用 create 函数创建应用程序对象
    app, _ := create(id)
    # 返回应用程序对象
    return app
}

# 创建一个命令对象，根据给定的标识
func Command(id clio.Identification) *cobra.Command {
    # 调用 create 函数创建应用程序对象和命令对象
    _, cmd := create(id)
    # 返回命令对象
    return cmd
}

# 创建应用程序对象和命令对象，根据给定的标识
func create(id clio.Identification) (clio.Application, *cobra.Command) {
    # 创建一个新的设置配置对象，根据给定的标识
    clioCfg := clio.NewSetupConfig(id).
        # 添加持久的 -c <path> 选项，用于从配置文件中读取应用程序配置
        WithGlobalConfigFlag().
        # 添加持久的 -v 和 -q 选项，与日志配置相关联
        WithGlobalLoggingFlags().
        # 在根命令的 --help 选项中呈现完整的应用程序配置
        WithConfigInRootHelp().
        # 根据日志配置和标准输入的状态（如果标准输入是 tty）选择一个 UI
        WithUIConstructor(
            func(cfg clio.Config) ([]clio.UI, error) {
                # 根据日志配置的静默状态选择一个 UI
                noUI := ui.None(cfg.Log.Quiet)
# 如果日志不允许使用标准输入或者日志是静默模式，则返回一个不包含用户界面的数组和空错误
if !cfg.Log.AllowUI(os.Stdin) || cfg.Log.Quiet {
    return []clio.UI{noUI}, nil
}

# 返回一个包含用户界面的数组和一个空错误
return []clio.UI{
    # 创建一个新的用户界面，根据日志的静默模式和默认处理程序配置创建 grypeHandler 和 syftHandler
    ui.New(cfg.Log.Quiet,
        grypeHandler.New(grypeHandler.DefaultHandlerConfig()),
        syftHandler.New(syftHandler.DefaultHandlerConfig()),
    ),
    noUI, # 返回一个不包含用户界面的对象
}, nil
},
).
WithInitializers(
    func(state *clio.State) error {
        # clio 正在设置并提供总线、红色存储和日志记录器给应用程序。一旦加载完成，我们可以将它们提升到内部包中以供全局使用。
        stereoscope.SetBus(state.Bus) # 将总线设置到 stereoscope 包中
        syft.SetBus(state.Bus) # 将总线设置到 syft 包中
        bus.Set(state.Bus) # 将总线设置到 bus 包中
# 设置redact库的状态
redact.Set(state.RedactStore)

# 设置日志库的状态
log.Set(state.Logger)
# 设置syft库的日志
syft.SetLogger(state.Logger)
# 设置stereoscope库的日志
stereoscope.SetLogger(state.Logger)

# 返回空值
return nil
},
).
# 在运行后清理stereoscope库
WithPostRuns(func(state *clio.State, err error) {
    stereoscope.Cleanup()
})

# 创建一个新的命令行应用程序
app := clio.New(*clioCfg)

# 创建根命令
rootCmd := commands.Root(app)

# 添加子命令
rootCmd.AddCommand(
# 创建一个应用程序和根命令
func NewApp() (*cobra.Command, *cobra.Command) {
	# 创建一个应用程序
	app := &cobra.Command{
		Use:   "syft",
		Short: "Syft is a CLI tool for generating a Software Bill of Materials (SBOM) from container images and filesystems",
		Long:  "Syft is a CLI tool for generating a Software Bill of Materials (SBOM) from container images and filesystems. It is designed to be as simple and straightforward as possible.",
		Run: func(cmd *cobra.Command, args []string) {
			cmd.Help()
		},
	}

	# 添加数据库命令
	app.AddCommand(commands.DB(app))
	# 添加完成命令
	app.AddCommand(commands.Completion())
	# 添加解释命令
	app.AddCommand(commands.Explain(app))
	# 添加版本命令
	app.AddCommand(clio.VersionCommand(id, syftVersion, dbVersion))

	# 返回应用程序和根命令
	return app, rootCmd
}

# 获取 Syft 版本信息
func syftVersion() (string, any) {
	# 读取构建信息
	buildInfo, ok := debug.ReadBuildInfo()
	if !ok {
		# 如果无法找到构建信息，则打印错误信息并返回空字符串
		log.Debug("unable to find the buildinfo section of the binary (syft version is unknown)")
		return "", ""
	}

	# 遍历构建信息的依赖项
	for _, d := range buildInfo.Deps {
		# 如果依赖项的路径为 "github.com/anchore/syft"，则返回 Syft 版本信息
		if d.Path == "github.com/anchore/syft" {
			return "Syft Version", d.Version
		}
	}
	# 如果未找到对应的依赖项，则返回空字符串
}
# 结束当前函数或代码块
}

# 记录调试信息，表示无法在二进制文件的构建信息部分找到'github.com/anchore/syft'
log.Debug("unable to find 'github.com/anchore/syft' from the buildinfo section of the binary")
# 返回空字符串
return "", ""
}

# 定义一个名为dbVersion的函数，返回一个字符串和一个未定义的变量
func dbVersion() (string, any) {
# 返回一个描述支持的数据库模式的字符串和一个名为vulnerability的未定义变量的模式版本
return "Supported DB Schema", vulnerability.SchemaVersion
}
```