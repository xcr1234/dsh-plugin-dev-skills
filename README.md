# dsh-plugin-dev

开发 DeepSeek Harness (DSH) 插件的 Agent Skill（遵循 [Agent Skills 规范](https://agentskills.io)，SKILL.md + references/ 渐进式披露）。

An Agent Skill for developing DeepSeek Harness (DSH) plugins, following the Agent Skills specification (SKILL.md + progressive disclosure via references/).

## 定位 / What it is

把 DSH 官方文档中分散在教程、参考手册与生成目录里的插件开发约定，收敛为可执行的标准：8 条硬规则、6 个场景工作流、决策速查表与完成前检查清单；12 个 references 按需加载详细标准（插件形态与生命周期、服务与依赖、五种事件分发模式、插件配置、上下文与 Fiber API、三种角色设计、工具开发、LLM 适配器协议、插件形态扩展、打包安装、workspace 包、seam 全表）。

A distilled, executable standard for DSH plugin development: hard rules, scenario workflows, decision tables, checklists, and 12 on-demand references covering the full plugin-development surface.

## 目录结构 / Layout

```
dsh-plugin-dev/
├── SKILL.md                       # 入口：frontmatter + 硬规则 + 工作流 + 决策表 + 检查清单
├── references/                    # 12 个按需加载的详细标准
├── examples/                      # 可复制、可运行的最小示例
│   ├── hello-plugin/              # 最小插件（bundle 格式，可直接安装）
│   └── greet-tool/                # 最小模型工具（defineTool + output.render）
└── evals/                         # description 触发评测集与方法
```

## 安装 / Installation

把整个 `dsh-plugin-dev` 文件夹复制到对应 agent 的技能目录即可，无需任何配置（无构建步骤、无脚本依赖）：

Copy the whole `dsh-plugin-dev` folder into the skill directory of your agent — no configuration, no build step, no script dependencies:

| Agent | 项目级 / Project-level | 用户级 / User-level |
| --- | --- | --- |
| DeepSeek Harness | `<project>/.dsh/skills/`（rank 100）或 `<project>/.agents/skills/`（rank 200） | `~/.dsh/skills/`（rank 400） |
| Claude Code | `<project>/.claude/skills/` | `~/.claude/skills/` |
| Codex | `<project>/.codex/skills/` | `~/.codex/skills/` |
| VS Code Copilot | `<project>/.agents/skills/` | `~/.agents/skills/` |
| 其它兼容 agent | 按该 agent 的 Agent Skills 发现目录约定 | 同上 |

安装验证：向 agent 提问"开发一个 DSH 插件 / 写一个 DSH 工具 / 接一个模型提供方"应触发本技能；在 DSH 中可用 `skill(dsh-plugin-dev)` 工具直接加载验证。

Verify by asking the agent to "开发一个 DSH 插件 / create a DSH tool" — the skill should trigger and load.

## 版本对应 / Version tracking

技能内容蒸馏自 **DeepSeek Harness 官方文档站**（deepseek-harness.github.io，2026-08 文档快照），并遵循官方「接口以生成参考为准」的原则：当技能内容与仓库自动生成的子系统页面/接口不一致时，**以官方生成参考为准**，并向本仓库提交修正。

The content was distilled from the official DeepSeek Harness docs (2026-08 snapshot). When the skill and the repository's generated references disagree, the generated references win — please file a fix here.

## 触发评测 / Trigger evals

`evals/trigger-queries.json` 是 description 的回归评测集（should-trigger / should-not-trigger 查询）。修改 description 前请先跑评测并记录通过率；评测方法见 `evals/README.md`。

## 示例 / Examples

- `examples/hello-plugin` —— 最小插件：`dsh plugin --profile demo add ./examples/hello-plugin` 后启动即可看到加载日志。
- `examples/greet-tool` —— 最小模型工具：安装后对 agent 说 "Use the greet tool to greet Ada."。

## 范围边界 / Scope

本技能覆盖**仓库内、文件式**的 DSH 插件开发。不覆盖：会话内动态插件（`cordis_define`/`cordis_run` 流）与 agent preset 组合编辑——这两类由各部署的专项技能/官方工具覆盖。

Covers file-based DSH plugin development. Out of scope: in-session dynamic plugins (`cordis_define`/`cordis_run`) and agent-preset composition editing.

## 维护与贡献 / Maintenance & contributing

- 更新任何 references 前，先核对官方文档对应页面（文档站或源码生成区块），并在 PR 中注明来源页面。
- 保持 Agent Skills 规范：name 为 kebab-case 且与目录一致；description ≤ 1024 字符（DSH 目录注入提醒默认 500 字符）；正文渐进式披露。
- 欢迎 PR：修正、补充示例、扩充评测集、提供英文/其它语言版本。

## License

MIT — 见 [LICENSE](LICENSE)。
