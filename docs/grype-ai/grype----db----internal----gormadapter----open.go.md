# `grype\grype\db\internal\gormadapter\open.go`

```
package gormadapter

import (
    "fmt"
    "os"

    "github.com/glebarez/sqlite"
    "gorm.io/gorm"
)

// 定义写入数据库时的性能优化语句
var writerStatements = []string{
    // 性能优化（注意：会导致写入中断时数据丢失）
    // 在我的电脑上，将写入时间从10分钟减少到10秒（内存利用率约为1GB）
    `PRAGMA synchronous = OFF`,
    `PRAGMA journal_mode = MEMORY`,
}

// 定义读取数据库时的选项
var readOptions = []string{
    "immutable=1",
    "cache=shared",
    "mode=ro",
}

// 打开一个新的连接到 sqlite3 数据库文件
func Open(path string, write bool) (*gorm.DB, error) {
    if write {
        // 文件可能存在也可能不存在，所以我们明确忽略错误
        _ = os.Remove(path)
    }

    // 创建连接字符串
    connStr, err := connectionString(path)
    if err != nil {
        return nil, err
    }

    if !write {
        // &immutable=1&cache=shared&mode=ro
        for _, o := range readOptions {
            connStr += fmt.Sprintf("&%s", o)
        }
    }

    // 打开数据库连接
    dbObj, err := gorm.Open(sqlite.Open(connStr), &gorm.Config{Logger: newLogger()})
    if err != nil {
        return nil, fmt.Errorf("unable to connect to DB: %w", err)
    }

    if write {
        // 执行写入优化语句
        for _, sqlStmt := range writerStatements {
            dbObj.Exec(sqlStmt)
            if dbObj.Error != nil {
                return nil, fmt.Errorf("unable to execute (%s): %w", sqlStmt, dbObj.Error)
            }
        }
    }

    return dbObj, nil
}

// 创建 sqlite3 的连接字符串
func connectionString(path string) (string, error) {
    if path == "" {
        return "", fmt.Errorf("no db filepath given")
    }
    return fmt.Sprintf("file:%s?cache=shared", path), nil
}
```