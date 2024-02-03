# `v2ray-core\infra\conf\json\reader_test.go`

```go
package json_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "io"  // 导入 io 包，用于进行 I/O 操作
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 cmp 包，用于比较数据

    "v2ray.com/core/common"  // 导入 common 包
    . "v2ray.com/core/infra/conf/json"  // 导入 json 配置包
)

func TestReader(t *testing.T) {
    data := []struct {  // 定义结构体切片
        input  string  // 输入字符串
        output string  // 期望输出字符串
    }{
        {
            `
content #comment 1
#comment 2
content 2`,  // 输入字符串
            `
content 

content 2`},  // 期望输出字符串
        {`content`, `content`},  // 输入字符串和期望输出字符串
        {" ", " "},  // 输入字符串和期望输出字符串
        {`con/*abcd*/tent`, "content"},  // 输入字符串和期望输出字符串
        {`
text // adlkhdf /*
//comment adfkj
text 2*/`, `
text 

text 2*`},  // 输入字符串和期望输出字符串
        {`"//"content`, `"//"content`},  // 输入字符串和期望输出字符串
        {`abcd'//'abcd`, `abcd'//'abcd`},  // 输入字符串和期望输出字符串
        {`"\""`, `"\""`},  // 输入字符串和期望输出字符串
        {`\"/*abcd*/\"`, `\"\"`},  // 输入字符串和期望输出字符串
    }

    for _, testCase := range data {  // 遍历测试数据
        reader := &Reader{  // 创建 Reader 对象
            Reader: bytes.NewReader([]byte(testCase.input)),  // 使用输入字符串创建字节流读取器
        }

        actual := make([]byte, 1024)  // 创建长度为 1024 的字节数组
        n, err := reader.Read(actual)  // 从 Reader 中读取数据到字节数组中
        common.Must(err)  // 检查错误
        if r := cmp.Diff(string(actual[:n]), testCase.output); r != "" {  // 比较实际输出和期望输出
            t.Error(r)  // 输出比较结果
        }
    }
}

func TestReader1(t *testing.T) {
    type dataStruct struct {  // 定义数据结构
        input  string  // 输入字符串
        output string  // 期望输出字符串
    }

    bufLen := 8  // 缓冲区长度

    data := []dataStruct{  // 定义数据结构切片
        {"loooooooooooooooooooooooooooooooooooooooog", "loooooooooooooooooooooooooooooooooooooooog"},  // 输入字符串和期望输出字符串
        {`{"t": "\/testlooooooooooooooooooooooooooooong"}`, `{"t": "\/testlooooooooooooooooooooooooooooong"}`},  // 输入字符串和期望输出字符串
        {`{"t": "\/test"}`, `{"t": "\/test"}`},  // 输入字符串和期望输出字符串
        {`"\// fake comment"`, `"\// fake comment"`},  // 输入字符串和期望输出字符串
        {`"\/\/\/\/\/"`, `"\/\/\/\/\/"`},  // 输入字符串和期望输出字符串
    }
    # 遍历测试数据集合
    for _, testCase := range data {
        # 创建一个读取器，使用测试用例的输入数据
        reader := &Reader{
            Reader: bytes.NewReader([]byte(testCase.input)),
        }
        # 创建一个空的目标字节切片
        target := make([]byte, 0)
        # 创建一个缓冲区，用于读取数据
        buf := make([]byte, bufLen)
        var n int
        var err error
        # 循环读取数据到缓冲区，直到出现错误
        for n, err = reader.Read(buf); err == nil; n, err = reader.Read(buf) {
            # 如果读取的数据长度大于缓冲区长度，输出错误信息
            if n > len(buf) {
                t.Error("n: ", n)
            }
            # 将读取的数据追加到目标字节切片中
            target = append(target, buf[:n]...)
            # 重置缓冲区
            buf = make([]byte, bufLen)
        }
        # 如果出现错误并且不是文件结束错误，输出错误信息
        if err != nil && err != io.EOF {
            t.Error("error: ", err)
        }
        # 如果目标字节切片转换为字符串后不等于测试用例的输出，输出错误信息
        if string(target) != testCase.output {
            t.Error("got ", string(target), " want ", testCase.output)
        }
    }
# 闭合前面的函数定义
```