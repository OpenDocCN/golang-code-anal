# `grype\grype\match\matcher_type.go`

```
// 定义枚举类型 MatcherType，表示匹配器的类型
const (
    UnknownMatcherType MatcherType = "UnknownMatcherType"  // 未知匹配器类型
    StockMatcher       MatcherType = "stock-matcher"       // 股票匹配器
    ApkMatcher         MatcherType = "apk-matcher"         // APK 匹配器
    RubyGemMatcher     MatcherType = "ruby-gem-matcher"    // Ruby Gem 匹配器
    DpkgMatcher        MatcherType = "dpkg-matcher"        // Dpkg 匹配器
    RpmMatcher         MatcherType = "rpm-matcher"         // Rpm 匹配器
    JavaMatcher        MatcherType = "java-matcher"        // Java 匹配器
    PythonMatcher      MatcherType = "python-matcher"      // Python 匹配器
    DotnetMatcher      MatcherType = "dotnet-matcher"      // Dotnet 匹配器
    JavascriptMatcher  MatcherType = "javascript-matcher"  // JavaScript 匹配器
    MsrcMatcher        MatcherType = "msrc-matcher"        // Msrc 匹配器
    PortageMatcher     MatcherType = "portage-matcher"     // Portage 匹配器
    GoModuleMatcher    MatcherType = "go-module-matcher"   // Go 模块匹配器
    OpenVexMatcher     MatcherType = "openvex-matcher"     // OpenVex 匹配器
    RustMatcher        MatcherType = "rust-matcher"        // Rust 匹配器
)
# 定义一个包含不同类型匹配器的数组
var AllMatcherTypes = []MatcherType{
    ApkMatcher,          # APK 包匹配器
    RubyGemMatcher,      # RubyGem 包匹配器
    DpkgMatcher,         # Dpkg 包匹配器
    RpmMatcher,          # Rpm 包匹配器
    JavaMatcher,         # Java 包匹配器
    PythonMatcher,       # Python 包匹配器
    DotnetMatcher,       # Dotnet 包匹配器
    JavascriptMatcher,   # Javascript 包匹配器
    MsrcMatcher,         # Msrc 包匹配器
    PortageMatcher,      # Portage 包匹配器
    GoModuleMatcher,     # GoModule 包匹配器
    OpenVexMatcher,      # OpenVex 包匹配器
    RustMatcher,         # Rust 包匹配器
}

# 定义一个匹配器类型
type MatcherType string
```