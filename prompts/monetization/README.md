# Monetization Workflow Prompts

Create effective monetization systems for your Roblox games. These prompts help you generate game passes, developer products, premium features, and ethical reward systems.

---

## 💎 Game Pass System

### Prompt Template

```
Generate a Roblox game pass system for [pass type].

Requirements:
- Pass benefits: [list of benefits]
- Verification method: [server-side check]
- UI integration: [how to show pass status]
- Include: [purchase prompts, owned indicators]
```

### Example Usage

**Input:**
```
Generate a Roblox game pass system for VIP membership.

Requirements:
- Pass benefits: 2x coins, exclusive areas, special chat tag, VIP server access
- Verification method: server-side ownership check
- UI integration: VIP badge next to player name
- Include: purchase prompt button, benefits display
```

**Expected Output:**

**Server Script (GamePassManager.lua):**
```lua
-- GamePassManager.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Game Pass IDs (replace with your actual IDs)
local GAME_PASSES = {
    VIP = 123456789,  -- Replace with real ID
    DoubleCoins = 123456790,
    ExtraInventory = 123456791
}

-- Create RemoteEvents for client communication
local passEvents = Instance.new("Folder")
passEvents.Name = "GamePassEvents"
passEvents.Parent = ReplicatedStorage

local checkPassEvent = Instance.new("RemoteFunction")
checkPassEvent.Name = "CheckGamePass"
checkPassEvent.Parent = passEvents

local passOwnedEvent = Instance.new("RemoteEvent")
passOwnedEvent.Name = "PassOwned"
passOwnedEvent.Parent = passEvents

-- Cache for pass ownership (reduces API calls)
local passCache = {}

-- Check if player owns a game pass
local function playerOwnsPass(player, passName)
    local passId = GAME_PASSES[passName]
    if not passId then return false end
    
    local cacheKey = player.UserId .. "_" .. passName
    
    -- Check cache first
    if passCache[cacheKey] ~= nil then
        return passCache[cacheKey]
    end
    
    -- Query MarketplaceService
    local success, ownsPass = pcall(function()
        return MarketplaceService:UserOwnsGamePassAsync(player.UserId, passId)
    end)
    
    if success then
        passCache[cacheKey] = ownsPass
        return ownsPass
    end
    
    return false
end

-- Apply VIP benefits
local function applyVIPBenefits(player)
    -- Add VIP tag
    local vipTag = Instance.new("BoolValue")
    vipTag.Name = "IsVIP"
    vipTag.Value = true
    vipTag.Parent = player
    
    -- Give access to VIP area
    local vipAccess = Instance.new("BoolValue")
    vipAccess.Name = "VIPAccess"
    vipAccess.Value = true
    vipAccess.Parent = player
    
    -- Set coin multiplier
    local coinMultiplier = Instance.new("NumberValue")
    coinMultiplier.Name = "CoinMultiplier"
    coinMultiplier.Value = 2
    coinMultiplier.Parent = player
    
    print("[GamePass] VIP benefits applied to " .. player.Name)
end

-- Handle new players
local function onPlayerAdded(player)
    passCache[player.UserId] = passCache[player.UserId] or {}
    
    -- Check all passes
    for passName, passId in pairs(GAME_PASSES) do
        if playerOwnsPass(player, passName) then
            passOwnedEvent:FireClient(player, passName)
            
            -- Apply specific benefits
            if passName == "VIP" then
                applyVIPBenefits(player)
            end
        end
    end
end

-- Handle pass purchased during gameplay
MarketplaceService.PromptGamePassPurchaseFinished:Connect(function(player, passId, wasPurchased)
    if not wasPurchased then return end
    
    for passName, id in pairs(GAME_PASSES) do
        if id == passId then
            -- Update cache
            local cacheKey = player.UserId .. "_" .. passName
            passCache[cacheKey] = true
            
            -- Notify client
            passOwnedEvent:FireClient(player, passName)
            
            -- Apply benefits
            if passName == "VIP" then
                applyVIPBenefits(player)
            end
            
            print("[GamePass] " .. player.Name .. " purchased " .. passName)
            break
        end
    end
end)

-- Remote function to check pass ownership
checkPassEvent.OnServerInvoke = function(player, passName)
    return playerOwnsPass(player, passName)
end

-- Cleanup on leave
local function onPlayerRemoving(player)
    for passName, _ in pairs(GAME_PASSES) do
        local cacheKey = player.UserId .. "_" .. passName
        passCache[cacheKey] = nil
    end
end

Players.PlayerAdded:Connect(onPlayerAdded)
Players.PlayerRemoving:Connect(onPlayerRemoving)

-- Handle existing players
for _, player in pairs(Players:GetPlayers()) do
    coroutine.wrap(onPlayerAdded)(player)
end

-- Module API
local GamePassManager = {}

function GamePassManager:PlayerHasPass(player, passName)
    return playerOwnsPass(player, passName)
end

function GamePassManager:GetPassId(passName)
    return GAME_PASSES[passName]
end

function GamePassManager:PromptPurchase(player, passName)
    local passId = GAME_PASSES[passName]
    if passId then
        MarketplaceService:PromptGamePassPurchase(player, passId)
    end
end

return GamePassManager
```

