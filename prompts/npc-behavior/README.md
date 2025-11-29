# NPC Behavior Prompts

Create intelligent NPCs with pathfinding, dialogue systems, and dynamic behaviors. These prompts help you generate NPCs that feel alive and responsive.

---

## 🤖 Basic NPC Pathfinding

### Prompt Template

```
Generate a Roblox NPC script with pathfinding that [describe behavior].

Requirements:
- Movement type: [patrol/follow/flee/wander]
- Speed: [walkspeed value]
- Pathfinding settings: [AgentRadius, AgentHeight, etc.]
- Behavior triggers: [what causes behavior changes]
```

### Example Usage

**Input:**
```
Generate a Roblox NPC script with pathfinding that patrols between waypoints.

Requirements:
- Movement type: patrol between points in a loop
- Speed: 16 walkspeed
- Pathfinding settings: default humanoid size
- Behavior triggers: chase players within 30 studs, return to patrol when player escapes
```

**Expected Output:**
```lua
-- PatrolNPC.lua
-- Place in the NPC model (NPC should have Humanoid and HumanoidRootPart)

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")

local npc = script.Parent
local humanoid = npc:WaitForChild("Humanoid")
local rootPart = npc:WaitForChild("HumanoidRootPart")

-- Configuration
local WALK_SPEED = 16
local CHASE_SPEED = 24
local DETECTION_RANGE = 30
local LOSE_RANGE = 50
local WAYPOINT_REACHED_DISTANCE = 3

-- Get waypoints (children of a folder named "Waypoints" in workspace)
local waypointsFolder = workspace:FindFirstChild("Waypoints")
local waypoints = {}

if waypointsFolder then
    for _, waypoint in pairs(waypointsFolder:GetChildren()) do
        if waypoint:IsA("BasePart") then
            table.insert(waypoints, waypoint)
        end
    end
    -- Sort by name for ordered patrol
    table.sort(waypoints, function(a, b)
        return a.Name < b.Name
    end)
end

local currentWaypointIndex = 1
local currentTarget = nil
local isChasing = false

-- Pathfinding setup
local pathParams = {
    AgentRadius = 2,
    AgentHeight = 5,
    AgentCanJump = true,
    AgentCanClimb = false,
    WaypointSpacing = 4
}

-- Create path to target position
local function createPath(targetPosition)
    local path = PathfindingService:CreatePath(pathParams)
    
    local success, errorMessage = pcall(function()
        path:ComputeAsync(rootPart.Position, targetPosition)
    end)
    
    if success and path.Status == Enum.PathStatus.Success then
        return path
    end
    
    return nil
end

-- Move to position using pathfinding
local function moveToPosition(position)
    local path = createPath(position)
    
    if not path then
        -- Direct movement fallback
        humanoid:MoveTo(position)
        return
    end
    
    local waypoints = path:GetWaypoints()
    
    for _, waypoint in pairs(waypoints) do
        if isChasing then
            -- Recalculate if chasing (target might move)
            break
        end
        
        humanoid:MoveTo(waypoint.Position)
        
        if waypoint.Action == Enum.PathWaypointAction.Jump then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
        
        -- Wait for movement
        local moveFinished = humanoid.MoveToFinished:Wait()
        
        if not moveFinished then
            break
        end
    end
end

-- Find nearest player within range
local function findNearestPlayer()
    local nearestPlayer = nil
    local nearestDistance = DETECTION_RANGE
    
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local playerRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local playerHumanoid = player.Character:FindFirstChild("Humanoid")
            
            if playerRoot and playerHumanoid and playerHumanoid.Health > 0 then
                local distance = (rootPart.Position - playerRoot.Position).Magnitude
                
                if distance < nearestDistance then
                    nearestDistance = distance
                    nearestPlayer = player
                end
            end
        end
    end
    
    return nearestPlayer
end

-- Check if should stop chasing
local function shouldStopChasing()
    if not currentTarget then return true end
    
    local character = currentTarget.Character
    if not character then return true end
    
    local playerRoot = character:FindFirstChild("HumanoidRootPart")
    local playerHumanoid = character:FindFirstChild("Humanoid")
    
    if not playerRoot or not playerHumanoid then return true end
    if playerHumanoid.Health <= 0 then return true end
    
    local distance = (rootPart.Position - playerRoot.Position).Magnitude
    return distance > LOSE_RANGE
end

-- Chase behavior
local function chase()
    isChasing = true
    humanoid.WalkSpeed = CHASE_SPEED
    
    while isChasing and currentTarget do
        local character = currentTarget.Character
        if character then
            local playerRoot = character:FindFirstChild("HumanoidRootPart")
            if playerRoot then
                moveToPosition(playerRoot.Position)
            end
        end
        
        if shouldStopChasing() then
            isChasing = false
            currentTarget = nil
        end
        
        wait(0.1)
    end
    
    humanoid.WalkSpeed = WALK_SPEED
end

-- Patrol behavior
local function patrol()
    if #waypoints == 0 then
        warn("[PatrolNPC] No waypoints found!")
        return
    end
    
    while true do
        -- Check for players to chase
        local nearestPlayer = findNearestPlayer()
        
        if nearestPlayer then
            currentTarget = nearestPlayer
            chase()
        else
            -- Continue patrol
            local targetWaypoint = waypoints[currentWaypointIndex]
            
            if targetWaypoint then
                moveToPosition(targetWaypoint.Position)
                
                -- Move to next waypoint
                currentWaypointIndex = currentWaypointIndex + 1
                if currentWaypointIndex > #waypoints then
                    currentWaypointIndex = 1
                end
            end
        end
        
        wait(0.5)
    end
end

-- Initialize
humanoid.WalkSpeed = WALK_SPEED

-- Start patrol
spawn(patrol)

-- Handle NPC death
humanoid.Died:Connect(function()
    -- Cleanup or respawn logic here
    script:Destroy()
end)
```

