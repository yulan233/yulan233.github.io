---
date: '2025-12-20T16:06:32+08:00'
draft: false
title: '在Win和ArchLinux上，让Caps Lock键变成中英文切换键'
tags: ["小技巧","win","linux","archlinux","keyd","autohotkey"]
---
### 在 Windows 和 Arch Linux 上，让 Caps Lock 键变成中英文切换键
用惯了 Mac 之后，再回到 Windows 或 Linux 电脑上，总有些小地方让人感觉不太适应。对我来说，其中一个就是键盘上的 Caps Lock 键。
在 macOS 上，Caps Lock 键默认可以用来快速切换中英文输入法，这个设计非常直观，也减少了手指的移动距离。但在 Windows 和大多数 Linux 发行版里，这个键通常只负责大小写锁定，一个功能对我来说使用频率并不高。于是，我便开始琢磨，能不能在这两大主流操作系统上也实现类似的功能。

经过一番探索，我找到了解决方案。在 Windows 上，我们可以使用 AutoHotkey；而在 Arch Linux 上，我选择的是`Keyd`。
---
#### Windows 篇：使用 AutoHotkey
AutoHotkey 是一款免费且功能强大的自动化脚本软件，可以通过编写脚本来定制各种键盘快捷键。我们只需要用到它最基础的一点功能。
**准备工作**
首先，你需要从 AutoHotkey 的官网（[https://www.autohotkey.com/](https://www.autohotkey.com/)）下载并安装它。安装时选择默认选项即可。
**实现 Caps Lock 切换中英文**
这个脚本不仅能让单击 Caps Lock 切换中英文，还保留了长按切换大小写的功能。
在使用前，请确保你的 Windows 系统默认的中英文切换快捷键是 `Ctrl + Space`。如果不是，可以在“设置”->“时间和语言”->“语言和区域”->“输入”->“高级键盘设置”中进行修改。
脚本内容如下：

```autohotkey
#Requires AutoHotkey v2.0
#SingleInstance Force
; 让脚本在后台持续运行
Persistent
global isCapsPressed := false
CapsLock::
{
	if (KeyWait("CapsLock", "T0.5")) {
		if(!isCapsPressed){
			; 如果是单击，则发送 Ctrl+Space 来切换中英文
			Send "^{Space}"
		}
		global isCapsPressed := false
	} else {
		if(!isCapsPressed){
			global isCapsPressed := true
			; 如果是长按（超过0.5秒），则切换大小写锁定状态
			SetCapsLockState !GetKeyState("CapsLock", "T")
		}
	}
}
```
**如何使用这个脚本：**

1. 在电脑的任意位置新建一个文本文档。

2. 将上面的代码复制并粘贴到这个文本文档中。

3. 保存文件，然后将文件名从 `.txt` 修改为 `.ahk`，例如 `SwitchIME.ahk`。

4. 双击运行这个 `.ahk` 文件。

**设置开机自启动**
   每次开机都手动双击运行脚本会比较麻烦，我们可以把它设置成开机自动启动。

1. 打开文件资源管理器。

2. 在顶部的地址栏输入 `shell:startup`，然后按回车键。这时会打开 Windows 的“启动”文件夹。

3. 将你刚才创建的那个 `.ahk` 脚本文件，复制或剪切到这个文件夹里。

#### Arch Linux 篇：使用 keyd
在 Arch Linux 上，我选择的是`keyd`，为什么是这个，没啥原因，只是选了这个:3
**准备工作：安装 keyd**
在 Arch Linux 上，安装 `keyd` 非常简单，直接使用 `pacman` 即可。

```bash
sudo pacman -S keyd
```
安装完成后，需要启用并启动 `keyd` 服务（这里是设置为开机自启和现在就启动）：
```bash
sudo systemctl enable --now keyd
```
**配置 keyd 实现 Caps Lock 切换**
`keyd` 的配置文件位于 `/etc/keyd/` 目录下，例如 `default.conf`，这是`keyd`的默认配置。

```bash
sudo nano /etc/keyd/default.conf
```
将以下内容粘贴到文件中：
```ini
[ids]
*

[main]
# 将 Caps Lock 配置为：
# - 短按：作为一次性的Ctrl+空格组合键
# - 长按 500ms：切换大小写锁定状态
capslock = timeout(macro(C-space),500,capslock)
```
**配置详解：**

因为我是用的fcitx5作为输入法，然后它默认是`Ctrl+空格`作为输入法切换，然后只要把这个shift的按键绑定取消在加个这个就可以实现我们想要的效果了，下面是这条命令的解释。

`timeout(<动作1>, <超时时间>, <动作2>)`

如果该按键被单独按住超过 `<超时时间>` 毫秒，则激活第二个动作；如果按住时间少于 `<超时时间>` 毫秒，或在 `<超时时间>` 到期前按下另一个键，则执行第一个动作。

**应用配置并验证**
保存并关闭文件后，需要重新加载 `keyd` 配置使其生效：

```bash
sudo keyd reload
```
现在，你可以立即测试一下：
*   **单击** Caps Lock，应该会切换输入法。
*   **长按** Caps Lock 超过 500 毫秒，应该会切换大小写锁定。