# `grype\grype\db\test-fixtures\tls\listing.py`

```
# 导入 urllib.request 模块，用于发送 HTTP 请求
import urllib.request
# 导入 json 模块，用于处理 JSON 数据
import json
# 导入 os 模块，用于与操作系统交互
import os

# 以只读方式打开 listing.json 文件
with open('listing.json', 'r') as fh:
    # 读取文件内容并将其解析为 JSON 格式
    data = json.loads(fh.read())

# 从 JSON 数据中获取特定字段的值
entry = data["available"]["3"][-1]

# 获取当前主机的主机名
hostname = os.popen('hostname').read().strip()

# 以写入方式打开 www/listing.json 文件
with open('www/listing.json', 'w') as fh:
    # 将数据以 JSON 格式写入文件
    json.dump(
        {
            "available": {
                entry["version"]: [
                    {
                        "built": entry["built"],
                        "version": entry["version"],
                        "url": f"https://{hostname}.local/db.tar.gz",
                    }
                ]
            }
        },
        fh
    )
# 打印 entry 字典中的 "url" 键对应的值
print(entry["url"])
```