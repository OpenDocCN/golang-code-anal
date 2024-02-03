# `kubo\core\commands\completion.go`

```go
package commands

import (
    "io"  // 导入io包，用于输入输出操作
    "sort"  // 导入sort包，用于排序
    "text/template"  // 导入text/template包，用于文本模板操作

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入自定义包cmds
)

type completionCommand struct {
    Name         string  // 定义命令名称字段
    FullName     string  // 定义完整命令名称字段
    Description  string  // 定义描述字段
    Subcommands  []*completionCommand  // 定义子命令列表字段
    Flags        []*singleOption  // 定义标志列表字段
    Options      []*singleOption  // 定义选项列表字段
    ShortFlags   []string  // 定义短标志列表字段
    ShortOptions []string  // 定义短选项列表字段
    LongFlags    []string  // 定义长标志列表字段
    LongOptions  []string  // 定义长选项列表字段
    IsFinal      bool  // 定义是否为最终命令字段
}

type singleOption struct {
    LongNames   []string  // 定义长名称列表字段
    ShortNames  []string  // 定义短名称列表字段
    Description string  // 定义描述字段
}

func commandToCompletions(name string, fullName string, cmd *cmds.Command) *completionCommand {
    parsed := &completionCommand{  // 创建命令解析对象
        Name:        name,  // 设置命令名称
        FullName:    fullName,  // 设置完整命令名称
        Description: cmd.Helptext.Tagline,  // 设置描述
        IsFinal:     len(cmd.Subcommands) == 0,  // 判断是否为最终命令
    }
    for name, subCmd := range cmd.Subcommands {  // 遍历子命令
        parsed.Subcommands = append(parsed.Subcommands,  // 将子命令解析对象添加到子命令列表中
            commandToCompletions(name, fullName+" "+name, subCmd))  // 递归调用命令解析函数
    }
    sort.Slice(parsed.Subcommands, func(i, j int) bool {  // 对子命令列表进行排序
        return parsed.Subcommands[i].Name < parsed.Subcommands[j].Name  // 按名称升序排序
    })
}
    # 遍历命令的选项列表
    for _, opt := range cmd.Options:
        # 创建单个选项对象，设置描述信息
        flag := &singleOption{Description: opt.Description()}
        # 将选项的名称添加到长选项名称列表中
        flag.LongNames = append(flag.LongNames, opt.Name())
        # 如果选项类型为布尔类型
        if opt.Type() == cmds.Bool:
            # 将选项名称添加到长选项列表中
            parsed.LongFlags = append(parsed.LongFlags, opt.Name())
            # 遍历选项的名称列表
            for _, name := range opt.Names():
                # 如果名称长度为1，则将其添加到短选项列表中
                if len(name) == 1:
                    parsed.ShortFlags = append(parsed.ShortFlags, name)
                    # 将名称添加到单个选项的短名称列表中
                    flag.ShortNames = append(flag.ShortNames, name)
                    break
            # 将单个选项添加到选项列表中
            parsed.Flags = append(parsed.Flags, flag)
        else:
            # 将选项名称添加到长选项列表中
            parsed.LongOptions = append(parsed.LongOptions, opt.Name())
            # 遍历选项的名称列表
            for _, name := range opt.Names():
                # 如果名称长度为1，则将其添加到短选项列表中
                if len(name) == 1:
                    parsed.ShortOptions = append(parsed.ShortOptions, name)
                    # 将名称添加到单个选项的短名称列表中
                    flag.ShortNames = append(flag.ShortNames, name)
                    break
            # 将单个选项添加到选项列表中
            parsed.Options = append(parsed.Options, flag)
    # 对长选项列表进行排序
    sort.Slice(parsed.LongFlags, func(i, j int) bool {
        return parsed.LongFlags[i] < parsed.LongFlags[j]
    })
    # 对短选项列表进行排序
    sort.Slice(parsed.ShortFlags, func(i, j int) bool {
        return parsed.ShortFlags[i] < parsed.ShortFlags[j]
    })
    # 对长选项列表进行排序
    sort.Slice(parsed.LongOptions, func(i, j int) bool {
        return parsed.LongOptions[i] < parsed.LongOptions[j]
    })
    # 对短选项列表进行排序
    sort.Slice(parsed.ShortOptions, func(i, j int) bool {
        return parsed.ShortOptions[i] < parsed.ShortOptions[j]
    })
    # 返回解析后的选项对象
    return parsed
}
// 定义全局变量，用于存储模板
var bashCompletionTemplate, fishCompletionTemplate, zshCompletionTemplate *template.Template

// 初始化函数
func init() {
    // 定义命令模板
    commandTemplate := template.Must(template.New("command").Parse(`
while [[ ${index} -lt ${COMP_CWORD} ]]; do
    case "${COMP_WORDS[index]}" in
        -*)
        let index++
            continue
        ;;
    {{ range .Subcommands }}
    "{{ .Name }}")
        let index++
        {{ template "command" . }}
        return 0
            ;;
    {{ end }}
    esac
    break
done

if [[ "${word}" == -* ]]; then
{{ if .ShortFlags -}}
    _ipfs_compgen -W $'{{ range .ShortFlags }}-{{.}} \n{{end}}' -- "${word}"
{{ end -}}
{{- if .ShortOptions -}}
    _ipfs_compgen -S = -W $'{{ range .ShortOptions }}-{{.}}\n{{end}}' -- "${word}"
{{ end -}}
{{- if .LongFlags -}}
    _ipfs_compgen -W $'{{ range .LongFlags }}--{{.}} \n{{end}}' -- "${word}"
{{ end -}}
{{- if .LongOptions -}}
    _ipfs_compgen -S = -W $'{{ range .LongOptions }}--{{.}}\n{{end}}' -- "${word}"
{{ end -}}
    return 0
