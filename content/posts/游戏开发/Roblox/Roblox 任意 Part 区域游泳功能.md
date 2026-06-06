---
date: '2025-11-22T17:20:08+08:00'
draft: false
title: 'Roblox 任意 Part 区域游泳功能'
tags: ["Roblox","游戏开发","lua"]
---

### 核心功能实现
1. **区域检测**
   - 使用 `PartZone` 模块创建检测区域（名为 "water"）（**可以使用`ZonePlus`来实现**）
   - 当玩家进入指定 Part（Sea）时激活游泳状态
   - 玩家离开区域时恢复正常状态

2. **游泳状态控制**
   ```lua
   -- 进入水体时
   humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)  -- 禁用跳跃
   humanoid:ChangeState(Enum.HumanoidStateType.Swimming)           -- 强制游泳状态
   
   -- 离开水体时
   humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)   -- 恢复跳跃能力
   ```

2. **使用`LinearVelocity`推动人物移动**
   - 创建 `LinearVelocity` 对象施加推进力
   ```lua
   Swim = Instance.new("LinearVelocity")
   Swim.MaxAxesForce = Vector3.new(3000,3000,3000)
   Swim.Attachment0 = Root.RootAttachment           -- 绑定到角色根部件
   ```

4. **游泳运动机制**
   ```lua
   function Swimming()
       -- 按移动方向和步行速度推进
       Swim.VectorVelocity = Humanoid.MoveDirection * Humanoid.WalkSpeed
   end
   ```

---

### 关键代码设计
1. **状态管理优化**
   - 动态创建/销毁 `LinearVelocity` 对象（`Swim`）
   - 离开水域时通过 `Swim:Destroy()` 释放资源

1. **每帧运行**
   ```lua
   RS.PreRender:Connect(function()
     if Swim then Swimming() end  -- 每帧更新游泳动力
   end)
   ```
   - 利用 `PreRender` 事件保证流畅的水中移动
   - 帧率无关的物理更新

1. **玩家的状态设置**
   ```lua
   humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
   ```
   - 避免角色在水体中出现异常姿态

---

### 使用注意事项
1. **必要依赖**
   - 需提前准备一个工具包，这个包的功能包含能检测玩家进入和离开区域，可以用使用`ZonePlus`，我这里使用的是我自己写的垃圾模块。
   - 场景中需存在名为 "water" 的 Part 作为水体区域

2. **角色适配**
   - 自动处理角色加载等待：`CharacterAdded:Wait()`
   - 兼容动态角色重置情况

3. **物理参数调节**
   - 通过修改 `MaxAxesForce` 调整游泳灵敏度
   - 可扩展浮力控制（增加 Y 轴向上力）

## 完整代码
`swim.luau`
```lua
local Sea=game.Workspace:WaitForChild("water")
local PartZone=require(game:GetService("ReplicatedStorage").PartZone.PartZoneCore)
local StarterGui=game:GetService("StarterGui")
local RS:RunService=game:GetService("RunService")
local Character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")


local Swim:LinearVelocity
PartZone.CreateZone("water",Sea,
	-- 进入区域触发
	function(player:Player)
		local character = player.Character
		
		local humanoid = character:FindFirstChild("Humanoid")
		if not Swim then
			Swim = Instance.new("LinearVelocity")
			Swim.Parent = Root
			Swim.ForceLimitMode=Enum.ForceLimitMode.PerAxis
			Swim.MaxAxesForce=Vector3.new(3000,3000,3000)
			Swim.Attachment0=Root.RootAttachment
		end
		Swim.VectorVelocity=Vector3.new(0,0,0)
		if humanoid then
			humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
			humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
			humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
			humanoid:ChangeState(Enum.HumanoidStateType.Swimming)
		end
	end,
	-- 离开区域触发
	function(player:Player)
		local character = player.Character
		local humanoid = character:FindFirstChild("Humanoid")
		Swim:Destroy()
		Swim=nil
		if humanoid then
			humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
			humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, true)
			humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
		end
	end
)

local function Swimming()
	--if Root.Position.Y < Sea.Position.Y then
	--	Swim.VectorVelocity=Humanoid.MoveDirection * Humanoid.WalkSpeed
	--end
	Swim.VectorVelocity=Humanoid.MoveDirection * Humanoid.WalkSpeed
end

RS.PreRender:Connect(function(deltaTimeRender: number)
	if Swim then
		Swimming()
	end
end)
```
