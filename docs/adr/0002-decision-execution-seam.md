# ADR-0002: Decision/Execution seam with Plan module

`init-project.sh` 的 8 个 Step 通过 4 个全局布尔变量（`use_opencode`、`use_claude`、`use_mpskills`、`use_trellis`）通信。理解一条执行路径需要全文交叉引用所有 `if $use_x` 分支，缺乏 locality。

**决定：** 用 `declare -A PLAN` 替代 4 个布尔变量，Plan 只有两个 key（`tool`、`skills`），带语义值而非 true/false。在构造 Plan 和执行 Step 之间标记 `# ── Seam ──` 分界线。Steps 封装为函数，显式接收 Plan。

## Considered Options

- **保持 4 个布尔值**（现状）— 无 seam，无 locality，新增工具/技能组时组合爆炸
- **Plan + Step 函数**（已选）— 显式 seam，Plan 是可测试的纯数据，函数签名表达依赖
- **Plan + `run_step` 调度器** — 被拒绝；抽象层在当前规模下收益不够

## Consequences

- Decision phase 的输出（Plan）可以独立断言测试
- 新增 Step 只需写一个函数并在 Execution phase 调用，不涉及输入逻辑
- `case "${PLAN[tool]}"` 代替了 `$use_opencode && $use_claude` 交叉条件，但 tool/both/claude 三种值的 case 模式在摘要和完成两处重复（已知 smell，尚未修复）
- Step 编号从 1-8 压缩为 1-6（模板合并）
