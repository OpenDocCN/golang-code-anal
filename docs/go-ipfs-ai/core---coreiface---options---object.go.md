# `kubo\core\coreiface\options\object.go`

```go
package options

// 定义创建对象时的设置结构体
type ObjectNewSettings struct {
    Type string
}

// 定义放置对象时的设置结构体
type ObjectPutSettings struct {
    InputEnc string
    DataType string
    Pin      bool
}

// 定义添加链接时的设置结构体
type ObjectAddLinkSettings struct {
    Create bool
}

// 定义创建对象时的选项函数类型
type (
    ObjectNewOption     func(*ObjectNewSettings) error
    ObjectPutOption     func(*ObjectPutSettings) error
    ObjectAddLinkOption func(*ObjectAddLinkSettings) error
)

// 创建对象时的选项函数，根据传入的选项设置对象的属性
func ObjectNewOptions(opts ...ObjectNewOption) (*ObjectNewSettings, error) {
    options := &ObjectNewSettings{
        Type: "empty",
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 放置对象时的选项函数，根据传入的选项设置对象的属性
func ObjectPutOptions(opts ...ObjectPutOption) (*ObjectPutSettings, error) {
    options := &ObjectPutSettings{
        InputEnc: "json",
        DataType: "text",
        Pin:      false,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 添加链接时的选项函数，根据传入的选项设置对象的属性
func ObjectAddLinkOptions(opts ...ObjectAddLinkOption) (*ObjectAddLinkSettings, error) {
    options := &ObjectAddLinkSettings{
        Create: false,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }
    return options, nil
}

// 对象选项结构体
type objectOpts struct{}

// 对象选项结构体实例
var Object objectOpts

// Type 是 Object.New 的选项，允许更改创建的 dag 节点的类型
func (objectOpts) Type(t string) ObjectNewOption {
    return func(settings *ObjectNewSettings) error {
        settings.Type = t
        return nil
    }
}

// InputEnc 是 Object.Put 的选项，指定数据的输入编码，默认为 "json"
func (objectOpts) InputEnc(e string) ObjectPutOption {
    // TODO: 添加对输入编码的设置
}
    # 返回一个函数，该函数接受一个ObjectPutSettings类型的参数，并返回一个error类型的结果
    return func(settings *ObjectPutSettings) error {
        # 将参数settings的InputEnc字段设置为e
        settings.InputEnc = e
        # 返回空值
        return nil
    }
// DataType 是 Object.Put 的一个选项，用于指定在使用 Json 或 XML 输入编码时数据字段的编码方式
//
// 支持的类型:
// * "text" (默认)
// * "base64"
func (objectOpts) DataType(t string) ObjectPutOption {
    return func(settings *ObjectPutSettings) error {
        settings.DataType = t
        return nil
    }
}

// Pin 是 Object.Put 的一个选项，用于指定是否固定添加的对象，默认为 false
func (objectOpts) Pin(pin bool) ObjectPutOption {
    return func(settings *ObjectPutSettings) error {
        settings.Pin = pin
        return nil
    }
}

// Create 是 Object.AddLink 的一个选项，用于指定是否为子对象创建所需的目录
func (objectOpts) Create(create bool) ObjectAddLinkOption {
    return func(settings *ObjectAddLinkSettings) error {
        settings.Create = create
        return nil
    }
}
```