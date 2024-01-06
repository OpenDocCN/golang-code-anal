# `grype\grype\db\test-fixtures\tls\serve.py`

```
# 导入 HTTPServer 和 SimpleHTTPRequestHandler 类
from http.server import HTTPServer, SimpleHTTPRequestHandler
# 导入 ssl 模块
import ssl
# 导入 logging 模块
import logging

# 设置端口号
port = 443
# 设置目录
directory = "www"

# 创建自定义的处理程序类，继承自 SimpleHTTPRequestHandler
class Handler(SimpleHTTPRequestHandler):
    # 初始化方法，设置目录
    def __init__(self, *args, **kwargs):
        super().__init__(*args, directory=directory, **kwargs)

    # 处理 GET 请求的方法
    def do_GET(self):
        # 记录请求头信息
        logging.error(self.headers)
        # 调用父类的 do_GET 方法处理请求
        SimpleHTTPRequestHandler.do_GET(self)

# 创建 HTTP 服务器对象，绑定地址和端口，并指定处理程序类
httpd = HTTPServer(('0.0.0.0', port), Handler)
# 创建 SSL 上下文对象
sslctx = ssl.SSLContext()
# 设置 SSL 上下文对象的选项，禁用 TLSv1 和 TLSv1.1
sslctx.options |= ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1
# 加载服务器证书和私钥，用于建立安全的 SSL/TLS 连接
sslctx.load_cert_chain(certfile='server.crt', keyfile="server.key")
# 将 HTTP 服务器的套接字包装成 SSL/TLS 套接字，用于安全通信
httpd.socket = sslctx.wrap_socket(httpd.socket, server_side=True)
# 打印服务器运行的地址和端口，使用 HTTPS 协议
print(f"Server running on https://0.0.0.0:{port}")
# 启动 HTTP 服务器，持续监听并处理客户端请求
httpd.serve_forever()
```