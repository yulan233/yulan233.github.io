---
date: '2025-11-22T16:01:23+08:00'
draft: false
title: '关于 Roblox GUI 悬停效果的实现'
tags: ["Roblox","游戏开发","lua"]
---

在 Roblox 开发中，实现 GUI 悬停效果时，`MouseEnter` 和 `MouseLeave` 事件有时会表现得不稳定。本文记录了这些传统方法遇到的问题，并介绍一个更可靠的替代方案：使用 `GuiObject` 的 `GuiState` 属性。
#### 传统方法及其局限性
常用的 `MouseEnter` 和 `MouseLeave` 事件存在一些已知问题：
1.  **快速移动导致的事件丢失**：当鼠标快速划过 GUI 元素时，引擎可能无法正确触发 `MouseEnter` 或 `MouseLeave`，导致效果缺失或闪烁。
2.  **删除元素不会触发Leave**：当你删除绑定了`MouseLeave`的元素的时候不会触发这个事件，会导致很多问题。
3.  **动态 GUI 的处理复杂**：在鼠标悬停期间，如果 GUI 对象的位置或大小发生变化，事件监听逻辑容易出错。
另一种尝试是使用 `InputBegan` 和 `InputEnded` 事件，并检查 `UserInputType` 是否为 `MouseMovement`。
```lua
-- 使用 InputBegan/Ended 的示例（不推荐）
Object.InputBegan:Connect(function(Input)
	if Input.UserInputType == Enum.UserInputType.MouseMovement then
		print("Mouse entered")
	end
end)
Object.InputEnded:Connect(function(Input)
	if Input.UserInputType == Enum.UserInputType.MouseMovement then
		print("Mouse left")
	end
end)
```
这种方法检测的是“鼠标在对象上开始/停止移动”，而非真正的“进入/离开”。当鼠标静止在对象上然后直接移出时，`InputEnded` 可能不会被触发。它同样会受到子元素的干扰。
#### 使用 `GuiState` 属性
一个更稳定的方案是监听 `GuiObject` 的 `GuiState` 属性。`GuiState` 是一个枚举，它直接反映了 GUI 对象的当前交互状态。其中两个关键状态是：
*   `Enum.GuiState.Hover`：鼠标指针悬停在对象上。
*   `Enum.GuiState.Idle`：鼠标指针不在对象上。
通过监听 `GuiState` 的变化，可以获得一个由引擎底层管理的状态信号，从而避免了上述传统方法的问题。
#### 结合 TweenService 的实现示例
以下代码展示了如何结合 `TweenService` 和 `GuiState` 来实现一个带动画和音效的按钮悬停效果。
```lua
-- 获取服务
local TweenService = game:GetService("TweenService")
-- 定义动画信息
local TWEEN_INFO = TweenInfo.new(
	0.1,                           -- 动画时长
	Enum.EasingStyle.Linear,       -- 缓动样式
	Enum.EasingDirection.InOut     -- 缓动方向
)
-- 获取按钮对象（假设此脚本位于按钮内部）
local Button = script.Parent
local HoverSound = Button:WaitForChild("Sound")
-- 初始化 Tween 变量
local tween = TweenService:Create(Button, TWEEN_INFO, {})
-- 监听 "GuiState" 属性的变化
Button:GetPropertyChangedSignal("GuiState"):Connect(function()
	if Button.GuiState == Enum.GuiState.Hover then
		-- 鼠标悬停时
		tween:Cancel() -- 取消当前动画
		tween = TweenService:Create(Button, TWEEN_INFO, {
			Size = UDim2.new(0, 250, 0, 63) -- 悬停尺寸
		})
		tween:Play()
		HoverSound:Play()
		
	elseif Button.GuiState == Enum.GuiState.Idle then
		-- 鼠标离开时
		tween:Cancel() -- 取消当前动画
		tween = TweenService:Create(Button, TWEEN_INFO, {
			Size = UDim2.new(0, 210, 0, 63) -- 原始尺寸
		})
		tween:Play()
	end
end)
```
#### 代码要点说明
1.  **`GetPropertyChangedSignal("GuiState")`**：此方法创建一个事件，仅在 `GuiState` 属性值改变时触发，比逐帧检查更高效。
2.  **状态判断**：在回调函数中，直接检查 `Button.GuiState` 的值来执行相应逻辑。`Hover` 对应鼠标进入，`Idle` 对应鼠标离开。
3.  **`tween:Cancel()`**：在创建新动画前取消旧动画，可以防止用户快速移入移出时，多个动画排队执行导致的视觉延迟或抖动。
4.  **动态创建 Tween**：每次状态改变时都重新创建 `Tween` 对象，并设定新的目标属性，使代码逻辑清晰。
#### 总结
对于 Roblox GUI 开发，使用 `GuiState` 属性来处理悬停效果是比 `MouseEnter` 和 `MouseLeave` 更可靠的选择。它在处理子元素、快速移动等场景时表现更稳定，同时代码逻辑也更直接。在新的项目中，建议优先考虑使用 `GuiState`。
