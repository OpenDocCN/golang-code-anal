# `kubo\thirdparty\notifier\notifier.go`

```go
// Package notifier provides a simple notification dispatcher
// meant to be embedded in larger structures who wish to allow
// clients to sign up for event notifications.
package notifier

import (
    "sync"

    process "github.com/jbenet/goprocess"
    ratelimit "github.com/jbenet/goprocess/ratelimit"
)

// Notifiee is a generic interface. Clients implement
// their own Notifiee interfaces to ensure type-safety
// of notifications:
//
//    type RocketNotifiee interface{
//      Countdown(r Rocket, countdown time.Duration)
//      LiftedOff(Rocket)
//      ReachedOrbit(Rocket)
//      Detached(Rocket, Capsule)
//      Landed(Rocket)
//    }
type Notifiee interface{}

// Notifier is a notification dispatcher. It's meant
// to be composed, and its zero-value is ready to be used.
//
//    type Rocket struct {
//      notifier notifier.Notifier
//    }
type Notifier struct {
    mu   sync.RWMutex // guards notifiees
    nots map[Notifiee]struct{}
    lim  *ratelimit.RateLimiter
}

// RateLimited returns a rate limited Notifier. only limit goroutines
// will be spawned. If limit is zero, no rate limiting happens. This
// is the same as `Notifier{}`.
func RateLimited(limit int) *Notifier {
    n := &Notifier{}
    if limit > 0 {
        n.lim = ratelimit.NewRateLimiter(process.Background(), limit)
    }
    return n
}

// Notify signs up Notifiee e for notifications. This function
// is meant to be called behind your own type-safe function(s):
//
//    // generic function for pattern-following
//    func (r *Rocket) Notify(n Notifiee) {
//      r.notifier.Notify(n)
//    }
//
//    // or as part of other functions
//    func (r *Rocket) Onboard(a Astronaut) {
//      r.astronauts = append(r.austronauts, a)
//      r.notifier.Notify(a)
//    }
func (n *Notifier) Notify(e Notifiee) {
    n.mu.Lock() // 获取写锁
    if n.nots == nil { // 如果 nots 为 nil，则初始化为一个空的 map
        n.nots = make(map[Notifiee]struct{})
    }
    n.nots[e] = struct{}{} // 将 Notifiee 添加到 nots 中
    n.mu.Unlock() // 释放写锁
}
// StopNotify 停止通知 Notifiee e。这个函数
// 应该在你自己的类型安全函数的后台调用：
//
//    // 用于遵循模式的通用函数
//    func (r *Rocket) StopNotify(n Notifiee) {
//      r.notifier.StopNotify(n)
//    }
//
//    // 或作为其他函数的一部分
//    func (r *Rocket) Detach(c Capsule) {
//      r.notifier.StopNotify(c)
//      r.capsule = nil
//    }
func (n *Notifier) StopNotify(e Notifiee) {
    n.mu.Lock()
    if n.nots != nil { // 这样零值就可以被使用了。
        delete(n.nots, e)
    }
    n.mu.Unlock()
}

// NotifyAll 使用给定的通知消息通知通知器的通知者。
// 这是通过对每个通知者调用给定的函数来完成的。它
// 应该与您自己的类型安全通知函数一起使用：
//
//    func (r *Rocket) Launch() {
//      r.notifyAll(func(n Notifiee) {
//        n.Launched(r)
//      })
//    }
//
//    // 将其设置为私有，这样只有您可以使用它。这个函数是必要的
//    // 确保您只在一个地方进行向上转型。您可以控制添加为通知者的人。
//    如果 Go 添加了泛型，也许我们可以摆脱这个
//    // 方法，但目前它就像用类型安全接口包装一个无类型容器一样。
//    func (r *Rocket) notifyAll(notify func(Notifiee)) {
//      r.notifier.NotifyAll(func(n notifier.Notifiee) {
//        notify(n.(Notifiee))
//      })
//    }
//
// 请注意：每个通知都在自己的 goroutine 中启动，因此它们
// 可以并行处理，并且无论通知做什么，它都 _永远_ 不会阻塞客户端。
// 这样消费者 _无法_ 意外地向您的对象添加阻止您的钩子。
func (n *Notifier) NotifyAll(notify func(Notifiee)) {
    n.mu.Lock()
    defer n.mu.Unlock()

    if n.nots == nil { // 这样零值就可以被使用了。
        return
    }

    // 没有速率限制。
    // 如果限制为空，则遍历通知列表，使用并发方式通知每个接收者，然后返回
    if n.lim == nil {
        for notifiee := range n.nots {
            go notify(notifiee)
        }
        return
    }

    // 使用速率限制
    // 使用速率限制执行通知操作
    n.lim.Go(func(worker process.Process) {
        // 遍历通知列表
        for notifiee := range n.nots {
            // 重新绑定循环数据，以避免数据竞争
            notifiee := notifiee 
            // 使用速率限制执行通知操作
            n.lim.LimitedGo(func(worker process.Process) {
                notify(notifiee)
            })
        }
    })
# 闭合前面的函数定义
```