---

## 💬 NPC Dialogue System

### Prompt Template

```
Generate a Roblox NPC dialogue system with [dialogue features].

Requirements:
- Dialogue structure: [linear/branching/quest-based]
- Trigger method: [proximity/click/interact key]
- UI style: [chat bubble/full screen/minimal]
- Include: [choices/conditions/rewards]
```

### Example Usage

**Input:**
```
Generate a Roblox NPC dialogue system with branching conversations.

Requirements:
- Dialogue structure: branching with player choices
- Trigger method: click on NPC (ProximityPrompt)
- UI style: chat GUI at bottom of screen
- Include: choices that affect next dialogue, simple quest giving
```

**Expected Output:**

**Server Script (DialogueServer.lua):**
```lua
-- DialogueServer.lua
-- Place in ServerScriptService

local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Create RemoteEvents
local dialogueEvents = Instance.new("Folder")
dialogueEvents.Name = "DialogueEvents"
dialogueEvents.Parent = ReplicatedStorage

local startDialogue = Instance.new("RemoteEvent")
startDialogue.Name = "StartDialogue"
startDialogue.Parent = dialogueEvents

local dialogueChoice = Instance.new("RemoteEvent")
dialogueChoice.Name = "DialogueChoice"
dialogueChoice.Parent = dialogueEvents

local endDialogue = Instance.new("RemoteEvent")
endDialogue.Name = "EndDialogue"
endDialogue.Parent = dialogueEvents

-- Dialogue data storage
local DialogueData = {
    ["Merchant"] = {
        ["start"] = {
            speaker = "Merchant",
            text = "Welcome, traveler! What brings you to my shop?",
            choices = {
                { text = "I'm looking to buy supplies.", next = "buy" },
                { text = "Do you have any work for me?", next = "quest" },
                { text = "Just passing through.", next = "goodbye" }
            }
        },
        ["buy"] = {
            speaker = "Merchant",
            text = "I have potions, weapons, and armor. What interests you?",
            choices = {
                { text = "Show me your potions.", next = "potions" },
                { text = "What weapons do you have?", next = "weapons" },
                { text = "Never mind.", next = "goodbye" }
            }
        },
        ["quest"] = {
            speaker = "Merchant",
            text = "Actually, yes! Goblins have been stealing my shipments. Defeat 5 of them and I'll reward you handsomely!",
            choices = {
                { text = "I'll take care of them!", next = "quest_accept", action = "give_quest" },
                { text = "That sounds dangerous...", next = "quest_decline" }
            }
        },
        ["quest_accept"] = {
            speaker = "Merchant",
            text = "Excellent! Return to me when the job is done. Good luck!",
            choices = {
                { text = "I'll be back.", next = "end" }
            }
        },
        ["quest_decline"] = {
            speaker = "Merchant",
            text = "I understand. The offer stands if you change your mind.",
            choices = {
                { text = "Maybe later.", next = "goodbye" }
            }
        },
        ["potions"] = {
            speaker = "Merchant",
            text = "I have health potions for 50 coins and mana potions for 75 coins.",
            choices = {
                { text = "I'll take a health potion.", next = "buy_health", action = "buy_health_potion" },
                { text = "Back to main options.", next = "buy" }
            }
        },
        ["weapons"] = {
            speaker = "Merchant",
            text = "I have iron swords for 200 coins and steel swords for 500 coins.",
            choices = {
                { text = "Maybe later.", next = "buy" }
            }
        },
        ["buy_health"] = {
            speaker = "Merchant",
            text = "Here you go! One health potion. Use it wisely!",
            choices = {
                { text = "Thanks!", next = "goodbye" }
            }
        },
        ["goodbye"] = {
            speaker = "Merchant",
            text = "Safe travels, friend!",
            choices = {
                { text = "Goodbye.", next = "end" }
            }
        }
    }
}

-- Player quest tracking
local playerQuests = {}

-- Handle dialogue actions
local function handleAction(player, action)
    if action == "give_quest" then
        playerQuests[player.UserId] = playerQuests[player.UserId] or {}
        playerQuests[player.UserId]["goblin_quest"] = {
            progress = 0,
            target = 5,
            completed = false
        }
        print(player.Name .. " accepted the goblin quest!")
        
    elseif action == "buy_health_potion" then
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local coins = leaderstats:FindFirstChild("Coins")
            if coins and coins.Value >= 50 then
                coins.Value = coins.Value - 50
                -- Give potion logic here
                print(player.Name .. " bought a health potion!")
            end
        end
    end
end

-- Handle player choice
dialogueChoice.OnServerEvent:Connect(function(player, npcName, choiceIndex, currentNode)
    local npcDialogue = DialogueData[npcName]
    if not npcDialogue then return end
    
    local node = npcDialogue[currentNode]
    if not node or not node.choices then return end
    
    local choice = node.choices[choiceIndex]
    if not choice then return end
    
    -- Handle action if exists
    if choice.action then
        handleAction(player, choice.action)
    end
    
    -- Send next dialogue or end
    if choice.next == "end" then
        endDialogue:FireClient(player)
    else
        local nextNode = npcDialogue[choice.next]
        if nextNode then
            startDialogue:FireClient(player, npcName, choice.next, nextNode)
        end
    end
end)

-- Module for NPCs to use
local DialogueModule = {}

function DialogueModule.SetupNPC(npc, dialogueName)
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = "Talk"
    prompt.ObjectText = dialogueName
    prompt.MaxActivationDistance = 10
    prompt.Parent = npc:FindFirstChild("HumanoidRootPart") or npc.PrimaryPart or npc
    
    prompt.Triggered:Connect(function(player)
        local dialogue = DialogueData[dialogueName]
        if dialogue and dialogue["start"] then
            startDialogue:FireClient(player, dialogueName, "start", dialogue["start"])
        end
    end)
end

return DialogueModule
```

