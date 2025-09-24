local VirtualUser = game:GetService("VirtualUser")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Teams = game:GetService("Teams")

Players.LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

local platform = Instance.new("Part")
platform.Anchored = true
platform.Size = Vector3.new(50, 1, 50)
platform.Position = Vector3.new(500, 0, 0)
platform.Name = "AntiFallPlatform"
platform.Parent = workspace

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "R3TH PRIV",
    Icon = 0,
    LoadingTitle = "R3TH PRIV",
    LoadingSubtitle = "by R3TH",
    ShowText = "R3TH PRIV",
    Theme = "Default",
    ToggleUIKeybind = "K",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
})

local Tab = Window:CreateTab("Main", 4483362458)

local LocalPlayer = Players.LocalPlayer

local allSeenTargetNames = {}
local selectedTargets = {}
local selectedTeams = {}
local flagKillWhitelistTeams = {}
local swordKillEnabled = false
local autoTPEnabled = false
local autoEquipEnabled = false
local killInAreaEnabled = false

local function arraysEqual(a, b)
    if #a ~= #b then return false end
    for i = 1, #a do
        if a[i] ~= b[i] then return false end
    end
    return true
end

local function GetPlayerNames()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and not table.find(allSeenTargetNames, plr.Name) then
            table.insert(allSeenTargetNames, plr.Name)
        end
    end
    return allSeenTargetNames
end

local Dropdown = Tab:CreateDropdown({
    Name = "Target Players",
    Options = GetPlayerNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "Dropdown1",
    Callback = function(Options)
        selectedTargets = Options
    end,
})

local lastPlayerList = {}

local function refreshDropdown()
    local newPlayerList = GetPlayerNames()
    table.sort(newPlayerList)
    table.sort(lastPlayerList)
    if not arraysEqual(newPlayerList, lastPlayerList) then
        lastPlayerList = newPlayerList
        Dropdown:Refresh(newPlayerList, selectedTargets)
    end
end

task.spawn(function()
    while true do
        pcall(function()
            refreshDropdown()
        end)
        task.wait(2)
    end
end)

local function GetTeamNames()
    local list = {}
    for _, team in pairs(Teams:GetTeams()) do
        table.insert(list, team.Name)
    end
    return list
end

local TeamDropdown = Tab:CreateDropdown({
    Name = "Target Teams",
    Options = GetTeamNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "TeamDropdown",
    Callback = function(Options)
        selectedTeams = Options
    end,
})

local FlagKillWhitelistDropdown = Tab:CreateDropdown({
    Name = "Flag Whitelist",
    Options = GetTeamNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "FlagKillWhitelistDropdown",
    Callback = function(Options)
        flagKillWhitelistTeams = Options
    end,
})

local lastTeamList = {}

local function refreshTeamDropdowns()
    local newTeamList = GetTeamNames()
    table.sort(newTeamList)
    table.sort(lastTeamList)
    if not arraysEqual(newTeamList, lastTeamList) then
        lastTeamList = newTeamList
        TeamDropdown:Refresh(newTeamList, selectedTeams)
        FlagKillWhitelistDropdown:Refresh(newTeamList, flagKillWhitelistTeams)
    end
end

task.spawn(function()
    while true do
        pcall(function()
            refreshTeamDropdowns()
        end)
        task.wait(2)
    end
end)

local function SwordKillLoop()
    local localChar = LocalPlayer.Character
    if not localChar then return end

    local tool = localChar:FindFirstChildOfClass("Tool") or LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
    if not tool then return end

    local toolHandle = tool:FindFirstChild("Handle")
    if not toolHandle then return end

    local targetsToKill = {}

    if #selectedTeams > 0 then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Team and table.find(selectedTeams, plr.Team.Name) and plr.Name ~= "TheRealJohnny3ins" then
                table.insert(targetsToKill, plr.Name)
            end
        end
    else
        for _, name in ipairs(selectedTargets) do
            if name ~= "TheRealJohnny3ins" then
                table.insert(targetsToKill, name)
            end
        end
    end

    for _, targetUsername in ipairs(targetsToKill) do
        local Player = Players:FindFirstChild(targetUsername)
        if Player then
            local targetChar = Player.Character
            if targetChar then
                local Humanoid = targetChar:FindFirstChild("Humanoid")
                if Humanoid and Humanoid.Health > 0 then
                    for _, part in ipairs(targetChar:GetChildren()) do
                        if part:IsA("BasePart") then
                            tool:FindFirstChild("Use"):FireServer()
                            firetouchinterest(toolHandle, part, 0)
                            firetouchinterest(toolHandle, part, 1)
                        end
                    end
                end
            end
        end
    end
