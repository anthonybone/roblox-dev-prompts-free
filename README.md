# 🎮 Roblox Developer Prompt Suite

> A complete prompt collection for solo Roblox developers. Generate Lua scripts, datastore logic, NPC behavior, UI layouts, monetization workflows, debugging solutions, and testing plans.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 📋 What's Inside

This repository contains AI-ready prompts designed specifically for Roblox development. Each prompt template is battle-tested and includes:

- **Clear templates** - Fill in the blanks for your specific needs
- **Real examples** - See how to use each prompt effectively  
- **Working code** - Copy-paste ready Lua scripts
- **Best practices** - Learn proper patterns as you generate code

## 🗂️ Prompt Categories

| Category | Description | Use When... |
|----------|-------------|-------------|
| [📜 Lua Scripts](/prompts/lua-scripts/) | Generate game mechanics, player systems, tools | You need new gameplay features |
| [💾 DataStore](/prompts/datastore/) | Data persistence, player saves, leaderboards | You need to save/load player data |
| [🤖 NPC Behavior](/prompts/npc-behavior/) | Pathfinding, dialogue, enemy AI | You need intelligent characters |
| [🎨 UI Design](/prompts/ui-design/) | Menus, HUDs, shops, notifications | You need player interfaces |
| [💎 Monetization](/prompts/monetization/) | Game passes, products, rewards | You need to monetize your game |
| [🐛 Debugging](/prompts/debugging/) | Error fixing, optimization, logging | You have bugs to squash |
| [🧪 Testing](/prompts/testing/) | Test plans, validation, security | You need to verify your game works |

## 🚀 Quick Start

### 1. Choose Your Category
Browse the categories above and find the prompts that match your needs.

### 2. Copy a Template
Each category has prompt templates you can copy:

\`\`\`
Generate a Roblox Lua script that [describe what you want].

Requirements:
- [List specific requirements]
- [Performance considerations]
- [Any specific Roblox services to use]

The script should be placed in [ServerScriptService/StarterPlayerScripts/etc].
\`\`\`

### 3. Fill in the Details
Replace the bracketed sections with your specific requirements.

### 4. Use with Your AI Assistant
Paste the filled-in prompt into ChatGPT, Claude, or your preferred AI tool.

---

## 📝 Example Usage

### Generating a Coin Collection System

**Your Prompt:**
\`\`\`
Generate a Roblox Lua script that creates a coin collection system.

Requirements:
- Coins should be parts in the workspace with the name "Coin"
- When a player touches a coin, it should disappear and add 10 points
- Use a leaderboard to display the score
- Include debounce to prevent multiple collections

The script should be placed in ServerScriptService.
\`\`\`

**Result:** A complete, working script with proper error handling and best practices.

### Creating an NPC Shopkeeper

**Your Prompt:**
\`\`\`
Generate a Roblox NPC dialogue system with branching conversations.

Requirements:
- Dialogue structure: branching with player choices
- Trigger method: click on NPC (ProximityPrompt)
- UI style: chat GUI at bottom of screen
- Include: choices that affect next dialogue, simple quest giving
\`\`\`

**Result:** Complete server and client scripts with dialogue trees, UI, and quest integration.

---

## 📁 Repository Structure

\`\`\`
roblox-dev-prompts-free/
├── README.md                    # You are here!
├── LICENSE                      # MIT License
└── prompts/
    ├── lua-scripts/
    │   └── README.md           # Script generation prompts
    ├── datastore/
    │   └── README.md           # Data persistence prompts
    ├── npc-behavior/
    │   └── README.md           # NPC and AI prompts
    ├── ui-design/
    │   └── README.md           # UI/UX prompts
    ├── monetization/
    │   └── README.md           # Monetization prompts
    ├── debugging/
    │   └── README.md           # Debugging prompts
    └── testing/
        └── README.md           # Testing prompts
\`\`\`

---

## 🎯 Tips for Better Results

### Be Specific
❌ "Make a combat system"  
✅ "Generate a melee combat system with sword swings, 1 second cooldown, and hitbox detection"

### Include Context
❌ "Save player data"  
✅ "Save player data including coins, level, and inventory to DataStore with retry logic"

### Specify Location
❌ "Make a script"  
✅ "Create a ServerScript for ServerScriptService that handles..."

### Mention Security
❌ "Handle purchases"  
✅ "Handle purchases with server-side validation to prevent exploits"

---

## 🔧 For Solo Developers

This suite is designed with solo developers in mind:

- **Copy-paste ready** - No need to understand everything at first
- **Learn as you go** - Code includes comments explaining the "why"
- **Best practices built-in** - Security, performance, and patterns included
- **Modular design** - Use what you need, ignore the rest

---

## 📚 Learning Resources

New to Roblox development? These prompts also help you learn:

1. **Start with Lua Scripts** - Basic game mechanics
2. **Move to DataStore** - Understand data persistence
3. **Add NPC Behavior** - Learn pathfinding and AI
4. **Build UI** - Create player interfaces
5. **Add Monetization** - Prepare for launch
6. **Debug & Test** - Polish your game

---

## 🤝 Contributing

Found a useful prompt pattern? Have improvements? Contributions are welcome!

1. Fork the repository
2. Add your prompts following the existing format
3. Submit a pull request

---

## 📄 License

MIT License - Use these prompts freely in your projects!

---

## ⭐ Support

If this helps your development:
- Star this repository
- Share with other Roblox developers
- [Report issues](https://github.com/anthonybone/roblox-dev-prompts-free/issues)

---

Made with ❤️ for the Roblox developer community
