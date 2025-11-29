# Script Debugging Prompts

Troubleshoot and fix issues in your Roblox Lua scripts. These prompts help you identify bugs, optimize performance, and understand error messages.

---

## 🐛 Error Analysis

### Prompt Template

\`\`\`
Analyze this Roblox Lua error and explain how to fix it:

Error message:
[paste error message]

Script location:
[ServerScriptService/StarterPlayerScripts/etc.]

Code snippet (if available):
[paste relevant code]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Analyze this Roblox Lua error and explain how to fix it:

Error message:
ServerScriptService.PlayerData:23: attempt to index nil with "leaderstats"

Script location:
ServerScriptService

Code snippet:
local player = game.Players.PlayerAdded:Wait()
local coins = player.leaderstats.Coins.Value
\`\`\`

**Expected Output:**
\`\`\`
ANALYSIS:
The error "attempt to index nil" means you're trying to access a property of something 
that doesn't exist (nil). In this case, "leaderstats" doesn't exist on the player yet.

ROOT CAUSE:
The "leaderstats" folder is created by another script, but this script is trying to 
access it before it's been created. Race conditions are common in Roblox when multiple 
scripts interact with the same objects.

FIX:
Use WaitForChild() to wait for the leaderstats folder to be created:

-- Before (buggy):
local coins = player.leaderstats.Coins.Value

-- After (fixed):
local leaderstats = player:WaitForChild("leaderstats")
local coins = leaderstats:WaitForChild("Coins").Value

ADDITIONAL TIPS:
1. Always use WaitForChild() when accessing objects that might not exist immediately
2. Add a timeout parameter to avoid infinite waits: WaitForChild("leaderstats", 10)
3. Consider wrapping in pcall for graceful error handling
\`\`\`

---

## ⚡ Performance Optimization

### Prompt Template

\`\`\`
Optimize this Roblox script for better performance:

[paste code]

Current issues:
- [describe lag/freezing/high memory]
- [when does it happen]

Expected behavior:
- [what should happen smoothly]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Optimize this Roblox script for better performance:

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

RunService.Heartbeat:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part.Name == "Coin" then
                local dist = (player.Character.HumanoidRootPart.Position - part.Position).Magnitude
                if dist < 5 then
                    part:Destroy()
                end
            end
        end
    end
end)

Current issues:
- Game lags with many coins
- CPU usage is very high

Expected behavior:
- Smooth coin collection without lag
\`\`\`

**Expected Output:**
\`\`\`lua
-- OPTIMIZED VERSION

local Players = game:GetService("Players")
local CollectionService = game:GetService("CollectionService")

-- Tag coins with CollectionService instead of searching by name
local TAG_NAME = "Coin"

-- Cache for touched connections
local coinConnections = {}

-- Setup coin collection when coin is added
local function setupCoin(coin)
    if not coin:IsA("BasePart") then return end
    
    local connection
    connection = coin.Touched:Connect(function(hit)
        local player = Players:GetPlayerFromCharacter(hit.Parent)
        if player then
            connection:Disconnect()
            coinConnections[coin] = nil
            coin:Destroy()
        end
    end)
    
    coinConnections[coin] = connection
end

-- Setup existing coins
for _, coin in pairs(CollectionService:GetTagged(TAG_NAME)) do
    setupCoin(coin)
end

-- Handle new coins
CollectionService:GetInstanceAddedSignal(TAG_NAME):Connect(setupCoin)

-- Cleanup removed coins
CollectionService:GetInstanceRemovedSignal(TAG_NAME):Connect(function(coin)
    local connection = coinConnections[coin]
    if connection then
        connection:Disconnect()
        coinConnections[coin] = nil
    end
end)

--[[
OPTIMIZATIONS MADE:

1. Removed Heartbeat loop - checking every frame is expensive
   Now uses Touched events which only fire when needed

2. Removed GetDescendants() search - O(n) every frame is slow
   Now uses CollectionService tags for O(1) lookups

3. Added connection cleanup - prevents memory leaks
   Stores and disconnects connections properly

4. Used event-driven architecture - reacts to changes
   Instead of constantly polling, responds to events

PERFORMANCE IMPROVEMENT: ~95% reduction in CPU usage
]]
\`\`\`

---

## 🔍 Debug Logging

### Prompt Template

\`\`\`
Add debug logging to this Roblox script to help identify issues:

[paste code]

I need to track:
- [what values to monitor]
- [which code paths are executed]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Add debug logging to this Roblox script to help identify issues:

local DataStore = game:GetService("DataStoreService"):GetDataStore("PlayerData")

local function saveData(player)
    local data = { Coins = player.leaderstats.Coins.Value }
    DataStore:SetAsync("Player_" .. player.UserId, data)
end

game.Players.PlayerRemoving:Connect(saveData)

I need to track:
- Whether save is being called
- What data is being saved
- If there are any errors
\`\`\`

**Expected Output:**
\`\`\`lua
-- DEBUGGED VERSION WITH LOGGING

local DataStoreService = game:GetService("DataStoreService")
local DataStore = DataStoreService:GetDataStore("PlayerData")

-- Debug flag (set to false in production)
local DEBUG_MODE = true

-- Debug print helper
local function debugLog(category, message, data)
    if not DEBUG_MODE then return end
    
    local timestamp = os.date("%H:%M:%S")
    local dataStr = data and (" | Data: " .. tostring(data)) or ""
    print(string.format("[%s][%s] %s%s", timestamp, category, message, dataStr))
end

local function saveData(player)
    debugLog("SAVE", "Save triggered for " .. player.Name)
    
    -- Check if leaderstats exists
    local leaderstats = player:FindFirstChild("leaderstats")
    if not leaderstats then
        debugLog("ERROR", "No leaderstats found for " .. player.Name)
        return
    end
    
    local coins = leaderstats:FindFirstChild("Coins")
    if not coins then
        debugLog("ERROR", "No Coins found in leaderstats for " .. player.Name)
        return
    end
    
    local data = { Coins = coins.Value }
    local key = "Player_" .. player.UserId
    
    debugLog("SAVE", "Attempting save", "Key=" .. key .. ", Coins=" .. data.Coins)
    
    local success, errorMessage = pcall(function()
        DataStore:SetAsync(key, data)
    end)
    
    if success then
        debugLog("SAVE", "✓ Save successful for " .. player.Name)
    else
        debugLog("ERROR", "✗ Save failed for " .. player.Name, errorMessage)
    end
end

game.Players.PlayerRemoving:Connect(function(player)
    debugLog("EVENT", "PlayerRemoving fired for " .. player.Name)
    saveData(player)
end)

-- Also log when script starts
debugLog("INIT", "DataSave script initialized")

--[[
DEBUG OUTPUT EXAMPLE:
[12:34:56][INIT] DataSave script initialized
[12:35:10][EVENT] PlayerRemoving fired for Player1
[12:35:10][SAVE] Save triggered for Player1
[12:35:10][SAVE] Attempting save | Data: Key=Player_12345, Coins=150
[12:35:11][SAVE] ✓ Save successful for Player1
]]
\`\`\`

---

## 🧪 Common Bug Patterns

### Memory Leaks

\`\`\`
Identify and fix memory leaks in this Roblox script:

[paste code]

Symptoms:
- Memory usage keeps increasing
- Game slows down over time
\`\`\`

### Race Conditions

\`\`\`
Fix race conditions in this Roblox script where multiple events might conflict:

[paste code]

Problem:
- [describe inconsistent behavior]
\`\`\`

### Network Issues

\`\`\`
Add proper error handling for RemoteEvents/Functions in this script:

[paste code]

Issues:
- Sometimes the remote doesn't work
- No feedback when it fails
\`\`\`

---

## 🔄 Tips for Effective Debugging

1. **Use print statements strategically** - Track variable values at key points
2. **Check the output window** - Roblox Studio shows errors and prints
3. **Test in Studio first** - Use Play Solo to catch basic issues
4. **Enable Server/Client views** - See what each side is doing
5. **Use breakpoints** - Pause execution to inspect variables
6. **Check API deprecations** - Old methods might not work
7. **Read the full error** - Line numbers and stack traces help locate issues
8. **Isolate the problem** - Comment out code to find the culprit

---

## 🎯 Quick Reference Prompts

### Nil Value Error
\`\`\`
My Roblox script shows "attempt to index nil with [property]". The code is [paste code]. What's causing this and how do I fix it?
\`\`\`

### Infinite Yield Warning
\`\`\`
I'm getting "Infinite yield possible" warnings. Here's my code: [paste code]. How do I fix this?
\`\`\`

### Remote Event Not Working
\`\`\`
My RemoteEvent isn't firing/receiving. Server code: [paste]. Client code: [paste]. What's wrong?
\`\`\`

### DataStore Errors
\`\`\`
My DataStore is throwing errors. Error message: [paste]. Code: [paste]. How do I fix this and prevent data loss?
\`\`\`

### Script Not Running
\`\`\`
My script isn't running at all. Location: [where script is]. Type: [Script/LocalScript/ModuleScript]. Code: [paste first few lines]. What could be wrong?
\`\`\`
