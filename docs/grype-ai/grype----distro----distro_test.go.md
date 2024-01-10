# `grype\grype\distro\distro_test.go`

```
package distro

import (
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包

    "github.com/anchore/grype/internal/stringutil"  // 导入字符串工具包
    "github.com/anchore/syft/syft/linux"  // 导入 Linux 包
    "github.com/anchore/syft/syft/source"  // 导入源码包
)

func Test_NewDistroFromRelease(t *testing.T) {  // 定义测试函数
    tests := []struct {  // 定义测试用例结构体
        name               string  // 测试用例名称
        release            linux.Release  // Linux 发行版信息
        expectedVersion    string  // 期望的版本号
        expectedRawVersion string  // 期望的原始版本号
        expectedType       Type  // 期望的类型
        expectErr          bool  // 是否期望出错
    # 第一个测试用例：从 version-id 推导版本号
    {
        name: "go case: derive version from version-id",
        release: linux.Release{
            ID:        "centos",
            VersionID: "8",
            Version:   "7",
        },
        expectedType:       CentOS,
        expectedRawVersion: "8",
        expectedVersion:    "8.0.0",
    },
    # 第二个测试用例：当 release id 缺失时，使用 release name 作为备用
    {
        name: "fallback to release name when release id is missing",
        release: linux.Release{
            Name:      "windows",
            VersionID: "8",
        },
        expectedType:       Windows,
        expectedRawVersion: "8",
        expectedVersion:    "8.0.0",
    },
    # 第三个测试用例：当 version-id 缺失时，使用 version 作为备用
    {
        name: "fallback to version when version-id missing",
        release: linux.Release{
            ID:      "centos",
            Version: "8",
        },
        expectedType:       CentOS,
        expectedRawVersion: "8",
        expectedVersion:    "8.0.0",
    },
    # 第四个测试用例：缺失版本号导致错误
    {
        name: "missing version results in error",
        release: linux.Release{
            ID: "centos",
        },
        expectedType: CentOS,
    },
    # 第五个测试用例：错误的发行版类型导致错误
    {
        name: "bogus distro type results in error",
        release: linux.Release{
            ID:        "bogosity",
            VersionID: "8",
        },
        expectErr: true,
    },
    # 第六个测试用例：不稳定的 debian 版本
    {
        # syft -o json debian:testing | jq .distro
        name: "unstable debian",
        release: linux.Release{
            ID:              "debian",
            VersionID:       "",
            Version:         "",
            PrettyName:      "Debian GNU/Linux trixie/sid",
            VersionCodename: "trixie",
            Name:            "Debian GNU/Linux",
        },
        expectedType:       Debian,
        expectedRawVersion: "unstable",
        expectedVersion:    "",
    },
}
    # 遍历测试用例切片，每个测试用例包含名称和测试函数
    for _, test := range tests {
        # 使用测试名称创建子测试，传入测试函数
        t.Run(test.name, func(t *testing.T) {
            # 从发布信息创建新的数据对象
            d, err := NewFromRelease(test.release)
            # 如果期望出现错误
            if test.expectErr {
                # 断言出现错误
                require.Error(t, err)
                # 返回
                return
            } else {
                # 断言没有错误
                require.NoError(t, err)
            }

            # 断言数据对象的类型是否符合期望
            assert.Equal(t, test.expectedType, d.Type)
            # 如果期望的版本不为空
            if test.expectedVersion != "" {
                # 断言版本号是否符合期望
                assert.Equal(t, test.expectedVersion, d.Version.String())
            }
            # 如果期望的原始版本不为空
            if test.expectedRawVersion != "" {
                # 断言原始版本号是否符合期望
                assert.Equal(t, test.expectedRawVersion, d.FullVersion())
            }
        })
    }
func Test_NewDistroFromRelease_Coverage(t *testing.T) {
    // 定义测试用例的结构体
    tests := []struct {
        fixture string
        Type    Type
        Version string
    }

    // 创建观察到的发行版集合和定义的发行版集合
    observedDistros := stringutil.NewStringSet()
    definedDistros := stringutil.NewStringSet()

    // 遍历所有发行版类型，将其添加到定义的发行版集合中
    for _, distroType := range All {
        definedDistros.Add(string(distroType))
    }

    // 从定义的发行版集合中移除 Windows
    definedDistros.Remove(string(Windows))

    // 遍历测试用例
    for _, test := range tests {
        // 在子测试中运行
        t.Run(test.fixture, func(t *testing.T) {
            // 从目录创建新的源
            s, err := source.NewFromDirectory(source.DirectoryConfig{Path: test.fixture})
            require.NoError(t, err)

            // 获取文件解析器
            resolver, err := s.FileResolver(source.SquashedScope)
            require.NoError(t, err)

            // 确保 syft 能够获取所需的原始信息
            release := linux.IdentifyRelease(resolver)
            require.NotNil(t, release, "empty linux release info")

            // 从 syft 原始信息创建新的发行版
            d, err := NewFromRelease(*release)
            require.NoError(t, err)

            // 将观察到的发行版添加到集合中
            observedDistros.Add(d.Type.String())

            // 断言发行版类型和版本是否符合预期
            assert.Equal(t, test.Type, d.Type)
            if test.Version != "" {
                assert.Equal(t, d.Version.String(), test.Version)
            }
        })
    }

    // 确保测试用例与可识别的发行版保持同步
    if len(observedDistros) < len(definedDistros) {
        for _, d := range definedDistros.ToSlice() {
            t.Logf("   defined: %s", d)
        }
        for _, d := range observedDistros.ToSlice() {
            t.Logf("   observed: %s", d)
        }
        t.Errorf("distro coverage incomplete (defined=%d, coverage=%d)", len(definedDistros), len(observedDistros))
    }
}

func TestDistro_FullVersion(t *testing.T) {
    # 定义测试用例的结构体数组，包含版本号和期望结果
    tests := []struct {
        version  string
        expected string
    }{
        {
            version:  "8",
            expected: "8",
        },
        {
            version:  "18.04",
            expected: "18.04",
        },
        {
            version:  "0",
            expected: "0",
        },
        {
            version:  "18.1.2",
            expected: "18.1.2",
        },
    }

    # 遍历测试用例数组
    for _, test := range tests {
        # 使用测试框架运行测试，传入版本号和匿名函数
        t.Run(test.version, func(t *testing.T) {
            # 根据版本号创建新的 Linux 发行版对象
            d, err := NewFromRelease(linux.Release{
                ID:      "centos",
                Version: test.version,
            })
            # 断言错误为空
            require.NoError(t, err)
            # 断言实际结果等于期望结果
            assert.Equal(t, test.expected, d.FullVersion())
        })
    }
func TestDistro_MajorVersion(t *testing.T) {
    // 定义测试用例，包括版本号和期望的主要版本号
    tests := []struct {
        version  string
        expected string
    }{
        {
            version:  "8",
            expected: "8",
        },
        {
            version:  "18.04",
            expected: "18",
        },
        {
            version:  "0",
            expected: "0",
        },
        {
            version:  "18.1.2",
            expected: "18",
        },
    }

    // 遍历测试用例
    for _, test := range tests {
        // 运行单个测试用例
        t.Run(test.version, func(t *testing.T) {
            // 根据测试用例的版本号创建新的发行版对象
            d, err := NewFromRelease(linux.Release{
                ID:      "centos",
                Version: test.version,
            })
            // 断言不出现错误
            require.NoError(t, err)
            // 断言实际的主要版本号与期望的主要版本号相等
            assert.Equal(t, test.expected, d.MajorVersion())
        })
    }
}
```