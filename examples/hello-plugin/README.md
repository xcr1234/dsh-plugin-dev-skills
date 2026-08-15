# hello-plugin

最小 DSH 插件示例（组合包/bundle 格式，纯 JS、无构建步骤）。加载后打印加载日志，并用 ctx.effect 演示"注册即副作用、卸载自动清理"（每 5 秒心跳，停用时自动清除定时器）。

## 安装

```bash
dsh plugin --profile demo add ./examples/hello-plugin
dsh --profile demo --dump-config    # 应看到 "# == dsh-hello-plugin" 层
dsh --profile demo                  # 启动后日志出现 [hello-plugin] plugin loaded!
```

首次使用会初始化 profile（@deepseek-ai/dsh-base 作为第一个组合包）。若跳过 dump-config 直接启动即可看到心跳日志。

## 说明

- package.json 的 dsh.bundle 声明指向 cordis.patch.yml；patch 行按包名引用插件（Node 模块解析才能找到已安装代码）。
- 换成你自己的插件：复制本目录，改包名、index.js 导出与 patch 中的 id/name。
- 相关标准：dsh-plugin-dev 技能的 references/plugin-anatomy.md（生命周期）与 references/packaging.md（打包与层序）。