**Client Script (DialogueClient.lua):**
```lua
-- DialogueClient.lua
-- Place in StarterPlayerScripts

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local dialogueEvents = ReplicatedStorage:WaitForChild("DialogueEvents")
local startDialogue = dialogueEvents:WaitForChild("StartDialogue")
local dialogueChoice = dialogueEvents:WaitForChild("DialogueChoice")
local endDialogue = dialogueEvents:WaitForChild("EndDialogue")

local currentNPC = nil
local currentNode = nil

-- Create dialogue GUI
local function createDialogueGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DialogueGUI"
    screenGui.Enabled = false
    screenGui.Parent = playerGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0.6, 0, 0.25, 0)
    mainFrame.Position = UDim2.new(0.2, 0, 0.7, 0)
    mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    local speakerLabel = Instance.new("TextLabel")
    speakerLabel.Name = "Speaker"
    speakerLabel.Size = UDim2.new(0.3, 0, 0, 30)
    speakerLabel.Position = UDim2.new(0, 15, 0, -30)
    speakerLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    speakerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    speakerLabel.TextSize = 18
    speakerLabel.Font = Enum.Font.GothamBold
    speakerLabel.Parent = mainFrame
    
    local speakerCorner = Instance.new("UICorner")
    speakerCorner.CornerRadius = UDim.new(0, 8)
    speakerCorner.Parent = speakerLabel
    
    local dialogueText = Instance.new("TextLabel")
    dialogueText.Name = "DialogueText"
    dialogueText.Size = UDim2.new(1, -30, 0.5, 0)
    dialogueText.Position = UDim2.new(0, 15, 0, 15)
    dialogueText.BackgroundTransparency = 1
    dialogueText.TextColor3 = Color3.fromRGB(255, 255, 255)
    dialogueText.TextSize = 16
    dialogueText.Font = Enum.Font.Gotham
    dialogueText.TextWrapped = true
    dialogueText.TextXAlignment = Enum.TextXAlignment.Left
    dialogueText.TextYAlignment = Enum.TextYAlignment.Top
    dialogueText.Parent = mainFrame
    
    local choicesFrame = Instance.new("Frame")
    choicesFrame.Name = "Choices"
    choicesFrame.Size = UDim2.new(1, -30, 0.45, 0)
    choicesFrame.Position = UDim2.new(0, 15, 0.5, 5)
    choicesFrame.BackgroundTransparency = 1
    choicesFrame.Parent = mainFrame
    
    local choicesLayout = Instance.new("UIListLayout")
    choicesLayout.Padding = UDim.new(0, 5)
    choicesLayout.Parent = choicesFrame
    
    return screenGui
end

local dialogueGUI = createDialogueGUI()
local mainFrame = dialogueGUI.MainFrame
local speakerLabel = mainFrame.Speaker
local dialogueText = mainFrame.DialogueText
local choicesFrame = mainFrame.Choices

-- Typewriter effect
local function typewriterEffect(label, text, speed)
    label.Text = ""
    for i = 1, #text do
        label.Text = string.sub(text, 1, i)
        wait(speed or 0.03)
    end
end

-- Show dialogue
local function showDialogue(npcName, nodeKey, nodeData)
    currentNPC = npcName
    currentNode = nodeKey
    
    dialogueGUI.Enabled = true
    speakerLabel.Text = nodeData.speaker
    
    -- Clear old choices
    for _, child in pairs(choicesFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- Typewriter effect for text
    typewriterEffect(dialogueText, nodeData.text)
    
    -- Add choice buttons
    for i, choice in ipairs(nodeData.choices) do
        local choiceButton = Instance.new("TextButton")
        choiceButton.Name = "Choice" .. i
        choiceButton.Size = UDim2.new(1, 0, 0, 30)
        choiceButton.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
        choiceButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        choiceButton.TextSize = 14
        choiceButton.Font = Enum.Font.Gotham
        choiceButton.Text = i .. ". " .. choice.text
        choiceButton.TextXAlignment = Enum.TextXAlignment.Left
        choiceButton.AutoButtonColor = true
        choiceButton.Parent = choicesFrame
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 5)
        buttonCorner.Parent = choiceButton
        
        local padding = Instance.new("UIPadding")
        padding.PaddingLeft = UDim.new(0, 10)
        padding.Parent = choiceButton
        
        choiceButton.MouseButton1Click:Connect(function()
            dialogueChoice:FireServer(currentNPC, i, currentNode)
        end)
        
        -- Hover effect
        choiceButton.MouseEnter:Connect(function()
            TweenService:Create(choiceButton, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(70, 70, 100)
            }):Play()
        end)
        
        choiceButton.MouseLeave:Connect(function()
            TweenService:Create(choiceButton, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(50, 50, 70)
            }):Play()
        end)
    end
end

-- Hide dialogue
local function hideDialogue()
    dialogueGUI.Enabled = false
    currentNPC = nil
    currentNode = nil
end

-- Event handlers
startDialogue.OnClientEvent:Connect(showDialogue)
endDialogue.OnClientEvent:Connect(hideDialogue)
```

