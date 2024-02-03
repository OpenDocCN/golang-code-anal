# `kubo\core\commands\config.go`

```go
package commands

import (
    "encoding/json"  // 导入处理 JSON 数据的包
    "errors"  // 导入处理错误的包
    "fmt"  // 导入格式化输出的包
    "io"  // 导入输入输出的包
    "os"  // 导入操作系统功能的包
    "os/exec"  // 导入执行外部命令的包
    "strings"  // 导入处理字符串的包

    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境的包
    "github.com/ipfs/kubo/repo"  // 导入仓库相关的包
    "github.com/ipfs/kubo/repo/fsrepo"  // 导入文件系统仓库相关的包

    "github.com/elgris/jsondiff"  // 导入处理 JSON 差异的包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 IPFS 命令相关的包
    config "github.com/ipfs/kubo/config"  // 导入配置相关的包
)

// ConfigUpdateOutput is config profile apply command's output
type ConfigUpdateOutput struct {
    OldCfg map[string]interface{}  // 旧配置的键值对
    NewCfg map[string]interface{}  // 新配置的键值对
}

type ConfigField struct {
    Key   string  // 配置字段的键
    Value interface{}  // 配置字段的值
}

const (
    configBoolOptionName   = "bool"  // 布尔类型配置选项的名称
    configJSONOptionName   = "json"  // JSON 类型配置选项的名称
    configDryRunOptionName = "dry-run"  // 模拟运行配置选项的名称
)

var ConfigCmd = &cmds.Command{  // 定义 ConfigCmd 命令
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline: "Get and set IPFS config values.",  // 命令的简短描述
        ShortDescription: `  // 命令的简短描述
'ipfs config' controls configuration variables. It works like 'git config'.
The configuration values are stored in a config file inside your IPFS_PATH.`,
        LongDescription: `  // 命令的详细描述
'ipfs config' controls configuration variables. It works
much like 'git config'. The configuration values are stored in a config
file inside your IPFS repository (IPFS_PATH).

Examples:

Get the value of the 'Datastore.Path' key:

  $ ipfs config Datastore.Path

Set the value of the 'Datastore.Path' key:

  $ ipfs config Datastore.Path ~/.ipfs/datastore
`,
    },
    Subcommands: map[string]*cmds.Command{  // 子命令
        "show":    configShowCmd,  // 显示配置的子命令
        "edit":    configEditCmd,  // 编辑配置的子命令
        "replace": configReplaceCmd,  // 替换配置的子命令
        "profile": configProfileCmd,  // 配置文件的子命令
    },
    Arguments: []cmds.Argument{  // 命令的参数
        cmds.StringArg("key", true, false, "The key of the config entry (e.g. \"Addresses.API\")."),  // 配置条目的键
        cmds.StringArg("value", false, false, "The value to set the config entry to."),  // 要设置的配置条目的值
    },
    Options: []cmds.Option{  // 命令的选项
        cmds.BoolOption(configBoolOptionName, "Set a boolean value."),  // 设置布尔值的选项
        cmds.BoolOption(configJSONOptionName, "Parse stringified JSON."),  // 解析 JSON 字符串的选项
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取请求参数
        args := req.Arguments
        # 获取参数中的键值
        key := args[0]

        var output *ConfigField

        # 这是一个临时修复，直到我们将私钥从配置文件中移出来
        switch strings.ToLower(key) {
        case "identity", "identity.privkey":
            return errors.New("cannot show or change private key through API")
        default:
        }

        # 临时修复，直到我们将 ApiKey 密钥从配置文件中移出来
        # （远程服务是一个映射，因此需要更高级的阻止）
        if blocked := matchesGlobPrefix(key, config.PinningConcealSelector); blocked {
            return errors.New("cannot show or change pinning services credentials")
        }

        # 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }
        # 打开存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        # 延迟关闭存储库
        defer r.Close()
        # 如果参数长度为2
        if len(args) == 2 {
            # 获取值
            value := args[1]

            # 如果解析为 JSON，则进行 JSON 解析
            if parseJSON, _ := req.Options[configJSONOptionName].(bool); parseJSON {
                var jsonVal interface{}
                if err := json.Unmarshal([]byte(value), &jsonVal); err != nil {
                    err = fmt.Errorf("failed to unmarshal json. %s", err)
                    return err
                }

                output, err = setConfig(r, key, jsonVal)
            } else if isbool, _ := req.Options[configBoolOptionName].(bool); isbool {
                output, err = setConfig(r, key, value == "true")
            } else {
                output, err = setConfig(r, key, value)
            }
        } else {
            output, err = getConfig(r, key)
        }

        if err != nil {
            return err
        }

        # 发射一次结果
        return cmds.EmitOnce(res, output)
    },
    # 创建 Encoders 对象，包含不同类型的编码器
    Encoders: cmds.EncoderMap{
        # 文本类型的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *ConfigField) error {
            # 如果请求参数长度为2，则返回空
            if len(req.Arguments) == 2 {
                return nil
            }

            # 将配置字段的值转换为人类可读的格式
            buf, err := config.HumanOutput(out.Value)
            if err != nil {
                return err
            }
            # 在缓冲区末尾添加换行符
            buf = append(buf, byte('\n'))

            # 将缓冲区内容写入到输出流中
            _, err = w.Write(buf)
            return err
        }),
    },
    # 创建空的配置字段对象
    Type: ConfigField{},
// matchesGlobPrefix函数用于判断key是否与glob匹配
// key是由逗号分隔的字符串"parts"组成的序列
// glob是由字符串"patterns"组成的序列
// matchesGlobPrefix尝试将前K个parts分别与前K个patterns匹配，其中K是key或glob中较短的长度
// 如果pattern是"*"或小写的pattern等于小写的part，则pattern与part匹配
//
// 例如：
//
//    matchesGlobPrefix("foo.bar", []string{"*", "bar", "baz"}) 返回true
//    matchesGlobPrefix("foo.bar.baz", []string{"*", "bar"}) 返回true
//    matchesGlobPrefix("foo.bar", []string{"baz", "*"}) 返回false
func matchesGlobPrefix(key string, glob []string) bool {
    // 将key按"."分割成字符串数组k
    k := strings.Split(key, ".")
    // 遍历glob中的每个pattern
    for i, g := range glob {
        // 如果i大于等于k的长度，则跳出循环
        if i >= len(k) {
            break
        }
        // 如果pattern是"*"，则继续下一次循环
        if g == "*" {
            continue
        }
        // 如果pattern与part不相等（不区分大小写），则返回false
        if !strings.EqualFold(k[i], g) {
            return false
        }
    }
    // 如果所有的pattern都匹配，则返回true
    return true
}

// 定义configShowCmd变量，类型为*cmds.Command
var configShowCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        // 简短描述
        Tagline: "Output config file contents.",
        // 短描述
        ShortDescription: `
