# `kubo\core\node\libp2p\fd\sys_not_unix.go`

```
// 根据构建标记判断当前操作系统是否为 Linux、Darwin 或 Windows
// 如果不是以上三种操作系统，则执行以下代码
package fd

// 获取当前进程的文件描述符数量
func GetNumFDs() int {
    // 返回文件描述符数量为 0
    return 0
}
```