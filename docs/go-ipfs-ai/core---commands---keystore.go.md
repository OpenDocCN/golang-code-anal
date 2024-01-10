# `kubo\core\commands\keystore.go`

```
package commands

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "crypto/ed25519"  // 导入 ed25519 包，用于加密和签名
    "crypto/x509"  // 导入 x509 包，用于操作证书
    "encoding/pem"  // 导入 pem 包，用于操作 PEM 编码的数据
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于进行 I/O 操作
    "os"  // 导入 os 包，提供了对操作系统功能的访问
    "path/filepath"  // 导入 filepath 包，用于操作文件路径
    "strings"  // 导入 strings 包，用于操作字符串
    "text/tabwriter"  // 导入 tabwriter 包，用于格式化表格输出

    keystore "github.com/ipfs/boxo/keystore"  // 导入 keystore 包，用于存储密钥
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包，用于创建命令行接口
    oldcmds "github.com/ipfs/kubo/commands"  // 导入 oldcmds 包，用于处理旧的命令
    config "github.com/ipfs/kubo/config"  // 导入 config 包，用于处理配置信息
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包，用于处理命令环境
    "github.com/ipfs/kubo/core/commands/e"  // 导入 e 包
    ke "github.com/ipfs/kubo/core/commands/keyencode"  // 导入 ke 包，用于编码密钥
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包，用于处理核心接口选项
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 导入 fsrepo 包，用于处理文件系统存储库
    migrations "github.com/ipfs/kubo/repo/fsrepo/migrations"  // 导入 migrations 包，用于处理存储库迁移
    "github.com/libp2p/go-libp2p/core/crypto"  // 导入 crypto 包，用于处理 libp2p 核心加密
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入 peer 包，用于处理 libp2p 核心对等节点
    mbase "github.com/multiformats/go-multibase"  // 导入 mbase 包，用于处理多格式的基数编码
)

var KeyCmd = &cmds.Command{  // 定义 KeyCmd 变量，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置 Helptext 字段为 cmds.HelpText 结构体
        Tagline: "Create and list IPNS name keypairs",  // 设置 Tagline 字段
        ShortDescription: `  // 设置 ShortDescription 字段
'ipfs key gen' generates a new keypair for usage with IPNS and 'ipfs name
publish'.

  > ipfs key gen --type=rsa --size=2048 mykey
  > ipfs name publish --key=mykey QmSomeHash

