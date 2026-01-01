local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

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
local COOLDOWN = 0.03 -- 30ms

-- MODOS
local legionAtivo = false
local modoReducao = false

-- RANGES
local RANGE = 27
local RANGE_T = 30

-- LISTA DE MODELOS ACEITOS
local NOMES_ACEITOS = {
    ["Airhorn"] = true, ["BallBasketball"] = true, ["BallMagicLight"] = true, ["BalloonModel"] = true,
    ["BellSmall"] = true, ["BombDarkMatter"] = true, ["BombMissile"] = true, ["BoxCrateWood"] = true,
    ["BubbleBlower"] = true, ["BucketPaint"] = true, ["Campfire"] = true, ["ChildrensTable"] = true,
    ["ChildrensTableBench"] = true, ["ClockAlarm"] = true, ["CupMugBrown"] = true, ["CupMugWhite"] = true,
    ["DiceBig"] = true, ["FoodPizzaCheese"] = true, ["FoodPizzaPepperoni"] = true, ["FoodPlate"] = true,
    ["FoodSodaCan"] = true, ["InstrumentBrassTrumpet"] = true, ["InstrumentBrassVuvuzela"] = true,
    ["InstrumentDrumBongos"] = true, ["InstrumentDrumSnare"] = true, ["InstrumentPianoKeyboard"] = true,
    ["NinjaKatana"] = true, ["NinjaKunai"] = true, ["NinjaShuriken"] = true, ["ToolCleaver"] = true,
    ["ToolDiggingForkRusty"] = true
}

-- LISTA DE ESPECIAIS
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

-- Atualizar ranges
local function atualizarRanges()
    if legionAtivo then
        if modoReducao then
            RANGE = 20
            RANGE_T = 20
        else
            RANGE = 30
            RANGE_T = 30
        end
    else
        if modoReducao then
            RANGE = 17
            RANGE_T = 20
        else
            RANGE = 27
            RANGE_T = 30
        end
    end
end

-- Detectar alvo na mira
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
        local playerCheck = Players:GetPlayerFromCharacter(model)
        if playerCheck then
            if legionAtivo then
                if hit:IsA("BasePart") and hit:IsDescendantOf(model) then
                    return model
                end
            else
                local torso = model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso")
                local leftArm = model:FindFirstChild("Left Arm") or model:FindFirstChild("LeftUpperArm")
                local rightArm = model:FindFirstChild("Right Arm") or model:FindFirstChild("RightUpperArm")
                if (torso and (hit == torso or hit:IsDescendantOf(torso))) or
                   (leftArm and (hit == leftArm or hit:IsDescendantOf(leftArm))) or
                   (rightArm and (hit == rightArm or hit:IsDescendantOf(rightArm))) then
                    return model
                end
            end
        else
            return model
        end
    end

    if NOMES_ACEITOS[model.Name] then
        return model
    end

    return nil
end

-- Detectar alvo especial
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

-- Clique usando mouse1click com cooldown
local function click()
    local agora = tick()
    if agora - ultimoClique < COOLDOWN then return end
    ultimoClique = agora
    mouse1click() -- substitui VirtualUser
end

-- Tentar clicar
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

    if input.KeyCode == Enum.KeyCode.H then
        modoReducao = not modoReducao
        atualizarRanges()
        print("Modo Redução:", modoReducao and "Ativado (Ranges 17/20)" or "Desativado (Ranges 27/30)")
    elseif input.KeyCode == Enum.KeyCode.L then
        legionAtivo = not legionAtivo
        atualizarRanges()
        print("Legion Destroyer:", legionAtivo and (modoReducao and "Ativado (Ranges 20/20)" or "Ativado (Ranges 30/30)") or "Desativado")
    elseif input.KeyCode == Enum.KeyCode.R then
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

-- Inicializa ranges
atualizarRanges()
