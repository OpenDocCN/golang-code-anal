# `grype\test\cli\test-fixtures\image-node-subprocess\app.js`

```
# 调用子进程执行 grype 命令，传入参数 "-vv" 和 "registry:busybox:latest"
require("child_process").spawn("grype", [
    "-vv",
    "registry:busybox:latest",
], {
  // 我们希望看到来自 stdout/stderr 的任何输出，因此它们从父进程继承。
  // 真正的测试是确保当没有从 stdin 提供任何内容时，管道输入不会永远挂起，
  // 并且当用户有输入时不使用 stdin。也就是说，确保我们不仅仅使用 "stdin is a pipe" 作为期望从 stdin 接收分析输入的唯一指示器。
  stdio: ["pipe", "inherit", "inherit"]
});
```