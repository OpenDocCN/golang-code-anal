# `kubebench-aquasecurity\integration\integration.go`

```go
package integration

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，提供 I/O 原语
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "strings"  // 导入 strings 包，用于操作字符串
    "time"  // 导入 time 包，用于时间相关操作

    batchv1 "k8s.io/api/batch/v1"  // 导入 k8s.io/api/batch/v1 包，用于批处理 API
    apiv1 "k8s.io/api/core/v1"  // 导入 k8s.io/api/core/v1 包，用于核心 API
    corev1 "k8s.io/api/core/v1"  // 导入 k8s.io/api/core/v1 包，用于核心 API
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"  // 导入 k8s.io/apimachinery/pkg/apis/meta/v1 包，用于元数据 API
    yaml "k8s.io/apimachinery/pkg/util/yaml"  // 导入 k8s.io/apimachinery/pkg/util/yaml 包，用于 YAML 处理
    "k8s.io/client-go/kubernetes"  // 导入 k8s.io/client-go/kubernetes 包，用于 Kubernetes 客户端
    "k8s.io/client-go/tools/clientcmd"  // 导入 k8s.io/client-go/tools/clientcmd 包，用于客户端配置
    "sigs.k8s.io/kind/pkg/cluster"  // 导入 sigs.k8s.io/kind/pkg/cluster 包，用于集群操作
    "sigs.k8s.io/kind/pkg/cluster/create"  // 导入 sigs.k8s.io/kind/pkg/cluster/create 包，用于创建集群
)

// 使用 Kind 运行作业
func runWithKind(ctx *cluster.Context, clientset *kubernetes.Clientset, jobName, kubebenchYAML, kubebenchImg string, timeout time.Duration) (string, error) {
    // 部署作业
    err := deployJob(clientset, kubebenchYAML, kubebenchImg)
    if err != nil {
        return "", err
    }

    // 查找作业对应的 Pod
    p, err := findPodForJob(clientset, jobName, timeout)
    if err != nil {
        return "", err
    }

    // 获取 Pod 的日志
    output := getPodLogs(clientset, p)

    // 删除作业
    err = clientset.BatchV1().Jobs(apiv1.NamespaceDefault).Delete(jobName, nil)
    if err != nil {
        return "", err
    }

    return output, nil
}

// 设置集群
func setupCluster(clusterName, kindCfg string, duration time.Duration) (*cluster.Context, error) {
    options := create.WithConfigFile(kindCfg)
    toptions := create.WaitForReady(duration)
    ctx := cluster.NewContext(clusterName)
    // 创建集群
    if err := ctx.Create(options, toptions); err != nil {
        return nil, err
    }

    return ctx, nil
}

// 获取客户端集
func getClientSet(configPath string) (*kubernetes.Clientset, error) {
    // 从配置文件构建客户端配置
    config, err := clientcmd.BuildConfigFromFlags("", configPath)
    if err != nil {
        return nil, err
    }
    // 创建 Kubernetes 客户端
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        return nil, err
    }

    return clientset, nil
}

// 部署作业
func deployJob(clientset *kubernetes.Clientset, kubebenchYAML, kubebenchImg string) error {
    // 读取作业的 YAML 文件
    jobYAML, err := ioutil.ReadFile(kubebenchYAML)
    if err != nil {
        return err
    }

    // 创建 YAML 解码器
    decoder := yaml.NewYAMLOrJSONDecoder(bytes.NewReader(jobYAML), len(jobYAML))
    // 创建作业对象
    job := &batchv1.Job{}
    # 使用短变量声明语句将 decoder.Decode(job) 的结果赋值给 err，如果有错误则返回该错误
    if err := decoder.Decode(job); err != nil {
        return err
    }
    # 将 job 中的第一个容器的镜像设置为 kubebenchImg
    job.Spec.Template.Spec.Containers[0].Image = kubebenchImg
    # 使用 clientset 创建一个新的 Job，并将结果赋值给 _，如果有错误则返回该错误
    _, err = clientset.BatchV1().Jobs(apiv1.NamespaceDefault).Create(job)
    # 返回创建 Job 的结果
    return err
}
# 定义一个函数，用于查找与指定作业关联的 Pod
func findPodForJob(clientset *kubernetes.Clientset, jobName string, duration time.Duration) (*apiv1.Pod, error) {
    # 创建一个空的失败 Pod 集合
    failedPods := make(map[string]struct{})
    # 根据作业名称创建选择器
    selector := fmt.Sprintf("job-name=%s", jobName)
    # 设置超时时间
    timeout := time.After(duration)
    # 循环查找 Pod
    for {
        # 每次循环暂停 3 秒
        time.Sleep(3 * time.Second)
    podfailed:
        select {
        # 如果超时，则返回错误
        case <-timeout:
            return nil, fmt.Errorf("podList - timed out: no Pod found for Job %s", jobName)
        default:
            # 获取指定 Namespace 下符合选择器条件的 Pod 列表
            pods, err := clientset.CoreV1().Pods(apiv1.NamespaceDefault).List(metav1.ListOptions{
                LabelSelector: selector,
            })
            if err != nil {
                return nil, err
            }
            # 打印找到的 Pod 数量
            fmt.Printf("Found (%d) pods\n", len(pods.Items))
            # 遍历找到的 Pod 列表
            for _, cp := range pods.Items {
                # 如果 Pod 在失败集合中，则跳过
                if _, found := failedPods[cp.Name]; found {
                    continue
                }
                # 如果 Pod 名称以作业名称开头
                if strings.HasPrefix(cp.Name, jobName) {
                    # 打印 Pod 名称和状态
                    fmt.Printf("pod (%s) - %#v\n", cp.Name, cp.Status.Phase)
                    # 如果 Pod 状态为成功，则返回该 Pod
                    if cp.Status.Phase == apiv1.PodSucceeded {
                        return &cp, nil
                    }
                    # 如果 Pod 状态为失败
                    if cp.Status.Phase == apiv1.PodFailed {
                        # 打印 Pod 名称、状态，并进行重试
                        fmt.Printf("pod (%s) - %s - retrying...\n", cp.Name, cp.Status.Phase)
                        fmt.Print(getPodLogs(clientset, &cp))
                        # 将失败的 Pod 加入失败集合，并跳出当前循环
                        failedPods[cp.Name] = struct{}{}
                        break podfailed
                    }
                }
            }
        }
    }
}

# 获取 Pod 的日志
func getPodLogs(clientset *kubernetes.Clientset, pod *apiv1.Pod) string {
    # 设置获取 Pod 日志的选项
    podLogOpts := corev1.PodLogOptions{}
    # 发起获取 Pod 日志的请求
    req := clientset.CoreV1().Pods(pod.Namespace).GetLogs(pod.Name, &podLogOpts)
    # 打开日志流
    podLogs, err := req.Stream()
    if err != nil {
        return "getPodLogs - error in opening stream"
    }
    defer podLogs.Close()
    # 创建一个缓冲区，用于存储日志内容
    buf := new(bytes.Buffer)
    # 将日志内容拷贝到缓冲区
    _, err = io.Copy(buf, podLogs)
    # 如果错误不为空，表示在从podLogs复制信息到缓冲区时出现错误
    if err != nil:
        # 返回错误信息
        return "getPodLogs - error in copy information from podLogs to buf"
    
    # 返回缓冲区中的字符串
    return buf.String()
# 闭合前面的函数定义
```