'ipfs key list' lists the available keys.

  > ipfs key list
  self
  mykey
        `,
    },
    Subcommands: map[string]*cmds.Command{  // 设置 Subcommands 字段为 map 类型
        "gen":    keyGenCmd,  // 设置 "gen" 键对应的值为 keyGenCmd
        "export": keyExportCmd,  // 设置 "export" 键对应的值为 keyExportCmd
        "import": keyImportCmd,  // 设置 "import" 键对应的值为 keyImportCmd
        "list":   keyListCmd,  // 设置 "list" 键对应的值为 keyListCmd
        "rename": keyRenameCmd,  // 设置 "rename" 键对应的值为 keyRenameCmd
        "rm":     keyRmCmd,  // 设置 "rm" 键对应的值为 keyRmCmd
        "rotate": keyRotateCmd,  // 设置 "rotate" 键对应的值为 keyRotateCmd
        "sign":   keySignCmd,  // 设置 "sign" 键对应的值为 keySignCmd
        "verify": keyVerifyCmd,  // 设置 "verify" 键对应的值为 keyVerifyCmd
    },
}

type KeyOutput struct {  // 定义 KeyOutput 结构体
    Name string  // 定义 Name 字段
    Id   string //nolint  // 定义 Id 字段
}

type KeyOutputList struct {  // 定义 KeyOutputList 结构体
    Keys []KeyOutput  // 定义 Keys 字段为 KeyOutput 类型的切片
}

// KeyRenameOutput define the output type of keyRenameCmd
type KeyRenameOutput struct {  // 定义 KeyRenameOutput 结构体
    Was       string  // 定义 Was 字段
    Now       string  // 定义 Now 字段
    Id        string //nolint  // 定义 Id 字段
    Overwrite bool  // 定义 Overwrite 字段
}

const (  // 定义常量块
    keyStoreAlgorithmDefault = options.Ed25519Key  // 设置 keyStoreAlgorithmDefault 常量
    keyStoreTypeOptionName   = "type"  // 设置 keyStoreTypeOptionName 常量
    keyStoreSizeOptionName   = "size"  // 设置 keyStoreSizeOptionName 常量
    # 定义一个变量 oldKeyOptionName，用于存储字符串 "oldkey"
    oldKeyOptionName         = "oldkey"
// 定义 keyGenCmd 命令，用于创建新的密钥对
var keyGenCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Create a new keypair",  // 帮助文本，用于描述命令的简要作用
    },
    Options: []cmds.Option{
        cmds.StringOption(keyStoreTypeOptionName, "t", "type of the key to create: rsa, ed25519").WithDefault(keyStoreAlgorithmDefault),  // 定义命令选项，用于指定要创建的密钥类型，默认为 keyStoreAlgorithmDefault
        cmds.IntOption(keyStoreSizeOptionName, "s", "size of the key to generate"),  // 定义命令选项，用于指定要生成的密钥大小
        ke.OptionIPNSBase,  // IPNS 基础选项
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("name", true, false, "name of key to create"),  // 定义命令参数，用于指定要创建的密钥的名称
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取环境中的 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        typ, f := req.Options[keyStoreTypeOptionName].(string)  // 获取命令选项中的密钥类型
        if !f {
            return fmt.Errorf("please specify a key type with --type")  // 如果未指定密钥类型，则返回错误信息
        }

        name := req.Arguments[0]  // 获取命令参数中的密钥名称
        if name == "self" {
            return fmt.Errorf("cannot create key with name 'self'")  // 如果密钥名称为 "self"，则返回错误信息
        }

        opts := []options.KeyGenerateOption{options.Key.Type(typ)}  // 创建密钥生成选项列表，指定密钥类型

        size, sizefound := req.Options[keyStoreSizeOptionName].(int)  // 获取命令选项中的密钥大小
        if sizefound {
            opts = append(opts, options.Key.Size(size))  // 如果指定了密钥大小，则将其添加到选项列表中
        }
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))  // 从命令选项中获取密钥编码器
        if err != nil {
            return err
        }

        key, err := api.Key().Generate(req.Context, name, opts...)  // 调用 API 对象的生成密钥方法，生成密钥
        if err != nil {
            return err
        }

        return cmds.EmitOnce(res, &KeyOutput{  // 发送密钥输出到响应流
            Name: name,
            Id:   keyEnc.FormatID(key.ID()),  // 格式化密钥 ID，并设置到输出中
        })
    },
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ko *KeyOutput) error {  // 定义文本编码器，用于将密钥输出写入到流中
            _, err := w.Write([]byte(ko.Id + "\n"))  // 将密钥 ID 写入到流中
            return err
        }),
    },
    Type: KeyOutput{},  // 设置命令类型为 KeyOutput
}

const (
    // Key format options used both for importing and exporting.
    // 用于导入和导出的密钥格式选项
    # 定义密钥格式选项的名称
    keyFormatOptionName            = "format"
    # 定义 PEM PKCS8 明文格式选项的名称
    keyFormatPemCleartextOption    = "pem-pkcs8-cleartext"
    # 定义 Libp2p Protobuf 明文格式选项的名称
    keyFormatLibp2pCleartextOption = "libp2p-protobuf-cleartext"
    # 定义允许任何类型密钥的选项名称
    keyAllowAnyTypeOptionName      = "allow-any-key-type"
# 定义 keyExportCmd 命令
var keyExportCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Export a keypair",
        ShortDescription: `
Exports a named libp2p key to disk.

By default, the output will be stored at './<key-name>.key', but an alternate
path can be specified with '--output=<path>' or '-o=<path>'.

It is possible to export a private key to interoperable PEM PKCS8 format by explicitly
passing '--format=pem-pkcs8-cleartext'. The resulting PEM file can then be consumed
elsewhere. For example, using openssl to get a PEM with public key:

  $ ipfs key export testkey --format=pem-pkcs8-cleartext -o privkey.pem
  $ openssl pkey -in privkey.pem -pubout > pubkey.pem
