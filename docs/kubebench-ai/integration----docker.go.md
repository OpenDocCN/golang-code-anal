# `kubebench-aquasecurity\integration\docker.go`

```
package integration

import (
    "os"  // 导入操作系统相关的包
    "path/filepath"  // 导入文件路径相关的包

    "github.com/pkg/errors"  // 导入错误处理相关的包

    "sigs.k8s.io/kind/pkg/cluster"  // 导入集群相关的包
    clusternodes "sigs.k8s.io/kind/pkg/cluster/nodes"  // 导入集群节点相关的包
    "sigs.k8s.io/kind/pkg/container/docker"  // 导入 Docker 容器相关的包
    "sigs.k8s.io/kind/pkg/fs"  // 导入文件系统相关的包
    "sigs.k8s.io/kind/pkg/util/concurrent"  // 导入并发处理相关的包
)

func loadImageFromDocker(imageName string, kindCtx *cluster.Context) error {

    // 检查本地是否存在指定的镜像，并获取其 ID，如果不存在则返回错误
    _, err := docker.ImageID(imageName)
    if err != nil {
        return errors.Errorf("Image: %q not present locally", imageName)
    }

    selectedNodes, err := kindCtx.ListInternalNodes()  // 获取集群内部节点列表
    if err != nil {
        return err
    }

    // 将镜像保存为 tar 文件
    dir, err := fs.TempDir("", "image-tar")  // 创建临时目录
    if err != nil {
        return errors.Wrap(err, "failed to create tempdir")
    }
    defer os.RemoveAll(dir)  // 在函数返回前删除临时目录
    imageTarPath := filepath.Join(dir, "image.tar")  // 拼接镜像 tar 文件路径

    err = docker.Save(imageName, imageTarPath)  // 将镜像保存为 tar 文件
    if err != nil {
        return err
    }

    // 在选定的节点上加载镜像
    fns := []func() error{}  // 创建函数切片
    for _, selectedNode := range selectedNodes {
        selectedNode := selectedNode  // 捕获循环变量
        fns = append(fns, func() error {
            return loadImage(imageTarPath, &selectedNode)  // 加载镜像到节点
        })
    }
    return concurrent.UntilError(fns)  // 并发执行函数切片中的函数
}

// 将镜像 tar 文件加载到节点上
func loadImage(imageTarName string, node *clusternodes.Node) error {
    f, err := os.Open(imageTarName)  // 打开镜像 tar 文件
    if err != nil {
        return errors.Wrap(err, "failed to open image")
    }
    defer f.Close()  // 在函数返回前关闭文件
    return node.LoadImageArchive(f)  // 加载镜像到节点
}
```