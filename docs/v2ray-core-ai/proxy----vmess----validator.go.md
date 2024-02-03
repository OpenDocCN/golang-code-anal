# `v2ray-core\proxy\vmess\validator.go`

```go
// +build !confonly

package vmess

import (
    "crypto/hmac" // 导入加密哈希消息认证码包
    "crypto/sha256" // 导入SHA-256哈希算法包
    "hash" // 导入哈希函数包
    "hash/crc64" // 导入CRC64哈希算法包
    "strings" // 导入字符串处理包
    "sync" // 导入同步包
    "sync/atomic" // 导入原子操作包
    "time" // 导入时间包

    "v2ray.com/core/common" // 导入V2Ray通用包
    "v2ray.com/core/common/dice" // 导入随机数生成包
    "v2ray.com/core/common/protocol" // 导入V2Ray协议包
    "v2ray.com/core/common/serial" // 导入序列化包
    "v2ray.com/core/common/task" // 导入任务包
    "v2ray.com/core/proxy/vmess/aead" // 导入VMess AEAD包
)

const (
    updateInterval   = 10 * time.Second // 更新间隔为10秒
    cacheDurationSec = 120 // 缓存持续时间为120秒
)

type user struct {
    user    protocol.MemoryUser // 用户信息
    lastSec protocol.Timestamp // 上次验证时间
}

// TimedUserValidator is a user Validator based on time.
type TimedUserValidator struct {
    sync.RWMutex // 读写锁
    users    []*user // 用户列表
    userHash map[[16]byte]indexTimePair // 用户哈希表
    hasher   protocol.IDHash // ID哈希
    baseTime protocol.Timestamp // 基准时间
    task     *task.Periodic // 定时任务

    behaviorSeed  uint64 // 行为种子
    behaviorFused bool // 行为融合

    aeadDecoderHolder *aead.AuthIDDecoderHolder // AEAD解码器持有者
}

type indexTimePair struct {
    user    *user // 用户
    timeInc uint32 // 时间增量

    taintedFuse *uint32 // 污染熔断
}

// NewTimedUserValidator creates a new TimedUserValidator.
func NewTimedUserValidator(hasher protocol.IDHash) *TimedUserValidator {
    tuv := &TimedUserValidator{
        users:             make([]*user, 0, 16), // 初始化用户列表
        userHash:          make(map[[16]byte]indexTimePair, 1024), // 初始化用户哈希表
        hasher:            hasher, // 设置哈希算法
        baseTime:          protocol.Timestamp(time.Now().Unix() - cacheDurationSec*2), // 设置基准时间
        aeadDecoderHolder: aead.NewAuthIDDecoderHolder(), // 创建AEAD解码器持有者
    }
    tuv.task = &task.Periodic{
        Interval: updateInterval, // 设置定时任务间隔
        Execute: func() error {
            tuv.updateUserHash() // 更新用户哈希表
            return nil
        },
    }
    common.Must(tuv.task.Start()) // 启动定时任务
    return tuv
}

func (v *TimedUserValidator) generateNewHashes(nowSec protocol.Timestamp, user *user) {
    var hashValue [16]byte // 哈希值数组
    genEndSec := nowSec + cacheDurationSec // 生成结束时间为当前时间加上缓存持续时间
    # 定义一个匿名函数，用于生成给定 ID 的哈希值
    genHashForID := func(id *protocol.ID) {
        # 计算 ID 的哈希值
        idHash := v.hasher(id.Bytes())
        # 获取生成哈希值的起始时间
        genBeginSec := user.lastSec
        # 如果起始时间小于当前时间减去缓存持续时间，则将起始时间设置为当前时间减去缓存持续时间
        if genBeginSec < nowSec-cacheDurationSec {
            genBeginSec = nowSec - cacheDurationSec
        }
        # 循环遍历起始时间到结束时间的时间戳
        for ts := genBeginSec; ts <= genEndSec; ts++ {
            # 将 ID 的哈希值与时间戳转换成字节流并写入哈希值
            common.Must2(serial.WriteUint64(idHash, uint64(ts)))
            # 对哈希值进行求和并重置哈希值
            idHash.Sum(hashValue[:0])
            idHash.Reset()

            # 将用户哈希值映射到索引时间对，并存入用户哈希表中
            v.userHash[hashValue] = indexTimePair{
                user:        user,
                timeInc:     uint32(ts - v.baseTime),
                taintedFuse: new(uint32),
            }
        }
    }

    # 获取用户账户信息
    account := user.user.Account.(*MemoryAccount)

    # 为用户 ID 生成哈希值
    genHashForID(account.ID)
    # 遍历用户的备用 ID，为每个 ID 生成哈希值
    for _, id := range account.AlterIDs {
        genHashForID(id)
    }
    # 更新用户的最后一次生成哈希值的时间戳
    user.lastSec = genEndSec
# 删除过期的哈希值
func (v *TimedUserValidator) removeExpiredHashes(expire uint32) {
    # 遍历用户哈希表
    for key, pair := range v.userHash {
        # 如果哈希值的时间增量小于过期时间
        if pair.timeInc < expire {
            # 删除该哈希值
            delete(v.userHash, key)
        }
    }
}

# 更新用户哈希值
func (v *TimedUserValidator) updateUserHash() {
    # 获取当前时间
    now := time.Now()
    # 将当前时间转换为协议时间戳
    nowSec := protocol.Timestamp(now.Unix())
    # 加锁
    v.Lock()
    # 延迟解锁
    defer v.Unlock()

    # 遍历用户列表
    for _, user := range v.users {
        # 生成新的哈希值
        v.generateNewHashes(nowSec, user)
    }

    # 计算过期时间
    expire := protocol.Timestamp(now.Unix() - cacheDurationSec)
    # 如果过期时间大于基准时间
    if expire > v.baseTime {
        # 删除过期的哈希值
        v.removeExpiredHashes(uint32(expire - v.baseTime))
    }
}

# 添加用户
func (v *TimedUserValidator) Add(u *protocol.MemoryUser) error {
    # 加锁
    v.Lock()
    # 延迟解锁
    defer v.Unlock()

    # 获取当前时间的Unix时间戳
    nowSec := time.Now().Unix()

    # 创建用户对象
    uu := &user{
        user:    *u,
        lastSec: protocol.Timestamp(nowSec - cacheDurationSec),
    }
    # 将用户添加到用户列表
    v.users = append(v.users, uu)
    # 生成新的哈希值
    v.generateNewHashes(protocol.Timestamp(nowSec), uu)

    # 获取用户账户
    account := uu.user.Account.(*MemoryAccount)
    # 如果行为未融合
    if !v.behaviorFused {
        # 创建哈希函数
        hashkdf := hmac.New(func() hash.Hash { return sha256.New() }, []byte("VMESSBSKDF"))
        hashkdf.Write(account.ID.Bytes())
        # 更新行为种子
        v.behaviorSeed = crc64.Update(v.behaviorSeed, crc64.MakeTable(crc64.ECMA), hashkdf.Sum(nil))
    }

    # 复制用户命令密钥
    var cmdkeyfl [16]byte
    copy(cmdkeyfl[:], account.ID.CmdKey())
    # 添加用户到AEAD解码器持有者
    v.aeadDecoderHolder.AddUser(cmdkeyfl, u)

    return nil
}

# 获取用户
func (v *TimedUserValidator) Get(userHash []byte) (*protocol.MemoryUser, protocol.Timestamp, bool, error) {
    # 延迟解锁
    defer v.RUnlock()
    # 加锁
    v.RLock()

    # 设置行为融合标志
    v.behaviorFused = true

    # 复制固定大小的哈希值
    var fixedSizeHash [16]byte
    copy(fixedSizeHash[:], userHash)
    # 查找哈希值
    pair, found := v.userHash[fixedSizeHash]
    if found {
        user := pair.user.user
        # 如果哈希值未被污染
        if atomic.LoadUint32(pair.taintedFuse) == 0 {
            # 返回用户、时间戳、true和无错误
            return &user, protocol.Timestamp(pair.timeInc) + v.baseTime, true, nil
        }
        # 返回nil、0、false和错误
        return nil, 0, false, ErrTainted
    }
    # 返回nil、0、false和错误
    return nil, 0, false, ErrNotFound
}
// 获取给定用户哈希对应的 AEAD 对象和用户信息，使用读锁
func (v *TimedUserValidator) GetAEAD(userHash []byte) (*protocol.MemoryUser, bool, error) {
    // 在函数返回时释放读锁
    defer v.RUnlock()
    // 获取读锁
    v.RLock()
    // 创建一个长度为 16 的数组 userHashFL，并将 userHash 的内容复制到其中
    var userHashFL [16]byte
    copy(userHashFL[:], userHash)

    // 使用 AEAD 解码器匹配用户哈希，返回用户信息和错误
    userd, err := v.aeadDecoderHolder.Match(userHashFL)
    if err != nil {
        return nil, false, err
    }
    return userd.(*protocol.MemoryUser), true, err
}

// 移除指定邮箱对应的用户信息，使用写锁
func (v *TimedUserValidator) Remove(email string) bool {
    // 获取写锁
    v.Lock()
    // 在函数返回时释放写锁
    defer v.Unlock()

    // 初始化索引为 -1
    idx := -1
    // 遍历用户列表
    for i := range v.users {
        // 如果邮箱匹配成功
        if strings.EqualFold(v.users[i].user.Email, email) {
            // 记录索引，复制用户账户的命令密钥到 cmdkeyfl，移除用户对应的 AEAD 对象
            idx = i
            var cmdkeyfl [16]byte
            copy(cmdkeyfl[:], v.users[i].user.Account.(*MemoryAccount).ID.CmdKey())
            v.aeadDecoderHolder.RemoveUser(cmdkeyfl)
            break
        }
    }
    // 如果未找到匹配的邮箱，返回 false
    if idx == -1 {
        return false
    }
    // 获取用户列表长度
    ulen := len(v.users)

    // 将待删除用户信息替换为最后一个用户信息，删除最后一个用户信息，更新用户列表
    v.users[idx] = v.users[ulen-1]
    v.users[ulen-1] = nil
    v.users = v.users[:ulen-1]

    return true
}

// 实现 common.Closable 接口的 Close 方法
func (v *TimedUserValidator) Close() error {
    return v.task.Close()
}

// 获取行为种子，使用写锁
func (v *TimedUserValidator) GetBehaviorSeed() uint64 {
    // 获取写锁
    v.Lock()
    // 在函数返回时释放写锁
    defer v.Unlock()
    // 标记行为种子已使用，如果行为种子为 0，则生成一个新的行为种子
    v.behaviorFused = true
    if v.behaviorSeed == 0 {
        v.behaviorSeed = dice.RollUint64()
    }
    return v.behaviorSeed
}

// 烧毁指定用户哈希对应的污点保护，使用读锁
func (v *TimedUserValidator) BurnTaintFuse(userHash []byte) error {
    // 获取读锁
    v.RLock()
    // 在函数返回时释放读锁
    defer v.RUnlock()
    // 创建一个长度为 16 的数组 userHashFL，并将 userHash 的内容复制到其中
    var userHashFL [16]byte
    copy(userHashFL[:], userHash)

    // 查找用户哈希对应的污点保护，如果找到则烧毁，否则返回错误
    pair, found := v.userHash[userHashFL]
    if found {
        if atomic.CompareAndSwapUint32(pair.taintedFuse, 0, 1) {
            return nil
        }
        return ErrTainted
    }
    return ErrNotFound
}

// 定义错误变量 ErrNotFound
var ErrNotFound = newError("Not Found")

// 定义错误变量 ErrTainted
var ErrTainted = newError("ErrTainted")
```