# `grype\cmd\grype\cli\options\match.go`

```
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
	matcherConfig         `yaml:",inline" mapstructure:",squash"` // 内联匹配器配置选项
}
// AlwaysUseCPEForStdlib 表示是否始终使用CPE来匹配标准库
// yaml:"always-use-cpe-for-stdlib" 表示在yaml配置文件中的字段名
// json:"always-use-cpe-for-stdlib" 表示在json配置文件中的字段名
// mapstructure:"always-use-cpe-for-stdlib" 表示在map结构中的字段名

func defaultGolangConfig() golangConfig {
	// 返回默认的Golang配置
	return golangConfig{
		matcherConfig: matcherConfig{
			UseCPEs: false, // 默认不使用CPEs进行匹配
		},
		AlwaysUseCPEForStdlib: true, // 默认始终使用CPE来匹配标准库
	}
}

func defaultMatchConfig() matchConfig {
	useCpe := matcherConfig{UseCPEs: true} // 使用CPE进行匹配
	dontUseCpe := matcherConfig{UseCPEs: false} // 不使用CPE进行匹配
	return matchConfig{
		Java:       dontUseCpe, // Java默认不使用CPE进行匹配
		Dotnet:     dontUseCpe, // Dotnet默认不使用CPE进行匹配
		Golang:     defaultGolangConfig(), // Golang使用默认的Golang配置
		Javascript: dontUseCpe, // Javascript默认不使用CPE进行匹配
	}
}
# 定义一个包含不同编程语言和对应使用或不使用CPE的字典
{
    # Python不使用CPE
    Python: dontUseCpe,
    # Ruby不使用CPE
    Ruby: dontUseCpe,
    # Rust不使用CPE
    Rust: dontUseCpe,
    # Stock使用CPE
    Stock: useCpe,
}
```