# `kubesploit\pkg\modules\srdi\srdi.go`

```
/*
Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
此文件是Kubesploit的一部分。
版权所有© 2021 CyberArk Software Ltd。保留所有权利。

Kubesploit是自由软件：您可以根据
由Free Software Foundation发布的GNU通用公共许可证的条款
修改它，无论是许可证的第3版还是
任何以后的版本。

Kubesploit是希望它对增强组织的安全性有所帮助。
Kubesploit不得以任何恶意方式使用。
Kubesploit按原样分发，没有任何保证；包括暗示的保证
适销性或特定用途的适用性。请参阅
GNU通用公共许可证以获取更多详细信息。

您应该收到GNU通用公共许可证的副本
连同Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。
*/

// 外部程序和代码库的附加许可证在文件末尾
// 移至文件末尾，因为这样会生成太多行的注释而导致IDE错误

package srdi

import (
    // 标准库
    "encoding/base64"
    "encoding/binary"
    "fmt"
    "io/ioutil"
    "math"
    "os"
    "strconv"
    "strings"

    // Merlin
    "kubesploit/pkg/modules/shellcode"
)

// Parse是所有扩展模块的初始入口点。所有验证检查和处理将在此处执行
// 函数输入类型仅限于字符串，因此需要额外处理
func Parse(options map[string]string) ([]string, error) {
    // 1. 检查所有参数是否都存在
    // 2. 检查每个参数
    // "commands": [{{dll.Value}}", "{{clearHeader.Value}}", "{{function.Value}}", "{{args.Value}}", "{{pid.Value}}", "{{method.Value}}"]

    if len(options) != 6 {
        return nil, fmt.Errorf("预期提供了6个参数，实际提供了%d个", len(options))
    }
    // 检查提供的路径是否存在 DLL 文件
    _, err := os.Stat(options["dll"])
    if os.IsNotExist(err) {
        return nil, fmt.Errorf("the provided directory does not exist: %s", options["dll"])
    }
    // 将 clearHeader 转换为布尔值
    clearHeader, errClear := strconv.ParseBool(options["clearHeader"])
    if errClear != nil {
        return nil, fmt.Errorf("there was an error parsing %s to boolean:\r\n%s", options["clearHeader"], errClear.Error())
    }

    // 将 PID 转换为整数
    if options["pid"] != "" {
        _, errPid := strconv.Atoi(options["pid"])
        if errPid != nil {
            return nil, fmt.Errorf("there was an error converting the PID to an integer:\r\n%s", errPid.Error())
        }
    }

    if strings.ToLower(options["method"]) != "self" && options["pid"] == "" {
        return nil, fmt.Errorf("a valid PID must be provided for any method except self")
    }

    // 验证 Method 是否是有效类型
    var method string
    switch strings.ToLower(options["method"]) {
    case "self":
        method = "self"
    case "remote":
        method = "remote"
    case "rtlcreateuserthread":
        method = "RtlCreateUserThread"
    case "userapc":
        method = "UserAPC"
    default:
        return nil, fmt.Errorf("invlaid shellcode execution method: %s", method)

    }
    // TODO 在 shellcode.go 中添加类型作为常量或列表

    // 将 DLL 转换为反射式 shellcode
    sc, errShellcode := dllToReflectiveShellcode(options["dll"], options["function"], clearHeader, options["args"])
    if errShellcode != nil {
        return nil, errShellcode
    }
    // 将 shellcode 进行 base64 编码
    b64 := base64.StdEncoding.EncodeToString(sc)
    // 获取 shellcode 作业
    command, errCommand := shellcode.GetJob(method, b64, options["pid"])
    if errCommand != nil {
        return nil, fmt.Errorf("there was an error getting the shellcode job:\r\n%s", errCommand.Error())
    }

    return command, nil
// dllToReflectiveShellcode将现有的Windows DLL转换为包含反射加载器的位置无关shellcode，以在内存中加载和执行shellcode。
// 该函数代码改编自Leo Loobeek的工作，网址为：
// https://gist.githubusercontent.com/leoloobeek/c726719d25d7e7953d4121bd93dd2ed3/raw/05f20bae7aa6cd21e20a52034b9547a19e211c5e/ShellcodeRDI.go
// Leo Loobeek的工作基于Nick Landers（@monoxgas）的sRDI项目，网址为https://github.com/monoxgas/sRDI/
// Nick Landers的工作基于Dan Staples的工作，后者又基于Stephen Fewer的工作
// Nick Landers关于位置无关代码（PIC）的工作基于Matthew Graeber的工作
// Matthew Graeber的工作基于Alan Turing的工作
// 最后，引用Lee Christensen的名字是为了好运和pwnage
// dllPath是源DLL的文件路径，以字符串形式表示，用于转换为反射shellcode
// functionName是在DllMain之后要调用的函数的名称（可选）
// clearHeader如果设置为true，将删除PE头部（可选）
// userDataStr用于定义应与注入的DLL一起调用的任何参数（可选）
func dllToReflectiveShellcode(dllPath string, functionName string, clearHeader bool, userDataStr string) ([]byte, error) {

    // TODO 确保文件存在
    dllBytes, err := ioutil.ReadFile(dllPath) // #nosec G304 允许用户指定任意文件的预期功能
    if err != nil {
        return nil, err
    }

    var functionHash []byte

    if functionName != "" {
        hashFunctionUint32 := hashFunctionName(functionName)
        functionHash = pack(hashFunctionUint32)
    } else {
        functionHash = pack(uint32(0x10))
    }

    flags := 0

    if clearHeader {
        flags |= 0x1
    }

    var userData []byte
    if userDataStr != "" {
        userData = []byte(userDataStr)
    } else {
        // 如果条件不满足，将 userData 设置为字节流 "None"
        userData = []byte("None")
    }

    // 声明一个名为 shellcodez 的空字节流切片
    var shellcodez []byte

    // 返回 shellcodez 和空错误
    return shellcodez, nil
// pack 是一个辅助函数，类似于 Python3 中的 struct.pack
func pack(val uint32) []byte {
    // 创建一个长度为 4 的字节数组
    bytes := make([]byte, 4)
    // 将 val 转换为小端字节序，存入字节数组
    binary.LittleEndian.PutUint32(bytes, val)
    // 返回字节数组
    return bytes
}

// hashFunctionName 创建提供的函数名的 ROR-13 哈希
func hashFunctionName(name string) uint32 {
    // 将函数名转换为字节数组
    function := []byte(name)
    // 在函数名后追加一个字节 0x00
    function = append(function, 0x00)

    // 初始化函数哈希值为 0
    functionHash := uint32(0)

    // 遍历函数名的每个字节
    for _, b := range function {
        // 对函数哈希值进行 ROR-13 运算
        functionHash = ror(functionHash, 13, 32)
        // 将当前字节的值加到函数哈希值上
        functionHash += uint32(b)
    }

    // 返回函数哈希值
    return functionHash
}

// ROR-13 实现
func ror(val uint32, rBits uint32, maxBits uint32) uint32 {
    // 计算 2 的 maxBits 次方减 1
    exp := uint32(math.Exp2(float64(maxBits))) - 1
    // 返回 ROR-13 运算的结果
    return ((val & exp) >> (rBits % maxBits)) | (val << (maxBits - (rBits % maxBits)) & exp)
}

// is64BitDLL 将查看提供的文件的 PE 头部作为字节数组，以确定它是否是一个 64 位应用程序
func is64BitDLL(dllBytes []byte) bool {
    // 定义 IA64 和 AMD64 的机器码
    machineIA64 := uint16(512)
    machineAMD64 := uint16(34404)

    // 从字节数组中提取 PE 头部偏移量
    headerOffset := binary.LittleEndian.Uint32(dllBytes[60:64])
    // 从 PE 头部中提取机器码
    machine := binary.LittleEndian.Uint16(dllBytes[headerOffset+4 : headerOffset+4+2])

    // 如果机器码是 IA64 或 AMD64，则返回 true，表示是 64 位应用程序
    if machine == machineIA64 || machine == machineAMD64 {
        return true
    }
    // 否则返回 false
    return false
}
# 二进制形式的再分发必须在文档和/或其他提供的材料中复制上述版权声明、条件列表和以下免责声明
# 其贡献者的名称不得用于未经特定事先书面许可即可认可或推广从此软件衍生的产品

# 此软件由版权所有者和贡献者"按原样"提供，包括但不限于，对适销性和特定用途的暗示保证。在任何情况下，版权所有者或贡献者均不对任何直接、间接、附带、特殊、示范性或后果性损害（包括但不限于，替代商品或服务的采购；使用、数据或利润的损失；或业务中断）负责，无论是在合同、严格责任或侵权行为（包括疏忽或其他方式）的任何责任理论下，即使已被告知可能发生此类损害。

# 反射式 DLL 注入原语及相关作品根据以下许可证发布
# 版权所有（c）2015年，Dan Staples
# 版权所有（c）2011年，Harmony Security 的 Stephen Fewer（www.harmonysecurity.com）
# 保留所有权利。

# 在源代码和二进制形式中重新分发和使用，无论是否经过修改，都是允许的
# 前提是满足以下条件：

# * 源代码的再分发必须保留上述版权声明、条件列表和以下免责声明。

# * 二进制形式的再分发必须在文档和/或其他提供的材料中复制上述版权声明、条件列表和以下免责声明
# 版权声明和许可证信息
# 本段代码包含了软件的版权声明和许可证信息

# 版权声明
# 未经特定事先书面许可，不得使用 Harmony Security 的名称或其贡献者的名称来认可或推广基于本软件的产品。

# 软件许可证
# 本软件由版权所有者和贡献者“按原样”提供，不提供任何明示或暗示的担保，包括但不限于对适销性和特定用途的适用性的暗示担保。在任何情况下，版权所有者或贡献者均不对任何直接、间接、附带、特殊、惩罚性或后果性损害（包括但不限于替代商品或服务的采购、使用、数据或利润的损失或业务中断）负责，无论是在合同、严格责任或侵权行为（包括疏忽或其他方式）的任何理论下，即使已被告知可能发生此类损害。

# GNU GPLv3 许可证
# 本项目中的所有其他作品均根据 GNU GPLv3 许可证授权

# GNU GPLv3 许可证文本
# 版权声明和许可证信息的一部分，包括了 GNU GPLv3 许可证的完整文本
# 通用公共许可证适用于我们大部分软件；它也适用于作者以这种方式发布的任何其他作品。您也可以将其应用于您的程序。
# 当我们谈论自由软件时，我们指的是自由，而不是价格。我们的通用公共许可证旨在确保您有权利分发自由软件的副本（如果您愿意，也可以收费），您可以获得源代码或者在需要时获取它，您可以更改软件或者在新的自由程序中使用它的部分，并且您知道您可以做这些事情。
# 为了保护您的权利，我们需要防止他人剥夺您这些权利或要求您放弃这些权利。因此，如果您分发软件的副本，或者修改它，您有一定的责任来尊重他人的自由。
# 例如，如果您无偿或有偿地分发这样的程序的副本，您必须将您获得的自由传递给接收者。您必须确保他们也能够获得源代码。并且您必须向他们展示这些条款，以便他们知道自己的权利。
# 使用 GNU GPL 的开发人员通过两个步骤保护您的权利：（1）在软件上声明版权，（2）向您提供这个许可证，给予您合法的复制、分发和/或修改它的权限。
# 为了开发人员和作者的保护，GPL 清楚地解释了这个自由软件没有任何保证。为了用户和作者的利益，GPL 要求修改版本被标记为已更改，以便它们的问题不会错误地归因于先前版本的作者。
# 一些设备被设计为拒绝用户访问安装或运行其中的软件的修改版本，尽管制造商可以这样做。这与保护用户更改软件的自由的目标根本不兼容。这种滥用的系统模式发生在为个人设计的产品领域中。
# 以下是需要注释的代码。
# 这部分代码是 GNU 通用公共许可证（GPL）的一部分，用于定义许可证的术语和条件
# 0. 定义
# “本许可证”指的是 GNU 通用公共许可证的第3版。
# “版权”还包括适用于其他类型作品的类似版权法，比如半导体掩膜。
# “程序”指的是在本许可证下许可的任何可版权作品。每个被许可人都被称为“你”。 “被许可人”和“接收者”可以是个人或组织。
# “修改”作品意味着以需要版权许可的方式从作品中复制或调整全部或部分作品，而不是制作精确的副本。结果作品称为“修改版本”或“基于”先前作品的作品。
# “覆盖作品”指的是未经修改的程序或基于程序的作品。
# “传播”作品意味着对作品进行任何可能使您在适用的版权法下直接或间接承担侵权责任的行为，除了在计算机上执行它或修改私人副本。 传播包括复制，分发（带或不带修改），向公众提供，并在一些国家还包括其他活动。
# 传播工作意味着任何使其他方能够制作或接收副本的传播方式。仅通过计算机网络与用户进行交互，而没有传输副本，不算是传播。

# 交互式用户界面应当显示“适当的法律声明”，以便包括一个方便和显眼的特性，其中（1）显示适当的版权声明，和（2）告知用户工作没有保修（除非提供了保修），许可证持有者可以根据本许可证传播工作，并告知如何查看本许可证的副本。如果界面呈现用户命令或选项列表，比如菜单，列表中的一个显眼项目满足这个标准。

# 1. 源代码。

# “源代码”指的是对工作进行修改的首选形式。“目标代码”指的是工作的非源代码形式。

# “标准接口”指的是一个官方标准，由公认的标准机构定义，或者针对特定编程语言指定的接口，是在使用该语言的开发人员中广泛使用的接口。

# 可执行工作的“系统库”包括除了工作本身之外的任何东西，（a）被包含在主要组件的正常打包形式中，但不是该主要组件的一部分，并且仅用于使工作与该主要组件一起使用，或者实现一个标准接口，对于该接口的实现在源代码形式上对公众是可用的。（b）在这个上下文中，“主要组件”指的是特定操作系统（如果有的话）上的一个主要的基本组件（内核，窗口系统等），或者用于生成工作的编译器，或者用于运行工作的目标代码解释器。

# 以目标代码形式存在的工作的“相应源代码”指的是生成、安装和（对于可执行工作）运行所需的所有源代码。
# 该部分代码缺少上下文，无法添加注释。
# 3. Protecting Users' Legal Rights From Anti-Circumvention Law.
# 保护用户免受反规避法律的侵犯

# 任何受覆盖的作品都不得被视为在任何适用法律下构成有效技术措施的一部分，该法律履行了1996年12月20日通过的《WIPO版权条约》第11条项下的义务，或类似法律禁止或限制规避这种措施。

# 当您传播受覆盖的作品时，您放弃了禁止规避技术措施的任何法律权力，以使这种规避通过行使本许可证下对受覆盖作品的权利而实现，并且您放弃了任何限制操作或修改作品的意图，作为强制执行您或第三方对禁止规避技术措施的法律权利的手段，针对作品的用户。

# 4. Conveying Verbatim Copies.
# 传播逐字复制的副本

# 您可以传播程序源代码的逐字复制，以任何媒介，只要您在每个副本上显著和适当地发布适当的版权声明；保持所有声明，说明本许可证和根据第7节添加的任何不允许的条款适用于代码；保持所有无任何保证的声明；并且向所有接收者提供本许可证的副本以及程序。

# 您可以为您传播的每份副本收取任何价格或不收取价格，并且您可以提供支持或保修保护以换取费用。

# 5. Conveying Modified Source Versions.
# 传播修改后的源代码版本

# 您可以传播基于程序的作品，或者从程序中产生它的修改，以源代码形式根据第4节的条款，只要您还满足所有这些条件：

# a) 作品必须携带显著的声明，说明您对其进行了修改，并提供相关日期。
# 代码段b和c是关于在发布作品时需要包含的许可和注意事项
b) 作品必须明显声明其是根据此许可证发布的，并且遵守第7节添加的任何条件。这个要求修改了第4节中“保持所有通知完整”的要求。
c) 你必须将整个作品作为一个整体根据此许可证授权给任何获得副本的人。因此，此许可证将适用于整个作品，以及所有部分，无论它们如何打包。此许可证不允许以其他方式授权作品，但如果你已经单独收到了这样的许可，它不会使这样的许可无效。

# 代码段d是关于作品是否需要显示适当的法律通知
d) 如果作品具有交互式用户界面，则每个界面必须显示适当的法律通知；但是，如果程序具有不显示适当法律通知的交互式界面，你的作品不需要让它们这样做。

# 代码段6是关于以非源代码形式传播作品的条款
6. 传播非源代码形式。
你可以根据第4和第5节的条款以目标代码形式传播作品，前提是你也以以下方式之一传播可机器阅读的对应源代码：
a) 将目标代码传播在物理产品中（包括物理分发媒介），并附带固定在耐用物理媒介上的对应源代码，通常用于软件交换。
b) 将目标代码传播在物理产品中
# 由于这部分代码不是Python代码，因此无法为其添加注释。
# 从“相应源代码”中提取作为系统库的部分，不需要包含在传达目标代码工作中。
# “用户产品”可以是（1）“消费产品”，即通常用于个人、家庭或家用目的的有形个人财产，或者（2）设计或销售用于纳入住宅的任何东西。在确定产品是否为消费产品时，应优先考虑疑问情况。对于特定用户收到的特定产品，“通常使用”指的是该类产品的典型或常见用途，而不考虑特定用户的身份或特定用户实际使用、期望使用或预期使用产品的方式。无论产品是否具有重要的商业、工业或非消费者用途，只要这些用途代表产品的唯一重要使用方式，产品就是消费产品。
# “用户产品的安装信息”指的是在用户产品中安装和执行修改版本的覆盖工作所需的任何方法、程序、授权密钥或其他信息，这些信息必须足以确保修改后的目标代码的持续功能不会仅因为进行了修改而受到阻止或干扰。
# 如果您根据本节在用户产品中传达目标代码工作，并且传达发生在将用户产品的拥有权和使用权永久转让给接收者或者在固定期限内的交易中（无论交易如何被描述），则在本节下传达的相应源代码必须附有安装信息。但是，如果您或任何第三方都无法保留安装的能力，则不适用此要求。
# 代码段中没有需要注释的程序代码，这部分是许可证的文本，不需要添加注释
# 该部分代码并非Python代码，而是许可证的条款说明，不需要添加注释
# 8. Termination.
# 终止。

# You may not propagate or modify a covered work except as expressly
# provided under this License.  Any attempt otherwise to propagate or
# modify it is void, and will automatically terminate your rights under
# this License (including any patent licenses granted under the third
# paragraph of section 11).
# 除非在本许可证明确规定，否则您不得传播或修改受覆盖的作品。否则任何尝试传播或修改都是无效的，并将自动终止您在本许可证下的权利（包括在第11节第三段授予的任何专利许可）。

# However, if you cease all violation of this License, then your
# license from a particular copyright holder is reinstated (a)
# provisionally, unless and until the copyright holder explicitly and
# finally terminates your license, and (b) permanently, if the copyright
# holder fails to notify you of the violation by some reasonable means
# prior to 60 days after the cessation.
# 但是，如果您停止违反本许可证的所有规定，则您从特定版权持有人那里获得的许可证将被恢复（a）暂时，除非版权持有人明确并最终终止您的许可证，并且（b）永久，如果版权持有人在停止后60天内未通过某种合理方式通知您违规。

# Moreover, your license from a particular copyright holder is
# reinstated permanently if the copyright holder notifies you of the
# violation by some reasonable means, this is the first time you have
# received notice of violation of this License (for any work) from that
# copyright holder, and you cure the violation prior to 30 days after
# your receipt of the notice.
# 此外，如果版权持有人通过某种合理方式通知您违规，并且这是您首次从该版权持有人那里收到有关此许可证的违规通知（对于任何作品），并且您在收到通知后30天内纠正了违规，您从特定版权持有人那里获得的许可将永久恢复。

# Termination of your rights under this section does not terminate the
# licenses of parties who have received copies or rights from you under
# this License.  If your rights have been terminated and not permanently
# reinstated, you do not qualify to receive new licenses for the same
# material under section 10.
# 根据本节的规定终止您的权利不会终止已从您那里根据本许可证收到副本或权利的各方的许可。如果您的权利已被终止且未被永久恢复，则您不符合根据第10节接收相同材料的新许可证的资格。

# 9. Acceptance Not Required for Having Copies.
# 不需要接受许可证即可拥有副本。

# You are not required to accept this License in order to receive or
# run a copy of the Program.  Ancillary propagation of a covered work
# occurring solely as a consequence of using peer-to-peer transmission
# to receive a copy likewise does not require acceptance.  However,
# nothing other than this License grants you permission to propagate or
# modify any covered work.  These actions infringe copyright if you do
# not accept this License.  Therefore, by modifying or propagating a
# 您不需要接受此许可证即可接收或运行程序的副本。仅因使用点对点传输接收副本而发生的受覆盖作品的附带传播也不需要接受。但是，除了本许可证外，没有其他内容授予您传播或修改任何受覆盖作品的权限。如果您不接受此许可证，则这些行为侵犯版权。因此，通过修改或传播
# 10. 自动授权下游接收方
# 每次传递一个受覆盖作品，接收方自动从原始许可人那里获得许可，以运行、修改和传播该作品，受本许可证约束。您不负责强制第三方遵守本许可证。

# “实体交易”是指转让组织的控制权，或者实质上转让全部资产，或者细分一个组织，或者合并组织的交易。如果受实体交易的影响导致受覆盖作品的传播，那么该交易的每一方也会获得前一段中该方的前任权益人根据前一段可以提供的对作品的许可，以及如果前任权益人拥有或可以通过合理努力获得作品的相应源代码，那么该方也会获得对作品相应源代码的所有权。

# 您不得对在本许可证下授予或确认的权利施加任何进一步的限制。例如，您不得对在本许可证下授予的权利收取许可费、版税或其他费用，也不得提起诉讼（包括在诉讼中提出反诉或反诉），声称通过制作、使用、销售、提供出售或进口程序或其任何部分侵犯任何专利权主张。

# 11. 专利
# “贡献者”是指授权在本许可证下使用程序或程序基础上的作品的版权持有人。因此授权的作品称为贡献者的“贡献者版本”。

# 贡献者的“基本专利权主张”是指由贡献者拥有或控制的所有专利权主张，无论是已经获得还是今后获得的，只要通过本许可证允许的某种方式制作、使用或销售其贡献者版本，就会侵犯这些专利权主张。
# 但不包括仅因进一步修改贡献者版本而侵犯的声明。对于此定义，"控制"包括授予专利转让许可的权利，以符合本许可证的要求。

# 每个贡献者向您授予非独占的、全球范围内的、免版税的专利许可，以便制作、使用、销售、提供出售、进口和以其他方式运行、修改和传播其贡献者版本的内容。

# 在接下来的三段中，"专利许可"是指任何明示协议或承诺，无论如何命名，都不会执行专利（例如明示允许实施专利或不起诉专利侵权的明示许可）。向一方"授予"这样的专利许可意味着达成这样的协议或承诺，不会针对该方执行专利。

# 如果您传输了一项受覆盖的作品，并且明知依赖专利许可，而该作品的对应源代码对任何人都不可免费复制，并且符合本许可证的条款，通过公开可用的网络服务器或其他便利的方式，那么您必须要么（1）使对应源代码可用，要么（2）安排放弃对该特定作品的专利许可的利益，要么（3）以符合本许可证要求的方式，安排将专利许可延伸到下游接收方。"明知依赖"意味着您确切知道，如果没有专利许可，您在一个国家传输受覆盖的作品，或者您的接收方在一个国家使用受覆盖的作品，将侵犯该国家的一个或多个可辨认专利，而您有理由相信这些专利是有效的。

# 如果根据单一交易或安排，您传输或通过获得传输受覆盖作品的方式传播，向某些方授予专利许可
# 以下是需要注释的代码。
# 由于这段代码不是Python代码，而是许可证文本，因此无法为每一行代码添加注释。
# 请提供实际的Python代码，以便我可以为您添加注释。
# 13. 使用 GNU Affero 通用公共许可证
# 尽管本许可证的任何其他规定，您有权限将任何受覆盖的作品与根据 GNU Affero 通用公共许可证第 3 版许可的作品链接或组合成单个组合作品，并传达结果作品。本许可证的条款将继续适用于作为受覆盖作品的部分，但是 GNU Affero 通用公共许可证第 13 条关于通过网络进行交互的特殊要求将适用于该组合作品。

# 14. 本许可证的修订版本
# 自由软件基金会可能不时发布 GNU 通用公共许可证的修订版和/或新版本。这些新版本在精神上类似于当前版本，但可能在细节上有所不同，以解决新的问题或关注点。

# 每个版本都被赋予一个区分版本号。如果程序规定 GNU 通用公共许可证的某个编号版本“或任何以后的版本”适用于它，您可以选择遵循该编号版本的条款和条件，或者遵循自由软件基金会发布的任何以后的版本。如果程序没有指定 GNU 通用公共许可证的版本号，您可以选择自由软件基金会发布的任何版本。

# 如果程序规定代理可以决定使用哪个未来版本的 GNU 通用公共许可证，该代理对某个版本的公开接受声明永久授权您选择该版本用于该程序。

# 以后的许可证版本可能会给您额外或不同的权限。但是，由于您选择遵循以后的版本，不会对任何作者或版权持有人施加额外的义务。

# 15. 免责声明
# 对于程序，只要允许的范围内，没有任何保证。
# 应用法律。除非另有书面说明，版权持有人和/或其他方以"原样"提供程序，不提供任何形式的保证，包括但不限于对适销性和特定用途的默示保证。程序的质量和性能的全部风险由您承担。如果程序被证明有缺陷，您将承担所有必要的维修、修理或更正成本。

16. 责任限制。

除非适用法律要求或书面同意，任何版权持有人或任何其他根据上述许可修改和/或传达程序的其他方，均不对您承担任何损害责任，包括因使用或无法使用程序而产生的任何一般、特殊、附带或间接损害（包括但不限于数据丢失或数据被渲染不准确或您或第三方遭受的损失或程序与任何其他程序无法运行的故障），即使该持有人或其他方已被告知可能发生此类损害。

17. 第15和16条的解释。

如果上述的免责声明和责任限制根据其条款无法产生当地法律效力，审查法院应适用最接近于在与程序相关的所有民事责任上绝对放弃的当地法律，除非在返回费用的情况下，保证或承担责任附带程序的副本。

条款和条件的结束

如何将这些条款应用于您的新程序

如果您开发了一个新程序，并且希望它对公众有最大的可能用途，最好的方法是使其成为每个人都可以在这些条款下重新分发和更改的自由软件。

为此，请将以下通知附加到程序。最安全的方法是将它们附加到每个源文件的开头，以最有效地
# 声明排除保修；每个文件至少应包含“版权”行和指向完整通知的指针。
# 一行用于说明程序的名称和简要介绍其功能。
# 版权所有（C）{年份} {作者姓名}
# 本程序是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是（根据您的选择）任何以后的版本。
# 本程序是基于希望它有用而分发的，但没有任何保证；甚至没有适销性或特定用途的暗示保证。详细信息请参见GNU通用公共许可证。
# 您应该已经收到了GNU通用公共许可证的副本。如果没有，请参阅<http://www.gnu.org/licenses/>。
# 还应提供如何通过电子邮件和纸质邮件联系您的信息。
# 如果程序进行终端交互，请在交互模式启动时输出类似于此的简短通知：
# {项目} 版权所有（C）{年份} {全名}
# 本程序绝对不提供任何保证；详细信息请键入“show w”。
# 这是自由软件，您可以在一定条件下重新分发它；详细信息请键入“show c”。
# 假设命令“show w”和“show c”应显示通用公共许可证的相应部分。当然，您的程序命令可能不同；对于GUI界面，您将使用“关于”框。
# 如果您是程序员，还应该让您的雇主（如果您是程序员）或学校（如果有的话）签署程序的“版权免责声明”，如果有必要的话。有关此事的更多信息，以及如何申请和遵循GNU GPL，请参见<http://www.gnu.org/licenses/>。
# GNU通用公共许可证不允许将您的程序合并到专有程序中。如果您的程序是一个子程序库，您
/*
如果你认为允许将专有应用程序与该库进行链接更有用，那么可以使用GNU Lesser General Public License而不是这个许可证。但首先，请阅读<http://www.gnu.org/philosophy/why-not-lgpl.html>。
*/
```