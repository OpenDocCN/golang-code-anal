# `grype\test\cli\test-fixtures\image-java-subprocess\app.java`

```
# 导入 IOException 类
import java.io.IOException;

# 创建 GrypeExecutionTest 类
public class GrypeExecutionTest {

  # 主函数
  public static void main(String[] args) {
    try {
      # 创建进程构建器，指定执行命令和参数
      ProcessBuilder builder = new ProcessBuilder("grype", "registry:busybox:latest", "-vv");

      # 将子进程的输入/输出流与当前进程的输入/输出流相连
      builder.inheritIO();
      # 启动子进程
      Process process = builder.start();

      # 等待子进程结束
      process.waitFor();

    } catch (IOException | InterruptedException e) {
      # 捕获异常并打印堆栈信息
      e.printStackTrace();
    }
  }
}
```