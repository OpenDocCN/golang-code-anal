# `grype\grype\pkg\apk_metadata.go`

```
// 定义一个名为pkg的包

// 定义一个结构体ApkMetadata，包含一个名为Files的ApkFileRecord切片，使用json标签指定序列化时的字段名为files
type ApkMetadata struct {
    Files []ApkFileRecord `json:"files"`
}

// 定义一个结构体ApkFileRecord，表示APK数据库条目中的单个文件列表和元数据（可能有多个文件记录），包含一个名为Path的字符串字段，使用json标签指定序列化时的字段名为path
type ApkFileRecord struct {
    Path string `json:"path"`
}
```