# `kubo\core\coreapi\key.go`

```
package coreapi

import (
    "context" // 引入上下文包，用于处理请求的取消、超时等操作
    "crypto/rand" // 引入加密随机数生成包
    "errors" // 引入错误处理包
    "fmt" // 引入格式化包
    "sort" // 引入排序包

    "github.com/ipfs/boxo/ipns" // 引入IPNS包
    "github.com/ipfs/boxo/path" // 引入路径包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 引入核心接口包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 引入选项包
    "github.com/ipfs/kubo/tracing" // 引入追踪包
    crypto "github.com/libp2p/go-libp2p/core/crypto" // 引入加密包
    peer "github.com/libp2p/go-libp2p/core/peer" // 引入对等节点包
    "go.opentelemetry.io/otel/attribute" // 引入属性包
    "go.opentelemetry.io/otel/trace" // 引入追踪包
)

type KeyAPI CoreAPI // 定义KeyAPI类型为CoreAPI类型

type key struct { // 定义key结构体
    name   string // key的名称
    peerID peer.ID // key的对等节点ID
    path   path.Path // key的路径
}

func newKey(name string, pid peer.ID) (*key, error) { // 创建新的key
    p, err := path.NewPath("/ipns/" + ipns.NameFromPeer(pid).String()) // 生成路径
    if err != nil {
        return nil, err
    }
    return &key{
        name:   name,
        peerID: pid,
        path:   p,
    }, nil
}

// Name returns the key name
func (k *key) Name() string { // 返回key的名称
    return k.name
}

// Path returns the path of the key.
func (k *key) Path() path.Path { // 返回key的路径
    return k.path
}

// ID returns key PeerID
func (k *key) ID() peer.ID { // 返回key的对等节点ID
    return k.peerID
}

// Generate generates new key, stores it in the keystore under the specified
// name and returns a base58 encoded multihash of its public key.
func (api *KeyAPI) Generate(ctx context.Context, name string, opts ...caopts.KeyGenerateOption) (coreiface.Key, error) { // 生成新的key
    _, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Generate", trace.WithAttributes(attribute.String("name", name))) // 创建追踪span
    defer span.End() // 延迟结束span

    options, err := caopts.KeyGenerateOptions(opts...) // 获取生成key的选项
    if err != nil {
        return nil, err
    }

    if name == "self" { // 如果名称为'self'
        return nil, fmt.Errorf("cannot create key with name 'self'") // 返回错误信息
    }

    _, err = api.repo.Keystore().Get(name) // 获取指定名称的key
    if err == nil {
        return nil, fmt.Errorf("key with name '%s' already exists", name) // 返回错误信息
    }

    var sk crypto.PrivKey // 定义私钥
    var pk crypto.PubKey // 定义公钥

    switch options.Algorithm { // 根据选项的算法进行处理
    # 根据算法类型选择生成密钥对
    case "rsa":
        # 如果密钥长度未指定，则使用默认长度
        if options.Size == -1 {
            options.Size = caopts.DefaultRSALen
        }

        # 生成 RSA 密钥对
        priv, pub, err := crypto.GenerateKeyPairWithReader(crypto.RSA, options.Size, rand.Reader)
        if err != nil {
            return nil, err
        }

        # 设置私钥和公钥
        sk = priv
        pk = pub
    case "ed25519":
        # 生成 Ed25519 密钥对
        priv, pub, err := crypto.GenerateEd25519Key(rand.Reader)
        if err != nil {
            return nil, err
        }

        # 设置私钥和公钥
        sk = priv
        pk = pub
    default:
        # 如果算法类型不被识别，则返回错误
        return nil, fmt.Errorf("unrecognized key type: %s", options.Algorithm)
    }

    # 将生成的私钥存储到密钥库中
    err = api.repo.Keystore().Put(name, sk)
    if err != nil {
        return nil, err
    }

    # 从公钥生成对等节点 ID
    pid, err := peer.IDFromPublicKey(pk)
    if err != nil {
        return nil, err
    }

    # 返回新生成的密钥
    return newKey(name, pid)
// List函数返回存储在密钥库中的密钥列表。
func (api *KeyAPI) List(ctx context.Context) ([]coreiface.Key, error) {
    // 创建一个跟踪span
    _, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "List")
    // 延迟span的结束
    defer span.End()

    // 从存储库中获取密钥列表
    keys, err := api.repo.Keystore().List()
    if err != nil {
        return nil, err
    }

    // 对密钥列表进行排序
    sort.Strings(keys)

    // 创建一个包含密钥的输出列表
    out := make([]coreiface.Key, len(keys)+1)
    // 创建一个新的密钥并将其放入输出列表的第一个位置
    out[0], err = newKey("self", api.identity)
    if err != nil {
        return nil, err
    }

    // 遍历密钥列表，获取每个密钥的公钥并创建新的密钥对象
    for n, k := range keys {
        privKey, err := api.repo.Keystore().Get(k)
        if err != nil {
            return nil, err
        }

        pubKey := privKey.GetPublic()

        pid, err := peer.IDFromPublicKey(pubKey)
        if err != nil {
            return nil, err
        }

        // 将新的密钥放入输出列表
        out[n+1], err = newKey(k, pid)
        if err != nil {
            return nil, err
        }
    }
    return out, nil
}

// Rename函数将`oldName`重命名为`newName`。返回密钥和是否覆盖了另一个密钥，或者返回错误。
func (api *KeyAPI) Rename(ctx context.Context, oldName string, newName string, opts ...caopts.KeyRenameOption) (coreiface.Key, bool, error) {
    // 创建一个跟踪span
    _, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Rename", trace.WithAttributes(attribute.String("oldname", oldName), attribute.String("newname", newName)))
    // 延迟span的结束
    defer span.End()

    // 获取重命名选项
    options, err := caopts.KeyRenameOptions(opts...)
    if err != nil {
        return nil, false, err
    }
    span.SetAttributes(attribute.Bool("force", options.Force))

    ks := api.repo.Keystore()

    // 如果旧名称为"self"，则返回错误
    if oldName == "self" {
        return nil, false, fmt.Errorf("cannot rename key with name 'self'")
    }

    // 如果新名称为"self"，则返回错误
    if newName == "self" {
        return nil, false, fmt.Errorf("cannot overwrite key with name 'self'")
    }

    // 获取旧密钥并获取其公钥
    oldKey, err := ks.Get(oldName)
    if err != nil {
        return nil, false, fmt.Errorf("no key named %s was found", oldName)
    }

    pubKey := oldKey.GetPublic()

    // 从公钥获取对等ID
    pid, err := peer.IDFromPublicKey(pubKey)
    // 如果发生错误，则返回 nil、false 和错误信息
    if err != nil {
        return nil, false, err
    }

    // 这很重要，因为未来的代码将删除键 `oldName`，即使它与 newName 相同。
    if newName == oldName {
        // 根据旧名称和进程ID创建新的键
        k, err := newKey(oldName, pid)
        return k, false, err
    }

    // 是否覆盖已存在的键
    overwrite := false
    if options.Force {
        // 检查是否存在同名键
        exist, err := ks.Has(newName)
        if err != nil {
            return nil, false, err
        }

        if exist {
            // 如果存在同名键，设置覆盖标志，并删除同名键
            overwrite = true
            err := ks.Delete(newName)
            if err != nil {
                return nil, false, err
            }
        }
    }

    // 将旧键存储为新键
    err = ks.Put(newName, oldKey)
    if err != nil {
        return nil, false, err
    }

    // 删除旧键
    err = ks.Delete(oldName)
    if err != nil {
        return nil, false, err
    }

    // 根据新名称和进程ID创建新的键
    k, err := newKey(newName, pid)
    return k, overwrite, err
// Remove方法从keystore中移除密钥。返回被移除密钥的ipns路径。
func (api *KeyAPI) Remove(ctx context.Context, name string) (coreiface.Key, error) {
    // 创建一个跟踪span
    _, span := tracing.Span(ctx, "CoreAPI.KeyAPI", "Remove", trace.WithAttributes(attribute.String("name", name)))
    // 延迟span的结束
    defer span.End()

    // 获取keystore
    ks := api.repo.Keystore()

    // 如果name为"self"，则返回错误
    if name == "self" {
        return nil, fmt.Errorf("cannot remove key with name 'self'")
    }

    // 从keystore中获取被移除的密钥
    removed, err := ks.Get(name)
    if err != nil {
        return nil, fmt.Errorf("no key named %s was found", name)
    }

    // 获取被移除密钥的公钥
    pubKey := removed.GetPublic()

    // 从公钥获取peer ID
    pid, err := peer.IDFromPublicKey(pubKey)
    if err != nil {
        return nil, err
    }

    // 从keystore中删除密钥
    err = ks.Delete(name)
    if err != nil {
        return nil, err
    }

    // 返回新的密钥
    return newKey("", pid)
}

// Self方法返回当前API的身份密钥
func (api *KeyAPI) Self(ctx context.Context) (coreiface.Key, error) {
    // 如果身份为空，则返回错误
    if api.identity == "" {
        return nil, errors.New("identity not loaded")
    }

    // 返回新的"self"密钥
    return newKey("self", api.identity)
}

// signedMessagePrefix是签名消息的前缀
const signedMessagePrefix = "libp2p-key signed message:"

// Sign方法使用指定的密钥对数据进行签名
func (api *KeyAPI) Sign(ctx context.Context, name string, data []byte) (coreiface.Key, []byte, error) {
    var (
        sk  crypto.PrivKey
        err error
    )
    // 如果name为空或者为"self"，则使用私钥进行签名
    if name == "" || name == "self" {
        name = "self"
        sk = api.privateKey
    } else {
        // 否则从keystore中获取指定名称的私钥
        sk, err = api.repo.Keystore().Get(name)
    }
    if err != nil {
        return nil, nil, err
    }

    // 从私钥获取peer ID
    pid, err := peer.IDFromPrivateKey(sk)
    if err != nil {
        return nil, nil, err
    }

    // 创建新的密钥
    key, err := newKey(name, pid)
    if err != nil {
        return nil, nil, err
    }

    // 在数据前添加签名消息的前缀
    data = append([]byte(signedMessagePrefix), data...)

    // 使用私钥对数据进行签名
    sig, err := sk.Sign(data)
    if err != nil {
        return nil, nil, err
    }

    // 返回密钥和签名
    return key, sig, nil
}

// Verify方法验证数据的签名
func (api *KeyAPI) Verify(ctx context.Context, keyOrName string, signature, data []byte) (coreiface.Key, bool, error) {
    var (
        name string
        pk   crypto.PubKey
        err  error
    )
    # 如果 keyOrName 为空字符串或者为 "self"，则设置 name 为 "self"，并获取私钥的公钥
    if keyOrName == "" || keyOrName == "self" {
        name = "self"
        pk = api.privateKey.GetPublic()
    # 如果 keyOrName 在仓库中存在，则设置 name 为 keyOrName，并获取对应公钥
    } else if sk, err := api.repo.Keystore().Get(keyOrName); err == nil {
        name = keyOrName
        pk = sk.GetPublic()
    # 如果 keyOrName 是一个有效的 IPNS 名称或者 Peer ID，则提取公钥
    } else if ipnsName, err := ipns.NameFromString(keyOrName); err == nil:
        # 这适用于 IPNS 名称和 Peer ID
        name = ""
        pk, err = ipnsName.Peer().ExtractPublicKey()
        # 如果提取公钥出错，则返回错误
        if err != nil:
            return nil, false, err
    # 如果 keyOrName 既不是已知的密钥，也不是有效的 IPNS 名称或 Peer ID，则返回错误
    } else {
        return nil, false, fmt.Errorf("'%q' is not a known key, an IPNS Name, or a valid PeerID", keyOrName)
    }

    # 根据公钥获取对应的 Peer ID
    pid, err := peer.IDFromPublicKey(pk)
    # 如果获取失败，则返回错误
    if err != nil {
        return nil, false, err
    }

    # 根据 name 和 pid 创建新的密钥
    key, err := newKey(name, pid)
    # 如果创建失败，则返回错误
    if err != nil {
        return nil, false, err
    }

    # 将签名消息前缀和数据拼接在一起
    data = append([]byte(signedMessagePrefix), data...)

    # 使用公钥验证数据的签名
    valid, err := pk.Verify(data, signature)
    # 如果验证失败，则返回错误
    if err != nil {
        return nil, false, err
    }

    # 返回密钥、验证结果和空错误
    return key, valid, nil
# 闭合前面的函数定义
```