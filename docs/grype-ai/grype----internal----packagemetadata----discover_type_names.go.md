# `grype\grype\internal\packagemetadata\discover_type_names.go`

```go
package packagemetadata

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "go/ast"  // 导入 go/ast 包，用于表示 Go 语言的抽象语法树
    "go/parser"  // 导入 go/parser 包，用于解析 Go 源码文件
    "go/token"  // 导入 go/token 包，定义了 Go 语言的词法标记
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "path/filepath"  // 导入 path/filepath 包，用于操作文件路径
    "sort"  // 导入 sort 包，用于对切片进行排序
    "strings"  // 导入 strings 包，用于处理字符串
    "unicode"  // 导入 unicode 包，提供对 Unicode 字符的定义
    
    "github.com/scylladb/go-set/strset"  // 导入第三方包，用于操作字符串集合
)

var metadataExceptions = strset.New(  // 定义字符串集合，用于存储元数据的例外情况
    "FileMetadata",  // 添加一个例外情况
)

func DiscoverTypeNames() ([]string, error) {
    root, err := RepoRoot()  // 调用 RepoRoot 函数，获取仓库根目录
    if err != nil {
        return nil, err
    }
    files, err := filepath.Glob(filepath.Join(root, "grype/pkg/*.go"))  // 获取指定目录下的所有 Go 源文件
    if err != nil {
        return nil, err
    }
    return findMetadataDefinitionNames(files...)  // 调用 findMetadataDefinitionNames 函数，查找元数据定义的名称
}

func RepoRoot() (string, error) {
    root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()  // 执行 git 命令，获取仓库根目录
    if err != nil {
        return "", fmt.Errorf("unable to find repo root dir: %+v", err)  // 如果执行命令出错，返回错误信息
    }
    absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root))  // 获取仓库根目录的绝对路径
    if err != nil {
        return "", fmt.Errorf("unable to get abs path to repo root: %w", err)  // 如果获取绝对路径出错，返回错误信息
    }
    return absRepoRoot, nil  // 返回仓库根目录的绝对路径
}

func findMetadataDefinitionNames(paths ...string) ([]string, error) {
    names := strset.New()  // 创建一个新的字符串集合，用于存储元数据定义的名称
    usedNames := strset.New()  // 创建一个新的字符串集合，用于存储已使用的名称
    for _, path := range paths {  // 遍历传入的文件路径
        metadataDefinitions, usedTypeNames, err := findMetadataDefinitionNamesInFile(path)  // 调用 findMetadataDefinitionNamesInFile 函数，查找文件中的元数据定义名称
        if err != nil {
            return nil, err  // 如果查找出错，返回错误信息
        }

        // useful for debugging...
        // fmt.Println(path)
        // fmt.Println("Defs:", metadataDefinitions)
        // fmt.Println("Used Types:", usedTypeNames)
        // fmt.Println()

        names.Add(metadataDefinitions...)  // 将找到的元数据定义名称添加到 names 集合中
        usedNames.Add(usedTypeNames...)  // 将已使用的名称添加到 usedNames 集合中
    }

    // any definition that is used within another struct should not be considered a top-level metadata definition
    names.Remove(usedNames.List()...)  // 从 names 集合中移除已使用的名称

    strNames := names.List()  // 将 names 集合转换为切片
    sort.Strings(strNames)  // 对切片进行排序

    // note: 3 is a point-in-time gut check. This number could be updated if new metadata definitions are added, but is not required.
    // 注意：3 是一个即时的直觉检查。如果添加了新的元数据定义，则可以更新此数字，但不是必需的。
}
    // 如果找到的元数据定义少于3个，可能会出现生成过程中的重大问题，比如生成了0个定义。
    if len(strNames) < 3 {
        // 返回错误，指出找到的元数据定义不足（发现：X个）
        return nil, fmt.Errorf("not enough metadata definitions found (discovered: " + fmt.Sprintf("%d", len(strNames)) + ")")
    }

    // 返回找到的元数据定义
    return strNames, nil
// 在给定路径中查找元数据定义的名称，并返回元数据定义的名称列表和使用的类型名称列表
func findMetadataDefinitionNamesInFile(path string) ([]string, []string, error) {
    // 设置解析器
    fs := token.NewFileSet()
    f, err := parser.ParseFile(fs, path, nil, parser.ParseComments)
    if err != nil {
        return nil, nil, err
    }

    var metadataDefinitions []string
    var usedTypeNames []string
    for _, decl := range f.Decls {
        // 检查声明是否为类型声明
        spec, ok := decl.(*ast.GenDecl)
        if !ok || spec.Tok != token.TYPE {
            continue
        }

        // 遍历类型声明中声明的所有类型
        for _, typ := range spec.Specs {
            // 检查类型是否为结构类型
            spec, ok := typ.(*ast.TypeSpec)
            if !ok || spec.Type == nil {
                continue
            }

            structType, ok := spec.Type.(*ast.StructType)
            if !ok {
                continue
            }

            // 检查结构类型是否以"Metadata"结尾
            name := spec.Name.String()

            // 只查找以"Metadata"结尾的导出类型
            if isMetadataTypeCandidate(name) {
                // 打印结构类型的完整声明
                metadataDefinitions = append(metadataDefinitions, name)
                usedTypeNames = append(usedTypeNames, typeNamesUsedInStruct(structType)...)
            }
        }
    }
    return metadataDefinitions, usedTypeNames, nil
}

func typeNamesUsedInStruct(structType *ast.StructType) []string {
    // 递归查找结构类型中使用的所有类型名称
    var names []string
    // 遍历结构体类型的字段列表
    for i := range structType.Fields.List {
        // 捕获所有类型的名称（而不是字段名称）
        ast.Inspect(structType.Fields.List[i].Type, func(n ast.Node) bool {
            // 尝试将节点转换为标识符类型
            ident, ok := n.(*ast.Ident)
            // 如果无法转换，则继续检查下一个节点
            if !ok {
                return true
            }

            // 将类型名称添加到列表中
            names = append(names, ident.Name)

            // 继续检查
            return true
        })
    }

    // 返回类型名称列表
    return names
# 检查给定的名称是否是元数据类型的候选名称
func isMetadataTypeCandidate(name string) bool:
    # 返回名称长度大于0，并且以"Metadata"结尾，并且首字母大写（即导出的）
    return len(name) > 0 &&
        strings.HasSuffix(name, "Metadata") &&
        unicode.IsUpper(rune(name[0])) && // 必须是导出的
        !metadataExceptions.Has(name)  # 名称不在元数据异常列表中
```