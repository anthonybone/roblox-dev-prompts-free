# Zero-Knowledge Testing Prompts

Test your Roblox games even without prior knowledge of the codebase. These prompts help you create test plans, validate functionality, and ensure quality.

---

## 🧪 Feature Testing

### Prompt Template

\`\`\`
Create a test plan for testing [feature] in a Roblox game.

Feature description:
- [what the feature does]
- [expected inputs and outputs]

Test without knowing:
- Internal implementation
- Specific variable names
- Code structure
\`\`\`

### Example Usage

**Input:**
\`\`\`
Create a test plan for testing a coin collection system in a Roblox game.

Feature description:
- Players can collect coins by walking into them
- Each coin adds points to the player's score
- Coins disappear after collection

Test without knowing:
- How coins are spawned
- What the score variable is called
- Server vs client implementation
\`\`\`

**Expected Output:**
\`\`\`markdown
# Coin Collection System Test Plan

## Test Environment Setup
1. Start a test server in Roblox Studio
2. Note initial player score (if visible on leaderboard)
3. Locate coins in the game world

## Functional Tests

### TC-001: Basic Coin Collection
**Steps:**
1. Walk character towards a visible coin
2. Make contact with the coin

**Expected Results:**
- Coin should disappear
- Score should increase
- Visual/audio feedback (if any) should trigger

**Pass Criteria:** All expected results occur

---

### TC-002: Score Increment Verification
**Steps:**
1. Note current score
2. Collect exactly 1 coin
3. Note new score

**Expected Results:**
- Score increases by a fixed amount
- Change is visible on UI

**Pass Criteria:** Score increased by expected amount

---

### TC-003: Multiple Coin Collection
**Steps:**
1. Note current score
2. Collect 5 coins in succession
3. Note final score

**Expected Results:**
- Each coin adds to score
- Total increase = individual coin value × 5
- All 5 coins disappear

**Pass Criteria:** Score math is correct

---

### TC-004: Coin Persistence
**Steps:**
1. Approach a coin but don't touch it
2. Walk away
3. Return to the coin

**Expected Results:**
- Coin should still be present
- Coin should still be collectable

**Pass Criteria:** Uncollected coins persist

---

## Edge Case Tests

### TC-005: Simultaneous Collection Attempt
**Steps:**
1. In multiplayer, have 2 players approach same coin
2. Both attempt to collect simultaneously

**Expected Results:**
- Only one player should receive points
- Coin should only be collected once
- No errors or duplications

**Pass Criteria:** No double-counting

---

### TC-006: Character Respawn
**Steps:**
1. Collect some coins
2. Reset/die and respawn
3. Check score

**Expected Results:**
- Score should persist after respawn
- Previously collected coins should stay gone

**Pass Criteria:** Progress is maintained

---

### TC-007: Rapid Movement Through Coins
**Steps:**
1. Move very quickly through multiple coins
2. Use speed boost if available

**Expected Results:**
- All touched coins should be collected
- No coins should be "missed" due to speed

**Pass Criteria:** No collection misses

---

## Performance Tests

### TC-008: Many Coins Present
**Steps:**
1. Observe game with many coins spawned
2. Monitor for frame drops or lag

**Expected Results:**
- Game maintains acceptable framerate
- No significant lag

**Pass Criteria:** Performance is acceptable

---

## Results Template

| Test ID | Status | Notes |
|---------|--------|-------|
| TC-001  | ⬜     |       |
| TC-002  | ⬜     |       |
| TC-003  | ⬜     |       |
| TC-004  | ⬜     |       |
| TC-005  | ⬜     |       |
| TC-006  | ⬜     |       |
| TC-007  | ⬜     |       |
| TC-008  | ⬜     |       |

Status: ✅ Pass | ❌ Fail | ⬜ Not Run
\`\`\`

---

## 🎮 Gameplay Flow Testing

### Prompt Template

\`\`\`
Create a user journey test for [game type] covering:
- First-time player experience
- Core gameplay loop
- Edge cases a new player might encounter
\`\`\`

### Example Usage

**Input:**
\`\`\`
Create a user journey test for an obby (obstacle course) game covering:
- First-time player experience
- Core gameplay loop
- Edge cases a new player might encounter
\`\`\`

**Expected Output:**
\`\`\`markdown
# Obby Game User Journey Test

## Phase 1: First-Time Player Experience

### Journey Step 1: Game Entry
**Actions:**
1. Join the game for the first time
2. Observe spawn location and surroundings

**Verify:**
- [ ] Player spawns in a safe location
- [ ] Starting area is clear and visible
- [ ] Instructions or tutorial hints are present (if any)
- [ ] UI elements are visible and readable

---

### Journey Step 2: Understanding the Objective
**Actions:**
1. Look around the starting area
2. Identify the first obstacle or path

**Verify:**
- [ ] Next step/objective is visually apparent
- [ ] Direction to proceed is clear
- [ ] No confusion about what to do

---

### Journey Step 3: First Obstacle Attempt
**Actions:**
1. Attempt the first obstacle
2. Fail intentionally to test respawn

**Verify:**
- [ ] Obstacle is reasonably achievable
- [ ] Failure results in respawn at checkpoint
- [ ] No softlocks (stuck without dying)
- [ ] Respawn is quick and non-punishing

---

## Phase 2: Core Gameplay Loop

### Journey Step 4: Progression
**Actions:**
1. Complete first few obstacles
2. Reach first checkpoint (if applicable)

**Verify:**
- [ ] Progress is saved at checkpoints
- [ ] Difficulty ramps gradually
- [ ] Each obstacle feels fair

---

### Journey Step 5: Challenge Variety
**Actions:**
1. Progress through multiple stages
2. Note different obstacle types

**Verify:**
- [ ] Obstacles vary in type
- [ ] Skills learned early are useful later
- [ ] No cheap deaths or unfair mechanics

---

## Phase 3: Edge Cases

### Edge Case 1: Skipping Sections
**Actions:**
1. Try to jump over or around obstacles
2. Attempt to clip through walls

**Verify:**
- [ ] Shortcuts are intentional or blocked
- [ ] Can't skip to later sections unfairly

---

### Edge Case 2: AFK/Idle Behavior
**Actions:**
1. Stand still for extended period
2. Leave character in potentially dangerous spot

**Verify:**
- [ ] Game handles idle players gracefully
- [ ] No auto-kick during reasonable time

---

### Edge Case 3: Rejoining
**Actions:**
1. Leave the game mid-progress
2. Rejoin shortly after

**Verify:**
- [ ] Progress is saved
- [ ] Player returns to appropriate checkpoint
- [ ] No data loss or corruption

---

## User Journey Results Summary

| Phase | Step | Pass/Fail | Issues Found |
|-------|------|-----------|--------------|
| 1     | 1    |           |              |
| 1     | 2    |           |              |
| 1     | 3    |           |              |
| 2     | 4    |           |              |
| 2     | 5    |           |              |
| 3     | EC1  |           |              |
| 3     | EC2  |           |              |
| 3     | EC3  |           |              |
\`\`\`

---

## 🔐 Security Testing

### Prompt Template

\`\`\`
Create security tests for [feature] to check for common exploits:
- Client-side manipulation
- RemoteEvent abuse
- Data validation issues
\`\`\`

### Example Usage

**Input:**
\`\`\`
Create security tests for a shop purchase system to check for common exploits:
- Client-side manipulation
- RemoteEvent abuse
- Data validation issues
\`\`\`

**Expected Output:**
\`\`\`markdown
# Shop System Security Test Plan

## Threat Model
- Attackers may modify client-side values
- Attackers may fire RemoteEvents with invalid data
- Attackers may attempt to purchase items without payment

## Security Tests

### SEC-001: Client-Side Currency Modification
**Attack Vector:**
Modify local currency display/value via exploits

**Test Steps:**
1. Note server-side currency value (check leaderboard)
2. If possible, modify client-side currency display
3. Attempt to purchase expensive item

**Expected Defense:**
- Server validates actual currency on purchase
- Client display doesn't affect server state

**Pass Criteria:** Purchase fails or uses real server value

---

### SEC-002: RemoteEvent Parameter Manipulation
**Attack Vector:**
Fire purchase RemoteEvent with modified parameters

**Test Steps:**
1. Identify purchase remote (if accessible)
2. Attempt to send: negative price, zero price, wrong item ID
3. Observe server response

**Expected Defense:**
- Server validates all parameters
- Invalid requests are rejected
- No currency gained from invalid requests

**Pass Criteria:** All invalid requests properly rejected

---

### SEC-003: Race Condition Purchase
**Attack Vector:**
Rapidly send multiple purchase requests

**Test Steps:**
1. Have exactly enough currency for 1 purchase
2. Rapidly fire purchase request multiple times

**Expected Defense:**
- Only one purchase succeeds
- Currency doesn't go negative
- No duplicate items received

**Pass Criteria:** Single purchase processed correctly

---

### SEC-004: Price Mismatch Attack
**Attack Vector:**
Send purchase with mismatched item ID and price

**Test Steps:**
1. Find expensive item and cheap item
2. Attempt to buy expensive item at cheap price

**Expected Defense:**
- Server looks up correct price by item ID
- Incorrect price parameter is ignored

**Pass Criteria:** Correct price always charged

---

### SEC-005: Inventory Overflow
**Attack Vector:**
Purchase items beyond inventory limit

**Test Steps:**
1. Fill inventory to maximum (if limit exists)
2. Attempt to purchase more items

**Expected Defense:**
- Purchase prevented when inventory full
- Clear error message shown
- No items lost, no phantom items

**Pass Criteria:** Inventory limits enforced

---

## Security Assessment Matrix

| Test ID | Risk Level | Status | Vulnerable? |
|---------|------------|--------|-------------|
| SEC-001 | High       |        |             |
| SEC-002 | Critical   |        |             |
| SEC-003 | High       |        |             |
| SEC-004 | Critical   |        |             |
| SEC-005 | Medium     |        |             |

Risk Levels: Critical > High > Medium > Low
\`\`\`

---

## 📊 Test Automation Scripts

### Prompt Template

\`\`\`
Generate a Roblox test script that automatically validates:
- [feature to test]
- [expected behavior]
- [pass/fail conditions]
\`\`\`

### Example Usage

**Input:**
\`\`\`
Generate a Roblox test script that automatically validates:
- Leaderboard score display
- Score updates when coins collected
- Pass/fail conditions for accuracy
\`\`\`

**Expected Output:**
\`\`\`lua
-- LeaderboardTest.lua
-- Place in ServerScriptService (run in Studio test mode)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Test framework
local TestResults = {
    passed = 0,
    failed = 0,
    tests = {}
}

local function logTest(name, passed, message)
    local status = passed and "✅ PASS" or "❌ FAIL"
    print(string.format("[TEST] %s: %s - %s", status, name, message or ""))
    
    table.insert(TestResults.tests, {
        name = name,
        passed = passed,
        message = message
    })
    
    if passed then
        TestResults.passed = TestResults.passed + 1
    else
        TestResults.failed = TestResults.failed + 1
    end
end

local function assertEqual(actual, expected, testName)
    local passed = actual == expected
    local message = passed and "Values match" or 
        string.format("Expected %s, got %s", tostring(expected), tostring(actual))
    logTest(testName, passed, message)
    return passed
end

local function assertExists(object, testName)
    local passed = object ~= nil
    logTest(testName, passed, passed and "Object exists" or "Object is nil")
    return passed
end

-- Wait for test player
local function waitForTestPlayer(timeout)
    timeout = timeout or 30
    local startTime = tick()
    
    while tick() - startTime < timeout do
        local players = Players:GetPlayers()
        if #players > 0 then
            return players[1]
        end
        wait(0.5)
    end
    
    return nil
end

-- Main test suite
local function runTests()
    print("\n========== LEADERBOARD TEST SUITE ==========\n")
    
    -- Wait for player
    print("Waiting for test player...")
    local player = waitForTestPlayer()
    
    if not player then
        print("❌ No player joined. Tests cannot run.")
        return
    end
    
    print("Player found: " .. player.Name .. "\n")
    wait(2) -- Let player fully load
    
    -- TEST 1: Leaderstats exists
    local leaderstats = player:FindFirstChild("leaderstats")
    assertExists(leaderstats, "Leaderstats folder exists")
    
    if not leaderstats then
        print("\n⚠️ Cannot continue without leaderstats")
        return
    end
    
    -- TEST 2: Coins value exists
    local coins = leaderstats:FindFirstChild("Coins")
    assertExists(coins, "Coins value exists in leaderstats")
    
    if not coins then
        print("\n⚠️ Cannot continue without Coins value")
        return
    end
    
    -- TEST 3: Initial value is number
    local initialValue = coins.Value
    local isNumber = type(initialValue) == "number"
    logTest("Coins value is a number", isNumber, 
        "Type: " .. type(initialValue))
    
    -- TEST 4: Value is non-negative
    local isNonNegative = initialValue >= 0
    logTest("Coins value is non-negative", isNonNegative,
        "Value: " .. tostring(initialValue))
    
    -- TEST 5: Value change detection
    print("\nTesting value change detection...")
    local valueChanged = false
    local newValue = nil
    
    local connection = coins.Changed:Connect(function(value)
        valueChanged = true
        newValue = value
    end)
    
    -- Simulate adding coins (this would normally happen via gameplay)
    local testAddAmount = 10
    coins.Value = coins.Value + testAddAmount
    wait(0.1)
    
    logTest("Value change event fires", valueChanged, 
        valueChanged and "Event fired correctly" or "Event did not fire")
    
    if newValue then
        assertEqual(newValue, initialValue + testAddAmount, 
            "Value changed by correct amount")
    end
    
    connection:Disconnect()
    
    -- TEST 6: Leaderboard UI visibility (if applicable)
    local playerGui = player:FindFirstChild("PlayerGui")
    if playerGui then
        local leaderboardUI = playerGui:FindFirstChild("Leaderboard") or
                             playerGui:FindFirstChild("PlayerList")
        if leaderboardUI then
            logTest("Leaderboard UI exists", true, "UI found")
        else
            logTest("Leaderboard UI exists", false, 
                "Using default Roblox leaderboard (OK)")
        end
    end
    
    -- Print summary
    print("\n========== TEST RESULTS ==========")
    print(string.format("Passed: %d", TestResults.passed))
    print(string.format("Failed: %d", TestResults.failed))
    print(string.format("Total:  %d", TestResults.passed + TestResults.failed))
    print("==================================\n")
end

-- Run tests after short delay
spawn(function()
    wait(3)
    runTests()
end)
\`\`\`

---

## 🔄 Tips for Effective Testing

1. **Test as a new player** - Clear your data and experience the game fresh
2. **Try to break things** - Players will find unexpected uses
3. **Test multiplayer** - Many bugs only appear with multiple players
4. **Check mobile** - Touch controls behave differently
5. **Test edge cases** - Minimum values, maximum values, empty states
6. **Document everything** - Screenshots and videos help reproduction
7. **Regression test** - Re-run tests after making changes

---

## 🎯 Quick Reference Prompts

### Smoke Test
\`\`\`
Create a quick smoke test checklist for a Roblox [game type] that covers the most critical functionality in under 5 minutes.
\`\`\`

### Multiplayer Test
\`\`\`
Design multiplayer test scenarios for [feature] that could have synchronization or race condition issues.
\`\`\`

### Mobile Test
\`\`\`
Create a mobile-specific test plan for [game] covering touch controls, screen size adaptation, and performance on lower-end devices.
\`\`\`

### Regression Test
\`\`\`
After changing [feature], what tests should I run to make sure I didn"t break anything else in the game?
\`\`\`
