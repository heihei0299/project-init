# 审查 init-project.sh 命令别名改动

## Goal

审查 `step5_aliases` 中移除 `templateFile` 注入、改用 `.opencode/commands/*.md` 文件定义的改动。

## Background

opencode 配置 schema 不支持 `templateFile` 键，命令应通过 `.opencode/commands/*.md` 文件定义（frontmatter + body）。
`init-project.sh` 此前先创建 .md 文件，又往 opencode.json 注入 `templateFile` 的 `command` 块，导致配置报错。

## Review Scope

- `init-project.sh` `step5_aliases()` 的 Python 脚本部分
- `templates/opencode.json` 是否已无 `command` 块
- 生成的 `.opencode/commands/*.md` 格式是否正确

## Acceptance Criteria

- [ ] Python 脚本正确生成 frontmatter + template 格式的 .md 文件
- [ ] 不再往 opencode.json 写入 `command` 块
- [ ] 无无效的 import（如不再需要的 `json`）
- [ ] shell 语法通过 `bash -n`
