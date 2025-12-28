_G.TrollSXMode = true
_G.SelectedWeapon = "Glock" -- MUDE PARA O NOME EXATO DA ARMA

local Players = game:GetService("Players")
local player = Players.LocalPlayer

local function getWeaponByName(name)
    for _, tool in pairs(player.Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == name then
            return tool
        end
    end
end

local function clearWelds(handle)
    for _, v in pairs(handle:GetChildren()) do
        if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
            v:Destroy()
        end
    end
end

local function attachWeaponToPelvis(tool)
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

local function run()
    local weapon = getWeaponByName(_G.SelectedWeapon)
    if weapon then
        attachWeaponToPelvis(weapon)
        warn("TROLL SX MODE ATIVADO")
    else
        warn("ARMA NÃO ENCONTRADA NO BACKPACK")
    end
end

player.CharacterAdded:Connect(function()
    task.wait(1)
    run()
end)

run()
