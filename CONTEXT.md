# 领域术语表

## init-project.sh

项目入口脚本（~35 行），职责仅为 source lib/ 模块 + 编排计划 → 执行。不包含业务逻辑。

_避免：_ 安装脚本、bootstrap

## lib/

模块化库目录，包含 4 个职责独立的 shell 模块：

- **config.sh** — 配置数据（TEMPLATE_MAP、INSTALL_MAP、STEP_LABELS、选项定义、CONFIG_DELIM）
- **utils.sh** — 辅助函数（ensure_file、cmd_available、try_install、confirm_and_run、prompt_choice、yes_no）
- **plan.sh** — 决策模块（_check_git_status、_collect_tool/_skill、collect_plan、tool_label、skills_label、print_plan_summary）
- **steps.sh** — 执行模块（step1_git_init ~ step5_codegraph，从 config.sh 读取数据驱动）

## Step

初始化流程中的逻辑步骤，共 5 步编号 1-5，定义在 `$STEP_LABELS` 数组中。步骤函数通过字符串描述集中管理，无需改动函数名即可调整步骤顺序或数量。

_步骤：_ ① git init → ② 写入配置文件 → ③ 安装技能组 → ④ 注入别名 → ⑤ CodeGraph 索引

_避免：_ phase

## Plan

Decision phase 的输出，Execution phase 的输入。`declare -A PLAN` 关联数组，携带用户选择：`PLAN[tool]`（opencode/claude/both）和 `PLAN[skills]`（mpskills/trellis）。所有读取 PLAN 的函数均显式接收 `$1` 参数，不依赖全局变量。

_避免：_ 配置、选择结果、参数

## Template

`templates/` 目录中的源文件，初始化时通过 TEMPLATE_MAP 按条件复制到目标项目。`always` 无条件复制（如 .gitignore），`opencode`/`claude` 按工具选择复制。

_避免：_ 模板文件、配置模板

## CONFIG_DELIM

配置数据的分隔符（`|`），在 `config.sh` 中定义，`utils.sh` 解析时引用。修改分隔符只需改一处。

## PROJECT_ROOT

由 `config.sh` 通过 `BASH_SOURCE[0]` 自计算的模块根目录，替代原入口脚本的 `$SCRIPT_DIR` 变量。lib 模块不再依赖入口脚本设置路径变量。
