# `grype\grype\db\v4\advisory.go`

```go
// 定义一个结构体类型 Advisory，表示有关漏洞的发布声明（可能包括有关漏洞解决方案的声明）
type Advisory struct {
    ID   string `json:"id"`   // 定义结构体字段 ID，表示漏洞的唯一标识符，并使用 json 标签指定 JSON 序列化时的字段名为 "id"
    Link string `json:"link"` // 定义结构体字段 Link，表示漏洞的链接，并使用 json 标签指定 JSON 序列化时的字段名为 "link"
}
```