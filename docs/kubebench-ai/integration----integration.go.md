# `kubebench-aquasecurity\integration\integration.go`

```
// 导入所需的包
package integration

import (
	"bytes"  // 导入 bytes 包，用于操作字节
	"fmt"    // 导入 fmt 包，用于格式化输出
	"io"     // 导入 io 包，用于 I/O 操作
	"io/ioutil"  // 导入 ioutil 包，用于读取文件内容
	"strings"   // 导入 strings 包，用于字符串操作
	"time"      // 导入 time 包，用于时间操作

	batchv1 "k8s.io/api/batch/v1"  // 导入 k8s.io/api/batch/v1 包中的 batchv1
	apiv1 "k8s.io/api/core/v1"     // 导入 k8s.io/api/core/v1 包中的 apiv1
	corev1 "k8s.io/api/core/v1"    // 导入 k8s.io/api/core/v1 包中的 corev1
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"  // 导入 k8s.io/apimachinery/pkg/apis/meta/v1 包中的 metav1
	yaml "k8s.io/apimachinery/pkg/util/yaml"  // 导入 k8s.io/apimachinery/pkg/util/yaml 包中的 yaml
	"k8s.io/client-go/kubernetes"  // 导入 k8s.io/client-go/kubernetes 包中的 kubernetes
	"k8s.io/client-go/tools/clientcmd"  // 导入 k8s.io/client-go/tools/clientcmd 包中的 clientcmd
	"sigs.k8s.io/kind/pkg/cluster"  // 导入 sigs.k8s.io/kind/pkg/cluster 包中的 cluster
	"sigs.k8s.io/kind/pkg/cluster/create"  // 导入 sigs.k8s.io/kind/pkg/cluster/create 包中的 create
)
# 使用指定的上下文、客户端集和作业名称、Kubebench YAML 文件、Kubebench 镜像以及超时时间来运行作业
func runWithKind(ctx *cluster.Context, clientset *kubernetes.Clientset, jobName, kubebenchYAML, kubebenchImg string, timeout time.Duration) (string, error) {
    # 部署作业并返回可能的错误
    err := deployJob(clientset, kubebenchYAML, kubebenchImg)
    if err != nil {
        return "", err
    }

    # 查找作业对应的 Pod，并返回可能的错误
    p, err := findPodForJob(clientset, jobName, timeout)
    if err != nil {
        return "", err
    }

    # 获取 Pod 的日志输出
    output := getPodLogs(clientset, p)

    # 删除作业，并返回可能的错误
    err = clientset.BatchV1().Jobs(apiv1.NamespaceDefault).Delete(jobName, nil)
    if err != nil {
        return "", err
    }

    # 返回作业的输出和无错误
    return output, nil
}
// 设置集群，创建一个新的集群上下文
func setupCluster(clusterName, kindCfg string, duration time.Duration) (*cluster.Context, error) {
    // 使用指定的配置文件创建选项
    options := create.WithConfigFile(kindCfg)
    // 设置等待集群准备就绪的选项
    toptions := create.WaitForReady(duration)
    // 创建集群上下文
    ctx := cluster.NewContext(clusterName)
    // 如果创建集群出现错误，则返回错误
    if err := ctx.Create(options, toptions); err != nil {
        return nil, err
    }

    // 返回集群上下文
    return ctx, nil
}

// 获取客户端集
func getClientSet(configPath string) (*kubernetes.Clientset, error) {
    // 从指定的配置文件构建配置
    config, err := clientcmd.BuildConfigFromFlags("", configPath)
    // 如果构建配置出现错误，则返回错误
    if err != nil {
        return nil, err
    }
    // 使用配置创建新的 Kubernetes 客户端集
    clientset, err := kubernetes.NewForConfig(config)
    // 如果创建客户端集出现错误，则返回错误
    if err != nil {
		// 如果出现错误，返回空值和错误信息
		return nil, err
	}

	// 返回客户端集合和空错误信息
	return clientset, nil
}

// 部署作业
func deployJob(clientset *kubernetes.Clientset, kubebenchYAML, kubebenchImg string) error {
	// 读取 Kubebench YAML 文件
	jobYAML, err := ioutil.ReadFile(kubebenchYAML)
	if err != nil {
		// 如果出现错误，返回错误信息
		return err
	}

	// 创建 YAML 解码器
	decoder := yaml.NewYAMLOrJSONDecoder(bytes.NewReader(jobYAML), len(jobYAML))
	job := &batchv1.Job{}
	// 解码 YAML 文件到作业对象
	if err := decoder.Decode(job); err != nil {
		// 如果出现错误，返回错误信息
		return err
	}
	// 设置作业容器的镜像
	job.Spec.Template.Spec.Containers[0].Image = kubebenchImg

	// 创建作业
	_, err = clientset.BatchV1().Jobs(apiv1.NamespaceDefault).Create(job)
```

