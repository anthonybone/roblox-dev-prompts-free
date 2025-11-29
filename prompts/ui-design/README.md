# UI Design Prompts

Create beautiful, functional user interfaces for your Roblox games. These prompts help you generate responsive GUIs, menus, HUDs, and interactive elements.

---

## 🎨 Main Menu Design

### Prompt Template

\`\`\`
Generate a Roblox main menu UI with [style/theme].

Requirements:
- Buttons: [list of buttons]
- Style: [modern/retro/fantasy/minimal]
- Animations: [fade/slide/scale]
- Include: [logo/background/music toggle]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Generate a Roblox main menu UI with modern dark theme.

Requirements:
- Buttons: Play, Shop, Settings, Credits
- Style: modern with rounded corners and gradients
- Animations: smooth fade and slide transitions
- Include: game logo placeholder, animated background particles
\`\`\`

**Expected Output:**
\`\`\`lua
-- MainMenuUI.lua
-- Place in StarterGui as a LocalScript inside a ScreenGui

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MainMenu"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

-- Background
local background = Instance.new("Frame")
background.Name = "Background"
background.Size = UDim2.new(1, 0, 1, 0)
background.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
background.BorderSizePixel = 0
background.Parent = screenGui

-- Gradient overlay
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 40)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 20))
})
gradient.Rotation = 45
gradient.Parent = background

-- Logo placeholder
local logoFrame = Instance.new("Frame")
logoFrame.Name = "LogoFrame"
logoFrame.Size = UDim2.new(0, 400, 0, 150)
logoFrame.Position = UDim2.new(0.5, -200, 0.15, 0)
logoFrame.BackgroundTransparency = 1
logoFrame.Parent = screenGui

local logoText = Instance.new("TextLabel")
logoText.Name = "Logo"
logoText.Size = UDim2.new(1, 0, 1, 0)
logoText.BackgroundTransparency = 1
logoText.Text = "YOUR GAME"
logoText.TextColor3 = Color3.fromRGB(255, 255, 255)
logoText.TextSize = 60
logoText.Font = Enum.Font.GothamBlack
logoText.Parent = logoFrame

-- Button container
local buttonContainer = Instance.new("Frame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(0, 300, 0, 300)
buttonContainer.Position = UDim2.new(0.5, -150, 0.45, 0)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Parent = screenGui

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 15)
listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
listLayout.Parent = buttonContainer

-- Button creation function
local function createButton(text, order)
    local button = Instance.new("TextButton")
    button.Name = text .. "Button"
    button.Size = UDim2.new(1, 0, 0, 50)
    button.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Text = text
    button.TextSize = 20
    button.Font = Enum.Font.GothamSemibold
    button.AutoButtonColor = false
    button.LayoutOrder = order
    button.Parent = buttonContainer
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = button
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(80, 80, 120)
    stroke.Thickness = 2
    stroke.Parent = button
    
    -- Hover effects
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(60, 60, 100),
            Size = UDim2.new(1.05, 0, 0, 55)
        }):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(40, 40, 60),
            Size = UDim2.new(1, 0, 0, 50)
        }):Play()
    end)
    
    return button
end

-- Create buttons
local playButton = createButton("▶ PLAY", 1)
local shopButton = createButton("🛒 SHOP", 2)
local settingsButton = createButton("⚙ SETTINGS", 3)
local creditsButton = createButton("📜 CREDITS", 4)

-- Button click handlers
playButton.MouseButton1Click:Connect(function()
    print("Starting game...")
end)

shopButton.MouseButton1Click:Connect(function()
    print("Opening shop...")
end)

settingsButton.MouseButton1Click:Connect(function()
    print("Opening settings...")
end)

creditsButton.MouseButton1Click:Connect(function()
    print("Opening credits...")
end)
\`\`\`

---

## 📊 HUD (Heads-Up Display)

### Prompt Template

\`\`\`
Generate a Roblox HUD for [game type].

Display elements:
- [List what to show: health, ammo, minimap, etc.]

Style preferences:
- [Position, size, transparency]
- [Update frequency]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Generate a Roblox HUD for an action RPG.

Display elements:
- Health bar with current/max values
- Experience bar with level indicator
- Quick action slots (3 abilities)
- Currency display

Style preferences:
- Health at top-left, XP below it
- Action slots at bottom-center
- Currency at top-right
- Animated value changes
\`\`\`

**Expected Output:**
\`\`\`lua
-- ActionRPGHUD.lua
-- Place in StarterGui

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create main HUD
local hudGui = Instance.new("ScreenGui")
hudGui.Name = "ActionRPGHUD"
hudGui.ResetOnSpawn = false
hudGui.Parent = playerGui

-- HEALTH BAR
local healthFrame = Instance.new("Frame")
healthFrame.Name = "HealthFrame"
healthFrame.Size = UDim2.new(0, 250, 0, 35)
healthFrame.Position = UDim2.new(0, 20, 0, 20)
healthFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
healthFrame.BorderSizePixel = 0
healthFrame.Parent = hudGui

local healthCorner = Instance.new("UICorner")
healthCorner.CornerRadius = UDim.new(0, 8)
healthCorner.Parent = healthFrame

local healthBar = Instance.new("Frame")
healthBar.Name = "HealthBar"
healthBar.Size = UDim2.new(1, -10, 1, -10)
healthBar.Position = UDim2.new(0, 5, 0, 5)
healthBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
healthBar.BorderSizePixel = 0
healthBar.Parent = healthFrame

local healthFill = Instance.new("Frame")
healthFill.Name = "Fill"
healthFill.Size = UDim2.new(1, 0, 1, 0)
healthFill.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
healthFill.BorderSizePixel = 0
healthFill.Parent = healthBar

local healthText = Instance.new("TextLabel")
healthText.Name = "HealthText"
healthText.Size = UDim2.new(1, 0, 1, 0)
healthText.BackgroundTransparency = 1
healthText.Text = "100 / 100"
healthText.TextColor3 = Color3.fromRGB(255, 255, 255)
healthText.TextSize = 14
healthText.Font = Enum.Font.GothamBold
healthText.Parent = healthBar

-- Update function
local function updateHealth(current, max)
    local percentage = math.clamp(current / max, 0, 1)
    
    TweenService:Create(healthFill, TweenInfo.new(0.3), {
        Size = UDim2.new(percentage, 0, 1, 0)
    }):Play()
    
    healthText.Text = string.format("%d / %d", current, max)
end

-- Connect to humanoid
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

humanoid.HealthChanged:Connect(function(health)
    updateHealth(health, humanoid.MaxHealth)
end)

updateHealth(humanoid.Health, humanoid.MaxHealth)
\`\`\`

---

## 🛒 Shop Interface

### Prompt Template

\`\`\`
Generate a Roblox shop UI for [shop type].

Features:
- Item display: [grid/list/carousel]
- Categories: [list categories]
- Purchase flow: [confirm dialog/instant buy]
- Include: [search, filters, item preview]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Generate a Roblox shop UI for a weapon shop.

Features:
- Item display: grid layout with icons
- Categories: Swords, Bows, Staffs
- Purchase flow: confirmation dialog
- Include: player currency, item rarity colors
\`\`\`

**Expected Output:**
\`\`\`lua
-- WeaponShopUI.lua
-- Place in StarterGui

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local RarityColors = {
    Common = Color3.fromRGB(150, 150, 150),
    Uncommon = Color3.fromRGB(50, 200, 50),
    Rare = Color3.fromRGB(50, 150, 255),
    Epic = Color3.fromRGB(180, 50, 255),
    Legendary = Color3.fromRGB(255, 180, 0)
}

-- Create shop GUI
local shopGui = Instance.new("ScreenGui")
shopGui.Name = "WeaponShop"
shopGui.Enabled = false
shopGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 700, 0, 500)
mainFrame.Position = UDim2.new(0.5, -350, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = shopGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 15)
mainCorner.Parent = mainFrame

-- Header with title and close button
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 50)
header.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
header.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(0.5, 0, 1, 0)
titleLabel.Position = UDim2.new(0, 20, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚔️ Weapon Shop"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 24
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = header

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 40, 0, 40)
closeButton.Position = UDim2.new(1, -45, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 20
closeButton.Parent = header

closeButton.MouseButton1Click:Connect(function()
    shopGui.Enabled = false
end)

-- Items grid
local itemsFrame = Instance.new("ScrollingFrame")
itemsFrame.Size = UDim2.new(1, -40, 1, -130)
itemsFrame.Position = UDim2.new(0, 20, 0, 110)
itemsFrame.BackgroundTransparency = 1
itemsFrame.Parent = mainFrame

local itemsGrid = Instance.new("UIGridLayout")
itemsGrid.CellSize = UDim2.new(0, 150, 0, 180)
itemsGrid.CellPadding = UDim2.new(0, 15, 0, 15)
itemsGrid.Parent = itemsFrame

-- Item card creation
local function createItemCard(item)
    local card = Instance.new("TextButton")
    card.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    card.Text = ""
    card.Parent = itemsFrame
    
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, 10)
    cardCorner.Parent = card
    
    local cardStroke = Instance.new("UIStroke")
    cardStroke.Color = RarityColors[item.rarity] or Color3.fromRGB(100, 100, 100)
    cardStroke.Thickness = 2
    cardStroke.Parent = card
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -10, 0, 25)
    nameLabel.Position = UDim2.new(0, 5, 0, 75)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = item.name
    nameLabel.TextColor3 = RarityColors[item.rarity]
    nameLabel.TextSize = 14
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = card
    
    local priceLabel = Instance.new("TextLabel")
    priceLabel.Size = UDim2.new(1, 0, 0, 25)
    priceLabel.Position = UDim2.new(0, 0, 1, -30)
    priceLabel.BackgroundTransparency = 1
    priceLabel.Text = "💰 " .. item.price
    priceLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    priceLabel.TextSize = 14
    priceLabel.Font = Enum.Font.GothamBold
    priceLabel.Parent = card
    
    return card
end
\`\`\`

---

## 🔄 Tips for Better UI Design

1. **Use UICorner** - Rounded corners look more modern
2. **Add hover effects** - Makes UI feel responsive
3. **Implement tweening** - Smooth animations improve UX
4. **Consider mobile** - Test touch controls and scaling
5. **Use consistent spacing** - UIListLayout and UIPadding help
6. **Color contrast** - Ensure text is readable
7. **Provide feedback** - Show loading states and confirmations

---

## 🎯 Quick Reference Prompts

### Inventory System
\`\`\`
Generate a Roblox inventory UI with drag-and-drop items, item stacking, tooltip on hover, and equipment slots.
\`\`\`

### Settings Menu
\`\`\`
Generate a Roblox settings menu with audio sliders, graphics quality dropdown, keybind configuration, and save/load settings.
\`\`\`

### Notification System
\`\`\`
Generate a Roblox notification UI that shows achievement popups, quest updates, and system messages with auto-dismiss.
\`\`\`

### Loading Screen
\`\`\`
Generate a Roblox loading screen with animated progress bar, tip rotation, and asset preloading feedback.
\`\`\`
