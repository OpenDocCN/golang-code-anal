# `kubebench-aquasecurity\check\test_test.go`

```
// 版权声明，版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0版本授权，除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 导入所需的包
package check

import (
    "fmt" // 导入fmt包，用于格式化输入输出
    "io/ioutil" // 导入io/ioutil包，用于读取文件内容
    "os" // 导入os包，用于操作文件和目录
// 导入 "strings" 和 "testing" 包
"strings"
"testing"
)

// 定义全局变量
var (
	in       []byte
	controls *Controls
)

// 初始化函数
func init() {
	// 读取文件内容到变量 in
	var err error
	in, err = ioutil.ReadFile("data")
	if err != nil {
		// 如果读取文件失败，抛出错误
		panic("Failed reading test data: " + err.Error())
	}

	// 从环境变量中获取 USER 变量的值
	user := os.Getenv("USER")
	// 替换文件内容中的 "$user" 变量为实际的用户值
	s := strings.Replace(string(in), "$user", user, -1)
// 使用给定的字符串创建控制对象，返回控制对象和错误信息
controls, err = NewControls(MASTER, []byte(s))
// 如果创建控制对象时发生错误，抛出异常并打印错误信息
if err != nil {
    panic("Failed creating test controls: " + err.Error())
}

func TestTestExecute(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        *Check
        str       string
        strConfig string
        strEnv    string
    }{
        {
            // 从控制对象中获取第一个分组的第一个检查项
            controls.Groups[0].Checks[0],
            // 设置测试字符串
            "2:45 ../kubernetes/kube-apiserver --allow-privileged=false --option1=20,30,40",
            // 设置配置字符串为空
            "",
            // 设置环境变量字符串为空
            "",
# 控制组的第一个检查项，设置命令和参数
{
    controls.Groups[0].Checks[1],
    "2:45 ../kubernetes/kube-apiserver --allow-privileged=false",
    "",
    "",
},
# 控制组的第二个检查项，设置命令和参数
{
    controls.Groups[0].Checks[2],
    "niinai   13617  2635 99 19:26 pts/20   00:03:08 ./kube-apiserver --insecure-port=0 --anonymous-auth",
    "",
    "",
},
# 控制组的第三个检查项，设置命令和参数
{
    controls.Groups[0].Checks[3],
    "2:45 ../kubernetes/kube-apiserver --secure-port=0 --audit-log-maxage=40 --option",
    "",
    "",
},
# 对 controls.Groups[0].Checks[4] 进行操作
# 使用 "2:45 ../kubernetes/kube-apiserver --max-backlog=20 --secure-port=0 --audit-log-maxage=40 --option" 进行操作
# 空字符串
# 空字符串
{
    controls.Groups[0].Checks[4],
    "2:45 ../kubernetes/kube-apiserver --max-backlog=20 --secure-port=0 --audit-log-maxage=40 --option",
    "",
    "",
},
# 对 controls.Groups[0].Checks[5] 进行操作
# 使用 "2:45 ../kubernetes/kube-apiserver --option --admission-control=WebHook,RBAC ---audit-log-maxage=40" 进行操作
# 空字符串
# 空字符串
{
    controls.Groups[0].Checks[5],
    "2:45 ../kubernetes/kube-apiserver --option --admission-control=WebHook,RBAC ---audit-log-maxage=40",
    "",
    "",
},
# 对 controls.Groups[0].Checks[6] 进行操作
# 使用 "2:45 .. --kubelet-clientkey=foo --kubelet-client-certificate=bar --admission-control=Webhook,RBAC" 进行操作
# 空字符串
# 空字符串
{
    controls.Groups[0].Checks[6],
    "2:45 .. --kubelet-clientkey=foo --kubelet-client-certificate=bar --admission-control=Webhook,RBAC",
    "",
    "",
},
# 对 controls.Groups[0].Checks[7] 进行操作
# 使用 "2:45 ..  --secure-port=0 --kubelet-client-certificate=bar --admission-control=Webhook,RBAC" 进行操作
# 空字符串
"",
"",
# 控制组0的第8个检查项
{
    controls.Groups[0].Checks[8],
    "644",
    "",
    "",
},
# 控制组0的第9个检查项
{
    controls.Groups[0].Checks[9],
    "640",
    "",
    "",
},
# 控制组0的第9个检查项
{
    controls.Groups[0].Checks[9],
    "600",
    "",
    "",
}
# 控制组中的第一个检查项，设置命令和参数
{
    controls.Groups[0].Checks[10],
    "2:45 ../kubernetes/kube-apiserver --option --admission-control=WebHook,RBAC ---audit-log-maxage=40",
    "",
    "",
},
# 控制组中的第二个检查项，设置命令和参数
{
    controls.Groups[0].Checks[11],
    "2:45 ../kubernetes/kube-apiserver --option --admission-control=WebHook,RBAC ---audit-log-maxage=40",
    "",
    "",
},
# 控制组中的第三个检查项，设置命令和参数
{
    controls.Groups[0].Checks[12],
    "2:45 ../kubernetes/kube-apiserver --option --admission-control=WebHook,Something,RBAC ---audit-log-maxage=40",
    "",
    "",
},
// 使用 controls.Groups[0].Checks[13] 进行检查
// 参数为 "2:45 ../kubernetes/kube-apiserver --option --admission-control=Something ---audit-log-maxage=40"
// 无额外信息
// 无额外信息
{
    // 使用 controls.Groups[0].Checks[14] 进行检查
    // 参数为 "2:45 kube-apiserver some-arg: some-val --admission-control=Something ---audit-log-maxage=40"
    // 无额外信息
    // 无额外信息
},
{
    // 使用 controls.Groups[0].Checks[14] 进行检查
    // 参数为 "2:45 kube-apiserver some-arg:some-val --admission-control=Something ---audit-log-maxage=40"
    // 无额外信息
    // 无额外信息
},
# 访问 controls 对象的第一个分组的第 16 个检查项，并传入空字符串和指定的 JSON 格式参数
{
    controls.Groups[0].Checks[15],
    "",
    "{\"readOnlyPort\": 15000}",
    "",
},
# 访问 controls 对象的第一个分组的第 17 个检查项，并传入空字符串和指定的 JSON 格式参数
{
    controls.Groups[0].Checks[16],
    "",
    "{\"stringValue\": \"WebHook,Something,RBAC\"}",
    "",
},
# 访问 controls 对象的第一个分组的第 18 个检查项，并传入空字符串和指定的 JSON 格式参数
{
    controls.Groups[0].Checks[17],
    "",
    "{\"trueValue\": true}",
    "",
},
# 访问 controls 对象的第一个分组的第 19 个检查项，并传入空字符串
{
    controls.Groups[0].Checks[18],
    "",
# 第一个元素：设置只读端口为15000
{
    controls.Groups[0].Checks[19],  # 获取第一个组的第19个检查
    "",  # 空字符串
    "{\"readOnlyPort\": 15000}",  # 设置只读端口为15000的JSON格式字符串
    "",  # 空字符串
},
# 第二个元素：禁用匿名用户认证
{
    controls.Groups[0].Checks[19],  # 获取第一个组的第19个检查
    "",  # 空字符串
    "{\"authentication\": { \"anonymous\": {\"enabled\": false}}}",  # 设置禁用匿名用户认证的JSON格式字符串
    "",  # 空字符串
},
# 第三个元素：设置只读端口为15000
{
    controls.Groups[0].Checks[20],  # 获取第一个组的第20个检查
    "",  # 空字符串
    "readOnlyPort: 15000",  # 设置只读端口为15000的字符串
    "",  # 空字符串
},
# 第四个元素：设置只读端口为15000
{
    controls.Groups[0].Checks[21],  # 获取第一个组的第21个检查
    "",  # 空字符串
    "readOnlyPort: 15000",  # 设置只读端口为15000的字符串
    "",  # 空字符串
},
# 设置控件组中第一个检查的属性
{
    controls.Groups[0].Checks[22],  # 控件组中第一个检查的属性
    "",  # 空字符串
    "authentication:\n  anonymous:\n    enabled: false",  # 设置认证方式为禁用匿名用户
    "",  # 空字符串
},
# 设置控件组中第一个检查的属性
{
    controls.Groups[0].Checks[26],  # 控件组中第一个检查的属性
    "",  # 空字符串
    "currentMasterVersion: 1.12.7",  # 设置当前主版本号为1.12.7
    "",  # 空字符串
},
# 设置控件组中第一个检查的属性
{
    controls.Groups[0].Checks[27],  # 控件组中第一个检查的属性
    "--peer-client-cert-auth",  # 设置属性为--peer-client-cert-auth
    "",  # 空字符串
    "",  # 空字符串
},
# 对 controls.Groups[0].Checks[27] 进行设置，传入参数 "--abc=true --peer-client-cert-auth --efg=false"，其它参数为空
{
    controls.Groups[0].Checks[27],
    "--abc=true --peer-client-cert-auth --efg=false",
    "",
    "",
},
# 对 controls.Groups[0].Checks[27] 进行设置，传入参数 "--abc --peer-client-cert-auth --efg"，其它参数为空
{
    controls.Groups[0].Checks[27],
    "--abc --peer-client-cert-auth --efg",
    "",
    "",
},
# 对 controls.Groups[0].Checks[27] 进行设置，传入参数 "--peer-client-cert-auth=true"，其它参数为空
{
    controls.Groups[0].Checks[27],
    "--peer-client-cert-auth=true",
    "",
    "",
},
# 对 controls.Groups[0].Checks[27] 进行设置，传入参数 "--abc --peer-client-cert-auth=true --efg"，其它参数为空
{
    controls.Groups[0].Checks[27],
    "--abc --peer-client-cert-auth=true --efg",
    "",
    "",
}
# 空字符串
# 空字符串
# 空字符串
# 空字符串

# 控制组中的第一个检查的命令
# 设置了一些命令行参数
# 空字符串
# 空字符串

# 控制组中的第一个检查的命令
# 设置了一些命令行参数
# 空字符串
# 设置了一些环境变量

# 控制组中的第一个检查的命令
# 设置了一些命令行参数
# 空字符串
# 空字符串
# 第一个元素的含义是对 controls.Groups[0].Checks[31] 的描述和设置参数
{
    controls.Groups[0].Checks[31],  # 控制组中的第一个检查项
    "2:45 ../kubernetes/kube-apiserver --option1=20,30,40",  # 描述和设置参数
    "",  # 空字符串
    "INSECURE_PORT=0",  # 设置不安全端口为0
},
# 第二个元素的含义是对 controls.Groups[0].Checks[32] 的描述和设置参数
{
    controls.Groups[0].Checks[32],  # 控制组中的第二个检查项
    "2:45 ../kubernetes/kube-apiserver --option1=20,30,40",  # 描述和设置参数
    "",  # 空字符串
    "AUDIT_LOG_MAXAGE=40",  # 设置审计日志最大年龄为40
},
# 第三个元素的含义是对 controls.Groups[0].Checks[33] 的描述和设置参数
{
    controls.Groups[0].Checks[33],  # 控制组中的第三个检查项
    "2:45 ../kubernetes/kube-apiserver --option1=20,30,40",  # 描述和设置参数
    "",  # 空字符串
    "MAX_BACKLOG=20",  # 设置最大后台日志为20
},
	for _, c := range cases {
		// 遍历测试用例
		t.Run(c.Text, func(t *testing.T) {
			// 运行子测试
			c.Check.AuditOutput = c.str
			c.Check.AuditConfigOutput = c.strConfig
			c.Check.AuditEnvOutput = c.strEnv
			// 设置测试用例的输出
			res, err := c.Check.execute()
			// 执行测试用例
			if err != nil {
				// 如果有错误，输出错误信息
				t.Errorf(err.Error())
			}
			if !res.testResult {
				// 如果测试结果不符合预期，输出错误信息
				t.Errorf("expected:%v, got:%v", true, res)
			}
		})
	}
}

func TestTestExecuteExceptions(t *testing.T) {

	cases := []struct {
```
```go
		// 定义测试用例结构
# 定义一个结构体，包含两个字段：Check 和 string
*Check
str string
}{
    # 第一个元素，使用 controls.Groups[0].Checks[23] 作为 Check，"this is not valid json {} at all" 作为 string
    {
        controls.Groups[0].Checks[23],
        "this is not valid json {} at all",
    },
    # 第二个元素，使用 controls.Groups[0].Checks[24] 作为 Check，"{\"key\": \"value\"}" 作为 string
    {
        controls.Groups[0].Checks[24],
        "{\"key\": \"value\"}",
    },
    # 第三个元素，使用 controls.Groups[0].Checks[25] 作为 Check，"broken } yaml\nenabled: true" 作为 string
    {
        controls.Groups[0].Checks[25],
        "broken } yaml\nenabled: true",
    },
    # 第四个元素，使用 controls.Groups[0].Checks[26] 作为 Check，"currentMasterVersion: 1.11" 作为 string
    {
        controls.Groups[0].Checks[26],
        "currentMasterVersion: 1.11",
    },
    # ...
// 对 controls.Groups[0].Checks[26] 进行测试
// 输出信息为 "currentMasterVersion: "
// 遍历测试用例
for _, c := range cases {
	// 运行测试
	t.Run(c.Text, func(t *testing.T) {
		// 设置 c.Check.AuditConfigOutput 为 c.str
		c.Check.AuditConfigOutput = c.str
		// 执行测试
		res, err := c.Check.execute()
		// 如果有错误，输出错误信息
		if err != nil {
			t.Errorf(err.Error())
		}
		// 如果测试结果为真，输出预期结果和实际结果
		if res.testResult {
			t.Errorf("expected:%v, got:%v", false, res)
		}
	})
}
```

