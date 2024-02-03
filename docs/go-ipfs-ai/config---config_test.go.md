# `kubo\config\config_test.go`

```go
package config

import (
    "testing"
)

func TestClone(t *testing.T) {
    // 创建一个新的 Config 对象
    c := new(Config)
    // 设置新 Config 对象的 PeerID 属性为 "faketest"
    c.Identity.PeerID = "faketest"
    // 设置新 Config 对象的 HTTPHeaders 属性为包含一个键值对 "foo": {"bar"} 的 map

    newCfg, err := c.Clone()
    // 如果克隆操作出现错误，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }
    // 检查新克隆的 Config 对象的 PeerID 属性是否与原对象相同，如果不同则终止测试
    if newCfg.Identity.PeerID != c.Identity.PeerID {
        t.Fatal("peer ID not preserved")
    }

    // 修改原 Config 对象的 HTTPHeaders 属性
    c.API.HTTPHeaders["foo"] = []string{"baz"}
    // 检查新克隆的 Config 对象的 HTTPHeaders 属性是否与原对象相同，如果不同则终止测试
    if newCfg.API.HTTPHeaders["foo"][0] != "bar" {
        t.Fatal("HTTP headers not preserved")
    }

    // 删除原 Config 对象的 HTTPHeaders 属性中的 "foo" 键
    delete(c.API.HTTPHeaders, "foo")
    // 再次检查新克隆的 Config 对象的 HTTPHeaders 属性是否与原对象相同，如果不同则终止测试
    if newCfg.API.HTTPHeaders["foo"][0] != "bar" {
        t.Fatal("HTTP headers not preserved")
    }
}
```