# `grype\grype\db\listing.go`

```
package db

import (
	"encoding/json"  // 导入 JSON 编解码包
	"fmt"  // 导入格式化包
	"os"  // 导入操作系统功能包
	"sort"  // 导入排序功能包

	"github.com/spf13/afero"  // 导入文件系统操作包
)

const ListingFileName = "listing.json"  // 定义常量 ListingFileName 为 "listing.json"

// Listing represents the json file which is served up and made available for applications to download and
// consume one or more vulnerability db flat files.
type Listing struct {
	Available map[int][]ListingEntry `json:"available"`  // 定义 Listing 结构体，包含一个 map 类型的 Available 字段，用于存储 ListingEntry 结构体的数组
}

// NewListing creates a listing from one or more given ListingEntries.
# 创建一个新的列表对象，包含给定的列表条目
func NewListing(entries ...ListingEntry) Listing {
    # 创建一个空的可用条目的映射
    listing := Listing{
        Available: make(map[int][]ListingEntry),
    }
    # 遍历给定的条目列表
    for _, entry := range entries {
        # 如果给定版本的条目不存在，则创建一个空的条目列表
        if _, ok := listing.Available[entry.Version]; !ok {
            listing.Available[entry.Version] = make([]ListingEntry, 0)
        }
        # 将条目添加到对应版本的条目列表中
        listing.Available[entry.Version] = append(listing.Available[entry.Version], entry)
    }

    # 对每个版本的条目列表按照日期降序排序
    for idx := range listing.Available {
        listingEntries := listing.Available[idx]
        sort.SliceStable(listingEntries, func(i, j int) bool {
            return listingEntries[i].Built.After(listingEntries[j].Built)
        })
    }

    # 返回新的列表对象
    return listing
}
}

// 从给定的文件路径加载一个 Listing 对象
func NewListingFromFile(fs afero.Fs, path string) (Listing, error) {
    // 使用给定的文件系统打开文件
    f, err := fs.Open(path)
    if err != nil {
        return Listing{}, fmt.Errorf("unable to open DB listing path: %w", err)
    }
    // 在函数返回之前关闭文件
    defer f.Close()

    // 创建一个空的 Listing 对象
    var l Listing
    // 从文件中解析 JSON 数据到 Listing 对象
    err = json.NewDecoder(f).Decode(&l)
    if err != nil {
        return Listing{}, fmt.Errorf("unable to parse DB listing: %w", err)
    }

    // 对每个条目按日期降序排序
    for idx := range l.Available {
        listingEntries := l.Available[idx]
        // 使用稳定排序算法对条目进行排序
        sort.SliceStable(listingEntries, func(i, j int) bool {
// 按照建筑日期对列表条目进行排序
return listingEntries[i].Built.After(listingEntries[j].Built)
})

// 返回满足给定版本约束的列表中的ListingEntry
func (l *Listing) BestUpdate(targetSchema int) *ListingEntry {
// 检查是否存在目标模式的可用列表条目
if listingEntries, ok := l.Available[targetSchema]; ok {
// 如果存在可用条目，则返回第一个条目
if len(listingEntries) > 0 {
return &listingEntries[0]
}
}
// 如果没有可用条目，则返回nil
return nil
}

// 将当前列表写入给定的文件路径
func (l Listing) Write(toPath string) error {
// 将当前列表内容转换为格式化的JSON字符串
contents, err := json.MarshalIndent(&l, "", " ")
# 如果发生错误，则返回一个带有错误信息的错误对象
if err != nil:
    return fmt.Errorf("failed to encode listing file: %w", err)

# 将内容写入指定路径的文件中，设置文件权限为 0600
err = os.WriteFile(toPath, contents, 0600)
if err != nil:
    return fmt.Errorf("failed to write listing file: %w", err)

# 如果没有发生错误，则返回空值
return nil
```