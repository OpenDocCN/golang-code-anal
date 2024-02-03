# `grype\cmd\grype\cli\options\datasources.go`

```go
package options

import (
    "github.com/anchore/grype/grype/matcher/java"
)

const (
    defaultMavenBaseURL = "https://search.maven.org/solrsearch/select"
)

type externalSources struct {
    Enable bool  `yaml:"enable" json:"enable" mapstructure:"enable"`  // 外部资源是否启用的标志
    Maven  maven `yaml:"maven" json:"maven" mapstructure:"maven"`  // Maven 仓库配置
}

type maven struct {
    SearchUpstreamBySha1 bool   `yaml:"search-upstream" json:"searchUpstreamBySha1" mapstructure:"search-maven-upstream"`  // 是否通过 SHA1 搜索上游
    BaseURL              string `yaml:"base-url" json:"baseUrl" mapstructure:"base-url"`  // Maven 仓库的基本 URL
}

func defaultExternalSources() externalSources {
    return externalSources{
        Maven: maven{
            SearchUpstreamBySha1: true,  // 默认启用通过 SHA1 搜索上游
            BaseURL:              defaultMavenBaseURL,  // 默认 Maven 仓库的基本 URL
        },
    }
}

func (cfg externalSources) ToJavaMatcherConfig() java.ExternalSearchConfig {
    // 总是尊重全局配置是否禁用
    smu := cfg.Maven.SearchUpstreamBySha1
    if !cfg.Enable {
        smu = cfg.Enable
    }
    return java.ExternalSearchConfig{
        SearchMavenUpstream: smu,  // 返回 Java 匹配器的外部搜索配置
        MavenBaseURL:        cfg.Maven.BaseURL,  // 返回 Java 匹配器的 Maven 仓库基本 URL
    }
}
```