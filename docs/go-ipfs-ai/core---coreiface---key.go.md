# `kubo\core\coreiface\key.go`

```
// 定义了 Key 接口，包含了 Name、Path 和 ID 三个方法
type Key interface {
    // 返回 key 的名称
    Name() string

    // 返回 key 的路径
    Path() path.Path

    // 返回 key 的 PeerID
    ID() peer.ID
}

// 定义了 KeyAPI 接口，包含了 Generate、Rename、List、Self、Remove、Sign 和 Verify 七个方法
type KeyAPI interface {
    // 生成新的 key，在指定的名称下存储在 keystore 中，并返回其公钥的 base58 编码的 multihash
    Generate(ctx context.Context, name string, opts ...options.KeyGenerateOption) (Key, error)

    // 将 oldName 的 key 重命名为 newName。返回重命名后的 key，以及是否覆盖了另一个 key，或者返回错误
    Rename(ctx context.Context, oldName string, newName string, opts ...options.KeyRenameOption) (Key, bool, error)

    // 列出存储在 keystore 中的 keys
    List(ctx context.Context) ([]Key, error)

    // 返回 'main' 节点的 key
    Self(ctx context.Context) (Key, error)

    // 从 keystore 中移除 key。返回移除的 key 的 ipns 路径
    Remove(ctx context.Context, name string) (Key, error)

    // 使用指定名称的 key 对给定数据进行签名。返回用于签名的 key，签名结果，以及错误
    Sign(ctx context.Context, name string, data []byte) (Key, []byte, error)

    // 验证给定的数据和签名是否匹配。返回用于验证的 key，签名和数据是否匹配，以及错误
    Verify(ctx context.Context, keyOrName string, signature, data []byte) (Key, bool, error)
}
```