NOTE: For security reasons, this command will omit your private key and remote services. If you would like to make a full backup of your config (private key included), you must copy the config file from your repo.
`,
    },
    // 类型
    Type: make(map[string]interface{}),
    # Run 函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        # 从请求选项中获取配置文件名
        configFileOpt, _ := req.Options[ConfigFileOption].(string)
        # 获取配置文件的完整路径
        fname, err := config.Filename(cfgRoot, configFileOpt)
        if err != nil {
            return err
        }

        # 读取配置文件内容
        data, err := os.ReadFile(fname)
        if err != nil {
            return err
        }

        # 解析配置文件内容为 map
        var cfg map[string]interface{}
        err = json.Unmarshal(data, &cfg)
        if err != nil {
            return err
        }

        # 清除敏感信息字段的值
        cfg, err = scrubValue(cfg, []string{config.IdentityTag, config.PrivKeyTag})
        if err != nil {
            return err
        }

        # 清除 API 相关信息字段的值
        cfg, err = scrubValue(cfg, []string{config.APITag, config.AuthorizationTag})
        if err != nil {
            return err
        }

        # 清除可选值字段的值
        cfg, err = scrubOptionalValue(cfg, config.PinningConcealSelector)
        if err != nil {
            return err
        }

        # 将处理后的配置信息发送给响应
        return cmds.EmitOnce(res, &cfg)
    },
    # 设置编码器，将结果以人类可读的 JSON 格式返回
    Encoders: cmds.EncoderMap{
        cmds.Text: HumanJSONEncoder,
    },
// 定义 HumanJSONEncoder 变量，使用 MakeTypedEncoder 函数创建一个编码器
var HumanJSONEncoder = cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *map[string]interface{}) error {
    // 调用 config.HumanOutput 函数将 out 转换为字节流
    buf, err := config.HumanOutput(out)
    // 如果转换出错，则返回错误
    if err != nil {
        return err
    }
    // 在 buf 后追加换行符
    buf = append(buf, byte('\n'))
    // 将 buf 写入到 w 中
    _, err = w.Write(buf)
    return err
})

// Scrubs value and returns error if missing
// 如果值缺失，则清除并返回错误
func scrubValue(m map[string]interface{}, key []string) (map[string]interface{}, error) {
    return scrubMapInternal(m, key, false)
}

// Scrubs value and returns no error if missing
// 如果值缺失，则清除并返回无错误
func scrubOptionalValue(m map[string]interface{}, key []string) (map[string]interface{}, error) {
    return scrubMapInternal(m, key, true)
}

// Scrubs either map or value based on the type of u
// 根据 u 的类型，清除 map 或 value
func scrubEither(u interface{}, key []string, okIfMissing bool) (interface{}, error) {
    // 将 u 转换为 map[string]interface{} 类型
    m, ok := u.(map[string]interface{})
    // 如果转换成功，则调用 scrubMapInternal 函数清除 map
    if ok {
        return scrubMapInternal(m, key, okIfMissing)
    }
    // 否则调用 scrubValueInternal 函数清除 value
    return scrubValueInternal(m, key, okIfMissing)
}

// Scrubs value and returns error if missing
// 如果值缺失，则返回错误
func scrubValueInternal(v interface{}, key []string, okIfMissing bool) (interface{}, error) {
    // 如果 v 为 nil 且不允许缺失，则返回错误
    if v == nil && !okIfMissing {
        return nil, errors.New("failed to find specified key")
    }
    return nil, nil
}

// Scrubs map and returns error if missing
// 如果 map 缺失，则返回错误
func scrubMapInternal(m map[string]interface{}, key []string, okIfMissing bool) (map[string]interface{}, error) {
    // 如果 key 长度为 0，则删除值并返回空 map
    if len(key) == 0 {
        return make(map[string]interface{}), nil // delete value
    }
    // 创建一个新的 map
    n := map[string]interface{}{}
    // 遍历原始 map
    for k, v := range m {
        // 如果 key[0] 为 "*" 或与 k 不区分大小写相等
        if key[0] == "*" || strings.EqualFold(key[0], k) {
            // 调用 scrubEither 函数清除指定的 key
            u, err := scrubEither(v, key[1:], okIfMissing)
            // 如果出错，则返回错误
            if err != nil {
                return nil, err
            }
            // 如果清除后的值不为 nil，则添加到新的 map 中
            if u != nil {
                n[k] = u
            }
        } else {
            // 否则将原始值添加到新的 map 中
            n[k] = v
        }
    }
    return n, nil
}

// 定义 configEditCmd 命令
var configEditCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Open the config file for editing in $EDITOR.",
        ShortDescription: `