end

local function AutoTeleportToTargets()
    while autoTPEnabled do
        pcall(function()
            local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local hrp = character:WaitForChild("HumanoidRootPart")
            hrp.CFrame = CFrame.new(500, 5, 0)
        end)
        task.wait(0.5)
    end
end

local function AutoEquipSword()
    while autoEquipEnabled do
        pcall(function()
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
                    if backpack then
                        local tool = backpack:FindFirstChildOfClass("Tool")
                        if tool and humanoid and humanoid.Health > 0 then
                            humanoid:EquipTool(tool)
                        end
                    end
                end
            end
        end)
        task.wait(1)
    end
end

local middlePosition = Vector3.new(-28, 42, -83)
local killRadius = 200

local function KillPlayersInArea()
    local localChar = LocalPlayer.Character
    if not localChar then return end

    local tool = localChar:FindFirstChildOfClass("Tool") or LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
    if not tool then return end

    local toolHandle = tool:FindFirstChild("Handle")
    if not toolHandle then return end

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Name ~= "TheRealJohnny3ins" then
            local char = plr.Character
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local humanoid = char:FindFirstChild("Humanoid")
                if hrp and humanoid and humanoid.Health > 0 then
                    local dist = (hrp.Position - middlePosition).Magnitude
                    if dist <= killRadius and hrp.Position.Y < 60 then
                        if not plr.Team or not table.find(flagKillWhitelistTeams, plr.Team.Name) then
                            for _, part in ipairs(char:GetChildren()) do
                                if part:IsA("BasePart") then
                                    tool:FindFirstChild("Use"):FireServer()
                                    firetouchinterest(toolHandle, part, 0)
                                    firetouchinterest(toolHandle, part, 1)
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end

