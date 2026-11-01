# 参考：Agent Skills 规范速查

> 用于回答格式、字段、加载机制、运行载体相关的具体问题。所有事实来自官方一手来源，链接见文末。
> 截至 2026-10-20，Agent Skills 已是跨平台开放标准（2025-12-18 起由 agentskills.io 托管规范）。

## 目录

- [1. 最小单元与目录结构](#1-最小单元与目录结构)
- [2. frontmatter 字段与约束](#2-frontmatter-字段与约束)
- [3. 渐进披露三级](#3-渐进披露三级)
- [4. 三种运行载体](#4-三种运行载体)
- [5. 已知限制](#5-已知限制)
- [6. 本机 Trae 的约定](#6-本机-trae-的约定)

## 1. 最小单元与目录结构

一个 Skill = 一个目录 + 必需的 `SKILL.md`（YAML frontmatter + Markdown 正文）。

```text
<skill-folder>/
├── SKILL.md           # 必需
├── scripts/           # 可选：确定性或需反复重写的可执行代码
├── references/        # 可选：长文档，按需加载
└── assets/            # 可选：模板、图片、样板文件（应复制而非读入上下文）
```

Skill **不是工具、不是 API**，是一份说明书：不新增能力，只新增"做某类事的标准流程"。

## 2. frontmatter 字段与约束

| 字段 | 必填 | 约束 |
|---|---|---|
| `name` | 是 | 1–64 字符；仅小写字母、数字、连字符；不得以连字符开头/结尾，不得出现连续 `--`；**必须与父目录同名**；**不得包含 `anthropic` 或 `claude`** |
| `description` | 是 | 1–1024 字符；必须同时写清"做什么 + 何时用"；**禁止 XML 标签** |
| `license` | 否 | 许可证标识 |
| `compatibility` | 否 | 兼容性说明，≤500 字符 |
| `metadata` | 否 | 字符串键值映射（author、version 等） |
| `allowed-tools` | 否 | 空格分隔的预授权工具清单（官方标注 **Experimental**） |

Claude Code 另有专有字段：`argument-hint`、`disable-model-invocation`、`user-invocable`、`model`、`effort`、`context: fork`、`agent`、`hooks`。

**`description` 是唯一参与路由匹配的文本**——所以它必须自包含：做什么、何时用、不用于什么。触发关键词放在这里，写在正文里无效。

## 3. 渐进披露三级

| 级别 | 何时加载 | 加载什么 | 官方给的成本量级 |
|---|---|---|---|
| L1 元数据 | 启动时全部加载 | 每个技能的 `name` + `description` | 约 100 tokens / 个 |
| L2 指令正文 | 技能被触发时 | `SKILL.md` 正文 | 建议 < 5k tokens、< 500 行 |
| L3+ 资源与代码 | 按需 | `references/`、`assets/`、`scripts/` | 脚本**代码本身不进上下文**，仅执行输出计入 |

设计含义：L1 便宜 → 技能可以有很多个；L2 只在被选中时付费 → 单个技能可以写厚；L3 让重资产常驻磁盘而不常驻上下文。

## 4. 三种运行载体

| 载体 | 安装位置/方式 | 触发方式 | 备注 |
|---|---|---|---|
| Claude 应用 | 设置中启用，或上传 zip | 预置技能自动生效 | 自定义技能为个人级，不跨组织共享 |
| Claude Code | `~/.claude/skills`、`.claude/skills`、企业托管、插件 | `/技能名` 直调，或自动触发 | 遵循开放标准并做扩展（见第 2 节专有字段） |
| API | `container` 参数指定 `skill_id` + 代码执行工具；自定义走 `/v1/skills` | 由模型按 `description` 自行选择 | 需 beta header：`code-execution-2025-08-25`、`skills-2025-10-02`、`files-api-2025-04-14`；**单请求最多 8 个技能** |

## 5. 已知限制

- **跨 surface 不同步**：同一技能在 Claude 应用 / Code / API 之间不会自动同步。
- **共享范围不一致**：应用侧个人级、API 侧工作区级、Code 侧文件系统级。
- **运行时环境不同**：API 侧无网络、不可安装新包；Code 侧有完整网络。技能正文若含 `pip install`、下载外链等步骤，在 API 侧会直接失败。
- **单请求技能数上限 8**：必须靠路由分批，不能一次全绑。

## 6. 本机 Trae 的约定

| 类型 | 路径 |
|---|---|
| 项目级技能 | `<工作区>/.trae/skills/<name>/SKILL.md`（目录名固定 `.trae`，**不是** `.trae-cn`） |
| 用户级技能 | `C:\Users\Lenovo\.trae-cn\skills\<name>\SKILL.md` |

- 项目级与用户级目录结构相同，默认**只写 `name` 与 `description`** 两个 frontmatter 字段。
- 参考资料目录布局，可直接看已装技能 `C:\Users\Lenovo\.trae-cn\skills\ppt-page\`（含 `references/`、`scripts/`、`assets/` 三层）。

## 官方来源

- 规范页 — https://agentskills.io/specification
- 概览（字段、三级披露、三端用法）— https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/overview
- 最佳实践 — https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/best-practices
- Claude Code 技能 — https://docs.anthropic.com/en/docs/claude-code/skills
- 发布博文（2025-10-16）— https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
