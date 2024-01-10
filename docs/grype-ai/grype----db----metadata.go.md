# `grype\grype\db\metadata.go`

```
package db

import (
    "encoding/json" // 导入 JSON 编解码包
    "fmt" // 导入格式化包
    "os" // 导入操作系统功能包
    "path" // 导入路径操作包
    "time" // 导入时间包

    "github.com/spf13/afero" // 导入文件系统操作包

    "github.com/anchore/grype/internal/file" // 导入文件操作包
    "github.com/anchore/grype/internal/log" // 导入日志包
)

const MetadataFileName = "metadata.json" // 定义元数据文件名常量

// Metadata represents the basic identifying information of a database flat file (built/version) and a way to
// verify the contents (checksum).
type Metadata struct {
    Built    time.Time // 数据库构建时间
    Version  int // 数据库版本号
    Checksum string // 数据库内容校验和
}

// MetadataJSON is a helper struct for parsing and assembling Metadata objects to and from JSON.
type MetadataJSON struct {
    Built    string `json:"built"` // 数据库构建时间的 JSON 标签
    Version  int    `json:"version"` // 数据库版本号的 JSON 标签
    Checksum string `json:"checksum"` // 数据库内容校验和的 JSON 标签
}

// ToMetadata converts a MetadataJSON object to a Metadata object.
func (m MetadataJSON) ToMetadata() (Metadata, error) {
    build, err := time.Parse(time.RFC3339, m.Built) // 解析 RFC3339 格式的时间
    if err != nil {
        return Metadata{}, fmt.Errorf("cannot convert built time (%s): %+v", m.Built, err) // 返回错误信息
    }

    metadata := Metadata{
        Built:    build.UTC(), // 设置构建时间为 UTC 时间
        Version:  m.Version, // 设置版本号
        Checksum: m.Checksum, // 设置内容校验和
    }

    return metadata, nil // 返回 Metadata 对象
}

func metadataPath(dir string) string {
    return path.Join(dir, MetadataFileName) // 返回元数据文件路径
}

// NewMetadataFromDir generates a Metadata object from a directory containing a vulnerability.db flat file.
func NewMetadataFromDir(fs afero.Fs, dir string) (*Metadata, error) {
    metadataFilePath := metadataPath(dir) // 获取元数据文件路径
    exists, err := file.Exists(fs, metadataFilePath) // 检查文件是否存在
    if err != nil {
        return nil, fmt.Errorf("unable to check if DB metadata path exists (%s): %w", metadataFilePath, err) // 返回错误信息
    }
    if !exists {
        return nil, nil // 如果文件不存在则返回空
    }
    f, err := fs.Open(metadataFilePath) // 打开元数据文件
    if err != nil {
        return nil, fmt.Errorf("unable to open DB metadata path (%s): %w", metadataFilePath, err) // 返回错误信息
    }
    defer f.Close() // 延迟关闭文件

    var m Metadata // 定义 Metadata 对象
    err = json.NewDecoder(f).Decode(&m) // 解析 JSON 数据到 Metadata 对象
    # 如果 err 不为空，表示出现错误
    if err != nil:
        # 返回空值和格式化的错误信息，包括元数据文件路径和具体错误
        return nil, fmt.Errorf("unable to parse DB metadata (%s): %w", metadataFilePath, err)
    # 如果没有错误，返回解析后的元数据对象和空错误
    return &m, nil
// UnmarshalJSON 方法用于将 JSON 数据解析为 Metadata 对象
func (m *Metadata) UnmarshalJSON(data []byte) error {
    // 创建 MetadataJSON 对象
    var mj MetadataJSON
    // 解析 JSON 数据到 MetadataJSON 对象
    if err := json.Unmarshal(data, &mj); err != nil {
        return err
    }
    // 将 MetadataJSON 对象转换为 Metadata 对象
    me, err := mj.ToMetadata()
    if err != nil {
        return err
    }
    // 将转换后的 Metadata 对象赋值给当前对象
    *m = me
    return nil
}

// IsSupersededBy 方法用于判断给定的 ListingEntry 是否比当前 Metadata 对象中的版本更新
func (m *Metadata) IsSupersededBy(entry *ListingEntry) bool {
    // 如果当前 Metadata 为空，则无法比较，直接返回 true
    if m == nil {
        log.Debugf("cannot find existing metadata, using update...")
        return true
    }

    // 如果给定的 ListingEntry 的版本号大于当前 Metadata 的版本号，则返回 true
    if entry.Version > m.Version {
        log.Debugf("update is a newer version than the current database, using update...")
        return true
    }

    // 如果给定的 ListingEntry 的构建时间晚于当前 Metadata 的构建时间，则返回 true
    if entry.Built.After(m.Built) {
        log.Debugf("existing database (%s) is older than candidate update (%s), using update...", m.Built.String(), entry.Built.String())
        return true
    }

    // 如果以上条件都不满足，则返回 false
    log.Debugf("existing database is already up to date")
    return false
}

// String 方法用于返回 Metadata 对象的字符串表示
func (m Metadata) String() string {
    return fmt.Sprintf("Metadata(built=%s version=%d checksum=%s)", m.Built, m.Version, m.Checksum)
}

// Write 方法用于将 Metadata 对象写入指定路径的文件
func (m Metadata) Write(toPath string) error {
    // 创建 MetadataJSON 对象
    metadata := MetadataJSON{
        Built:    m.Built.UTC().Format(time.RFC3339),
        Version:  m.Version,
        Checksum: m.Checksum,
    }

    // 将 MetadataJSON 对象编码为 JSON 格式的数据
    contents, err := json.MarshalIndent(&metadata, "", " ")
    if err != nil {
        return fmt.Errorf("failed to encode metadata file: %w", err)
    }

    // 将编码后的数据写入指定路径的文件
    err = os.WriteFile(toPath, contents, 0600)
    if err != nil {
        return fmt.Errorf("failed to write metadata file: %w", err)
    }
    return nil
}
```