`,
    },
    # 参数列表，包括名称、是否必需、是否从标准输入读取等信息
    Arguments: []cmds.Argument{
        cmds.StringArg("name", true, false, "name of key to export").EnableStdin(),
    },
    # 选项列表，包括名称、简写、描述和默认值
    Options: []cmds.Option{
        cmds.StringOption(outputOptionName, "o", "The path where the output should be stored."),
        cmds.StringOption(keyFormatOptionName, "f", "The format of the exported private key, libp2p-protobuf-cleartext or pem-pkcs8-cleartext.").WithDefault(keyFormatLibp2pCleartextOption),
    },
    # 不允许远程执行
    NoRemote: true,
    },
    # 定义 PostRunMap 结构体，包含 CLI 命令的处理函数
    PostRun: cmds.PostRunMap{
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 获取响应中的请求对象
            req := res.Request()

            # 从响应中获取下一个值
            v, err := res.Next()
            if err != nil {
                return err
            }

            # 将值转换为 io.Reader 接口
            outReader, ok := v.(io.Reader)
            if !ok {
                return e.New(e.TypeErr(outReader, v))
            }

            # 获取输出路径和导出格式选项
            outPath, _ := req.Options[outputOptionName].(string)
            exportFormat, _ := req.Options[keyFormatOptionName].(string)
            if outPath == "" {
                # 根据导出格式设置文件扩展名
                var fileExtension string
                switch exportFormat {
                case keyFormatPemCleartextOption:
                    fileExtension = "pem"
                case keyFormatLibp2pCleartextOption:
                    fileExtension = "key"
                }
                # 根据请求参数和文件扩展名生成输出路径
                trimmed := strings.TrimRight(fmt.Sprintf("%s.%s", req.Arguments[0], fileExtension), "/")
                _, outPath = filepath.Split(trimmed)
                outPath = filepath.Clean(outPath)
            }

            # 创建文件
            file, err := os.Create(outPath)
            if err != nil {
                return err
            }
            # 延迟关闭文件
            defer file.Close()

            # 根据导出格式进行处理
            switch exportFormat {
            case keyFormatPemCleartextOption:
                # 读取私钥数据并编码为 PEM 格式写入文件
                privKeyBytes, err := io.ReadAll(outReader)
                if err != nil {
                    return err
                }

                err = pem.Encode(file, &pem.Block{
                    Type:  "PRIVATE KEY",
                    Bytes: privKeyBytes,
                })
                if err != nil {
                    return fmt.Errorf("encoding PEM block: %w", err)
                }

            case keyFormatLibp2pCleartextOption:
                # 将数据直接拷贝到文件中
                _, err = io.Copy(file, outReader)
                if err != nil {
                    return err
                }
            }

            # 返回空错误表示处理成功
            return nil
        },
    },
# 定义名为keyImportCmd的命令对象，用于导入密钥并打印导入的密钥ID
var keyImportCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Import a key and prints imported key id",  # 简短描述导入密钥并打印导入的密钥ID
        ShortDescription: `  # 详细描述导入密钥的功能和用法
Imports a key and stores it under the provided name.  # 导入密钥并将其存储在提供的名称下

By default, the key is assumed to be in 'libp2p-protobuf-cleartext' format,  # 默认情况下，假定密钥采用'libp2p-protobuf-cleartext'格式
however it is possible to import private keys wrapped in interoperable PEM PKCS8  # 但也可以导入包装在可互操作的PEM PKCS8中的私钥
by passing '--format=pem-pkcs8-cleartext'.  # 通过传递'--format=pem-pkcs8-cleartext'参数

The PEM format allows for key generation outside of the IPFS node:  # PEM格式允许在IPFS节点外生成密钥
  $ openssl genpkey -algorithm ED25519 > ed25519.pem  # 使用openssl生成ED25519算法的密钥
  $ ipfs key import test-openssl -f pem-pkcs8-cleartext ed25519.pem  # 使用ipfs导入PEM格式的密钥
