# DataStore Logic Prompts

Create robust data persistence systems for your Roblox games. These prompts help you generate reliable DataStore code with proper error handling, retries, and session locking.

---

## 💾 Basic Data Saving

### Prompt Template

```
Generate a Roblox DataStore script that saves and loads [data types] for players.

Requirements:
- Data structure: [describe your data schema]
- Auto-save interval: [X] minutes
- Save on player leaving
- Include retry logic for failed saves
- Handle edge cases like [specific concerns]
```

### Example Usage

**Input:**
```
Generate a Roblox DataStore script that saves and loads player stats.

Requirements:
- Data structure: Coins (number), Level (number), Inventory (table of item IDs)
- Auto-save interval: 5 minutes
- Save on player leaving
- Include retry logic for failed saves (3 attempts)
- Handle edge cases like player rejoining quickly
```

**Expected Output:**
```lua
-- DataManager.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local RunService = game:GetService("RunService")

local playerDataStore = DataStoreService:GetDataStore("PlayerData_v1")

local AUTO_SAVE_INTERVAL = 300 -- 5 minutes
local MAX_RETRIES = 3
local RETRY_DELAY = 1

-- Cache for player data
local playerDataCache = {}
local dataLoaded = {}

-- Default data template
local DEFAULT_DATA = {
    Coins = 0,
    Level = 1,
    Inventory = {}
}

-- Deep copy function
local function deepCopy(original)
    local copy = {}
    for key, value in pairs(original) do
        if type(value) == "table" then
            copy[key] = deepCopy(value)
        else
            copy[key] = value
        end
    end
    return copy
end

-- Save with retry logic
local function savePlayerData(player, data)
    local userId = player.UserId
    local key = "Player_" .. userId
    
    for attempt = 1, MAX_RETRIES do
        local success, errorMessage = pcall(function()
            playerDataStore:SetAsync(key, data)
        end)
        
        if success then
            print("[DataManager] Saved data for " .. player.Name)
            return true
        else
            warn("[DataManager] Save attempt " .. attempt .. " failed for " .. player.Name .. ": " .. errorMessage)
            if attempt < MAX_RETRIES then
                wait(RETRY_DELAY * attempt)
            end
        end
    end
    
    warn("[DataManager] All save attempts failed for " .. player.Name)
    return false
end

-- Load with retry logic
local function loadPlayerData(player)
    local userId = player.UserId
    local key = "Player_" .. userId
    
    for attempt = 1, MAX_RETRIES do
        local success, result = pcall(function()
            return playerDataStore:GetAsync(key)
        end)
        
        if success then
            if result then
                print("[DataManager] Loaded data for " .. player.Name)
                return result
            else
                print("[DataManager] No existing data for " .. player.Name .. ", using defaults")
                return deepCopy(DEFAULT_DATA)
            end
        else
            warn("[DataManager] Load attempt " .. attempt .. " failed for " .. player.Name .. ": " .. result)
            if attempt < MAX_RETRIES then
                wait(RETRY_DELAY * attempt)
            end
        end
    end
    
    warn("[DataManager] All load attempts failed for " .. player.Name .. ", using defaults")
    return deepCopy(DEFAULT_DATA)
end

-- Setup player data
local function onPlayerAdded(player)
    -- Prevent loading if already cached (quick rejoin)
    if playerDataCache[player.UserId] and dataLoaded[player.UserId] then
        print("[DataManager] Using cached data for " .. player.Name)
        return
    end
    
    local data = loadPlayerData(player)
    playerDataCache[player.UserId] = data
    dataLoaded[player.UserId] = true
    
    -- Create leaderstats
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local coins = Instance.new("IntValue")
    coins.Name = "Coins"
    coins.Value = data.Coins
    coins.Parent = leaderstats
    
    local level = Instance.new("IntValue")
    level.Name = "Level"
    level.Value = data.Level
    level.Parent = leaderstats
    
    -- Sync changes to cache
    coins.Changed:Connect(function(newValue)
        if playerDataCache[player.UserId] then
            playerDataCache[player.UserId].Coins = newValue
        end
    end)
    
    level.Changed:Connect(function(newValue)
        if playerDataCache[player.UserId] then
            playerDataCache[player.UserId].Level = newValue
        end
    end)
end

-- Save and cleanup on leave
local function onPlayerRemoving(player)
    local data = playerDataCache[player.UserId]
    if data then
        savePlayerData(player, data)
        
        -- Delay cleanup for quick rejoin handling
        delay(60, function()
            if not Players:FindFirstChild(player.Name) then
                playerDataCache[player.UserId] = nil
                dataLoaded[player.UserId] = nil
            end
        end)
    end
end

-- Auto-save loop
spawn(function()
    while true do
        wait(AUTO_SAVE_INTERVAL)
        
        for _, player in pairs(Players:GetPlayers()) do
            local data = playerDataCache[player.UserId]
            if data then
                savePlayerData(player, data)
            end
        end
    end
end)

-- Bind to player events
Players.PlayerAdded:Connect(onPlayerAdded)
Players.PlayerRemoving:Connect(onPlayerRemoving)

-- Handle existing players (for script reload in Studio)
for _, player in pairs(Players:GetPlayers()) do
    coroutine.wrap(onPlayerAdded)(player)
end

-- Bind to game close
game:BindToClose(function()
    if RunService:IsStudio() then
        wait(2)
    else
        local saveJobs = {}
        
        for _, player in pairs(Players:GetPlayers()) do
            local data = playerDataCache[player.UserId]
            if data then
                table.insert(saveJobs, coroutine.create(function()
                    savePlayerData(player, data)
                end))
            end
        end
        
        for _, job in pairs(saveJobs) do
            coroutine.resume(job)
        end
        
        wait(5)
    end
end)

-- Module API for other scripts
local DataManager = {}

function DataManager:GetData(player)
    return playerDataCache[player.UserId]
end

function DataManager:SetData(player, key, value)
    if playerDataCache[player.UserId] then
        playerDataCache[player.UserId][key] = value
        return true
    end
    return false
end

function DataManager:AddToInventory(player, itemId)
    local data = playerDataCache[player.UserId]
    if data then
        table.insert(data.Inventory, itemId)
        return true
    end
    return false
end

return DataManager
```

