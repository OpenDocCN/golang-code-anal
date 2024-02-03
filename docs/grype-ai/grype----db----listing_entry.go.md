# `grype\grype\db\listing_entry.go`

```go
package db

import (
    "crypto/sha256" // 导入用于计算 SHA256 哈希的包
    "encoding/json" // 导入用于 JSON 编码和解码的包
    "fmt" // 导入格式化输出包
    "net/url" // 导入处理 URL 的包
    "path" // 导入处理文件路径的包
    "path/filepath" // 导入处理文件路径的包
    "time" // 导入处理时间的包

    "github.com/spf13/afero" // 导入文件系统操作的包

    "github.com/anchore/grype/internal/file" // 导入文件操作相关的包
)

// ListingEntry represents basic metadata about a database archive such as what is in the archive (built/version)
// as well as how to obtain and verify the archive (URL/checksum).
type ListingEntry struct {
    Built    time.Time // 数据库归档的构建时间，使用 RFC 3339 格式
    Version  int // 数据库归档的版本号
    URL      *url.URL // 数据库归档的 URL 地址
    Checksum string // 数据库归档的校验和
}

// ListingEntryJSON is a helper struct for converting a ListingEntry into JSON (or parsing from JSON)
type ListingEntryJSON struct {
    Built    string `json:"built"` // 数据库归档的构建时间，JSON 格式
    Version  int    `json:"version"` // 数据库归档的版本号，JSON 格式
    URL      string `json:"url"` // 数据库归档的 URL 地址，JSON 格式
    Checksum string `json:"checksum"` // 数据库归档的校验和，JSON 格式
}

// NewListingEntryFromArchive creates a new ListingEntry based on the metadata from a database flat file.
func NewListingEntryFromArchive(fs afero.Fs, metadata Metadata, dbArchivePath string, baseURL *url.URL) (ListingEntry, error) {
    checksum, err := file.HashFile(fs, dbArchivePath, sha256.New()) // 计算数据库归档文件的 SHA256 校验和
    if err != nil {
        return ListingEntry{}, fmt.Errorf("unable to find db archive checksum: %w", err) // 如果计算校验和出错，则返回错误
    }

    dbArchiveName := filepath.Base(dbArchivePath) // 获取数据库归档文件的文件名
    fileURL, _ := url.Parse(baseURL.String()) // 解析基础 URL
    fileURL.Path = path.Join(fileURL.Path, dbArchiveName) // 拼接数据库归档文件的 URL 地址

    return ListingEntry{
        Built:    metadata.Built, // 使用元数据中的构建时间
        Version:  metadata.Version, // 使用元数据中的版本号
        URL:      fileURL, // 使用拼接好的 URL 地址
        Checksum: "sha256:" + checksum, // 使用计算得到的 SHA256 校验和
    }, nil
}

// ToListingEntry converts a ListingEntryJSON to a ListingEntry.
func (l ListingEntryJSON) ToListingEntry() (ListingEntry, error) {
    build, err := time.Parse(time.RFC3339, l.Built) // 解析 JSON 中的构建时间
    if err != nil {
        return ListingEntry{}, fmt.Errorf("cannot convert built time (%s): %+v", l.Built, err) // 如果解析出错，则返回错误
    }

    u, err := url.Parse(l.URL) // 解析 JSON 中的 URL 地址
    if err != nil {
        return ListingEntry{}, fmt.Errorf("cannot parse url (%s): %+v", l.URL, err) // 如果解析出错，则返回错误
    }
}
    # 返回一个ListingEntry对象，包含构建时间、版本、URL和校验和
    return ListingEntry{
        Built:    build.UTC(),  # 设置构建时间为当前时间
        Version:  l.Version,    # 设置版本为l的版本
        URL:      u,            # 设置URL为u
        Checksum: l.Checksum,   # 设置校验和为l的校验和
    }, nil  # 返回nil作为错误信息
# 反序列化 JSON 数据到 ListingEntry 结构体
func (l *ListingEntry) UnmarshalJSON(data []byte) error {
    # 创建 ListingEntryJSON 变量
    var lej ListingEntryJSON
    # 将 JSON 数据解析到 ListingEntryJSON 变量中
    if err := json.Unmarshal(data, &lej); err != nil {
        return err
    }
    # 将 ListingEntryJSON 转换为 ListingEntry
    le, err := lej.ToListingEntry()
    if err != nil {
        return err
    }
    # 将转换后的 ListingEntry 赋值给当前对象
    *l = le
    return nil
}

# 将 ListingEntry 对象序列化为 JSON 数据
func (l *ListingEntry) MarshalJSON() ([]byte, error) {
    return json.Marshal(&ListingEntryJSON{
        Built:    l.Built.Format(time.RFC3339),
        Version:  l.Version,
        Checksum: l.Checksum,
        URL:      l.URL.String(),
    })
}

# 返回 ListingEntry 对象的字符串表示
func (l ListingEntry) String() string {
    return fmt.Sprintf("Listing(url=%s)", l.URL)
}
```