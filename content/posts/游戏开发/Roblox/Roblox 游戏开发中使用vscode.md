---
date: '2025-11-22T17:22:45+08:00'
draft: false
title: 'Roblox 游戏开发中使用vscode'
tags: ["Roblox","游戏开发","lua"]
---

在 Roblox 游戏开发领域，高效的工具管理与资源整合是提升开发效率的关键。aftman 工作流凭借其强大的工具管理能力，成为开发者的得力助手。它以 aftman 作为工具管理器（**也可以用使用npm，毕竟aftman很久不更新了**），统筹 wally 和 rojo 两大工具，前者用于管理第三方库，后者负责同步，搭配 vscode 中的 roblox lsp 插件实现语法补全，还可借助 roblox lsp（knit）插件和 selene 进一步优化语法与错误检测。接下来，就为大家详细介绍具体使用方法。

## 安装 aftman

想要开启 aftman 工作流，首先要安装 aftman。前往 GitHub 网站，找到与你系统对应的版本进行下载安装。安装完成后，便可着手创建新项目。在当前文件夹下，通过运行aftman init命令，创建一个新的aftman.toml文件，这是后续管理工具的重要配置文件。
**（这部分也以使用npm进行，不过wally好像在npm中没有包，可以自己放入环境变量中，rojo可以直接使用npm进行安装，比aftman方便些）**

## 添加工具

### 添加 rojo

添加工具时，以 rojo 为例（==我这里使用的是当前的最新版，现在最新可能不是这个版本==）你可以使用以下命令：

```bash
# 添加最新版本的rojo
aftman add rojo-rbx/rojo
# 添加指定版本7.5.1的rojo
aftman add rojo-rbx/rojo@7.5.1
# 以不同的二进制名称添加工具，例如添加ripgrep并命名为rg
aftman add BurntSushi/ripgrep rg
```

### 添加 wally

使用 aftman 添加 wally 的流程如下：

```bash
# 初始化aftman配置
aftman init
# 添加wally工具
aftman add UpliftGames/wally
# 安装已添加的工具
aftman install
```
#### 配置 wally

在 wally 中添加服务器或客户端的包，需要对配置文件进行设置。在相关配置文件中，通过以下内容进行配置：

```toml
[package]
name = "11588/one"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"
[dependencies]
Signal = "sleitnick/signal@^1"
Trove = "sleitnick/trove@1.5.0"
[server-dependencies]
# 这里用于添加服务器中的内容
```

其中，package部分定义了包的基本信息，dependencies用于添加通用依赖，server-dependencies则专门用于添加服务器端的依赖内容，一般客户端都会放到`ReplicatedStorage`中，相当于客户端服务端共享，但是不互通。

#### rojo 同步 wally 库

完成上述配置后，需要通过 rojo 实现 wally 库的同步。在 rojo 的配置文件中，添加如下内容：

```json
{
    "name": "wally test",
    "tree": {
        "$className": "DataModel",
        "ReplicatedStorage": {
            "$className": "ReplicatedStorage",
            "$ignoreUnknownInstances": true,
            "Packages": {
                "$path": "Packages"
            }
        },
        "ServerScriptService": {
            "$className": "ServerScriptService",
            "Package": {
                "$path": "ServerPackages"
            }
        }
    }
}
```

通过这样的配置，rojo 能够将 wally 管理的第三方库同步到 Roblox 游戏项目中。