---

## 🔐 Session Locking

### Prompt Template

```
Generate a Roblox DataStore script with session locking to prevent data duplication exploits.

Requirements:
- Lock data when player joins
- Release lock when player leaves
- Handle server crashes gracefully
- Lock timeout: [X] minutes
```

### Example Usage

**Input:**
```
Generate a Roblox DataStore script with session locking to prevent data duplication exploits.

Requirements:
- Lock data when player joins
- Release lock when player leaves
- Handle server crashes gracefully
- Lock timeout: 10 minutes
```

**Expected Output:**
```lua
-- SessionLockedDataStore.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local dataStore = DataStoreService:GetDataStore("PlayerData_v1")
local sessionStore = DataStoreService:GetDataStore("SessionLocks_v1")

local LOCK_TIMEOUT = 600 -- 10 minutes
local SERVER_ID = HttpService:GenerateGUID(false)

local playerDataCache = {}
local activeSessions = {}

-- Check if lock is valid
local function isLockValid(lockData)
    if not lockData then return false end
    
    local currentTime = os.time()
    local lockTime = lockData.timestamp or 0
    
    return (currentTime - lockTime) < LOCK_TIMEOUT
end

-- Acquire session lock
local function acquireLock(userId)
    local key = "Lock_" .. userId
    
    local success, result = pcall(function()
        return sessionStore:UpdateAsync(key, function(oldData)
            if oldData and isLockValid(oldData) and oldData.serverId ~= SERVER_ID then
                -- Lock held by another server
                return nil
            end
            
            -- Acquire or refresh lock
            return {
                serverId = SERVER_ID,
                timestamp = os.time()
            }
        end)
    end)
    
    if success and result then
        return true
    end
    return false
end

-- Release session lock
local function releaseLock(userId)
    local key = "Lock_" .. userId
    
    pcall(function()
        sessionStore:UpdateAsync(key, function(oldData)
            if oldData and oldData.serverId == SERVER_ID then
                return nil -- Remove lock
            end
            return oldData
        end)
    end)
end

-- Refresh lock periodically
local function startLockRefresh(userId)
    activeSessions[userId] = true
    
    spawn(function()
        while activeSessions[userId] do
            wait(LOCK_TIMEOUT / 2)
            if activeSessions[userId] then
                acquireLock(userId)
            end
        end
    end)
end

-- Player join handler
local function onPlayerAdded(player)
    local userId = player.UserId
    
    -- Try to acquire lock
    local lockAcquired = false
    for attempt = 1, 5 do
        if acquireLock(userId) then
            lockAcquired = true
            break
        end
        warn("[SessionLock] Waiting for lock release for " .. player.Name .. " (attempt " .. attempt .. ")")
        wait(2)
    end
    
    if not lockAcquired then
        player:Kick("Your data is being used on another server. Please wait a moment and rejoin.")
        return
    end
    
    -- Start lock refresh
    startLockRefresh(userId)
    
    -- Load data
    local success, data = pcall(function()
        return dataStore:GetAsync("Player_" .. userId)
    end)
    
    if success then
        playerDataCache[userId] = data or { Coins = 0, Level = 1 }
        print("[SessionLock] Data loaded for " .. player.Name)
    else
        player:Kick("Failed to load your data. Please rejoin.")
        activeSessions[userId] = nil
        releaseLock(userId)
    end
end

-- Player leave handler
local function onPlayerRemoving(player)
    local userId = player.UserId
    
    -- Stop lock refresh
    activeSessions[userId] = nil
    
    -- Save data
    local data = playerDataCache[userId]
    if data then
        pcall(function()
            dataStore:SetAsync("Player_" .. userId, data)
        end)
        playerDataCache[userId] = nil
    end
    
    -- Release lock
    releaseLock(userId)
    print("[SessionLock] Session ended for " .. player.Name)
end

Players.PlayerAdded:Connect(onPlayerAdded)
Players.PlayerRemoving:Connect(onPlayerRemoving)

-- Release all locks on server shutdown
game:BindToClose(function()
    for userId, _ in pairs(activeSessions) do
        activeSessions[userId] = nil
        
        local data = playerDataCache[userId]
        if data then
            pcall(function()
                dataStore:SetAsync("Player_" .. userId, data)
            end)
        end
        
        releaseLock(userId)
    end
    
    wait(3)
end)
```

