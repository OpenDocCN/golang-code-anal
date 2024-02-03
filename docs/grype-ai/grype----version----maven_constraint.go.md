# `grype\grype\version\maven_constraint.go`

```go
package version

import "fmt"

type mavenConstraint struct {
    raw        string
    expression constraintExpression
}

func newMavenConstraint(raw string) (mavenConstraint, error) {
    if raw == "" {
        // 如果原始字符串为空，则返回空的 mavenConstraint 对象
        return mavenConstraint{}, nil
    }

    // 根据原始字符串创建约束表达式
    constraints, err := newConstraintExpression(raw, newMavenComparator)
    if err != nil {
        // 如果无法解析 maven 约束短语，则返回错误
        return mavenConstraint{}, fmt.Errorf("unable to parse maven constraint phrase: %w", err)
    }

    // 返回包含原始字符串和约束表达式的 mavenConstraint 对象
    return mavenConstraint{
        raw:        raw,
        expression: constraints,
    }, nil
}

func newMavenComparator(unit constraintUnit) (Comparator, error) {
    // 根据约束单元的版本创建 Maven 版本对象
    ver, err := newMavenVersion(unit.version)
    if err != nil {
        // 如果无法解析约束版本，则返回错误
        return nil, fmt.Errorf("unable to parse constraint version (%s): %w", unit.version, err)
    }

    // 返回 Maven 版本对象
    return ver, nil
}

func (c mavenConstraint) supported(format Format) bool {
    // 检查是否支持指定格式的约束
    return format == MavenFormat
}

func (c mavenConstraint) Satisfied(version *Version) (satisfied bool, err error) {
    if c.raw == "" && version != nil {
        // 如果原始字符串为空且版本不为空，则始终满足约束
        return true, nil
    }

    if version == nil {
        if c.raw != "" {
            // 如果约束不为空且未给定版本，则始终不满足约束
            return false, nil
        }

        return true, nil
    }

    if !c.supported(version.Format) {
        // 如果不支持指定格式的约束，则返回错误
        return false, fmt.Errorf("(maven) unsupported format: %s", version.Format)
    }

    if version.rich.mavenVer == nil {
        // 如果未给定 Maven 版本，则返回错误
        return false, fmt.Errorf("no rich apk version given: %+v", version)
    }

    // 检查约束表达式是否满足给定版本
    return c.expression.satisfied(version)
}

func (c mavenConstraint) String() string {
    if c.raw == "" {
        // 如果原始字符串为空，则返回 "none (maven)"
        return "none (maven)"
    }

    // 返回包含原始字符串的 mavenConstraint 字符串表示形式
    return fmt.Sprintf("%s (maven)", c.raw)
}
```