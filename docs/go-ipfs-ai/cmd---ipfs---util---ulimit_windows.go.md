# `kubo\cmd\ipfs\util\ulimit_windows.go`

```
// 如果操作系统是 Windows，则构建此代码块
// 设置构建标记为 windows
// 包名为 util
func init() {
    // 初始化函数，设置支持文件描述符管理为 false
    supportsFDManagement = false
}
```