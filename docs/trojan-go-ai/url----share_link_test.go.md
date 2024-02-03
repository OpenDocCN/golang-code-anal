# `trojan-go\url\share_link_test.go`

```go
// 导入所需的包
package url

import (
    crand "crypto/rand"  // 导入加密随机数生成包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "io/ioutil"  // 导入输入输出工具包
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包
)

// 测试处理特洛伊木马端口默认情况
func TestHandleTrojanPort_Default(t *testing.T) {
    port, e := handleTrojanPort("")  // 调用处理特洛伊木马端口函数，传入空字符串
    assert.Nil(t, e, "empty port should not error")  // 断言空端口不应该出错
    assert.EqualValues(t, 443, port, "empty port should fallback to 443")  // 断言空端口应该回退到443
}

// 测试处理特洛伊木马端口非数字情况
func TestHandleTrojanPort_NotNumber(t *testing.T) {
    _, e := handleTrojanPort("fuck")  // 调用处理特洛伊木马端口函数，传入非数字字符串
    assert.Error(t, e, "non-numerical port should error")  // 断言非数字端口应该出错
}

// 测试处理特洛伊木马端口合法数字情况
func TestHandleTrojanPort_GoodNumber(t *testing.T) {
    testCases := []string{"443", "8080", "10086", "80", "65535", "1"}  // 定义合法端口号测试用例
    for _, testCase := range testCases {
        _, e := handleTrojanPort(testCase)  // 遍历测试用例，调用处理特洛伊木马端口函数
        assert.Nil(t, e, "good port %s should not error", testCase)  // 断言合法端口不应该出错
    }
}

// 测试处理特洛伊木马端口非法数字情况
func TestHandleTrojanPort_InvalidNumber(t *testing.T) {
    testCases := []string{"443.0", "443.000", "8e2", "3.5", "9.99", "-1", "-65535", "65536", "0"}  // 定义非法端口号测试用例

    for _, testCase := range testCases {
        _, e := handleTrojanPort(testCase)  // 遍历测试用例，调用处理特洛伊木马端口函数
        assert.Error(t, e, "invalid number port %s should error", testCase)  // 断言非法端口应该出错
    }
}

// 测试从URL创建新的分享信息为空情况
func TestNewShareInfoFromURL_Empty(t *testing.T) {
    _, e := NewShareInfoFromURL("")  // 调用从URL创建新的分享信息函数，传入空字符串
    assert.Error(t, e, "empty link should lead to error")  // 断言空链接应该导致错误
}

// 测试从URL创建新的分享信息为随机字符串情况
func TestNewShareInfoFromURL_RandomCrap(t *testing.T) {
    for i := 0; i < 100; i++ {
        randomCrap, _ := ioutil.ReadAll(io.LimitReader(crand.Reader, 10))  // 生成随机字符串
        _, e := NewShareInfoFromURL(string(randomCrap))  // 调用从URL创建新的分享信息函数，传入随机字符串
        assert.Error(t, e, "random crap %v should lead to error", randomCrap)  // 断言随机字符串应该导致错误
    }
}

// 测试从URL创建新的分享信息为非特洛伊木马链接情况
func TestNewShareInfoFromURL_NotTrojanGo(t *testing.T) {
    # 定义测试用例数组，包含三个待测试的链接
    testCases := []string{
        "trojan://what.ever@www.twitter.com:443?allowInsecure=1&allowInsecureHostname=1&allowInsecureCertificate=1&sessionTicket=0&tfo=1#some-trojan",
        "ssr://d3d3LnR3aXR0ZXIuY29tOjgwOmF1dGhfc2hhMV92NDpjaGFjaGEyMDpwbGFpbjpZbkpsWVd0M1lXeHMvP29iZnNwYXJhbT0mcmVtYXJrcz02TC1INXB5ZjVwZTI2WmUwNzd5YU1qQXlNQzB3TnkweE9DQXhNam8xTlRveU1RJmdyb3VwPVEzUkRiRzkxWkNCVFUxSQ",
        "vmess://eyJhZGQiOiJtb3RoZXIuZnVja2VyIiwiYWlkIjowLCJpZCI6IjFmYzI0NzVmLThmNDMtM2FlYi05MzUyLTU2MTFhZjg1NmQyOSIsIm5ldCI6InRjcCIsInBvcnQiOjEwMDg2LCJwcyI6Iui/h+acn+aXtumXtO+8mjIwMjAtMDYtMjMiLCJ0bHMiOiJub25lIiwidHlwZSI6Im5vbmUiLCJ2IjoyfQ==",
    }

    # 遍历测试用例数组
    for _, testCase := range testCases {
        # 调用 NewShareInfoFromURL 函数，传入测试用例链接，获取返回值和错误信息
        _, e := NewShareInfoFromURL(testCase)
        # 使用断言判断是否有错误，如果有错误则输出错误信息
        assert.Error(t, e, "non trojan-go link %s should not decode", testCase)
    }
# 测试当 Trojan 主机为空时，是否能正确返回错误
func TestNewShareInfoFromURL_EmptyTrojanHost(t *testing.T):
    _, e := NewShareInfoFromURL("trojan-go://fuckyou@:443/")
    assert.Error(t, e, "empty host should not decode")

# 测试当密码格式错误时，是否能正确返回错误
func TestNewShareInfoFromURL_BadPassword(t *testing.T):
    testCases := []string{
        "trojan-go://we:are:the:champion@114514.go",
        "trojan-go://evilpassword:@1919810.me",
        "trojan-go://evilpassword::@1919810.me",
        "trojan-go://@password.404",
        "trojan-go://mother.fuck#yeah",
    }

    for _, testCase := range testCases:
        _, e := NewShareInfoFromURL(testCase)
        assert.Error(t, e, "bad password link %s should not decode", testCase)

# 测试当密码格式正确时，是否能正确解析
func TestNewShareInfoFromURL_GoodPassword(t *testing.T):
    testCases := []string{
        "trojan-go://we%3Aare%3Athe%3Achampion@114514.go",
        "trojan-go://evilpassword%3A@1919810.me",
        "trojan-go://passw0rd-is-a-must@password.200",
    }

    for _, testCase := range testCases:
        _, e := NewShareInfoFromURL(testCase)
        assert.Nil(t, e, "good password link %s should decode", testCase)

# 测试当端口格式错误时，是否能正确返回错误
func TestNewShareInfoFromURL_BadPort(t *testing.T):
    testCases := []string{
        "trojan-go://pswd@example.com:114514",
        "trojan-go://pswd@example.com:443.0",
        "trojan-go://pswd@example.com:-1",
        "trojan-go://pswd@example.com:8e2",
        "trojan-go://pswd@example.com:65536",
    }

    for _, testCase := range testCases:
        _, e := NewShareInfoFromURL(testCase)
        assert.Error(t, e, "decode url %s with invalid port should error", testCase)

# 测试当查询参数格式错误时，是否能正确返回错误
func TestNewShareInfoFromURL_BadQuery(t *testing.T):
    testCases := []string{
        "trojan-go://cao@ni.ma?NMSL=%CG%GE%CAONIMA",
        "trojan-go://ni@ta.ma:13/?#%2e%fu",
    }

    for _, testCase := range testCases:
        _, e := NewShareInfoFromURL(testCase)
        assert.Error(t, e, "parse bad query should error")

# 测试当 SNI 为空时，是否能正确处理
func TestNewShareInfoFromURL_SNI_Empty(t *testing.T):
    # 使用 NewShareInfoFromURL 函数从指定的 URL 创建共享信息对象，_ 表示忽略第一个返回值
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?sni=")
    # 断言 e 是否为错误，如果不是则输出指定的错误信息
    assert.Error(t, e, "empty SNI should not be allowed")
// 测试从 URL 中创建 ShareInfo 对象，SNI 默认情况
func TestNewShareInfoFromURL_SNI_Default(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象
    info, e := NewShareInfoFromURL("trojan-go://a@b.c")
    // 断言错误为空
    assert.Nil(t, e)
    // 断言 trojan 主机名等于 SNI，即默认情况下 SNI 应该是 trojan 主机名
    assert.Equal(t, info.TrojanHost, info.SNI, "default sni should be trojan hostname")
}

// 测试从 URL 中创建 ShareInfo 对象，SNI 多个情况
func TestNewShareInfoFromURL_SNI_Multiple(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象，包含多个 SNI 参数
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?sni=a&sni=b&sni=c")
    // 断言应该出现错误，因为不允许有多个 SNI 参数
    assert.Error(t, e, "multiple SNIs should not be allowed")
}

// 测试从 URL 中创建 ShareInfo 对象，类型为空情况
func TestNewShareInfoFromURL_Type_Empty(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象，类型为空
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=")
    // 断言应该出现错误，因为空类型不允许
    assert.Error(t, e, "empty type should not be allowed")
}

// 测试从 URL 中创建 ShareInfo 对象，类型默认情况
func TestNewShareInfoFromURL_Type_Default(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象
    info, e := NewShareInfoFromURL("trojan-go://a@b.c")
    // 断言错误为空
    assert.Nil(t, e)
    // 断言类型等于原始类型，即默认情况下类型应该是原始类型
    assert.Equal(t, ShareInfoTypeOriginal, info.Type, "default type should be original")
}

// 测试从 URL 中创建 ShareInfo 对象，类型无效情况
func TestNewShareInfoFromURL_Type_Invalid(t *testing.T) {
    // 无效的类型列表
    invalidTypes := []string{"nmsl", "dio"}
    for _, invalidType := range invalidTypes {
        // 从 URL 中创建 ShareInfo 对象，类型为无效类型
        _, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?type=%s", invalidType))
        // 断言应该出现错误，因为无效类型不允许
        assert.Error(t, e, "%s should not be a valid type", invalidType)
    }
}

// 测试从 URL 中创建 ShareInfo 对象，类型多个情况
func TestNewShareInfoFromURL_Type_Multiple(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象，包含多个类型参数
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=a&type=b&type=c")
    // 断言应该出现错误，因为不允许有多个类型参数
    assert.Error(t, e, "multiple types should not be allowed")
}

// 测试从 URL 中创建 ShareInfo 对象，主机名为空情况
func TestNewShareInfoFromURL_Host_Empty(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象，主机名为空
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?host=")
    // 断言应该出现错误，因为空主机名不允许
    assert.Error(t, e, "empty host should not be allowed")
}

// 测试从 URL 中创建 ShareInfo 对象，主机名默认情况
func TestNewShareInfoFromURL_Host_Default(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象
    info, e := NewShareInfoFromURL("trojan-go://a@b.c")
    // 断言错误为空
    assert.Nil(t, e)
    // 断言主机名等于 trojan 主机名，即默认情况下主机名应该是 trojan 主机名
    assert.Equal(t, info.TrojanHost, info.Host, "default host should be trojan hostname")
}

// 测试从 URL 中创建 ShareInfo 对象，主机名多个情况
func TestNewShareInfoFromURL_Host_Multiple(t *testing.T) {
    // 从 URL 中创建 ShareInfo 对象，包含多个主机名参数
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?host=a&host=b&host=c")
    // 断言应该出现错误，因为不允许有多个主机名参数
    assert.Error(t, e, "multiple hosts should not be allowed")
}
# 测试从 URL 中创建新的分享信息，类型为 WS，包含多个路径，预期会出错
func TestNewShareInfoFromURL_Type_WS_Multiple(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入带有多个路径的 URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws&path=a&path=b&path=c")
    assert.Error(t, e, "multiple paths should not be allowed in wss")

# 测试从 URL 中创建新的分享信息，类型为 WS，不包含路径，预期会出错
func TestNewShareInfoFromURL_Path_WS_None(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入不包含路径的 URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws")
    assert.Error(t, e, "ws should require path")

# 测试从 URL 中创建新的分享信息，类型为 WS，包含空路径，预期会出错
func TestNewShareInfoFromURL_Path_WS_Empty(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入包含空路径的 URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws&path=")
    assert.Error(t, e, "empty path should not be allowed in ws")

# 测试从 URL 中创建新的分享信息，类型为 WS，包含无效路径，预期会出错
func TestNewShareInfoFromURL_Path_WS_Invalid(t *testing.T):
    # 定义无效路径列表
    invalidPaths := []string{"../", ".+!", " "}
    # 遍历无效路径列表
    for _, invalidPath := range invalidPaths:
        # 调用 NewShareInfoFromURL 函数，传入包含无效路径的 URL，期望返回错误
        _, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?type=ws&path=%s", invalidPath))
        assert.Error(t, e, "%s should not be a valid path in ws", invalidPath)

# 测试从 URL 中创建新的分享信息，类型为原始，包含空路径，预期不会出错
func TestNewShareInfoFromURL_Path_Plain_Empty(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入包含空路径的 URL，期望不返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?type=original&path=")
    assert.Nil(t, e, "empty path should be ignored in original mode")

# 测试从 URL 中创建新的分享信息，加密方式为空，预期会出错
func TestNewShareInfoFromURL_Encryption_Empty(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入加密方式为空的 URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=")
    assert.Error(t, e, "encryption should not be empty")

# 测试从 URL 中创建新的分享信息，加密方式为未知，预期会出错
func TestNewShareInfoFromURL_Encryption_Unknown(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入加密方式为未知的 URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=motherfucker")
    assert.Error(t, e, "unknown encryption should not be supported")

# 测试从 URL 中创建新的分享信息，加密方式为 none，预期不会出错
func TestNewShareInfoFromURL_Encryption_None(t *testing.T):
    # 调用 NewShareInfoFromURL 函数，传入加密方式为 none 的 URL，期望不返回错误
    _, e := NewShareInfoFromURL("trojan-go://what@ever.me?encryption=none")
    assert.Nil(t, e, "should support none encryption")

# 测试从 URL 中创建新的分享信息，加密方式为 SS，包含不支持的加密方法，预期会出错
func TestNewShareInfoFromURL_Encryption_SS_NotSupportedMethods(t *testing.T):
    # 定义不支持的加密方法列表
    invalidMethods := []string{"rc4-md5", "rc4", "des-cfb", "table", "salsa20-ctr"}
    # 遍历无效的加密方法列表
    for _, invalidMethod := range invalidMethods {
        # 使用无效的加密方法创建分享信息对象
        _, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?encryption=ss%%3B%s%%3Ashabi", invalidMethod))
        # 断言应该出现错误，提示该加密方法不应该被 ss 支持
        assert.Error(t, e, "encryption %s should not be supported by ss", invalidMethod)
    }
# 测试函数：从URL中创建新的共享信息，加密方式为SS，无密码
func TestNewShareInfoFromURL_Encryption_SS_NoPassword(t *testing.T):
    # 调用NewShareInfoFromURL函数，传入指定的URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=ss%3Baes-256-gcm%3A")
    # 断言是否返回错误，错误信息为"empty ss password should not be allowed"
    assert.Error(t, e, "empty ss password should not be allowed")

# 测试函数：从URL中创建新的共享信息，加密方式为SS，参数错误
func TestNewShareInfoFromURL_Encryption_SS_BadParams(t *testing.T):
    # 调用NewShareInfoFromURL函数，传入指定的URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=ss%3Ba")
    # 断言是否返回错误，错误信息为"broken ss param should not be allowed"
    assert.Error(t, e, "broken ss param should not be allowed")

# 测试函数：从URL中创建新的共享信息，多重加密
func TestNewShareInfoFromURL_Encryption_Multiple(t *testing.T):
    # 调用NewShareInfoFromURL函数，传入指定的URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=a&encryption=b&encryption=c")
    # 断言是否返回错误，错误信息为"multiple encryption should not be allowed"
    assert.Error(t, e, "multiple encryption should not be allowed")

# 测试函数：从URL中创建新的共享信息，插件为空
func TestNewShareInfoFromURL_Plugin_Empty(t *testing.T):
    # 调用NewShareInfoFromURL函数，传入指定的URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?plugin=")
    # 断言是否返回错误，错误信息为"plugin should not be empty"
    assert.Error(t, e, "plugin should not be empty")

# 测试函数：从URL中创建新的共享信息，多重插件
func TestNewShareInfoFromURL_Plugin_Multiple(t *testing.T):
    # 调用NewShareInfoFromURL函数，传入指定的URL，期望返回错误
    _, e := NewShareInfoFromURL("trojan-go://a@b.c?plugin=a&plugin=b&plugin=c")
    # 断言是否返回错误，错误信息为"multiple plugin should not be allowed"
    assert.Error(t, e, "multiple plugin should not be allowed")
```