---

## 👾 Enemy AI Behavior

### Prompt Template

```
Generate a Roblox enemy AI script with [behavior type].

Requirements:
- AI states: [idle/patrol/chase/attack/flee]
- Attack type: [melee/ranged/area]
- Difficulty modifiers: [health/damage/speed]
- Special abilities: [optional abilities]
```

### Example Usage

**Input:**
```
Generate a Roblox enemy AI script with state machine behavior.

Requirements:
- AI states: idle, patrol, chase, attack, retreat
- Attack type: melee with lunge
- Difficulty modifiers: configurable health, damage, and detection range
- Special abilities: enrage at low health (faster, more damage)
```

**Expected Output:**
```lua
-- EnemyAI.lua
-- Place in enemy model

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local Debris = game:GetService("Debris")

local enemy = script.Parent
local humanoid = enemy:WaitForChild("Humanoid")
local rootPart = enemy:WaitForChild("HumanoidRootPart")

-- Configuration (easily adjustable for difficulty)
local Config = {
    -- Base stats
    MaxHealth = 100,
    WalkSpeed = 14,
    ChaseSpeed = 20,
    
    -- Detection
    DetectionRange = 40,
    AttackRange = 5,
    LoseTargetRange = 60,
    
    -- Combat
    Damage = 15,
    AttackCooldown = 1.5,
    LungeForce = 50,
    
    -- Enrage (below 30% health)
    EnrageThreshold = 0.3,
    EnrageSpeedMultiplier = 1.3,
    EnrageDamageMultiplier = 1.5,
    
    -- Retreat
    RetreatHealthThreshold = 0.15,
    RetreatDistance = 30
}

-- State machine
local States = {
    IDLE = "idle",
    PATROL = "patrol",
    CHASE = "chase",
    ATTACK = "attack",
    RETREAT = "retreat"
}

local currentState = States.IDLE
local currentTarget = nil
local lastAttackTime = 0
local isEnraged = false
local originalPosition = rootPart.Position

-- Initialize
humanoid.MaxHealth = Config.MaxHealth
humanoid.Health = Config.MaxHealth
humanoid.WalkSpeed = Config.WalkSpeed

-- Pathfinding
local pathParams = {
    AgentRadius = 2,
    AgentHeight = 5,
    AgentCanJump = true
}

local function createPath(targetPosition)
    local path = PathfindingService:CreatePath(pathParams)
    pcall(function()
        path:ComputeAsync(rootPart.Position, targetPosition)
    end)
    return path.Status == Enum.PathStatus.Success and path or nil
end

-- Find target
local function findTarget()
    local nearestPlayer = nil
    local nearestDistance = Config.DetectionRange
    
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local playerRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local playerHumanoid = player.Character:FindFirstChild("Humanoid")
            
            if playerRoot and playerHumanoid and playerHumanoid.Health > 0 then
                local distance = (rootPart.Position - playerRoot.Position).Magnitude
                
                -- Raycast to check line of sight
                local rayParams = RaycastParams.new()
                rayParams.FilterDescendantsInstances = {enemy}
                local rayResult = workspace:Raycast(rootPart.Position, (playerRoot.Position - rootPart.Position), rayParams)
                
                if distance < nearestDistance then
                    if not rayResult or rayResult.Instance:IsDescendantOf(player.Character) then
                        nearestDistance = distance
                        nearestPlayer = player
                    end
                end
            end
        end
    end
    
    return nearestPlayer
end

-- Check if target is valid
local function isTargetValid()
    if not currentTarget then return false end
    
    local character = currentTarget.Character
    if not character then return false end
    
    local targetRoot = character:FindFirstChild("HumanoidRootPart")
    local targetHumanoid = character:FindFirstChild("Humanoid")
    
    if not targetRoot or not targetHumanoid then return false end
    if targetHumanoid.Health <= 0 then return false end
    
    local distance = (rootPart.Position - targetRoot.Position).Magnitude
    return distance <= Config.LoseTargetRange
end

-- Attack function
local function attack()
    if not currentTarget or not currentTarget.Character then return end
    
    local targetRoot = currentTarget.Character:FindFirstChild("HumanoidRootPart")
    local targetHumanoid = currentTarget.Character:FindFirstChild("Humanoid")
    
    if not targetRoot or not targetHumanoid then return end
    
    local currentTime = tick()
    if currentTime - lastAttackTime < Config.AttackCooldown then return end
    
    lastAttackTime = currentTime
    
    -- Lunge towards target
    local direction = (targetRoot.Position - rootPart.Position).Unit
    local lungeForce = isEnraged and Config.LungeForce * 1.2 or Config.LungeForce
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = direction * lungeForce + Vector3.new(0, 10, 0)
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVelocity.Parent = rootPart
    Debris:AddItem(bodyVelocity, 0.2)
    
    -- Deal damage after short delay
    delay(0.2, function()
        if currentTarget and currentTarget.Character then
            local tRoot = currentTarget.Character:FindFirstChild("HumanoidRootPart")
            local tHumanoid = currentTarget.Character:FindFirstChild("Humanoid")
            
            if tRoot and tHumanoid then
                local distance = (rootPart.Position - tRoot.Position).Magnitude
                if distance <= Config.AttackRange + 2 then
                    local damage = isEnraged and Config.Damage * Config.EnrageDamageMultiplier or Config.Damage
                    tHumanoid:TakeDamage(damage)
                end
            end
        end
    end)
end

-- Check enrage
local function checkEnrage()
    local healthPercent = humanoid.Health / humanoid.MaxHealth
    
    if not isEnraged and healthPercent <= Config.EnrageThreshold then
        isEnraged = true
        
        -- Visual feedback
        for _, part in pairs(enemy:GetDescendants()) do
            if part:IsA("BasePart") then
                part.BrickColor = BrickColor.new("Really red")
            end
        end
        
        -- Stat changes are applied in movement
        print(enemy.Name .. " is now ENRAGED!")
    end
end

-- State behaviors
local stateBehaviors = {
    [States.IDLE] = function()
        humanoid.WalkSpeed = 0
        
        -- Look for targets
        local target = findTarget()
        if target then
            currentTarget = target
            return States.CHASE
        end
        
        -- Random chance to patrol
        if math.random() < 0.01 then
            return States.PATROL
        end
        
        return States.IDLE
    end,
    
    [States.PATROL] = function()
        humanoid.WalkSpeed = Config.WalkSpeed
        
        -- Check for targets
        local target = findTarget()
        if target then
            currentTarget = target
            return States.CHASE
        end
        
        -- Random patrol movement
        local patrolOffset = Vector3.new(
            math.random(-20, 20),
            0,
            math.random(-20, 20)
        )
        local patrolTarget = originalPosition + patrolOffset
        
        humanoid:MoveTo(patrolTarget)
        humanoid.MoveToFinished:Wait()
        
        return States.IDLE
    end,
    
    [States.CHASE] = function()
        -- Check health for retreat
        local healthPercent = humanoid.Health / humanoid.MaxHealth
        if healthPercent <= Config.RetreatHealthThreshold then
            return States.RETREAT
        end
        
        if not isTargetValid() then
            currentTarget = nil
            return States.IDLE
        end
        
        local targetRoot = currentTarget.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return States.IDLE end
        
        local distance = (rootPart.Position - targetRoot.Position).Magnitude
        
        -- Set speed based on enrage
        local speed = isEnraged and Config.ChaseSpeed * Config.EnrageSpeedMultiplier or Config.ChaseSpeed
        humanoid.WalkSpeed = speed
        
        if distance <= Config.AttackRange then
            return States.ATTACK
        end
        
        -- Move towards target
        humanoid:MoveTo(targetRoot.Position)
        
        return States.CHASE
    end,
    
    [States.ATTACK] = function()
        if not isTargetValid() then
            currentTarget = nil
            return States.IDLE
        end
        
        local targetRoot = currentTarget.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return States.IDLE end
        
        local distance = (rootPart.Position - targetRoot.Position).Magnitude
        
        if distance > Config.AttackRange then
            return States.CHASE
        end
        
        -- Face target
        local lookVector = (targetRoot.Position - rootPart.Position).Unit
        rootPart.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + lookVector)
        
        -- Attack
        attack()
        
        return States.ATTACK
    end,
    
    [States.RETREAT] = function()
        if not currentTarget or not currentTarget.Character then
            return States.IDLE
        end
        
        local targetRoot = currentTarget.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return States.IDLE end
        
        -- Run away from target
        local direction = (rootPart.Position - targetRoot.Position).Unit
        local retreatPosition = rootPart.Position + direction * Config.RetreatDistance
        
        humanoid.WalkSpeed = Config.ChaseSpeed
        humanoid:MoveTo(retreatPosition)
        
        local distance = (rootPart.Position - targetRoot.Position).Magnitude
        if distance >= Config.RetreatDistance then
            currentTarget = nil
            return States.IDLE
        end
        
        return States.RETREAT
    end
}

-- Main AI loop
spawn(function()
    while humanoid.Health > 0 do
        checkEnrage()
        
        local behavior = stateBehaviors[currentState]
        if behavior then
            currentState = behavior()
        end
        
        wait(0.1)
    end
end)

-- Death handling
humanoid.Died:Connect(function()
    -- Drop loot, give XP, etc.
    print(enemy.Name .. " has been defeated!")
    
    wait(3)
    enemy:Destroy()
end)
```

---

## 🔄 Tips for Better NPC Behavior

1. **Use state machines** - Easier to debug and extend
2. **Implement line of sight** - Raycasts for realistic detection
3. **Add reaction delays** - Makes AI feel more natural
4. **Use pathfinding wisely** - Expensive, cache paths when possible
5. **Handle edge cases** - Character death, teleportation, etc.
6. **Test with multiple NPCs** - Ensure performance at scale

---

## 🎯 Quick Reference Prompts

### Friendly NPC
```
Generate a Roblox friendly NPC that greets players, gives tips, and can be asked questions about the game world.
```

### Shop Keeper
```
Generate a Roblox shop NPC with a GUI that displays items for sale, handles purchases, and updates player inventory.
```

### Quest Giver
```
Generate a Roblox quest NPC that tracks player progress, offers multiple quests, and gives rewards upon completion.
```

### Boss Enemy
```
Generate a Roblox boss enemy AI with multiple attack phases, special abilities, and health bar UI.
```
