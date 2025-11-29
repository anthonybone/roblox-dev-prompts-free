# Lua Script Generation Prompts

Generate Roblox Lua scripts from natural language descriptions. Perfect for solo devs who want to quickly prototype game mechanics without writing boilerplate code.

---

## 📝 Basic Script Generation

### Prompt Template

```
Generate a Roblox Lua script that [describe what you want the script to do].

Requirements:
- [List specific requirements]
- [Performance considerations]
- [Any specific Roblox services to use]

The script should be placed in [ServerScriptService/StarterPlayerScripts/etc].
```

### Example Usage

**Input:**
```
Generate a Roblox Lua script that creates a coin collection system.

Requirements:
- Coins should be parts in the workspace with the name "Coin"
- When a player touches a coin, it should disappear and add 10 points to their score
- Use a leaderboard to display the score
- Include debounce to prevent multiple collections

The script should be placed in ServerScriptService.
```

**Expected Output:**
```lua
-- CoinCollectionSystem.lua
-- Place in ServerScriptService

local Players = game:GetService("Players")

-- Create leaderboard for new players
local function setupLeaderboard(player)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local coins = Instance.new("IntValue")
    coins.Name = "Coins"
    coins.Value = 0
    coins.Parent = leaderstats
end

Players.PlayerAdded:Connect(setupLeaderboard)

-- Setup coin collection
local function setupCoin(coin)
    local debounce = false
    
    coin.Touched:Connect(function(hit)
        if debounce then return end
        
        local player = Players:GetPlayerFromCharacter(hit.Parent)
        if player then
            debounce = true
            
            local leaderstats = player:FindFirstChild("leaderstats")
            if leaderstats then
                local coins = leaderstats:FindFirstChild("Coins")
                if coins then
                    coins.Value = coins.Value + 10
                end
            end
            
            coin:Destroy()
        end
    end)
end

-- Find all existing coins
for _, coin in pairs(workspace:GetDescendants()) do
    if coin.Name == "Coin" and coin:IsA("BasePart") then
        setupCoin(coin)
    end
end

-- Handle coins added later
workspace.DescendantAdded:Connect(function(descendant)
    if descendant.Name == "Coin" and descendant:IsA("BasePart") then
        setupCoin(descendant)
    end
end)
```

---

## 🎮 Player Movement Script

### Prompt Template

```
Generate a Roblox Lua script for custom player movement with [movement type].

Include:
- [Movement features]
- [Input handling]
- [Animation considerations]
```

### Example Usage

**Input:**
```
Generate a Roblox Lua script for custom player movement with double jump.

Include:
- Double jump that can only be used once per ground touch
- Visual effect when double jumping
- Sound effect placeholder
```

**Expected Output:**
```lua
-- DoubleJump.lua
-- Place in StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local canDoubleJump = false
local hasDoubleJumped = false
local DOUBLE_JUMP_POWER = 50

-- Reset double jump when landing
humanoid.StateChanged:Connect(function(oldState, newState)
    if newState == Enum.HumanoidStateType.Landed then
        canDoubleJump = false
        hasDoubleJumped = false
    elseif newState == Enum.HumanoidStateType.Jumping then
        canDoubleJump = true
    elseif newState == Enum.HumanoidStateType.Freefall then
        canDoubleJump = true
    end
end)

-- Handle double jump input
UserInputService.JumpRequest:Connect(function()
    if canDoubleJump and not hasDoubleJumped then
        hasDoubleJumped = true
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            rootPart.Velocity = Vector3.new(
                rootPart.Velocity.X,
                DOUBLE_JUMP_POWER,
                rootPart.Velocity.Z
            )
            
            -- Visual effect
            local effect = Instance.new("Part")
            effect.Size = Vector3.new(2, 0.5, 2)
            effect.Anchored = true
            effect.CanCollide = false
            effect.Transparency = 0.5
            effect.BrickColor = BrickColor.new("Bright blue")
            effect.Position = rootPart.Position - Vector3.new(0, 3, 0)
            effect.Parent = workspace
            
            game:GetService("Debris"):AddItem(effect, 0.5)
            
            -- Sound placeholder
            -- local sound = Instance.new("Sound")
            -- sound.SoundId = "rbxassetid://YOUR_SOUND_ID"
            -- sound.Parent = rootPart
            -- sound:Play()
        end
    end
end)

-- Handle character respawn
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    hasDoubleJumped = false
    canDoubleJump = false
end)
```

---

## ⚔️ Combat System Script

### Prompt Template

```
Generate a Roblox combat system script with [combat type].

Features needed:
- [Attack mechanics]
- [Damage system]
- [Cooldowns]
- [Hitbox detection method]
```

### Example Usage

**Input:**
```
Generate a Roblox combat system script with melee sword attacks.

Features needed:
- Left click to swing sword
- 1 second cooldown between swings
- Hitbox detection using region3 or touched events
- Deal 25 damage to other players
```

**Expected Output:**
```lua
-- SwordCombat.lua (Tool Script)
-- Place inside a Tool in StarterPack

local tool = script.Parent
local player = game:GetService("Players").LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local COOLDOWN = 1
local DAMAGE = 25
local ATTACK_RANGE = 5

local canAttack = true
local remoteEvent = game:GetService("ReplicatedStorage"):WaitForChild("SwordDamageEvent")

tool.Activated:Connect(function()
    if not canAttack then return end
    canAttack = false
    
    -- Play animation (placeholder)
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        -- local anim = humanoid:LoadAnimation(swingAnimation)
        -- anim:Play()
    end
    
    -- Detect hits
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if rootPart then
        local hitPlayers = {}
        
        for _, otherPlayer in pairs(game:GetService("Players"):GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local otherRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if otherRoot then
                    local distance = (rootPart.Position - otherRoot.Position).Magnitude
                    if distance <= ATTACK_RANGE then
                        table.insert(hitPlayers, otherPlayer)
                    end
                end
            end
        end
        
        -- Send damage to server
        for _, hitPlayer in pairs(hitPlayers) do
            remoteEvent:FireServer(hitPlayer, DAMAGE)
        end
    end
    
    -- Cooldown
    wait(COOLDOWN)
    canAttack = true
end)

-- Handle character respawn
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
end)
```

---

## 🔄 Tips for Better Results

1. **Be specific about script location** - ServerScriptService, StarterPlayerScripts, etc.
2. **Mention required services** - DataStoreService, ReplicatedStorage, etc.
3. **Specify security requirements** - Server-side validation, RemoteEvents vs RemoteFunctions
4. **Include performance needs** - Debounce, rate limiting, optimization
5. **Describe edge cases** - Player disconnection, character respawn, etc.

---

## 🎯 Quick Reference Prompts

### Generic Game Loop
```
Generate a Roblox game loop script that runs a [game type] round system with [X] minute rounds, intermission periods, and player teleportation between maps.
```

### Tool Creation
```
Generate a Roblox tool script for a [tool type] that [functionality]. Include activation, deactivation, and equipped/unequipped handling.
```

### Event System
```
Generate a Roblox event system using BindableEvents that allows [describe communication pattern] between scripts.
```

### Admin Commands
```
Generate a Roblox admin command system that recognizes chat commands starting with "/" and includes commands for [list commands like kick, ban, teleport].
```
