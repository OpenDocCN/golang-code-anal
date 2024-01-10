# `v2ray-core\common\signal\pubsub\pubsub.go`

```
package pubsub

import (
    "errors"  // 导入 errors 包
    "sync"  // 导入 sync 包
    "time"  // 导入 time 包

    "v2ray.com/core/common"  // 导入 common 包
    "v2ray.com/core/common/signal/done"  // 导入 done 包
    "v2ray.com/core/common/task"  // 导入 task 包
)

type Subscriber struct {
    buffer chan interface{}  // 定义一个通道，用于存储消息
    done   *done.Instance  // 定义一个 done 实例
}

func (s *Subscriber) push(msg interface{}) {  // 定义 push 方法，用于向通道中推送消息
    select {
    case s.buffer <- msg:  // 如果通道未满，则将消息推送到通道中
    default:  // 如果通道已满，则不做任何操作
    }
}

func (s *Subscriber) Wait() <-chan interface{} {  // 定义 Wait 方法，返回一个只读通道
    return s.buffer  // 返回通道
}

func (s *Subscriber) Close() error {  // 定义 Close 方法，用于关闭通道
    return s.done.Close()  // 调用 done 实例的 Close 方法
}

func (s *Subscriber) IsClosed() bool {  // 定义 IsClosed 方法，用于判断通道是否已关闭
    return s.done.Done()  // 返回通道是否已关闭的状态
}

type Service struct {
    sync.RWMutex  // 定义一个读写锁
    subs  map[string][]*Subscriber  // 定义一个 map，用于存储订阅者
    ctask *task.Periodic  // 定义一个周期性任务
}

func NewService() *Service {  // 定义 NewService 方法，用于创建一个新的 Service 实例
    s := &Service{  // 创建 Service 实例
        subs: make(map[string][]*Subscriber),  // 初始化 subs 字段
    }
    s.ctask = &task.Periodic{  // 创建周期性任务
        Execute:  s.Cleanup,  // 执行 Cleanup 方法
        Interval: time.Second * 30,  // 设置执行间隔为 30 秒
    }
    return s  // 返回 Service 实例
}

// Cleanup cleans up internal caches of subscribers.
// Visible for testing only.
func (s *Service) Cleanup() error {  // 定义 Cleanup 方法，用于清理订阅者的内部缓存
    s.Lock()  // 加锁
    defer s.Unlock()  // 延迟解锁

    if len(s.subs) == 0 {  // 如果订阅者列表为空
        return errors.New("nothing to do")  // 返回错误信息
    }

    for name, subs := range s.subs {  // 遍历订阅者列表
        newSub := make([]*Subscriber, 0, len(s.subs))  // 创建一个新的订阅者列表
        for _, sub := range subs {  // 遍历订阅者
            if !sub.IsClosed() {  // 如果订阅者未关闭
                newSub = append(newSub, sub)  // 将订阅者添加到新的订阅者列表中
            }
        }
        if len(newSub) == 0 {  // 如果新的订阅者列表为空
            delete(s.subs, name)  // 删除订阅者列表中的订阅者
        } else {
            s.subs[name] = newSub  // 更新订阅者列表
        }
    }

    if len(s.subs) == 0 {  // 如果订阅者列表为空
        s.subs = make(map[string][]*Subscriber)  // 重新初始化订阅者列表
    }
    return nil  // 返回空值
}

func (s *Service) Subscribe(name string) *Subscriber {  // 定义 Subscribe 方法，用于订阅消息
    sub := &Subscriber{  // 创建订阅者实例
        buffer: make(chan interface{}, 16),  // 初始化通道，设置缓冲区大小为 16
        done:   done.New(),  // 创建一个新的 done 实例
    }
    s.Lock()  // 加锁
    subs := append(s.subs[name], sub)  // 将订阅者添加到订阅者列表中
    s.subs[name] = subs  // 更新订阅者列表
    s.Unlock()  // 解锁
    common.Must(s.ctask.Start())  // 启动周期性任务
    return sub  // 返回订阅者实例
}

func (s *Service) Publish(name string, message interface{}) {  // 定义 Publish 方法，用于发布消息
    s.RLock()  // 加读锁
    defer s.RUnlock()  // 延迟解读锁
    # 遍历名为name的订阅列表中的每个订阅
    for _, sub := range s.subs[name] {
        # 如果订阅未关闭
        if !sub.IsClosed() {
            # 将消息推送给订阅
            sub.push(message)
        }
    }
# 闭合前面的函数定义
```