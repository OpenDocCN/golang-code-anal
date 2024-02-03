# `grype\grype\db\test-fixtures\tls\listing.py`

```go
# 导入 urllib.request 模块，用于发送 HTTP 请求
import urllib.request
# 导入 json 模块，用于处理 JSON 数据
import json
# 导入 os 模块，用于执行操作系统相关的功能
import os

# 以只读方式打开 listing.json 文件，并将其内容加载为 JSON 数据
with open('listing.json', 'r') as fh:
    data = json.loads(fh.read())

# 从 JSON 数据中获取指定键的数值
entry = data["available"]["3"][-1]

# 获取当前主机的主机名
hostname = os.popen('hostname').read().strip()

# 以只写方式打开 www/listing.json 文件，并将数据以 JSON 格式写入文件
with open('www/listing.json', 'w') as fh:
    json.dump(
        {
            "available": {
                entry["version"]: [
                    {
                        "built": entry["built"],
                        "version": entry["version"],
                        "url": f"https://{hostname}.local/db.tar.gz",
                        "checksum": entry["checksum"]
                    }
                ]
            }
        }, fh)

# 打印 entry 字典中的 "url" 键对应的值
print(entry["url"])
```