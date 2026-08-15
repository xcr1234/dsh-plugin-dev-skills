# dsh-plugin-dev

<p align="center">
  <samp>
    <strong>中文</strong> ·
    <a href="./README.md">English</a>
  </samp>
</p>

一套遵循 [Agent Skills 规范](https://agentskills.io) 的技能，用于开发 **DeepSeek Harness（DSH）** 插件。

DSH 是一个插件化的 Agent Harness SDK：模型适配器、工具注册表、会话日志、甚至 agent loop 本身，全都是可以从配置里替换的 Cordis 插件。本技能把官方文档里散落在教程、参考手册与生成目录中的约定，收敛成一套可执行的标准——任何加载了它的 agent，都能用同一种方式开发 DSH 插件。

## 技能里有什么

```
dsh-plugin-dev/
├── SKILL.md      # 入口：frontmatter、8 条硬规则、6 个场景工作流、决策速查表、完成前检查清单
├── references/   # 12 份详细标准，按需加载（渐进式披露）
├── examples/     # 两个可复制、可运行的最小示例
│   ├── hello-plugin/
│   └── greet-tool/
└── evals/        # description 的触发评测集与评测方法
```

references 覆盖：插件形态与生命周期 · 服务与依赖注入 · 五种事件分发模式 · 插件配置 · 上下文/Fiber/注册表 API · 三种角色能力设计（Definition/Provider/Consumer）· 工具开发 · LLM 适配器协议 · 插件形态扩展（工具/钩子/UI/协议桥）· 打包与安装 · 仓库内 workspace 包 · 完整能力 seam 目录。

## 安装

把整个 `dsh-plugin-dev` 文件夹复制到你的 agent 技能目录即可：无需构建、无需脚本依赖、无需任何配置。

| Agent | 项目级 | 用户级 |
| --- | --- | --- |
| DeepSeek Harness | `<project>/.dsh/skills/`（rank 100）或 `<project>/.agents/skills/`（rank 200） | `~/.dsh/skills/`（rank 400） |
| Claude Code | `<project>/.claude/skills/` | `~/.claude/skills/` |
| Codex | `<project>/.codex/skills/` | `~/.codex/skills/` |
| VS Code Copilot | `<project>/.agents/skills/` | `~/.agents/skills/` |
| 其它兼容 agent | 按该 agent 的技能目录约定 | 同上 |

验证：向 agent 提问"开发一个 DSH 插件 / 写一个 DSH 工具"，技能应被触发；在 DSH 里也可以直接用 `skill(dsh-plugin-dev)` 工具加载确认。

## 版本对应

内容蒸馏自 DeepSeek Harness 官方文档站（deepseek-harness.github.io，2026-08 快照），并遵循官方「接口以生成参考为准」的原则：技能内容与仓库生成参考不一致时，**以生成参考为准**。发现偏差欢迎提 issue 或 PR。

## 触发评测

`evals/trigger-queries.json` 是 description 的回归评测集（12 条正例 + 9 条负例）。修改 description 前请先跑评测并记录通过率；方法论（含训练/验证集划分、防过拟合）见 `evals/README.md`。

## 示例

- `examples/hello-plugin` —— 最小插件（bundle 格式）：`dsh plugin --profile demo add ./examples/hello-plugin` 后 `dsh --profile demo` 启动，应看到加载日志和每 5 秒一次的心跳，卸载时自动清理。
- `examples/greet-tool` —— 最小模型工具：安装后对 agent 说 "Use the greet tool to greet Ada."，应收到 "Hello, Ada!"。

## 范围边界

覆盖**仓库内、文件式**的 DSH 插件开发：插件包、cordis.yml 行、patch overlay、工具、适配器、组合包、profile、仓库内 workspace 包。不覆盖会话内动态插件（`cordis_define`/`cordis_run` 流）与 agent preset 组合编辑——这两类由各部署的专项技能或官方工具负责。

## 维护与贡献

- 更新任何 references 前，先核对官方文档对应页面（文档站或源码生成区块），并在 PR 中注明来源。
- 遵守 Agent Skills 约束：name 为 kebab-case 且与目录一致；description ≤ 1024 字符（DSH 目录注入提醒默认 500）；正文渐进式披露。
- 欢迎 PR：修正、更多示例、扩充评测集、其它语言版本。

## License

MIT——见 [LICENSE](LICENSE)。
