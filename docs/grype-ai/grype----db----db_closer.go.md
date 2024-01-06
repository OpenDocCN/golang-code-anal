# `grype\grype\db\db_closer.go`

```
// 导入名为 v5 的包，该包位于 github.com/anchore/grype/grype/db/v5
import v5 "github.com/anchore/grype/grype/db/v5"

// Closer 结构体用于关闭数据库连接并释放任何分配的数据库资源
// 如果在某个周期性的 SBOM 扫描过程中需要重复加载漏洞数据库，则这个结构体尤其有用
type Closer struct {
    // 继承自 v5 包中的 DBCloser 结构体
    v5.DBCloser
}
```