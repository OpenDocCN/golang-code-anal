# `v2ray-core\transport\internet\headers\wechat\wechat_test.go`

```
package wechat_test

import (
    "context"  // 导入上下文包，用于处理请求的取消、超时等操作
    "testing"  // 导入测试包，用于编写和运行测试函数

    "v2ray.com/core/common"  // 导入v2ray的common包，用于处理通用的操作
    "v2ray.com/core/common/buf"  // 导入v2ray的buf包，用于处理缓冲区操作
    . "v2ray.com/core/transport/internet/headers/wechat"  // 导入v2ray的微信传输协议包

)

func TestUTPWrite(t *testing.T) {
    // 创建一个新的视频聊天对象，并返回视频原始数据和错误信息
    videoRaw, err := NewVideoChat(context.Background(), &VideoConfig{})
    // 如果有错误发生，则终止程序
    common.Must(err)

    // 将视频原始数据转换为视频聊天对象
    video := videoRaw.(*VideoChat)

    // 创建一个新的缓冲区
    payload := buf.New()
    // 将视频数据序列化到缓冲区中
    video.Serialize(payload.Extend(video.Size()))

    // 如果缓冲区长度不等于视频数据长度，则输出错误信息
    if payload.Len() != video.Size() {
        t.Error("expected payload size ", video.Size(), " but got ", payload.Len())
    }
}
```