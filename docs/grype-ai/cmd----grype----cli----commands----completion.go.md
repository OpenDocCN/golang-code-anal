# `grype\cmd\grype\cli\commands\completion.go`

```go
package commands

import (
    "context"
    "os"
    "strings"

    "github.com/docker/docker/api/types"
    "github.com/docker/docker/api/types/filters"
    "github.com/docker/docker/client"
    "github.com/spf13/cobra"
)

// Completion returns a command to provide completion to various terminal shells
// Completion函数返回一个命令，用于为各种终端shell提供自动补全
func Completion() *cobra.Command {
    return &cobra.Command{
        Use:   "completion [bash|zsh|fish]",
        // Use字段定义命令的使用方式
        Short: "Generate a shell completion for Grype (listing local docker images)",
        // Short字段提供命令的简短描述
        Long: `To load completions (docker image list):

Bash:

$ source <(grype completion bash)

# To load completions for each session, execute once:
Linux:
  $ grype completion bash > /etc/bash_completion.d/grype
MacOS:
  $ grype completion bash > /usr/local/etc/bash_completion.d/grype

Zsh:

# If shell completion is not already enabled in your environment you will need
# to enable it.  You can execute the following once:

$ echo "autoload -U compinit; compinit" >> ~/.zshrc

# To load completions for each session, execute once:
$ grype completion zsh > "${fpath[1]}/_grype"

# You will need to start a new shell for this setup to take effect.

Fish:

$ grype completion fish | source

# To load completions for each session, execute once:
$ grype completion fish > ~/.config/fish/completions/grype.fish
`,
        // Long字段提供命令的详细描述
        DisableFlagsInUseLine: true,
        // DisableFlagsInUseLine字段用于禁用命令使用行中的标志
        ValidArgs:             []string{"bash", "fish", "zsh"},
        // ValidArgs字段定义命令的有效参数
        Args:                  cobra.MatchAll(cobra.ExactArgs(1), cobra.OnlyValidArgs),
        // Args字段定义命令的参数匹配规则
        RunE: func(cmd *cobra.Command, args []string) error {
            var err error
            switch args[0] {
            case "zsh":
                err = cmd.Root().GenZshCompletion(os.Stdout)
            case "bash":
                err = cmd.Root().GenBashCompletion(os.Stdout)
            case "fish":
                err = cmd.Root().GenFishCompletion(os.Stdout, true)
            }
            return err
        },
        // RunE字段定义命令的执行逻辑
    }
}
func listLocalDockerImages(prefix string) ([]string, error) {
    // 创建一个空的字符串切片用于存储镜像的标签
    var repoTags = make([]string, 0)
    // 创建一个后台上下文
    ctx := context.Background()
    // 使用环境变量创建一个 Docker 客户端
    cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
    // 如果创建客户端出现错误，则返回空的标签切片和错误
    if err != nil {
        return repoTags, err
    }

    // 只想要返回有标签的镜像
    imageListArgs := filters.NewArgs()
    imageListArgs.Add("dangling", "false")
    // 获取本地镜像列表
    images, err := cli.ImageList(ctx, types.ImageListOptions{All: false, Filters: imageListArgs})
    // 如果获取镜像列表出现错误，则返回空的标签切片和错误
    if err != nil {
        return repoTags, err
    }

    // 遍历获取到的镜像列表
    for _, image := range images {
        // 镜像可能有多个标签
        for _, tag := range image.RepoTags {
            // 如果标签以指定前缀开头，则将其添加到 repoTags 中
            if strings.HasPrefix(tag, prefix) {
                repoTags = append(repoTags, tag)
            }
        }
    }
    // 返回获取到的镜像标签切片和空错误
    return repoTags, nil
}

func dockerImageValidArgsFunction(_ *cobra.Command, _ []string, toComplete string) ([]string, cobra.ShellCompDirective) {
    // 由于使用了 ValidArgsFunction，Cobra 在解析所有提供的标志和参数之后会调用此函数
    dockerImageRepoTags, err := listLocalDockerImages(toComplete)
    // 如果出现错误，则指示应忽略自动补全，并返回一个包含错误信息的字符串切片
    if err != nil {
        return []string{"completion failed"}, cobra.ShellCompDirectiveError
    }
    // 如果没有找到 Docker 镜像，则返回一个包含错误信息的字符串切片
    if len(dockerImageRepoTags) == 0 {
        return []string{"no docker images found"}, cobra.ShellCompDirectiveError
    }
    // ShellCompDirectiveDefault 表示在提供自动补全后，shell 将执行其默认行为（不暗示其他可能的指令）
    return dockerImageRepoTags, cobra.ShellCompDirectiveDefault
}
```