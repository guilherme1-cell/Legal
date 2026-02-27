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

-- Detectar alvo
