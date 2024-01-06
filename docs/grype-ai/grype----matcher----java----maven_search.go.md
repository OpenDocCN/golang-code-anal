# `grype\grype\matcher\java\maven_search.go`

```
// 导入所需的包
package java

import (
	"encoding/json"  // 导入 JSON 编解码包
	"errors"  // 导入错误处理包
	"fmt"  // 导入格式化输出包
	"net/http"  // 导入 HTTP 请求包
	"sort"  // 导入排序包

	"github.com/anchore/grype/grype/pkg"  // 导入 grype 包中的 pkg 模块
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包中的 pkg 模块
)

// MavenSearcher 是一个接口，包含了 GetMavenPackageBySha 方法
type MavenSearcher interface {
	// GetMavenPackageBySha 根据 sha1 摘要构建一个 maven 数据包
	GetMavenPackageBySha(string) (*pkg.Package, error)
}

// mavenSearch 实现了 MavenSearcher 接口
// 定义一个结构体 mavenSearch，包含一个指向 http 客户端的指针和一个基本 URL 字符串
type mavenSearch struct {
	client  *http.Client
	baseURL string
}

// 定义一个结构体 mavenAPIResponse，包含一个 Response 结构体，其中包含 numFound 和 docs 两个字段
type mavenAPIResponse struct {
	Response struct {
		NumFound int `json:"numFound"`
		Docs     []struct {
			ID           string `json:"id"`
			GroupID      string `json:"g"`
			ArtifactID   string `json:"a"`
			Version      string `json:"v"`
			P            string `json:"p"`
			VersionCount int    `json:"versionCount"`
		} `json:"docs"`
	} `json:"response"`
}

// 定义一个方法，接收一个 mavenSearch 结构体的指针和一个字符串参数 sha1，返回一个 pkg.Package 类型的指针和一个错误
func (ms *mavenSearch) GetMavenPackageBySha(sha1 string) (*pkg.Package, error) {
	// 创建一个新的 HTTP 请求，使用 GET 方法，指定 URL 为 ms.baseURL
	req, err := http.NewRequest(http.MethodGet, ms.baseURL, nil)
	// 如果创建请求时出现错误，返回错误信息
	if err != nil {
		return nil, fmt.Errorf("unable to initialize HTTP client: %w", err)
	}

	// 获取请求的 URL 查询参数
	q := req.URL.Query()
	// 设置查询参数 "q" 为格式化后的 sha1Query
	q.Set("q", fmt.Sprintf(sha1Query, sha1))
	// 设置查询参数 "rows" 为 "1"
	q.Set("rows", "1")
	// 设置查询参数 "wt" 为 "json"
	q.Set("wt", "json")
	// 将设置后的查询参数重新编码并设置为请求的原始查询字符串
	req.URL.RawQuery = q.Encode()

	// 发送 HTTP 请求并获取响应
	resp, err := ms.client.Do(req)
	// 如果发送请求时出现错误，返回错误信息
	if err != nil {
		return nil, fmt.Errorf("sha1 search error: %w", err)
	}
	// 在函数返回前关闭响应的主体
	defer resp.Body.Close()

	// 如果响应的状态码不是 200 OK，返回错误信息
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("status %s from %s", resp.Status, req.URL.String())
	}
// 使用json解码器将响应体解析为mavenAPIResponse结构体
var res mavenAPIResponse
if err = json.NewDecoder(resp.Body).Decode(&res); err != nil {
    return nil, fmt.Errorf("json decode error: %w", err)
}

// 如果响应中没有文档，则返回错误
if len(res.Response.Docs) == 0 {
    return nil, fmt.Errorf("digest %s: %w", sha1, errors.New("no artifact found"))
}

// 对文档进行排序，以便找到具有相同SHA-1摘要的构件
// 例如 "javax.servlet:jstl" 和 "jstl:jstl"
docs := res.Response.Docs
sort.Slice(docs, func(i, j int) bool {
    return docs[i].ID < docs[j].ID
})
d := docs[0]

// 返回一个包含名称的Package结构体，格式为"groupID:artifactID"
return &pkg.Package{
    Name:     fmt.Sprintf("%s:%s", d.GroupID, d.ArtifactID),
		// 设置版本号为d.Version
		Version:  d.Version,
		// 设置语言为syftPkg.Java
		Language: syftPkg.Java,
		// 设置元数据为pkg.JavaMetadata
		Metadata: pkg.JavaMetadata{
			// 设置PomArtifactID为d.ArtifactID
			PomArtifactID: d.ArtifactID,
			// 设置PomGroupID为d.GroupID
			PomGroupID:    d.GroupID,
		},
		// 返回结果和空错误
	}, nil
```