To use 'ipfs config edit', you must have the $EDITOR environment
variable set to your preferred text editor.
`,
    },
    // 设置 NoRemote 为 true
    NoRemote: true,
    // 创建额外的命令参数，设置不使用仓库为 true
    Extra:    CreateCmdExtras(SetDoesNotUseRepo(true)),
    // 定义一个匿名函数，接受请求、响应和环境参数，返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取配置根目录和错误信息
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        // 从请求的选项中获取配置文件选项
        configFileOpt, _ := req.Options[ConfigFileOption].(string)
        // 获取配置文件名和错误信息
        filename, err := config.Filename(cfgRoot, configFileOpt)
        if err != nil {
            return err
        }

        // 调用 editConfig 函数并返回结果
        return editConfig(filename)
    },
# 定义一个名为configReplaceCmd的命令对象，用于替换配置文件
var configReplaceCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Replace the config with <file>.",  # 简短描述命令的作用
        ShortDescription: `
Make sure to back up the config file first if necessary, as this operation
can't be undone.
`,  # 提供更详细的命令说明
    },

    Arguments: []cmds.Argument{  # 定义命令的参数
        cmds.FileArg("file", true, false, "The file to use as the new config."),  # 接受一个文件作为新配置的参数
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  # 定义命令的执行函数
        cfgRoot, err := cmdenv.GetConfigRoot(env)  # 获取配置文件的根目录
        if err != nil {
            return err
        }

        r, err := fsrepo.Open(cfgRoot)  # 打开配置文件
        if err != nil {
            return err
        }
        defer r.Close()  # 延迟关闭配置文件

        file, err := cmdenv.GetFileArg(req.Files.Entries())  # 获取命令参数中的文件
        if err != nil {
            return err
        }
        defer file.Close()  # 延迟关闭文件

        return replaceConfig(r, file)  # 调用替换配置的函数
    },
}

# 定义一个名为configProfileCmd的命令对象，用于应用配置文件的配置文件
var configProfileCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Apply profiles to config.",  # 简短描述命令的作用
        ShortDescription: fmt.Sprintf(`
