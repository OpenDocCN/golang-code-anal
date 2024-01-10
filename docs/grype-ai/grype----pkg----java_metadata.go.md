# `grype\grype\pkg\java_metadata.go`

```
# 定义了一个名为 JavaMetadata 的结构体，用于存储 Java 元数据信息
type JavaMetadata struct {
    VirtualPath    string   `json:"virtualPath"`  # 虚拟路径，用于存储文件的路径信息
    PomArtifactID  string   `json:"pomArtifactID"`  # POM 项目的 ArtifactID
    PomGroupID     string   `json:"pomGroupID"`  # POM 项目的 GroupID
    ManifestName   string   `json:"manifestName"`  # Manifest 文件的名称
    ArchiveDigests []Digest `json:"archiveDigests"`  # 存储 Digest 结构体的切片，用于存储归档文件的摘要信息
}

# 定义了一个名为 Digest 的结构体，用于存储摘要信息
type Digest struct {
    Algorithm string `json:"algorithm"`  # 摘要算法
    Value     string `json:"value"`  # 摘要值
}
```