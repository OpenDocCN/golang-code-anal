# `grype\grype\db\db_closer.go`

```
# 导入名为 v5 的包，该包位于 github.com/anchore/grype/grype/db/v5
import v5 "github.com/anchore/grype/grype/db/v5"

# Closer 结构体定义，用于关闭数据库连接并释放任何分配的数据库资源
# 在某些周期性的 SBOM 扫描过程中，如果漏洞数据库被重复加载，这将特别有用
type Closer struct {
    # 继承自 v5 包中的 DBCloser 结构体
    v5.DBCloser
}
```