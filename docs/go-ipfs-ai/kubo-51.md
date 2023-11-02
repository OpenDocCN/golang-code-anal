# go-ipfs 源码解析 51

# `/opt/kubo/repo/fsrepo/migrations/migrations_test.go`

package migrations

import (
	"context"
	"fmt"
	"log"
	"os"
	"path/filepath"
	"strings"
	"testing"

	config "github.com/ipfs/kubo/config"
)

func TestFindMigrations(t *testing.T) {
	tmpDir := t.TempDir()

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	migs, bins, err := findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != 0 {
		t.Fatal("should not have found migrations")
	}

	for i := 1; i < 6; i++ {
		createFakeBin(i-1, i, tmpDir)
	}

	origPath := os.Getenv("PATH")
	os.Setenv("PATH", tmpDir)
	defer os.Setenv("PATH", origPath)

	migs, bins, err = findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != len(migs) {
		t.Fatal("missing", len(migs)-len(bins), "migrations")
	}

	os.Remove(bins[migs[2]])

	migs, bins, err = findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(bins) != len(migs)-1 {
		t.Fatal("should be missing one migration bin")
	}
}

func createFakeBin(i int, binName, tmpDir string) {
	file := filepath.Join(tmpDir, strings.Replace(fmt.Sprintf("%.2d", i), ""))
	err := os.Create(file)
	if err != nil {
		t.Fatalf("failed to create temporary bin %v: %v", i, err)
	}
	defer file.Close()
}


```
package migrations

import (
	"context"
	"fmt"
	"log"
	"os"
	"path/filepath"
	"strings"
	"testing"

	config "github.com/ipfs/kubo/config"
)

func TestFindMigrations(t *testing.T) {
	tmpDir := t.TempDir()

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	migs, bins, err := findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != 0 {
		t.Fatal("should not have found migrations")
	}

	for i := 1; i < 6; i++ {
		createFakeBin(i-1, i, tmpDir)
	}

	origPath := os.Getenv("PATH")
	os.Setenv("PATH", tmpDir)
	defer os.Setenv("PATH", origPath)

	migs, bins, err = findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != len(migs) {
		t.Fatal("missing", len(migs)-len(bins), "migrations")
	}

	os.Remove(bins[migs[2]])

	migs, bins, err = findMigrations(ctx, 0, 5)
	if err != nil {
		t.Fatal(err)
	}
	if len(bins) != len(migs)-1 {
		t.Fatal("should be missing one migration bin")
	}
}

```

该测试用例函数旨在验证“findMigrations”函数的正确性，功能如下：

1. 创建一个临时目录，并进入该目录。
2. 构造迁移文件（migrations）并初始化计数器。
3. 使用“findMigrations”函数查找至少5个文件。
4. 如果查找失败，打印错误并退出。
5. 如果找到了5个迁移文件，打印OK并退出。
6. 使用“createFakeBin”函数为前4个发现的文件创建临时文件并删除这些文件的优先级。
7. 使用“findMigrations”函数查找剩余的文件。
8. 如果剩余的文件不足5个，打印错误并退出。
9. 如果剩余的文件为5个，打印剩余的文件并退出。
10. 如果剩余的文件为4个，打印错误并退出。
11. 如果所有测试都通过，那么打印OK并退出。

函数的作用是验证“findMigrations”函数的行为，它通过测试其失败和成功情况来检查函数是否正常工作。


```
func TestFindMigrationsReverse(t *testing.T) {
	tmpDir := t.TempDir()

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	migs, bins, err := findMigrations(ctx, 5, 0)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != 0 {
		t.Fatal("should not have found migrations")
	}

	for i := 1; i < 6; i++ {
		createFakeBin(i-1, i, tmpDir)
	}

	origPath := os.Getenv("PATH")
	os.Setenv("PATH", tmpDir)
	defer os.Setenv("PATH", origPath)

	migs, bins, err = findMigrations(ctx, 5, 0)
	if err != nil {
		t.Fatal(err)
	}
	if len(migs) != 5 {
		t.Fatal("expected 5 migrations")
	}
	if len(bins) != len(migs) {
		t.Fatal("missing", len(migs)-len(bins), "migrations:", migs)
	}

	os.Remove(bins[migs[2]])

	migs, bins, err = findMigrations(ctx, 5, 0)
	if err != nil {
		t.Fatal(err)
	}
	if len(bins) != len(migs)-1 {
		t.Fatal("should be missing one migration bin")
	}
}

```

该代码是一个 Go 语言中的测试函数，名为 `TestFetchMigrations`。它使用 Go 标准库中的 `testing` 和 `log` 包，以及自定义的 `fetchMigrations` 函数。

函数的作用是测试 `fetchMigrations` 函数的正确性。具体来说，它创建一个测试服务器，下载两个迁移文件（`fs-repo-1-to-2` 和 `fs-repo-2-to-3`），并将下载的迁移文件存放在一个临时目录中。然后，它调用 `fetchMigrations` 函数下载这两个文件，并检查所有下载的文件是否正确地存在于磁盘上。如果存在文件错误，函数会打印错误消息并退出测试。如果所有下载的文件都正确，函数会打印一条日志消息。

函数的复杂度为较低，因为它只需要创建一个临时目录，并打印一些日志消息。


```
func TestFetchMigrations(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	ts := createTestServer()
	defer ts.Close()
	fetcher := NewHttpFetcher(CurrentIpfsDist, ts.URL, "", 0)

	tmpDir := t.TempDir()

	needed := []string{"fs-repo-1-to-2", "fs-repo-2-to-3"}
	buf := new(strings.Builder)
	buf.Grow(256)
	logger := log.New(buf, "", 0)
	fetched, err := fetchMigrations(ctx, fetcher, needed, tmpDir, logger)
	if err != nil {
		t.Fatal(err)
	}

	for _, bin := range fetched {
		_, err = os.Stat(bin)
		if os.IsNotExist(err) {
			t.Error("expected file to exist:", bin)
		}
	}

	// Check expected log output
	for _, mig := range needed {
		logOut := fmt.Sprintf("Downloading migration: %s", mig)
		if !strings.Contains(buf.String(), logOut) {
			t.Fatalf("did not find expected log output %q", logOut)
		}
		logOut = fmt.Sprintf("Downloaded and unpacked migration: %s", filepath.Join(tmpDir, mig))
		if !strings.Contains(buf.String(), logOut) {
			t.Fatalf("did not find expected log output %q", logOut)
		}
	}
}

```

这段代码的作用是测试一个名为 "RunMigrations" 的函数，它接受一个 testing.T 类型的参数。测试中，函数会模拟在一个本地开发环境中执行一个文件系统的迁移，以升级到 test-default-ipfs 环境。

具体来说，这段代码会执行以下操作：

1. 创建一个名为 "fakeHome" 的临时目录，用于存放模拟的文件系统。
2. 将模拟的文件系统的根目录设置为 fakeHome。
3. 创建名为 "test-default-ipfs" 的文件系统，并设置其文件权限为 0770。
4. 尝试创建一个名为 "test-default-ipfs" 的文件系统，如果执行成功，说明 downgrade 操作成功。
5. 尝试将 test-default-ipfs 文件系统中的 test-default-ipfs10-to-11 目录下的文件迁移到 test-default-ipfs 目录下的 test-default-ipfs10-to-11 目录，如果执行成功，说明 downgrade 操作成功。
6. 模拟一个 HTTP 请求，使用当前的 IPFS 作为分发仓库，请求的 URL 是 "/api/v1/migrate/{试验ID}"，其中 {试验ID} 是文件系统版本号。
7. 在模拟的 HTTP 请求中，如果 downgrade 操作成功，模拟返回的错误信息将包含 "downgrade not allowed" 字段。
8. 模拟另一个 HTTP 请求，使用当前的 IPFS 作为分发仓库，请求的 URL 是 "/api/v1/migrate/{试验ID}"，其中 {试验ID} 是文件系统版本号。
9. 如果 migration 过程失败，将在模拟的 HTTP 请求中捕获该错误，然后根据错误信息来判断是哪个文件系统版本的问题，并输出相应的错误信息。


```
func TestRunMigrations(t *testing.T) {
	fakeHome := t.TempDir()

	os.Setenv("HOME", fakeHome)
	fakeIpfs := filepath.Join(fakeHome, ".ipfs")

	err := os.Mkdir(fakeIpfs, os.ModePerm)
	if err != nil {
		panic(err)
	}

	testVer := 11
	err = WriteRepoVersion(fakeIpfs, testVer)
	if err != nil {
		t.Fatal(err)
	}

	ts := createTestServer()
	defer ts.Close()
	fetcher := NewHttpFetcher(CurrentIpfsDist, ts.URL, "", 0)

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	targetVer := 9

	err = RunMigration(ctx, fetcher, targetVer, fakeIpfs, false)
	if err == nil || !strings.HasPrefix(err.Error(), "downgrade not allowed") {
		t.Fatal("expected 'downgrade not alloed' error")
	}

	err = RunMigration(ctx, fetcher, targetVer, fakeIpfs, true)
	if err != nil {
		if !strings.HasPrefix(err.Error(), "migration fs-repo-10-to-11 failed") {
			t.Fatal(err)
		}
	}
}

```

