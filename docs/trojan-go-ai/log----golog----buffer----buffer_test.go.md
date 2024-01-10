# `trojan-go\log\golog\buffer\buffer_test.go`

```
// Buffer-like byte slice
// Copyright (c) 2017 Fadhli Dzil Ikram
//
// Test file for buffer

// 导入 buffer 包
package buffer

// 导入测试包
import (
    "testing"

    . "github.com/smartystreets/goconvey/convey"
)

// 测试 BufferAllocation 函数
func TestBufferAllocation(t *testing.T) {
    // 给定未分配的 buffer
    Convey("Given new unallocated buffer", t, func() {
        var buf Buffer

        // 当追加数据时
        Convey("When appended with data", func() {
            data := []byte("Hello")
            buf.Append(data)

            // 应该具有与原始数据相同的内容
            Convey("It should have same content as the original data", func() {
                So(buf.Bytes(), ShouldResemble, data)
            })
        })

        // 当追加单个字节时
        Convey("When appended with single byte", func() {
            data := byte('H')
            buf.AppendByte(data)

            // 应该具有 1 个字节的长度
            Convey("It should have 1 byte length", func() {
                So(len(buf), ShouldEqual, 1)
            })

            // 应该具有相同的内容
            Convey("It should have same content", func() {
                So(buf.Bytes()[0], ShouldEqual, data)
            })
        })

        // 当追加整数时
        Convey("When appended with integer", func() {
            data := 12345
            repr := []byte("012345")
            buf.AppendInt(data, len(repr))

            // 应该具有与整数表示相同的内容
            Convey("Should have same content with the integer representation", func() {
                So(buf.Bytes(), ShouldResemble, repr)
            })
        })
    })
}

// 测试 BufferReset 函数
func TestBufferReset(t *testing.T) {
    # 定义测试用例，给定已分配的缓冲区
    Convey("Given allocated buffer", t, func() {
        # 声明一个缓冲区变量
        var buf Buffer
        # 定义数据和替换数据
        data := []byte("Hello")
        replace := []byte("World")
        # 将数据追加到缓冲区
        buf.Append(data)

        # 当缓冲区重置时
        Convey("When buffer reset", func() {
            # 重置缓冲区
            buf.Reset()

            # 应该长度为零
            Convey("It should have zero length", func() {
                So(len(buf), ShouldEqual, 0)
            })
        })

        # 当缓冲区重置并替换为另一个追加时
        Convey("When buffer reset and replaced with another append", func() {
            # 重置缓冲区
            buf.Reset()
            # 追加替换数据
            buf.Append(replace)

            # 应该与替换数据具有相同的内容
            Convey("It should have same content with the replaced data", func() {
                So(buf.Bytes(), ShouldResemble, replace)
            })
        })
    })
# 闭合前面的函数定义
```