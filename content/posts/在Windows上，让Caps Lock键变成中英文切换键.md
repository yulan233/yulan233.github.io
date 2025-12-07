---
date: '2025-12-07T16:06:32+08:00'
draft: false
title: '在Windows上，让Caps Lock键变成中英文切换键'
tags: ["小技巧","win"]
---
### 在Windows上，让Caps Lock键变成中英文切换键
用惯了Mac之后，再回到Windows电脑上，总有些小地方让人感觉不太适应。对我来说，其中一个就是键盘上的Caps Lock键。
在macOS上，Caps Lock键默认可以用来快速切换中英文输入法，这个设计非常直观，也减少了手指的移动距离。但在Windows系统里，这个键通常只负责大小写锁定，一个功能对我来说使用频率并不高。于是，我便开始琢磨，能不能在Windows上也实现类似的功能。
经过一番搜索，我发现了一款名为AutoHotkey的免费软件。它功能非常强大，可以通过编写脚本来定制各种键盘快捷键和自动化操作。虽然它的功能很多，但我们只需要用到其中最基础的一点。

#### 准备工作
首先，你需要从AutoHotkey的官网（[https://www.autohotkey.com/](https://www.autohotkey.com/)）下载并安装它。安装时选择默认选项即可。
安装完成后，我们就可以开始编写脚本了。

#### 实现Caps Lock切换中英文
网上有很多人和我有同样的需求，所以相关的脚本也并不难找。下面这个脚本就能很好地实现我们的目标，它不仅能让单击Caps Lock切换中英文，还保留了长按切换大小写的功能，可以说是相当完善了。
在使用前，请确保你的Windows系统默认的中英文切换快捷键是`Ctrl + Space`。如果不是，可以在“设置”->“时间和语言”->“语言和区域”->“输入”->“高级键盘设置”中进行修改。
脚本内容如下：

```ank
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
1.  在电脑的任意位置新建一个文本文档。
2.  将上面的代码复制并粘贴到这个文本文档中。
3.  保存文件，然后将文件名从`.txt`修改为`.ahk`，例如`SwitchIME.ahk`。
4.  双击运行这个`.ahk`文件。
现在，你的Caps Lock键就已经被“改造”了。试着单击一下，应该就能切换输入法了；如果长按不放，则会像原来一样切换大小写锁定。
#### 设置开机自启动
每次开机都手动双击运行脚本会比较麻烦，我们可以把它设置成开机自动启动。
方法很简单：
1.  打开文件资源管理器。
2.  在顶部的地址栏输入`shell:startup`，然后按回车键。这时会打开Windows的“启动”文件夹。
3.  将你刚才创建的那个`.ahk`脚本文件，复制或剪切到这个文件夹里。
完成以上步骤后，下次重启电脑时，这个脚本就会自动在后台运行，你无需任何额外操作就能直接享受这个便利的功能了。