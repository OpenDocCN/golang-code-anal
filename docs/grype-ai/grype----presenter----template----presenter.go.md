# `grype\grype\presenter\template\presenter.go`

```go
package template

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于输入输出操作
    "os"   // 导入 os 包，提供对操作系统功能的访问
    "reflect"  // 导入 reflect 包，用于反射操作
    "sort"  // 导入 sort 包，用于排序操作
    "text/template"  // 导入 text/template 包，用于文本模板操作

    "github.com/Masterminds/sprig/v3"  // 导入第三方包 Masterminds/sprig/v3
    "github.com/mitchellh/go-homedir"  // 导入第三方包 mitchellh/go-homedir

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    "github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
)

// Presenter is an implementation of presenter.Presenter that formats output according to a user-provided Go text template.
type Presenter struct {
    id                 clio.Identification  // 定义 clio.Identification 类型的 id 字段
    matches            match.Matches  // 定义 match.Matches 类型的 matches 字段
    ignoredMatches     []match.IgnoredMatch  // 定义 match.IgnoredMatch 类型的切片 ignoredMatches 字段
    packages           []pkg.Package  // 定义 pkg.Package 类型的切片 packages 字段
    context            pkg.Context  // 定义 pkg.Context 类型的 context 字段
    metadataProvider   vulnerability.MetadataProvider  // 定义 vulnerability.MetadataProvider 类型的 metadataProvider 字段
    appConfig          interface{}  // 定义空接口类型的 appConfig 字段
    dbStatus           interface{}  // 定义空接口类型的 dbStatus 字段
    pathToTemplateFile string  // 定义字符串类型的 pathToTemplateFile 字段
}

// NewPresenter returns a new template.Presenter.
func NewPresenter(pb models.PresenterConfig, templateFile string) *Presenter {
    return &Presenter{
        id:                 pb.ID,  // 初始化 id 字段
        matches:            pb.Matches,  // 初始化 matches 字段
        ignoredMatches:     pb.IgnoredMatches,  // 初始化 ignoredMatches 字段
        packages:           pb.Packages,  // 初始化 packages 字段
        metadataProvider:   pb.MetadataProvider,  // 初始化 metadataProvider 字段
        context:            pb.Context,  // 初始化 context 字段
        appConfig:          pb.AppConfig,  // 初始化 appConfig 字段
        dbStatus:           pb.DBStatus,  // 初始化 dbStatus 字段
        pathToTemplateFile: templateFile,  // 初始化 pathToTemplateFile 字段
    }
}

// Present creates output using a user-supplied Go template.
func (pres *Presenter) Present(output io.Writer) error {
    expandedPathToTemplateFile, err := homedir.Expand(pres.pathToTemplateFile)  // 使用 homedir.Expand 方法扩展模板文件路径
    if err != nil {
        return fmt.Errorf("unable to expand path %q", pres.pathToTemplateFile)  // 返回错误信息
    }

    templateContents, err := os.ReadFile(expandedPathToTemplateFile)  // 读取模板文件内容
    if err != nil {
        return fmt.Errorf("unable to get output template: %w", err)  // 返回错误信息
    }

    templateName := expandedPathToTemplateFile  // 将扩展后的模板文件路径赋值给 templateName
    // 使用给定的模板名称创建模板对象，并解析模板内容，同时注册自定义函数
    tmpl, err := template.New(templateName).Funcs(FuncMap).Parse(string(templateContents))
    // 如果解析过程中出现错误，则返回解析模板失败的错误
    if err != nil {
        return fmt.Errorf("unable to parse template: %w", err)
    }

    // 使用给定的参数创建一个新的文档对象
    document, err := models.NewDocument(pres.id, pres.packages, pres.context, pres.matches, pres.ignoredMatches, pres.metadataProvider,
        pres.appConfig, pres.dbStatus)
    // 如果创建文档对象过程中出现错误，则返回错误
    if err != nil {
        return err
    }

    // 使用创建好的模板对象执行文档对象，并将结果输出到指定的输出流
    err = tmpl.Execute(output, document)
    // 如果执行过程中出现错误，则返回执行模板失败的错误
    if err != nil {
        return fmt.Errorf("unable to execute supplied template: %w", err)
    }

    // 执行完毕，返回空值表示执行成功
    return nil
// FuncMap 是一个返回自定义函数供模板作者使用的 template.FuncMap 的函数
var FuncMap = func() template.FuncMap {
    // 创建一个包含 sprig.HermeticTxtFuncMap() 函数的 FuncMap
    f := sprig.HermeticTxtFuncMap()
    // 添加名为 "getLastIndex" 的自定义函数到 FuncMap
    f["getLastIndex"] = func(collection interface{}) int {
        // 检查集合类型是否为切片
        if v := reflect.ValueOf(collection); v.Kind() == reflect.Slice {
            // 返回切片的最后一个元素的索引
            return v.Len() - 1
        }
        // 如果集合类型不是切片，则返回 0
        return 0
    }
    // 添加名为 "byMatchName" 的自定义函数到 FuncMap
    f["byMatchName"] = func(collection interface{}) interface{} {
        // 尝试将集合转换为 models.Match 类型的切片
        matches, ok := collection.([]models.Match)
        // 如果转换成功，则对 matches 进行排序并返回
        if !ok {
            return collection
        }
        sort.Sort(models.MatchSort(matches))
        return matches
    }
    // 返回自定义函数的 FuncMap
    return f
}()
```