# `grype\grype\internal\packagemetadata\discover_type_names.go`

```
package packagemetadata

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"go/ast"  // 导入 go/ast 包，用于表示 Go 语言的抽象语法树
	"go/parser"  // 导入 go/parser 包，用于解析 Go 源码文件
	"go/token"  // 导入 go/token 包，用于定义 Go 语言的词法标记
	"os/exec"  // 导入 os/exec 包，用于执行外部命令
	"path/filepath"  // 导入 path/filepath 包，用于操作文件路径
	"sort"  // 导入 sort 包，用于对数据进行排序
	"strings"  // 导入 strings 包，用于处理字符串
	"unicode"  // 导入 unicode 包，用于处理 Unicode 字符
	"github.com/scylladb/go-set/strset"  // 导入第三方包，用于操作字符串集合
)

var metadataExceptions = strset.New(
	"FileMetadata",  // 定义一个字符串集合，用于存储元数据的例外情况
)
# DiscoverTypeNames 函数用于发现类型名称，返回类型名称的切片和可能的错误
func DiscoverTypeNames() ([]string, error) {
    # 调用 RepoRoot 函数获取仓库根目录
    root, err := RepoRoot()
    if err != nil {
        return nil, err
    }
    # 使用 filepath.Glob 函数获取匹配指定模式的文件列表
    files, err := filepath.Glob(filepath.Join(root, "grype/pkg/*.go"))
    if err != nil {
        return nil, err
    }
    # 调用 findMetadataDefinitionNames 函数查找元数据定义的名称
    return findMetadataDefinitionNames(files...)
}

# RepoRoot 函数用于获取仓库根目录
func RepoRoot() (string, error) {
    # 使用 exec.Command 函数执行 git 命令获取仓库根目录
    root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()
    if err != nil {
        return "", fmt.Errorf("unable to find repo root dir: %+v", err)
    }
    # 使用 filepath.Abs 函数获取绝对路径
    absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root)))
    if err != nil {
        return "", fmt.Errorf("unable to get abs path to repo root: %w", err)
    }
	}
	return absRepoRoot, nil
}

// 查找元数据定义的名称
func findMetadataDefinitionNames(paths ...string) ([]string, error) {
	// 创建一个空的字符串集合
	names := strset.New()
	// 创建一个空的字符串集合
	usedNames := strset.New()
	// 遍历传入的路径
	for _, path := range paths {
		// 在文件中查找元数据定义的名称
		metadataDefinitions, usedTypeNames, err := findMetadataDefinitionNamesInFile(path)
		// 如果发生错误，返回错误
		if err != nil {
			return nil, err
		}

		// 用于调试的有用信息...
		// fmt.Println(path)
		// fmt.Println("Defs:", metadataDefinitions)
		// fmt.Println("Used Types:", usedTypeNames)
		// fmt.Println()

		// 将元数据定义的名称添加到名称集合中
		names.Add(metadataDefinitions...)
// 将usedTypeNames中的元素添加到usedNames中
usedNames.Add(usedTypeNames...)

// 任何在另一个结构体中使用的定义都不应被视为顶层元数据定义
names.Remove(usedNames.List()...)

// 获取names中的所有元素，并按字母顺序排序
strNames := names.List()
sort.Strings(strNames)

// 注意：3是一个时点的内部检查。如果添加了新的元数据定义，则可以更新此数字，但不是必需的。
// 它真的是用来捕捉生成过程中可能出现的任何重大问题，比如生成了0个定义。
if len(strNames) < 3 {
    return nil, fmt.Errorf("not enough metadata definitions found (discovered: " + fmt.Sprintf("%d", len(strNames)) + ")")
}

// 返回排序后的元数据定义名称列表和nil
return strNames, nil
}

// 在文件中查找元数据定义的名称
func findMetadataDefinitionNamesInFile(path string) ([]string, []string, error) {
    // 设置解析器
	fs := token.NewFileSet() 
	// 创建一个新的文件集合，用于存储文件和位置信息

	f, err := parser.ParseFile(fs, path, nil, parser.ParseComments)
	// 解析指定路径的文件，并返回其抽象语法树表示，同时保留注释信息

	if err != nil {
		return nil, nil, err
		// 如果解析出错，返回错误信息
	}

	var metadataDefinitions []string
	var usedTypeNames []string
	// 定义两个字符串数组，用于存储元数据定义和使用的类型名称

	for _, decl := range f.Decls {
		// 遍历文件中的声明

		// 检查声明是否为类型声明
		spec, ok := decl.(*ast.GenDecl)
		if !ok || spec.Tok != token.TYPE {
			continue
			// 如果不是类型声明，继续下一个声明
		}

		// 遍历类型声明中的所有类型
		for _, typ := range spec.Specs {
			// 检查类型是否为结构体类型
			spec, ok := typ.(*ast.TypeSpec)
			if !ok || spec.Type == nil {
				// 如果不是结构体类型，继续下一个类型
// 继续循环，跳过当前迭代
continue
}

// 尝试将 spec.Type 转换为 *ast.StructType 类型，如果成功则继续，否则跳过当前迭代
structType, ok := spec.Type.(*ast.StructType)
if !ok {
    continue
}

// 检查结构类型是否以 "Metadata" 结尾
name := spec.Name.String()

// 只查找以 "Metadata" 结尾的导出类型
if isMetadataTypeCandidate(name) {
    // 打印结构类型的完整声明
    metadataDefinitions = append(metadataDefinitions, name)
    usedTypeNames = append(usedTypeNames, typeNamesUsedInStruct(structType)...)
}
}
}
// 返回找到的 metadata 类型和使用的类型名称
return metadataDefinitions, usedTypeNames, nil
func typeNamesUsedInStruct(structType *ast.StructType) []string {
	// 递归地查找结构类型中使用的所有类型名称
	var names []string
	for i := range structType.Fields.List {
		// 捕获所有类型的名称（而不是字段名称）
		ast.Inspect(structType.Fields.List[i].Type, func(n ast.Node) bool {
			ident, ok := n.(*ast.Ident)
			if !ok {
				return true
			}

			// 将类型名称添加到列表中
			names = append(names, ident.Name)

			// 继续检查
			return true
		})
	}
}
# 返回变量 names
	return names
}

# 判断给定的字符串是否是元数据类型的候选项
func isMetadataTypeCandidate(name string) bool {
	# 判断字符串长度大于0，并且以"Metadata"结尾，并且首字母大写（即为导出类型），并且不在元数据异常列表中
	return len(name) > 0 &&
		strings.HasSuffix(name, "Metadata") &&
		unicode.IsUpper(rune(name[0])) && # 必须是导出类型
		!metadataExceptions.Has(name)
}
```