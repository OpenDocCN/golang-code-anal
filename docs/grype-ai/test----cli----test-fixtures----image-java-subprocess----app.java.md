# `grype\test\cli\test-fixtures\image-java-subprocess\app.java`

```
# 导入 IOException 类
import java.io.IOException;

# 创建 GrypeExecutionTest 类
public class GrypeExecutionTest {

  # 主函数
  public static void main(String[] args) {
    # 尝试执行以下代码
    try {
      # 创建进程构建器，执行 grype 命令，传入参数 "registry:busybox:latest", "-vv"
      ProcessBuilder builder = new ProcessBuilder("grype", "registry:busybox:latest", "-vv");

      # 将子进程的输入/输出流与当前进程的输入/输出流相连
      builder.inheritIO();
      # 启动子进程
      Process process = builder.start();

      # 等待子进程终止
      process.waitFor();

    # 捕获 IOException 或 InterruptedException 异常
    } catch (IOException | InterruptedException e) {
      # 打印异常信息
      e.printStackTrace();
    }
  }
}
```