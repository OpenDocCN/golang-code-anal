# `grype\grype\db\internal\gormadapter\logger.go`

```
// 导入所需的包
package gormadapter

import (
	"context" // 导入上下文包
	"time" // 导入时间包

	"gorm.io/gorm/logger" // 导入gorm日志包

	"github.com/anchore/grype/internal/log" // 导入内部日志包
)

// 定义日志适配器结构体
type logAdapter struct {
}

// 创建新的日志记录器
func newLogger() logger.Interface {
	return logAdapter{} // 返回日志适配器实例
}

// 设置日志级别
func (l logAdapter) LogMode(logger.LogLevel) logger.Interface {
	return l // 返回日志适配器实例
}
// Info方法，用于记录信息日志，但未实现具体功能
func (l logAdapter) Info(_ context.Context, _ string, _ ...interface{}) {
    // unimplemented
}

// Warn方法，用于记录警告日志，调用log.Warnf输出日志
func (l logAdapter) Warn(_ context.Context, fmt string, v ...interface{}) {
    log.Warnf("gorm: "+fmt, v...)
}

// Error方法，用于记录错误日志，调用log.Errorf输出日志
func (l logAdapter) Error(_ context.Context, fmt string, v ...interface{}) {
    log.Errorf("gorm: "+fmt, v...)
}

// Trace方法，用于记录跟踪日志，但未实现具体功能
func (l logAdapter) Trace(_ context.Context, _ time.Time, _ func() (sql string, rowsAffected int64), _ error) {
    // unimplemented
}
```