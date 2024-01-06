# `grype\test\validate-grype-db-schema.py`

```
#!/usr/bin/env python
# 指定脚本解释器为 Python

import re
# 导入正则表达式模块
import os
# 导入操作系统模块
import sys
# 导入系统模块
import collections
# 导入集合模块

dir_pattern = r'grype/db/v(?P<version>\d+)'
# 定义目录模式的正则表达式模式
db_dir_regex = re.compile(dir_pattern)
# 编译目录模式的正则表达式

import_regex = re.compile(rf'github.com/anchore/grype/{dir_pattern}')
# 编译导入模式的正则表达式

def report_schema_versions_found(title, schema_to_locations):
    # 定义函数，打印找到的模式版本及其位置
    for schema, locations in sorted(schema_to_locations.items()):
        print(f"{title} schema: {schema}")
        for location in locations:
            print(f"  {location}")
    print()
    # 打印空行

def assert_single_schema_version(schema_to_locations):
    # 定义函数，断言只有一个模式版本
# 获取所有schema版本的列表
schema_versions_found = list(schema_to_locations.keys())

# 尝试将所有版本转换为整数，如果有非数字的版本则抛出异常
try:
    for x in schema_versions_found:
        int(x)
except ValueError:
    sys.exit("Non-numeric schema found: %s" % ", ".join(schema_versions_found))

# 如果存在多个schema版本，则退出并打印错误信息
if len(schema_to_locations) > 1:
    sys.exit("Found multiple schemas: %s" % ", ".join(schema_versions_found))
# 如果没有找到任何schema版本，则退出并打印错误信息
elif len(schema_to_locations) == 0:
    sys.exit("No schemas found!")

# 查找数据库schema的使用情况
def find_db_schema_usages(filter_out_regexes=None, keep_regexes=None):
    # 创建一个默认值为列表的字典，用于存储schema和其位置
    schema_to_locations = collections.defaultdict(list)

    # 遍历当前目录及其子目录下的所有文件
    for root, dirs, files in os.walk("."):
        # 遍历每个文件
        for file in files:
            # 如果文件不是以".go"结尾，则跳过
            if not file.endswith(".go"):
                continue
# 将 root 和 file 拼接成完整的文件路径
location = os.path.join(root, file)

# 如果存在需要过滤的正则表达式列表
if filter_out_regexes:
    # 初始化是否需要过滤的标志为 False
    do_filter = False
    # 遍历过滤正则表达式列表
    for regex in filter_out_regexes:
        # 如果正则表达式匹配到了文件路径
        if regex.findall(location):
            # 将需要过滤的标志设为 True
            do_filter = True
            # 跳出循环
            break
    # 如果需要过滤，则跳过当前文件
    if do_filter:
        continue

# 如果存在需要保留的正则表达式列表
if keep_regexes:
    # 初始化是否需要保留的标志为 False
    do_keep = False
    # 遍历保留正则表达式列表
    for regex in keep_regexes:
        # 如果正则表达式匹配到了文件路径
        if regex.findall(location):
            # 将需要保留的标志设为 True
            do_keep = True
            # 跳出循环
            break
    # 如果不需要保留，则跳过当前文件
    if not do_keep:
        continue
# 跟踪所有导入的内容（从此处开始，这是 db/v# 代码的唯一可能的消费者）
with open(location) as f:
    # 从文件中查找所有匹配的导入内容，并将其添加到对应的位置列表中
    for match in import_regex.findall(f.read(), re.MULTILINE):
        schema_to_locations[match].append(location)

# 返回包含导入内容的位置字典
return schema_to_locations


# 断言特定模式版本的前缀
def assert_schema_version_prefix(schema, locations):
    for location in locations:
        # 如果位置中不包含特定模式版本的前缀，则退出并打印错误信息
        if f"/grype/db/v{schema}" not in location:
            sys.exit(f"found cross-schema reference: {location}")


# 验证模式的消费者
def validate_schema_consumers():
    # 查找数据库模式的使用情况，并过滤掉指定的正则表达式
    schema_to_locations = find_db_schema_usages(filter_out_regexes=[db_dir_regex])
    # 报告找到的模式版本
    report_schema_versions_found("Consumers of", schema_to_locations)
    # 断言只有单一的模式版本
    assert_single_schema_version(schema_to_locations)
    # 打印找到的消费模式版本
    print("Consuming schema versions found: %s" % list(schema_to_locations.keys())[0])
# 验证数据库模式定义的正确性
def validate_schema_definitions():
    # 找到数据库模式的使用情况，并将结果保存在字典中
    schema_to_locations = find_db_schema_usages(keep_regexes=[db_dir_regex])
    # 报告找到的模式版本
    report_schema_versions_found("Definitions of", schema_to_locations)
    # 确保每个定义都不会跨越其他模式的定义
    for schema, locations in schema_to_locations.items():
        assert_schema_version_prefix(schema, locations)
    # 打印验证结果
    print("Verified that schema definitions don't cross-import")

# 主函数
def main():
    # 验证数据库模式定义的正确性
    validate_schema_definitions()
    # 打印空行
    print()
    # 验证数据库模式的消费者
    validate_schema_consumers()

# 如果是主程序入口
if __name__ == "__main__":
    # 调用主函数
    main()
```