**Client Script (GamePassUI.lua):**
```lua
-- GamePassUI.lua
-- Place in StarterPlayerScripts

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local passEvents = ReplicatedStorage:WaitForChild("GamePassEvents")
local checkPass = passEvents:WaitForChild("CheckGamePass")
local passOwned = passEvents:WaitForChild("PassOwned")

-- VIP Pass ID (replace with your actual ID)
local VIP_PASS_ID = 123456789

-- Create VIP purchase UI
local function createVIPPromptUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "VIPPrompt"
    screenGui.Parent = playerGui
    
    local promptFrame = Instance.new("Frame")
    promptFrame.Name = "PromptFrame"
    promptFrame.Size = UDim2.new(0, 350, 0, 400)
    promptFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
    promptFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    promptFrame.Visible = false
    promptFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = promptFrame
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 215, 0)
    stroke.Thickness = 3
    stroke.Parent = promptFrame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(1, 0, 0, 50)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "⭐ VIP MEMBERSHIP ⭐"
    titleLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Parent = promptFrame
    
    local benefitsLabel = Instance.new("TextLabel")
    benefitsLabel.Size = UDim2.new(1, -40, 0, 200)
    benefitsLabel.Position = UDim2.new(0, 20, 0, 60)
    benefitsLabel.BackgroundTransparency = 1
    benefitsLabel.Text = "✓ 2x Coin Earnings\n\n✓ Exclusive VIP Areas\n\n✓ Special Chat Tag\n\n✓ VIP Server Access\n\n✓ Priority Support"
    benefitsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    benefitsLabel.TextSize = 18
    benefitsLabel.Font = Enum.Font.Gotham
    benefitsLabel.TextXAlignment = Enum.TextXAlignment.Left
    benefitsLabel.TextYAlignment = Enum.TextYAlignment.Top
    benefitsLabel.Parent = promptFrame
    
    local buyButton = Instance.new("TextButton")
    buyButton.Name = "BuyButton"
    buyButton.Size = UDim2.new(0.8, 0, 0, 50)
    buyButton.Position = UDim2.new(0.1, 0, 1, -120)
    buyButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    buyButton.Text = "💎 Purchase VIP"
    buyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyButton.TextSize = 20
    buyButton.Font = Enum.Font.GothamBold
    buyButton.Parent = promptFrame
    
    local buyCorner = Instance.new("UICorner")
    buyCorner.CornerRadius = UDim.new(0, 10)
    buyCorner.Parent = buyButton
    
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0.8, 0, 0, 40)
    closeButton.Position = UDim2.new(0.1, 0, 1, -60)
    closeButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    closeButton.Text = "Maybe Later"
    closeButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    closeButton.TextSize = 16
    closeButton.Font = Enum.Font.Gotham
    closeButton.Parent = promptFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeButton
    
    -- Button handlers
    buyButton.MouseButton1Click:Connect(function()
        MarketplaceService:PromptGamePassPurchase(player, VIP_PASS_ID)
    end)
    
    closeButton.MouseButton1Click:Connect(function()
        promptFrame.Visible = false
    end)
    
    return promptFrame
end

local vipPrompt = createVIPPromptUI()

-- Create VIP badge for player list
local function createVIPBadge()
    local badge = Instance.new("TextLabel")
    badge.Name = "VIPBadge"
    badge.Size = UDim2.new(0, 50, 0, 20)
    badge.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
    badge.Text = "VIP"
    badge.TextColor3 = Color3.fromRGB(0, 0, 0)
    badge.TextSize = 12
    badge.Font = Enum.Font.GothamBold
    
    local badgeCorner = Instance.new("UICorner")
    badgeCorner.CornerRadius = UDim.new(0, 5)
    badgeCorner.Parent = badge
    
    return badge
end

-- Handle pass owned notification
passOwned.OnClientEvent:Connect(function(passName)
    if passName == "VIP" then
        vipPrompt.Visible = false
        
        -- Show thank you message
        local thankYou = Instance.new("TextLabel")
        thankYou.Size = UDim2.new(0, 300, 0, 50)
        thankYou.Position = UDim2.new(0.5, -150, 0.3, 0)
        thankYou.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        thankYou.Text = "🎉 Thank you for VIP!"
        thankYou.TextColor3 = Color3.fromRGB(255, 255, 255)
        thankYou.TextSize = 20
        thankYou.Font = Enum.Font.GothamBold
        thankYou.Parent = playerGui
        
        local thankCorner = Instance.new("UICorner")
        thankCorner.CornerRadius = UDim.new(0, 10)
        thankCorner.Parent = thankYou
        
        wait(3)
        thankYou:Destroy()
    end
end)

-- Function to show VIP prompt
local function showVIPPrompt()
    local hasVIP = checkPass:InvokeServer("VIP")
    if not hasVIP then
        vipPrompt.Visible = true
    end
end
```

