# `grype\grype\version\semantic_constraint_test.go`

```
package version

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestVersionSemantic(t *testing.T) {
	tests := []testCase{
		// empty values
		{version: "2.3.1", constraint: "", satisfied: true},  // 测试空值情况
		// typical cases
		{version: "0.9.9-r0", constraint: "< 0.9.12-r1", satisfied: true}, // 典型案例
		{version: "1.5.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: true},  // 典型案例
		{version: "0.2.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: true},  // 典型案例
		{version: "0.0.1", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},  // 典型案例
		{version: "0.6.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},  // 典型案例
		{version: "2.5.0", constraint: "> 0.1.0, < 0.5.0 || > 1.0.0, < 2.0.0", satisfied: false},  // 典型案例
		{version: "2.3.1", constraint: "2.3.1", satisfied: true},  // 典型案例
# 版本号为 "2.3.1"，满足约束 "= 2.3.1"，返回 true
{version: "2.3.1", constraint: "= 2.3.1", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "  =   2.3.1"，返回 true
{version: "2.3.1", constraint: "  =   2.3.1", satisfied: true},
# 版本号为 "2.3.1"，满足约束 ">= 2.3.1"，返回 true
{version: "2.3.1", constraint: ">= 2.3.1", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2.0.0"，返回 true
{version: "2.3.1", constraint: "> 2.0.0", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2.0"，返回 true
{version: "2.3.1", constraint: "> 2.0", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2"，返回 true
{version: "2.3.1", constraint: "> 2", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2, < 3"，返回 true
{version: "2.3.1", constraint: "> 2, < 3", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2.3, < 3.1"，返回 true
{version: "2.3.1", constraint: "> 2.3, < 3.1", satisfied: true},
# 版本号为 "2.3.1"，满足约束 "> 2.3.0, < 3.1"，返回 true
{version: "2.3.1", constraint: "> 2.3.0, < 3.1", satisfied: true},
# 版本号为 "2.3.1"，满足约束 ">= 2.3.1, < 3.1"，返回 true
{version: "2.3.1", constraint: ">= 2.3.1, < 3.1", satisfied: true},
# 版本号为 "2.3.1"，不满足约束 "  =  2.3.2"，返回 false
{version: "2.3.1", constraint: "  =  2.3.2", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 ">= 2.3.2"，返回 false
{version: "2.3.1", constraint: ">= 2.3.2", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 "> 2.3.1"，返回 false
{version: "2.3.1", constraint: "> 2.3.1", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 "< 2.0.0"，返回 false
{version: "2.3.1", constraint: "< 2.0.0", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 "< 2.0"，返回 false
{version: "2.3.1", constraint: "< 2.0", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 "< 2"，返回 false
{version: "2.3.1", constraint: "< 2", satisfied: false},
# 版本号为 "2.3.1"，不满足约束 "< 2, > 3"，返回 false
{version: "2.3.1", constraint: "< 2, > 3", satisfied: false},
# 版本号为 "2.3.1+meta"，满足约束 "2.3.1"，返回 true
{version: "2.3.1+meta", constraint: "2.3.1", satisfied: true},
# 版本号为 "2.3.1+meta"，满足约束 "= 2.3.1"，返回 true
{version: "2.3.1+meta", constraint: "= 2.3.1", satisfied: true},
# 版本号为 "2.3.1+meta"，满足约束 "  =   2.3.1"，返回 true
{version: "2.3.1+meta", constraint: "  =   2.3.1", satisfied: true},
# 以下是一系列版本约束条件和对应的版本号，以及是否满足约束条件的标志

# 示例1：版本号为 "2.3.1+meta"，约束条件为 ">= 2.3.1"，满足约束条件
{version: "2.3.1+meta", constraint: ">= 2.3.1", satisfied: true},

# 示例2：版本号为 "2.3.1+meta"，约束条件为 "> 2.0.0"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2.0.0", satisfied: true},

# 示例3：版本号为 "2.3.1+meta"，约束条件为 "> 2.0"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2.0", satisfied: true},

# 示例4：版本号为 "2.3.1+meta"，约束条件为 "> 2"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2", satisfied: true},

# 示例5：版本号为 "2.3.1+meta"，约束条件为 "> 2, < 3"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2, < 3", satisfied: true},

# 示例6：版本号为 "2.3.1+meta"，约束条件为 "> 2.3, < 3.1"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2.3, < 3.1", satisfied: true},

# 示例7：版本号为 "2.3.1+meta"，约束条件为 "> 2.3.0, < 3.1"，满足约束条件
{version: "2.3.1+meta", constraint: "> 2.3.0, < 3.1", satisfied: true},

# 示例8：版本号为 "2.3.1+meta"，约束条件为 ">= 2.3.1, < 3.1"，满足约束条件
{version: "2.3.1+meta", constraint: ">= 2.3.1, < 3.1", satisfied: true},

# 示例9：版本号为 "2.3.1+meta"，约束条件为 "  =  2.3.2"，不满足约束条件
{version: "2.3.1+meta", constraint: "  =  2.3.2", satisfied: false},

# 示例10：版本号为 "2.3.1+meta"，约束条件为 ">= 2.3.2"，不满足约束条件
{version: "2.3.1+meta", constraint: ">= 2.3.2", satisfied: false},

# 示例11：版本号为 "2.3.1+meta"，约束条件为 "> 2.3.1"，不满足约束条件
{version: "2.3.1+meta", constraint: "> 2.3.1", satisfied: false},

# 示例12：版本号为 "2.3.1+meta"，约束条件为 "< 2.0.0"，不满足约束条件
{version: "2.3.1+meta", constraint: "< 2.0.0", satisfied: false},

# 示例13：版本号为 "2.3.1+meta"，约束条件为 "< 2.0"，不满足约束条件
{version: "2.3.1+meta", constraint: "< 2.0", satisfied: false},

# 示例14：版本号为 "2.3.1+meta"，约束条件为 "< 2"，不满足约束条件
{version: "2.3.1+meta", constraint: "< 2", satisfied: false},

# 示例15：版本号为 "2.3.1+meta"，约束条件为 "< 2, > 3"，不满足约束条件
{version: "2.3.1+meta", constraint: "< 2, > 3", satisfied: false},

# 示例16：版本号为 "1.0.0-alpha"，约束条件为 "> 1.0.0-alpha.1"，不满足约束条件
{version: "1.0.0-alpha", constraint: "> 1.0.0-alpha.1", satisfied: false},

# 示例17：版本号为 "1.0.0-alpha"，约束条件为 "< 1.0.0-alpha.1"，满足约束条件
{version: "1.0.0-alpha", constraint: "< 1.0.0-alpha.1", satisfied: true},

# 以下是注释
// from https://github.com/hashicorp/go-version/issues/61
// and https://semver.org/#spec-item-11
// A larger set of pre-release fields has a higher precedence than a smaller set, if all of the preceding identifiers are equal.
# 创建包含版本和约束的对象列表
{version: "1.0.0-alpha.1", constraint: "> 1.0.0-alpha.beta", satisfied: false},
{version: "1.0.0-alpha.1", constraint: "< 1.0.0-alpha.beta", satisfied: true},
{version: "1.0.0-alpha.beta", constraint: "> 1.0.0-beta", satisfied: false},
{version: "1.0.0-alpha.beta", constraint: "< 1.0.0-beta", satisfied: true},
{version: "1.0.0-beta", constraint: "> 1.0.0-beta.2", satisfied: false},
{version: "1.0.0-beta", constraint: "< 1.0.0-beta.2", satisfied: true},
{version: "1.0.0-beta.2", constraint: "> 1.0.0-beta.11", satisfied: false},
{version: "1.0.0-beta.2", constraint: "< 1.0.0-beta.11", satisfied: true},
{version: "1.0.0-beta.11", constraint: "> 1.0.0-rc.1", satisfied: false},
{version: "1.0.0-beta.11", constraint: "< 1.0.0-rc.1", satisfied: true},
{version: "1.0.0-rc.1", constraint: "> 1.0.0", satisfied: false},
{version: "1.0.0-rc.1", constraint: "< 1.0.0", satisfied: true},
{version: "1.20rc1", constraint: " = 1.20.0-rc1", satisfied: true},
{version: "1.21rc2", constraint: " = 1.21.1", satisfied: false},
{version: "1.21rc2", constraint: " = 1.21", satisfied: false},
{version: "1.21rc2", constraint: " = 1.21-rc2", satisfied: true},
{version: "1.21rc2", constraint: " = 1.21.0-rc2", satisfied: true},
{version: "1.21rc2", constraint: " = 1.21.0rc2", satisfied: true},
{version: "1.0.0-alpha.1", constraint: "> 1.0.0-alpha.1", satisfied: false},
{version: "1.0.0-alpha.2", constraint: "> 1.0.0-alpha.1", satisfied: true},
# 定义了一组测试数据，每个数据包含版本号、约束条件和是否满足的标志
{version: "1.2.0-beta", constraint: ">1.0, <2.0", satisfied: true},
{version: "1.2.0-beta", constraint: ">1.0", satisfied: true},
{version: "1.2.0-beta", constraint: "<2.0", satisfied: true},
{version: "1.2.0", constraint: ">1.0, <2.0", satisfied: true},

# 遍历测试数据
for _, test := range tests {
    # 对每个测试数据运行子测试
    t.Run(test.tName(), func(t *testing.T) {
        # 根据约束条件创建语义约束对象
        constraint, err := newSemanticConstraint(test.constraint)
        # 断言创建约束对象时没有错误
        assert.NoError(t, err, "unexpected error from newSemanticConstraint: %v", err)
        
        # 对测试数据进行版本约束断言
        test.assertVersionConstraint(t, SemanticFormat, constraint)
    })
}
```