`,
    },
    Options: []cmds.Option{  # 定义命令的选项
        ke.OptionIPNSBase,  # IPNS基础选项
        cmds.StringOption(keyFormatOptionName, "f", "The format of the private key to import, libp2p-protobuf-cleartext or pem-pkcs8-cleartext.").WithDefault(keyFormatLibp2pCleartextOption),  # 导入私钥的格式选项，默认为libp2p-protobuf-cleartext
        cmds.BoolOption(keyAllowAnyTypeOptionName, "Allow importing any key type.").WithDefault(false),  # 允许导入任何类型的密钥
    },
    Arguments: []cmds.Argument{  # 定义命令的参数
        cmds.StringArg("name", true, false, "name to associate with key in keychain"),  # 与密钥关联的名称
        cmds.FileArg("key", true, false, "key provided by generate or export"),  # 由generate或export提供的密钥
    },
    },
    Encoders: cmds.EncoderMap{  # 定义命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ko *KeyOutput) error {  # 使用文本编码器
            _, err := w.Write([]byte(ko.Id + "\n"))  # 写入导入的密钥ID
            return err  # 返回错误（如果有）
        }),
    },
    Type: KeyOutput{},  # 定义命令的输出类型为KeyOutput
}

# 定义名为keyListCmd的命令对象，用于列出所有本地密钥对
var keyListCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List all local keypairs.",  # 简短描述列出所有本地密钥对
    },
    Options: []cmds.Option{  # 定义命令的选项
        cmds.BoolOption("l", "Show extra information about keys."),  # 显示有关密钥的额外信息
        ke.OptionIPNSBase,  # IPNS基础选项
    },
    # 定义一个名为 Run 的函数，接受请求、响应和环境作为参数，并返回一个错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求的选项中获取 IPNSBase，将其转换为密钥编码器
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))
        if err != nil {
            return err
        }

        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 使用 API 对象获取密钥列表
        keys, err := api.Key().List(req.Context)
        if err != nil {
            return err
        }

        # 创建一个空的密钥输出列表
        list := make([]KeyOutput, 0, len(keys))

        # 遍历密钥列表，将每个密钥的名称和格式化后的 ID 添加到输出列表中
        for _, key := range keys {
            list = append(list, KeyOutput{
                Name: key.Name(),
                Id:   keyEnc.FormatID(key.ID()),
            })
        }

        # 将密钥输出列表发送给响应
        return cmds.EmitOnce(res, &KeyOutputList{list})
    },
    # 定义一个编码器映射，将文本格式与密钥输出列表编码器关联
    Encoders: cmds.EncoderMap{
        cmds.Text: keyOutputListEncoders(),
    },
    # 定义类型为 KeyOutputList 的结构
    Type: KeyOutputList{},
// 定义常量 keyStoreForceOptionName 为 "force"
const (
    keyStoreForceOptionName = "force"
)

// 定义 keyRenameCmd 命令
var keyRenameCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Rename a keypair.", // 帮助文本，用于描述命令的简短说明
    },
    Arguments: []cmds.Argument{ // 定义命令的参数
        cmds.StringArg("name", true, false, "name of key to rename"), // 字符串参数，名称为 name，必需，不可多次出现，描述为 "name of key to rename"
        cmds.StringArg("newName", true, false, "new name of the key"), // 字符串参数，名称为 newName，必需，不可多次出现，描述为 "new name of the key"
    },
    Options: []cmds.Option{ // 定义命令的选项
        cmds.BoolOption(keyStoreForceOptionName, "f", "Allow to overwrite an existing key."), // 布尔选项，名称为 keyStoreForceOptionName，简写为 f，描述为 "Allow to overwrite an existing key."
        ke.OptionIPNSBase, // IPNS 基础选项
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 定义命令的运行函数
        api, err := cmdenv.GetApi(env, req) // 获取环境中的 API
        if err != nil {
            return err
        }
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string)) // 从选项中获取 keyEnc
        if err != nil {
            return err
        }

        name := req.Arguments[0] // 获取第一个参数值
        newName := req.Arguments[1] // 获取第二个参数值
        force, _ := req.Options[keyStoreForceOptionName].(bool) // 获取 force 选项值

        key, overwritten, err := api.Key().Rename(req.Context, name, newName, options.Key.Force(force)) // 重命名密钥
        if err != nil {
            return err
        }

        return cmds.EmitOnce(res, &KeyRenameOutput{ // 发送一次命令输出
            Was:       name, // 原名称
            Now:       newName, // 新名称
            Id:        keyEnc.FormatID(key.ID()), // 格式化 ID
            Overwrite: overwritten, // 是否覆盖
        })
    },
    Encoders: cmds.EncoderMap{ // 定义命令的编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, kro *KeyRenameOutput) error { // 文本编码器
            if kro.Overwrite { // 如果覆盖
                fmt.Fprintf(w, "Key %s renamed to %s with overwriting\n", kro.Id, cmdenv.EscNonPrint(kro.Now)) // 输出重命名信息
            } else {
                fmt.Fprintf(w, "Key %s renamed to %s\n", kro.Id, cmdenv.EscNonPrint(kro.Now)) // 输出重命名信息
            }
            return nil
        }),
    },
    Type: KeyRenameOutput{}, // 命令的输出类型为 KeyRenameOutput
}

// 定义 keyRmCmd 命令
var keyRmCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Remove a keypair.", // 帮助文本，用于描述命令的简短说明
    },
    # 定义命令的参数
    Arguments: []cmds.Argument{
        # 定义字符串类型的参数，必填，启用标准输入
        cmds.StringArg("name", true, true, "names of keys to remove").EnableStdin(),
    },
    # 定义命令的选项
    Options: []cmds.Option{
        # 定义布尔类型的选项，用于显示关于键的额外信息
        cmds.BoolOption("l", "Show extra information about keys."),
        # 引用外部定义的选项
        ke.OptionIPNSBase,
    },
    # 定义命令的运行逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        # 从请求的选项中获取 IPNSBase，并转换为 KeyEncoder 对象
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))
        if err != nil {
            return err
        }

        # 从请求的参数中获取键的名称列表
        names := req.Arguments

        # 创建一个空的 KeyOutput 列表
        list := make([]KeyOutput, 0, len(names))
        # 遍历键的名称列表
        for _, name := range names {
            # 调用 API 对象的 Key 方法，删除指定名称的键
            key, err := api.Key().Remove(req.Context, name)
            if err != nil {
                return err
            }

            # 将删除的键的名称和格式化后的 ID 添加到列表中
            list = append(list, KeyOutput{
                Name: name,
                Id:   keyEnc.FormatID(key.ID()),
            })
        }

        # 将 KeyOutputList 列表通过 res 发送一次性的输出
        return cmds.EmitOnce(res, &KeyOutputList{list})
    },
    # 定义命令的编码器
    Encoders: cmds.EncoderMap{
        # 定义文本格式的编码器
        cmds.Text: keyOutputListEncoders(),
    },
    # 定义命令的类型
    Type: KeyOutputList{},
// 定义名为 keyRotateCmd 的命令对象
var keyRotateCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Rotates the IPFS identity.", // 命令简短描述
        ShortDescription: `
