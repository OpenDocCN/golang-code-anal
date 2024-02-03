# `grype\grype\db\listing.go`

```go
package db

import (
    "encoding/json"  // 导入处理 JSON 数据的包
    "fmt"  // 导入格式化输出的包
    "os"  // 导入操作系统功能的包
    "sort"  // 导入排序功能的包

    "github.com/spf13/afero"  // 导入文件系统抽象层的包
)

const ListingFileName = "listing.json"  // 定义常量，表示文件名为 listing.json

// Listing represents the json file which is served up and made available for applications to download and
// consume one or more vulnerability db flat files.
type Listing struct {
    Available map[int][]ListingEntry `json:"available"`  // 定义 Listing 结构体，包含一个 available 字段，用于存储 ListingEntry 数据
}

// NewListing creates a listing from one or more given ListingEntries.
func NewListing(entries ...ListingEntry) Listing {
    listing := Listing{  // 创建 Listing 结构体
        Available: make(map[int][]ListingEntry),  // 初始化 Available 字段
    }
    for _, entry := range entries {  // 遍历传入的 ListingEntry
        if _, ok := listing.Available[entry.Version]; !ok {  // 如果版本号不存在
            listing.Available[entry.Version] = make([]ListingEntry, 0)  // 创建对应版本号的空列表
        }
        listing.Available[entry.Version] = append(listing.Available[entry.Version], entry)  // 将 entry 添加到对应版本号的列表中
    }

    // sort each entry descending by date
    for idx := range listing.Available {  // 遍历 Available 字段
        listingEntries := listing.Available[idx]  // 获取对应版本号的列表
        sort.SliceStable(listingEntries, func(i, j int) bool {  // 对列表进行排序
            return listingEntries[i].Built.After(listingEntries[j].Built)  // 按照日期降序排序
        })
    }

    return listing  // 返回创建的 Listing
}

// NewListingFromFile loads a Listing from a given filepath.
func NewListingFromFile(fs afero.Fs, path string) (Listing, error) {
    f, err := fs.Open(path)  // 打开指定路径的文件
    if err != nil {
        return Listing{}, fmt.Errorf("unable to open DB listing path: %w", err)  // 如果打开文件失败，返回错误信息
    }
    defer f.Close()  // 延迟关闭文件

    var l Listing  // 声明一个 Listing 变量
    err = json.NewDecoder(f).Decode(&l)  // 解析文件内容到 Listing 变量
    if err != nil {
        return Listing{}, fmt.Errorf("unable to parse DB listing: %w", err)  // 如果解析失败，返回错误信息
    }

    // sort each entry descending by date
    for idx := range l.Available {  // 遍历 Available 字段
        listingEntries := l.Available[idx]  // 获取对应版本号的列表
        sort.SliceStable(listingEntries, func(i, j int) bool {  // 对列表进行排序
            return listingEntries[i].Built.After(listingEntries[j].Built)  // 按照日期降序排序
        })
    }

    return l, nil  // 返回加载的 Listing
}
// BestUpdate 根据给定的版本约束条件，从列表中返回符合条件的 ListingEntry
func (l *Listing) BestUpdate(targetSchema int) *ListingEntry {
    // 检查是否存在目标版本的列表条目
    if listingEntries, ok := l.Available[targetSchema]; ok {
        // 如果存在，则返回第一个符合条件的条目
        if len(listingEntries) > 0 {
            return &listingEntries[0]
        }
    }
    // 如果不存在符合条件的条目，则返回空
    return nil
}

// Write 将当前列表写入指定的文件路径
func (l Listing) Write(toPath string) error {
    // 将当前列表内容进行格式化并编码为 JSON 格式
    contents, err := json.MarshalIndent(&l, "", " ")
    if err != nil {
        // 如果编码失败，则返回错误信息
        return fmt.Errorf("failed to encode listing file: %w", err)
    }

    // 将编码后的内容写入指定的文件路径
    err = os.WriteFile(toPath, contents, 0600)
    if err != nil {
        // 如果写入失败，则返回错误信息
        return fmt.Errorf("failed to write listing file: %w", err)
    }
    // 如果写入成功，则返回空
    return nil
}
```