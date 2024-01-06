# `grype\cmd\grype\cli\options\datasources.go`

```
package options

import (
	"github.com/anchore/grype/grype/matcher/java" // 导入 java 包
)

const (
	defaultMavenBaseURL = "https://search.maven.org/solrsearch/select" // 设置默认的 Maven 仓库基础 URL
)

type externalSources struct {
	Enable bool  `yaml:"enable" json:"enable" mapstructure:"enable"` // 外部资源是否启用的标志
	Maven  maven `yaml:"maven" json:"maven" mapstructure:"maven"` // Maven 仓库配置信息
}

type maven struct {
	SearchUpstreamBySha1 bool   `yaml:"search-upstream" json:"searchUpstreamBySha1" mapstructure:"search-maven-upstream"` // 是否通过 SHA1 值搜索上游依赖
	BaseURL              string `yaml:"base-url" json:"baseUrl" mapstructure:"base-url"` // Maven 仓库的基础 URL
}
# 返回默认的外部资源配置
func defaultExternalSources() externalSources {
	# 返回包含 Maven 配置的外部资源
	return externalSources{
		Maven: maven{
			SearchUpstreamBySha1: true,  # 根据 SHA1 值搜索上游
			BaseURL:              defaultMavenBaseURL,  # 设置默认的 Maven 基础 URL
		},
	}
}

# 将外部资源配置转换为 Java 匹配器配置
func (cfg externalSources) ToJavaMatcherConfig() java.ExternalSearchConfig {
	# 始终尊重全局配置是否禁用
	smu := cfg.Maven.SearchUpstreamBySha1  # 获取 Maven 配置中的搜索上游 SHA1 值的设置
	if !cfg.Enable {  # 如果全局配置被禁用
		smu = cfg.Enable  # 将搜索上游 SHA1 值的设置设为全局配置的启用状态
	}
	# 返回 Java 匹配器配置
	return java.ExternalSearchConfig{
		SearchMavenUpstream: smu,  # 设置搜索 Maven 上游
		MavenBaseURL:        cfg.Maven.BaseURL,  # 设置 Maven 基础 URL
	}
}
抱歉，我无法为您提供代码注释，因为您没有提供需要解释的代码。如果您有任何代码需要解释，请提供给我，我将竭诚为您服务。
```