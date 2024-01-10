# `kubo\thirdparty\dir\dir.go`

```
// 在 dir 包中声明一个函数，用于确保目录存在并可写
func Writable(path string) error {
    // 如果路径不存在，则创建路径
    if err := os.MkdirAll(path, os.ModePerm); err != nil {
        return err
    }
    // 检查目录是否可写
    if f, err := os.Create(filepath.Join(path, "._check_writable")); err == nil {
        f.Close()
        os.Remove(f.Name())
    } else {
        return errors.New("'" + path + "' is not writable")
    }
    return nil
}
```