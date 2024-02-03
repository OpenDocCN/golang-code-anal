# `grype\cmd\grype\cli\options\match.go`

```go
// matchConfig 包含用户通过应用程序配置可用的所有与匹配相关的配置选项。
type matchConfig struct {
    Java       matcherConfig `yaml:"java" json:"java" mapstructure:"java"`                   // java匹配器的设置
    Dotnet     matcherConfig `yaml:"dotnet" json:"dotnet" mapstructure:"dotnet"`             // dotnet匹配器的设置
    Golang     golangConfig  `yaml:"golang" json:"golang" mapstructure:"golang"`             // golang匹配器的设置
    Javascript matcherConfig `yaml:"javascript" json:"javascript" mapstructure:"javascript"` // javascript匹配器的设置
    Python     matcherConfig `yaml:"python" json:"python" mapstructure:"python"`             // python匹配器的设置
    Ruby       matcherConfig `yaml:"ruby" json:"ruby" mapstructure:"ruby"`                   // ruby匹配器的设置
    Rust       matcherConfig `yaml:"rust" json:"rust" mapstructure:"rust"`                   // rust匹配器的设置
    Stock      matcherConfig `yaml:"stock" json:"stock" mapstructure:"stock"`                // 默认/stock匹配器的设置
}

// matcherConfig 包含匹配器配置选项
type matcherConfig struct {
    UseCPEs bool `yaml:"using-cpes" json:"using-cpes" mapstructure:"using-cpes"` // 是否在匹配过程中使用CPEs
}

// golangConfig 包含golang匹配器的配置选项
type golangConfig struct {
    matcherConfig         `yaml:",inline" mapstructure:",squash"`
    AlwaysUseCPEForStdlib bool `yaml:"always-use-cpe-for-stdlib" json:"always-use-cpe-for-stdlib" mapstructure:"always-use-cpe-for-stdlib"` // 是否在匹配过程中始终使用CPEs
}

// defaultGolangConfig 返回默认的golang配置
func defaultGolangConfig() golangConfig {
    return golangConfig{
        matcherConfig: matcherConfig{
            UseCPEs: false,
        },
        AlwaysUseCPEForStdlib: true,
    }
}

// defaultMatchConfig 返回默认的匹配配置
func defaultMatchConfig() matchConfig {
    useCpe := matcherConfig{UseCPEs: true}
    dontUseCpe := matcherConfig{UseCPEs: false}
}
    # 返回匹配配置的映射
    return matchConfig{
        # Java语言对应的配置为不使用CPE
        Java:       dontUseCpe,
        # Dotnet语言对应的配置为不使用CPE
        Dotnet:     dontUseCpe,
        # Golang语言对应的配置为默认Golang配置
        Golang:     defaultGolangConfig(),
        # Javascript语言对应的配置为不使用CPE
        Javascript: dontUseCpe,
        # Python语言对应的配置为不使用CPE
        Python:     dontUseCpe,
        # Ruby语言对应的配置为不使用CPE
        Ruby:       dontUseCpe,
        # Rust语言对应的配置为不使用CPE
        Rust:       dontUseCpe,
        # Stock语言对应的配置为使用CPE
        Stock:      useCpe,
    }
# 闭合前面的函数定义
```