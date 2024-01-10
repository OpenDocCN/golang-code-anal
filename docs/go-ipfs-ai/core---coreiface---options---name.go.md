# `kubo\core\coreiface\options\name.go`

```
package options

import (
    "time"

    "github.com/ipfs/boxo/namesys"
)

const (
    DefaultNameValidTime = 24 * time.Hour
)

type NamePublishSettings struct {
    ValidTime        time.Duration
    Key              string
    TTL              *time.Duration
    CompatibleWithV1 bool
    AllowOffline     bool
}

type NameResolveSettings struct {
    Cache bool

    ResolveOpts []namesys.ResolveOption
}

type (
    NamePublishOption func(*NamePublishSettings) error
    NameResolveOption func(*NameResolveSettings) error
)

// NamePublishOptions is a function that takes a variable number of NamePublishOption functions and returns NamePublishSettings and an error
func NamePublishOptions(opts ...NamePublishOption) (*NamePublishSettings, error) {
    options := &NamePublishSettings{
        ValidTime: DefaultNameValidTime,
        Key:       "self",

        AllowOffline: false,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

// NameResolveOptions is a function that takes a variable number of NameResolveOption functions and returns NameResolveSettings and an error
func NameResolveOptions(opts ...NameResolveOption) (*NameResolveSettings, error) {
    options := &NameResolveSettings{
        Cache: true,
    }

    for _, opt := range opts {
        err := opt(options)
        if err != nil {
            return nil, err
        }
    }

    return options, nil
}

type nameOpts struct{}

var Name nameOpts

// ValidTime is an option for Name.Publish which specifies for how long the
// entry will remain valid. Default value is 24h
func (nameOpts) ValidTime(validTime time.Duration) NamePublishOption {
    return func(settings *NamePublishSettings) error {
        settings.ValidTime = validTime
        return nil
    }
}

// Key is an option for Name.Publish which specifies the key to use for
// publishing. Default value is "self" which is the node's own PeerID.
// The key parameter must be either PeerID or keystore key alias.
//
// You can use KeyAPI to list and generate more names and their respective keys.
func (nameOpts) Key(key string) NamePublishOption {
    // Add code here to specify the key to use for publishing
}
    # 返回一个函数，该函数接受一个NamePublishSettings类型的参数，并返回一个error类型的值
    return func(settings *NamePublishSettings) error {
        # 将参数settings的Key属性设置为key
        settings.Key = key
        # 返回空值
        return nil
    }
// AllowOffline是Name.Publish的一个选项，指定是否允许在节点离线时发布。默认值为false
func (nameOpts) AllowOffline(allow bool) NamePublishOption {
    return func(settings *NamePublishSettings) error {
        settings.AllowOffline = allow
        return nil
    }
}

// TTL是Name.Publish的一个选项，指定发布记录应该被缓存的时间持续时间（注意：实验性质）。
func (nameOpts) TTL(ttl time.Duration) NamePublishOption {
    return func(settings *NamePublishSettings) error {
        settings.TTL = &ttl
        return nil
    }
}

// CompatibleWithV1是Name.Publish的一个选项，指定创建的记录是否应与V1 IPNS记录向后兼容。
func (nameOpts) CompatibleWithV1(compatible bool) NamePublishOption {
    return func(settings *NamePublishSettings) error {
        settings.CompatibleWithV1 = compatible
        return nil
    }
}

// Cache是Name.Resolve的一个选项，指定是否应该使用缓存。默认值为true
func (nameOpts) Cache(cache bool) NameResolveOption {
    return func(settings *NameResolveSettings) error {
        settings.Cache = cache
        return nil
    }
}

func (nameOpts) ResolveOption(opt namesys.ResolveOption) NameResolveOption {
    return func(settings *NameResolveSettings) error {
        settings.ResolveOpts = append(settings.ResolveOpts, opt)
        return nil
    }
}
```