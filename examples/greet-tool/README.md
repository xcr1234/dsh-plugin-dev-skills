# greet-tool

最小模型工具示例：注册一个 greet 工具（defineTool + output.render），模型可以调用它并收到结果。@deepseek-ai/dsh-tools 由 dsh 安装目录提供，无需额外安装。

## 安装

```bash
dsh plugin --profile demo add ./examples/greet-tool
dsh --profile demo
```

然后在 Web UI 里输入：

> Use the greet tool to greet Ada.

模型会调用 greet 并收到 "Hello, Ada!"。

## 说明

- inject: ['tools'] 让 Cordis 等待工具注册表就绪；defineTool 根据 parameters 推导并校验 args；execute 返回 output.schema 声明的规范值，output.render 再把它转成面向模型的内容。
- 进阶：给问候语加配置（Config + Schemastery）见 references/config.md；把工具背后的能力拆成三种角色见 references/three-roles.md。
