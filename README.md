-- Serviços
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

-- Variáveis
local isNight = false
local canAttack = true
local cooldown = 2
local knife = nil

-- Criar GUI notificação
local ScreenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
ScreenGui.Name = "NeggarNotification"

local function createNotification(text, duration)
    local notifFrame = Instance.new("Frame", ScreenGui)
    notifFrame.Size = UDim2.new(0, 300, 0, 50)
    notifFrame.Position = UDim2.new(1, -310, 0, 10)
    notifFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    notifFrame.BackgroundTransparency = 0.5
    notifFrame.BorderSizePixel = 0
    notifFrame.AnchorPoint = Vector2.new(1, 0)

    local notifText = Instance.new("TextLabel", notifFrame)
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.TextColor3 = Color3.fromRGB(0, 255, 0)
    notifText.TextStrokeTransparency = 0
    notifText.Text = text
    notifText.Font = Enum.Font.GothamBold
    notifText.TextScaled = true

    notifFrame.Visible = true
    TweenService:Create(notifFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0}):Play()
    wait(duration or 3)
    TweenService:Create(notifFrame, TweenInfo.new(0.5), {BackgroundTransparency = 0.5}):Play()
    wait(0.5)
    notifFrame:Destroy()
end

-- Notificação inicial
spawn(function()
    createNotification("Script Neggar carregado com sucesso!", 4)
end)

-- Cria faca estilizada
local function createKnife()
    local knife = Instance.new("Tool")
    knife.Name = "NeggarKnife"
    knife.RequiresHandle = true
    knife.CanBeDropped = false

    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(1, 0.3, 5) -- Hitbox grande
    handle.BrickColor = BrickColor.new("Really black")
    handle.Material = Enum.Material.Neon
    handle.Anchored = false
    handle.CanCollide = false
    handle.Parent = knife

    local mesh = Instance.new("SpecialMesh", handle)
    mesh.MeshType = Enum.MeshType.FileMesh
    mesh.MeshId = "rbxassetid://4763727850" -- Exemplo mesh de faca estilizada
    mesh.TextureId = "rbxassetid://4763728101"
    mesh.Scale = Vector3.new(1.5, 1.5, 1.5)

    knife.GripForward = Vector3.new(0, 0, -1)
    knife.GripPos = Vector3.new(0, 0, 0)
    knife.GripRight = Vector3.new(1, 0, 0)
    knife.GripUp = Vector3.new(0, 1, 0)

    return knife
end

-- Spawn 3 noobs pretos
local function spawnNoobs()
    for i = 1, 3 do
        local noob = Instance.new("Part", workspace)
        noob.Size = Vector3.new(2, 5, 1)
        noob.Position = hrp.Position + Vector3.new(math.random(-10, 10), 0, math.random(-10, 10))
        noob.BrickColor = BrickColor.new("Really black")
        noob.Anchored = false
        noob.Name = "NeggarNoob"

        local humanoid = Instance.new("Humanoid", noob)
        humanoid.Health = 100
        humanoid.MaxHealth = 100
    end
end

-- Aplica invisibilidade e indetectabilidade
local function applyInvisibility()
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = 0.7
            part.CanCollide = false
        end
    end
    hum.WalkSpeed = 30
end

local function removeInvisibility()
    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
            part.CanCollide = true
        end
    end
    hum.WalkSpeed = 16
end

-- Ataque da faca
local function attack()
    if not canAttack or not knife then return end
    canAttack = false

    local rayOrigin = hrp.Position
    local rayDirection = hrp.CFrame.LookVector * 5
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {char}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    local rayResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)

    if rayResult and rayResult.Instance and rayResult.Instance.Parent then
        local targetHumanoid = rayResult.Instance.Parent:FindFirstChild("Humanoid")
        if targetHumanoid and targetHumanoid.Health > 0 then
            if isNight then
                targetHumanoid.Health = 0
                createNotification("Inimigo eliminado instantaneamente!", 2)
            else
                targetHumanoid:TakeDamage(20)
            end
        end
    end

    wait(cooldown)
    canAttack = true
end

-- Detecta noite e aplica buff
Lighting:GetPropertyChangedSignal("ClockTime"):Connect(function()
    local time = Lighting.ClockTime
    local wasNight = isNight
    isNight = time >= 18 or time <= 6

    if isNight and not wasNight then
        applyInvisibility()

        if not player.Backpack:FindFirstChild("NeggarKnife") and not char:FindFirstChild("NeggarKnife") then
            knife = createKnife()
            knife.Parent = player.Backpack
        end

        spawnNoobs()
        createNotification("Instintos do Neggar ativados", 3)

        StarterGui:SetCore("ChatMakeSystemMessage", {
            Text = player.Name .. " diz: neggars neggars";
            Color = Color3.fromRGB(0, 255, 0);
            Font = Enum.Font.GothamBold;
            FontSize = Enum.FontSize.Size24;
        })

    elseif not isNight and wasNight then
        removeInvisibility()

        if char:FindFirstChild("NeggarKnife") then
            char.NeggarKnife:Destroy()
        elseif player.Backpack:FindFirstChild("NeggarKnife") then
            player.Backpack.NeggarKnife:Destroy()
        end
    end
end)

-- Conecta ataque ao clique do mouse
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if char:FindFirstChildOfClass("Tool") and char:FindFirstChildOfClass("Tool").Name == "NeggarKnife" then
            attack()
        end
    end
end)