---

## 📊 Ordered DataStore (Leaderboards)

### Prompt Template

```
Generate a Roblox OrderedDataStore script for a global leaderboard.

Requirements:
- Track: [stat name]
- Display top [X] players
- Update frequency: [interval]
- Include GUI display code
```

### Example Usage

**Input:**
```
Generate a Roblox OrderedDataStore script for a global leaderboard.

Requirements:
- Track: Coins
- Display top 10 players
- Update frequency: every 2 minutes
- Include GUI display code
```

**Expected Output:**
```lua
-- GlobalLeaderboard.lua
-- Place in ServerScriptService

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local coinsLeaderboard = DataStoreService:GetOrderedDataStore("CoinsLeaderboard")

local UPDATE_INTERVAL = 120 -- 2 minutes
local TOP_COUNT = 10

-- Update player's leaderboard entry
local function updatePlayerEntry(player)
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local coins = leaderstats:FindFirstChild("Coins")
        if coins then
            pcall(function()
                coinsLeaderboard:SetAsync(tostring(player.UserId), coins.Value)
            end)
        end
    end
end

-- Get top players
local function getTopPlayers()
    local success, pages = pcall(function()
        return coinsLeaderboard:GetSortedAsync(false, TOP_COUNT)
    end)
    
    if success then
        local topPlayers = {}
        local data = pages:GetCurrentPage()
        
        for rank, entry in ipairs(data) do
            local userId = tonumber(entry.key)
            local coins = entry.value
            
            -- Get player name
            local playerName = "Unknown"
            local nameSuccess, name = pcall(function()
                return Players:GetNameFromUserIdAsync(userId)
            end)
            
            if nameSuccess then
                playerName = name
            end
            
            table.insert(topPlayers, {
                Rank = rank,
                Name = playerName,
                UserId = userId,
                Coins = coins
            })
        end
        
        return topPlayers
    end
    
    return {}
end

-- Update leaderboard GUI
local function updateLeaderboardGUI()
    local topPlayers = getTopPlayers()
    
    -- Fire to all clients to update their GUIs
    local updateEvent = game:GetService("ReplicatedStorage"):FindFirstChild("LeaderboardUpdate")
    if updateEvent then
        updateEvent:FireAllClients(topPlayers)
    end
end

-- Main update loop
spawn(function()
    while true do
        -- Update all player entries
        for _, player in pairs(Players:GetPlayers()) do
            updatePlayerEntry(player)
        end
        
        -- Update GUI
        updateLeaderboardGUI()
        
        wait(UPDATE_INTERVAL)
    end
end)

-- Update on player coin change (optional real-time updates)
Players.PlayerAdded:Connect(function(player)
    player:WaitForChild("leaderstats"):WaitForChild("Coins").Changed:Connect(function()
        updatePlayerEntry(player)
    end)
end)
```

