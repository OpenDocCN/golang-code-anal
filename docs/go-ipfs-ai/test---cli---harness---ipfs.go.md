# `kubo\test\cli\harness\ipfs.go`

```
package harness

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "reflect"  // 导入反射包
    "strings"  // 导入字符串处理包

    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
)

func (n *Node) IPFSCommands() []string {
    res := n.IPFS("commands").Stdout.String()  // 调用 IPFS 命令获取命令列表，并将结果转换为字符串
    res = strings.TrimSpace(res)  // 去除字符串两端的空白字符
    split := SplitLines(res)  // 将字符串按行分割
    var cmds []string  // 声明字符串切片
    for _, line := range split {  // 遍历分割后的字符串切片
        trimmed := strings.TrimSpace(line)  // 去除每行字符串两端的空白字符
        if trimmed == "ipfs" {  // 如果去除空白字符后的字符串为 "ipfs"
            continue  // 跳过当前循环
        }
        cmds = append(cmds, trimmed)  // 将去除空白字符后的字符串添加到 cmds 切片中
    }
    return cmds  // 返回命令列表
}

func (n *Node) SetIPFSConfig(key string, val interface{}, flags ...string) {
    valBytes, err := json.Marshal(val)  // 将 val 转换为 JSON 格式的字节流
    if err != nil {  // 如果转换过程中出现错误
        log.Panicf("marshling config for key '%s': %s", key, err)  // 输出错误信息并终止程序
    }
    valStr := string(valBytes)  // 将 JSON 格式的字节流转换为字符串

    args := []string{"config", "--json"}  // 声明字符串切片并初始化
    args = append(args, flags...)  // 将 flags 切片的元素添加到 args 切片中
    args = append(args, key, valStr)  // 将 key 和 valStr 添加到 args 切片中
    n.IPFS(args...)  // 调用 IPFS 命令

    // validate the config was set correctly

    // Create a new value which is a pointer to the same type as the source.
    var newVal any  // 声明变量 newVal
    if val != nil {  // 如果 val 不为 nil
        // If it is not nil grab the type with reflect.
        newVal = reflect.New(reflect.TypeOf(val)).Interface()  // 使用反射获取与 val 相同类型的指针
    } else {  // 如果 val 为 nil
        // else just set a pointer to an any.
        var anything any  // 声明变量 anything
        newVal = &anything  // 将 anything 的指针赋值给 newVal
    }
    n.GetIPFSConfig(key, newVal)  // 调用 GetIPFSConfig 方法获取配置信息
    // dereference newVal using reflect to load the resulting value
    if !reflect.DeepEqual(val, reflect.ValueOf(newVal).Elem().Interface()) {  // 如果 val 与 newVal 的值不相等
        log.Panicf("key '%s' did not retain value '%s' after it was set, got '%s'", key, val, newVal)  // 输出错误信息并终止程序
    }
}

func (n *Node) GetIPFSConfig(key string, val interface{}) {
    res := n.IPFS("config", key)  // 调用 IPFS 命令获取指定配置信息
    valStr := strings.TrimSpace(res.Stdout.String())  // 将结果转换为字符串并去除两端空白字符
    // only when the result is a string is the result not well-formed JSON,
    // so check the value type and add quotes if it's expected to be a string
    reflectVal := reflect.ValueOf(val)  // 使用反射获取 val 的值
}
    # 检查反射值的类型是否为指针，并且指针指向的值的类型是否为字符串
    if reflectVal.Kind() == reflect.Ptr && reflectVal.Elem().Kind() == reflect.String {
        # 如果是指针指向的值的类型为字符串，则将其格式化为带双引号的字符串
        valStr = fmt.Sprintf(`"%s"`, valStr)
    }
    # 使用 JSON 解组将字符串转换为对应类型的值
    err := json.Unmarshal([]byte(valStr), val)
    # 如果解组过程中出现错误，则记录错误信息并终止程序
    if err != nil {
        log.Fatalf("unmarshaling config for key '%s', value '%s': %s", key, valStr, err)
    }
# 定义一个方法，用于向 IPFS 添加字符串类型的内容
func (n *Node) IPFSAddStr(content string, args ...string) string {
    # 记录日志，表示节点正在添加内容，并打印节点 ID、内容预览和参数
    log.Debugf("node %d adding content '%s' with args: %v", n.ID, PreviewStr(content), args)
    # 调用 IPFSAdd 方法，传入字符串内容和参数，并返回结果
    return n.IPFSAdd(strings.NewReader(content), args...)
}

# 定义一个方法，用于向 IPFS 添加内容
func (n *Node) IPFSAdd(content io.Reader, args ...string) string {
    # 记录日志，表示节点正在添加内容，并打印节点 ID 和参数
    log.Debugf("node %d adding with args: %v", n.ID, args)
    # 构建完整的参数列表，包括默认参数和传入的参数
    fullArgs := []string{"add", "-q"}
    fullArgs = append(fullArgs, args...)
    # 调用 Runner.MustRun 方法执行 IPFS 命令，并获取结果
    res := n.Runner.MustRun(RunRequest{
        Path:    n.IPFSBin,
        Args:    fullArgs,
        CmdOpts: []CmdOpt{RunWithStdin(content)},
    })
    # 获取命令执行结果的标准输出，并去除首尾空白字符
    out := strings.TrimSpace(res.Stdout.String())
    # 记录日志，表示添加结果
    log.Debugf("add result: %q", out)
    # 返回命令执行结果
    return out
}

# 定义一个方法，用于向 IPFS 导入 DAG 数据
func (n *Node) IPFSDagImport(content io.Reader, cid string, args ...string) error {
    # 记录日志，表示节点正在进行 DAG 导入，并打印节点 ID 和参数
    log.Debugf("node %d dag import with args: %v", n.ID, args)
    # 构建完整的参数列表，包括默认参数和传入的参数
    fullArgs := []string{"dag", "import", "--pin-roots=false"}
    fullArgs = append(fullArgs, args...)
    # 调用 Runner.MustRun 方法执行 IPFS 命令，并获取结果
    res := n.Runner.MustRun(RunRequest{
        Path:    n.IPFSBin,
        Args:    fullArgs,
        CmdOpts: []CmdOpt{RunWithStdin(content)},
    })
    # 如果执行结果中存在错误，则返回该错误
    if res.Err != nil {
        return res.Err
    }
    # 再次调用 Runner.MustRun 方法执行 IPFS 命令，检查导入的数据块状态，并返回结果
    res = n.Runner.MustRun(RunRequest{
        Path: n.IPFSBin,
        Args: []string{"block", "stat", "--offline", cid},
    })
    return res.Err
}
```