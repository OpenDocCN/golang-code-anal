# `v2ray-core\common\uuid\uuid_test.go`

```
package uuid_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"  // 导入用于比较的包

    "v2ray.com/core/common"  // 导入通用包
    . "v2ray.com/core/common/uuid"  // 导入 UUID 包，使用 . 表示在后续代码中可以直接使用包里的函数和变量，而不需要添加包名前缀
)

func TestParseBytes(t *testing.T) {
    str := "2418d087-648d-4990-86e8-19dca1d006d3"  // 定义字符串
    bytes := []byte{0x24, 0x18, 0xd0, 0x87, 0x64, 0x8d, 0x49, 0x90, 0x86, 0xe8, 0x19, 0xdc, 0xa1, 0xd0, 0x06, 0xd3}  // 定义字节切片

    uuid, err := ParseBytes(bytes)  // 使用 ParseBytes 函数解析字节切片为 UUID 对象
    common.Must(err)  // 检查错误，如果有错误则终止程序
    if diff := cmp.Diff(uuid.String(), str); diff != "" {  // 比较解析后的 UUID 字符串与预期字符串是否相同
        t.Error(diff)  // 如果不同则输出差异
    }

    _, err = ParseBytes([]byte{1, 3, 2, 4})  // 使用错误的字节切片解析 UUID 对象
    if err == nil {  // 如果没有错误
        t.Fatal("Expect error but nil")  // 输出错误信息
    }
}

func TestParseString(t *testing.T) {
    str := "2418d087-648d-4990-86e8-19dca1d006d3"  // 定义字符串
    expectedBytes := []byte{0x24, 0x18, 0xd0, 0x87, 0x64, 0x8d, 0x49, 0x90, 0x86, 0xe8, 0x19, 0xdc, 0xa1, 0xd0, 0x06, 0xd3}  // 定义预期的字节切片

    uuid, err := ParseString(str)  // 使用 ParseString 函数解析字符串为 UUID 对象
    common.Must(err)  // 检查错误，如果有错误则终止程序
    if r := cmp.Diff(expectedBytes, uuid.Bytes()); r != "" {  // 比较解析后的 UUID 字节切片与预期字节切片是否相同
        t.Fatal(r)  // 如果不同则输出错误信息
    }

    _, err = ParseString("2418d087")  // 使用错误的字符串解析 UUID 对象
    if err == nil {  // 如果没有错误
        t.Fatal("Expect error but nil")  // 输出错误信息
    }

    _, err = ParseString("2418d087-648k-4990-86e8-19dca1d006d3")  // 使用错误的字符串解析 UUID 对象
    if err == nil {  // 如果没有错误
        t.Fatal("Expect error but nil")  // 输出错误信息
    }
}

func TestNewUUID(t *testing.T) {
    uuid := New()  // 创建新的 UUID 对象
    uuid2, err := ParseString(uuid.String())  // 使用 UUID 对象的字符串形式解析为新的 UUID 对象

    common.Must(err)  // 检查错误，如果有错误则终止程序
    if uuid.String() != uuid2.String() {  // 比较两个 UUID 对象的字符串形式是否相同
        t.Error("uuid string: ", uuid.String(), " != ", uuid2.String())  // 如果不同则输出错误信息
    }
    if r := cmp.Diff(uuid.Bytes(), uuid2.Bytes()); r != "" {  // 比较两个 UUID 对象的字节切片是否相同
        t.Error(r)  // 如果不同则输出错误信息
    }
}

func TestRandom(t *testing.T) {
    uuid := New()  // 创建新的 UUID 对象
    uuid2 := New()  // 创建新的 UUID 对象

    if uuid.String() == uuid2.String() {  // 比较两个 UUID 对象的字符串形式是否相同
        t.Error("duplicated uuid")  // 如果相同则输出错误信息
    }
}

func TestEquals(t *testing.T) {
    var uuid *UUID  // 声明 UUID 对象指针
    var uuid2 *UUID  // 声明 UUID 对象指针
    if !uuid.Equals(uuid2) {  // 比较两个 UUID 对象是否相同
        t.Error("empty uuid should equal")  // 如果不同则输出错误信息
    }

    uuid3 := New()  // 创建新的 UUID 对象
    if uuid.Equals(&uuid3) {  // 比较两个 UUID 对象是否相同
        t.Error("nil uuid equals non-nil uuid")  // 如果相同则输出错误信息
    }
}
```