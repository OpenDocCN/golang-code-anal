# `kubo\core\node\libp2p\fd\sys_windows.go`

```
// 根据构建标记判断是否在 Windows 平台下编译
// 包名为 fd
// 导入 math 包
func GetNumFDs() int {
    // 返回最大整数值，表示文件描述符的数量
    return math.MaxInt
}
```