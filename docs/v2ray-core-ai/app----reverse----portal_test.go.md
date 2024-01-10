# `v2ray-core\app\reverse\portal_test.go`

```
// 声明 reverse_test 包
package reverse_test

// 导入 testing 包
import (
    "testing"

    // 导入 reverse 包
    "v2ray.com/core/app/reverse"
    // 导入 common 包
    "v2ray.com/core/common"
)

// 定义 TestStaticPickerEmpty 函数，参数为 t *testing.T
func TestStaticPickerEmpty(t *testing.T) {
    // 调用 reverse.NewStaticMuxPicker() 方法，返回 picker 和 err
    picker, err := reverse.NewStaticMuxPicker()
    // 如果 err 不为 nil，则触发 panic
    common.Must(err)
    // 调用 picker.PickAvailable() 方法，返回 worker 和 err
    worker, err := picker.PickAvailable()
    // 如果 err 不为 nil，则输出错误信息
    if err == nil {
        t.Error("expected error, but nil")
    }
    // 如果 worker 不为 nil，则输出错误信息
    if worker != nil {
        t.Error("expected nil worker, but not nil")
    }
}
```