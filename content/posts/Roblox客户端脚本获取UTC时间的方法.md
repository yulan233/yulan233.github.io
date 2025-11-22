---
date: '2025-11-22T16:40:08+08:00'
draft: false
title: 'Roblox客户端脚本获取UTC时间的方法'
tags: ["Roblox","游戏开发","lua"]
---

在 Roblox 开发中，我们经常需要获取一个不受玩家本地时区影响的标准时间，即 UTC（协调世界时）。虽然在服务器上可以直接使用 `os.time()`，因为它本身就在一个受控的环境中运行，但在客户端，玩家的设备时区各不相同，直接获取本地时间可能会导致数据不一致。
这里记录一个从客户端获取 UTC 时间的 Lua 脚本函数，并对其原理进行简单说明。

### 脚本代码

```lua
TimeAbout = {}
TimeAbout.GetUTC = function(add:number?)
    -- 如果传入的参数不是数字，则默认为 0
    if type(add) ~= "number" then
        add = 0
    end
    -- 获取当前本地时间戳
    local currentTime = os.time()
    
    -- 使用 "!" 标记获取当前时间的 UTC 时间表
    local UTC = os.date("!*t", currentTime)
    
    -- 获取当前时间的本地时间表
    local localtime = os.date("*t", currentTime)
    
    -- 计算本地时间与 UTC 时间的时间差（秒）
    -- os.difftime 会返回两个时间戳的差值
    local diff = os.difftime(os.time(UTC), os.time(localtime))
    
    -- 将本地时间戳加上时差，得到 UTC 时间戳
    local utcTime = currentTime + diff
    
    -- 返回 UTC 时间戳，并加上可选的小时偏移量
    return utcTime + add * 3600
end
```
### 实现原理说明
这个函数的核心思想是计算出本地时间与 UTC 时间之间的时差，然后用这个时差来修正本地时间戳。
1. **获取本地时间戳**：`os.time()` 函数返回一个表示当前本地时间的 Unix 时间戳（自 1970 年 1 月 1 日以来的秒数）。

2.  **获取 UTC 和本地时间表**：
    *   `os.date("!*t", currentTime)`：`os.date` 函数的第一个参数是格式化字符串。当字符串以 `!` 开头时，它会将时间解释为 UTC 时间，而不是本地时间。`*t` 则表示返回一个包含年、月、日、时等字段的时间表。
    *   `os.date("*t", currentTime)`：不加 `!` 则返回本地时间的时间表。
    
3.  **计算时差**：
    *   `os.time(UTC)` 和 `os.time(localtime)` 会将这两个时间表分别转换回时间戳。由于 `UTC` 表是基于 UTC 时间的，而 `localtime` 表是基于本地时间的，这两个时间戳之间的差值 `diff` 正好是本地时区与 UTC 的偏移量（以秒为单位）。
    *   例如，如果本地时间是东八区（UTC+8），那么本地时间戳会比 UTC 时间戳大 8 * 3600 = 28800 秒。因此 `diff` 的值就是 -28800。
    
4.  **修正并返回**：
    * 将原始的本地时间戳 `currentTime` 加上这个时差 `diff`，就得到了准确的 UTC 时间戳 `utcTime`。
    
    * `add` 参数是一个可选的数字，代表要增加或减少的小时数。函数会将其转换为秒（`add * 3600`）并加到最终的 UTC戳上，方便进行时间计算。
    

## 结尾

可能从服务端直接获取时间会更好一些，不过这也是一种不依靠服务端就能获取到utc时间的方法，可能有时候会有需要。
