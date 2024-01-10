# `grype\grype\match\matcher_type.go`

```
package match

// 定义匹配器类型常量
const (
    UnknownMatcherType MatcherType = "UnknownMatcherType"
    StockMatcher       MatcherType = "stock-matcher"
    ApkMatcher         MatcherType = "apk-matcher"
    RubyGemMatcher     MatcherType = "ruby-gem-matcher"
    DpkgMatcher        MatcherType = "dpkg-matcher"
    RpmMatcher         MatcherType = "rpm-matcher"
    JavaMatcher        MatcherType = "java-matcher"
    PythonMatcher      MatcherType = "python-matcher"
    DotnetMatcher      MatcherType = "dotnet-matcher"
    JavascriptMatcher  MatcherType = "javascript-matcher"
    MsrcMatcher        MatcherType = "msrc-matcher"
    PortageMatcher     MatcherType = "portage-matcher"
    GoModuleMatcher    MatcherType = "go-module-matcher"
    OpenVexMatcher     MatcherType = "openvex-matcher"
    RustMatcher        MatcherType = "rust-matcher"
)

// 定义所有匹配器类型的切片
var AllMatcherTypes = []MatcherType{
    ApkMatcher,
    RubyGemMatcher,
    DpkgMatcher,
    RpmMatcher,
    JavaMatcher,
    PythonMatcher,
    DotnetMatcher,
    JavascriptMatcher,
    MsrcMatcher,
    PortageMatcher,
    GoModuleMatcher,
    OpenVexMatcher,
    RustMatcher,
}

// 定义匹配器类型
type MatcherType string
```