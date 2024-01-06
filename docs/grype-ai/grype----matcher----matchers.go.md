# `grype\grype\matcher\matchers.go`

```
// 导入各种语言的匹配器包
import (
	"github.com/anchore/grype/grype/matcher/apk"  // 导入apk匹配器包
	"github.com/anchore/grype/grype/matcher/dotnet"  // 导入dotnet匹配器包
	"github.com/anchore/grype/grype/matcher/dpkg"  // 导入dpkg匹配器包
	"github.com/anchore/grype/grype/matcher/golang"  // 导入golang匹配器包
	"github.com/anchore/grype/grype/matcher/java"  // 导入java匹配器包
	"github.com/anchore/grype/grype/matcher/javascript"  // 导入javascript匹配器包
	"github.com/anchore/grype/grype/matcher/msrc"  // 导入msrc匹配器包
	"github.com/anchore/grype/grype/matcher/portage"  // 导入portage匹配器包
	"github.com/anchore/grype/grype/matcher/python"  // 导入python匹配器包
	"github.com/anchore/grype/grype/matcher/rpm"  // 导入rpm匹配器包
	"github.com/anchore/grype/grype/matcher/ruby"  // 导入ruby匹配器包
	"github.com/anchore/grype/grype/matcher/rust"  // 导入rust匹配器包
	"github.com/anchore/grype/grype/matcher/stock"  // 导入stock匹配器包
)

// Config 包含用于高级配置的各个匹配器结构体使用的值
type Config struct {
# 定义了一系列不同语言的MatcherConfig，用于匹配不同语言的配置信息
Java       java.MatcherConfig
Ruby       ruby.MatcherConfig
Python     python.MatcherConfig
Dotnet     dotnet.MatcherConfig
Javascript javascript.MatcherConfig
Golang     golang.MatcherConfig
Rust       rust.MatcherConfig
Stock      stock.MatcherConfig
}

# 根据给定的配置信息创建默认的Matchers列表
func NewDefaultMatchers(mc Config) []Matcher {
	# 返回一个包含不同语言Matcher的列表
	return []Matcher{
		&dpkg.Matcher{},  # 创建dpkg的Matcher
		ruby.NewRubyMatcher(mc.Ruby),  # 创建Ruby的Matcher
		python.NewPythonMatcher(mc.Python),  # 创建Python的Matcher
		dotnet.NewDotnetMatcher(mc.Dotnet),  # 创建Dotnet的Matcher
		&rpm.Matcher{},  # 创建rpm的Matcher
		java.NewJavaMatcher(mc.Java),  # 创建Java的Matcher
		javascript.NewJavascriptMatcher(mc.Javascript),  # 创建Javascript的Matcher
		&apk.Matcher{},  # 创建apk的Matcher
# 创建一个新的 Golang 匹配器，并将其添加到匹配器列表中
golang.NewGolangMatcher(mc.Golang),

# 创建一个新的 Msrc 匹配器，并将其添加到匹配器列表中
&msrc.Matcher{},

# 创建一个新的 Portage 匹配器，并将其添加到匹配器列表中
&portage.Matcher{},

# 创建一个新的 Rust 匹配器，并将其添加到匹配器列表中
rust.NewRustMatcher(mc.Rust),

# 创建一个新的 Stock 匹配器，并将其添加到匹配器列表中
stock.NewStockMatcher(mc.Stock),
```