# dsh-plugin-dev

<p align="center">
  <samp>
    <strong>English</strong> ·
    <a href="./README.zh.md">中文</a>
  </samp>
</p>

An [Agent Skills](https://agentskills.io)–compliant skill that teaches agents how to develop plugins for **DeepSeek Harness (DSH)**.

DeepSeek Harness is a plugin-based SDK for building agent harnesses: model adapters, the tool registry, the session log, even the agent loop itself — everything is a Cordis plugin that can be swapped from configuration. This skill distills the official DSH documentation into one executable standard for writing those plugins, so any agent that loads it develops DSH plugins the same way every time.

## What's inside

```
dsh-plugin-dev/
├── SKILL.md      # entry point: frontmatter, 8 hard rules, 6 scenario workflows,
│                 # decision tables, and a pre-completion checklist
├── references/   # 12 detailed standards, loaded on demand (progressive disclosure)
├── examples/     # two minimal, copy-and-run example plugins
│   ├── hello-plugin/
│   └── greet-tool/
└── evals/        # trigger-evaluation set and methodology for the description
```

The reference library covers: plugin anatomy & lifecycle · services & dependency injection · all five event dispatch modes · plugin configuration · Context/Fiber/registry APIs · three-role capability design (Definition/Provider/Consumer) · tool development · the LLM adapter protocol · plugin form extensions (tool/hook/UI/protocol bridge) · packaging & installation · in-repo workspace packages · the complete capability-seam catalog.

## Installation

Copy the whole `dsh-plugin-dev` folder into your agent's skill directory. No build step, no script dependencies, no configuration.

| Agent | Project-level | User-level |
| --- | --- | --- |
| DeepSeek Harness | `<project>/.dsh/skills/` (rank 100) or `<project>/.agents/skills/` (rank 200) | `~/.dsh/skills/` (rank 400) |
| Claude Code | `<project>/.claude/skills/` | `~/.claude/skills/` |
| Codex | `<project>/.codex/skills/` | `~/.codex/skills/` |
| VS Code Copilot | `<project>/.agents/skills/` | `~/.agents/skills/` |
| Other compatible agents | follow that agent's skill-directory convention | same |

To verify: ask the agent "开发一个 DSH 插件" or "write a DSH tool" — the skill should trigger. In DSH you can also load it directly with the `skill(dsh-plugin-dev)` tool.

## Version tracking

The content was distilled from the official DeepSeek Harness documentation site (deepseek-harness.github.io, snapshot dated 2026-08) and follows the project's own principle: when the skill disagrees with the repository's generated references, **the generated references win**. If you hit a discrepancy, please open an issue or PR here.

## Trigger evals

`evals/trigger-queries.json` is the regression set for the skill's description (12 should-trigger + 9 should-not-trigger queries). Before changing the description, run the evals and record pass rates — `evals/README.md` explains the methodology, including train/validation splits to avoid overfitting.

## Examples

- `examples/hello-plugin` — a minimal plugin in bundle format. Run `dsh plugin --profile demo add ./examples/hello-plugin`, then `dsh --profile demo`: you should see `[hello-plugin] plugin loaded!` and a heartbeat every 5 seconds, cleaned up automatically on unload.
- `examples/greet-tool` — a minimal model-facing tool. After installing it, ask the agent: "Use the greet tool to greet Ada." It should reply "Hello, Ada!".

## Scope

This skill covers **file-based** DSH plugin development: plugin packages, cordis.yml rows, patch overlays, tools, adapters, bundles, profiles, and in-repo workspace packages. Out of scope: in-session dynamic plugins (the `cordis_define`/`cordis_run` flow) and agent-preset composition editing — those are handled by each deployment's own dedicated skills and tools.

## Contributing

- Before updating any reference, check the corresponding official docs page (site or generated source blocks) and cite it in your PR.
- Respect the Agent Skills constraints: kebab-case `name` matching the folder; `description` ≤ 1024 chars (DSH's catalog reminder caps at 500); progressive disclosure in the body.
- PRs are welcome: fixes, more examples, a larger eval set, and translations.

## License

MIT — see [LICENSE](LICENSE).
