# project-init

个人项目初始化脚本，支持选择 OpenCode 或 Claude，并可安装 Matt Pocock Skills 或 Trellis 技能组。

## 使用

在新项目目录运行：

~~~sh
init-project.sh
~~~

脚本会检查项目是否已初始化，再按选择创建 Git 仓库和配置文件、安装技能组，并可选建立 CodeGraph 索引。步骤可重复运行，已存在的内容会跳过。

也可以把脚本加入 `PATH`：

~~~sh
cp init-project.sh ~/bin/init-project.sh
~~~

确保 `~/bin` 位于 `PATH` 中。

## 自定义模板

模板位于 `templates/`。编辑相应文件即可调整新项目配置；`CLAUDE.md` 从 `AGENTS.md` 生成，共用同一份源文件。

主要实现位于 `lib/`：`config.sh` 定义工具、技能组和模板映射，`steps.sh` 实现初始化步骤。测试位于 `tests/`。