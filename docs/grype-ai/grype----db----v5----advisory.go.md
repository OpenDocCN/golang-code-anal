# `grype\grype\db\v5\advisory.go`

```
package v5
// 定义一个名为 v5 的包

// Advisory 表示关于漏洞的已发布声明（可能是关于漏洞解决方案的声明）。
type Advisory struct {
    ID   string `json:"id"` // 漏洞的唯一标识符
    Link string `json:"link"` // 漏洞相关链接
}
```