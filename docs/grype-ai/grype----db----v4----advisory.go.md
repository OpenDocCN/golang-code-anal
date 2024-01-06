# `grype\grype\db\v4\advisory.go`

```
package v4
// 定义一个名为 v4 的包

// Advisory 表示关于漏洞的已发布声明（可能是关于漏洞解决方案的声明）。
type Advisory struct {
    ID   string `json:"id"`  // 表示漏洞的唯一标识符
    Link string `json:"link"`  // 表示漏洞相关链接
}
```