Available profiles:
%s
`, buildProfileHelp()),  # 提供更详细的命令说明，包括可用的配置文件
    },

    Subcommands: map[string]*cmds.Command{  # 定义子命令
        "apply": configProfileApplyCmd,  # 子命令为应用配置文件
    },
}

# 定义一个名为configProfileApplyCmd的命令对象，用于应用配置文件的配置文件
var configProfileApplyCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Apply profile to config.",  # 简短描述命令的作用
    },
    Options: []cmds.Option{  # 定义命令的选项
        cmds.BoolOption(configDryRunOptionName, "print difference between the current config and the config that would be generated"),  # 定义一个布尔选项，用于打印当前配置和将要生成的配置之间的差异
    },
    Arguments: []cmds.Argument{  # 定义命令的参数
        cmds.StringArg("profile", true, false, "The profile to apply to the config."),  # 接受一个配置文件作为参数
    },
}
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从配置文件中获取指定名称的配置文件
        profile, ok := config.Profiles[req.Arguments[0]]
        if !ok {
            return fmt.Errorf("%s is not a profile", req.Arguments[0])
        }

        # 从请求中获取是否为模拟运行的标志
        dryRun, _ := req.Options[configDryRunOptionName].(bool)
        # 获取配置文件的根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        # 转换配置文件，获取旧配置、新配置和可能的错误
        oldCfg, newCfg, err := transformConfig(cfgRoot, req.Arguments[0], profile.Transform, dryRun)
        if err != nil {
            return err
        }

        # 清除旧配置中的私钥信息，获取清除后的配置和可能的错误
        oldCfgMap, err := scrubPrivKey(oldCfg)
        if err != nil {
            return err
        }

        # 清除新配置中的私钥信息，获取清除后的配置和可能的错误
        newCfgMap, err := scrubPrivKey(newCfg)
        if err != nil {
            return err
        }

        # 发送一次性的输出，包括旧配置和新配置
        return cmds.EmitOnce(res, &ConfigUpdateOutput{
            OldCfg: oldCfgMap,
            NewCfg: newCfgMap,
        })
    },
    # 编码器映射，将文本格式的输出编码为指定格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *ConfigUpdateOutput) error {
            # 比较旧配置和新配置的差异，格式化为 JSON 格式
            diff := jsondiff.Compare(out.OldCfg, out.NewCfg)
            buf := jsondiff.Format(diff)

            # 将格式化后的差异写入输出流
            _, err := w.Write(buf)
            return err
        }),
    },
    # 配置更新输出的类型
    Type: ConfigUpdateOutput{},
// buildProfileHelp 构建配置文件帮助信息
func buildProfileHelp() string {
    var out string

    // 遍历配置文件中的每个配置文件名和配置文件
    for name, profile := range config.Profiles {
        // 将配置文件描述按换行符分割成多行
        dlines := strings.Split(profile.Description, "\n")
        // 在每一行描述前添加四个空格
        for i := range dlines {
            dlines[i] = "    " + dlines[i]
        }

        // 将格式化后的配置文件名和描述拼接到输出字符串中
        out = out + fmt.Sprintf("  '%s':\n%s\n", name, strings.Join(dlines, "\n"))
    }

    return out
}

// scrubPrivKey 为安全原因清除私钥
func scrubPrivKey(cfg *config.Config) (map[string]interface{}, error) {
    // 将配置文件对象转换为 map
    cfgMap, err := config.ToMap(cfg)
    if err != nil {
        return nil, err
    }

    // 清除指定字段的值
    cfgMap, err = scrubValue(cfgMap, []string{config.IdentityTag, config.PrivKeyTag})
    if err != nil {
        return nil, err
    }

    return cfgMap, nil
}

// transformConfig 返回旧配置和新配置，而不是它们之间的差异，因为应用命令可以通过这种方式提供稳定的 API。
// 如果 dryRun 为 true，则不应更新并持久化存储库的配置。否则，应更新并持久化存储库的配置。
func transformConfig(configRoot string, configName string, transformer config.Transformer, dryRun bool) (*config.Config, *config.Config, error) {
    // 打开配置根目录的存储库
    r, err := fsrepo.Open(configRoot)
    if err != nil {
        return nil, nil, err
    }
    defer r.Close()

    // 获取旧配置
    oldCfg, err := r.Config()
    if err != nil {
        return nil, nil, err
    }

    // 复制一份旧配置，以避免意外更新存储库的配置
    newCfg, err := oldCfg.Clone()
    if err != nil {
        return nil, nil, err
    }

    // 对新配置进行转换
    err = transformer(newCfg)
    if err != nil {
        return nil, nil, err
    }

    // 如果不是 dryRun 模式，则备份旧配置并更新存储库的配置
    if !dryRun {
        _, err = r.BackupConfig("pre-" + configName + "-")
        if err != nil {
            return nil, nil, err
        }

        err = r.SetConfig(newCfg)
        if err != nil {
            return nil, nil, err
        }
    }

    return oldCfg, newCfg, nil
}

// getConfig 获取存储库中指定键的配置字段
func getConfig(r repo.Repo, key string) (*ConfigField, error) {
    // 通过调用 GetConfigKey 方法获取指定 key 对应的 value 值
    value, err := r.GetConfigKey(key)
    // 如果获取过程中出现错误，返回错误信息
    if err != nil {
        return nil, fmt.Errorf("failed to get config value: %q", err)
    }
    // 返回包含 key 和 value 的 ConfigField 结构体指针
    return &ConfigField{
        Key:   key,
        Value: value,
    }, nil
}

这是一个函数结束的标志，表示上一个函数的结束。


func setConfig(r repo.Repo, key string, value interface{}) (*ConfigField, error) {
    // 使用提供的键值对设置配置项，并返回设置后的配置项和可能出现的错误
    err := r.SetConfigKey(key, value)
    if err != nil {
        return nil, fmt.Errorf("failed to set config value: %s (maybe use --json?)", err)
    }
    return getConfig(r, key)
}

这是一个设置配置项的函数，根据提供的键值对设置配置项，并返回设置后的配置项和可能出现的错误。


func editConfig(filename string) error {
    // 获取环境变量中的编辑器
    editor := os.Getenv("EDITOR")
    if editor == "" {
        return errors.New("ENV variable $EDITOR not set")
    }

    // 使用编辑器打开指定的文件
    cmd := exec.Command(editor, filename)
    cmd.Stdin, cmd.Stdout, cmd.Stderr = os.Stdin, os.Stdout, os.Stderr
    return cmd.Run()
}

这是一个编辑配置的函数，根据环境变量中的编辑器打开指定的文件进行编辑。


func replaceConfig(r repo.Repo, file io.Reader) error {
    var newCfg config.Config
    // 解析输入的文件内容为配置对象
    if err := json.NewDecoder(file).Decode(&newCfg); err != nil {
        return errors.New("failed to decode file as config")
    }

    // 处理身份私钥（秘密信息）
    // ...

    // 处理固定远程服务（每个服务的 API 密钥是秘密信息）
    // ...
}

这是一个替换配置的函数，根据输入的文件内容解析为配置对象，并处理其中的私钥和远程服务信息。
    // 遍历旧服务列表，获取服务名和服务对象
    for name, oldSvc := range oldServices {
        // 检查新服务列表中是否存在同名服务
        if newSvc, hadSvc := newServices[name]; hadSvc {
            // 如果新服务的 API 信息发生变化，或者 Endpoint 发生变化且不为空，则返回错误
            // （与配置显示的互操作性：允许 Endpoint 只要没有发生变化）
            if len(newSvc.API.Key) != 0 || (len(newSvc.API.Endpoint) != 0 && newSvc.API.Endpoint != oldSvc.API.Endpoint) {
                return errors.New("cannot change remote pinning services api info with `config replace`")
            }
            // 重新应用 API 信息并将服务存储在更新后的配置中
            newSvc.API = oldSvc.API
            newCfg.Pinning.RemoteServices[name] = newSvc
        } else {
            // 尝试添加或删除远程固定服务时返回错误
            return errors.New("cannot add or remove remote pinning services with 'config replace'")
        }
    }

    // 设置新的配置并返回结果
    return r.SetConfig(&newCfg)
// 获取远程固定服务列表
func getRemotePinningServices(r repo.Repo) (map[string]config.RemotePinningService, error) {
    // 声明一个旧的服务列表
    var oldServices map[string]config.RemotePinningService
    // 获取远程服务标签
    if remoteServicesTag, err := getConfig(r, config.RemoteServicesPath); err == nil {
        // 似乎 golang 无法将 map[string]interface{} 类型断言为 map[string]config.RemotePinningService
        // 因此我们必须手动复制数据 :-|
        if val, ok := remoteServicesTag.Value.(map[string]interface{}); ok {
            // 将值转换为 JSON 字符串
            jsonString, err := json.Marshal(val)
            if err != nil {
                return nil, err
            }
            // 将 JSON 字符串解析为旧服务列表
            err = json.Unmarshal(jsonString, &oldServices)
            if err != nil {
                return nil, err
            }
        }
    }
    // 返回旧服务列表和空错误
    return oldServices, nil
}
```