---
date: '2026-06-06T15:01:23+08:00'
draft: false
title: '关于roblox-ts的一些踩坑点'
tags: ["Roblox","游戏开发","typescript","roblox-ts"]
---

## VS Code 配置问题

VS Code 默认使用内置的 TypeScript 版本，而非工作区指定的版本。这会导致一些问题，比如：

- `tsconfig.json` 中提示需要添加忽略配置
- `*.ts` 代码中提示某些内置函数/方法不存在

虽然 roblox-ts 的 VS Code 插件会自动帮我们设置工作区 TypeScript 版本，但 VS Code 不会自动识别。需要在 `.vscode/settings.json` 中添加以下配置，让 VS Code 提示是否切换到工作区版本：

```json
{
  "js/ts.tsdk.promptToUseWorkspaceVersion": true
}
```

添加后，VS Code 右下角会弹出提示，点击确认即可使用项目中的 TypeScript 版本，上述报错即可解决。