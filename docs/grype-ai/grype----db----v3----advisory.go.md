# `grype\grype\db\v3\advisory.go`

```
package v3
// 声明一个名为 v3 的包

// Advisory 表示关于漏洞的已发布声明（可能是关于漏洞解决方案的声明）。
type Advisory struct {
    ID   string  // 漏洞的唯一标识符
    Link string  // 漏洞相关链接
}
```