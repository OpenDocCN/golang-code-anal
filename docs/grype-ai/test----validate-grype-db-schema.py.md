# `grype\test\validate-grype-db-schema.py`

```
#!/usr/bin/env python
# 设置脚本的解释器为 Python

import re
# 导入正则表达式模块
import os
# 导入操作系统模块
import sys
# 导入系统模块
import collections
# 导入 collections 模块

dir_pattern = r'grype/db/v(?P<version>\d+)'
# 定义目录模式的正则表达式模式
db_dir_regex = re.compile(dir_pattern)
# 编译目录模式的正则表达式

import_regex = re.compile(rf'github.com/anchore/grype/{dir_pattern}')
# 编译导入模式的正则表达式

def report_schema_versions_found(title, schema_to_locations):
    # 定义函数，用于报告找到的模式版本
    for schema, locations in sorted(schema_to_locations.items()):
        # 遍历排序后的模式到位置的字典
        print(f"{title} schema: {schema}")
        # 打印标题和模式
        for location in locations:
            # 遍历位置
            print(f"  {location}")
            # 打印位置
    print()
    # 打印空行

def assert_single_schema_version(schema_to_locations):
    # 定义函数，用于断言只有单个模式版本
    schema_versions_found = list(schema_to_locations.keys())
    # 获取找到的模式版本列表
    try:
        for x in schema_versions_found:
            int(x)
    except ValueError:
        sys.exit("Non-numeric schema found: %s" % ", ".join(schema_versions_found))
    # 检查是否有非数字的模式版本，如果有则退出并打印错误信息

    if len(schema_to_locations) > 1:
        sys.exit("Found multiple schemas: %s" % ", ".join(schema_versions_found))
    # 如果找到多个模式版本，则退出并打印错误信息
    elif len(schema_to_locations) == 0:
        sys.exit("No schemas found!")
    # 如果没有找到模式版本，则退出并打印错误信息

def find_db_schema_usages(filter_out_regexes=None, keep_regexes=None):
    # 定义函数，用于查找数据库模式的使用情况
    schema_to_locations = collections.defaultdict(list)
    # 创建一个默认值为列表的字典用于存储模式到位置的映射
    # 遍历当前目录及其子目录下的所有文件和文件夹
    for root, dirs, files in os.walk("."):
        # 遍历当前目录下的所有文件
        for file in files:
            # 如果文件不是以".go"结尾，则跳过当前循环，继续下一个文件
            if not file.endswith(".go"):
                continue
            # 获取文件的完整路径
            location = os.path.join(root, file)

            # 如果存在需要过滤的正则表达式
            if filter_out_regexes:
                # 初始化是否需要过滤的标志为False
                do_filter = False
                # 遍历所有需要过滤的正则表达式
                for regex in filter_out_regexes:
                    # 如果正则表达式匹配到文件路径，则设置需要过滤的标志为True，并跳出循环
                    if regex.findall(location):
                        do_filter = True
                        break
                # 如果需要过滤，则跳过当前文件，继续下一个文件
                if do_filter:
                    continue

            # 如果存在需要保留的正则表达式
            if keep_regexes:
                # 初始化是否需要保留的标志为False
                do_keep = False
                # 遍历所有需要保留的正则表达式
                for regex in keep_regexes:
                    # 如果正则表达式匹配到文件路径，则设置需要保留的标志为True，并跳出循环
                    if regex.findall(location):
                        do_keep = True
                        break
                # 如果不需要保留，则跳过当前文件，继续下一个文件
                if not do_keep:
                    continue

            # 记录所有导入的模块（从这一点开始，这是db/v#代码的唯一可能的使用者）
            with open(location) as f:
                # 读取文件内容，并使用正则表达式查找所有的导入模块
                for match in import_regex.findall(f.read(), re.MULTILINE):
                    # 将匹配到的导入模块添加到schema_to_locations字典中对应的列表中
                    schema_to_locations[match].append(location)

    # 返回包含导入模块和对应文件路径的字典
    return schema_to_locations
# 检查给定的模式是否在指定的位置列表中存在对应的版本前缀
def assert_schema_version_prefix(schema, locations):
    # 遍历位置列表
    for location in locations:
        # 如果位置列表中不包含指定模式的版本前缀，则退出程序并打印错误信息
        if f"/grype/db/v{schema}" not in location:
            sys.exit(f"found cross-schema reference: {location}")


# 验证模式的消费者
def validate_schema_consumers():
    # 查找数据库模式的使用情况，并过滤掉指定的正则表达式
    schema_to_locations = find_db_schema_usages(filter_out_regexes=[db_dir_regex])
    # 报告找到的模式版本的消费者
    report_schema_versions_found("Consumers of", schema_to_locations)
    # 确保每个消费者只使用单一的模式版本
    assert_single_schema_version(schema_to_locations)
    # 打印找到的消费者模式版本
    print("Consuming schema versions found: %s" % list(schema_to_locations.keys())[0])


# 验证模式的定义
def validate_schema_definitions():
    # 查找数据库模式的使用情况，并保留指定的正则表达式
    schema_to_locations = find_db_schema_usages(keep_regexes=[db_dir_regex])
    # 报告找到的模式版本的定义
    report_schema_versions_found("Definitions of", schema_to_locations)
    # 确保每个定义都不会跨越其他模式的定义
    for schema, locations in schema_to_locations.items():
        assert_schema_version_prefix(schema, locations)
    # 打印验证结果，确保模式定义不会相互引用
    print("Verified that schema definitions don't cross-import")


# 主函数
def main():
    # 验证模式的定义
    validate_schema_definitions()
    # 打印空行
    print()
    # 验证模式的消费者
    validate_schema_consumers()


# 如果当前脚本被直接执行，则调用主函数
if __name__ == "__main__":
    main()
```