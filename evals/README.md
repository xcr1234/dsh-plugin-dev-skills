# 触发评测（Trigger evals）

本目录是 description 的回归评测集，方法论来自 Agent Skills 官方文档《Optimizing skill descriptions》。

## 评测集

`trigger-queries.json` 包含 12 条 should-trigger（应触发）与 9 条 should-not-trigger（不应触发）查询：

- **正例设计**：不同措辞、繁简、错别字容忍、中英文、直接点名与隐式描述（"加一个能读文件的工具"未提 DSH）、多步工作流（打包安装、workspace 包）。
- **负例设计**：无关领域（天气、翻译、SQL、Chrome 扩展、GitHub Actions）+ **易混淆近邻**（Koishi 同为 Cordis 系但目标框架不同；claude code hooks 配置属于其它专项技能）。

## 运行方法

1. 用目标 agent 逐条发查询，记录是否调用了本技能（DSH 中可用 `skill` 工具调用来确认；Claude Code 可用 `--output-format json` 检查 `Skill` 工具调用，见官方文档示例脚本）。
2. 每条查询按"should_trigger 与是否触发一致"计通过；统计通过率 = 通过条数 / 总条数。
3. 每条多跑 2-3 次取多数（触发本身有随机性）。

## 训练/验证集划分（防过拟合）

- **训练集（~60%）**：用来定位失败、指导修改 description。
- **验证集（~40%）**：只在确认修改是否泛化时使用，不进修改过程。
- 修改原则：正例失败 → description 过窄，加使用场景；负例误触 → description 过宽，写清边界。**不要直接抄失败查询里的关键词**——找它代表的类别。选最优版本按验证集通过率，不一定是最后一个版本。

## 记录

每次修改 description 后，把通过率记入 CHANGELOG 或提交信息，例如：`evals: zh 12/12, en 3/3, negatives 9/9`。
