# `grype\grype\pkg\java_metadata.go`

```
# 定义了一个名为JavaMetadata的结构体，用于存储Java元数据信息
type JavaMetadata struct {
    # 虚拟路径
    VirtualPath    string   `json:"virtualPath"`
    # POM文件的ArtifactID
    PomArtifactID  string   `json:"pomArtifactID"`
    # POM文件的GroupID
    PomGroupID     string   `json:"pomGroupID"`
    # MANIFEST文件的名称
    ManifestName   string   `json:"manifestName"`
    # 存储Digest结构体的切片，用于存储归档文件的摘要信息
    ArchiveDigests []Digest `json:"archiveDigests"`
}

# 定义了一个名为Digest的结构体，用于存储摘要信息
type Digest struct {
    # 摘要算法
    Algorithm string `json:"algorithm"`
    # 摘要值
    Value     string `json:"value"`
}
```