local ToggleSwordKill = Tab:CreateToggle({
    Name = "Sword Kill",
    CurrentValue = false,
    Flag = "Toggle1",
    Callback = function(Value)
        swordKillEnabled = Value
        if Value then
            spawn(function()
                while swordKillEnabled do
                    pcall(function()
                        SwordKillLoop()
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end,
})

local ToggleAutoTP = Tab:CreateToggle({
    Name = "Auto Hide",
    CurrentValue = false,
    Flag = "ToggleAutoTP",
    Callback = function(Value)
        autoTPEnabled = Value
        if Value then
            spawn(AutoTeleportToTargets)
        end
    end,
})

local ToggleAutoEquip = Tab:CreateToggle({
    Name = "Auto Equip Sword",
    CurrentValue = false,
    Flag = "ToggleAutoEquip",
    Callback = function(Value)
        autoEquipEnabled = Value
        if Value then
            spawn(AutoEquipSword)
        end
    end,
})

local ToggleKillInArea = Tab:CreateToggle({
    Name = "Kill Players Near Flag",
    CurrentValue = false,
    Flag = "ToggleKillInArea",
    Callback = function(Value)
        killInAreaEnabled = Value
        if Value then
            spawn(function()
                while killInAreaEnabled do
                    pcall(function()
                        KillPlayersInArea()
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end,
})

local circle = workspace:WaitForChild("Koth"):WaitForChild("Circle")

local kothHealingPlatform = Instance.new("Part")
kothHealingPlatform.Name = "UnderCirclePlatform"
kothHealingPlatform.Anchored = true
kothHealingPlatform.Size = Vector3.new(50, 1, 50)
kothHealingPlatform.Position = circle.Position - Vector3.new(0, 25, 0)
kothHealingPlatform.Parent = workspace
kothHealingPlatform.CanCollide = true
kothHealingPlatform.Material = Enum.Material.ForceField
kothHealingPlatform.BrickColor = BrickColor.new("Bright red")

local KothMode = false
local KothLoopRunning = false

RunService.Stepped:Connect(function()
    if not KothMode then return end
    local char = Players.LocalPlayer.Character
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end)

local function randomAboveCircle()
    local hrp = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local offset = Vector3.new(
        math.random(-10, 10),
        math.random(4, 20),
        math.random(-10, 10)
    )
    hrp.CFrame = CFrame.new(circle.Position + offset)
end

local function hideUnderCircle()
    local hrp = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    hrp.CFrame = CFrame.new(kothHealingPlatform.Position + Vector3.new(0, 3, 0))
end

local function KothLoop()
    if KothLoopRunning then return end
    KothLoopRunning = true

    local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        KothLoopRunning = false
        return
    end

    local healing = false

    while KothMode do
        pcall(function()
            if humanoid.Health < 70 and not healing then
                healing = true
                hideUnderCircle()
                repeat
                    wait()
                    humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                    if not humanoid then break end
                until (humanoid and humanoid.Health >= humanoid.MaxHealth) or not KothMode
                healing = false
            end
    
            if not healing then
                randomAboveCircle()
            end
        end)
        task.wait()
    end

    KothLoopRunning = false
end

local KothToggle = Tab:CreateToggle({
    Name = "Steal Flag",
    CurrentValue = false,
    Flag = "ToggleKothAutoTP",
    Callback = function(Value)
        KothMode = Value
        if Value then
            spawn(KothLoop)
        end
    end,
})

local enhfeefefe = Tab:CreateButton({
    Name = "Get All Guns",
    Callback = function()
        local lp = Players.LocalPlayer
        local char = lp.Character or lp.CharacterAdded:Wait()

        local function isToolGiverRelated(part)
            local parent = part.Parent
            if parent and parent.Name:match("^ToolGiver") then
                return true
            elseif parent and parent.Parent and parent.Parent.Name:match("^ToolGiver") then
                return true
            end
            return false
        end

        local function fireTouch(part)
            if not part:IsA("BasePart") then return end
            if not isToolGiverRelated(part) then return end

            for _, ti in ipairs(part:GetChildren()) do
                if ti:IsA("TouchTransmitter") then
                    for _, myPart in ipairs(char:GetDescendants()) do
                        if myPart:IsA("BasePart") then
                            firetouchinterest(myPart, part, 0)
                            firetouchinterest(myPart, part, 1)
                        end
                    end
                end
            end
        end

        for _, v in ipairs(workspace:GetDescendants()) do
            fireTouch(v)
        end
    end,
})
local VirtualUser = game:GetService("VirtualUser")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Teams = game:GetService("Teams")

Players.LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

local platform = Instance.new("Part")
platform.Anchored = true
platform.Size = Vector3.new(50, 1, 50)
platform.Position = Vector3.new(500, 0, 0)
platform.Name = "AntiFallPlatform"
platform.Parent = workspace

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "R3TH PRIV",
    Icon = 0,
    LoadingTitle = "R3TH PRIV",
    LoadingSubtitle = "by R3TH",
    ShowText = "R3TH PRIV",
    Theme = "Default",
    ToggleUIKeybind = "K",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
})

local Tab = Window:CreateTab("Main", 4483362458)

local LocalPlayer = Players.LocalPlayer

local allSeenTargetNames = {}
local selectedTargets = {}
local selectedTeams = {}
local flagKillWhitelistTeams = {}
local swordKillEnabled = false
local autoTPEnabled = false
local autoEquipEnabled = false
local killInAreaEnabled = false

local function arraysEqual(a, b)
    if #a ~= #b then return false end
    for i = 1, #a do
        if a[i] ~= b[i] then return false end
    end
    return true
end

local function GetPlayerNames()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and not table.find(allSeenTargetNames, plr.Name) then
            table.insert(allSeenTargetNames, plr.Name)
        end
    end
    return allSeenTargetNames
end

local Dropdown = Tab:CreateDropdown({
    Name = "Target Players",
    Options = GetPlayerNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "Dropdown1",
    Callback = function(Options)
        selectedTargets = Options
    end,
})

local lastPlayerList = {}

local function refreshDropdown()
    local newPlayerList = GetPlayerNames()
    table.sort(newPlayerList)
    table.sort(lastPlayerList)
    if not arraysEqual(newPlayerList, lastPlayerList) then
        lastPlayerList = newPlayerList
        Dropdown:Refresh(newPlayerList, selectedTargets)
    end
end

task.spawn(function()
    while true do
        pcall(function()
            refreshDropdown()
        end)
        task.wait(2)
    end
end)

local function GetTeamNames()
    local list = {}
    for _, team in pairs(Teams:GetTeams()) do
        table.insert(list, team.Name)
    end
    return list
end

local TeamDropdown = Tab:CreateDropdown({
    Name = "Target Teams",
    Options = GetTeamNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "TeamDropdown",
    Callback = function(Options)
        selectedTeams = Options
    end,
})

local FlagKillWhitelistDropdown = Tab:CreateDropdown({
    Name = "Flag Whitelist",
    Options = GetTeamNames(),
    CurrentOption = {},
    MultipleOptions = true,
    Flag = "FlagKillWhitelistDropdown",
    Callback = function(Options)
        flagKillWhitelistTeams = Options
    end,
})

local lastTeamList = {}

local function refreshTeamDropdowns()
    local newTeamList = GetTeamNames()
    table.sort(newTeamList)
    table.sort(lastTeamList)
    if not arraysEqual(newTeamList, lastTeamList) then
        lastTeamList = newTeamList
        TeamDropdown:Refresh(newTeamList, selectedTeams)
        FlagKillWhitelistDropdown:Refresh(newTeamList, flagKillWhitelistTeams)
    end
end

task.spawn(function()
    while true do
        pcall(function()
            refreshTeamDropdowns()
        end)
        task.wait(2)
    end
end)

local function SwordKillLoop()
    local localChar = LocalPlayer.Character
    if not localChar then return end

    local tool = localChar:FindFirstChildOfClass("Tool") or LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
    if not tool then return end

    local toolHandle = tool:FindFirstChild("Handle")
    if not toolHandle then return end

    local targetsToKill = {}

    if #selectedTeams > 0 then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Team and table.find(selectedTeams, plr.Team.Name) and plr.Name ~= "TheRealJohnny3ins" then
                table.insert(targetsToKill, plr.Name)
            end
        end
    else
        for _, name in ipairs(selectedTargets) do
            if name ~= "TheRealJohnny3ins" then
                table.insert(targetsToKill, name)
            end
        end
    end

    for _, targetUsername in ipairs(targetsToKill) do
        local Player = Players:FindFirstChild(targetUsername)
        if Player then
            local targetChar = Player.Character
            if targetChar then
                local Humanoid = targetChar:FindFirstChild("Humanoid")
                if Humanoid and Humanoid.Health > 0 then
                    for _, part in ipairs(targetChar:GetChildren()) do
                        if part:IsA("BasePart") then
                            tool:FindFirstChild("Use"):FireServer()
                            firetouchinterest(toolHandle, part, 0)
                            firetouchinterest(toolHandle, part, 1)
                        end
                    end
                end
            end
        end
    end
end

local function AutoTeleportToTargets()
    while autoTPEnabled do
        pcall(function()
            local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local hrp = character:WaitForChild("HumanoidRootPart")
            hrp.CFrame = CFrame.new(500, 5, 0)
        end)
        task.wait(0.5)
    end
end

local function AutoEquipSword()
    while autoEquipEnabled do
        pcall(function()
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
                    if backpack then
                        local tool = backpack:FindFirstChildOfClass("Tool")
                        if tool and humanoid and humanoid.Health > 0 then
                            humanoid:EquipTool(tool)
                        end
                    end
                end
            end
        end)
        task.wait(1)
    end
end

local middlePosition = Vector3.new(-28, 42, -83)
local killRadius = 200

local function KillPlayersInArea()
    local localChar = LocalPlayer.Character
    if not localChar then return end

    local tool = localChar:FindFirstChildOfClass("Tool") or LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
    if not tool then return end

    local toolHandle = tool:FindFirstChild("Handle")
    if not toolHandle then return end

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Name ~= "TheRealJohnny3ins" then
            local char = plr.Character
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local humanoid = char:FindFirstChild("Humanoid")
                if hrp and humanoid and humanoid.Health > 0 then
                    local dist = (hrp.Position - middlePosition).Magnitude
                    if dist <= killRadius and hrp.Position.Y < 60 then
                        if not plr.Team or not table.find(flagKillWhitelistTeams, plr.Team.Name) then
                            for _, part in ipairs(char:GetChildren()) do
                                if part:IsA("BasePart") then
                                    tool:FindFirstChild("Use"):FireServer()
                                    firetouchinterest(toolHandle, part, 0)
                                    firetouchinterest(toolHandle, part, 1)
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end

local ToggleSwordKill = Tab:CreateToggle({
    Name = "Sword Kill",
    CurrentValue = false,
    Flag = "Toggle1",
    Callback = function(Value)
        swordKillEnabled = Value
        if Value then
            spawn(function()
                while swordKillEnabled do
                    pcall(function()
                        SwordKillLoop()
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end,
})

local ToggleAutoTP = Tab:CreateToggle({
    Name = "Auto Hide",
    CurrentValue = false,
    Flag = "ToggleAutoTP",
    Callback = function(Value)
        autoTPEnabled = Value
        if Value then
            spawn(AutoTeleportToTargets)
        end
    end,
})

local ToggleAutoEquip = Tab:CreateToggle({
    Name = "Auto Equip Sword",
    CurrentValue = false,
    Flag = "ToggleAutoEquip",
    Callback = function(Value)
        autoEquipEnabled = Value
        if Value then
            spawn(AutoEquipSword)
        end
    end,
})

local ToggleKillInArea = Tab:CreateToggle({
    Name = "Kill Players Near Flag",
    CurrentValue = false,
    Flag = "ToggleKillInArea",
    Callback = function(Value)
        killInAreaEnabled = Value
        if Value then
            spawn(function()
                while killInAreaEnabled do
                    pcall(function()
                        KillPlayersInArea()
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end,
})

local circle = workspace:WaitForChild("Koth"):WaitForChild("Circle")

local kothHealingPlatform = Instance.new("Part")
kothHealingPlatform.Name = "UnderCirclePlatform"
kothHealingPlatform.Anchored = true
kothHealingPlatform.Size = Vector3.new(50, 1, 50)
kothHealingPlatform.Position = circle.Position - Vector3.new(0, 25, 0)
kothHealingPlatform.Parent = workspace
kothHealingPlatform.CanCollide = true
kothHealingPlatform.Material = Enum.Material.ForceField
kothHealingPlatform.BrickColor = BrickColor.new("Bright red")

local KothMode = false
local KothLoopRunning = false

RunService.Stepped:Connect(function()
    if not KothMode then return end
    local char = Players.LocalPlayer.Character
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end)

local function randomAboveCircle()
    local hrp = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local offset = Vector3.new(
        math.random(-10, 10),
        math.random(4, 20),
        math.random(-10, 10)
    )
    hrp.CFrame = CFrame.new(circle.Position + offset)
end

local function hideUnderCircle()
    local hrp = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    hrp.CFrame = CFrame.new(kothHealingPlatform.Position + Vector3.new(0, 3, 0))
end

local function KothLoop()
    if KothLoopRunning then return end
    KothLoopRunning = true

    local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        KothLoopRunning = false
        return
    end

    local healing = false

    while KothMode do
        pcall(function()
            if humanoid.Health < 70 and not healing then
                healing = true
                hideUnderCircle()
                repeat
                    wait()
                    humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                    if not humanoid then break end
                until (humanoid and humanoid.Health >= humanoid.MaxHealth) or not KothMode
                healing = false
            end
    
            if not healing then
                randomAboveCircle()
            end
        end)
        task.wait()
    end

    KothLoopRunning = false
end

local KothToggle = Tab:CreateToggle({
    Name = "Steal Flag",
    CurrentValue = false,
    Flag = "ToggleKothAutoTP",
    Callback = function(Value)
        KothMode = Value
        if Value then
            spawn(KothLoop)
        end
    end,
})

local enhfeefefe = Tab:CreateButton({
    Name = "Get All Guns",
    Callback = function()
        local lp = Players.LocalPlayer
        local char = lp.Character or lp.CharacterAdded:Wait()

        local function isToolGiverRelated(part)
            local parent = part.Parent
            if parent and parent.Name:match("^ToolGiver") then
                return true
            elseif parent and parent.Parent and parent.Parent.Name:match("^ToolGiver") then
                return true
            end
            return false
        end

        local function fireTouch(part)
            if not part:IsA("BasePart") then return end
            if not isToolGiverRelated(part) then return end

            for _, ti in ipairs(part:GetChildren()) do
                if ti:IsA("TouchTransmitter") then
                    for _, myPart in ipairs(char:GetDescendants()) do
                        if myPart:IsA("BasePart") then
                            firetouchinterest(myPart, part, 0)
                            firetouchinterest(myPart, part, 1)
                        end
                    end
                end
            end
        end

        for _, v in ipairs(workspace:GetDescendants()) do
            fireTouch(v)
        end
    end,
})