---

## 💰 Developer Products (Consumables)

### Prompt Template

```
Generate a Roblox developer product system for [product type].

Requirements:
- Products: [list of products and prices]
- Purchase handling: [what happens when bought]
- Receipt processing: [server-side validation]
- Include: [purchase history, refund handling]
```

### Example Usage

**Input:**
```
Generate a Roblox developer product system for in-game currency.

Requirements:
- Products: 100 coins ($0.99), 500 coins ($3.99), 1000 coins ($6.99)
- Purchase handling: add coins to player balance
- Receipt processing: with DataStore logging
- Include: purchase confirmation, bonus for larger packs
```

**Expected Output:**
```lua
-- DeveloperProductManager.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local DataStoreService = game:GetService("DataStoreService")

local purchaseHistoryStore = DataStoreService:GetDataStore("PurchaseHistory")

-- Product configuration (replace IDs with real ones)
local PRODUCTS = {
    [111111111] = { name = "100 Coins", coins = 100, bonus = 0 },
    [222222222] = { name = "500 Coins", coins = 500, bonus = 50 },   -- 10% bonus
    [333333333] = { name = "1000 Coins", coins = 1000, bonus = 200 } -- 20% bonus
}

-- Grant coins to player
local function grantCoins(player, amount)
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local coins = leaderstats:FindFirstChild("Coins")
        if coins then
            coins.Value = coins.Value + amount
            return true
        end
    end
    return false
end

-- Log purchase to DataStore
local function logPurchase(player, productId, receiptId)
    local key = "Receipts_" .. player.UserId
    
    pcall(function()
        purchaseHistoryStore:UpdateAsync(key, function(oldData)
            oldData = oldData or {}
            table.insert(oldData, {
                productId = productId,
                receiptId = receiptId,
                timestamp = os.time()
            })
            -- Keep only last 100 receipts
            while #oldData > 100 do
                table.remove(oldData, 1)
            end
            return oldData
        end)
    end)
end

-- Check if receipt was already processed
local function isReceiptProcessed(player, receiptId)
    local key = "Receipts_" .. player.UserId
    
    local success, data = pcall(function()
        return purchaseHistoryStore:GetAsync(key)
    end)
    
    if success and data then
        for _, receipt in pairs(data) do
            if receipt.receiptId == receiptId then
                return true
            end
        end
    end
    
    return false
end

-- Process receipt callback
local function processReceipt(receiptInfo)
    local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
    
    if not player then
        -- Player left, will retry later
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
    
    -- Check if already processed
    if isReceiptProcessed(player, receiptInfo.PurchaseId) then
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    local productInfo = PRODUCTS[receiptInfo.ProductId]
    
    if not productInfo then
        warn("[Products] Unknown product: " .. receiptInfo.ProductId)
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
    
    -- Calculate total coins (base + bonus)
    local totalCoins = productInfo.coins + productInfo.bonus
    
    -- Grant coins
    if grantCoins(player, totalCoins) then
        -- Log the purchase
        logPurchase(player, receiptInfo.ProductId, receiptInfo.PurchaseId)
        
        print(string.format(
            "[Products] %s purchased %s (%d + %d bonus coins)",
            player.Name,
            productInfo.name,
            productInfo.coins,
            productInfo.bonus
        ))
        
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    return Enum.ProductPurchaseDecision.NotProcessedYet
end

-- Set the callback
MarketplaceService.ProcessReceipt = processReceipt

-- Module API
local ProductManager = {}

function ProductManager:PromptPurchase(player, productId)
    MarketplaceService:PromptProductPurchase(player, productId)
end

function ProductManager:GetProductInfo(productId)
    return PRODUCTS[productId]
end

return ProductManager
```

