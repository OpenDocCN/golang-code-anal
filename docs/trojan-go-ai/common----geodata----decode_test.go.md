# `trojan-go\common\geodata\decode_test.go`

```go
package geodata_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "errors"  // 导入 errors 包，用于处理错误
    "io/fs"  // 导入 io/fs 包，用于文件系统操作
    "os"  // 导入 os 包，提供操作系统功能
    "path/filepath"  // 导入 path/filepath 包，用于处理文件路径
    "runtime"  // 导入 runtime 包，提供与 Go 运行时环境交互的函数
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/common/geodata"  // 导入自定义包 geodata
)

func init() {
    const (
        geoipURL   = "https://raw.githubusercontent.com/v2fly/geoip/release/geoip.dat"  // 定义 geoipURL 常量
        geositeURL = "https://raw.githubusercontent.com/v2fly/domain-list-community/release/dlc.dat"  // 定义 geositeURL 常量
    )

    wd, err := os.Getwd()  // 获取当前工作目录
    common.Must(err)  // 检查错误

    tempPath := filepath.Join(wd, "..", "..", "test", "temp")  // 拼接临时文件路径
    os.Setenv("TROJAN_GO_LOCATION_ASSET", tempPath)  // 设置环境变量 TROJAN_GO_LOCATION_ASSET

    geoipPath := common.GetAssetLocation("geoip.dat")  // 获取 geoip.dat 文件路径
    geositePath := common.GetAssetLocation("geosite.dat")  // 获取 geosite.dat 文件路径

    if _, err := os.Stat(geoipPath); err != nil && errors.Is(err, fs.ErrNotExist) {  // 检查 geoip.dat 文件是否存在
        common.Must(os.MkdirAll(tempPath, 0o755))  // 创建临时文件夹
        geoipBytes, err := common.FetchHTTPContent(geoipURL)  // 获取 geoip.dat 文件内容
        common.Must(err)  // 检查错误
        common.Must(common.WriteFile(geoipPath, geoipBytes))  // 写入 geoip.dat 文件
    }
    if _, err := os.Stat(geositePath); err != nil && errors.Is(err, fs.ErrNotExist) {  // 检查 geosite.dat 文件是否存在
        common.Must(os.MkdirAll(tempPath, 0o755))  // 创建临时文件夹
        geositeBytes, err := common.FetchHTTPContent(geositeURL)  // 获取 geosite.dat 文件内容
        common.Must(err)  // 检查错误
        common.Must(common.WriteFile(geositePath, geositeBytes))  // 写入 geosite.dat 文件
    }
}

func TestDecodeGeoIP(t *testing.T) {
    filename := common.GetAssetLocation("geoip.dat")  // 获取 geoip.dat 文件路径
    result, err := geodata.Decode(filename, "test")  // 解码 geoip.dat 文件
    if err != nil {
        t.Error(err)  // 输出错误信息
    }

    expected := []byte{10, 4, 84, 69, 83, 84, 18, 8, 10, 4, 127, 0, 0, 0, 16, 8}  // 定义期望的字节内容
    if !bytes.Equal(result, expected) {  // 检查解码结果是否与期望相同
        t.Errorf("failed to load geoip:test, expected: %v, got: %v", expected, result)  // 输出错误信息
    }
}

func TestDecodeGeoSite(t *testing.T) {
    filename := common.GetAssetLocation("geosite.dat")  // 获取 geosite.dat 文件路径
    result, err := geodata.Decode(filename, "test")  // 解码 geosite.dat 文件
    if err != nil {
        t.Error(err)  // 输出错误信息
    }
    # 定义期望的字节序列
    expected := []byte{10, 4, 84, 69, 83, 84, 18, 20, 8, 3, 18, 16, 116, 101, 115, 116, 46, 101, 120, 97, 109, 112, 108, 101, 46, 99, 111, 109}
    # 检查结果是否与期望的字节序列相等
    if !bytes.Equal(result, expected) {
        # 如果不相等，则输出错误信息
        t.Errorf("failed to load geosite:test, expected: %v, got: %v", expected, result)
    }
# 定义基准测试函数，用于测试加载地理位置数据的性能
func BenchmarkLoadGeoIP(b *testing.B) {
    # 初始化内存统计信息变量 m1 和 m2
    m1 := runtime.MemStats{}
    m2 := runtime.MemStats{}

    # 创建地理数据加载器对象
    loader := geodata.NewGeodataLoader()

    # 读取内存统计信息到 m1
    runtime.ReadMemStats(&m1)
    # 加载中国地理位置数据
    cn, _ := loader.LoadGeoIP("cn")
    # 加载私有地理位置数据
    private, _ := loader.LoadGeoIP("private")
    # 保持对 cn 和 private 的引用，防止被 GC 回收
    runtime.KeepAlive(cn)
    runtime.KeepAlive(private)
    # 读取内存统计信息到 m2
    runtime.ReadMemStats(&m2)

    # 报告内存分配增量
    b.ReportMetric(float64(m2.Alloc-m1.Alloc)/1024, "KiB(GeoIP-Alloc)")
    # 报告总内存分配增量
    b.ReportMetric(float64(m2.TotalAlloc-m1.TotalAlloc)/1024/1024, "MiB(GeoIP-TotalAlloc)")
}

# 定义基准测试函数，用于测试加载地理位置站点数据的性能
func BenchmarkLoadGeoSite(b *testing.B) {
    # 初始化内存统计信息变量 m3 和 m4
    m3 := runtime.MemStats{}
    m4 := runtime.MemStats{}

    # 创建地理数据加载器对象
    loader := geodata.NewGeodataLoader()

    # 读取内存统计信息到 m3
    runtime.ReadMemStats(&m3)
    # 加载中国地理位置站点数据
    cn, _ := loader.LoadGeoSite("cn")
    # 加载非中国地理位置站点数据
    notcn, _ := loader.LoadGeoSite("geolocation-!cn")
    # 加载私有地理位置站点数据
    private, _ := loader.LoadGeoSite("private")
    # 保持对 cn、notcn 和 private 的引用，防止被 GC 回收
    runtime.KeepAlive(cn)
    runtime.KeepAlive(notcn)
    runtime.KeepAlive(private)
    # 读取内存统计信息到 m4
    runtime.ReadMemStats(&m4)

    # 报告内存分配增量
    b.ReportMetric(float64(m4.Alloc-m3.Alloc)/1024/1024, "MiB(GeoSite-Alloc)")
    # 报告总内存分配增量
    b.ReportMetric(float64(m4.TotalAlloc-m3.TotalAlloc)/1024/1024, "MiB(GeoSite-TotalAlloc)")
}
```