# `grype\grype\matcher\java\maven_search.go`

```
package java

import (
    "encoding/json"  // 导入用于 JSON 编解码的包
    "errors"  // 导入用于定义错误的包
    "fmt"  // 导入用于格式化输出的包
    "net/http"  // 导入用于发送 HTTP 请求的包
    "sort"  // 导入用于排序的包

    "github.com/anchore/grype/grype/pkg"  // 导入 grype 包中的 pkg 模块
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包中的 pkg 模块
)

// MavenSearcher is the interface that wraps the GetMavenPackageBySha method.
type MavenSearcher interface {
    // GetMavenPackageBySha provides an interface for building a package from maven data based on a sha1 digest
    GetMavenPackageBySha(string) (*pkg.Package, error)  // 定义了一个接口方法，根据 sha1 哈希值构建 Maven 数据包
}

// mavenSearch implements the MavenSearcher interface
type mavenSearch struct {
    client  *http.Client  // 定义了一个 HTTP 客户端
    baseURL string  // 定义了一个基础 URL
}

type mavenAPIResponse struct {
    Response struct {
        NumFound int `json:"numFound"`  // 定义了一个 JSON 结构体字段
        Docs     []struct {  // 定义了一个 JSON 结构体切片
            ID           string `json:"id"`  // 定义了一个 JSON 结构体字段
            GroupID      string `json:"g"`  // 定义了一个 JSON 结构体字段
            ArtifactID   string `json:"a"`  // 定义了一个 JSON 结构体字段
            Version      string `json:"v"`  // 定义了一个 JSON 结构体字段
            P            string `json:"p"`  // 定义了一个 JSON 结构体字段
            VersionCount int    `json:"versionCount"`  // 定义了一个 JSON 结构体字段
        } `json:"docs"`  // 定义了一个 JSON 结构体切片
    } `json:"response"`  // 定义了一个 JSON 结构体字段
}

func (ms *mavenSearch) GetMavenPackageBySha(sha1 string) (*pkg.Package, error) {
    req, err := http.NewRequest(http.MethodGet, ms.baseURL, nil)  // 创建一个 HTTP GET 请求
    if err != nil {
        return nil, fmt.Errorf("unable to initialize HTTP client: %w", err)  // 如果请求初始化失败，则返回错误
    }

    q := req.URL.Query()  // 获取请求的查询参数
    q.Set("q", fmt.Sprintf(sha1Query, sha1))  // 设置查询参数
    q.Set("rows", "1")  // 设置查询参数
    q.Set("wt", "json")  // 设置查询参数
    req.URL.RawQuery = q.Encode()  // 将查询参数编码并设置为原始查询字符串

    resp, err := ms.client.Do(req)  // 发送 HTTP 请求
    if err != nil {
        return nil, fmt.Errorf("sha1 search error: %w", err)  // 如果发送请求出错，则返回错误
    }
    defer resp.Body.Close()  // 延迟关闭响应体

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("status %s from %s", resp.Status, req.URL.String())  // 如果响应状态码不是 200，则返回错误
    }

    var res mavenAPIResponse  // 定义一个 mavenAPIResponse 结构体变量
    if err = json.NewDecoder(resp.Body).Decode(&res); err != nil {
        return nil, fmt.Errorf("json decode error: %w", err)  // 如果 JSON 解码出错，则返回错误
    }
    # 如果返回的文档列表长度为0，则返回错误信息
    if len(res.Response.Docs) == 0:
        return nil, fmt.Errorf("digest %s: %w", sha1, errors.New("no artifact found"))

    # 文档可能具有相同的SHA-1摘要。
    # 例如 "javax.servlet:jstl" 和 "jstl:jstl"
    # 对文档列表进行排序
    docs := res.Response.Docs
    sort.Slice(docs, func(i, j int) bool {
        return docs[i].ID < docs[j].ID
    })
    # 获取排序后的第一个文档
    d := docs[0]

    # 返回一个包对象，包含名称、版本、语言和元数据
    return &pkg.Package{
        Name:     fmt.Sprintf("%s:%s", d.GroupID, d.ArtifactID),
        Version:  d.Version,
        Language: syftPkg.Java,
        Metadata: pkg.JavaMetadata{
            PomArtifactID: d.ArtifactID,
            PomGroupID:    d.GroupID,
        },
    }, nil
# 闭合前面的函数定义
```