**Client-side GUI Script:**
```lua
-- LeaderboardGUI.lua
-- Place in StarterPlayerScripts

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create leaderboard GUI
local function createLeaderboardGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "LeaderboardGUI"
    screenGui.Parent = playerGui
    
    local frame = Instance.new("Frame")
    frame.Name = "LeaderboardFrame"
    frame.Size = UDim2.new(0, 250, 0, 350)
    frame.Position = UDim2.new(1, -260, 0, 10)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.BorderSizePixel = 0
    frame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.Text = "🏆 Top Players"
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextSize = 20
    title.Font = Enum.Font.GothamBold
    title.Parent = frame
    
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Name = "ListFrame"
    listFrame.Size = UDim2.new(1, -20, 1, -50)
    listFrame.Position = UDim2.new(0, 10, 0, 45)
    listFrame.BackgroundTransparency = 1
    listFrame.ScrollBarThickness = 4
    listFrame.Parent = frame
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 5)
    listLayout.Parent = listFrame
    
    return listFrame
end

local listFrame = createLeaderboardGUI()

-- Update leaderboard display
local function updateDisplay(topPlayers)
    -- Clear existing entries
    for _, child in pairs(listFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    -- Add new entries
    for _, playerData in ipairs(topPlayers) do
        local entry = Instance.new("Frame")
        entry.Size = UDim2.new(1, 0, 0, 30)
        entry.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        entry.BorderSizePixel = 0
        entry.Parent = listFrame
        
        local entryCorner = Instance.new("UICorner")
        entryCorner.CornerRadius = UDim.new(0, 5)
        entryCorner.Parent = entry
        
        local rankLabel = Instance.new("TextLabel")
        rankLabel.Size = UDim2.new(0, 30, 1, 0)
        rankLabel.BackgroundTransparency = 1
        rankLabel.Text = "#" .. playerData.Rank
        rankLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        rankLabel.TextSize = 14
        rankLabel.Font = Enum.Font.GothamBold
        rankLabel.Parent = entry
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(0.5, -35, 1, 0)
        nameLabel.Position = UDim2.new(0, 35, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = playerData.Name
        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLabel.TextSize = 12
        nameLabel.Font = Enum.Font.Gotham
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Parent = entry
        
        local coinsLabel = Instance.new("TextLabel")
        coinsLabel.Size = UDim2.new(0.4, 0, 1, 0)
        coinsLabel.Position = UDim2.new(0.6, 0, 0, 0)
        coinsLabel.BackgroundTransparency = 1
        coinsLabel.Text = "💰 " .. tostring(playerData.Coins)
        coinsLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
        coinsLabel.TextSize = 12
        coinsLabel.Font = Enum.Font.Gotham
        coinsLabel.TextXAlignment = Enum.TextXAlignment.Right
        coinsLabel.Parent = entry
    end
end

-- Listen for updates
local updateEvent = ReplicatedStorage:WaitForChild("LeaderboardUpdate")
updateEvent.OnClientEvent:Connect(updateDisplay)
```

---

## 🔄 Tips for DataStore Success

1. **Always use pcall** - DataStore operations can fail
2. **Implement retries** - Network issues happen
3. **Version your data stores** - Use `_v1`, `_v2` suffixes for migrations
4. **Test in Studio** - Use Studio Access to API setting
5. **Don't save too frequently** - Respect rate limits (60 + 10*playerCount per minute)
6. **Use UpdateAsync for transactions** - Prevents race conditions
7. **Bind to game close** - Ensure data saves on server shutdown

---

## 🎯 Quick Reference Prompts

### Simple Key-Value Store
```
Generate a simple Roblox DataStore wrapper that saves/loads a single value for a key with error handling.
```

### Data Migration
```
Generate a Roblox script that migrates player data from DataStore version 1 to version 2, adding new fields [list fields] while preserving existing data.
```

### Backup System
```
Generate a Roblox DataStore backup system that saves player data to multiple keys with timestamps, keeping the last 3 backups.
```
