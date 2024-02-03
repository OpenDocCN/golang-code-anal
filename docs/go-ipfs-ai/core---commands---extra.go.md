# `kubo\core\commands\extra.go`

```go
package commands

import cmds "github.com/ipfs/go-ipfs-cmds"

// CreateCmdExtras 创建命令的额外信息
func CreateCmdExtras(opts ...func(e *cmds.Extra)) *cmds.Extra {
    e := new(cmds.Extra)
    for _, o := range opts {
        o(e)
    }
    return e
}

// doesNotUseRepo 表示不使用存储库的类型
type doesNotUseRepo struct{}

// SetDoesNotUseRepo 设置不使用存储库的标志
func SetDoesNotUseRepo(val bool) func(e *cmds.Extra) {
    return func(e *cmds.Extra) {
        e.SetValue(doesNotUseRepo{}, val)
    }
}

// GetDoesNotUseRepo 获取不使用存储库的标志
func GetDoesNotUseRepo(e *cmds.Extra) (val bool, found bool) {
    return getBoolFlag(e, doesNotUseRepo{})
}

// doesNotUseConfigAsInput 描述不使用配置作为输入的命令
type doesNotUseConfigAsInput struct{}

// SetDoesNotUseConfigAsInput 设置不使用配置作为输入的标志
func SetDoesNotUseConfigAsInput(val bool) func(e *cmds.Extra) {
    return func(e *cmds.Extra) {
        e.SetValue(doesNotUseConfigAsInput{}, val)
    }
}

// GetDoesNotUseConfigAsInput 获取不使用配置作为输入的标志
func GetDoesNotUseConfigAsInput(e *cmds.Extra) (val bool, found bool) {
    return getBoolFlag(e, doesNotUseConfigAsInput{})
}

// preemptsAutoUpdate 描述必须在不触发自动更新的情况下执行的命令
type preemptsAutoUpdate struct{}

// SetPreemptsAutoUpdate 设置不触发自动更新的标志
func SetPreemptsAutoUpdate(val bool) func(e *cmds.Extra) {
    return func(e *cmds.Extra) {
        e.SetValue(preemptsAutoUpdate{}, val)
    }
}

// GetPreemptsAutoUpdate 获取不触发自动更新的标志
func GetPreemptsAutoUpdate(e *cmds.Extra) (val bool, found bool) {
    return getBoolFlag(e, preemptsAutoUpdate{})
}

// getBoolFlag 获取布尔标志
func getBoolFlag(e *cmds.Extra, key interface{}) (val bool, found bool) {
    var ival interface{}
    ival, found = e.GetValue(key)
    if !found {
        return false, false
    }
    val = ival.(bool)
    return val, found
}
```