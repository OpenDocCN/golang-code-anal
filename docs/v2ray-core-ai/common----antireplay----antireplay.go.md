# `v2ray-core\common\antireplay\antireplay.go`

```
package antireplay

import (
    cuckoo "github.com/seiflotfy/cuckoofilter"  // 导入 cuckoofilter 包
    "sync"  // 导入 sync 包
    "time"  // 导入 time 包
)

func NewAntiReplayWindow(AntiReplayTime int64) *AntiReplayWindow {
    arw := &AntiReplayWindow{}  // 创建 AntiReplayWindow 结构体对象
    arw.AntiReplayTime = AntiReplayTime  // 设置 AntiReplayTime 属性值
    return arw  // 返回 AntiReplayWindow 对象
}

type AntiReplayWindow struct {
    lock           sync.Mutex  // 定义互斥锁
    poolA          *cuckoo.Filter  // 定义 cuckoo.Filter 类型的指针 poolA
    poolB          *cuckoo.Filter  // 定义 cuckoo.Filter 类型的指针 poolB
    lastSwapTime   int64  // 定义最后一次交换时间
    PoolSwap       bool  // 定义布尔类型的 PoolSwap
    AntiReplayTime int64  // 定义 AntiReplayTime
}

func (aw *AntiReplayWindow) Check(sum []byte) bool {
    aw.lock.Lock()  // 加锁

    if aw.lastSwapTime == 0 {  // 如果最后一次交换时间为0
        aw.lastSwapTime = time.Now().Unix()  // 设置最后一次交换时间为当前时间
        aw.poolA = cuckoo.NewFilter(100000)  // 初始化 poolA
        aw.poolB = cuckoo.NewFilter(100000)  // 初始化 poolB
    }

    tnow := time.Now().Unix()  // 获取当前时间
    timediff := tnow - aw.lastSwapTime  // 计算时间差

    if timediff >= aw.AntiReplayTime {  // 如果时间差大于等于 AntiReplayTime
        if aw.PoolSwap {  // 如果 PoolSwap 为真
            aw.PoolSwap = false  // 设置 PoolSwap 为假
            aw.poolA.Reset()  // 重置 poolA
        } else {
            aw.PoolSwap = true  // 设置 PoolSwap 为真
            aw.poolB.Reset()  // 重置 poolB
        }
        aw.lastSwapTime = tnow  // 更新最后一次交换时间
    }

    ret := aw.poolA.InsertUnique(sum) && aw.poolB.InsertUnique(sum)  // 将数据插入 poolA 和 poolB，返回插入结果
    aw.lock.Unlock()  // 解锁
    return ret  // 返回结果
}
```