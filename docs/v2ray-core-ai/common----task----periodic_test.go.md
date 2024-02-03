# `v2ray-core\common\task\periodic_test.go`

```go
// 定义一个名为 task_test 的包
package task_test

// 导入所需的包
import (
    "testing"
    "time"

    "v2ray.com/core/common"
    . "v2ray.com/core/common/task"
)

// 定义测试函数 TestPeriodicTaskStop
func TestPeriodicTaskStop(t *testing.T) {
    // 初始化变量 value 为 0
    value := 0
    // 创建一个周期性任务对象 task
    task := &Periodic{
        // 设置任务执行间隔为 2 秒
        Interval: time.Second * 2,
        // 设置任务执行的函数
        Execute: func() error {
            // 每次执行使 value 自增
            value++
            return nil
        },
    }
    // 强制启动任务
    common.Must(task.Start())
    // 休眠 5 秒
    time.Sleep(time.Second * 5)
    // 强制关闭任务
    common.Must(task.Close())
    // 检查 value 是否为 3，不是则输出错误信息
    if value != 3 {
        t.Fatal("expected 3, but got ", value)
    }
    // 休眠 4 秒
    time.Sleep(time.Second * 4)
    // 再次检查 value 是否为 3，不是则输出错误信息
    if value != 3 {
        t.Fatal("expected 3, but got ", value)
    }
    // 再次强制启动任务
    common.Must(task.Start())
    // 休眠 3 秒
    time.Sleep(time.Second * 3)
    // 检查 value 是否为 5，不是则输出错误信息
    if value != 5 {
        t.Fatal("Expected 5, but ", value)
    }
    // 再次强制关闭任务
    common.Must(task.Close())
}
```