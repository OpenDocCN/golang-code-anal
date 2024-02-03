# `trojan-go\common\geodata\interface.go`

```go
# 定义了一个名为geodata的包
package geodata

# 导入了v2router包，并将其命名为v2router
import v2router "github.com/v2fly/v2ray-core/v4/app/router"

# 定义了一个接口类型GeodataLoader，包含四个方法
type GeodataLoader interface {
    # 从指定文件中加载IP地址和国家信息，返回CIDR和错误
    LoadIP(filename, country string) ([]*v2router.CIDR, error)
    # 从指定文件中加载站点信息，返回Domain和错误
    LoadSite(filename, list string) ([]*v2router.Domain, error)
    # 加载指定国家的地理IP信息，返回CIDR和错误
    LoadGeoIP(country string) ([]*v2router.CIDR, error)
    # 加载指定列表的地理站点信息，返回Domain和错误
    LoadGeoSite(list string) ([]*v2router.Domain, error)
}
```