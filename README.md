local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

local dodgeDistance = 10
local isDodging = false

-- Ajuste aqui para detectar se você é killer no seu jogo
local isKiller = false -- Mude para true se for killer

-- Função para checar se posição é segura (não colide com nada)
local function isPositionSafe(position)
    local regionSize = Vector3.new(3, 5, 3)
    local region = Region3.new(position - regionSize/2, position + regionSize/2)
    region = region:ExpandToGrid(4)

    local partsInRegion = Workspace:FindPartsInRegion3WithIgnoreList(region, {Character}, 10)
    for _, part in pairs(partsInRegion) do
        if part.CanCollide then
            return false
        end
    end
    return true
end

-- Função para desviar tipo Sans
local function dodge()
    if isDodging then return end
    isDodging = true

    local rootPos = HumanoidRootPart.Position
    local rootCFrame = HumanoidRootPart.CFrame

    local rightPos = rootPos + rootCFrame.RightVector * dodgeDistance
    local leftPos = rootPos - rootCFrame.RightVector * dodgeDistance

    local safePos = nil

    if isPositionSafe(rightPos) then
        safePos = rightPos
    elseif isPositionSafe(leftPos) then
        safePos = leftPos
    else
        safePos = rootPos -- Sem lugar seguro, fica parado
    end

    -- Raycast para achar chão e ajustar altura
    local rayOrigin = safePos + Vector3.new(0, 50, 0)
    local rayDirection = Vector3.new(0, -100, 0)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

    local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    if raycastResult then
        local finalPos = raycastResult.Position + Vector3.new(0, 5, 0)
        HumanoidRootPart.CFrame = CFrame.new(finalPos)
    else
        HumanoidRootPart.CFrame = CFrame.new(safePos + Vector3.new(0, 5, 0))
    end

    wait(1)
    isDodging = false
end

-- Função para teletransportar para o player mais próximo
local function teleportToNearestPlayer()
    if not isKiller then return end

    local closestPlayer = nil
    local closestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (player.Character.HumanoidRootPart.Position - HumanoidRootPart.Position).Magnitude
            if dist < closestDistance then
                closestDistance = dist
                closestPlayer = player
            end
        end
    end

    if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPos = closestPlayer.Character.HumanoidRootPart.Position
        HumanoidRootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 5, 0))
    end
end

-- Conectar entradas de teclado
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        dodge()
    elseif input.KeyCode == Enum.KeyCode.M then
        teleportToNearestPlayer()
    end
end)

-- Atualiza referência caso personagem morra e renasça
LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
end)