// 根据作业名称和持续时间查找对应的 Pod
func findPodForJob(clientset *kubernetes.Clientset, jobName string, duration time.Duration) (*apiv1.Pod, error) {
	// 保存失败的 Pod 的集合
	failedPods := make(map[string]struct{})
	// 根据作业名称创建选择器
	selector := fmt.Sprintf("job-name=%s", jobName)
	// 设置超时时间
	timeout := time.After(duration)
	// 循环查找对应的 Pod
	for {
		// 每次循环暂停 3 秒
		time.Sleep(3 * time.Second)
		// 标签选择器
	podfailed:
		select {
		// 超时处理
		case <-timeout:
			return nil, fmt.Errorf("podList - timed out: no Pod found for Job %s", jobName)
		// 默认情况
		default:
			// 获取符合选择器条件的 Pod 列表
			pods, err := clientset.CoreV1().Pods(apiv1.NamespaceDefault).List(metav1.ListOptions{
				LabelSelector: selector,
			})
			// 错误处理
			if err != nil {
				return nil, err
			}
			// 打印找到的 pod 数量
			fmt.Printf("Found (%d) pods\n", len(pods.Items))
			// 遍历 pods 列表
			for _, cp := range pods.Items {
				// 如果在 failedPods 中找到了当前 pod 的名字，则跳过
				if _, found := failedPods[cp.Name]; found {
					continue
				}

				// 如果当前 pod 的名字以 jobName 开头
				if strings.HasPrefix(cp.Name, jobName) {
					// 打印当前 pod 的名字和状态
					fmt.Printf("pod (%s) - %#v\n", cp.Name, cp.Status.Phase)
					// 如果当前 pod 的状态为成功，则返回该 pod 和空错误
					if cp.Status.Phase == apiv1.PodSucceeded {
						return &cp, nil
					}

					// 如果当前 pod 的状态为失败
					if cp.Status.Phase == apiv1.PodFailed {
						// 打印当前 pod 的名字和状态，并进行重试
						fmt.Printf("pod (%s) - %s - retrying...\n", cp.Name, cp.Status.Phase)
						// 打印当前 pod 的日志
						fmt.Print(getPodLogs(clientset, &cp))
						// 将当前 pod 加入到 failedPods 中
						failedPods[cp.Name] = struct{}{}
						// 跳出当前循环
						break podfailed
					}
				}
// 获取 Pod 的日志信息
func getPodLogs(clientset *kubernetes.Clientset, pod *apiv1.Pod) string {
    // 创建 PodLogOptions 对象
    podLogOpts := corev1.PodLogOptions{}
    // 根据 Pod 的命名空间和名称获取日志请求对象
    req := clientset.CoreV1().Pods(pod.Namespace).GetLogs(pod.Name, &podLogOpts)
    // 打开日志流
    podLogs, err := req.Stream()
    if err != nil {
        return "getPodLogs - error in opening stream"
    }
    defer podLogs.Close() // 延迟关闭日志流

    // 创建一个字节缓冲区
    buf := new(bytes.Buffer)
    // 将日志流中的信息复制到缓冲区中
    _, err = io.Copy(buf, podLogs)
    if err != nil {
        return "getPodLogs - error in copy information from podLogs to buf"
    }
# 返回buf中的字符串数据。
```