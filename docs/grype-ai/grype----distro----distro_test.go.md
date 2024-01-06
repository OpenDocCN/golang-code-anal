# `grype\grype\distro\distro_test.go`

```
package distro

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
	"github.com/stretchr/testify/require"  // 导入断言包

	"github.com/anchore/grype/internal/stringutil"  // 导入字符串工具包
	"github.com/anchore/syft/syft/linux"  // 导入Linux包
	"github.com/anchore/syft/syft/source"  // 导入源码包
)

func Test_NewDistroFromRelease(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例结构
		name               string  // 测试用例名称
		release            linux.Release  // Linux发行版
		expectedVersion    string  // 期望版本号
		expectedRawVersion string  // 期望原始版本号
		expectedType       Type  // 期望类型
```

注释：以上代码是一个Go语言的测试函数，导入了一些测试和断言的包，并定义了一个测试用例的结构。
		expectErr          bool  // 期望是否出现错误
	}{
		{
			name: "go case: derive version from version-id",  // 测试用例名称
			release: linux.Release{  // 创建一个 linux.Release 结构体对象
				ID:        "centos",  // 设置 ID 字段值为 "centos"
				VersionID: "8",  // 设置 VersionID 字段值为 "8"
				Version:   "7",  // 设置 Version 字段值为 "7"
			},
			expectedType:       CentOS,  // 期望的操作系统类型为 CentOS
			expectedRawVersion: "8",  // 期望的原始版本号为 "8"
			expectedVersion:    "8.0.0",  // 期望的版本号为 "8.0.0"
		},
		{
			name: "fallback to release name when release id is missing",  // 测试用例名称
			release: linux.Release{  // 创建一个 linux.Release 结构体对象
				Name:      "windows",  // 设置 Name 字段值为 "windows"
				VersionID: "8",  // 设置 VersionID 字段值为 "8"
			},
			expectedType:       Windows,  // 期望的操作系统类型为 Windows
```

在这段代码中，我们创建了一个包含测试用例的结构体切片。每个测试用例包含了名称、操作系统版本信息和期望的结果。
# 定义一个预期的原始版本号为 "8"，预期的版本号为 "8.0.0" 的测试用例
{
    name: "fallback to version when version-id missing",
    # 创建一个名为 "centos" 的 linux.Release 对象，设置 ID 为 "centos"，版本为 "8"
    release: linux.Release{
        ID:      "centos",
        Version: "8",
    },
    # 预期的类型为 CentOS
    expectedType:       CentOS,
    # 预期的原始版本号为 "8"
    expectedRawVersion: "8",
    # 预期的版本号为 "8.0.0"
    expectedVersion:    "8.0.0",
},
# 定义一个预期的原始版本号为 "8"，预期的版本号为 "8.0.0" 的测试用例
{
    name: "fallback to version when version-id missing",
    # 创建一个名为 "centos" 的 linux.Release 对象，设置 ID 为 "centos"，版本为 "8"
    release: linux.Release{
        ID:      "centos",
        Version: "8",
    },
    # 预期的类型为 CentOS
    expectedType:       CentOS,
    # 预期的原始版本号为 "8"
    expectedRawVersion: "8",
    # 预期的版本号为 "8.0.0"
    expectedVersion:    "8.0.0",
},
# 定义一个预期的类型为 CentOS 的测试用例
{
    name: "missing version results in error",
    # 创建一个名为 "centos" 的 linux.Release 对象，设置 ID 为 "centos"
    release: linux.Release{
        ID: "centos",
    },
    # 预期的类型为 CentOS
    expectedType: CentOS,
}
		{
			// 测试错误的发行版类型是否会导致错误
			name: "bogus distro type results in error",
			// 创建一个名为 release 的 linux.Release 结构体对象
			release: linux.Release{
				// 设置发行版 ID 为 "bogosity"
				ID:        "bogosity",
				// 设置发行版版本 ID 为 "8"
				VersionID: "8",
			},
			// 期望出现错误
			expectErr: true,
		},
		{
			// 测试不稳定的 Debian 发行版
			name: "unstable debian",
			// 创建一个名为 release 的 linux.Release 结构体对象
			release: linux.Release{
				// 设置发行版 ID 为 "debian"
				ID:              "debian",
				// 设置发行版版本 ID 为空字符串
				VersionID:       "",
				// 设置发行版版本为空字符串
				Version:         "",
				// 设置发行版的友好名称为 "Debian GNU/Linux trixie/sid"
				PrettyName:      "Debian GNU/Linux trixie/sid",
				// 设置发行版版本代号为 "trixie"
				VersionCodename: "trixie",
				// 设置发行版名称为 "Debian GNU/Linux"
				Name:            "Debian GNU/Linux",
			},
			// 期望的发行版类型为 Debian
			expectedType:       Debian,
# 定义一个测试用例的切片，包含了不同的测试数据
tests := []struct {
    name            string
    release         Release
    expectErr       bool
    expectedType    string
    expectedVersion string
    expectedRawVersion string
}{
    // 在这里添加不同的测试数据
}

# 遍历测试用例切片，对每个测试用例进行测试
for _, test := range tests {
    # 使用测试用例的名称创建一个子测试
    t.Run(test.name, func(t *testing.T) {
        # 根据测试用例的 release 创建一个新的对象
        d, err := NewFromRelease(test.release)
        # 如果期望出现错误
        if test.expectErr {
            # 断言应该出现错误
            require.Error(t, err)
            return
        } else {
            # 断言不应该出现错误
            require.NoError(t, err)
        }

        # 断言对象的类型是否符合预期
        assert.Equal(t, test.expectedType, d.Type)
        # 如果预期版本不为空
        if test.expectedVersion != "" {
            # 断言对象的版本是否符合预期
            assert.Equal(t, test.expectedVersion, d.Version.String())
        }
        # 如果预期原始版本不为空
        if test.expectedRawVersion != "" {
            # 断言对象的原始版本是否符合预期
# 定义测试函数，用于测试从发布中创建新发行版的情况
func Test_NewDistroFromRelease_Coverage(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        fixture string  # 测试用例的文件夹路径
        Type    Type    # 发行版类型
        Version string  # 发行版版本
    }{
        {
            fixture: "test-fixtures/os/alpine",  # Alpine 发行版的测试文件夹路径
            Type:    Alpine,  # 发行版类型为 Alpine
            Version: "3.11.6",  # 发行版版本为 3.11.6
        },
        {
            fixture: "test-fixtures/os/amazon",  # Amazon 发行版的测试文件夹路径
            ...
		{
			# 定义操作系统类型为 AmazonLinux，版本为 "2.0.0"
			Type:    AmazonLinux,
			Version: "2.0.0",
		},
		{
			# 定义操作系统类型为 Busybox，版本为 "1.31.1"
			fixture: "test-fixtures/os/busybox",
			Type:    Busybox,
			Version: "1.31.1",
		},
		{
			# 定义操作系统类型为 CentOS，版本为 "8.0.0"
			fixture: "test-fixtures/os/centos",
			Type:    CentOS,
			Version: "8.0.0",
		},
		{
			# 定义操作系统类型为 Debian，版本为 "8.0.0"
			fixture: "test-fixtures/os/debian",
			Type:    Debian,
			Version: "8.0.0",
		},
		{
			# 定义操作系统类型为 Fedora
			fixture: "test-fixtures/os/fedora",
# 定义一个包含操作系统类型和版本的列表
{
    # 定义操作系统类型为 Fedora，版本为 31.0.0
    fixture: "test-fixtures/os/fedora",
    Type:    Fedora,
    Version: "31.0.0",
},
{
    # 定义操作系统类型为 RedHat，版本为 7.3.0
    fixture: "test-fixtures/os/redhat",
    Type:    RedHat,
    Version: "7.3.0",
},
{
    # 定义操作系统类型为 Ubuntu，版本为 20.4.0
    fixture: "test-fixtures/os/ubuntu",
    Type:    Ubuntu,
    Version: "20.4.0",
},
{
    # 定义操作系统类型为 OracleLinux，版本为 8.3.0
    fixture: "test-fixtures/os/oraclelinux",
    Type:    OracleLinux,
    Version: "8.3.0",
},
{
    # 定义操作系统类型为自定义
    fixture: "test-fixtures/os/custom",
}
		{
			# 定义操作系统类型为 RedHat，版本为 "8.0.0"
			Type:    RedHat,
			Version: "8.0.0",
		},
		{
			# 定义操作系统类型为 OpenSuseLeap，版本为 "15.2.0"
			fixture: "test-fixtures/os/opensuse-leap",
			Type:    OpenSuseLeap,
			Version: "15.2.0",
		},
		{
			# 定义操作系统类型为 SLES，版本为 "15.2.0"
			fixture: "test-fixtures/os/sles",
			Type:    SLES,
			Version: "15.2.0",
		},
		{
			# 定义操作系统类型为 Photon，版本为 "2.0.0"
			fixture: "test-fixtures/os/photon",
			Type:    Photon,
			Version: "2.0.0",
		},
		{
			# 定义操作系统类型为 arch
# 定义一个包含不同操作系统类型和版本的列表
[
    {
        # 使用的测试数据文件路径
        fixture: "test-fixtures/os/ubuntu18",
        # 操作系统类型为 Ubuntu
        Type:    Ubuntu,
        # 操作系统版本为 18.0.0
        Version: "18.0.0",
    },
    {
        fixture: "test-fixtures/os/archlinux",
        Type:    ArchLinux,
    },
    {
        fixture: "test-fixtures/partial-fields/missing-id",
        Type:    Debian,
        Version: "8.0.0",
    },
    {
        fixture: "test-fixtures/partial-fields/unknown-id",
        Type:    Debian,
        Version: "8.0.0",
    },
    {
        fixture: "test-fixtures/os/centos6",
        Type:    CentOS,
        Version: "6.0.0",
    },
    {
        fixture: "test-fixtures/os/centos5",
        Type:    CentOS,
    },
]
# 创建一个包含不同操作系统信息的列表
[
    {
        fixture: "test-fixtures/os/ubuntu",  # 操作系统的测试数据路径
        Type:    Ubuntu,  # 操作系统类型
        Version: "20.04",  # 操作系统版本
    },
    {
        fixture: "test-fixtures/os/debian",
        Type:    Debian,
        Version: "10.10",
    },
    {
        fixture: "test-fixtures/os/mariner",
        Type:    Mariner,
        Version: "1.0.0",
    },
    {
        fixture: "test-fixtures/os/rockylinux",
        Type:    RockyLinux,
        Version: "8.4.0",
    },
    {
        fixture: "test-fixtures/os/almalinux",
        Type:    AlmaLinux,
        Version: "8.4.0",
    },
    {
        fixture: "test-fixtures/os/gentoo",
        Type:    Gentoo,
        # 缺少版本信息
    }
]
		},
		{
			fixture: "test-fixtures/os/wolfi",  // 设置测试用例的 fixture 为 "test-fixtures/os/wolfi"
			Type:    Wolfi,  // 设置测试用例的类型为 Wolfi
		},
		{
			fixture: "test-fixtures/os/chainguard",  // 设置测试用例的 fixture 为 "test-fixtures/os/chainguard"
			Type:    Chainguard,  // 设置测试用例的类型为 Chainguard
		},
	}

	observedDistros := stringutil.NewStringSet()  // 创建一个空的字符串集合 observedDistros
	definedDistros := stringutil.NewStringSet()  // 创建一个空的字符串集合 definedDistros

	for _, distroType := range All {  // 遍历 All 列表中的每个元素
		definedDistros.Add(string(distroType))  // 将 distroType 转换为字符串并添加到 definedDistros 集合中
	}

	// 对于 Windows 有些作弊。没有支持检测/解析 Windows 操作系统，因此除非手动将其添加到 "observed distros" 中，否则无法遵守此测试
# 从定义的发行版中移除 Windows 字符串
definedDistros.Remove(string(Windows))

# 遍历测试用例
for _, test := range tests {
    # 在测试中运行指定的测试用例
    t.Run(test.fixture, func(t *testing.T) {
        # 从目录创建新的源对象
        s, err := source.NewFromDirectory(source.DirectoryConfig{Path: test.fixture})
        require.NoError(t, err)

        # 从源对象创建文件解析器
        resolver, err := s.FileResolver(source.SquashedScope)
        require.NoError(t, err)

        # 确保 syft 能够获取所需的原始信息
        release := linux.IdentifyRelease(resolver)
        require.NotNil(t, release, "empty linux release info")

        # 从 syft 的原始信息中创建新的发行版对象
        d, err := NewFromRelease(*release)
        require.NoError(t, err)

        # 将观察到的发行版添加到集合中
        observedDistros.Add(d.Type.String())
    })
}
// 使用断言来比较测试中的类型和实际的类型是否相等
assert.Equal(t, test.Type, d.Type)

// 如果测试中的版本不为空，则使用断言来比较实际版本和测试中的版本是否相等
if test.Version != "" {
    assert.Equal(t, d.Version.String(), test.Version)
}

// 确保测试用例与可以识别的发行版保持同步
if len(observedDistros) < len(definedDistros) {
    // 遍历已定义的发行版并记录日志
    for _, d := range definedDistros.ToSlice() {
        t.Logf("   defined: %s", d)
    }
    // 遍历已观察到的发行版并记录日志
    for _, d := range observedDistros.ToSlice() {
        t.Logf("   observed: %s", d)
    }
    // 输出错误信息，指出发行版覆盖不完整
    t.Errorf("distro coverage incomplete (defined=%d, coverage=%d)", len(definedDistros), len(observedDistros))
}
# 定义一个测试用例切片，每个测试用例包含一个版本号和期望的结果
tests := []struct {
    version  string   # 版本号
    expected string   # 期望的结果
}{
    {
        version:  "8",   # 版本号为8，期望的结果也为8
        expected: "8",
    },
    {
        version:  "18.04",   # 版本号为18.04，期望的结果也为18.04
        expected: "18.04",
    },
    {
        version:  "0",   # 版本号为0，期望的结果也为0
        expected: "0",
    },
    {
        version:  "18.1.2",   # 版本号为18.1.2，期望的结果也为18.1.2
        expected: "18.1.2",
}
		},
	}

	for _, test := range tests {
		// 遍历测试用例
		t.Run(test.version, func(t *testing.T) {
			// 使用测试用例的版本号创建新的发行版对象
			d, err := NewFromRelease(linux.Release{
				ID:      "centos",
				Version: test.version,
			})
			// 断言错误为空
			require.NoError(t, err)
			// 断言发行版对象的完整版本号与预期值相等
			assert.Equal(t, test.expected, d.FullVersion())
		})
	}

}

func TestDistro_MajorVersion(t *testing.T) {

	tests := []struct {
		// 版本号
		version  string
		expected string
	}{
		{
			version:  "8",  // 版本号为"8"，期望结果为"8"
			expected: "8",
		},
		{
			version:  "18.04",  // 版本号为"18.04"，期望结果为"18"
			expected: "18",
		},
		{
			version:  "0",  // 版本号为"0"，期望结果为"0"
			expected: "0",
		},
		{
			version:  "18.1.2",  // 版本号为"18.1.2"，期望结果为"18"
			expected: "18",
		},
	}
# 遍历测试用例列表
for _, test := range tests:
    # 使用测试版本创建新的Release对象
    d, err := NewFromRelease(linux.Release{
        ID:      "centos",
        Version: test.version,
    })
    # 确保没有错误发生
    require.NoError(t, err)
    # 断言测试结果是否与预期相等
    assert.Equal(t, test.expected, d.MajorVersion())
```