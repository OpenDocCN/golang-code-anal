# `grype\grype\db\curator_test.go`

```
package db
// 导入 db 包

import (
	"bufio"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"os"
	"os/exec"
	"path"
	"path/filepath"
	"strconv"
	"syscall"
	"testing"
	"time"

	"github.com/gookit/color"
	// 导入 color 包
	"github.com/spf13/afero"
	// 导入 afero 包
	"github.com/stretchr/testify/assert"
	// 导入 assert 包
// 导入所需的包
"github.com/stretchr/testify/require" // 用于测试时断言的包
"github.com/wagoodman/go-progress" // 用于显示进度条的包

// 导入内部自定义的包
"github.com/anchore/grype/internal/file" // 文件操作相关的包
"github.com/anchore/grype/internal/stringutil" // 字符串操作相关的包

// 定义 testGetter 结构体
type testGetter struct {
	file  map[string]string // 文件映射
	dir   map[string]string // 目录映射
	calls stringutil.StringSet // 字符串集合
	fs    afero.Fs // 文件系统接口
}

// 创建 testGetter 结构体的实例
func newTestGetter(fs afero.Fs, f, d map[string]string) *testGetter {
	return &testGetter{
		file:  f, // 初始化文件映射
		dir:   d, // 初始化目录映射
		calls: stringutil.NewStringSet(), // 初始化字符串集合
		fs:    fs, // 初始化文件系统接口
// GetFile函数用于下载指定URL的文件到指定路径。URL必须指向单个文件。
func (g *testGetter) GetFile(dst, src string, _ ...*progress.Manual) error {
    // 将下载的URL添加到调用列表中
    g.calls.Add(src)
    // 如果文件映射中不存在该URL，则返回错误
    if _, ok := g.file[src]; !ok {
        return fmt.Errorf("blerg, no file!")
    }
    // 将文件内容写入指定路径
    return afero.WriteFile(g.fs, dst, []byte(g.file[src]), 0755)
}

// Get函数用于将给定URL下载到指定目录。目录必须已经存在。
func (g *testGetter) GetToDir(dst, src string, _ ...*progress.Manual) error {
    // 将下载的URL添加到调用列表中
    g.calls.Add(src)
    // 如果目录映射中不存在该URL，则返回错误
    if _, ok := g.dir[src]; !ok {
        return fmt.Errorf("blerg, no file!")
    }
    // 将目录内容写入指定目录
    return afero.WriteFile(g.fs, dst, []byte(g.dir[src]), 0755)
}
func newTestCurator(tb testing.TB, fs afero.Fs, getter file.Getter, dbDir, metadataUrl string, validateDbHash bool) Curator {
    // 创建一个新的 Curator 对象
    c, err := NewCurator(Config{
        DBRootDir:           dbDir,
        ListingURL:          metadataUrl,
        ValidateByHashOnGet: validateDbHash,
    })
    // 断言错误为空
    require.NoError(tb, err)

    // 设置 Curator 对象的 downloader 和 fs 属性
    c.downloader = getter
    c.fs = fs

    // 返回创建好的 Curator 对象
    return c
}

func Test_defaultHTTPClient(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name    string
        hasCert bool
# 定义测试用例数组，每个测试用例包含名称和是否有自定义证书的信息
tests := []struct {
    name    string  // 测试用例名称
    hasCert bool    // 是否有自定义证书
}{
    {
        name:    "no custom cert should use default system root certs",  // 没有自定义证书应该使用默认的系统根证书
        hasCert: false,  // 没有自定义证书
    },
    {
        name:    "should use single custom cert",  // 应该使用单个自定义证书
        hasCert: true,  // 有自定义证书
    },
}

# 遍历测试用例数组
for _, test := range tests {
    # 使用测试名称创建子测试
    t.Run(test.name, func(t *testing.T) {
        var certPath string
        # 如果有自定义证书，则生成证书文件
        if test.hasCert {
            certPath = generateCertFixture(t)
        }

        # 使用默认的 HTTP 客户端和证书路径创建 HTTP 客户端
        httpClient, err := defaultHTTPClient(afero.NewOsFs(), certPath)
        require.NoError(t, err)  // 断言没有错误发生
# 检查测试对象是否有证书
if test.hasCert {
    # 断言 HTTP 客户端的传输层安全配置不为空
    require.NotNil(t, httpClient.Transport.(*http.Transport).TLSClientConfig)
    # 断言 HTTP 客户端的传输层安全配置的根证书颁发机构数量为1
    assert.Len(t, httpClient.Transport.(*http.Transport).TLSClientConfig.RootCAs.Subjects(), 1)
} else {
    # 断言 HTTP 客户端的传输层安全配置为空
    assert.Nil(t, httpClient.Transport.(*http.Transport).TLSClientConfig)
}

# 生成证书文件路径
func generateCertFixture(t *testing.T) string {
    path := "test-fixtures/tls/server.crt"
    # 检查路径是否存在
    if _, err := os.Stat(path); !os.IsNotExist(err) {
        # 如果路径存在，则返回路径
        return path
    }

    # 如果路径不存在，则记录日志并生成密钥/证书 fixture
    t.Logf(color.Bold.Sprint("Generating Key/Cert Fixture"))
# 获取当前工作目录
cwd, err := os.Getwd()
if err != nil:
    t.Errorf("unable to get cwd: %+v", err)

# 创建一个执行命令的对象，执行 make server.crt 命令
cmd := exec.Command("make", "server.crt")
# 设置命令执行的目录
cmd.Dir = filepath.Join(cwd, "test-fixtures/tls")

# 获取命令执行的标准错误输出管道
stderr, err := cmd.StderrPipe()
if err != nil:
    t.Fatalf("could not get stderr: %+v", err)
# 获取命令执行的标准输出管道
stdout, err := cmd.StdoutPipe()
if err != nil:
    t.Fatalf("could not get stdout: %+v", err)

# 启动命令执行
err = cmd.Start()
if err != nil:
		// 如果启动命令失败，输出错误信息
		t.Fatalf("failed to start cmd: %+v", err)
	}

	// 定义一个函数，用于读取并展示命令的输出
	show := func(label string, reader io.ReadCloser) {
		// 创建一个新的扫描器，用于逐行读取输出内容
		scanner := bufio.NewScanner(reader)
		// 设置扫描器的分割函数为逐行扫描
		scanner.Split(bufio.ScanLines)
		// 逐行读取输出内容，并输出到日志中
		for scanner.Scan() {
			t.Logf("%s: %s", label, scanner.Text())
		}
	}
	// 启动两个 goroutine 分别读取并展示标准输出和标准错误输出
	go show("out", stdout)
	go show("err", stderr)

	// 等待命令执行结束，如果有错误则输出相关信息
	if err := cmd.Wait(); err != nil {
		// 如果是退出错误，则输出相应信息
		if exiterr, ok := err.(*exec.ExitError); ok {
			// 程序以非零退出码退出

			// 这段代码适用于 Unix 和 Windows。尽管 syscall 包通常依赖于平台，WaitStatus 在 Unix 和 Windows 下都有定义，并且在这两种情况下都有
// 如果 err 是 ExitError 类型，则获取其系统信息，并判断退出状态是否为 0，如果不是则输出错误信息
if status, ok := exiterr.Sys().(syscall.WaitStatus); ok {
    if status.ExitStatus() != 0 {
        t.Fatalf("failed to generate fixture: rc=%d", status.ExitStatus())
    }
} else {
    // 如果 err 不是 ExitError 类型，则输出错误信息
    t.Fatalf("unable to get generate fixture result: %+v", err)
}

// 循环遍历测试用例
for _, test := range tests {
    // 测试 CuratorDownload 方法
    // 如果测试用例中的 err 为 true，则表示预期出现错误，否则表示预期成功
    // 根据测试用例中的 entry 调用 CuratorDownload 方法，获取返回的路径
    path, err := CuratorDownload(test.entry)
    if test.err {
        // 如果预期出现错误，但实际没有出现错误，则输出错误信息
        if err == nil {
            t.Fatalf("expected error but got nil")
        }
    } else {
        // 如果预期成功，但实际出现错误，则输出错误信息
        if err != nil {
            t.Fatalf("unexpected error: %+v", err)
        }
    }
    // 检查返回的路径是否符合预期
    if path != test.expectedURL {
        t.Fatalf("unexpected URL: expected=%s, got=%s", test.expectedURL, path)
    }
}
// 创建一个测试用例切片，每个测试用例包含名称、条目、期望的URL
{
    name: "download populates returned tempdir", // 测试用例名称
    entry: &ListingEntry{ // 列表条目
        Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC), // 构建时间
        URL:      mustUrl(url.Parse("http://a-url/payload.tar.gz")), // URL
        Checksum: "sha256:deadbeefcafe", // 校验和
    },
    expectedURL: "http://a-url/payload.tar.gz?checksum=sha256%3Adeadbeefcafe", // 期望的URL
},

// 遍历测试用例切片
for _, test := range tests {
    // 使用测试名称创建子测试
    t.Run(test.name, func(t *testing.T) {
        metadataUrl := "http://metadata.io" // 元数据URL
        contents := "CONTENTS!!!" // 内容
        files := map[string]string{} // 文件映射
        dirs := map[string]string{ // 目录映射
            test.expectedURL: contents, // 使用测试的期望URL作为键，内容作为值
        }
        fs := afero.NewMemMapFs() // 创建内存映射文件系统
// 创建一个新的测试数据获取器，用于获取文件系统中的文件和目录
getter := newTestGetter(fs, files, dirs)
// 创建一个新的测试策展人，用于测试数据库的下载和读取
cur := newTestCurator(t, fs, getter, "/tmp/dbdir", metadataUrl, false)

// 下载指定的条目，并使用手动进度跟踪
path, err := cur.download(test.entry, &progress.Manual{})
if err != nil {
    t.Fatalf("could not download entry: %+v", err)
}

// 检查是否获取器调用包含了预期的 URL
if !getter.calls.Contains(test.expectedURL) {
    t.Fatalf("never made the appropriate fetch call: %+v", getter.calls)
}

// 打开下载的文件
f, err := fs.Open(path)
if err != nil {
    t.Fatalf("no db file: %+v", err)
}

// 读取文件内容
actual, err := afero.ReadAll(f)
if err != nil {
    t.Fatalf("bad db file read: %+v", err)
}
# 定义测试函数 TestCuratorValidate，用于测试 CuratorValidate 函数
func TestCuratorValidate(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name              string
        fixture           string
        constraint        int
        cfgValidateDbHash bool
        err               bool
    }{
        {
            name:              "good checksum & good constraint",
            fixture:           "test-fixtures/curator-validate/good-checksum",
            # 其他测试用例的定义
		{
			# 配置是否验证数据库哈希值
			cfgValidateDbHash: true,
			# 约束条件
			constraint:        1,
			# 错误标识
			err:               false,
		},
		{
			# 测试名称
			name:              "good checksum & bad constraint",
			# 测试数据路径
			fixture:           "test-fixtures/curator-validate/good-checksum",
			# 配置是否验证数据库哈希值
			cfgValidateDbHash: true,
			# 约束条件
			constraint:        2,
			# 预期结果为错误
			err:               true,
		},
		{
			# 测试名称
			name:              "bad checksum & good constraint",
			# 测试数据路径
			fixture:           "test-fixtures/curator-validate/bad-checksum",
			# 配置是否验证数据库哈希值
			cfgValidateDbHash: true,
			# 约束条件
			constraint:        1,
			# 预期结果为错误
			err:               true,
		},
		{
			# 测试名称
			name:              "bad checksum & bad constraint",
# 定义测试用例数组，每个元素包含不同的测试参数
tests := []struct {
    name:              string,  # 测试用例名称
    fixture:           string,  # 测试用例使用的文件路径
    cfgValidateDbHash: bool,    # 是否验证数据库哈希值的配置
    constraint:        int,     # 约束条件
    err:               bool,    # 期望是否出现错误
}

# 遍历测试用例数组，依次执行每个测试用例
for _, test := range tests {
    # 使用测试用例的名称创建子测试
    t.Run(test.name, func(t *testing.T) {
        # 定义 metadataUrl 变量并赋值
        metadataUrl := "http://metadata.io"
        
        # 创建一个新的操作系统文件系统
        fs := afero.NewOsFs()
        
        # 创建一个新的测试 getter 对象
        getter := newTestGetter(fs, nil, nil)
    }
}
// 创建一个新的测试 Curator 对象，传入测试参数和配置信息
cur := newTestCurator(t, fs, getter, "/tmp/dbdir", metadataUrl, test.cfgValidateDbHash)

// 设置 Curator 对象的目标模式为测试约束
cur.targetSchema = test.constraint

// 验证数据完整性，返回元数据和错误信息
md, err := cur.validateIntegrity(test.fixture)

// 如果没有错误但是预期有错误，则输出错误信息
if err == nil && test.err {
    t.Errorf("expected an error but got none")
} else if err != nil && !test.err {
    // 如果有错误但是预期没有错误，则输出错误信息和元数据
    assert.NotZero(t, md)
    t.Errorf("expected no error, got: %+v", err)
}

// 测试 CuratorDBPathHasSchemaVersion 函数
func TestCuratorDBPathHasSchemaVersion(t *testing.T) {
    // 创建一个新的测试 Curator 对象，传入测试参数和配置信息
    fs := afero.NewMemMapFs()
    dbRootPath := "/tmp/dbdir"
    cur := newTestCurator(t, fs, nil, dbRootPath, "http://metadata.io", false)
# 使用断言来验证目标目录是否与预期目录相匹配
assert.Equal(t, path.Join(dbRootPath, strconv.Itoa(cur.targetSchema)), cur.dbDir, "unexpected dir")
# 使用断言来验证目标路径是否包含预期路径
assert.Contains(t, cur.dbPath, path.Join(dbRootPath, strconv.Itoa(cur.targetSchema)), "unexpected path")
}

# 测试验证数据过期性的函数
func TestCurator_validateStaleness(t *testing.T):
    # 定义结构体字段
    type fields struct:
        validateAge     bool
        maxAllowedDBAge time.Duration
        md              Metadata

    # 获取当前时间
    now := time.Now().UTC()
    # 测试用例
    tests := []struct:
        name    string
        cur     *Curator
        fields  fields
        wantErr assert.ErrorAssertionFunc
    :
        {
# 创建一个名为 "no-validation" 的测试用例，设置 md 字段为当前时间
{
    name: "no-validation",
    fields: fields{
        md: Metadata{Built: now},
    },
    wantErr: assert.NoError,
},
# 创建一个名为 "up-to-date" 的测试用例，设置 maxAllowedDBAge 字段为2小时，validateAge 字段为True，md 字段为当前时间
{
    name: "up-to-date",
    fields: fields{
        maxAllowedDBAge: 2 * time.Hour,
        validateAge:     true,
        md:              Metadata{Built: now},
    },
    wantErr: assert.NoError,
},
# 创建一个名为 "stale-data" 的测试用例，设置 maxAllowedDBAge 字段为1小时，validateAge 字段为True
// 创建一个包含元数据的结构体，其中包含了构建时间
md: Metadata{Built: now.UTC().Add(-4 * time.Hour)},
// 定义一个函数，用于检查是否返回的错误包含特定的信息
wantErr: func(t assert.TestingT, err error, i ...interface{}) bool {
    return assert.ErrorContains(t, err, "the vulnerability database was built")
},
// 定义一个测试用例，用于测试过期数据且不进行验证的情况
name: "stale-data-no-validation",
fields: fields{
    // 设置最大允许的数据库年龄为1小时
    maxAllowedDBAge: time.Hour,
    // 设置不进行验证
    validateAge:     false,
    // 设置元数据的构建时间为4小时前
    md:              Metadata{Built: now.Add(-4 * time.Hour)},
},
// 定义一个函数，用于检查是否返回的错误为无错误
wantErr: assert.NoError,
// 遍历测试用例
for _, tt := range tests {
    // 运行测试用例
    t.Run(tt.name, func(t *testing.T) {
        // 创建一个Curator对象，并设置验证年龄
        c := &Curator{
            validateAge:        tt.fields.validateAge,
# 设置maxAllowedBuiltAge字段为tt.fields.maxAllowedDBAge的值
maxAllowedBuiltAge: tt.fields.maxAllowedDBAge,
# 调用validateStaleness方法，传入tt.fields.md作为参数，验证是否符合预期
tt.wantErr(t, c.validateStaleness(tt.fields.md), fmt.Sprintf("validateStaleness(%v)", tt.fields.md))
```