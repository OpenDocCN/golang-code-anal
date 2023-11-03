# trojan-go源码解析 16

# `version/version.go`

这段代码定义了一个名为`versionOption`的结构体，它包含一个布尔类型的变量`flag`和一个名称为`version`的常量变量。

同时，它导入了两个相关的常量：`option.Option`和` common.VERSION`。

该代码的作用是设置一个Trojan选项实例的`version`选项。`versionOption`是一个偶妙的名称，由于它包含一个布尔类型变量，我们可以将其视为一个开关，可以设置为`true`或`false`。

这个版本的函数可能被用于某个Trojan脚本，通过设置`versionOption.flag`来设置选项`version`的值。


```go
package version

import (
	"flag"
	"fmt"
	"runtime"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/constant"
	"github.com/p4gefau1t/trojan-go/option"
)

type versionOption struct {
	flag *bool
}

```

该代码定义了一个名为"versionOption"的函数类型，以及该函数类型的两个名为"Priority"和"Handle"的函数成员。

具体来说，该函数类型定义了一个"*versionOption"类型的变量c，该变量c包含一个名为"flag"的成员变量，如果该成员变量为true，则执行以下操作：

1. 输出"Trojan-Go"、"Go Version"、"OS/Arch"、"Git Commit"、"Developed by"和"Licensed under"文本。
2. 输出"Go/排查"和"PageFault"的链接。
3. 输出"页面出错"和"未设置"消息。

这里，函数Handle()的实现函数体中包含一个if语句和一个if语句的逆置，if语句检查flag变量是否为true，如果为true，则执行if语句中的内容。如果flag变量为false，则不执行if语句中的内容，转而返回common.NewError("not set")错误。


```go
func (*versionOption) Name() string {
	return "version"
}

func (*versionOption) Priority() int {
	return 10
}

func (c *versionOption) Handle() error {
	if *c.flag {
		fmt.Println("Trojan-Go", constant.Version)
		fmt.Println("Go Version:", runtime.Version())
		fmt.Println("OS/Arch:", runtime.GOOS+"/"+runtime.GOARCH)
		fmt.Println("Git Commit:", constant.Commit)
		fmt.Println("")
		fmt.Println("Developed by PageFault (p4gefau1t)")
		fmt.Println("Licensed under GNU General Public License version 3")
		fmt.Println("GitHub Repository:\thttps://github.com/p4gefau1t/trojan-go")
		fmt.Println("Trojan-Go Documents:\thttps://p4gefau1t.github.io/trojan-go/")
		return nil
	}
	return common.NewError("not set")
}

```

这段代码定义了一个名为 "init" 的函数，它创建了一个名为 "option" 的选项对象。

函数体内使用了一个名为 "RegisterHandler" 的函数，它接收一个名为 "handler" 的参数。这个函数的作用是将 "handler" 对象注册为 "option" 选项中的一个处理程序。

"handler" 对象是一个包含了两个属性的对象："flag" 是一个名为 "version" 的布尔类型，值为 false，而 "help" 是一个字符串类型的参数。这个函数的作用是设置 "handler" 对象在 "option" 选项中对应 "version" 标志的值，以便在 "option.Option()" 方法中正确地返回 "version" 选项的值。

此外，函数还使用了 "Register" 函数，这个函数的作用是注册 "handler" 对象为 "option" 选项中的处理程序。


```go
func init() {
	option.RegisterHandler(&versionOption{
		flag: flag.Bool("version", false, "Display version and help info"),
	})
}

```