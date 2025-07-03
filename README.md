-- Dandy's World Utility Script
-- Funções: ESP (Twisteds, Itens, Time), Full Bright, Inf Walk Speed, Auto Skill Check

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "DWScriptGui"

local function makeButton(name, pos)
    local b = Instance.new("TextButton")
    b.Name = name
    b.Text = name
    b.Size = UDim2.new(0,120,0,30)
    b.Position = UDim2.new(0,pos.X,0,pos.Y)
    b.Parent = ScreenGui
    b.BackgroundColor3 = Color3.fromRGB(40,40,40)
    b.TextColor3 = Color3.new(1,1,1)
    b.BorderSizePixel = 0
    return b
end

local buttons = {
    ESPTwisteds = makeButton("ESP Twisteds", Vector2.new(10,10)),
    ESPItens = makeButton("ESP Itens", Vector2.new(10,45)),
    ESPTime = makeButton("ESP Time", Vector2.new(10,80)),
    FullBright = makeButton("Full Bright", Vector2.new(10,115)),
    InfWalkSpeed = makeButton("Inf WalkSpeed", Vector2.new(10,150)),
    AutoSkillCheck = makeButton("Auto SkillCheck", Vector2.new(10,185)),
}

-- ESP Utility
local espConnections = {}
local function clearESP(tag)
    if espConnections[tag] then
        for _,v in ipairs(espConnections[tag]) do v:Destroy() end
    end
    espConnections[tag] = {}
end
local function addESP(instances, color, tag)
    clearESP(tag)
    espConnections[tag] = {}
    for _,inst in ipairs(instances) do
        if inst:IsA("Model") or inst:IsA("Part") then
            local box = Instance.new("BoxHandleAdornment")
            box.Adornee = inst:IsA("Model") and inst.PrimaryPart or inst
            box.AlwaysOnTop = true
            box.Size = (inst:IsA("Model") and inst.PrimaryPart and inst.PrimaryPart.Size) or inst.Size or Vector3.new(2,5,2)
            box.Color3 = color
            box.Transparency = 0.3
            box.ZIndex = 5
            box.Parent = ScreenGui
            table.insert(espConnections[tag], box)
        end
    end
end

-- ESP Buttons
local twistedsESPEnabled, itensESPEnabled, timeESPEnabled = false, false, false
local function getTwisteds()
    local res = {}
    for _,v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and v:FindFirstChild("TwistedId") then
            if v.PrimaryPart then table.insert(res, v) end
        end
    end
    return res
end
local function getItens()
    local res = {}
    for _,v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Part") and (v.Name:lower():find("item") or v.Name:lower():find("key") or v.Name:lower():find("piece")) then
            table.insert(res, v)
        end
    end
    return res
end
local function getTeamPlayers()
    local res = {}
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.Team == LocalPlayer.Team and plr ~= LocalPlayer then
            local char = plr.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                table.insert(res, char)
            end
        end
    end
    return res
end

buttons.ESPTwisteds.MouseButton1Click:Connect(function()
    twistedsESPEnabled = not twistedsESPEnabled
    if twistedsESPEnabled then
        addESP(getTwisteds(), Color3.new(1,0,0), "twisteds")
    else
        clearESP("twisteds")
    end
end)
buttons.ESPItens.MouseButton1Click:Connect(function()
    itensESPEnabled = not itensESPEnabled
    if itensESPEnabled then
        addESP(getItens(), Color3.new(1,1,0), "itens")
    else
        clearESP("itens")
    end
end)
buttons.ESPTime.MouseButton1Click:Connect(function()
    timeESPEnabled = not timeESPEnabled
    if timeESPEnabled then
        addESP(getTeamPlayers(), Color3.new(0,1,1), "team")
    else
        clearESP("team")
    end
end)
RunService.RenderStepped:Connect(function()
    if twistedsESPEnabled then addESP(getTwisteds(), Color3.new(1,0,0), "twisteds") end
    if itensESPEnabled then addESP(getItens(), Color3.new(1,1,0), "itens") end
    if timeESPEnabled then addESP(getTeamPlayers(), Color3.new(0,1,1), "team") end
end)

-- Full Bright
local fullBrightConn
local fullBrightEnabled = false
local function setFullBright(on)
    if on then
        Lighting.Brightness = 5
        Lighting.Ambient = Color3.new(1,1,1)
        Lighting.OutdoorAmbient = Color3.new(1,1,1)
        Lighting.FogEnd = 1e10
        Lighting.FogStart = 0
        Lighting.FogColor = Color3.new(1,1,1)
        if not fullBrightConn then
            fullBrightConn = Lighting.Changed:Connect(function()
                if fullBrightEnabled then
                    setFullBright(true)
                end
            end)
        end
    else
        if fullBrightConn then fullBrightConn:Disconnect() end
        fullBrightConn = nil
        -- Não restaura valores originais para manter o script limpo
    end
end
buttons.FullBright.MouseButton1Click:Connect(function()
    fullBrightEnabled = not fullBrightEnabled
    setFullBright(fullBrightEnabled)
end)

-- Inf Walk Speed
local walkSpeedEnabled = false
local function setWalkSpeed(on)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = on and 35 or 16
    end
end
buttons.InfWalkSpeed.MouseButton1Click:Connect(function()
    walkSpeedEnabled = not walkSpeedEnabled
    setWalkSpeed(walkSpeedEnabled)
end)
LocalPlayer.CharacterAdded:Connect(function()
    if walkSpeedEnabled then setWalkSpeed(true) end
end)

-- Auto Skill Check
local autoSkillEnabled = false
local skillConn
local function onSkillCheck()
    for _,desc in ipairs(workspace:GetDescendants()) do
        if desc.Name:lower():find("skill") and desc:IsA("Frame") and desc.Visible then
            local bar = desc:FindFirstChild("Bar")
            local yellow = desc:FindFirstChild("Yellow")
            local cursor = desc:FindFirstChild("Cursor")
            if bar and yellow and cursor then
                local function getPos(gui)
                    local abs = gui.AbsolutePosition.X
                    local size = gui.AbsoluteSize.X
                    return abs, abs + size
                end
                local yellowStart, yellowEnd = getPos(yellow)
                local function tryPress()
                    local cursorX = cursor.AbsolutePosition.X
                    if cursorX >= yellowStart and cursorX <= yellowEnd then
                        game:GetService("VirtualInputManager"):SendKeyEvent(true, Enum.KeyCode.E, false, game)
                    end
                end
                local conn = RunService.RenderStepped:Connect(tryPress)
                table.insert(espConnections, conn)
            end
        end
    end
end
buttons.AutoSkillCheck.MouseButton1Click:Connect(function()
    autoSkillEnabled = not autoSkillEnabled
    if autoSkillEnabled then
        skillConn = RunService.RenderStepped:Connect(onSkillCheck)
    else
        if skillConn then skillConn:Disconnect() end
        skillConn = nil
    end
end)
