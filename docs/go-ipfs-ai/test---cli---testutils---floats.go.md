# `kubo\test\cli\testutils\floats.go`

```
# 定义一个名为FloatTruncate的函数，用于截断浮点数的小数部分
package testutils

# 函数定义，接受两个参数：value为需要截断的浮点数，decimalPlaces为保留的小数位数
func FloatTruncate(value float64, decimalPlaces int) float64:
    # 初始化pow为1.0，用于存储10的幂次方
    pow := 1.0
    # 循环，将pow乘以10的幂次方，循环次数为decimalPlaces
    for i := 0; i < decimalPlaces; i++:
        pow *= 10.0
    # 返回截断后的浮点数，先将value乘以pow，然后转换为整数，再除以pow转换为浮点数
    return float64(int(value*pow)) / pow
```