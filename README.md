--=====================================================
--  TROLL SX MODE | BROKENHAVEN RP
--  Cola a arma no quadril (pelvis) do personagem
--=====================================================

-- ===== CONFIGURAÇÕES =====
_G.TrollSXMode = true
_G.SelectedWeapon = "Glock" 
-- exemplos comuns: "Glock", "AK", "Shotgun", "Pistol"

-- ===== SERVIÇOS =====
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- ===== FUNÇÃO: pegar arma pelo nome =====
local function getWeaponByName(name)
    local backpack = player:WaitForChild("Backpack")
    for _, tool in pairs(backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == name then
            return tool
        end
    end
    return nil
end

-- ===== FUNÇÃO: limpar welds antigos =====
local function clearWelds(handle)
    for _, v in pairs(handle:GetChildren()) do
        if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
            v:Destroy()
        end
    end
end

-- ===== FUNÇÃO PRINCIPAL: colar arma no quadril =====
local function attachWeaponToPelvis(tool)
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    local pelvis =
        char:FindFirstChild("LowerTorso")
        or char:FindFirstChild("HumanoidRootPart")
        or char:FindFirstChild("Torso")

    if not pelvis then
        warn("Pelvis não encontrada")
        return
    end

    local handle = tool:WaitForChild("Handle")

    -- equipa a arma
    humanoid:EquipTool(tool)
    task.wait(0.1)

    -- remove weld padrão da mão
    clearWelds(handle)

    handle.Anchored = false
    handle.CanCollide = false
    handle.Massless = true

    -- cria weld novo
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = pelvis
    weld.Part1 = handle
    weld.Parent = handle

    -- posição da arma no quadril (AJUSTÁVEL)
    handle.CFrame =
        pelvis.CFrame
        * CFrame.new(0, -0.9, -0.6) -- altura + avanço
        * CFrame.Angles(0, math.rad(180), 0) -- apontada pra frente
end

-- ===== ATIVAÇÃO =====
local function activateTrollSX()
    if not _G.TrollSXMode then return end

    local weapon = getWeaponByName(_G.SelectedWeapon)
    if weapon then
        attachWeaponToPelvis(weapon)
    else
        warn("Arma não encontrada no inventário: "..tostring(_G.SelectedWeapon))
    end
end

-- ===== AUTO REAPLICAR APÓS RESPAWN =====
player.CharacterAdded:Connect(function()
    task.wait(1)
    activateTrollSX()
end)

-- ===== EXECUTAR =====
activateTrollSX()
