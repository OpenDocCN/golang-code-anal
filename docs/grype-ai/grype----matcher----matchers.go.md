# `grype\grype\matcher\matchers.go`

```go
package matcher

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

// Config contains values used by individual matcher structs for advanced configuration
type Config struct {
    Java       java.MatcherConfig  // Java匹配器的配置
    Ruby       ruby.MatcherConfig  // Ruby匹配器的配置
    Python     python.MatcherConfig  // Python匹配器的配置
    Dotnet     dotnet.MatcherConfig  // Dotnet匹配器的配置
    Javascript javascript.MatcherConfig  // Javascript匹配器的配置
    Golang     golang.MatcherConfig  // Golang匹配器的配置
    Rust       rust.MatcherConfig  // Rust匹配器的配置
    Stock      stock.MatcherConfig  // Stock匹配器的配置
}

func NewDefaultMatchers(mc Config) []Matcher {
    return []Matcher{
        &dpkg.Matcher{},  // 创建并返回dpkg匹配器实例
        ruby.NewRubyMatcher(mc.Ruby),  // 创建并返回带有Ruby配置的Ruby匹配器实例
        python.NewPythonMatcher(mc.Python),  // 创建并返回带有Python配置的Python匹配器实例
        dotnet.NewDotnetMatcher(mc.Dotnet),  // 创建并返回带有Dotnet配置的Dotnet匹配器实例
        &rpm.Matcher{},  // 创建并返回rpm匹配器实例
        java.NewJavaMatcher(mc.Java),  // 创建并返回带有Java配置的Java匹配器实例
        javascript.NewJavascriptMatcher(mc.Javascript),  // 创建并返回带有Javascript配置的Javascript匹配器实例
        &apk.Matcher{},  // 创建并返回apk匹配器实例
        golang.NewGolangMatcher(mc.Golang),  // 创建并返回带有Golang配置的Golang匹配器实例
        &msrc.Matcher{},  // 创建并返回msrc匹配器实例
        &portage.Matcher{},  // 创建并返回portage匹配器实例
        rust.NewRustMatcher(mc.Rust),  // 创建并返回带有Rust配置的Rust匹配器实例
        stock.NewStockMatcher(mc.Stock),  // 创建并返回带有Stock配置的Stock匹配器实例
    }
}
```