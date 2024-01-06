# `grype\test\quality\gate.py`

```
#!/usr/bin/env python3
# 指定使用 Python3 解释器

import logging
# 导入日志模块
import os
# 导入操作系统模块
import re
# 导入正则表达式模块
import subprocess
# 导入子进程管理模块
import sys
# 导入系统模块
from typing import Optional
# 导入类型提示模块

import click
# 导入命令行解析模块
from tabulate import tabulate
# 导入表格生成模块
from dataclasses import dataclass, InitVar, field
# 导入数据类模块

import yardstick
# 导入 yardstick 模块
from yardstick import store, comparison, artifact, arrange
# 导入 yardstick 模块的子模块
from yardstick.cli import display, config
# 导入 yardstick 命令行界面模块

# 设置默认的结果集名称
default_result_set = "pr_vs_latest_via_sbom"
# 设置 grype 数据库在失败时不抛出异常
yardstick.utils.grype_db.raise_on_failure(False)
# 使用 dataclass 装饰器定义 Gate 类，用于存储门的信息
@dataclass
class Gate:
    # 初始化标签比较和标签比较统计信息
    label_comparisons: InitVar[Optional[list[comparison.AgainstLabels]]]
    label_comparison_stats: InitVar[Optional[comparison.ImageToolLabelStats]]

    # 存储失败原因的列表
    reasons: list[str] = field(default_factory=list)

    # 初始化方法，处理标签比较和标签比较统计信息
    def __post_init__(self, label_comparisons: Optional[list[comparison.AgainstLabels]], label_comparison_stats: Optional[comparison.ImageToolLabelStats]):
        # 如果没有标签比较和标签比较统计信息，则直接返回
        if not label_comparisons and not label_comparison_stats:
            return 
    
        # 初始化失败原因列表
        reasons = []

        # - 当当前 F1 分数低于上一个版本的 F1 分数时失败（或 F1 分数不确定时失败）
        # - 当不确定百分比 > 10% 时失败
        # - 当 FN 增加时失败
        # 猜测最新版本和当前版本的工具方向
        latest_release_tool, current_tool = guess_tool_orientation(label_comparison_stats.tools)

        # 根据图像获取最新版本的标签比较信息
        latest_release_comparisons_by_image = {comp.config.image: comp for comp in label_comparisons if comp.config.tool == latest_release_tool }
# 创建一个字典，键为comp.config.image，值为comp，其中comp来自label_comparisons，且comp.config.tool等于current_tool
current_comparisons_by_image = {comp.config.image: comp for comp in label_comparisons if comp.config.tool == current_tool }

# 遍历current_comparisons_by_image字典的键值对
for image, comp in current_comparisons_by_image.items():
    # 获取latest_release_comparisons_by_image[image].summary.f1_score的值
    latest_f1_score = latest_release_comparisons_by_image[image].summary.f1_score
    # 获取comp.summary.f1_score的值
    current_f1_score = comp.summary.f1_score
    # 如果current_f1_score小于latest_f1_score
    if current_f1_score < latest_f1_score:

    # 如果comp.summary.indeterminate_percent大于10
    if comp.summary.indeterminate_percent > 10:
        # 将一条原因添加到reasons列表中
        reasons.append(f"current indeterminate matches % is greater than 10%: {bcolors.BOLD+bcolors.UNDERLINE}current={comp.summary.indeterminate_percent:0.2f}%{bcolors.RESET} image={image}")

    # 获取latest_release_comparisons_by_image[image].summary.false_negatives的值
    latest_fns = latest_release_comparisons_by_image[image].summary.false_negatives
    # 获取comp.summary.false_negatives的值
    current_fns = comp.summary.false_negatives
    # 如果current_fns大于latest_fns
    if current_fns > latest_fns:

# 将reasons列表赋值给self.reasons
self.reasons = reasons

# 定义一个方法，判断reasons列表的长度是否为0，是则返回True，否则返回False
def passed(self):
    return len(self.reasons) == 0

# 定义一个函数，参数为一个字符串列表
def guess_tool_orientation(tools: list[str]):
    """
    给定一对工具，猜测哪个是最新版本，哪个是与最新版本进行比较的版本。
    返回（最新工具，当前工具）
    """
    # 如果工具数量不等于2，则抛出运行时错误
    if len(tools) != 2:
        raise RuntimeError("expected 2 tools, got %s" % tools)
    # 对工具进行排序
    tool_a, tool_b = sorted(tools)
    # 如果两个工具相同，则抛出值错误
    if tool_a == tool_b:
        raise ValueError("latest release tool and current tool are the same")
    # 如果工具a以"latest"结尾，则返回工具a和工具b
    if tool_a.endswith("latest"):
        return tool_a, tool_b
    # 如果工具b以"latest"结尾，则返回工具b和工具a
    elif tool_b.endswith("latest"):
        return tool_b, tool_a

    # 如果工具a包含"@path:"而工具b不包含，则返回工具b和工具a
    if "@path:" in tool_a and "@path:" not in tool_b:
        # tool_a is a local build, so compare it against tool_b
        return tool_b, tool_a

    # 如果工具b包含"@path:"而工具a不包含，则返回工具a和工具b
    if "@path:" in tool_b and "@path:" not in tool_a:
# 定义一个类 bcolors，包含了一些 ANSI 转义码，用于在终端中输出彩色文本
class bcolors:
    HEADER = '\033[95m'  # 紫色
    OKBLUE = '\033[94m'   # 蓝色
    OKCYAN = '\033[96m'   # 青色
    OKGREEN = '\033[92m'  # 绿色
    WARNING = '\033[93m'  # 黄色
    FAIL = '\033[91m'     # 红色
    BOLD = '\033[1m'      # 粗体
    UNDERLINE = '\033[4m' # 下划线
    RESET = '\033[0m'     # 重置颜色

# 如果标准输出不是终端，则将 HEADER 设置为空字符串，即不使用紫色
if not sys.stdout.isatty():
    bcolors.HEADER = ""
# 定义颜色常量，用于控制台输出的颜色
bcolors.OKBLUE = ""
bcolors.OKCYAN = ""
bcolors.OKGREEN = ""
bcolors.WARNING = ""
bcolors.FAIL = ""
bcolors.BOLD = ""
bcolors.UNDERLINE = ""
bcolors.RESET = ""

# 展示使用的结果
def show_results_used(results: list[artifact.ScanResult]):
    print(f"   Results used:")
    # 遍历结果列表，输出结果的ID、工具和镜像信息
    for idx, result in enumerate(results):
        branch = "├──"
        if idx == len(results) - 1:
            branch = "└──"
        print(f"    {branch} {result.ID} : {result.config.tool} against {result.config.image}")
    print()

# 验证函数，用于验证配置和结果
def validate(cfg: config.Application, result_set: str, images: list[str], always_run_label_comparison: bool, verbosity: int, label_entries: Optional[list[artifact.LabelEntry]] = None):
    # 打印验证信息
    print(f"{bcolors.HEADER}{bcolors.BOLD}Validating with {result_set!r}", bcolors.RESET)
# 从存储中加载指定名称的结果集对象
result_set_obj = store.result_set.load(name=result_set)

# 初始化一个空列表用于存储结果
ret = []

# 遍历结果集对象中的每个图像及其对应的结果状态
for image, result_states in result_set_obj.result_state_by_image.items():
    # 如果指定了图像列表并且当前图像不在列表中，则跳过当前图像
    if images and image not in images:
        print("Skipping image:", image)
        continue
    print()
    print("Testing image:", image)
    # 遍历当前图像的每个结果状态
    for state in result_states:
        print("   ", f"with {state.request.tool}")
    print()

    # 对图像进行质量验证，并将结果添加到ret列表中
    gate = validate_image(cfg, [s.config.path for s in result_states], always_run_label_comparison=always_run_label_comparison, verbosity=verbosity, label_entries=label_entries)
    ret.append(gate)

    # 检查是否未通过质量验证，如果是则打印失败信息
    failure = not gate.passed()
    if failure:
        print(f"{bcolors.FAIL}{bcolors.BOLD}Failed quality gate{bcolors.RESET}")
    # 打印失败原因
    for reason in gate.reasons:
# 打印每个原因
print(f"   - {reason}")

# 打印空行
print()

# 设置大小为120的size变量
size = 120
# 打印一条由"▁"组成的线
print("▁"*size)
# 打印一条由"░"组成的线
print("░"*size)
# 打印一条由"▔"组成的线
print("▔"*size)

# 返回结果
return ret

# 验证图片
def validate_image(cfg: config.Application, descriptions: list[str], always_run_label_comparison: bool, verbosity: int, label_entries: Optional[list[artifact.LabelEntry]] = None):
    # 进行相对比较
    # - 显示比较摘要（无门控操作）
    # - 列出所有单个匹配差异

    # 打印运行相对比较的消息
    print(f"{bcolors.HEADER}Running relative comparison...", bcolors.RESET)
    # 进行相对比较
    relative_comparison = yardstick.compare_results(descriptions=descriptions, year_max_limit=cfg.default_max_year)
    # 显示使用的结果
    show_results_used(relative_comparison.results)

    # 显示相对比较结果
    if verbosity > 0:
# 设置是否显示详细信息
details = verbosity > 1
# 显示保留的匹配项，根据verbosity决定是否显示详细信息，总结信息，以及是否显示共同项
display.preserved_matches(relative_comparison, details=details, summary=True, common=False)
# 打印空行
print()

# 如果没有找到差异，则退出
if not always_run_label_comparison and not sum([len(relative_comparison.unique[result.ID]) for result in relative_comparison.results]):
    print("no differences found between tool results")
    return Gate(None, None)

# 进行标签比较
print(f"{bcolors.HEADER}Running comparison against labels...", bcolors.RESET)
# 显示使用的结果
show_results_used(results)

# 如果verbosity大于0，则显示函数
show_fns = verbosity > 1
# 显示标签比较结果
display.label_comparison(
        results,
        comparisons_by_result_id,
        stats_by_image_tool_pair,
        show_fns=show_fns,
    # 设置参数 show_summaries 为 True
    show_summaries=True,
)

# 从结果中猜测最新版本和当前版本的工具方向
latest_release_tool, current_tool = guess_tool_orientation([r.config.tool for r in results])

# 显示相对比较的唯一差异，并与标签结论（TP/FP/FN/TN/Unknown）配对
all_rows: list[list[Any]] = []
for result in relative_comparison.results:
    # 获取相对比较结果的标签比较
    label_comparison = comparisons_by_result_id[result.ID]
    # 遍历相对比较结果的唯一匹配
    for unique_match in relative_comparison.unique[result.ID]:
        # 获取唯一匹配的标签
        labels = label_comparison.labels_by_match[unique_match.ID]
        # 如果没有标签，则标记为未知
        if not labels:
            label = "(unknown)"
        # 如果有多个不同的标签，则以逗号分隔显示
        elif len(set(labels)) > 1:
            label = ", ".join([l.name for l in labels])
        # 如果只有一个标签，则直接显示
        else:
            label = labels[0].name

        # 设置颜色为空
        color = ""
# 初始化注释为空字符串
commentary = ""
# 如果结果配置的工具与最新发布的工具相同
if result.config.tool == latest_release_tool:
    # 找到唯一结果的工具是最新发布的工具...
    if label == artifact.Label.TruePositive.name:
        # 糟糕！我们错过了一个案例（这是一个新的FN）
        color = bcolors.FAIL
        commentary = "(this is a new FN 😱)"
    elif artifact.Label.FalsePositive.name in label:
        # 我们摆脱了一个FP！["hip!", "hip!"]
        color = bcolors.OKBLUE
        commentary = "(got rid of a former FP 🙌)"
# 如果结果配置的工具与最新发布的工具不同
else:
    # 找到唯一结果的工具是当前工具...
    if label == artifact.Label.TruePositive.name:
        # 太棒了！我们找到了一个新的TP，之前的工具版本错过了！
        color = bcolors.OKBLUE
        commentary = "(this is a new TP 🙌)"
    elif artifact.Label.FalsePositive.name in label:
        # 唉，我们的更改导致了一个新的FP... 不太好，也许不太糟？
        color = bcolors.FAIL
# 添加一个新的评论到变量commentary中
commentary = "(this is a new FP 😱)"

# 将包含不同信息的列表添加到all_rows列表中
all_rows.append(
    [
        f"{color}{result.config.tool} ONLY{bcolors.RESET}",  # 添加工具名称到列表
        f"{color}{unique_match.package.name}@{unique_match.package.version}{bcolors.RESET}",  # 添加包名称和版本到列表
        f"{color}{unique_match.vulnerability.id}{bcolors.RESET}",  # 添加漏洞ID到列表
        f"{color}{label}{bcolors.RESET}",  # 添加标签到列表
        f"{commentary}",  # 添加评论到列表
    ]
)

# 定义一个函数，用于去除字符串中的ANSI转义码
def escape_ansi(line):
    ansi_escape = re.compile(r'(?:\x1B[@-_]|[\x80-\x9F])[0-?]*[ -/]*[@-~]')
    return ansi_escape.sub('', line)

# 对all_rows列表进行排序，但不考虑ANSI转义码
all_rows = sorted(all_rows, key=lambda x: escape_ansi(str(x[0]+x[1]+x[2]+x[3])))
# 如果all_rows列表为空，则打印消息
if len(all_rows) == 0:
    print("No differences found between tooling (with labels)")
    else:
        # 如果条件不满足，打印带有标签的工具之间的匹配差异
        print("Match differences between tooling (with labels):")
        # 设置缩进
        indent = "   "
        # 打印带有标签的工具之间的匹配差异的表格
        print(indent + tabulate([["TOOL PARTITION", "PACKAGE", "VULNERABILITY", "LABEL", "COMMENTARY"]]+all_rows, tablefmt="plain").replace("\n", "\n" + indent) + "\n")

    # 用可以评估通过/失败条件的数据填充质量门
    return Gate(label_comparisons=comparisons_by_result_id.values(), label_comparison_stats=stats_by_image_tool_pair)

@click.command()
@click.option("--image", "-i", "images", multiple=True, help="filter down to one or more images to validate with (don't use the full result set)")
@click.option("--label-comparison", "-l", "always_run_label_comparison", is_flag=True, help="run label comparison irregardless of relative comparison results")
@click.option("--breakdown-by-ecosystem", "-e", is_flag=True, help="show label comparison results broken down by ecosystem")
@click.option("--verbose", "-v", "verbosity", count=True, help="show details of all comparisons")
@click.option("--result-set", "-r", default=default_result_set, help="the result set to use for the quality gate")
def main(images: list[str], always_run_label_comparison: bool, breakdown_by_ecosystem: bool, verbosity: int, result_set: str):
    # 加载配置
    cfg = config.load()
    # 设置日志记录的详细程度
    setup_logging(verbosity)

    # 让我们不要加载比我们需要的标签更多的标签，基于我们正在验证的图像来设置这一点
# 如果 images 列表为空，则执行以下操作
if not images:
    # 初始化一个空集合
    images = set()
    # 从存储中加载指定名称的结果集对象
    result_set_obj = store.result_set.load(name=result_set)
    # 遍历结果集对象中的状态，将每个状态对应的配置图片添加到 images 集合中
    for state in result_set_obj.state:
        images.add(state.config.image)
    # 将 images 集合转换为列表，并按字母顺序排序
    images = sorted(list(images))

# 打印加载标签条目的提示信息
print("Loading label entries...", end=" ")
# 从存储中加载指定图片的标签条目，限制年份最大值为默认最大年份
label_entries = store.labels.load_for_image(images, year_max_limit=cfg.default_max_year)
# 打印加载完成的提示信息，显示加载的标签条目数量
print(f"done! {len(label_entries)} entries loaded")

# 初始化结果集列表，目前仅支持一个结果集，但可以添加更多
result_sets = [result_set]
# 初始化门列表
gates = []
# 遍历结果集列表
for result_set in result_sets:
    # 调用验证函数，将返回的门列表添加到 gates 列表中
    gates.extend(validate(cfg, result_set, images=images, always_run_label_comparison=always_run_label_comparison, verbosity=verbosity, label_entries=label_entries))
    # 打印空行
    print()
    
    # 如果需要按生态系统性能进行细分
    if breakdown_by_ecosystem:
        # 打印按生态系统性能进行标签比较的提示信息
        print(f"{bcolors.HEADER}Breaking down label comparison by ecosystem performance...", bcolors.RESET)
        # 调用比较函数，返回结果图片、标签条目和统计信息
        results_by_image, label_entries, stats = yardstick.compare_results_against_labels_by_ecosystem(result_set=result_set, year_max_limit=cfg.default_max_year, label_entries=label_entries)
# 显示按生态系统比较的标签
display.labels_by_ecosystem_comparison(
    results_by_image,  # 按图像结果
    stats,  # 统计数据
    show_images_used=False,  # 是否显示使用的图像
)
# 打印空行
print()

# 检查是否有任何一个质量门未通过
failure = not all([gate.passed() for gate in gates])
if failure:
    # 如果有未通过的质量门，打印未通过的原因
    print("Reasons for quality gate failure:")
for gate in gates:
    for reason in gate.reasons:
        # 遍历每个质量门的未通过原因，逐个打印
        print(f"   - {reason}")

if failure:
    # 如果有未通过的质量门，打印失败信息并退出程序
    print()
    print(f"{bcolors.FAIL}{bcolors.BOLD}Quality gate FAILED{bcolors.RESET}")
    sys.exit(1)
else:
    # 如果所有质量门都通过，打印通过信息
    print(f"{bcolors.OKGREEN}{bcolors.BOLD}Quality gate passed!{bcolors.RESET}")
# 设置日志记录的级别和格式
def setup_logging(verbosity: int):
    # 禁用特定的 pylint 错误
    # 导入 logging.config 模块
    import logging.config

    # 根据 verbosity 参数设置日志级别
    if verbosity in [0, 1, 2]:
        log_level = "WARN"
    elif verbosity == 3:
        log_level = "INFO"
    else:
        log_level = "DEBUG"

    # 配置日志记录的格式和级别
    logging.config.dictConfig(
        {
            "version": 1,
            "formatters": {
                "standard": {
                    # 设置日志记录的格式
                    "format": "%(asctime)s [%(levelname)s] %(message)s",
# 配置日志记录器的格式和级别
{
    "version": 1,  # 配置文件版本号
    "disable_existing_loggers": False,  # 是否禁用已存在的记录器

    # 配置日志记录器的格式
    "formatters": {
        "standard": {
            "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s",  # 日志记录格式
            "datefmt": "",  # 日期时间格式
        },
    },

    # 配置日志记录器的处理器
    "handlers": {
        "default": {
            "level": log_level,  # 日志记录级别
            "formatter": "standard",  # 使用的格式
            "class": "logging.StreamHandler",  # 日志处理器类
            "stream": "ext://sys.stderr",  # 输出流
        },
    },

    # 配置日志记录器
    "loggers": {
        "": {  # root logger
            "handlers": ["default"],  # 使用的处理器
            "level": log_level,  # 日志记录级别
        },
    },
}
# 如果当前脚本被直接执行，则调用main函数。
```