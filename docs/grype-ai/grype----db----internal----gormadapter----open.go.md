# `grype\grype\db\internal\gormadapter\open.go`

```
package gormadapter

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"os"   // 导入 os 包，用于操作系统功能
	"github.com/glebarez/sqlite"  // 导入 sqlite 包
	"gorm.io/gorm"  // 导入 gorm 包
)

var writerStatements = []string{  // 定义字符串数组 writerStatements
	// 性能改进（注意：会导致写入中断时丢失数据）。
	// 在我的电脑上，它将写入时间从 10 分钟减少到 10 秒（内存利用率约为 1GB）
	`PRAGMA synchronous = OFF`,  // 设置 PRAGMA 同步为 OFF
	`PRAGMA journal_mode = MEMORY`,  // 设置 PRAGMA 日志模式为 MEMORY
}

var readOptions = []string{  // 定义字符串数组 readOptions
	"immutable=1",  // 设置不可变性为 1
	"cache=shared",  // 设置缓存为共享模式
```
// 定义了只读模式的连接选项
readOptions := []string{
	"immutable=1",
	"cache=shared",
	"mode=ro",
}

// 打开一个新的连接到 sqlite3 数据库文件
func Open(path string, write bool) (*gorm.DB, error) {
	if write {
		// 如果需要写入，先尝试删除已存在的文件，忽略可能的错误
		_ = os.Remove(path)
	}

	// 生成数据库连接字符串
	connStr, err := connectionString(path)
	if err != nil {
		return nil, err
	}

	if !write {
		// 如果是只读模式，添加只读模式的连接选项
		for _, o := range readOptions {
			connStr += fmt.Sprintf("&%s", o)
		}
	}

	// 使用给定的连接字符串打开一个 SQLite 数据库连接
	dbObj, err := gorm.Open(sqlite.Open(connStr), &gorm.Config{Logger: newLogger()})
	if err != nil {
		// 如果连接失败，返回错误信息
		return nil, fmt.Errorf("unable to connect to DB: %w", err)
	}

	// 如果需要写入数据库
	if write {
		// 遍历写入语句列表，逐条执行
		for _, sqlStmt := range writerStatements {
			dbObj.Exec(sqlStmt)
			// 如果执行出错，返回错误信息
			if dbObj.Error != nil {
				return nil, fmt.Errorf("unable to execute (%s): %w", sqlStmt, dbObj.Error)
			}
		}
	}

	// 返回数据库连接对象
	return dbObj, nil
}

// ConnectionString creates a connection string for sqlite3
# 定义一个函数，接收一个字符串类型的参数 path
func connectionString(path string) (string, error) {
    # 如果路径为空，则返回一个错误，说明没有给定数据库文件路径
    if path == "" {
        return "", fmt.Errorf("no db filepath given")
    }
    # 格式化字符串，返回一个数据库连接字符串，使用给定的文件路径
    return fmt.Sprintf("file:%s?cache=shared", path), nil
}
```