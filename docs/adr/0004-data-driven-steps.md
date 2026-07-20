# ADR-0004: Steps 数据驱动化与最终解耦

## 决定

将步骤函数中的剩余硬编码分支替换为数据驱动模式，包括 step3 条件判断简化、step4 安装映射抽取、step5 扁平化、step 编号常量化。

## 理由

前序重构（ADR-0002、ADR-0003）已消除 Plan/Summary 的 case 重复和配置硬编码。但步骤函数内部仍有少量内联条件：

- step3_templates 的 `case "$cond" in opencode/claude` 可用统一表达式 `[[ ${plan_ref[tool]} == "$cond" || ${plan_ref[tool]} == "both" ]]` 替代
- step4_skills 的 `case "${plan_ref[skills]}" in mpskills/trellis` 可抽取为 `INSTALL_MAP`
- step5_aliases 的三层嵌套 if 可用 early return 扁平化
- `[Step X/6]` 字符串分散在 6 个函数中，修改步数需改 6 处

## 具体改动

### 1. step3 条件简化

```
BEFORE: case "$cond" in opencode) [[ ... ]] ;; claude) [[ ... ]] ;; esac
AFTER:  [[ ${plan_ref[tool]} == "$cond" || ${plan_ref[tool]} == "both" ]]
```

`cond` 直接作为数据匹配目标，`both` 作为通用通配。

### 2. INSTALL_MAP

```bash
INSTALL_MAP=(
  "mpskills|.agents/skills|Matt Pocock Skills|npx skills@latest add mattpocock/skills"
  "trellis||Trellis|npx @mindfoldhq/trellis init"
)
```

格式：`key|check_dir|label|command`。step4_skills 遍历 INSTALL_MAP，依据 `PLAN[skills]` 匹配执行。

### 3. step5 early return 扁平化

三层嵌套 if → 逆条件提前 return：

```
if not (opencode or both) or not mpskills → skip return
if not opencode.json exists → skip return
run aliases
```

### 4. step_echo / STEP_LABELS

```bash
STEP_LABELS=(
  "初始化 Git 仓库"
  "写入 .gitignore"
  ...
)

step_echo() {
  local num=$1
  echo "[Step $num/${#STEP_LABELS[@]}] ${STEP_LABELS[$((num-1))]}"
}
```

6 个步骤函数统一调用 `step_echo N`。增减步骤只需更新 STEP_LABELS。

### 5. 其他

- step1_git_init 增加 `cmd_available git` 检查
- `npm run lint` 增加 Python 语法检查
- package.json 声明 bats 为 devDependency

## 状态

已接受。

## Consequences

- 新增工具（如 VSCode）：需在 config.sh 添加 TOOL_CHOICES 条目 + INSTALL_MAP 条件条目 + TEMPLATE_MAP 条目
- 新增技能组：需在 config.sh 添加 SKILL_CHOICES 条目 + INSTALL_MAP 条目
- 新增或删除步骤：更新 config.sh 的 STEP_LABELS（自动适配编号）
- 步骤函数间不再存在共享条件逻辑，每步只关心自身执行
