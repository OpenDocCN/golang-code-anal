# `kubo\repo\onlyone.go`

```go
package repo

import (
    "sync"
)

// OnlyOne tracks open Repos by arbitrary key and returns the already
// open one.
type OnlyOne struct {
    mu     sync.Mutex
    active map[interface{}]*ref
}

// Open a Repo identified by key. If Repo is not already open, the
// open function is called, and the result is remembered for further
// use.
//
// Key must be comparable, or Open will panic. Make sure to pick keys
// that are unique across different concrete Repo implementations,
// e.g. by creating a local type:
//
//    type repoKey string
//    r, err := o.Open(repoKey(path), open)
//
// Call Repo.Close when done.
func (o *OnlyOne) Open(key interface{}, open func() (Repo, error)) (Repo, error) {
    // 加锁，确保并发安全
    o.mu.Lock()
    // 在函数返回时解锁
    defer o.mu.Unlock()
    // 如果活跃的 map 为空，初始化它
    if o.active == nil {
        o.active = make(map[interface{}]*ref)
    }

    // 查找 key 对应的 Repo 是否已经存在
    item, found := o.active[key]
    // 如果不存在
    if !found {
        // 调用 open 函数打开 Repo
        repo, err := open()
        // 如果出错，返回错误
        if err != nil {
            return nil, err
        }
        // 创建 ref 对象，记录 Repo 的引用次数
        item = &ref{
            parent: o,
            key:    key,
            Repo:   repo,
        }
        // 将 ref 对象加入活跃的 map 中
        o.active[key] = item
    }
    // 增加引用次数
    item.refs++
    // 返回 Repo 对象和 nil 错误
    return item, nil
}

type ref struct {
    parent *OnlyOne
    key    interface{}
    refs   uint32
    Repo
}

// 确保 ref 类型实现了 Repo 接口
var _ Repo = (*ref)(nil)

func (r *ref) Close() error {
    // 加锁，确保并发安全
    r.parent.mu.Lock()
    // 在函数返回时解锁
    defer r.parent.mu.Unlock()

    // 减少引用次数
    r.refs--
    // 如果还有其他引用，直接返回
    if r.refs > 0 {
        // others are holding it open
        return nil
    }

    // 最后一个引用，从活跃的 map 中删除，并调用 Repo 的 Close 方法
    delete(r.parent.active, r.key)
    return r.Repo.Close()
}
```