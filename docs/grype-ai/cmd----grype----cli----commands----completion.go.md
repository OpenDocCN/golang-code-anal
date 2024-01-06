# `grype\cmd\grype\cli\commands\completion.go`

```
// 导入必要的包
package commands

import (
	"context"  // 上下文包，用于控制程序的执行流程
	"os"  // 操作系统包，提供对操作系统功能的访问
	"strings"  // 字符串包，提供对字符串的操作

	"github.com/docker/docker/api/types"  // Docker API 类型包
	"github.com/docker/docker/api/types/filters"  // Docker API 过滤器包
	"github.com/docker/docker/client"  // Docker 客户端包
	"github.com/spf13/cobra"  // 命令行包
)

// Completion 返回一个命令，用于为各种终端 shell 提供自动补全
func Completion() *cobra.Command {
	return &cobra.Command{
		Use:   "completion [bash|zsh|fish]",  // 命令使用说明
		Short: "Generate a shell completion for Grype (listing local docker images)",  // 命令简短说明
		Long: `To load completions (docker image list):  // 命令详细说明
# 在 Bash 中加载 grype 的自动补全功能
$ source <(grype completion bash)

# 为了每次会话都能加载自动补全，执行一次以下命令：
Linux:
  $ grype completion bash > /etc/bash_completion.d/grype
MacOS:
  $ grype completion bash > /usr/local/etc/bash_completion.d/grype

# 在 Zsh 中：
# 如果你的环境中尚未启用 shell 自动补全，你需要启用它。你可以执行以下命令一次：

$ echo "autoload -U compinit; compinit" >> ~/.zshrc

# 为了每次会话都能加载自动补全，执行一次以下命令：
$ grype completion zsh > "${fpath[1]}/_grype"
# 为了使设置生效，需要启动一个新的 shell。

Fish:

# 从 grype 获取 Fish shell 的自动补全，并加载到当前会话中
$ grype completion fish | source

# 为了每次会话都加载自动补全，只需要执行一次以下命令：
$ grype completion fish > ~/.config/fish/completions/grype.fish
`,
		DisableFlagsInUseLine: true,  # 禁用使用行中的标志
		ValidArgs:             []string{"bash", "fish", "zsh"},  # 有效的参数列表
		Args:                  cobra.MatchAll(cobra.ExactArgs(1), cobra.OnlyValidArgs),  # 参数匹配规则
		RunE: func(cmd *cobra.Command, args []string) error {  # 定义一个运行函数
			var err error
			switch args[0] {  # 根据参数进行不同的处理
			case "zsh":
				err = cmd.Root().GenZshCompletion(os.Stdout)  # 生成 Zsh shell 的自动补全并输出到标准输出
			case "bash":
				err = cmd.Root().GenBashCompletion(os.Stdout)  # 生成 Bash shell 的自动补全并输出到标准输出
			case "fish":
// 生成 Fish shell 的自动补全脚本，并输出到标准输出流中
err = cmd.Root().GenFishCompletion(os.Stdout, true)

// 如果有错误发生，则返回错误
return err
```

```
// 列出本地 Docker 镜像
func listLocalDockerImages(prefix string) ([]string, error) {
	// 创建一个空的镜像标签列表
	var repoTags = make([]string, 0)
	// 创建一个后台上下文
	ctx := context.Background()
	// 创建一个 Docker 客户端
	cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
	// 如果创建客户端时发生错误，则返回空的镜像标签列表和错误
	if err != nil {
		return repoTags, err
	}

	// 只返回有标签的镜像
	imageListArgs := filters.NewArgs()
	imageListArgs.Add("dangling", "false")
	// 获取镜像列表
	images, err := cli.ImageList(ctx, types.ImageListOptions{All: false, Filters: imageListArgs})
	// 如果获取镜像列表时发生错误，则返回空的镜像标签列表和错误
	if err != nil {
		return repoTags, err
	}

	for _, image := range images {
		// 遍历每个镜像的标签
		for _, tag := range image.RepoTags {
			// 如果标签以指定前缀开头，则将其添加到 repoTags 列表中
			if strings.HasPrefix(tag, prefix) {
				repoTags = append(repoTags, tag)
			}
		}
	}
	// 返回筛选后的镜像标签列表和空错误
	return repoTags, nil
}

func dockerImageValidArgsFunction(_ *cobra.Command, _ []string, toComplete string) ([]string, cobra.ShellCompDirective) {
	// 由于我们使用了 ValidArgsFunction，Cobra 在解析所有提供的标志和参数之后会调用此函数
	dockerImageRepoTags, err := listLocalDockerImages(toComplete)
	if err != nil {
		// 表示发生错误，应忽略自动补全
		return []string{"completion failed"}, cobra.ShellCompDirectiveError
	}
	// 如果 dockerImageRepoTags 的长度为 0，则返回错误信息和错误指令
	if len(dockerImageRepoTags) == 0 {
		return []string{"no docker images found"}, cobra.ShellCompDirectiveError
	}
	// ShellCompDirectiveDefault 表示在提供完补全后，shell 将执行其默认行为（不暗示其他可能的指令）
	// 返回 dockerImageRepoTags 和 ShellCompDirectiveDefault 指令
	return dockerImageRepoTags, cobra.ShellCompDirectiveDefault
}
```