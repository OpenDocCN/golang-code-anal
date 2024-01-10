# `v2ray-core\common\buf\copy_test.go`

```
package buf_test

import (
    "crypto/rand"  // 导入加密随机数生成包
    "io"  // 导入输入输出包
    "testing"  // 导入测试包

    "github.com/golang/mock/gomock"  // 导入 gomock 包

    "v2ray.com/core/common/buf"  // 导入 buf 包
    "v2ray.com/core/common/errors"  // 导入 errors 包
    "v2ray.com/core/testing/mocks"  // 导入 mocks 包
)

func TestReadError(t *testing.T) {
    mockCtl := gomock.NewController(t)  // 创建 mock 控制器
    defer mockCtl.Finish()  // 延迟关闭 mock 控制器

    mockReader := mocks.NewReader(mockCtl)  // 创建 mock 读取器
    mockReader.EXPECT().Read(gomock.Any()).Return(0, errors.New("error"))  // 期望 mock 读取器读取时返回错误

    err := buf.Copy(buf.NewReader(mockReader), buf.Discard)  // 使用 buf 包中的 Copy 函数进行读取
    if err == nil {
        t.Fatal("expected error, but nil")  // 如果没有错误，输出错误信息
    }

    if !buf.IsReadError(err) {
        t.Error("expected to be ReadError, but not")  // 如果不是读取错误，输出错误信息
    }

    if err.Error() != "error" {
        t.Fatal("unexpected error message: ", err.Error())  // 如果错误信息不符合预期，输出错误信息
    }
}

func TestWriteError(t *testing.T) {
    mockCtl := gomock.NewController(t)  // 创建 mock 控制器
    defer mockCtl.Finish()  // 延迟关闭 mock 控制器

    mockWriter := mocks.NewWriter(mockCtl)  // 创建 mock 写入器
    mockWriter.EXPECT().Write(gomock.Any()).Return(0, errors.New("error"))  // 期望 mock 写入器写入时返回错误

    err := buf.Copy(buf.NewReader(rand.Reader), buf.NewWriter(mockWriter))  // 使用 buf 包中的 Copy 函数进行写入
    if err == nil {
        t.Fatal("expected error, but nil")  // 如果没有错误，输出错误信息
    }

    if !buf.IsWriteError(err) {
        t.Error("expected to be WriteError, but not")  // 如果不是写入错误，输出错误信息
    }

    if err.Error() != "error" {
        t.Fatal("unexpected error message: ", err.Error())  // 如果错误信息不符合预期，输出错误信息
    }
}

type TestReader struct{}  // 定义测试读取器结构体

func (TestReader) Read(b []byte) (int, error) {
    return len(b), nil  // 测试读取器的读取方法，返回读取的字节数和无错误
}

func BenchmarkCopy(b *testing.B) {
    reader := buf.NewReader(io.LimitReader(TestReader{}, 10240))  // 创建读取器，限制读取大小为 10240
    writer := buf.Discard  // 创建写入器，丢弃写入的数据

    b.ResetTimer()  // 重置计时器
    for i := 0; i < b.N; i++ {
        _ = buf.Copy(reader, writer)  // 使用 buf 包中的 Copy 函数进行读取和写入
    }
}
```