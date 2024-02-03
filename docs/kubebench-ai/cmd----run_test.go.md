# `kubebench-aquasecurity\cmd\run_test.go`

```go
package cmd

import (
    "io/ioutil"  // 导入io/ioutil包，用于读取文件内容
    "os"  // 导入os包，用于操作文件和目录
    "path/filepath"  // 导入path/filepath包，用于处理文件路径
    "testing"  // 导入testing包，用于编写测试函数
)

func TestGetTestYamlFiles(t *testing.T) {
    cases := []struct {  // 定义测试用例结构体
        name      string  // 测试用例名称
        targets   []string  // 目标文件名列表
        benchmark string  // 基准文件名
        succeed   bool  // 是否成功的标志
        expCount  int  // 期望的文件数量
    }{
        {
            name:      "Specify two targets",  // 第一个测试用例的名称
            targets:   []string{"one", "two"},  // 第一个测试用例的目标文件名列表
            benchmark: "benchmark",  // 第一个测试用例的基准文件名
            succeed:   true,  // 第一个测试用例的成功标志
            expCount:  2,  // 第一个测试用例的期望文件数量
        },
        {
            name:      "Specify a target that doesn't exist",  // 第二个测试用例的名称
            targets:   []string{"one", "missing"},  // 第二个测试用例的目标文件名列表
            benchmark: "benchmark",  // 第二个测试用例的基准文件名
            succeed:   false,  // 第二个测试用例的成功标志
        },
        {
            name:      "No targets specified - should return everything except config.yaml",  // 第三个测试用例的名称
            targets:   []string{},  // 第三个测试用例的目标文件名列表
            benchmark: "benchmark",  // 第三个测试用例的基准文件名
            succeed:   true,  // 第三个测试用例的成功标志
            expCount:  3,  // 第三个测试用例的期望文件数量
        },
        {
            name:      "Specify benchmark that doesn't exist",  // 第四个测试用例的名称
            targets:   []string{"one"},  // 第四个测试用例的目标文件名列表
            benchmark: "missing",  // 第四个测试用例的基准文件名
            succeed:   false,  // 第四个测试用例的成功标志
        },
    }

    // Set up temp config directory
    var err error  // 定义错误变量
    cfgDir, err = ioutil.TempDir("", "kube-bench-test")  // 创建临时目录
    if err != nil {  // 如果创建临时目录出错
        t.Fatalf("Failed to create temp directory")  // 输出错误信息
    }
    defer os.RemoveAll(cfgDir)  // 在函数返回前删除临时目录

    d := filepath.Join(cfgDir, "benchmark")  // 拼接目录路径
    err = os.Mkdir(d, 0766)  // 创建临时目录
    if err != nil {  // 如果创建临时目录出错
        t.Fatalf("Failed to create temp dir")  // 输出错误信息
    }

    // We never expect config.yaml to be returned
    for _, filename := range []string{"one.yaml", "two.yaml", "three.yaml", "config.yaml"} {  // 遍历文件名列表
        err = ioutil.WriteFile(filepath.Join(d, filename), []byte("hello world"), 0666)  // 写入临时文件
        if err != nil {  // 如果写入文件出错
            t.Fatalf("error writing temp file %s: %v", filename, err)  // 输出错误信息
        }
    }
}
    # 遍历测试用例集合
    for _, c := range cases {
        # 使用测试用例的名称创建子测试
        t.Run(c.name, func(t *testing.T) {
            # 获取测试用例的 YAML 文件和可能的错误
            yamlFiles, err := getTestYamlFiles(c.targets, c.benchmark)
            # 如果出现错误且测试用例期望成功，则输出错误信息
            if err != nil && c.succeed {
                t.Fatalf("Error %v", err)
            }
            # 如果没有错误且测试用例不期望成功，则输出预期失败的信息
            if err == nil && !c.succeed {
                t.Fatalf("Expected failure")
            }
            # 如果获取到的 YAML 文件数量与预期数量不符，则输出数量不符的信息
            if len(yamlFiles) != c.expCount {
                t.Fatalf("Expected %d, got %d", c.expCount, len(yamlFiles))
            }
        })
    }
# 定义测试函数TestTranslate，用于测试translate函数的功能
func TestTranslate(t *testing.T) {
    # 定义测试用例，包括原始字符串和期望的翻译结果
    cases := []struct {
        name     string
        original string
        expected string
    }{
        {
            name:     "keep",
            original: "controlplane",
            expected: "controlplane",
        },
        {
            name:     "translate",
            original: "worker",
            expected: "node",
        },
        {
            name:     "translateLower",
            original: "Worker",
            expected: "node",
        },
        {
            name:     "Lower",
            original: "ETCD",
            expected: "etcd",
        },
    }

    # 遍历测试用例
    for _, c := range cases {
        # 对每个测试用例运行子测试
        t.Run(c.name, func(t *testing.T) {
            # 调用translate函数进行翻译
            ret := translate(c.original)
            # 检查翻译结果是否符合期望
            if ret != c.expected {
                # 如果不符合期望，则输出错误信息
                t.Fatalf("Expected %q, got %q", c.expected, ret)
            }
        })
    }
}
```