这段代码定义了一个名为 `createFakeBin` 的函数，接受三个参数：

1. `from`：表示从 IP 地址到目标 IP 地址的迁移范围。
2. `to`：表示从 IP 地址到目标 IP 地址的距离。
3. `tmpDir`：表示临时目录。

该函数创建了一个名为 `migPath` 的文件夹，用于存储临时文件，并在其中创建一个名为 `ExeName` 的文件，用于存储来自 `migrationName` 函数的元数据。

函数首先创建一个空白的文件 `emptyFile`，并尝试将其写入到 `migPath`。如果失败，函数会输出一个错误并暂停程序。

接下来，函数尝试创建一个权限为 755 的文件，并将 `migPath` 目录的权限更改为 755。如果任何错误发生，函数会再次输出一个错误并暂停程序。

最后，函数会设置 `migPath` 目录下的文件 `emptyFile`，使其内容为 `ExeName` 文件的内容。


```
func createFakeBin(from, to int, tmpDir string) {
	migPath := filepath.Join(tmpDir, ExeName(migrationName(from, to)))
	emptyFile, err := os.Create(migPath)
	if err != nil {
		panic(err)
	}
	emptyFile.Close()
	err = os.Chmod(migPath, 0o755)
	if err != nil {
		panic(err)
	}
}

var testConfig = `
{
	"Bootstrap": [
		"/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
		"/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
	],
	"Migration": {
		"DownloadSources": ["IPFS", "HTTP", "127.0.0.1", "https://127.0.1.1"],
		"Keep": "cache"
	},
	"Peering": {
		"Peers": [
			{
				"ID": "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5",
				"Addrs": ["/ip4/127.0.0.1/tcp/4001", "/ip4/127.0.0.1/udp/4001/quic"]
			}
		]
	}
}
```

这段代码是一个测试用例，它的作用是测试一个名为`ReadMigrationConfigDefaults`的函数是否符合预期。

它首先创建了一个临时目录，并使用`makeConfig`函数将其设置为一个只读配置。接下来，它调用`ReadMigrationConfig`函数，并将`tmpDir`作为参数传递给该函数。如果函数执行成功，它将读取并返回一个`MigrationConfig`对象。

然后，该函数检查两个期望的配置值：`Keep`和`DownloadSources`。如果它们的值不符合期望，函数将错误并输出错误信息。

接下来，函数遍历`DownloadSources`的值，并检查它们是否与`DefaultMigrationDownloadSources`的值匹配。如果它们的值不匹配，函数将错误并输出错误信息。如果所有的下载源都匹配，函数将跳过该部分测试。


```
`

func TestReadMigrationConfigDefaults(t *testing.T) {
	tmpDir := makeConfig(t, "{}")

	cfg, err := ReadMigrationConfig(tmpDir, "")
	if err != nil {
		t.Fatal(err)
	}

	if cfg.Keep != config.DefaultMigrationKeep {
		t.Error("expected default value for Keep")
	}

	if len(cfg.DownloadSources) != len(config.DefaultMigrationDownloadSources) {
		t.Fatal("expected default number of download sources")
	}
	for i, src := range config.DefaultMigrationDownloadSources {
		if cfg.DownloadSources[i] != src {
			t.Errorf("wrong DownloadSource: %s", cfg.DownloadSources[i])
		}
	}
}

```

这段代码的作用是测试一个名为 `TestReadMigrationConfigErrors` 的函数，它会在每次运行时创建一个名为 `tmpDir` 的临时目录，并向其中存储一个包含一个 JSON 对象的配置文件。

接着，它使用 `ReadMigrationConfig` 函数读取这个配置文件，并检查是否有错误。如果错误不是由于文件中包含了 "unknown" 错误，则函数会输出错误并退出。在每次测试完成后，函数会将 `tmpDir` 中的所有内容删除，并再次使用 `ReadMigrationConfig` 函数读取并检查配置文件。

总的来说，这段代码的主要目的是测试 `ReadMigrationConfig` 函数的正确性，并验证它是否能够正确地读取和处理一个包含 "unknown" 错误的配置文件。