Generates a new ipfs identity and saves it to the ipfs config file.
Your existing identity key will be backed up in the Keystore.
The daemon must not be running when calling this command.

ipfs uses a repository in the local file system. By default, the repo is
located at ~/.ipfs. To change the repo location, set the $IPFS_PATH
environment variable:

    export IPFS_PATH=/path/to/ipfsrepo
`, // 命令详细描述
    },
    Arguments: []cmds.Argument{}, // 命令参数
    Options: []cmds.Option{ // 命令选项
        cmds.StringOption(oldKeyOptionName, "o", "Keystore name to use for backing up your existing identity"), // 用于备份现有身份的密钥库名称
        cmds.StringOption(keyStoreTypeOptionName, "t", "type of the key to create: rsa, ed25519").WithDefault(keyStoreAlgorithmDefault), // 要创建的密钥类型
        cmds.IntOption(keyStoreSizeOptionName, "s", "size of the key to generate"), // 要生成的密钥大小
    },
    NoRemote: true, // 不允许远程执行
    PreRun:   DaemonNotRunning, // 在运行之前检查守护程序是否正在运行
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 命令执行函数
        cctx := env.(*oldcmds.Context)
        nBitsForKeypair, nBitsGiven := req.Options[keyStoreSizeOptionName].(int) // 获取密钥对的位数
        algorithm, _ := req.Options[keyStoreTypeOptionName].(string) // 获取密钥类型
        oldKey, ok := req.Options[oldKeyOptionName].(string) // 获取备份旧密钥的密钥库名称
        if !ok {
            return fmt.Errorf("keystore name for backing up old key must be provided") // 如果未提供备份旧密钥的密钥库名称，则返回错误
        }
        if oldKey == "self" {
            return fmt.Errorf("keystore name for back up cannot be named 'self'") // 如果备份旧密钥的密钥库名称为 "self"，则返回错误
        }
        return doRotate(os.Stdout, cctx.ConfigRoot, oldKey, algorithm, nBitsForKeypair, nBitsGiven) // 执行密钥旋转操作
    },
}

// 执行密钥旋转操作
func doRotate(out io.Writer, repoRoot string, oldKey string, algorithm string, nBitsForKeypair int, nBitsGiven bool) error {
    // 打开存储库
    repo, err := fsrepo.Open(repoRoot)
    if err != nil {
        return fmt.Errorf("opening repo (%v)", err) // 如果打开存储库失败，则返回错误
    }
    defer repo.Close() // 延迟关闭存储库

    // 从存储库中读取配置文件
    // 从存储库中读取配置信息
    cfg, err := repo.Config()
    if err != nil {
        return fmt.Errorf("reading config from repo (%v)", err)
    }

    // 生成新的身份
    var identity config.Identity
    if nBitsGiven {
        // 如果给定了密钥位数，使用指定的选项创建身份
        identity, err = config.CreateIdentity(out, []options.KeyGenerateOption{
            options.Key.Size(nBitsForKeypair),
            options.Key.Type(algorithm),
        })
    } else {
        // 否则，使用指定的算法类型创建身份
        identity, err = config.CreateIdentity(out, []options.KeyGenerateOption{
            options.Key.Type(algorithm),
        })
    }
    if err != nil {
        return fmt.Errorf("creating identity (%v)", err)
    }

    // 将旧的身份保存到密钥库中
    oldPrivKey, err := cfg.Identity.DecodePrivateKey("")
    if err != nil {
        return fmt.Errorf("decoding old private key (%v)", err)
    }
    keystore := repo.Keystore()
    if err := keystore.Put(oldKey, oldPrivKey); err != nil {
        return fmt.Errorf("saving old key in keystore (%v)", err)
    }

    // 更新身份信息
    cfg.Identity = identity

    // 将配置文件写入存储库
    if err = repo.SetConfig(cfg); err != nil {
        return fmt.Errorf("saving new key to config (%v)", err)
    }
    return nil
}

// keyOutputListEncoders 返回一个编码器函数，用于将 KeyOutputList 编码为输出
func keyOutputListEncoders() cmds.EncoderFunc {
    return cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, list *KeyOutputList) error {
        // 获取请求中的选项，判断是否包含 ID
        withID, _ := req.Options["l"].(bool)

        // 创建一个 tabwriter，用于格式化输出
        tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
        // 遍历 KeyOutputList 中的 Keys
        for _, s := range list.Keys {
            // 根据是否包含 ID，格式化输出不同的内容
            if withID {
                fmt.Fprintf(tw, "%s\t%s\t\n", s.Id, cmdenv.EscNonPrint(s.Name))
            } else {
                fmt.Fprintf(tw, "%s\n", cmdenv.EscNonPrint(s.Name))
            }
        }
        // 刷新 tabwriter 缓冲区
        tw.Flush()
        return nil
    })
}

// KeySignOutput 包含一个 KeyOutput 和一个 Signature
type KeySignOutput struct {
    Key       KeyOutput
    Signature string
}

// keySignCmd 是一个命令对象，用于生成指定数据的签名
var keySignCmd = &cmds.Command{
    Status: cmds.Experimental,
    Helptext: cmds.HelpText{
        Tagline: "Generates a signature for the given data with a specified key. Useful for proving the key ownership.",
        LongDescription: `
