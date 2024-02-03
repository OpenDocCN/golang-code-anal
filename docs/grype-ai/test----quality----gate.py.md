# `grype\test\quality\gate.py`

```go
#!/usr/bin/env python3
# 设置脚本的解释器为 Python 3

import logging
import os
import re
import subprocess
import sys
from typing import Optional

import click
from tabulate import tabulate
from dataclasses import dataclass, InitVar, field

import yardstick
from yardstick import store, comparison, artifact, arrange
from yardstick.cli import display, config
# 导入所需的模块和库

# see the .yardstick.yaml configuration for details
# 设置默认的结果集名称
default_result_set = "pr_vs_latest_via_sbom"
# 设置 grype_db 在失败时不抛出异常
yardstick.utils.grype_db.raise_on_failure(False)

@dataclass
class Gate:
    label_comparisons: InitVar[Optional[list[comparison.AgainstLabels]]]
    label_comparison_stats: InitVar[Optional[comparison.ImageToolLabelStats]]

    reasons: list[str] = field(default_factory=list)

    def passed(self):
        return len(self.reasons) == 0
# 定义 Gate 类，用于存储比较结果和原因

def guess_tool_orientation(tools: list[str]):
    """
    Given a pair of tools, guess which is latest version, and which is the one
    being compared to the latest version.
    Returns (latest_tool, current_tool)
    """
    if len(tools) != 2:
        raise RuntimeError("expected 2 tools, got %s" % tools)
    tool_a, tool_b = sorted(tools)
    if tool_a == tool_b:
        raise ValueError("latest release tool and current tool are the same")
    if tool_a.endswith("latest"):
        return tool_a, tool_b
    elif tool_b.endswith("latest"):
        return tool_b, tool_a

    if "@path:" in tool_a and "@path:" not in tool_b:
        # tool_a is a local build, so compare it against tool_b
        return tool_b, tool_a

    if "@path:" in tool_b and "@path:" not in tool_a:
        # tool_b is a local build, so compare it against tool_a
        return tool_a, tool_b

    return tool_a, tool_b
# 定义函数 guess_tool_orientation，用于猜测工具的版本和比较

class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKCYAN = '\033[96m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'
    RESET = '\033[0m'
# 定义类 bcolors，用于设置终端输出的颜色

if not sys.stdout.isatty():
    bcolors.HEADER = ""
    bcolors.OKBLUE = ""
# 如果标准输出不是终端，则设置终端颜色为空
    # 定义颜色常量，用于控制台输出的颜色设置
    bcolors.OKCYAN = ""
    bcolors.OKGREEN = ""
    bcolors.WARNING = ""
    bcolors.FAIL = ""
    bcolors.BOLD = ""
    bcolors.UNDERLINE = ""
    bcolors.RESET = ""
# 定义一个函数，用于展示使用的结果
def show_results_used(results: list[artifact.ScanResult]):
    # 打印结果使用的提示信息
    print(f"   Results used:")
    # 遍历结果列表
    for idx, result in enumerate(results):
        # 设置分支符号
        branch = "├──"
        # 如果是最后一个结果，则使用不同的分支符号
        if idx == len(results) - 1:
            branch = "└──"
        # 打印每个结果的ID、工具和镜像信息
        print(f"    {branch} {result.ID} : {result.config.tool} against {result.config.image}")
    # 打印空行
    print()

# 定义一个验证函数，用于验证配置和结果集
def validate(cfg: config.Application, result_set: str, images: list[str], always_run_label_comparison: bool, verbosity: int, label_entries: Optional[list[artifact.LabelEntry]] = None):
    # 打印验证的提示信息
    print(f"{bcolors.HEADER}{bcolors.BOLD}Validating with {result_set!r}", bcolors.RESET)
    # 加载指定名称的结果集对象
    result_set_obj = store.result_set.load(name=result_set)

    # 初始化返回结果列表
    ret = []
    # 遍历结果集中的每个镜像和结果状态
    for image, result_states in result_set_obj.result_state_by_image.items():
        # 如果指定了镜像列表且当前镜像不在列表中，则跳过
        if images and image not in images:
            print("Skipping image:", image)
            continue
        # 打印测试镜像的提示信息
        print()
        print("Testing image:", image)
        # 遍历每个结果状态，并打印使用的工具
        for state in result_states:
            print("   ", f"with {state.request.tool}")
        # 打印空行
        print()

        # 对镜像进行验证，并将结果添加到返回列表中
        gate = validate_image(cfg, [s.config.path for s in result_states], always_run_label_comparison=always_run_label_comparison, verbosity=verbosity, label_entries=label_entries)
        ret.append(gate)

        # 检查是否验证失败，如果失败则打印失败信息和原因
        failure = not gate.passed()
        if failure:
            print(f"{bcolors.FAIL}{bcolors.BOLD}Failed quality gate{bcolors.RESET}")
        for reason in gate.reasons:
            print(f"   - {reason}")

        # 打印空行
        print()
        # 打印分隔线
        size = 120
        print("▁"*size)
        print("░"*size)
        print("▔"*size)
    # 返回结果列表
    return ret

# 定义一个验证镜像的函数
def validate_image(cfg: config.Application, descriptions: list[str], always_run_label_comparison: bool, verbosity: int, label_entries: Optional[list[artifact.LabelEntry]] = None):
    # 执行相对比较
    # - 显示比较摘要（不进行门控操作）
    # - 列出所有单个匹配差异

    # 打印运行相对比较的提示信息
    print(f"{bcolors.HEADER}Running relative comparison...", bcolors.RESET)
    # 使用 yardstick.compare_results 函数比较描述和默认最大年限，返回相对比较结果
    relative_comparison = yardstick.compare_results(descriptions=descriptions, year_max_limit=cfg.default_max_year)
    # 展示相对比较结果
    show_results_used(relative_comparison.results)
    
    # 如果verbosity大于0，则显示详细信息
    if verbosity > 0:
        details = verbosity > 1
        # 显示相对比较结果的保留匹配
        display.preserved_matches(relative_comparison, details=details, summary=True, common=False)
        print()
    
    # 如果没有找到任何差异，则返回空的 Gate 对象
    if not always_run_label_comparison and not sum([len(relative_comparison.unique[result.ID]) for result in relative_comparison.results]):
        print("no differences found between tool results")
        return Gate(None, None)
    
    # 进行标签比较
    print(f"{bcolors.HEADER}Running comparison against labels...", bcolors.RESET)
    # 使用 yardstick.compare_results_against_labels 函数比较描述和默认最大年限，以及标签条目
    results, label_entries, comparisons_by_result_id, stats_by_image_tool_pair = yardstick.compare_results_against_labels(descriptions=descriptions, year_max_limit=cfg.default_max_year, label_entries=label_entries)
    show_results_used(results)
    
    # 如果verbosity大于0，则显示详细信息
    if verbosity > 0:
        show_fns = verbosity > 1
        # 显示标签比较结果
        display.label_comparison(
                results,
                comparisons_by_result_id,
                stats_by_image_tool_pair,
                show_fns=show_fns,
                show_summaries=True,
            )
    
    # 猜测最新版本的工具和当前工具的方向
    latest_release_tool, current_tool = guess_tool_orientation([r.config.tool for r in results])
    
    # 显示相对比较唯一差异与标签结论（TP/FP/FN/TN/Unknown）
    all_rows: list[list[Any]] = []
    # 定义一个函数，用于去除 ANSI 转义码
    def escape_ansi(line):
        ansi_escape = re.compile(r'(?:\x1B[@-_]|[\x80-\x9F])[0-?]*[ -/]*[@-~]')
        return ansi_escape.sub('', line)
    
    # 按照去除 ANSI 转义码后的结果排序
    all_rows = sorted(all_rows, key=lambda x: escape_ansi(str(x[0]+x[1]+x[2]+x[3])))
    # 如果没有任何行，则打印提示信息
    if len(all_rows) == 0:
        print("No differences found between tooling (with labels)")
    # 如果条件不满足，则打印匹配工具之间的差异
    else:
        print("Match differences between tooling (with labels):")
        # 设置缩进
        indent = "   "
        # 打印表格，并替换换行符为带有缩进的换行符
        print(indent + tabulate([["TOOL PARTITION", "PACKAGE", "VULNERABILITY", "LABEL", "COMMENTARY"]]+all_rows, tablefmt="plain").replace("\n", "\n" + indent) + "\n")

    # 用数据填充质量门，以评估通过/失败条件
    return Gate(label_comparisons=comparisons_by_result_id.values(), label_comparison_stats=stats_by_image_tool_pair)
# 创建命令行接口
@click.command()
# 添加选项：过滤到一个或多个要验证的图像（不使用完整的结果集）
@click.option("--image", "-i", "images", multiple=True, help="filter down to one or more images to validate with (don't use the full result set)")
# 添加选项：无论相对比较结果如何，都运行标签比较
@click.option("--label-comparison", "-l", "always_run_label_comparison", is_flag=True, help="run label comparison irregardless of relative comparison results")
# 添加选项：按生态系统显示标签比较结果的细分
@click.option("--breakdown-by-ecosystem", "-e", is_flag=True, help="show label comparison results broken down by ecosystem")
# 添加选项：显示所有比较的详细信息
@click.option("--verbose", "-v", "verbosity", count=True, help="show details of all comparisons")
# 添加选项：用于质量门的结果集
@click.option("--result-set", "-r", default=default_result_set, help="the result set to use for the quality gate")
# 主函数，接收图像列表、是否总是运行标签比较、是否按生态系统细分、详细程度、结果集等参数
def main(images: list[str], always_run_label_comparison: bool, breakdown_by_ecosystem: bool, verbosity: int, result_set: str):
    # 加载配置
    cfg = config.load()
    # 设置日志记录的详细程度
    setup_logging(verbosity)

    # 如果没有指定图像，则根据结果集加载图像
    if not images:
        images = set()
        result_set_obj = store.result_set.load(name=result_set)
        for state in result_set_obj.state:
            images.add(state.config.image)
        images = sorted(list(images))
 
    # 打印信息：正在加载标签条目
    print("Loading label entries...", end=" ")
    # 加载图像的标签条目
    label_entries = store.labels.load_for_image(images, year_max_limit=cfg.default_max_year)
    # 打印信息：加载完成
    print(f"done! {len(label_entries)} entries loaded")

    # 结果集列表
    result_sets = [result_set] # today only one result set is supported, but more can be added
    # 质量门列表
    gates = []
    # 遍历结果集列表
    for result_set in result_sets:
        # 将验证结果添加到门列表中
        gates.extend(validate(cfg, result_set, images=images, always_run_label_comparison=always_run_label_comparison, verbosity=verbosity, label_entries=label_entries))
        # 打印空行
        print()
        
        # 如果按生态系统拆分
        if breakdown_by_ecosystem:
            # 打印按生态系统性能拆分的消息
            print(f"{bcolors.HEADER}Breaking down label comparison by ecosystem performance...", bcolors.RESET)
            # 对结果集按生态系统性能进行比较
            results_by_image, label_entries, stats = yardstick.compare_results_against_labels_by_ecosystem(result_set=result_set, year_max_limit=cfg.default_max_year, label_entries=label_entries)
            # 显示按生态系统比较的标签
            display.labels_by_ecosystem_comparison(
                results_by_image,
                stats,
                show_images_used=False,
            )
            # 打印空行
            print()

    # 检查是否有门未通过
    failure = not all([gate.passed() for gate in gates])
    # 如果有门未通过
    if failure:
        # 打印质量门失败的原因
        print("Reasons for quality gate failure:")
    # 遍历门列表
    for gate in gates:
        # 遍历每个门的失败原因
        for reason in gate.reasons:
            # 打印失败原因
            print(f"   - {reason}")

    # 如果有门未通过
    if failure:
        # 打印空行
        print()
        # 打印质量门失败的消息
        print(f"{bcolors.FAIL}{bcolors.BOLD}Quality gate FAILED{bcolors.RESET}")
        # 退出程序并返回错误状态码
        sys.exit(1)
    # 如果所有门都通过
    else:
        # 打印质量门通过的消息
        print(f"{bcolors.OKGREEN}{bcolors.BOLD}Quality gate passed!{bcolors.RESET}")
def setup_logging(verbosity: int):
    # 禁用 pylint 对重新定义外部名称和在顶层之外导入的警告
    # 导入 logging.config 模块
    import logging.config

    # 根据传入的 verbosity 参数设置日志级别
    if verbosity in [0, 1, 2]:
        log_level = "WARN"
    elif verbosity == 3:
        log_level = "INFO"
    else:
        log_level = "DEBUG"

    # 配置日志记录器的格式、处理器和日志级别
    logging.config.dictConfig(
        {
            "version": 1,
            "formatters": {
                "standard": {
                    # 设置日志记录的格式，包括时间、日志级别和消息
                    "format": "%(asctime)s [%(levelname)s] %(message)s",
                    "datefmt": "",
                },
            },
            "handlers": {
                "default": {
                    # 设置处理器的日志级别、格式和输出流
                    "level": log_level,
                    "formatter": "standard",
                    "class": "logging.StreamHandler",
                    "stream": "ext://sys.stderr",
                },
            },
            "loggers": {
                "": {  # 根记录器
                    # 设置根记录器的处理器和日志级别
                    "handlers": ["default"],
                    "level": log_level,
                },
            },
        }
    )

if __name__ == '__main__':
    main()
```