# `v2ray-core\testing\scenarios\common_coverage.go`

```
// 构建 V2Ray 程序，用于测试覆盖率
func BuildV2Ray() error {
    // 生成测试二进制文件路径
    genTestBinaryPath()
    // 如果测试二进制文件已存在，则直接返回
    if _, err := os.Stat(testBinaryPath); err == nil {
        return nil
    }
    // 使用 go 命令编译测试覆盖率相关的 V2Ray 程序
    cmd := exec.Command("go", "test", "-tags", "coverage coveragemain", "-coverpkg", "v2ray.com/core/...", "-c", "-o", testBinaryPath, GetSourcePath())
    return cmd.Run()
}

// 运行 V2Ray 程序，使用 Protobuf 格式的配置
func RunV2RayProtobuf(config []byte) *exec.Cmd {
    // 生成测试二进制文件路径
    genTestBinaryPath()
    // 获取环境变量中的 V2RAY_COV 目录，如果不存在则创建
    covDir := os.Getenv("V2RAY_COV")
    os.MkdirAll(covDir, os.ModeDir)
    // 生成随机的 ID 作为覆盖率文件的名称
    randomID := uuid.New()
    profile := randomID.String() + ".out"
    // 执行测试二进制文件，指定配置为标准输入，格式为 Protobuf，运行 TestRunMainForCoverage 测试，并输出覆盖率文件到指定目录
    proc := exec.Command(testBinaryPath, "-config=stdin:", "-format=pb", "-test.run", "TestRunMainForCoverage", "-test.coverprofile", profile, "-test.outputdir", covDir)
    proc.Stdin = bytes.NewBuffer(config)
    proc.Stderr = os.Stderr
    proc.Stdout = os.Stdout

    return proc
}
```