fi

while [[ ${index} -lt ${COMP_CWORD} ]]; do
    if [[ "${COMP_WORDS[index]}" != -* ]]; then
        let argidx++
    fi
    let index++
done

{{- if .Subcommands }}
if [[ "${argidx}" -eq 0 ]]; then
    _ipfs_compgen -W $'{{ range .Subcommands }}{{.Name}} \n{{end}}' -- "${word}"
fi
{{ end -}}
`))

    // 将命令模板赋值给 bashCompletionTemplate
    bashCompletionTemplate = template.Must(commandTemplate.New("root").Parse(`#!/bin/bash

_ipfs_compgen() {
  local oldifs="$IFS"
  IFS=$'\n'
  while read -r line; do
    COMPREPLY+=("$line")
  done < <(compgen "$@")
  IFS="$oldifs"
}

_ipfs() {
  COMPREPLY=()
  local index=1
  local argidx=0
  local word="${COMP_WORDS[COMP_CWORD]}"
  {{ template "command" . }}
}
complete -o nosort -o nospace -o default -F _ipfs ipfs
`))

    // 将命令模板赋值给 zshCompletionTemplate
    zshCompletionTemplate = template.Must(commandTemplate.New("root").Parse(`#!bin/zsh
autoload bashcompinit
bashcompinit
_ipfs_compgen() {
local oldifs="$IFS"
IFS=$'\n'
while read -r line; do
    COMPREPLY+=("$line")
done < <(compgen "$@")
IFS="$oldifs"
}

_ipfs() {
# 初始化自动补全回复数组
COMPREPLY=()
# 初始化索引
local index=1
# 初始化参数索引
local argidx=0
# 获取当前输入的单词
local word="${COMP_WORDS[COMP_CWORD]}"
# 使用模板生成命令
{{ template "command" . }}
# 结束函数定义
}
# 设置 ipfs 命令的自动补全
complete -o nosort -o nospace -o default -F _ipfs ipfs
# 定义 fishCommandTemplate 模板
`))

fishCommandTemplate := template.Must(template.New("command").Parse(`
{{- if .IsFinal -}}
# 设置 ipfs 命令的子命令自动补全
complete -c ipfs -n '__fish_ipfs_seen_all_subcommands_from{{ .FullName }}' -F
{{ end -}}
{{- range .Flags -}}
    # 设置 ipfs 命令的选项自动补全
    complete -c ipfs -n '__fish_ipfs_seen_all_subcommands_from{{ $.FullName }}' {{ range .ShortNames }}-s {{.}} {{end}}{{ range .LongNames }}-l {{.}} {{end}}-d "{{ .Description }}"
{{ end -}}
{{- range .Options -}}
    # 设置 ipfs 命令的参数自动补全
    complete -c ipfs -n '__fish_ipfs_seen_all_subcommands_from{{ $.FullName }}' -r {{ range .ShortNames }}-s {{.}} {{end}}{{ range .LongNames }}-l {{.}} {{end}}-d "{{ .Description }}"
{{ end -}}

{{- range .Subcommands }}
#{{ .FullName }}
# 设置 ipfs 命令的子命令自动补全
complete -c ipfs -n '__fish_ipfs_use_subcommand{{ .FullName }}' -a {{ .Name }} -d "{{ .Description }}"
{{ template "command" . }}
{{ end -}}
    `))
# 定义 fishCompletionTemplate 模板
fishCompletionTemplate = template.Must(fishCommandTemplate.New("root").Parse(`#!/usr/bin/env fish
# 定义函数，检查是否已经输入了所有子命令
function __fish_ipfs_seen_all_subcommands_from
     set -l cmd (commandline -poc)
     set -e cmd[1]
     for c in $argv
         if not contains -- $c $cmd
               return 1
        end
     end
     return 0
end

# 定义函数，检查是否使用了指定的子命令
function __fish_ipfs_use_subcommand
    set -e argv[-1]
    set -l cmd (commandline -poc)
    set -e cmd[1]
    for i in $cmd
        switch $i
            case '-*'
                continue
            case $argv[1]
                set argv $argv[2..]
                continue
            case '*'
                return 1
        end
    end
    test -z "$argv"
end

# 设置 ipfs 命令的帮助选项自动补全
complete -c ipfs -l help -d "Show the full command help text."

# 设置 ipfs 命令的自动补全
complete -c ipfs --keep-order --no-files

{{ template "command" . }}
`))
}

# 生成给定命令树的 bash 自动补全脚本
func writeBashCompletions(cmd *cmds.Command, out io.Writer) error {
    # 调用函数 commandToCompletions，传入参数 "ipfs", "", cmd，并将返回值赋给变量 cmds
    cmds := commandToCompletions("ipfs", "", cmd)
    # 调用 bashCompletionTemplate 的 Execute 方法，传入参数 out 和 cmds，并将结果返回
    return bashCompletionTemplate.Execute(out, cmds)
// writeFishCompletions 为给定的命令树生成 fish 自动补全脚本
func writeFishCompletions(cmd *cmds.Command, out io.Writer) error {
    // 将命令转换为自动补全命令
    cmds := commandToCompletions("ipfs", "", cmd)
    // 使用 fish 自动补全模板将命令写入输出流
    return fishCompletionTemplate.Execute(out, cmds)
}

// writeZshCompletions 为给定的命令树生成 zsh 自动补全脚本
func writeZshCompletions(cmd *cmds.Command, out io.Writer) error {
    // 将命令转换为自动补全命令
    cmds := commandToCompletions("ipfs", "", cmd)
    // 使用 zsh 自动补全模板将命令写入输出流
    return zshCompletionTemplate.Execute(out, cmds)
}
```