---

## 🎁 Daily Rewards System

### Prompt Template

```
Generate a Roblox daily rewards system with [reward structure].

Requirements:
- Reward progression: [describe daily rewards]
- Streak bonus: [how streaks work]
- Reset timing: [UTC/local time]
- Include: [UI, streak counter, miss penalty]
```

### Example Usage

**Input:**
```
Generate a Roblox daily rewards system with 7-day cycle.

Requirements:
- Reward progression: Day 1: 50 coins, Day 7: 500 coins + rare item
- Streak bonus: 10% more coins per consecutive day
- Reset timing: UTC midnight
- Include: claim UI, streak display, countdown timer
```

**Expected Output:**
```lua
-- DailyRewardsServer.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local dailyRewardsStore = DataStoreService:GetDataStore("DailyRewards_v1")

-- Reward configuration
local REWARDS = {
    [1] = { coins = 50, item = nil },
    [2] = { coins = 75, item = nil },
    [3] = { coins = 100, item = nil },
    [4] = { coins = 150, item = nil },
    [5] = { coins = 200, item = nil },
    [6] = { coins = 300, item = nil },
    [7] = { coins = 500, item = "RareChest" }
}

local STREAK_BONUS = 0.10 -- 10% per consecutive day
local SECONDS_IN_DAY = 86400

-- Create RemoteEvents
local rewardsFolder = Instance.new("Folder")
rewardsFolder.Name = "DailyRewardsEvents"
rewardsFolder.Parent = ReplicatedStorage

local claimReward = Instance.new("RemoteFunction")
claimReward.Name = "ClaimReward"
claimReward.Parent = rewardsFolder

local getRewardStatus = Instance.new("RemoteFunction")
getRewardStatus.Name = "GetRewardStatus"
getRewardStatus.Parent = rewardsFolder

-- Get current UTC day number
local function getCurrentDay()
    return math.floor(os.time() / SECONDS_IN_DAY)
end

-- Get player reward data
local function getPlayerData(player)
    local key = "Daily_" .. player.UserId
    
    local success, data = pcall(function()
        return dailyRewardsStore:GetAsync(key)
    end)
    
    if success and data then
        return data
    end
    
    return {
        lastClaimDay = 0,
        currentStreak = 0,
        rewardDay = 1
    }
end

-- Save player reward data
local function savePlayerData(player, data)
    local key = "Daily_" .. player.UserId
    
    pcall(function()
        dailyRewardsStore:SetAsync(key, data)
    end)
end

-- Grant reward to player
local function grantReward(player, day, streak)
    local reward = REWARDS[day]
    if not reward then return false end
    
    -- Calculate coins with streak bonus
    local streakMultiplier = 1 + (streak * STREAK_BONUS)
    local totalCoins = math.floor(reward.coins * streakMultiplier)
    
    -- Grant coins
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local coins = leaderstats:FindFirstChild("Coins")
        if coins then
            coins.Value = coins.Value + totalCoins
        end
    end
    
    -- Grant item if applicable
    if reward.item then
        -- Add item to player inventory
        print("[DailyRewards] " .. player.Name .. " received " .. reward.item)
    end
    
    return true, totalCoins
end

-- Claim reward handler
claimReward.OnServerInvoke = function(player)
    local data = getPlayerData(player)
    local currentDay = getCurrentDay()
    
    -- Check if already claimed today
    if data.lastClaimDay == currentDay then
        return {
            success = false,
            message = "Already claimed today!"
        }
    end
    
    -- Check streak
    local daysSinceLastClaim = currentDay - data.lastClaimDay
    
    if daysSinceLastClaim == 1 then
        -- Consecutive day, increase streak
        data.currentStreak = data.currentStreak + 1
    elseif daysSinceLastClaim > 1 then
        -- Streak broken, reset
        data.currentStreak = 0
        data.rewardDay = 1
    end
    
    -- Grant reward
    local success, coinsGranted = grantReward(player, data.rewardDay, data.currentStreak)
    
    if success then
        -- Update data
        data.lastClaimDay = currentDay
        data.rewardDay = data.rewardDay + 1
        if data.rewardDay > 7 then
            data.rewardDay = 1
        end
        
        savePlayerData(player, data)
        
        return {
            success = true,
            coinsGranted = coinsGranted,
            newStreak = data.currentStreak,
            nextRewardDay = data.rewardDay
        }
    end
    
    return {
        success = false,
        message = "Failed to grant reward"
    }
end

-- Get reward status handler
getRewardStatus.OnServerInvoke = function(player)
    local data = getPlayerData(player)
    local currentDay = getCurrentDay()
    
    local canClaim = data.lastClaimDay ~= currentDay
    local secondsUntilReset = SECONDS_IN_DAY - (os.time() % SECONDS_IN_DAY)
    
    return {
        canClaim = canClaim,
        currentStreak = data.currentStreak,
        rewardDay = data.rewardDay,
        secondsUntilReset = secondsUntilReset,
        rewards = REWARDS
    }
end
```

---

## 🔄 Tips for Ethical Monetization

1. **Be transparent** - Clearly show what players get for their money
2. **Avoid pay-to-win** - Cosmetics and convenience, not power
3. **Respect time** - Don't make free players feel punished
4. **Test purchases** - Use test products during development
5. **Handle failures gracefully** - Retry logic for network issues
6. **Log everything** - Track purchases for support requests
7. **Follow Roblox guidelines** - Stay compliant with ToS

---

## 🎯 Quick Reference Prompts

### Premium Subscription Benefits
```
Generate a Roblox system that checks if a player has Roblox Premium and grants them bonus rewards (extra coins, exclusive items).
```

### Limited Time Offers
```
Generate a Roblox limited-time offer system that shows special deals with countdown timers and creates urgency without being manipulative.
```

### Referral System
```
Generate a Roblox referral system where players can invite friends and both receive rewards when the friend reaches a certain level.
```

### Battle Pass
```
Generate a Roblox battle pass system with free and premium tiers, XP-based progression, and seasonal resets.
```
