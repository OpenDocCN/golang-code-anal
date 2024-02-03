# `trojan-go\config\config_test.go`

```go
package config

import (
    "context"
    "testing"

    "github.com/p4gefau1t/trojan-go/common"
)

type Foo struct {
    Field1 string `json,yaml:"field1"`  // 定义结构体字段 Field1，用于 JSON 和 YAML 标签
    Field2 bool   `json:"field2" yaml:"field2"`  // 定义结构体字段 Field2，用于 JSON 和 YAML 标签
}

type TestStruct struct {
    Field1 string `json,yaml:"field1"`  // 定义结构体字段 Field1，用于 JSON 和 YAML 标签
    Field2 bool   `json,yaml:"field2"`  // 定义结构体字段 Field2，用于 JSON 和 YAML 标签
    Field3 []Foo  `json,yaml:"field3"`  // 定义结构体字段 Field3，用于 JSON 和 YAML 标签
}

func creator() interface{} {
    return &TestStruct{}  // 返回 TestStruct 结构体的指针
}

func TestJSONConfig(t *testing.T) {
    RegisterConfigCreator("test", creator)  // 注册配置创建函数
    data := []byte(`
    {
        "field1": "test1",
        "field2": true,
        "field3": [
            {
                "field1": "aaaa",
                "field2": true
            }
        ]
    }
    `)  // 定义 JSON 格式的配置数据
    ctx, err := WithJSONConfig(context.Background(), data)  // 使用 JSON 格式的配置数据创建上下文
    common.Must(err)  // 检查错误
    c := FromContext(ctx, "test").(*TestStruct)  // 从上下文中获取配置并转换为 TestStruct 结构体
    if c.Field1 != "test1" || c.Field2 != true {  // 检查配置字段值是否符合预期
        t.Fail()  // 标记测试失败
    }
}

func TestYAMLConfig(t *testing.T) {
    RegisterConfigCreator("test", creator)  // 注册配置创建函数
    data := []byte(`
field1: 012345678
field2: true
field3:
  - field1: test
    field2: true
`)  // 定义 YAML 格式的配置数据
    ctx, err := WithYAMLConfig(context.Background(), data)  // 使用 YAML 格式的配置数据创建上下文
    common.Must(err)  // 检查错误
    c := FromContext(ctx, "test").(*TestStruct)  // 从上下文中获取配置并转换为 TestStruct 结构体
    if c.Field1 != "012345678" || c.Field2 != true || c.Field3[0].Field1 != "test" {  // 检查配置字段值是否符合预期
        t.Fail()  // 标记测试失败
    }
}
```