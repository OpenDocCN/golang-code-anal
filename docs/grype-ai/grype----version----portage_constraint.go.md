# `grype\grype\version\portage_constraint.go`

```
package version

import (
    "fmt"
)

type portageConstraint struct {
    raw        string
    expression constraintExpression
}

func newPortageConstraint(raw string) (portageConstraint, error) {
    if raw == "" {
        // 如果约束条件为空，则始终满足
        return portageConstraint{}, nil
    }

    // 根据原始字符串创建约束表达式
    constraints, err := newConstraintExpression(raw, newPortageComparator)
    if err != nil {
        return portageConstraint{}, fmt.Errorf("unable to parse portage constraint phrase: %w", err)
    }

    // 返回新的 Portage 约束对象
    return portageConstraint{
        raw:        raw,
        expression: constraints,
    }, nil
}

func newPortageComparator(unit constraintUnit) (Comparator, error) {
    // 创建新的 Portage 版本比较器
    ver := newPortageVersion(unit.version)
    return &ver, nil
}

func (c portageConstraint) supported(format Format) bool {
    // 检查约束是否支持指定的格式
    return format == PortageFormat
}

func (c portageConstraint) Satisfied(version *Version) (bool, error) {
    if c.raw == "" && version != nil {
        // 如果约束条件为空且版本不为空，则始终满足
        return true, nil
    } else if version == nil {
        if c.raw != "" {
            // 如果约束条件不为空且没有给定版本，则始终失败
            return false, nil
        }
        return true, nil
    }

    if !c.supported(version.Format) {
        // 如果约束不支持指定的格式，则失败
        return false, fmt.Errorf("(portage) unsupported format: %s", version.Format)
    }

    if version.rich.portVer == nil {
        // 如果版本没有给定 Portage 版本，则失败
        return false, fmt.Errorf("no rich portage version given: %+v", version)
    }

    // 检查约束表达式是否满足给定的版本
    return c.expression.satisfied(version)
}

func (c portageConstraint) String() string {
    if c.raw == "" {
        return "none (portage)"
    }
    return fmt.Sprintf("%s (portage)", c.raw)
}
```