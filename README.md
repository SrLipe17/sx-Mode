--====================================================
--  SX MODE HUB | BROKENHAVEN RP
--  UI + Troll Mode (arma colada no quadril)
--====================================================

-- ===== CONFIG =====
local SX_ENABLED = false
local SELECTED_WEAPON = nil

-- ===== SERVICES =====
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ===== FUNÇÕES =====
local function getWeapons()
    local list = {}
    for _,v in pairs(player.Backpack:GetChildren()) do
        if v:IsA("Tool") then
            table.insert(list, v.Name)
        end
    end
    return list
end

local function clearWelds(handle)
    for _,v in pairs(handle:GetChildren()) do
        if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
            v:Destroy()
        end
    end
end

local function attachWeapon(tool)
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")
    local pelvis = char:FindFirstChild("LowerTorso") or char:FindFirstChild("HumanoidRootPart")
    if not pelvis then return end

    humanoid:EquipTool(tool)
    task.wait(0.1)

    local handle = tool:WaitForChild("Handle")
    clearWelds(handle)

    handle.Anchored = false
    handle.CanCollide = false
    handle.Massless = true

    local weld = Instance.new("WeldConstraint")
    weld.Part0 = pelvis
    weld.Part1 = handle
    weld.Parent = handle

    handle.CFrame =
        pelvis.CFrame
        * CFrame.new(0, -0.9, -0.6)
        * CFrame.Angles(0, math.rad(180), 0)
end

local function runSX()
    if not SX_ENABLED then return end
    if not SELECTED_WEAPON then return end

    local tool = player.Backpack:FindFirstChild(SELECTED_WEAPON)
    if tool then
        attachWeapon(tool)
    end
end

-- ===== UI =====
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "SXModeHub"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 260)
frame.Position = UDim2.new(0.5, -150, 0.5, -130)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,40)
title.Text = "SX MODE HUB"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20

-- Toggle Button
local toggle = Instance.new("TextButton", frame)
toggle.Size = UDim2.new(0.8,0,0,40)
toggle.Position = UDim2.new(0.1,0,0,60)
toggle.Text = "SX MODE: OFF"
toggle.BackgroundColor3 = Color3.fromRGB(60,60,60)
toggle.TextColor3 = Color3.new(1,1,1)
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 16

toggle.MouseButton1Click:Connect(function()
    SX_ENABLED = not SX_ENABLED
    toggle.Text = SX_ENABLED and "SX MODE: ON" or "SX MODE: OFF"
    toggle.BackgroundColor3 = SX_ENABLED and Color3.fromRGB(0,170,0) or Color3.fromRGB(60,60,60)
    runSX()
end)

-- Dropdown
local dropdown = Instance.new("TextButton", frame)
dropdown.Size = UDim2.new(0.8,0,0,40)
dropdown.Position = UDim2.new(0.1,0,0,110)
dropdown.Text = "Selecionar Arma"
dropdown.BackgroundColor3 = Color3.fromRGB(50,50,50)
dropdown.TextColor3 = Color3.new(1,1,1)
dropdown.Font = Enum.Font.Gotham
dropdown.TextSize = 14

local listFrame = Instance.new("Frame", frame)
listFrame.Size = UDim2.new(0.8,0,0,80)
listFrame.Position = UDim2.new(0.1,0,0,155)
listFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
listFrame.Visible = false

local layout = Instance.new("UIListLayout", listFrame)

dropdown.MouseButton1Click:Connect(function()
    listFrame.Visible = not listFrame.Visible
    listFrame:ClearAllChildren()
    layout.Parent = listFrame

    for _,name in pairs(getWeapons()) do
        local btn = Instance.new("TextButton", listFrame)
        btn.Size = UDim2.new(1,0,0,30)
        btn.Text = name
        btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 14

        btn.MouseButton1Click:Connect(function()
            SELECTED_WEAPON = name
            dropdown.Text = "Arma: "..name
            listFrame.Visible = false
            runSX()
        end)
    end
end)

-- Reaplicar após respawn
player.CharacterAdded:Connect(function()
    task.wait(1)
    runSX()
end)

-- Tecla para esconder UI (RightShift)
UIS.InputBegan:Connect(function(i,g)
    if not g and i.KeyCode == Enum.KeyCode.RightShift then
        frame.Visible = not frame.Visible
    end
end)
