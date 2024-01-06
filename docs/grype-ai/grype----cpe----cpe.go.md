# `grype\grype\cpe\cpe.go`

```
// 导入必要的包
package cpe

import (
	"github.com/anchore/grype/internal/log"  // 导入日志包
	"github.com/anchore/syft/syft/cpe"  // 导入CPE包
)

// NewSlice函数用于创建CPE对象的切片
func NewSlice(cpeStrs ...string) ([]cpe.CPE, error) {
	var cpes []cpe.CPE  // 创建CPE对象的切片
	for _, c := range cpeStrs {  // 遍历传入的CPE字符串
		value, err := cpe.New(c)  // 根据CPE字符串创建CPE对象
		if err != nil {  // 如果创建CPE对象出错
			log.Warnf("excluding invalid CPE %q: %v", c, err)  // 记录日志，排除无效的CPE
			continue  // 继续下一个CPE字符串的处理
		}

		cpes = append(cpes, value)  // 将有效的CPE对象添加到切片中
	}
	return cpes, nil  // 返回CPE对象的切片和nil错误
}
# 定义一个函数，用于在候选列表中查找与给定CPE对象匹配（不考虑版本）的对象，并返回匹配的CPE对象列表
func MatchWithoutVersion(c cpe.CPE, candidates []cpe.CPE) []cpe.CPE {
	# 创建一个空的CPE对象列表，用于存储匹配的结果
	matches := make([]cpe.CPE, 0)
	# 遍历候选列表中的每个CPE对象
	for _, candidate := range candidates {
		# 复制候选对象，以便进行版本匹配
		canCopy := candidate
		# 如果给定的CPE对象与复制的候选对象匹配（不考虑版本）
		if c.MatchWithoutVersion(&canCopy) {
			# 将匹配的候选对象添加到结果列表中
			matches = append(matches, candidate)
		}
	}
	# 返回匹配的CPE对象列表
	return matches
}
```