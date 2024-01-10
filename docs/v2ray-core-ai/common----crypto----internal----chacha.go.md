# `v2ray-core\common\crypto\internal\chacha.go`

```
package internal

// 使用go:generate指令来运行chacha_core_gen.go文件

import (
    "encoding/binary"
)

const (
    wordSize  = 4                    // ChaCha20的字大小
    stateSize = 16                   // ChaCha20的状态大小，以字为单位
    blockSize = stateSize * wordSize // ChaCha20的块大小，以字节为单位
)

type ChaCha20Stream struct {
    state  [stateSize]uint32 // 作为16个32位字的数组的状态
    block  [blockSize]byte   // 作为64字节的数组的密钥流
    offset int               // 在块中使用的字节的偏移量
    rounds int               // 轮数
}

func NewChaCha20Stream(key []byte, nonce []byte, rounds int) *ChaCha20Stream {
    s := new(ChaCha20Stream)
    // 256位密钥的魔术常量
    s.state[0] = 0x61707865
    s.state[1] = 0x3320646e
    s.state[2] = 0x79622d32
    s.state[3] = 0x6b206574

    for i := 0; i < 8; i++ {
        s.state[i+4] = binary.LittleEndian.Uint32(key[i*4 : i*4+4])
    }

    switch len(nonce) {
    case 8:
        s.state[14] = binary.LittleEndian.Uint32(nonce[0:])
        s.state[15] = binary.LittleEndian.Uint32(nonce[4:])
    case 12:
        s.state[13] = binary.LittleEndian.Uint32(nonce[0:4])
        s.state[14] = binary.LittleEndian.Uint32(nonce[4:8])
        s.state[15] = binary.LittleEndian.Uint32(nonce[8:12])
    default:
        panic("bad nonce length")
    }

    s.rounds = rounds
    ChaCha20Block(&s.state, s.block[:], s.rounds)
    return s
}

func (s *ChaCha20Stream) XORKeyStream(dst, src []byte) {
    // 以64字节块为单位遍历输入，减去先前使用的密钥流量。当处理块大小能够被64整除时，这将产生最佳结果。
    i := 0
    max := len(src)
    # 循环，直到 i 小于 max
    for i < max {
        # 计算剩余的块大小
        gap := blockSize - s.offset

        # 计算限制范围
        limit := i + gap
        if limit > max {
            limit = max
        }

        # 保存当前偏移量
        o := s.offset
        # 循环处理数据
        for j := i; j < limit; j++ {
            # 对源数据和密钥块进行异或操作，存入目标数组
            dst[j] = src[j] ^ s.block[o]
            o++
        }

        # 更新 i
        i += gap
        # 更新偏移量
        s.offset = o

        # 如果偏移量达到块大小
        if o == blockSize {
            # 重置偏移量
            s.offset = 0
            # 更新状态
            s.state[12]++
            # 调用 ChaCha20Block 函数处理状态、密钥块和轮数
            ChaCha20Block(&s.state, s.block[:], s.rounds)
        }
    }
# 闭合前面的函数定义
```