# `v2ray-core\annotations.go`

```
// Annotation 是 V2Ray 中的一个概念。这个结构体仅用于文档目的，不会在任何地方被使用。
// 注解以 "v2ray:" 开头，作为函数或类型的元数据。
type Annotation struct {
    // API 用于可以在其他库中使用的类型或函数。可能的取值有：
    //
    // * v2ray:api:beta 用于准备好可以使用的类型或函数，但可能在将来发生变化。
    // * v2ray:api:stable 用于具有向后兼容性保证的类型或函数。
    // * v2ray:api:deprecated 用于不应再使用的类型或函数。
    //
    // 没有 api 注解的类型或函数不应该在外部使用。
    API string
}
```