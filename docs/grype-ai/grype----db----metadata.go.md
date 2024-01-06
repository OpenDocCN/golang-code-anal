# `grype\grype\db\metadata.go`

```
// 导入必要的包
package db

import (
	"encoding/json"  // 导入 JSON 编解码包
	"fmt"  // 导入格式化输出包
	"os"  // 导入操作系统相关功能包
	"path"  // 导入路径操作包
	"time"  // 导入时间相关功能包

	"github.com/spf13/afero"  // 导入文件系统操作包

	"github.com/anchore/grype/internal/file"  // 导入文件操作相关包
	"github.com/anchore/grype/internal/log"  // 导入日志记录包
)

const MetadataFileName = "metadata.json"  // 定义常量 MetadataFileName 为 "metadata.json"

// Metadata 表示数据库平面文件的基本标识信息（构建/版本）以及验证内容（校验和）的方式
type Metadata struct {
// Metadata 结构体用于存储元数据信息，包括构建时间、版本和校验和
type Metadata struct {
    Built    time.Time
    Version  int
    Checksum string
}

// MetadataJSON 是一个辅助结构体，用于解析和组装元数据对象到和从 JSON 格式
type MetadataJSON struct {
    Built    string `json:"built"` // RFC 3339，用于指定时间格式
    Version  int    `json:"version"`
    Checksum string `json:"checksum"`
}

// ToMetadata 将 MetadataJSON 对象转换为 Metadata 对象
func (m MetadataJSON) ToMetadata() (Metadata, error) {
    // 解析时间字符串为 time.Time 对象
    build, err := time.Parse(time.RFC3339, m.Built)
    if err != nil {
        return Metadata{}, fmt.Errorf("cannot convert built time (%s): %+v", m.Built, err)
    }

    // 创建 Metadata 对象并返回
    metadata := Metadata{
		// 构建时间戳
		Built:    build.UTC(),
		// 版本号
		Version:  m.Version,
		// 校验和
		Checksum: m.Checksum,
	}

	// 返回元数据对象和空错误
	return metadata, nil
}

// 返回元数据文件路径
func metadataPath(dir string) string {
	return path.Join(dir, MetadataFileName)
}

// 从包含 vulnerability.db 文件的目录生成 Metadata 对象
func NewMetadataFromDir(fs afero.Fs, dir string) (*Metadata, error) {
	// 获取元数据文件路径
	metadataFilePath := metadataPath(dir)
	// 检查元数据文件是否存在
	exists, err := file.Exists(fs, metadataFilePath)
	if err != nil {
		// 如果出错，返回错误信息
		return nil, fmt.Errorf("unable to check if DB metadata path exists (%s): %w", metadataFilePath, err)
	}
	// 如果文件不存在
	if !exists {
		// 返回空值和空值
		return nil, nil
	}
	// 打开元数据文件
	f, err := fs.Open(metadataFilePath)
	if err != nil {
		// 如果打开失败，返回错误信息
		return nil, fmt.Errorf("unable to open DB metadata path (%s): %w", metadataFilePath, err)
	}
	// 延迟关闭文件
	defer f.Close()

	// 创建元数据变量
	var m Metadata
	// 解析元数据文件
	err = json.NewDecoder(f).Decode(&m)
	if err != nil {
		// 如果解析失败，返回错误信息
		return nil, fmt.Errorf("unable to parse DB metadata (%s): %w", metadataFilePath, err)
	}
	// 返回解析后的元数据
	return &m, nil
}

func (m *Metadata) UnmarshalJSON(data []byte) error {
	// 创建元数据 JSON 变量
	var mj MetadataJSON
	// 解析 JSON 数据到元数据 JSON 变量
	if err := json.Unmarshal(data, &mj); err != nil {
		// 如果解析失败，返回错误信息
		return err
	}
	// 将元数据对象转换为元数据结构
	me, err := mj.ToMetadata()
	// 如果转换出错，返回错误
	if err != nil {
		return err
	}
	// 将转换后的元数据赋值给当前对象
	*m = me
	// 返回空值
	return nil
}

// IsSupersededBy 接收一个 ListingEntry 对象，并确定候选项是否比当前元数据对象中暗示的更新版本更新。
func (m *Metadata) IsSupersededBy(entry *ListingEntry) bool {
	// 如果当前元数据对象为空，记录日志并返回 true
	if m == nil {
		log.Debugf("cannot find existing metadata, using update...")
		// 任何有效的更新都比没有数据库要好，使用它！
		return true
	}

	// 如果候选项的版本号大于当前元数据对象的版本号，记录日志并返回 true
	if entry.Version > m.Version {
		log.Debugf("update is a newer version than the current database, using update...")
// 如果列表更新时间比现有数据库更新时间新，则返回 true
return true
}

// 如果条目的构建时间晚于数据库的构建时间，则记录日志并返回 true
if entry.Built.After(m.Built) {
    log.Debugf("现有数据库（%s）比候选更新（%s）旧，使用更新...", m.Built.String(), entry.Built.String())
    // 列表更新时间比现有数据库更新时间新，则返回 true
    return true
}

// 记录日志并返回 false，表示现有数据库已经是最新的
log.Debugf("现有数据库已经是最新的")
return false
}

// 将 Metadata 对象转换为字符串
func (m Metadata) String() string {
    return fmt.Sprintf("Metadata(built=%s version=%d checksum=%s)", m.Built, m.Version, m.Checksum)
}

// 将 Metadata 对象写入指定路径
func (m Metadata) Write(toPath string) error {
# 创建一个 MetadataJSON 结构体，并初始化其字段
metadata := MetadataJSON{
    Built:    m.Built.UTC().Format(time.RFC3339),  # 将时间转换为 RFC3339 格式并赋值给 Built 字段
    Version:  m.Version,  # 将版本号赋值给 Version 字段
    Checksum: m.Checksum,  # 将校验和赋值给 Checksum 字段
}

# 将 metadata 结构体转换为格式化的 JSON 字节流
contents, err := json.MarshalIndent(&metadata, "", " ")
if err != nil {
    return fmt.Errorf("failed to encode metadata file: %w", err)  # 如果转换失败，则返回错误
}

# 将 JSON 内容写入到指定路径的文件中
err = os.WriteFile(toPath, contents, 0600)
if err != nil {
    return fmt.Errorf("failed to write metadata file: %w", err)  # 如果写入失败，则返回错误
}
# 返回空值，表示操作成功
return nil
```