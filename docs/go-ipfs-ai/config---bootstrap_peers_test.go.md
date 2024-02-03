# `kubo\config\bootstrap_peers_test.go`

```go
package config

import (
    "sort"  // 导入排序包
    "testing"  // 导入测试包
)

func TestBoostrapPeerStrings(t *testing.T) {
    parsed, err := ParseBootstrapPeers(DefaultBootstrapAddresses)  // 解析默认引导节点地址
    if err != nil {
        t.Fatal(err)  // 如果有错误，测试失败
    }

    formatted := BootstrapPeerStrings(parsed)  // 格式化解析后的引导节点地址
    sort.Strings(formatted)  // 对格式化后的地址进行排序
    expected := append([]string{}, DefaultBootstrapAddresses...)  // 复制默认引导节点地址
    sort.Strings(expected)  // 对复制的默认地址进行排序

    for i, s := range formatted {  // 遍历格式化后的地址
        if expected[i] != s {  // 如果排序后的地址与预期地址不一致
            t.Fatalf("expected %s, %s", expected[i], s)  // 测试失败，输出预期地址和实际地址
        }
    }
}
```