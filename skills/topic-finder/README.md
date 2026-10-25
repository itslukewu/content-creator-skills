# topic-finder

**English** — For English-speaking users: use [`SKILL.md`](SKILL.md) and [`config.example.yaml`](config.example.yaml) in this folder (skill name `topic-finder`).

**中文** — 中文用户请使用 [`zh-CN/SKILL.md`](zh-CN/SKILL.md) 和 [`zh-CN/config.example.yaml`](zh-CN/config.example.yaml)（skill 名为 `topic-finder-zh`）。

Both versions describe the same method. Install only one. / 两个版本内容一致，只需安装其中一个。

## What it does / 做什么

Finds content topics for creators: infers an initial audience from benchmark accounts' outperforming posts when no audience profile exists, searches per fixed pillar and openly across benchmark posts, market signals, and saved references, then screens, de-duplicates, and outputs a topic table with source links and a readiness status.

为内容创作者找选题：没有受众画像时，先从对标账号的高表现内容反推初始受众；再按固定栏目定向找题，并从对标内容、市场信号和个人收藏中开放找题；筛选去重后输出带来源链接和内容准备度的选题表。

## Install / 安装

- **English**: upload this `topic-finder/` folder as a skill (the `zh-CN/` subfolder can be left out), or copy it to `~/.claude/skills/topic-finder/` for Claude Code.
- **中文**：把 `zh-CN/` 文件夹作为 skill 上传，或复制到 Claude Code 的 `~/.claude/skills/topic-finder-zh/`。

## Config / 配置

Copy `config.example.yaml` to `config.yaml` and fill it in. Keep your real config (account lists, database URLs) out of public repos, and never put secrets in it.

把 `config.example.yaml` 复制为 `config.yaml` 后填写。真实配置（账号名单、数据库地址）不要放进公开仓库，也不要写入任何密钥。
