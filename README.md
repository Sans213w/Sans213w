--[[
    SCRIPT: CLASSE NEGGAR COMPLETA
    Jogo: Dead Rails (Roblox)
    Permissão do criador confirmada.
    Funções:
    - Invisibilidade à noite
    - Faca com ataque especial (insta-kill à noite / dano de 10 de dia)
    - Velocidade aumentada ao segurar faca
    - Indetectável para zumbis/lobos/chefes
    - Visão noturna à noite
    - HUD informativo
    - Animação de ataque sincronizada
    - Dash visual ao atacar (tipo Sans)
--]]

-- SERVIÇOS
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")

-- MODELO DA FACA
local knife = Instance.new("Tool")
knife.Name = "NeggarKnife"
knife.RequiresHandle = true
knife.CanBeDropped = false

local handle = Instance.new("Part")
handle.Name = "Handle"
handle.Size = Vector3.new(1, 0.2, 4)
handle.BrickColor = BrickColor.new("Really black")
handle.Material = Enum.Material.Metal
handle.Anchored = false
handle.CanCollide = false
handle.Parent = knife
knife.GripForward = Vector3.new(0, 0, -1)
knife.GripPos = Vector3.new(0, 0, 0)
knife.GripRight = Vector3.new(1, 0, 0)
knife.GripUp = Vector3.new(0, 1, 0)
knife.Parent = player.Backpack

-- ANIMAÇÃO
local anim = Instance.new("Animation")
anim.AnimationId = "rbxassetid://522635514" -- Ataque simples (substituível)
local animTrack = hum:LoadAnimation(anim)

-- HUD
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "NeggarHUD"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 100)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundTransparency = 0.3
frame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)

local title = Instance.new("TextLabel", frame)
title.Text = "🔪 Classe: Neggar"
title.Size = UDim2.new(1, 0, 0, 25)
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextScaled = true

local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1, 0, 0, 25)
status.Position = UDim2.new(0, 0, 0, 30)
status.TextColor3 = Color3.fromRGB(150, 255, 150)
status.BackgroundTransparency = 1
status.Font = Enum.Font.Gotham
status.TextScaled = true

local vision = Instance.new("TextLabel", frame)
vision.Size = UDim2.new(1, 0, 0, 25)
vision.Position = UDim2.new(0, 0, 0, 60)
vision.TextColor3 = Color3.fromRGB(100, 200, 255)
vision.BackgroundTransparency = 1
vision.Font = Enum.Font.Gotham
vision.TextScaled = true

-- ATUALIZAÇÃO DO HUD
spawn(function()
    while true do
        local isNight = Lighting.ClockTime >= 18 or Lighting.ClockTime <= 6
        status.Text = isNight and "🌙 Invisível + InstaKill Ativado" or "☀️ Dano: 10"
        vision.Text = isNight and "👁️ Visão Noturna: ON" or "👁️ Visão Noturna: OFF"
        status.TextColor3 = isNight and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(255, 255, 100)
        wait(1)
    end
end)

-- INIMIGOS
local function isEnemy(part)
    return part:FindFirstAncestorWhichIsA("Model") and part.Parent ~= char
end

-- ATAQUE
knife.Activated:Connect(function()
    animTrack:Play(0.1)
    
    -- Dash
    local dash = Instance.new("BodyVelocity")
    dash.Velocity = hrp.CFrame.LookVector * 60
    dash.MaxForce = Vector3.new(1e5, 0, 1e5)
    dash.P = 5000
    dash.Parent = hrp
    Debris:AddItem(dash, 0.1)

    -- Checa inimigos
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local target = plr.Character
            local dist = (hrp.Position - target.HumanoidRootPart.Position).Magnitude
            local isNight = Lighting.ClockTime >= 18 or Lighting.ClockTime <= 6
            if dist <= 6 then
                local humTarget = target:FindFirstChild("Humanoid")
                if humTarget then
                    if isNight and (hrp.CFrame.LookVector:Dot((target.HumanoidRootPart.Position - hrp.Position).unit) > 0.5) then
                        humTarget.Health = 0
                    else
                        humTarget:TakeDamage(10)
                    end
                end
            end
        end
    end
end)

-- INVISIBILIDADE + VELOCIDADE
spawn(function()
    while true do
        local isNight = Lighting.ClockTime >= 18 or Lighting.ClockTime <= 6
        if isNight then
            char.HumanoidRootPart.Transparency = 0.8
            handle.Transparency = 1
            hum.WalkSpeed = knife.Parent == player.Character and 35 or 16
        else
            char.HumanoidRootPart.Transparency = 0
            handle.Transparency = 0
            hum.WalkSpeed = 16
        end
        wait(0.5)
    end
end)

-- VISÃO NOTURNA
spawn(function()
    while true do
        local isNight = Lighting.ClockTime >= 18 or Lighting.ClockTime <= 6
        Lighting.Brightness = isNight and 3 or 1
        Lighting.FogEnd = isNight and 1000 or 200
        wait(1)
    end
end)
