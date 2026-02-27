local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- ESTADOS
local last = nil
local pressR = false
local clickedR = false

local pressT = false
local clickedT = false
local lastEspecial = nil

local ultimoClique = 0
local COOLDOWN = 0.005 -- 5ms

-- RANGE fixo
local RANGE = 32
local RANGE_T = 32

-- ESPECIAIS: só pallet
local ESPECIAIS = {
    ["PalletLightBrown"] = true
}

-- Raycast params
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Blacklist
params.IgnoreWater = true

local function updateFilter()
    local char = LocalPlayer.Character
    if char then
        params.FilterDescendantsInstances = {char}
    else
        params.FilterDescendantsInstances = {}
    end
end

LocalPlayer.CharacterAdded:Connect(function()
    repeat task.wait() until LocalPlayer.Character
    updateFilter()
end)
updateFilter()

-- Detectar alvo humanoid inteiro
local function pegarAlvoNaMira()
    local origin = Camera.CFrame.Position
    local direction = Camera.CFrame.LookVector * RANGE
    local results = Workspace:Raycast(origin, direction, params)
    if not results then return nil end

    local hit = results.Instance
    if not hit then return nil end

    local model = hit:FindFirstAncestorOfClass("Model")
    if not model or model == LocalPlayer.Character then return nil end

    local hum = model:FindFirstChildWhichIsA("Humanoid")
    if hum and hum.Health > 0 then
        return model
    end

    return nil
end

-- Detectar alvo especial (pallet)
local function pegarEspecialNaMira()
    local origin = Camera.CFrame.Position
    local direction = Camera.CFrame.LookVector * RANGE_T
    local results = Workspace:Raycast(origin, direction, params)
    if not results then return nil end

    local hit = results.Instance
    if not hit then return nil end

    local model = hit:FindFirstAncestorOfClass("Model")
    if not model or model == LocalPlayer.Character then return nil end

    if ESPECIAIS[model.Name] then
        return model
    end

    return nil
end

-- Criar ou pegar BindableEvent
local clickEvent = ReplicatedStorage:FindFirstChild("ClickEvent") or Instance.new("BindableEvent")
clickEvent.Name = "ClickEvent"
clickEvent.Parent = ReplicatedStorage

-- Clique usando mouse1click com cooldown
local function click()
    local agora = tick()
    if agora - ultimoClique < COOLDOWN then return end
    ultimoClique = agora

    mouse1click() -- dispara clique real

    -- Dispara evento para overlay/escuta
    clickEvent:Fire()
end

-- Tentar clicar em humanoid
local function tentar()
    local alvo = pegarAlvoNaMira()
    if not alvo then
        last = nil
        return false
    end
    if alvo == last then return false end
    last = alvo
    click()
    return true
end

-- Tentar clicar em pallet
local function tentarEspecial()
    local alvo = pegarEspecialNaMira()
    if not alvo then
        lastEspecial = nil
        return false
    end
    if alvo == lastEspecial then return false end
    lastEspecial = alvo
    click()
    return true
end

-- Loop principal
RunService.Heartbeat:Connect(function()
    if pressR and not clickedR then
        if tentar() then
            clickedR = true
        end
    end
    if pressT and not clickedT then
        if tentarEspecial() then
            clickedT = true
        end
    end
end)

-- Input
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end

    if input.KeyCode == Enum.KeyCode.R then
        pressR = true
        clickedR = false
    elseif input.KeyCode == Enum.KeyCode.T then
        pressT = true
        clickedT = false
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.R then
        pressR = false
        clickedR = false
    elseif input.KeyCode == Enum.KeyCode.T then
        pressT = false
        clickedT = false
    end
end)
