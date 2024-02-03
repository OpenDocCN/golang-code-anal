# `kubo\test\dependencies\dependencies.go`

```go
// 标记为构建工具，用于构建工具包
// 根据构建标记选择需要构建的工具包
package tools

import (
    _ "github.com/Kubuxu/gocovmerge"  // 导入 gocovmerge 工具包
    _ "github.com/golangci/golangci-lint/cmd/golangci-lint"  // 导入 golangci-lint 工具包
    _ "github.com/ipfs/go-cidutil/cid-fmt"  // 导入 cid-fmt 工具包
    _ "github.com/ipfs/hang-fds"  // 导入 hang-fds 工具包
    _ "github.com/jbenet/go-random-files/random-files"  // 导入 random-files 工具包
    _ "github.com/jbenet/go-random/random"  // 导入 random 工具包
    _ "github.com/multiformats/go-multihash/multihash"  // 导入 multihash 工具包
    _ "gotest.tools/gotestsum"  // 导入 gotestsum 工具包
)
```