```
func TestReadMigrationConfigErrors(t *testing.T) {
	tmpDir := makeConfig(t, `{"Migration": {"Keep": "badvalue"}}`)

	_, err := ReadMigrationConfig(tmpDir, "")
	if err == nil {
		t.Fatal("expected error")
	}
	if !strings.HasPrefix(err.Error(), "unknown") {
		t.Fatal("did not get expected error:", err)
	}

	os.RemoveAll(tmpDir)
	_, err = ReadMigrationConfig(tmpDir, "")
	if err == nil {
		t.Fatal("expected error")
	}

	tmpDir = makeConfig(t, `}{`)
	_, err = ReadMigrationConfig(tmpDir, "")
	if err == nil {
		t.Fatal("expected error")
	}
}

```

这段代码是一个名为 `TestReadMigrationConfig` 的测试函数，用于测试 migration 配置文件中下载源的数量和下载源的匹配情况。

首先，在函数初始化时，使用 `makeConfig` 函数创建一个名为 `tmpDir` 的临时目录，并将其作为参数传递给 `ReadMigrationConfig` 函数，用于存储下载配置文件。

接着，使用 `ReadMigrationConfig` 函数读取配置文件并存储到 `cfg` 变量中，如果过程中出现错误，则使用 `t.Fatal` 函数进行错误处理。

然后，使用一系列条件语句检查 `cfg` 变量是否符合预期。首先检查 `cfg.DownloadSources` 数组长度是否为 4，如果不是 4，则使用 `t.Fatal` 函数进行错误处理。接着，遍历 `cfg.DownloadSources` 和 `expect` 变量，比较两者的对应元素是否相等，如果不相等，则使用 `t.Errorf` 函数进行错误处理。最后，检查 `cfg.Keep` 是否为 "cache"，如果不是，则使用 `t.Error` 函数进行错误处理。

综上所述，这段代码的作用是测试迁移配置文件中下载源的数量和下载源的匹配情况，如果出现错误则进行相应的错误处理。


```
func TestReadMigrationConfig(t *testing.T) {
	tmpDir := makeConfig(t, testConfig)

	cfg, err := ReadMigrationConfig(tmpDir, "")
	if err != nil {
		t.Fatal(err)
	}

	if len(cfg.DownloadSources) != 4 {
		t.Fatal("wrong number of DownloadSources")
	}
	expect := []string{"IPFS", "HTTP", "127.0.0.1", "https://127.0.1.1"}
	for i := range expect {
		if cfg.DownloadSources[i] != expect[i] {
			t.Errorf("wrong DownloadSource at %d", i)
		}
	}

	if cfg.Keep != "cache" {
		t.Error("wrong value for Keep")
	}
}

```

The code you provided is testing a migration fetcher that uses IPFS (InterPlanetary File System) to download files from a website. Here's how it works:

1. It imports the necessary packages.
2. It defines a variable `downloadSources` to store the sources where to fetch the files.
3. It defines a variable `mf` to store the MultiFetcher that will be used for the下载.
4. It calls the `GetMigrationFetcher` function with the `downloadSources` and an empty string as arguments.
5. It checks if the function returns an error. If it does, it logs it and returns.
6. If the function returns without error, it attempts to download the files using the MultiFetcher.
7. It checks if the MultiFetcher is an instance of `MultiFetcher`. If it's not, it logs the error and returns.
8. If the MultiFetcher is an instance of `MultiFetcher`, it checks if it has two fetchers. If it doesn't, it logs the error and returns.
9. It then downloads the files using the MultiFetcher.
10. It checks if the `downloadSources` variable is an empty string.
11. It should be noted that the `GetMigrationFetcher` function is not defined in the code you provided, and it appears to be an external function that is responsible for downloading files using IPFS.

Overall, this code tests a function that downloads files from a website using IPFS and a MultiFetcher. It checks if the MultiFetcher is an instance of `MultiFetcher`, and if the `downloadSources` variable is an empty string.


```
type mockIpfsFetcher struct{}

var _ Fetcher = (*mockIpfsFetcher)(nil)

func (m *mockIpfsFetcher) Fetch(ctx context.Context, filePath string) ([]byte, error) {
	return nil, nil
}

func (m *mockIpfsFetcher) Close() error {
	return nil
}

func TestGetMigrationFetcher(t *testing.T) {
	var f Fetcher
	var err error

	newIpfsFetcher := func(distPath string) Fetcher {
		return &mockIpfsFetcher{}
	}

	downloadSources := []string{"ftp://bad.gateway.io"}
	_, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err == nil || !strings.HasPrefix(err.Error(), "bad gateway addr") {
		t.Fatal("Expected bad gateway address error, got:", err)
	}

	downloadSources = []string{"::bad.gateway.io"}
	_, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err == nil || !strings.HasPrefix(err.Error(), "bad gateway addr") {
		t.Fatal("Expected bad gateway address error, got:", err)
	}

	downloadSources = []string{"http://localhost"}
	f, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err != nil {
		t.Fatal(err)
	}
	if rf, ok := f.(*RetryFetcher); !ok {
		t.Fatal("expected RetryFetcher")
	} else if _, ok := rf.Fetcher.(*HttpFetcher); !ok {
		t.Fatal("expected HttpFetcher")
	}

	downloadSources = []string{"ipfs"}
	f, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err != nil {
		t.Fatal(err)
	}
	if _, ok := f.(*mockIpfsFetcher); !ok {
		t.Fatal("expected IpfsFetcher")
	}

	downloadSources = []string{"http"}
	f, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err != nil {
		t.Fatal(err)
	}
	if rf, ok := f.(*RetryFetcher); !ok {
		t.Fatal("expected RetryFetcher")
	} else if _, ok := rf.Fetcher.(*HttpFetcher); !ok {
		t.Fatal("expected HttpFetcher")
	}

	downloadSources = []string{"IPFS", "HTTPS"}
	f, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err != nil {
		t.Fatal(err)
	}
	mf, ok := f.(*MultiFetcher)
	if !ok {
		t.Fatal("expected MultiFetcher")
	}
	if mf.Len() != 2 {
		t.Fatal("expected 2 fetchers in MultiFetcher")
	}

	downloadSources = []string{"ipfs", "https", "some.domain.io"}
	f, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err != nil {
		t.Fatal(err)
	}
	mf, ok = f.(*MultiFetcher)
	if !ok {
		t.Fatal("expected MultiFetcher")
	}
	if mf.Len() != 3 {
		t.Fatal("expected 3 fetchers in MultiFetcher")
	}

	downloadSources = nil
	_, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err == nil {
		t.Fatal("expected error when no sources specified")
	}

	downloadSources = []string{"", ""}
	_, err = GetMigrationFetcher(downloadSources, "", newIpfsFetcher)
	if err == nil {
		t.Fatal("expected error when empty string fetchers specified")
	}
}

```

该函数的作用是创建一个临时文件夹，并将一个字符串类型的参数configData写入该文件夹中。以下是函数的步骤：

1. 创建一个名为tmpDir的临时文件夹。
2. 使用os.Create创建一个名为filepath.Join(tmpDir, "config")的文件，并将其写入configData。
3. 如果创建文件或写入文件的过程中出现错误，则函数可能会崩溃并输出错误信息。
4. 返回文件夹的路径。

该函数的作用是创建一个临时文件夹，并将一个字符串类型的参数configData写入该文件夹中，然后返回文件夹的路径。这个函数可以在测试中用来暂存配置数据，以便在测试结束后，不需要将配置数据残留到 disk 中。


```
func makeConfig(t *testing.T, configData string) string {
	tmpDir := t.TempDir()

	cfgFile, err := os.Create(filepath.Join(tmpDir, "config"))
	if err != nil {
		t.Fatal(err)
	}
	if _, err = cfgFile.Write([]byte(configData)); err != nil {
		t.Fatal(err)
	}
	if err = cfgFile.Close(); err != nil {
		t.Fatal(err)
	}
	return tmpDir
}

```

# `/opt/kubo/repo/fsrepo/migrations/retryfetcher.go`

这段代码定义了一个名为`RetryFetcher`的结构体，它包含一个`Fetcher`字段和一个`MaxTries`字段。`Fetcher`字段是一个`Fetch`方法的指针，用于执行实际的网络请求。`MaxTries`字段用于限制最多尝试多少次网络请求，如果尝试次数超过了这个限制，则会返回一个错误。

该代码的作用是提供一个用于执行网络请求的 retry 机制。通过在 `Fetch`方法中执行最多 `MaxTries` 次网络请求，来实现从服务器下载文件的 retry 机制。如果网络请求过程中出现错误，则会返回一个错误，并设置 `lastErr` 字段的值为该错误，以便在之后的重试过程中使用。


```
package migrations

import (
	"context"
	"fmt"
)

type RetryFetcher struct {
	Fetcher
	MaxTries int
}

var _ Fetcher = (*RetryFetcher)(nil)

func (r *RetryFetcher) Fetch(ctx context.Context, filePath string) ([]byte, error) {
	var lastErr error
	for i := 0; i < r.MaxTries; i++ {
		out, err := r.Fetcher.Fetch(ctx, filePath)
		if err == nil {
			return out, nil
		}

		if ctx.Err() != nil {
			return nil, ctx.Err()
		}
		lastErr = err
	}
	return nil, fmt.Errorf("exceeded number of retries. last error was %w", lastErr)
}

```

此代码定义了一个名为`func`的函数，接收一个名为`r`的整数类型的参数`*RetryFetcher`。

此函数的作用是关闭`r.Fetcher`关闭的资源。由于`r.Fetcher`资源类型未定义，我们无法提供关于`r.Fetcher`的上下文信息。但我们可以确定的是，当一个资源关闭时，所有使用该资源的请求都将失败。因此，当`func`关闭`r.Fetcher`时，任何正在等待`r.Fetcher`资源完成请求的请求都将失败。


```
func (r *RetryFetcher) Close() error {
	return r.Fetcher.Close()
}

```

# `/opt/kubo/repo/fsrepo/migrations/unpack.go`

这段代码定义了一个名为 `unpackArchive` 的函数，它接受五个参数：

- `arcPath`：要解压缩的归档文件的完整路径。
- `atype`：归档文件的类型，可以是 "tar.gz" 或 "zip"。
- `root`：归档文件所在的目录。
- `name`：压缩文件的名字，可以用于区分不同版本的文件。
- `out`：解压缩后，将生成的文件保存到的目录。

函数首先定义了两个变量：`err` 和 `atype`，用于跟踪压缩过程的错误和结果类型。

然后，函数调用了两个子函数 `unpackTgz` 和 `unpackZip`，它们分别用于解析 "tar.gz" 和 "zip" 格式的归档文件。

如果解析过程出现错误，函数会捕获并返回错误信息。否则，如果归档文件是 "tar.gz" 格式，函数会返回 `unpackTgz` 函数的返回值；如果归档文件是 "zip" 格式，函数会返回 `unpackZip` 函数的返回值。

最后，函数会判断 `atype` 是否为 "tar.gz"，如果是，函数会继续执行 `unpackTgz` 的操作；如果不是，函数会执行 `unpackZip` 的操作。


```
package migrations

import (
	"archive/tar"
	"archive/zip"
	"compress/gzip"
	"errors"
	"fmt"
	"io"
	"os"
)

func unpackArchive(arcPath, atype, root, name, out string) error {
	var err error
	switch atype {
	case "tar.gz":
		err = unpackTgz(arcPath, root, name, out)
	case "zip":
		err = unpackZip(arcPath, root, name, out)
	default:
		err = fmt.Errorf("unrecognized archive type: %s", atype)
	}
	if err != nil {
		return err
	}
	return nil
}

```

该函数的作用是读取一个名为 "arcPath" 的归档文件，并将其解压缩为名为 "out" 的文件，如果解压缩过程中出现错误，则返回相应的错误信息。

具体来说，该函数首先打开一个名为 "arcPath" 的归档文件，并使用 "gzip" 包中的 "NewReader" 函数创建一个 "gzip" reader。然后，使用 "tar" 包中的 "NewReader" 函数创建一个 "tar" reader，并将其与 "gzip" reader 合并，使用 "unpack" 函数将归档文件解压缩为 "out" 文件。

在解压缩过程中，函数首先会遍历 "arcPath" 目录下的所有文件，如果找到了名为 "lookFor" 的文件，则会使用 "tar" reader 递归地读取该目录下的所有文件，直到找到名为 "lookFor" 的文件或者读取到 "tar" reader 的 endianness 字段为止。如果期间没有找到名为 "lookFor" 的文件，则函数会返回一个错误。

最后，如果 "arcPath" 或 "out" 目录不存在，则会抛出相应的错误。


```
func unpackTgz(arcPath, root, name, out string) error {
	fi, err := os.Open(arcPath)
	if err != nil {
		return fmt.Errorf("cannot open archive file: %w", err)
	}
	defer fi.Close()

	gzr, err := gzip.NewReader(fi)
	if err != nil {
		return fmt.Errorf("error opening gzip reader: %w", err)
	}
	defer gzr.Close()

	var bin io.Reader
	tarr := tar.NewReader(gzr)

	lookFor := root + "/" + name
	for {
		th, err := tarr.Next()
		if err != nil {
			if err == io.EOF {
				break
			}
			return fmt.Errorf("cannot read archive: %w", err)
		}

		if th.Name == lookFor {
			bin = tarr
			break
		}
	}

	if bin == nil {
		return errors.New("no binary found in archive")
	}

	return writeToPath(bin, out)
}

```

这段代码的主要目的是将一个ZIP档案中的某个文件夹(通过"arcPath"参数)中的压缩文件(通过"name"参数)解压到指定的目录(通过"out"参数)，并返回错误。

具体来说，代码执行以下步骤：

1. 打开一个ZIP档案的读取器(通过"zipr"变量)，并传入"arcPath"参数作为开始的位置。

2. 如果读取器初始化失败(错误)，则返回一个相应的错误。

3. 从读取器中遍历所有的文件。

4. 如果找到名为"lookFor"的文件(在"root"目录中，通过"name"参数指定)，则打开该文件的二进制内容，并将其存储在"bin"变量中。

5. 如果二进制内容也为空，则执行错误。

6. 最后，将二进制文件通过"writeToPath"函数写入指定路径，并返回结果。

"unpackZip"函数的作用是将ZIP档案中的某个文件夹解压到指定的目录，并返回错误。它需要传递三个参数：要解压缩的ZIP档案的路径、要解压缩的文件夹名称以及要写入的文件名。


```
func unpackZip(arcPath, root, name, out string) error {
	zipr, err := zip.OpenReader(arcPath)
	if err != nil {
		return fmt.Errorf("error opening zip reader: %w", err)
	}
	defer zipr.Close()

	lookFor := root + "/" + name
	var bin io.ReadCloser
	for _, fis := range zipr.File {
		if fis.Name == lookFor {
			rc, err := fis.Open()
			if err != nil {
				return fmt.Errorf("error extracting binary from archive: %w", err)
			}

			bin = rc
			break
		}
	}

	if bin == nil {
		return errors.New("no binary found in archive")
	}

	return writeToPath(bin, out)
}

```

该函数`writeToPath`的作用是创建一个输出文件并将其内容写入该文件中。它接受两个参数：一个闭包`rc`表示输入连接器(例如`os.Stderr`)，第二个参数`out`表示要写入的输出文件路径。

函数首先创建一个名为`out`的输出文件，如果出现错误，函数将返回一个`fmt.Errorf`类型的错误信息，其中错误参数为`out`和错误类型的`err`。

接下来，函数使用`os.Create`函数创建一个新的输出文件并将其写入连接器`rc`中。如果出现错误，函数将返回一个错误信息，其中错误参数为`out`和错误类型的`err`。

最后，函数使用`io.Copy`函数将连接器`rc`的内容写入到输出文件中。如果出现错误，函数将返回一个错误信息。

函数的实现使它可以在写入错误文件的同时创建一个新的输出文件。如果调用函数并且输出文件已存在，将创建覆盖原有文件而不是覆盖原有文件并写入新内容。


```
func writeToPath(rc io.Reader, out string) error {
	binfi, err := os.Create(out)
	if err != nil {
		return fmt.Errorf("error creating output file '%s': %w", out, err)
	}
	defer binfi.Close()

	_, err = io.Copy(binfi, rc)

	return err
}

```

# `/opt/kubo/repo/fsrepo/migrations/unpack_test.go`

该代码包是一个Migrations包，用于处理与文件系统相关的操作。具体来说，它实现了以下功能：

1. 解压缩/压缩归档文件，包括tar和zip格式。
2. 对输入文件进行编码和解码，包括gzip编码。
3. 对输入文件进行遍历。
4. 提取tar文件中的额外标志，如档案类型，大小等。
5. 生成测试文件的路径，用于测试。

该代码包可能还被用于其他Migrations相关操作，比如将数据迁移到云存储或导出为移动设备支持的格式等。


```
package migrations

import (
	"archive/tar"
	"archive/zip"
	"bufio"
	"compress/gzip"
	"io"
	"os"
	"path"
	"path/filepath"
	"strings"
	"testing"
)

```

该代码是一个名为 "TestUnpackArchive" 的测试函数，用于测试是否能够正确地打开或关闭不同类型的归档文件。

函数中包含三个测试用例：

1. 测试正常情况，即使用 "no-arch-type" 归档类型打开一个普通文件，并将其转换为 "tar.gz"、"zip" 和 "zip" 两种不同的归档类型。
2. 测试无法打开的错误情况，即使用 "no-archive" 归档类型尝试打开一个普通文件，并检查是否有 "cannot open archive file" 错误消息。
3. 测试无法打开的错误情况，即使用 "no-archive" 归档类型尝试打开一个普通文件，并检查是否有 "error opening zip reader" 错误消息。

函数的作用是验证是否能够正确地打开或关闭不同类型的归档文件，以帮助开发者检查代码的质量和完整性。


```
func TestUnpackArchive(t *testing.T) {
	// Check unrecognized archive type
	err := unpackArchive("", "no-arch-type", "", "", "")
	if err == nil || err.Error() != "unrecognized archive type: no-arch-type" {
		t.Fatal("expected 'unrecognized archive type' error")
	}

	// Test cannot open errors
	err = unpackArchive("no-archive", "tar.gz", "", "", "")
	if err == nil || !strings.HasPrefix(err.Error(), "cannot open archive file") {
		t.Fatal("expected 'cannot open' error, got:", err)
	}
	err = unpackArchive("no-archive", "zip", "", "", "")
	if err == nil || !strings.HasPrefix(err.Error(), "error opening zip reader") {
		t.Fatal("expected 'cannot open' error, got:", err)
	}
}

```

This is a Go test that tests the `unpackTgz` function. It uses the `testing.T` struct to indicate that this is a testing.go file, and it defines a single test, `TestUnpackTgz`.

Here's the implementation of the `TestUnpackTgz` function:
go
func TestUnpackTgz(t *testing.T) {
	tmpDir := t.TempDir()

	badTarGzip := filepath.Join(tmpDir, "bad.tar.gz")
	err := os.WriteFile(badTarGzip, []byte("bad-data"), 0o644)
	if err != nil {
		panic(err)
	}
	err = unpackTgz(badTarGzip, "", "abc", "abc")
	if err == nil || !strings.HasPrefix(err.Error(), "error opening gzip reader") {
		t.Fatal("expected error opening gzip reader, got:", err)
	}

	testTarGzip := filepath.Join(tmpDir, "test.tar.gz")
	testData := "some data"
	err = writeTarGzipFile(testTarGzip, "testroot", "testfile", testData)
	if err != nil {
		panic(err)
	}

	out := filepath.Join(tmpDir, "out.txt")

	// Test looking for file that is not in archive
	err = unpackTgz(testTarGzip, "testroot", "abc", out)
	if err == nil || err.Error() != "no binary found in archive" {
		t.Fatal("expected 'no binary found in archive' error, got:", err)
	}

	// Test that unpack works.
	err = unpackTgz(testTarGzip, "testroot", "testfile", out)
	if err != nil {
		t.Fatal(err)
	}

	fi, err := os.Stat(out)
	if err != nil {
		t.Fatal(err)
	}
	if fi.Size() != int64(len(testData)) {
		t.Fatal("unpacked file size is", fi.Size(), "expected", len(testData))
	}
}

The `unpackTgz` function is expected to handle an archive file `tar.gz` correctly. It takes three arguments:

* The name of the archive file to unpack.
* The name of the output file that will contain the unpacked data.
* A list of filenames in the archive that should be included in the output file.

The function reads the archive file and unpacks its contents to the specified output file. It checks for errors opening the archive file and returns them if any. It also checks that the output file has the expected size, and reports an error if it doesn't.


```
func TestUnpackTgz(t *testing.T) {
	tmpDir := t.TempDir()

	badTarGzip := filepath.Join(tmpDir, "bad.tar.gz")
	err := os.WriteFile(badTarGzip, []byte("bad-data\n"), 0o644)
	if err != nil {
		panic(err)
	}
	err = unpackTgz(badTarGzip, "", "abc", "abc")
	if err == nil || !strings.HasPrefix(err.Error(), "error opening gzip reader") {
		t.Fatal("expected error opening gzip reader, got:", err)
	}

	testTarGzip := filepath.Join(tmpDir, "test.tar.gz")
	testData := "some data"
	err = writeTarGzipFile(testTarGzip, "testroot", "testfile", testData)
	if err != nil {
		panic(err)
	}

	out := filepath.Join(tmpDir, "out.txt")

	// Test looking for file that is not in archive
	err = unpackTgz(testTarGzip, "testroot", "abc", out)
	if err == nil || err.Error() != "no binary found in archive" {
		t.Fatal("expected 'no binary found in archive' error, got:", err)
	}

	// Test that unpack works.
	err = unpackTgz(testTarGzip, "testroot", "testfile", out)
	if err != nil {
		t.Fatal(err)
	}

	fi, err := os.Stat(out)
	if err != nil {
		t.Fatal(err)
	}
	if fi.Size() != int64(len(testData)) {
		t.Fatal("unpacked file size is", fi.Size(), "expected", len(testData))
	}
}

```

该代码是一个 Go 语言中的测试函数，名为 TestUnpackZip。它的作用是测试一个名为 UnpackZip 的函数的正确性。

具体来说，该函数的行为如下：

1. 创建一个临时目录并设置为 UnpackZip 函数的临时目录。
2. 创建一个名为 "bad.zip" 的压缩文件，并尝试向其中写入 "bad-data" 字节数据。如果写入成功，则继续执行下一步。
3. 尝试解压缩 "bad.zip" 文件，并检查是否有错误。如果没有错误，则表示 UnpackZip 函数可以正常工作。
4. 创建一个名为 "test.zip" 的测试文件，并向其中写入 "some data" 字节数据。
5. 尝试解压缩 "test.zip" 文件，并检查是否能够成功解压缩。如果解压缩成功，则表示 UnpackZip 函数可以正常工作。
6. 创建一个名为 "out.txt" 的输出文件，并向其中写入 "no binary found in archive" 错误消息。
7. 尝试解压缩 "test.zip" 文件，并检查是否能够成功解压缩。如果解压缩成功，则表示 UnpackZip 函数可以正常工作，并且该错误消息应该在输出文件中出现。
8. 创建一个名为 "testroot" 的测试目录，并向其中创建一个名为 "testfile" 的文件。
9. 尝试解压缩 "test.zip" 文件，并检查是否能够成功解压缩。如果解压缩成功，则表示 UnpackZip 函数可以正常工作。
10. 检查输出文件中的内容是否正确。如果输出文件中的内容与预期不符，则说明 UnpackZip 函数存在错误。


```
func TestUnpackZip(t *testing.T) {
	tmpDir := t.TempDir()

	badZip := filepath.Join(tmpDir, "bad.zip")
	err := os.WriteFile(badZip, []byte("bad-data\n"), 0o644)
	if err != nil {
		panic(err)
	}
	err = unpackZip(badZip, "", "abc", "abc")
	if err == nil || !strings.HasPrefix(err.Error(), "error opening zip reader") {
		t.Fatal("expected error opening zip reader, got:", err)
	}

	testZip := filepath.Join(tmpDir, "test.zip")
	testData := "some data"
	err = writeZipFile(testZip, "testroot", "testfile", testData)
	if err != nil {
		panic(err)
	}

	out := filepath.Join(tmpDir, "out.txt")

	// Test looking for file that is not in archive
	err = unpackZip(testZip, "testroot", "abc", out)
	if err == nil || err.Error() != "no binary found in archive" {
		t.Fatal("expected 'no binary found in archive' error, got:", err)
	}

	// Test that unpack works.
	err = unpackZip(testZip, "testroot", "testfile", out)
	if err != nil {
		t.Fatal(err)
	}

	fi, err := os.Stat(out)
	if err != nil {
		t.Fatal(err)
	}
	if fi.Size() != int64(len(testData)) {
		t.Fatal("unpacked file size is", fi.Size(), "expected", len(testData))
	}
}

```

该函数的作用是创建一个名为 tarGzipFile 的 tar 归档文件，并将一个字符串数据体存储到该文件中。

具体来说，它执行以下操作：

1. 创建一个名为 archName 的文件，如果执行成功，则返回。
2. 创建一个名为 fileName 的文件，并将字符串 data 写入该文件中。如果执行失败，函数返回。
3. 创建一个名为 w 的缓冲Writer，用于向 tar 归档文件中写入数据。
4. 调用 writeTarGzip 函数，将文件根目录、文件名和数据写入 tar 归档文件中。如果写入过程中出现错误，函数返回。
5. 调用 w.Flush 函数，将缓冲的数据写入到文件中。如果调用失败，函数返回。
6. 调用 archFile.Close 函数，关闭 tar 归档文件。如果调用失败，函数返回。
7. 返回 nil，表示操作成功。


```
func writeTarGzipFile(archName, root, fileName, data string) error {
	archFile, err := os.Create(archName)
	if err != nil {
		return err
	}
	defer archFile.Close()
	w := bufio.NewWriter(archFile)

	err = writeTarGzip(root, fileName, data, w)
	if err != nil {
		return err
	}
	// Flush buffered data to file
	if err = w.Flush(); err != nil {
		return err
	}
	// Close tar file
	if err = archFile.Close(); err != nil {
		return err
	}
	return nil
}

```

该函数接受三个参数：根目录（`root`）、文件名（`fileName`）和数据字符串（`data`）。它使用`tar`和`gzip`库来创建一个归档文件并写入数据。

函数的作用是将数据写入一个名为`fileName.tar.gz`的归档文件中。首先，它创建一个名为`writeTarGzip`的函数，该函数接受三个参数：`root`、`fileName`和`data`。如果`fileName`为空，则函数返回一个错误。

如果`fileName`不为空，函数首先创建一个名为`hdr`的`tar.Header`对象，其中包含`fileName`、`mode`和`size`。如果出现任何错误，函数将返回并等待后续操作完成。如果`tw`已经关闭，函数将关闭`gzip`作家，并确保所有数据已写入缓冲区。最后，函数返回一个错误。

如果`tw`关闭或数据写入失败，函数将返回一个错误。


```
func writeTarGzip(root, fileName, data string, w io.Writer) error {
	// gzip writer writes to buffer
	gzw := gzip.NewWriter(w)
	defer gzw.Close()
	// tar writer writes to gzip
	tw := tar.NewWriter(gzw)
	defer tw.Close()

	var err error
	if fileName != "" {
		hdr := &tar.Header{
			Name: path.Join(root, fileName),
			Mode: 0o600,
			Size: int64(len(data)),
		}
		// Write header
		if err = tw.WriteHeader(hdr); err != nil {
			return err
		}
		// Write file body
		if _, err := tw.Write([]byte(data)); err != nil {
			return err
		}
	}

	if err = tw.Close(); err != nil {
		return err
	}
	// Close gzip writer; finish writing gzip data to buffer
	if err = gzw.Close(); err != nil {
		return err
	}
	return nil
}

```

该函数的主要作用是创建一个名为指定的归档文件夹的归档文件，并将指定的数据文件写入该文件夹中。

具体来说，该函数的实现步骤如下：

1. 创建一个名为指定归档文件夹的归档文件并关闭。
2. 创建一个名为指定文件名的数据文件并写入指定数据。
3. 调用writeZip函数将数据文件写入归档文件中。
4. 如果数据文件写入失败，函数将返回错误。
5. 如果归档文件写入成功，函数将返回 nil。


```
func writeZipFile(archName, root, fileName, data string) error {
	archFile, err := os.Create(archName)
	if err != nil {
		return err
	}
	defer archFile.Close()
	w := bufio.NewWriter(archFile)

	err = writeZip(root, fileName, data, w)
	if err != nil {
		return err
	}
	// Flush buffered data to file
	if err = w.Flush(); err != nil {
		return err
	}
	// Close zip file
	if err = archFile.Close(); err != nil {
		return err
	}
	return nil
}

```

此代码定义了一个名为`writeZip`的函数，接受三个参数：根目录`root`、文件名`fileName`和数据`data`，并返回一个错误。函数创建了一个名为`zw`的`zip.Writer`对象，用于写入 zip 文件。

函数首先创建一个名为`path.Join(root, fileName)`的路径，该路径与根目录`root`中的文件名`fileName`相对应。然后，使用`zw.Create`方法创建一个`zip.Writer`对象，并将其写入指定路径。

接下来，函数使用`f.Write`方法将数据写入到名为`data`的文件中。如果过程中出现错误，函数将返回一个错误。

最后，函数使用`tw.Close`方法关闭`zip.Writer`对象，确保所有写入操作都已完成。如果关闭过程中出现错误，函数将返回一个错误。

函数返回一个`nil`表示没有错误。


```
func writeZip(root, fileName, data string, w io.Writer) error {
	zw := zip.NewWriter(w)
	defer zw.Close()

	// Write file name
	f, err := zw.Create(path.Join(root, fileName))
	if err != nil {
		return err
	}
	// Write file data
	_, err = f.Write([]byte(data))
	if err != nil {
		return err
	}

	// Close zip writer
	if err = zw.Close(); err != nil {
		return err
	}
	return nil
}

```

# `/opt/kubo/repo/fsrepo/migrations/versions.go`

这段代码定义了一个名为 "migrations" 的包，其中包含了一些与 migrations(迁移)相关的方法和工具。

具体来说，这个包通过引入了 "bufio"、"bytes"、"context"、"errors"、"fmt"、"path" 和 "sort" 这些包，提供了一些用于处理文件、字符串以及路径相关操作的工具和函数。

其中，定义了一个名为 "strings.Map" 的函数，该函数接受一个可变长度的字符串参数，将其中的每个元素打乱顺序，并将结果存储在一个新字符串中。该函数使用了 "sort" 包中的 "sort" 函数来对字符串进行排序，然后使用了 "bufio" 包中的 "lines" 函数来读取文件中的每个字符串元素，最后将排序后的元素重新组合成新的字符串。

另外，定义了一个名为 "path.Combine" 的函数，该函数用于将两个或多个路径字符串组合成一个。该函数使用了 "path" 包中的 "path.join" 函数来连接两个或多个路径字符串，并将结果返回。

最后，定义了一个名为 "migrate" 的函数，该函数接受一个持久化器(类似于数据库的持久化器)和一个版本号(或者是最后一个提交时的版本号)，用于将应用程序的状态迁移到一个新的版本。该函数会读取应用程序配置文件中所有的持久化器，并将这些持久化器与新的版本号进行比较，如果应用程序配置文件中所有的持久化器与新的版本号不匹配，则会尝试将应用程序的状态持久化到新的版本号。


```
package migrations

import (
	"bufio"
	"bytes"
	"context"
	"errors"
	"fmt"
	"path"
	"sort"
	"strings"

	"github.com/blang/semver/v4"
)

```

此代码是一个名为 `LatestDistVersion` 的函数，它返回了指定发行版（distribution）上最新的版本，且只有在支持稳定版本（stableOnly）的情况下返回最新的版本。

函数首先从 `DistVersions` 函数中获取指定发行版的版本列表，然后遍历版本列表，对每个版本进行处理。在处理过程中，函数会检查给定的版本是否包含 `-rc` 或 `-dev` 这样的前缀。如果是，就跳过这个版本，如果不是，则返回该版本，同时返回一个错误。如果经过以上处理仍然无法找到一个非 `dev` 版本的版本，函数就会返回一个错误。

函数的实现主要使用了函数式编程的方法，以避免使用副作用（ side-effect）。这样的实现可以提高代码的可读性和可维护性，同时减少了代码的复杂度和出错率。


```
const distVersions = "versions"

// LatestDistVersion returns the latest version, of the specified distribution,
// that is available on the distribution site.
func LatestDistVersion(ctx context.Context, fetcher Fetcher, dist string, stableOnly bool) (string, error) {
	vs, err := DistVersions(ctx, fetcher, dist, false)
	if err != nil {
		return "", err
	}

	for i := len(vs) - 1; i >= 0; i-- {
		ver := vs[i]
		if stableOnly && strings.Contains(ver, "-rc") {
			continue
		}
		if strings.Contains(ver, "-dev") {
			continue
		}
		return ver, nil
	}
	return "", errors.New("could not find a non dev version")
}

```

此代码定义了一个名为 `DistVersions` 的函数，它返回指定发行版的所有可用版本。函数的第一个参数是一个 `Fetcher` 类型的参数，用于从指定分销网站获取版本文件；第二个参数是一个 `dist` 参数，指定了要查找的分发版；第三个参数是一个 `sortDesc` 布尔参数，用于指示是否按版本顺序排序。

函数内部首先从 `Fetcher` 获取指定分销版的所有版本，并使用 `bufio.NewScanner` 创建一个字符串读取器和一个字符串编码器，逐行读取版本文件内容并将其转换为 `semver.Version` 类型，然后将版本存储在一个 `var` 类型的数组中。

接下来，函数检查版本读取过程中是否出现错误，如果是，则返回一个空字符串和错误信息。然后，根据 `sortDesc` 参数确定版本排序的顺序，如果排序降序，则使用 `sort.Sort` 函数对版本进行排序，否则使用 `semver.Versions` 函数获取版本顺序并直接返回。

最后，函数创建一个字符串数组并将版本信息添加到其中，然后使用函数名称并传递输入参数，返回版本数组和 nil，如果错误则返回错误信息。


```
// DistVersions returns all versions of the specified distribution, that are
// available on the distriburion site.  List is in ascending order, unless
// sortDesc is true.
func DistVersions(ctx context.Context, fetcher Fetcher, dist string, sortDesc bool) ([]string, error) {
	versionBytes, err := fetcher.Fetch(ctx, path.Join(dist, distVersions))
	if err != nil {
		return nil, err
	}

	prefix := "v"
	var vers []semver.Version

	scan := bufio.NewScanner(bytes.NewReader(versionBytes))
	for scan.Scan() {
		ver, err := semver.Make(strings.TrimLeft(scan.Text(), prefix))
		if err != nil {
			continue
		}
		vers = append(vers, ver)
	}
	if scan.Err() != nil {
		return nil, fmt.Errorf("could not read versions: %w", scan.Err())
	}

	if sortDesc {
		sort.Sort(sort.Reverse(semver.Versions(vers)))
	} else {
		sort.Sort(semver.Versions(vers))
	}

	out := make([]string, len(vers))
	for i := range vers {
		out[i] = prefix + vers[i].String()
	}

	return out, nil
}

```

# `/opt/kubo/repo/fsrepo/migrations/versions_test.go`

这段代码是一个 Go 语言程序，旨在测试 Istio 服务器的版本管理功能。它包含一个名为 `DistVersions` 的函数，该函数使用 HTTP 客户端从给定的 IPFS 服务器下载版本信息。下载的版本信息保存在一个名为 `testDist` 的字符串中，如果下载失败，则该字符串为空。

下载的版本信息包含一个包含版本号的新闻稿版本列表。该函数使用 `testDist` 作为测试版的 IPFS 服务器。然后，它使用一系列门牌(版本号)下载最新的五个版本。下载完成后，它打印出最新的五个版本，然后打印出 `testDist` 字符串中包含的所有版本号。

该函数的输入参数是一个 HTTP 客户端和一个 IPFS 服务器。它使用一个名为 `testDist` 的字符串作为 HTTPFS 服务器的 URL。函数在开始时创建一个 HTTP 客户端并将其连接到 IPFS 服务器。然后，它调用 `DistVersions` 函数并传递它所需的参数。如果下载成功，它打印出最新的五个版本，并将 `testDist` 字符串设置为下载的版本号。


```
package migrations

import (
	"context"
	"testing"

	"github.com/blang/semver/v4"
)

const testDist = "go-ipfs"

func TestDistVersions(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	ts := createTestServer()
	defer ts.Close()
	fetcher := NewHttpFetcher("", ts.URL, "", 0)

	vers, err := DistVersions(ctx, fetcher, testDist, true)
	if err != nil {
		t.Fatal(err)
	}
	if len(vers) == 0 {
		t.Fatal("no versions of", testDist)
	}
	t.Log("There are", len(vers), "versions of", testDist)
	t.Log("Latest 5 are:", vers[:5])
}

```

这段代码的作用是测试一个名为"LatestDistVersion"的函数。该函数使用TestLatestDistVersion作为其测试名称，并使用一个名为ctx的上下文对象和cancel函数来确保上下文中的操作在函数运行时被取消。

函数的实现部分如下：


func TestLatestDistVersion(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	ts := createTestServer()
	defer ts.Close()
	fetcher := NewHttpFetcher("", ts.URL, "", 0)

	latest, err := LatestDistVersion(ctx, fetcher, testDist, false)
	if err != nil {
		t.Fatal(err)
	}
	if len(latest) < 6 {
		t.Fatal("latest version string too short", latest)
	}
	_, err = semver.New(latest[1:])
	if err != nil {
		t.Fatal("latest version has invalid format:", latest)
	}
	t.Log("Latest version of", testDist, "is", latest)
}


首先，函数创建了一个名为ts的上下文对象和一个名为cancel的取消函数。上下文对象用于确保函数在执行时被正确地取消，而取消函数在函数运行时执行，以避免任何未取消的上下文操作对函数结果的影响。

然后，函数创建了一个http.Fetcher对象，用于从给定的测试服务器下载最新的软件版本。下载的版本字段存储在名为latest的切片结构中。

接下来，函数使用上下文对象和下载的版本字段，调用一个名为LatestDistVersion的函数。如果函数成功执行并且下载的版本字段包含至少6个元素，那么它将使用Semver库中的Latest版本函数，并打印出当前版本的名称。如果函数在下载版本时遇到任何错误，它将打印错误并退出测试。


```
func TestLatestDistVersion(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	ts := createTestServer()
	defer ts.Close()
	fetcher := NewHttpFetcher("", ts.URL, "", 0)

	latest, err := LatestDistVersion(ctx, fetcher, testDist, false)
	if err != nil {
		t.Fatal(err)
	}
	if len(latest) < 6 {
		t.Fatal("latest version string too short", latest)
	}
	_, err = semver.New(latest[1:])
	if err != nil {
		t.Fatal("latest version has invalid format:", latest)
	}
	t.Log("Latest version of", testDist, "is", latest)
}

```

# `/opt/kubo/repo/fsrepo/migrations/ipfsfetcher/ipfsfetcher.go`

该代码是一个 Go 语言项目，名为 "ipfsfetcher"，旨在从 localhost（可能是本地运行的容器）开始，下载一个名为 "default.ipfs" 的 ipfs 链接，并将其下载到本地。下载过程需要使用 libp2p（零知识证明通过子进程通信，允许你创建自定义网络）技术，并利用平行网络（并行网络允许具有相同 IP 地址的多个节点形成一个虚拟网络，有助于提高下载速度）来加速下载过程。


```
package ipfsfetcher

import (
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/url"
	"os"
	gopath "path"
	"strings"
	"sync"

	iface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/files"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/repo/fsrepo"
	"github.com/ipfs/kubo/repo/fsrepo/migrations"
	peer "github.com/libp2p/go-libp2p/core/peer"
)

```

此代码定义了一个名为`IpfsFetcher`的结构体，表示一个IPFS下载器。它包括了以下字段：

- `distPath`：下载文件的分布式根目录。
- `limit`：下载文件的最大大小，以字节为单位。
- `repoRoot`：IPFS存储库的根目录。
- `userConfigFile`：用户配置文件的路径，其中包含IPFS的选项配置。

它还包含以下字段：

- `openOnce`：打开文件操作一次的同步锁。
- `openErr`：打开文件操作错误的同步错误。
- `closeOnce`：关闭文件操作一次的同步锁。
- `closeErr`：关闭文件操作错误的同步错误。

此外，还包含以下字段：

- `ipfs`：一个IPFS下载器的实例。
- `ipfsTmpDir`：用于临时下载文件的目录，使用IPFS下载器下载文件时，将被写入该目录。
- `ipfsStopFunc`：用于停止IPFS下载器的函数。

`IpfsFetcher`还包含一个名为`fetched`的数组，用于保存下载的文件路径，以及一个名为`mutex`的同步锁，用于确保对`fetched`数组的互斥访问。


```
const (
	// Default maximum download size.
	defaultFetchLimit = 1024 * 1024 * 512

	tempNodeTCPAddr = "/ip4/127.0.0.1/tcp/0"
)

type IpfsFetcher struct {
	distPath       string
	limit          int64
	repoRoot       *string
	userConfigFile string

	openOnce  sync.Once
	openErr   error
	closeOnce sync.Once
	closeErr  error

	ipfs         iface.CoreAPI
	ipfsTmpDir   string
	ipfsStopFunc func()

	fetched []path.Path
	mutex   sync.Mutex

	addrInfo peer.AddrInfo
}

```

这段代码定义了一个名为 `NewIpfsFetcher` 的函数，它接受四个参数：`distPath`、`fetchLimit`、`repoRoot` 和 `userConfigFile`。函数返回一个指向名为 `IpfsFetcher` 的接口类型的变量。

函数的作用如下：

1. 创建一个新的 `IpfsFetcher` 对象，如果不存在，则创建一个空对象的 `*IpfsFetcher` 类型。

2. 根据传入的参数，设置 `IpfsFetcher` 对象的属性和默认值。

3. 如果 `distPath` 参数为非空字符串，那么将 `/` 添加到其前面，以使其成为有效的 IPFS 路径。

4. 如果 `fetchLimit` 参数为非负整数，则将其设置为默认值 0 或设置为负数以使它为 0。

5. 如果 `repoRoot` 参数为非空字符串，则将其设置为 IPFS 的默认目录，否则将其设置为 IPFS 目录中 `repoRoot` 目录的默认值，如果 `repoRoot` 为空字符串，则将其设置为 IPFS 目录中 `repoRoot` 目录的默认值(默认为 `nil`)。

6. 如果 `userConfigFile` 参数为非空字符串，则将其设置为 IPFS 的用户配置文件。

7. 返回新创建的 `IpfsFetcher` 对象。


```
var _ migrations.Fetcher = (*IpfsFetcher)(nil)

// NewIpfsFetcher creates a new IpfsFetcher
//
// Specifying "" for distPath sets the default IPNS path.
// Specifying 0 for fetchLimit sets the default, -1 means no limit.
//
// Bootstrap and peer information in read from the IPFS config file in
// repoRoot, unless repoRoot is nil.  If repoRoot is empty (""), then read the
// config from the default IPFS directory.
func NewIpfsFetcher(distPath string, fetchLimit int64, repoRoot *string, userConfigFile string) *IpfsFetcher {
	f := &IpfsFetcher{
		limit:          defaultFetchLimit,
		distPath:       migrations.LatestIpfsDist,
		repoRoot:       repoRoot,
		userConfigFile: userConfigFile,
	}

	if distPath != "" {
		if !strings.HasPrefix(distPath, "/") {
			distPath = "/" + distPath
		}
		f.distPath = distPath
	}

	if fetchLimit != 0 {
		if fetchLimit < 0 {
			fetchLimit = 0
		}
		f.limit = fetchLimit
	}

	return f
}

```

该代码是一个名为 `IpfsFetcher` 的函数，它使用 IPFS(InterPlanetary File System) 下载文件并返回字节数组和错误。IPFS 是一种点对点分布式文件系统，可以在全球范围内通过网络进行分布式存储和共享。该函数的作用是下载文件并将其保存在本地，然后在需要时将其上传回 IPFS 的分布网站上。

具体来说，该函数首先初始化 IPFS 节点，包括创建一个临时节点并启动 IPFS 节点，然后从 IPFS 的分布网站上下载文件并将其保存在本地。下载过程中，函数使用一个读取器来限制文件的字节数，并在文件下载完成时调用一个限速器来确保下载速度不会过快。函数还记录已经下载的文件数量，以便在需要时可以将其上传回 IPFS。

该函数的输入参数是一个表示文件路径的参数 `filePath`，它是一个字符串。函数返回一个由字节数组和错误对象组成的 slice，其中字节数组包含文件内容，错误对象可以是任何错误类型的实例。


```
// Fetch attempts to fetch the file at the given path, from the distribution
// site configured for this HttpFetcher.
func (f *IpfsFetcher) Fetch(ctx context.Context, filePath string) ([]byte, error) {
	// Initialize and start IPFS node on first call to Fetch, since the fetcher
	// may be created by not used.
	f.openOnce.Do(func() {
		bootstrap, peers := readIpfsConfig(f.repoRoot, f.userConfigFile)
		f.ipfsTmpDir, f.openErr = initTempNode(ctx, bootstrap, peers)
		if f.openErr != nil {
			return
		}

		f.openErr = f.startTempNode(ctx)
	})

	fmt.Printf("Fetching with IPFS: %q\n", filePath)

	if f.openErr != nil {
		return nil, f.openErr
	}

	iPath, err := parsePath(gopath.Join(f.distPath, filePath))
	if err != nil {
		return nil, err
	}

	nd, err := f.ipfs.Unixfs().Get(ctx, iPath)
	if err != nil {
		return nil, err
	}

	f.recordFetched(iPath)

	fileNode, ok := nd.(files.File)
	if !ok {
		return nil, fmt.Errorf("%q is not a file", filePath)
	}

	var rc io.ReadCloser
	if f.limit != 0 {
		rc = migrations.NewLimitReadCloser(fileNode, f.limit)
	} else {
		rc = fileNode
	}
	defer rc.Close()

	return io.ReadAll(rc)
}

```

该函数名为 `func`，接收一个名为 `IpfsFetcher` 的指针参数 `f`，并返回一个错误类型。

函数的作用是关闭 `IpfsFetcher` 对象 `f`，并且关闭 `IpfsFetcher` 对象中的所有子对象。先执行一次 `f.closeOnce.Do` 函数，其中 `f.closeOnce` 是 `IpfsFetcher` 对象 `f` 中的一个名为 `closeOnce` 的字段，它是一个关闭操作的配置，`Do` 函数则是配置操作的回调函数。

在 `f.closeOnce.Do` 的回调函数中，首先执行一个判断 `f.ipfsStopFunc` 是否为空，如果是，则执行一个关闭 `IpfsFetcher` 对象的 `stop` 方法，并且等待该方法执行完毕。

然后，执行一个判断 `f.ipfsTmpDir` 是否为空，如果是，则执行一个关闭 `IpfsFetcher` 对象的 `closeErr` 方法，并返回一个错误。


```
func (f *IpfsFetcher) Close() error {
	f.closeOnce.Do(func() {
		if f.ipfsStopFunc != nil {
			// Tell ipfs node to stop and wait for it to stop
			f.ipfsStopFunc()
		}

		if f.ipfsTmpDir != "" {
			// Remove the temp ipfs dir
			f.closeErr = os.RemoveAll(f.ipfsTmpDir)
		}
	})
	return f.closeErr
}

```

此代码定义了一个名为 `IpfsFetcher` 的接口，以及两个函数：`AddrInfo` 和 `FetchedPaths`。函数内部使用了 `peer.AddrInfo` 类型来存储地址信息，因此我们可以推断出此 `IpfsFetcher` 接口是作为 IPFS(InterPlanetary File System)客户端使用的。

`AddrInfo` 函数返回了 `IpfsFetcher` 接口的 `addrInfo` 字段，也就是 IPFS 中的地址信息。

`FetchedPaths` 函数返回了所有由该客户端获取到的文件的路径。由于 `IpfsFetcher` 是通过 `Fetch` 函数来获取文件的，因此此函数返回了所有已下载的文件的路径。函数使用了 `mutex`(互斥锁)来保护同一时间只有一个客户端访问文件系统，避免了多个客户端同时修改文件系统的情况。

`recordFetched` 函数接收一个下载的文件的路径，并将其添加到客户端的 `fetched` 数组中。为了确保同一时间只有一个客户端访问文件系统，此函数使用了 `mutex`(互斥锁)来防止多个客户端同时记录下载的文件。


```
func (f *IpfsFetcher) AddrInfo() peer.AddrInfo {
	return f.addrInfo
}

// FetchedPaths returns the IPFS paths of all items fetched by this fetcher.
func (f *IpfsFetcher) FetchedPaths() []path.Path {
	f.mutex.Lock()
	defer f.mutex.Unlock()
	return f.fetched
}

func (f *IpfsFetcher) recordFetched(fetchedPath path.Path) {
	// Mutex protects against update by concurrent calls to Fetch
	f.mutex.Lock()
	defer f.mutex.Unlock()
	f.fetched = append(f.fetched, fetchedPath)
}

```

该函数的作用是创建一个临时IPFS目录，并在其中初始化一个配置了IPFS路由、通过DHT客户端节点、禁止接收远程连接的IPFS节点。同时，如果使用了指定的bootstrap配置，该函数将从bootstrap列表中选择一些地址，如果选择了一些地址失败，函数将返回一个错误。如果函数在创建目录和初始化配置时出现错误，函数将返回一个错误并输出错误信息。


```
func initTempNode(ctx context.Context, bootstrap []string, peers []peer.AddrInfo) (string, error) {
	identity, err := config.CreateIdentity(io.Discard, []options.KeyGenerateOption{
		options.Key.Type(options.Ed25519Key),
	})
	if err != nil {
		return "", err
	}
	cfg, err := config.InitWithIdentity(identity)
	if err != nil {
		return "", err
	}

	// create temporary ipfs directory
	dir, err := os.MkdirTemp("", "ipfs-temp")
	if err != nil {
		return "", fmt.Errorf("failed to get temp dir: %s", err)
	}

	// configure the temporary node
	cfg.Routing.Type = config.NewOptionalString("dhtclient")

	// Disable listening for inbound connections
	cfg.Addresses.Gateway = []string{}
	cfg.Addresses.API = []string{}
	cfg.Addresses.Swarm = []string{tempNodeTCPAddr}

	if len(bootstrap) != 0 {
		cfg.Bootstrap = bootstrap
	}

	if len(peers) != 0 {
		cfg.Peering.Peers = peers
	}

	// Assumes that repo plugins are already loaded
	err = fsrepo.Init(dir, cfg)
	if err != nil {
		os.RemoveAll(dir)
		return "", fmt.Errorf("failed to initialize ephemeral node: %s", err)
	}

	return dir, nil
}

```

这段代码是一个函数，接收一个名为 `IpfsFetcher` 的 `IpfsFetcher` 类型的参数 `f`，并返回一个错误。函数的作用是启动一个 temporary IPFS node，并将其与一个已经打开的 repo 文件系统进行关联。

具体来说，函数首先打开一个名为 `ipfsTmpDir` 的临时目录，如果这个目录不存在，函数会输出一个错误并退出。然后，函数使用一个新的 `lifetime` 上下文和一个取消的 `cancel` 函数来确保在函数结束时调用 `stopFunc`。

接下来，函数使用 `coreapi` 和 `ipfs` 两个已经注册过的 `IpfsFetcher` 客户端来创建一个新的 IPFS node。函数的 `stopFunc` 是一个取消函数，用于在 IPFS node 停止时通知它停止。

最后，函数创建了一个 `node` 变量来代表 IPFS node，并将其与一个已经打开的 repo 文件系统进行关联。然后，函数返回前一步的结果，即 `nil`。


```
func (f *IpfsFetcher) startTempNode(ctx context.Context) error {
	// Open the repo
	r, err := fsrepo.Open(f.ipfsTmpDir)
	if err != nil {
		return err
	}

	// Create a new lifetime context that is used to stop the temp ipfs node
	ctxIpfsLife, cancel := context.WithCancel(context.Background())

	// Construct the node
	node, err := core.NewNode(ctxIpfsLife, &core.BuildCfg{
		Online:  true,
		Routing: libp2p.DHTClientOption,
		Repo:    r,
	})
	if err != nil {
		cancel()
		r.Close()
		return err
	}

	ipfs, err := coreapi.NewCoreAPI(node)
	if err != nil {
		cancel()
		return err
	}

	stopFunc := func() {
		// Tell ipfs to stop
		cancel()
		// Wait until ipfs is stopped
		<-node.Context().Done()
	}

	addrs, err := ipfs.Swarm().LocalAddrs(ctx)
	if err != nil {
		// Failure to get the local swarm address only means that the
		// downloaded migrations cannot be fetched through the temporary node.
		// So, print the error message and keep going.
		fmt.Fprintln(os.Stderr, "cannot get local swarm address:", err)
	}

	f.addrInfo = peer.AddrInfo{
		ID:    node.Identity,
		Addrs: addrs,
	}

	f.ipfs = ipfs
	f.ipfsStopFunc = stopFunc

	return nil
}

```

此代码定义了一个名为 `parsePath` 的函数，它接受一个字符串参数 `fetchPath`。该函数的作用是解析 `fetchPath` 并返回其对应的路径，同时返回一个错误对象。

函数首先检查 `fetchPath` 是否已经是以 IPFS 的方式编码的路径。如果是，则返回该路径，否则继续解析。如果解析过程中出现错误，函数将返回一个错误对象。

函数首先将 `fetchPath` 解析为 URL 对象，然后根据协议字符串 `proto` 判断是否为 IPFS 路径。如果是，函数将返回一个以 IPFS 路径前缀为前缀的路径，否则返回一个错误对象。


```
func parsePath(fetchPath string) (path.Path, error) {
	if ipfsPath, err := path.NewPath(fetchPath); err == nil {
		return ipfsPath, nil
	}

	u, err := url.Parse(fetchPath)
	if err != nil {
		return nil, fmt.Errorf("%q could not be parsed: %s", fetchPath, err)
	}

	switch proto := u.Scheme; proto {
	case "ipfs", "ipld", "ipns":
		return path.NewPath(gopath.Join("/", proto, u.Host, u.Path))
	default:
		return nil, fmt.Errorf("%q is not an IPFS path", fetchPath)
	}
}

```

此代码的作用是读取Ipfs Config文件中的repoRoot和userConfigFile，并返回一个bootstrap[]string和peers[]peer.AddrInfo。

函数首先检查repoRoot是否为nil，如果是，则直接返回。然后，它尝试读取用户配置文件，如果遇到错误，将打印错误并返回。

接下来，函数尝试打开并读取config文件中的配置部分。如果打开成功，它尝试使用json.NewDecoder来解析config文件中的Bootstrap和Peering部分，如果解析失败，将打印错误并返回。如果解析成功，函数将根据需要返回bootstrap和peers。


```
func readIpfsConfig(repoRoot *string, userConfigFile string) (bootstrap []string, peers []peer.AddrInfo) {
	if repoRoot == nil {
		return
	}

	cfgPath, err := config.Filename(*repoRoot, userConfigFile)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
		return
	}

	cfgFile, err := os.Open(cfgPath)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
		return
	}
	defer cfgFile.Close()

	// Attempt to read bootstrap addresses
	var bootstrapCfg struct {
		Bootstrap []string
	}
	err = json.NewDecoder(cfgFile).Decode(&bootstrapCfg)
	if err != nil {
		fmt.Fprintln(os.Stderr, "cannot read bootstrap peers from config")
	} else {
		bootstrap = bootstrapCfg.Bootstrap
	}

	if _, err = cfgFile.Seek(0, 0); err != nil {
		// If Seek fails, only log the error and continue on to try to read the
		// peering config anyway as it might still be readable
		fmt.Fprintln(os.Stderr, err)
	}

	// Attempt to read peers
	var peeringCfg struct {
		Peering config.Peering
	}
	err = json.NewDecoder(cfgFile).Decode(&peeringCfg)
	if err != nil {
		fmt.Fprintln(os.Stderr, "cannot read peering from config")
	} else {
		peers = peeringCfg.Peering.Peers
	}

	return
}

```