```
// 测试 TestUnmarshal 函数
func TestTestUnmarshal(t *testing.T) {
# 定义一个结构体 kubeletConfig，包含 Kind、ApiVersion 和 Address 三个字段
type kubeletConfig struct {
    Kind       string
    ApiVersion string
    Address    string
}
# 定义一个测试用例切片 cases，每个测试用例包含 content 字符串、jsonInterface 接口类型和 expectedToFail 布尔值
cases := []struct {
    content        string
    jsonInterface  interface{}
    expectedToFail bool
}{
    # 第一个测试用例
    {
        # JSON 格式的字符串
        `{
        "kind": "KubeletConfiguration",
        "apiVersion": "kubelet.config.k8s.io/v1beta1",
        "address": "0.0.0.0"
        }
        `,
        # 期望将 JSON 解析为 kubeletConfig 结构体
        kubeletConfig{},
        # 期望测试不会失败
        false,
    }, {
# 定义一个名为 KubeletConfiguration 的对象
kind: KubeletConfiguration
# 设置 Kubelet 的地址为 0.0.0.0
address: 0.0.0.0
# 设置 API 版本为 kubelet.config.k8s.io/v1beta1
apiVersion: kubelet.config.k8s.io/v1beta1
# 配置认证方式
authentication:
  # 禁用匿名认证
  anonymous:
    enabled: false
  # 配置 webhook 认证
  webhook:
    # 设置缓存时间为 2 分钟
    cacheTTL: 2m0s
  # 启用认证
  enabled: true
  # 配置 x509 证书认证
  x509:
    # 设置客户端 CA 文件路径
    clientCAFile: /etc/kubernetes/pki/ca.crt
# 配置 TLS 加密套件
tlsCipherSuites:
  - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
# 创建一个空的 kubeletConfig 对象
kubeletConfig{},
# 设置为 false
false,
# 定义测试用例
cases := []struct {
    content         string          // 测试内容
    jsonInterface   interface{}     // JSON 接口
    expectedToFail  bool            // 预期是否失败
}{
    {
        // 测试内容
        `
        kind: ddress: 0.0.0.0
        apiVersion: kubelet.config.k8s.io/v1beta
        `,
        kubeletConfig{},    // JSON 接口
        true,               // 预期是否失败
    },
}

# 遍历测试用例
for id, c := range cases {
    t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
        # 解析测试内容到 JSON 接口
        err := unmarshal(c.content, &c.jsonInterface)
        # 判断是否有错误
        if err != nil {
            # 如果预期不应该失败，则输出错误信息
            if !c.expectedToFail {
                t.Errorf("should pass, got error:%v", err)
            }
        } else {
            # 如果预期应该失败，则输出错误信息
            if c.expectedToFail {
                t.Errorf("should fail, but passed")
            }
        }
    })
}
// 定义一个结构体 kubeletConfig，包含 Kind、ApiVersion 和 Address 字段
type kubeletConfig struct {
	Kind       string
	ApiVersion string
	Address    string
}

// 定义测试函数 TestExecuteJSONPath
func TestExecuteJSONPath(t *testing.T) {
	// 定义测试用例数组 cases
	cases := []struct {
		name           string         // 测试用例名称
		jsonPath       string         // JSON 路径
		jsonInterface  kubeletConfig  // JSON 接口
		expectedResult string         // 期望结果
		expectedToFail bool           // 是否期望失败
	}{
		{
			"JSONPath parse works, results don't match", // 测试用例名称
		{
			// 测试用例描述
			"JSONPath parse works, results match",
			// JSONPath 表达式
			"{.Kind}",
			// kubeletConfig 结构体
			kubeletConfig{
				// 配置类型
				Kind:       "KubeletConfiguration",
				// API 版本
				ApiVersion: "kubelet.config.k8s.io/v1beta1",
				// 地址
				Address:    "127.0.0.0",
			},
			// 期望的结果
			"KubeletConfiguration",
			// 是否匹配期望结果
			false,
		},
# 定义测试用例数组
{
    "JSONPath parse fails",  # 测试用例名称
    "{.ApiVersion",  # JSONPath 表达式
    kubeletConfig{  # kubeletConfig 结构体
        Kind:       "KubeletConfiguration",  # 类型
        ApiVersion: "kubelet.config.k8s.io/v1beta1",  # API 版本
        Address:    "127.0.0.0",  # 地址
    },
    "",  # 预期结果
    true,  # 是否预期失败
},
}
# 遍历测试用例数组
for _, c := range cases {
    t.Run(c.name, func(t *testing.T) {  # 运行测试用例
        result, err := executeJSONPath(c.jsonPath, c.jsonInterface)  # 执行 JSONPath 解析
        if err != nil && !c.expectedToFail {  # 如果出错且不是预期失败
            t.Fatalf("jsonPath:%q, expectedResult:%q got:%v", c.jsonPath, c.expectedResult, err)  # 报错
        }
        if c.expectedResult != result && !c.expectedToFail {  # 如果预期结果与实际结果不符且不是预期失败
            t.Errorf("jsonPath:%q, expectedResult:%q got:%q", c.jsonPath, c.expectedResult, result)  # 报错
// 定义测试函数，用于测试所有元素是否有效
func TestAllElementsValid(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        source []string
        target []string
        valid  bool
    }{
        // 第一个测试用例，源和目标都为空，应该返回有效
        {
            source: []string{},
            target: []string{},
            valid:  true,
        },
        // 第二个测试用例，源为"blah"，目标为空，应该返回无效
        {
            source: []string{"blah"},
            target: []string{},
            valid:  false,
# 创建一个包含 source、target 和 valid 字段的结构体
{
    # source 字段为空字符串数组
    source: []string{},
    # target 字段包含一组加密算法名称的字符串数组
    target: ["TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
            "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
            "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
            "TLS_RSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256"],
    # valid 字段为 false
    valid: false,
},
{
    # source 字段包含一组加密算法名称的字符串数组
    source: ["TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"],
    # target 字段包含一组加密算法名称的字符串数组
    target: ["TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
            "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
            "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
            "TLS_RSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256"],
    # valid 字段为 true
    valid: true,
},
{
    # source 字段包含一个名为 "blah" 的字符串数组
    source: ["blah"],
    # target 字段包含一组加密算法名称的字符串数组
    target: ["TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
            # (此处省略部分内容)
            "TLS_RSA_WITH_AES_128_GCM_SHA256"],
    # valid 字段为 true
    valid: true,
},
# 定义一个包含测试用例的切片
cases := []struct {
	source []string  # 源字符串切片
	target []string  # 目标字符串切片
	valid  bool      # 有效性标志
}{
	{
		source: []string{"TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "blah"},  # 源字符串切片的值
		target: []string{"TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
			"TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
			"TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
			"TLS_RSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256"},  # 目标字符串切片的值
		valid: false,  # 有效性标志的值
	},
}
# 遍历测试用例切片
for id, c := range cases {
	# 使用测试框架运行测试用例
	t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
		# 如果源字符串切片中的所有元素都存在于目标字符串切片中，且有效性标志为真，则输出错误信息
		if !allElementsValid(c.source, c.target) && c.valid {
			t.Errorf("Not All Elements in %q are found in %q", c.source, c.target)
		}
	})
}
	}
}

func TestSplitAndRemoveLastSeparator(t *testing.T) {
    // 定义测试用例
	cases := []struct {
		source     string   // 输入字符串
		valid      bool     // 是否有效
		elementCnt int      // 元素数量
	}{
		{
			valid:      true,   // 第一个测试用例，有效且包含8个元素
			elementCnt: 8,
		},
		{
			source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",  // 第二个测试用例，包含两个元素
			valid:      true,
			elementCnt: 2,
		},
		{
			source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",  // 第三个测试用例，包含两个元素
# 定义测试用例数组，每个元素包含源字符串、有效性和元素数量
cases := []struct {
    source:     string,  # 源字符串
    valid:      bool,    # 是否有效
    elementCnt: int,     # 元素数量
}{
    {
        source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",
        valid:      true,    # 有效
        elementCnt: 2,       # 元素数量为2
    },
    {
        source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, ",
        valid:      true,    # 有效
        elementCnt: 2,       # 元素数量为2
    },
    {
        source:     " TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",
        valid:      true,    # 有效
        elementCnt: 2,       # 元素数量为2
    },
}

# 遍历测试用例数组
for id, c := range cases {
    # 使用测试框架运行子测试
    t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
        # 调用函数splitAndRemoveLastSeparator，将源字符串按默认分隔符分割并移除最后一个分隔符
        as := splitAndRemoveLastSeparator(c.source, defaultArraySeparator)
        # 如果分割后的数组长度为0且测试用例标记为有效，则输出错误信息
        if len(as) == 0 && c.valid {
            t.Errorf("Split did not work with %q", c.source)
// 结束当前函数
			}

			// 如果元素数量不等于预期数量，输出错误信息
			if c.elementCnt != len(as) {
				t.Errorf("Split did not work with %q expected: %d got: %d", c.source, c.elementCnt, len(as))
			}
		})
	}
}

// 测试比较操作
func TestCompareOp(t *testing.T) {
	// 定义测试用例
	cases := []struct {
		label                 string
		op                    string
		flagVal               string
		compareValue          string
		expectedResultPattern string
		testResult            bool
	}{
		// 测试不匹配的操作
		{label: "empty - op", op: "", flagVal: "", compareValue: "", expectedResultPattern: "", testResult: false},
// 创建一个包含测试数据的对象，包括操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=blah", op: "blah", flagVal: "foo", compareValue: "bar", expectedResultPattern: "", testResult: false},

// 测试操作类型为 "eq" 的情况
{label: "op=eq, both empty", op: "eq", flagVal: "", compareValue: "", expectedResultPattern: "'' is equal to ''", testResult: true},

// 测试操作类型为 "eq"，标志值和比较值都为 "true" 的情况
{label: "op=eq, true==true", op: "eq", flagVal: "true",
    compareValue:          "true",
    expectedResultPattern: "'true' is equal to 'true'",
    testResult:            true},

// 测试操作类型为 "eq"，标志值和比较值都为 "false" 的情况
{label: "op=eq, false==false", op: "eq", flagVal: "false",
    compareValue:          "false",
    expectedResultPattern: "'false' is equal to 'false'",
    testResult:            true},

// 测试操作类型为 "eq"，标志值为 "false"，比较值为 "true" 的情况
{label: "op=eq, false==true", op: "eq", flagVal: "false",
    compareValue:          "true",
    expectedResultPattern: "'false' is equal to 'true'",
    testResult:            false},
# 创建一个包含测试用例的对象，测试字符串是否相等
{label: "op=eq, strings match", op: "eq", flagVal: "KubeletConfiguration",
    compareValue:          "KubeletConfiguration",
    expectedResultPattern: "'KubeletConfiguration' is equal to 'KubeletConfiguration'",
    testResult:            true},

# 创建一个包含测试用例的对象，测试 flagVal 是否为空，预期结果为不相等
{label: "op=eq, flagVal=empty", op: "eq", flagVal: "",
    compareValue:          "KubeletConfiguration",
    expectedResultPattern: "'' is equal to 'KubeletConfiguration'",
    testResult:            false},

# 创建一个包含测试用例的对象，测试 compareValue 是否为空，预期结果为不相等
{label: "op=eq, compareValue=empty", op: "eq", flagVal: "KubeletConfiguration",
    compareValue:          "",
    expectedResultPattern: "'KubeletConfiguration' is equal to ''",
    testResult:            false},

# 创建一个包含测试用例的对象，测试字符串是否不相等
{label: "op=noteq, both empty", op: "noteq", flagVal: "",
    compareValue: "", expectedResultPattern: "'' is not equal to ''",
    testResult: false},
# 创建一个包含不相等比较的测试用例对象，每个对象包含标签、操作、标志值、比较值、预期结果模式和测试结果
{label: "op=noteq, true!=true", op: "noteq", flagVal: "true",
    compareValue:          "true",
    expectedResultPattern: "'true' is not equal to 'true'",
    testResult:            false},

{label: "op=noteq, false!=false", op: "noteq", flagVal: "false",
    compareValue:          "false",
    expectedResultPattern: "'false' is not equal to 'false'",
    testResult:            false},

{label: "op=noteq, false!=true", op: "noteq", flagVal: "false",
    compareValue:          "true",
    expectedResultPattern: "'false' is not equal to 'true'",
    testResult:            true},

{label: "op=noteq, strings match", op: "noteq", flagVal: "KubeletConfiguration",
    compareValue:          "KubeletConfiguration",
    expectedResultPattern: "'KubeletConfiguration' is not equal to 'KubeletConfiguration'",
    testResult:            false},
// 第一个对象的注释
{label: "op=noteq, flagVal=empty", op: "noteq", flagVal: "",
    compareValue:          "KubeletConfiguration",
    expectedResultPattern: "'' is not equal to 'KubeletConfiguration'",
    testResult:            true},

// 第二个对象的注释
{label: "op=noteq, compareValue=empty", op: "noteq", flagVal: "KubeletConfiguration",
    compareValue:          "",
    expectedResultPattern: "'KubeletConfiguration' is not equal to ''",
    testResult:            true},

// Test Op "gt" 的注释
{label: "op=gt, both empty", op: "gt", flagVal: "",
    compareValue: "", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
{label: "op=gt, 0 > 0", op: "gt", flagVal: "0",
    compareValue: "0", expectedResultPattern: "0 is greater than 0",
    testResult: false},
{label: "op=gt, 4 > 5", op: "gt", flagVal: "4",
    compareValue: "5", expectedResultPattern: "4 is greater than 5",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较 5 是否大于 4
{label: "op=gt, 5 > 4", op: "gt", flagVal: "5",
    compareValue: "4", expectedResultPattern: "5 is greater than 4",
    testResult: true},
// 创建一个包含比较操作信息的对象，比较 5 是否大于 5
{label: "op=gt, 5 > 5", op: "gt", flagVal: "5",
    compareValue: "5", expectedResultPattern: "5 is greater than 5",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较 Pikachu 是否大于 5
{label: "op=gt, Pikachu > 5", op: "gt", flagVal: "Pikachu",
    compareValue: "5", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较 5 是否大于 Bulbasaur
{label: "op=gt, 5 > Bulbasaur", op: "gt", flagVal: "5",
    compareValue: "Bulbasaur", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较两个空值
{label: "op=lt, both empty", op: "lt", flagVal: "",
    compareValue: "", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较 0 是否小于 0
{label: "op=lt, 0 < 0", op: "lt", flagVal: "0",
    compareValue: "0", expectedResultPattern: "0 is lower than 0",
    testResult: false},
// 创建一个包含比较操作信息的对象，比较 4 是否小于 5
// 比较操作为小于，flagVal为"5"，compareValue为"4"，预期结果为"4 is lower than 5"，测试结果为true
{label: "op=lt, 5 < 4", op: "lt", flagVal: "5",
    compareValue: "4", expectedResultPattern: "4 is lower than 5",
    testResult: true},
// 比较操作为小于，flagVal为"5"，compareValue为"4"，预期结果为"5 is lower than 4"，测试结果为false
{label: "op=lt, 5 < 4", op: "lt", flagVal: "5",
    compareValue: "4", expectedResultPattern: "5 is lower than 4",
    testResult: false},
// 比较操作为小于，flagVal为"5"，compareValue为"5"，预期结果为"5 is lower than 5"，测试结果为false
{label: "op=lt, 5 < 5", op: "lt", flagVal: "5",
    compareValue: "5", expectedResultPattern: "5 is lower than 5",
    testResult: false},
// 比较操作为小于，flagVal为"Charmander"，compareValue为"5"，预期结果为"Invalid Number(s) used for comparison"，测试结果为false
{label: "op=lt, Charmander < 5", op: "lt", flagVal: "Charmander",
    compareValue: "5", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 比较操作为小于，flagVal为"5"，compareValue为"Charmeleon"，预期结果为"Invalid Number(s) used for comparison"，测试结果为false
{label: "op=lt, 5 < Charmeleon", op: "lt", flagVal: "5",
    compareValue: "Charmeleon", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 比较操作为大于等于，flagVal和compareValue均为空，预期结果为"Invalid Number(s) used for comparison"，测试结果为false
{label: "op=gte, both empty", op: "gte", flagVal: "",
    compareValue: "", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 比较操作为大于等于，flagVal为"0"，compareValue为"0"，预期结果为"0 is greater or equal to 0"，测试结果为true
{label: "op=gte, 0 >= 0", op: "gte", flagVal: "0",
    compareValue: "0", expectedResultPattern: "0 is greater or equal to 0",
    testResult: true},
// 测试结果为真
{label: "op=eq, 4 == 4", op: "eq", flagVal: "4",
    compareValue: "4", expectedResultPattern: "4 is equal to 4",
    testResult: true},
// 测试结果为假
{label: "op=eq, 4 == 5", op: "eq", flagVal: "4",
    compareValue: "5", expectedResultPattern: "4 is equal to 5",
    testResult: false},
// 测试结果为真
{label: "op=gte, 4 >= 5", op: "gte", flagVal: "4",
    compareValue: "5", expectedResultPattern: "4 is greater or equal to 5",
    testResult: false},
// 测试结果为真
{label: "op=gte, 5 >= 4", op: "gte", flagVal: "5",
    compareValue: "4", expectedResultPattern: "5 is greater or equal to 4",
    testResult: true},
// 测试结果为真
{label: "op=gte, 5 >= 5", op: "gte", flagVal: "5",
    compareValue: "5", expectedResultPattern: "5 is greater or equal to 5",
    testResult: true},
// 测试结果为假
{label: "op=gte, Ekans >= 5", op: "gte", flagVal: "Ekans",
    compareValue: "5", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 测试结果为假
{label: "op=gte, 4 >= Zubat", op: "gte", flagVal: "4",
    compareValue: "Zubat", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
// 测试 Op "lte"
{label: "op=lte, both empty", op: "lte", flagVal: "",
    compareValue: "", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},
# 以下是一系列的测试用例，用于测试不同的小于等于（lte）操作
# 测试用例1：0 <= 0，flagVal为0，compareValue为0，预期结果为"0 is lower or equal to 0"，测试结果为true
{label: "op=lte, 0 <= 0", op: "lte", flagVal: "0",
    compareValue: "0", expectedResultPattern: "0 is lower or equal to 0",
    testResult: true},

# 测试用例2：4 <= 5，flagVal为4，compareValue为5，预期结果为"4 is lower or equal to 5"，测试结果为true
{label: "op=lte, 4 <= 5", op: "lte", flagVal: "4",
    compareValue: "5", expectedResultPattern: "4 is lower or equal to 5",
    testResult: true},

# 测试用例3：5 <= 4，flagVal为5，compareValue为4，预期结果为"5 is lower or equal to 4"，测试结果为false
{label: "op=lte, 5 <= 4", op: "lte", flagVal: "5",
    compareValue: "4", expectedResultPattern: "5 is lower or equal to 4",
    testResult: false},

# 测试用例4：5 <= 5，flagVal为5，compareValue为5，预期结果为"5 is lower or equal to 5"，测试结果为true
{label: "op=lte, 5 <= 5", op: "lte", flagVal: "5",
    compareValue: "5", expectedResultPattern: "5 is lower or equal to 5",
    testResult: true},

# 测试用例5：Venomoth <= 4，flagVal为Venomoth，compareValue为4，预期结果为"Invalid Number(s) used for comparison"，测试结果为false
{label: "op=lte, Venomoth <= 4", op: "lte", flagVal: "Venomoth",
    compareValue: "4", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},

# 测试用例6：5 <= Meowth，flagVal为5，compareValue为Meowth，预期结果为"Invalid Number(s) used for comparison"，测试结果为false
{label: "op=lte, 5 <= Meowth", op: "lte", flagVal: "5",
    compareValue: "Meowth", expectedResultPattern: "Invalid Number(s) used for comparison",
    testResult: false},

# 测试 Op "has"
# 创建一个包含不同测试用例的对象数组
[
    {label: "op=has, both empty", op: "has", flagVal: "",
        compareValue: "", expectedResultPattern: "'' has ''",
        testResult: true},
    {label: "op=has, flagVal=empty", op: "has", flagVal: "",
        compareValue: "blah", expectedResultPattern: "'' has 'blah'",
        testResult: false},
    {label: "op=has, compareValue=empty", op: "has", flagVal: "blah",
        compareValue: "", expectedResultPattern: "'blah' has ''",
        testResult: true},
    {label: "op=has, 'blah' has 'la'", op: "has", flagVal: "blah",
        compareValue: "la", expectedResultPattern: "'blah' has 'la'",
        testResult: true},
    {label: "op=has, 'blah' has 'LA'", op: "has", flagVal: "blah",
        compareValue: "LA", expectedResultPattern: "'blah' has 'LA'",
        testResult: false},
    {label: "op=has, 'blah' has 'lo'", op: "has", flagVal: "blah",
        compareValue: "lo", expectedResultPattern: "'blah' has 'lo'",
        testResult: false},
    // Test Op "nothave"
]
# 创建一个包含测试数据的对象数组，每个对象包含了测试用例的标签、操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=nothave, both empty", op: "nothave", flagVal: "",
    compareValue: "", expectedResultPattern: " '' not have ''",
    testResult: false},
{label: "op=nothave, flagVal=empty", op: "nothave", flagVal: "",
    compareValue: "blah", expectedResultPattern: " '' not have 'blah'",
    testResult: true},
{label: "op=nothave, compareValue=empty", op: "nothave", flagVal: "blah",
    compareValue: "", expectedResultPattern: " 'blah' not have ''",
    testResult: false},
{label: "op=nothave, 'blah' not have 'la'", op: "nothave", flagVal: "blah",
    compareValue: "la", expectedResultPattern: " 'blah' not have 'la'",
    testResult: false},
{label: "op=nothave, 'blah' not have 'LA'", op: "nothave", flagVal: "blah",
    compareValue: "LA", expectedResultPattern: " 'blah' not have 'LA'",
    testResult: true},
{label: "op=nothave, 'blah' not have 'lo'", op: "nothave", flagVal: "blah",
    compareValue: "lo", expectedResultPattern: " 'blah' not have 'lo'",
    testResult: true},

// 测试操作类型为 "regex"
// 定义一个对象，包含操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=regex, both empty", op: "regex", flagVal: "",
    compareValue: "", expectedResultPattern: " '' matched by ''",
    testResult: true},
// 定义一个对象，包含操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=regex, flagVal=empty", op: "regex", flagVal: "",
    compareValue: "blah", expectedResultPattern: " '' matched by 'blah'",
    testResult: false},

// 测试操作 "valid_elements"
// 定义一个对象，包含操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=valid_elements, valid_elements both empty", op: "valid_elements", flagVal: "",
    compareValue: "", expectedResultPattern: "'' contains valid elements from ''",
    testResult: true},
// 定义一个对象，包含操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=valid_elements, valid_elements flagVal empty", op: "valid_elements", flagVal: "",
    compareValue: "a,b", expectedResultPattern: "'' contains valid elements from 'a,b'",
    testResult: false},
// 定义一个对象，包含操作类型、标志值、比较值、预期结果模式和测试结果
{label: "op=valid_elements, valid_elements expectedResultPattern empty", op: "valid_elements", flagVal: "a,b",
    compareValue: "", expectedResultPattern: "'a,b' contains valid elements from ''",
    testResult: false},
// 测试操作 "bitmask"
# 创建一个包含不同测试用例的对象数组
[
    {
        label: "op=bitmask, 644 AND 640", 
        op: "bitmask", 
        flagVal: "640",
        compareValue: "644", 
        expectedResultPattern: "bitmask '640' AND '644'",
        testResult: true
    },
    {
        label: "op=bitmask, 644 AND 777", 
        op: "bitmask", 
        flagVal: "777",
        compareValue: "644", 
        expectedResultPattern: "bitmask '777' AND '644'",
        testResult: false
    },
    {
        label: "op=bitmask, 644 AND 444", 
        op: "bitmask", 
        flagVal: "444",
        compareValue: "644", 
        expectedResultPattern: "bitmask '444' AND '644'",
        testResult: true
    },
    {
        label: "op=bitmask, 644 AND 211", 
        op: "bitmask", 
        flagVal: "211",
        compareValue: "644", 
        expectedResultPattern: "bitmask '211' AND '644'",
        testResult: false
    },
    {
        label: "op=bitmask, Harry AND 211", 
        op: "bitmask", 
        flagVal: "Harry",
        compareValue: "644", 
        expectedResultPattern: "Not numeric value - flag: Harry",
        testResult: false
    },
    {
        label: "op=bitmask, 644 AND Potter", 
        op: "bitmask", 
        flagVal: "211",
        compareValue: "Potter", 
        expectedResultPattern: "Not numeric value - flag: Potter",
        testResult: false
    }
]
# 遍历测试用例列表
for _, c := range cases:
    # 使用测试用例的标签创建子测试
    t.Run(c.label, func(t *testing.T) {
        # 调用比较操作函数，获取预期结果模式和测试结果
        expectedResultPattern, testResult := compareOp(c.op, c.flagVal, c.compareValue)
        # 检查预期结果模式是否与实际结果一致，如果不一致则输出错误信息
        if expectedResultPattern != c.expectedResultPattern:
            t.Errorf("'expectedResultPattern' did not match - op: %q expected:%q  got:%q", c.op, c.expectedResultPattern, expectedResultPattern)
        # 检查测试结果是否与预期结果一致，如果不一致则输出错误信息
        if testResult != c.testResult:
            t.Errorf("'testResult' did not match - lop: %q expected:%t  got:%t", c.op, c.testResult, testResult)
    })
# 定义测试函数 TestToNumeric
func TestToNumeric(t *testing.T) {
    # 定义测试用例列表
    cases := []struct {
        firstValue     string
        secondValue    string
        expectedToFail bool
    }{
# 定义测试用例数组，每个元素包含两个值和一个期望的结果
{
    firstValue:     "a",  # 第一个值为字符串"a"
    secondValue:    "b",  # 第二个值为字符串"b"
    expectedToFail: true,  # 期望结果为失败
},
{
    firstValue:     "5",  # 第一个值为字符串"5"
    secondValue:    "b",  # 第二个值为字符串"b"
    expectedToFail: true,  # 期望结果为失败
},
{
    firstValue:     "5",  # 第一个值为字符串"5"
    secondValue:    "6",  # 第二个值为字符串"6"
    expectedToFail: false, # 期望结果为成功
}

# 遍历测试用例数组，使用每个测试用例运行测试
for id, c := range cases {
    # 使用测试用例的值调用 toNumeric 函数，并获取返回的值和错误
    f, s, err := toNumeric(c.firstValue, c.secondValue)
# 如果测试用例标记为预期失败，并且没有出现错误，则输出错误信息
if c.expectedToFail && err == nil:
    t.Errorf("Expected error while converting %s and %s", c.firstValue, c.secondValue)

# 如果测试用例不标记为预期失败，并且返回的结果不符合预期（f不等于5或者s不等于6），则输出错误信息
if !c.expectedToFail && (f != 5 || s != 6):
    t.Errorf("Expected to return %d,%d - got %d,%d", 5, 6, f, s)
```