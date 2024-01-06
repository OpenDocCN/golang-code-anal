# `grype\grype\presenter\template\presenter.go`

```
// 导入模板包
package template

// 导入 fmt 包用于格式化输入输出
import (
	"fmt"
	// 导入 io 包用于输入输出操作
	"io"
	// 导入 os 包用于操作系统功能
	"os"
	// 导入 reflect 包用于反射操作
	"reflect"
	// 导入 sort 包用于排序操作
	"sort"
	// 导入 text/template 包用于文本模板操作
	"text/template"

	// 导入 sprig 包用于模板函数
	"github.com/Masterminds/sprig/v3"
	// 导入 homedir 包用于处理用户主目录
	"github.com/mitchellh/go-homedir"

	// 导入 clio 包用于命令行输入输出
	"github.com/anchore/clio"
	// 导入 grype 包用于漏洞匹配
	"github.com/anchore/grype/grype/match"
	// 导入 grype 包用于软件包操作
	"github.com/anchore/grype/grype/pkg"
	// 导入 grype 包用于展示模型
	"github.com/anchore/grype/grype/presenter/models"
	// 导入 grype 包用于漏洞操作
	"github.com/anchore/grype/grype/vulnerability"
)
// Presenter是一个presenter.Presenter的实现，根据用户提供的Go文本模板格式化输出。
type Presenter struct {
    id                 clio.Identification  // 用于标识Presenter的ID
    matches            match.Matches        // 匹配项
    ignoredMatches     []match.IgnoredMatch // 忽略的匹配项
    packages           []pkg.Package        // 包列表
    context            pkg.Context          // 上下文
    metadataProvider   vulnerability.MetadataProvider  // 元数据提供者
    appConfig          interface{}          // 应用配置
    dbStatus           interface{}          // 数据库状态
    pathToTemplateFile string               // 模板文件路径
}

// NewPresenter返回一个新的template.Presenter。
func NewPresenter(pb models.PresenterConfig, templateFile string) *Presenter {
    return &Presenter{
        id:                 pb.ID,               // 设置ID
        matches:            pb.Matches,          // 设置匹配项
        ignoredMatches:     pb.IgnoredMatches,   // 设置忽略的匹配项
        packages:           pb.Packages,         // 设置包列表
// metadataProvider:   pb.MetadataProvider - 一个元数据提供者对象
// context:            pb.Context - 一个上下文对象
// appConfig:          pb.AppConfig - 一个应用配置对象
// dbStatus:           pb.DBStatus - 一个数据库状态对象
// pathToTemplateFile: templateFile - 模板文件的路径

// Presenter 结构体的构造函数，用于创建一个 Presenter 对象
func NewPresenter(metadataProvider pb.MetadataProvider, context pb.Context, appConfig pb.AppConfig, dbStatus pb.DBStatus, templateFile string) *Presenter {
    return &Presenter{
        metadataProvider:   metadataProvider,
        context:            context,
        appConfig:          appConfig,
        dbStatus:           dbStatus,
        pathToTemplateFile: templateFile,
    }
}

// Present 方法使用用户提供的 Go 模板创建输出
func (pres *Presenter) Present(output io.Writer) error {
    // 将模板文件路径进行扩展，获取完整路径
    expandedPathToTemplateFile, err := homedir.Expand(pres.pathToTemplateFile)
    if err != nil {
        return fmt.Errorf("unable to expand path %q", pres.pathToTemplateFile)
    }

    // 读取模板文件的内容
    templateContents, err := os.ReadFile(expandedPathToTemplateFile)
    if err != nil {
        return fmt.Errorf("unable to get output template: %w", err)
    }
// 将扩展后的模板文件路径赋值给模板名称
templateName := expandedPathToTemplateFile
// 使用模板名称、自定义函数映射和模板内容创建模板对象
tmpl, err := template.New(templateName).Funcs(FuncMap).Parse(string(templateContents))
// 如果创建模板对象出现错误，返回错误信息
if err != nil {
    return fmt.Errorf("unable to parse template: %w", err)
}

// 使用给定的参数创建文档对象
document, err := models.NewDocument(pres.id, pres.packages, pres.context, pres.matches, pres.ignoredMatches, pres.metadataProvider,
    pres.appConfig, pres.dbStatus)
// 如果创建文档对象出现错误，返回错误信息
if err != nil {
    return err
}

// 使用文档对象执行模板，将结果输出到指定的输出流
err = tmpl.Execute(output, document)
// 如果执行模板出现错误，返回错误信息
if err != nil {
    return fmt.Errorf("unable to execute supplied template: %w", err)
}

// 执行完毕，返回空值表示成功
return nil
// FuncMap 是一个返回具有自定义函数的模板.FuncMap的函数，这些函数可供模板作者使用。
var FuncMap = func() template.FuncMap {
    // 使用 sprig.HermeticTxtFuncMap() 创建一个模板.FuncMap，并赋值给 f
    f := sprig.HermeticTxtFuncMap()
    // 添加名为 "getLastIndex" 的自定义函数到 FuncMap 中
    f["getLastIndex"] = func(collection interface{}) int {
        // 检查集合的类型是否为切片
        if v := reflect.ValueOf(collection); v.Kind() == reflect.Slice {
            // 返回切片的最后一个元素的索引
            return v.Len() - 1
        }
        // 如果集合不是切片，则返回 0
        return 0
    }
    // 添加名为 "byMatchName" 的自定义函数到 FuncMap 中
    f["byMatchName"] = func(collection interface{}) interface{} {
        // 将集合转换为 models.Match 类型的切片
        matches, ok := collection.([]models.Match)
        // 如果转换成功，则对 matches 进行排序
        if !ok {
            return collection
        }
        sort.Sort(models.MatchSort(matches))
        return matches
    }
    // 返回自定义函数的 FuncMap
    return f
}
这是一个语法错误，因为它是一个未匹配的右括号。
```