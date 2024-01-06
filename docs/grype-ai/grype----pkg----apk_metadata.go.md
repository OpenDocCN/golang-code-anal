# `grype\grype\pkg\apk_metadata.go`

```
// 定义一个名为 ApkMetadata 的结构体，用于存储 APK 文件的元数据
type ApkMetadata struct {
    // Files 字段是一个 ApkFileRecord 类型的切片，用于存储 APK 文件的记录
    Files []ApkFileRecord `json:"files"`
}

// ApkFileRecord 结构体表示 APK 数据库条目中的单个文件列表和元数据（可能有多个文件记录）
type ApkFileRecord struct {
    // Path 字段存储文件的路径
    Path string `json:"path"`
}
```