Sign arbitrary bytes, such as to prove ownership of a Peer ID or an IPNS Name.
To avoid signature reuse, the signed payload is always prefixed with
"libp2p-key signed message:".
`,
    },
    Options: []cmds.Option{
        cmds.StringOption("key", "k", "The name of the key to use for signing."),
        ke.OptionIPNSBase,
    },
    Arguments: []cmds.Argument{
        cmds.FileArg("data", true, false, "The data to sign.").EnableStdin(),
    },
    # 运行函数，处理请求并返回响应
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        # 从请求选项中获取 IPNS 基础编码，并转换为密钥编码器
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string))
        if err != nil {
            return err
        }

        # 从请求选项中获取密钥名称
        name, _ := req.Options["key"].(string)

        # 从请求文件中获取文件对象
        file, err := cmdenv.GetFileArg(req.Files.Entries())
        if err != nil {
            return err
        }
        # 延迟关闭文件
        defer file.Close()

        # 读取文件数据
        data, err := io.ReadAll(file)
        if err != nil {
            return err
        }

        # 使用 API 对象对数据进行签名
        key, signature, err := api.Key().Sign(req.Context, name, data)
        if err != nil {
            return err
        }

        # 将签名进行 Base64url 编码
        encodedSignature, err := mbase.Encode(mbase.Base64url, signature)
        if err != nil {
            return err
        }

        # 返回签名结果
        return res.Emit(&KeySignOutput{
            Key: KeyOutput{
                Name: key.Name(),
                Id:   keyEnc.FormatID(key.ID()),
            },
            Signature: encodedSignature,
        })
    },
    # 指定返回类型为 KeySignOutput
    Type: KeySignOutput{},
// 定义 KeyVerifyOutput 结构体，包含 KeyOutput 和 SignatureValid 两个字段
type KeyVerifyOutput struct {
    Key            KeyOutput
    SignatureValid bool
}

// 定义 keyVerifyCmd 命令
var keyVerifyCmd = &cmds.Command{
    Status: cmds.Experimental, // 设置命令状态为实验性
    Helptext: cmds.HelpText{ // 设置命令的帮助文本
        Tagline: "Verify that the given data and signature match.", // 命令简介
        LongDescription: ` // 命令详细描述
Verify if the given data and signatures match. To avoid the signature reuse,
the signed payload is always prefixed with "libp2p-key signed message:".
`,
    },
    Options: []cmds.Option{ // 设置命令选项
        cmds.StringOption("key", "k", "The name of the key to use for signing."), // key 选项
        cmds.StringOption("signature", "s", "Multibase-encoded signature to verify."), // signature 选项
        ke.OptionIPNSBase, // IPNSBase 选项
    },
    Arguments: []cmds.Argument{ // 设置命令参数
        cmds.FileArg("data", true, false, "The data to verify against the given signature.").EnableStdin(), // data 参数
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 设置命令执行函数
        api, err := cmdenv.GetApi(env, req) // 获取 API
        if err != nil {
            return err
        }
        keyEnc, err := ke.KeyEncoderFromString(req.Options[ke.OptionIPNSBase.Name()].(string)) // 从选项中获取 keyEnc
        if err != nil {
            return err
        }

        name, _ := req.Options["key"].(string) // 从选项中获取 name
        encodedSignature, _ := req.Options["signature"].(string) // 从选项中获取 encodedSignature

        _, signature, err := mbase.Decode(encodedSignature) // 解码签名
        if err != nil {
            return err
        }

        file, err := cmdenv.GetFileArg(req.Files.Entries()) // 获取文件参数
        if err != nil {
            return err
        }
        defer file.Close() // 延迟关闭文件

        data, err := io.ReadAll(file) // 读取文件内容
        if err != nil {
            return err
        }

        key, valid, err := api.Key().Verify(req.Context, name, signature, data) // 验证签名
        if err != nil {
            return err
        }

        return res.Emit(&KeyVerifyOutput{ // 返回 KeyVerifyOutput 结果
            Key: KeyOutput{ // 设置 KeyOutput 字段
                Name: key.Name(), // 设置 Name 字段
                Id:   keyEnc.FormatID(key.ID()), // 设置 Id 字段
            },
            SignatureValid: valid, // 设置 SignatureValid 字段
        })
    },
    # 定义一个类型为KeyVerifyOutput的空字典
    Type: KeyVerifyOutput{},
// DaemonNotRunning函数检查IPFS存储库是否被锁定，表明守护程序正在运行，并在守护程序正在运行时返回错误。
func DaemonNotRunning(req *cmds.Request, env cmds.Environment) error {
    // 将环境转换为旧的命令上下文
    cctx := env.(*oldcmds.Context)
    // 检查IPFS存储库是否被其他进程锁定
    daemonLocked, err := fsrepo.LockedByOtherProcess(cctx.ConfigRoot)
    if err != nil {
        return err
    }

    // 输出日志信息，检查守护程序是否正在运行
    log.Info("checking if daemon is running...")
    // 如果守护程序被锁定，则输出调试信息并返回客户端错误
    if daemonLocked {
        log.Debug("ipfs daemon is running")
        e := "ipfs daemon is running. please stop it to run this command"
        return cmds.ClientError(e)
    }

    // 如果守护程序未运行，则返回nil
    return nil
}
```