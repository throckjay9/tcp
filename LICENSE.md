-- Script para PCX DO TG com Aimbot, No Recoil, ESP Linha e Box
-- ATENÇÃO: Use apenas em servidores privados com permissão

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Configurações
local AimbotEnabled = true
local ESPEnabled = true
local NoRecoilEnabled = true
local TeamCheck = true

-- Aimbot
local AimbotKey = Enum.UserInputType.MouseButton2
local ShootKey = Enum.UserInputType.MouseButton1
local Smoothness = 0.3
local FOV = 100
local TargetPart = "Head"

-- Cores ESP
local EnemyColor = Color3.fromRGB(255, 50, 50)
local TeamColor = Color3.fromRGB(50, 255, 50)

-- Função para encontrar o jogador mais próximo
local function FindClosestPlayer()
    local closestPlayer, closestDistance = nil, FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local head = player.Character:FindFirstChild("Head")
            
            if humanoid and humanoid.Health > 0 and head then
                if TeamCheck and player.Team == LocalPlayer.Team then
                    continue
                end
                
                local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    if distance < closestDistance then
                        closestPlayer = player
                        closestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

-- Função para suavizar a mira
local function SmoothAim(targetPosition)
    local currentPosition = Camera.CFrame.Position
    local direction = (targetPosition - currentPosition).Unit
    local newPosition = currentPosition + (direction * Smoothness * 10)
    Camera.CFrame = CFrame.new(newPosition, targetPosition)
end

-- No Recoil
local function ApplyNoRecoil()
    if NoRecoilEnabled and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.AutoRotate = false
            humanoid.CameraOffset = Vector3.new(0, 0, 0)
        end
    end
end

-- ESP
local ESPObjects = {}

local function CreateESP(player)
    if ESPObjects[player] then return end
    
    local character = player.Character
    if not character then return end
    
    ESPObjects[player] = {}
    
    -- Box ESP
    local box = Instance.new("BoxHandleAdornment")
    box.Name = player.Name .. "_Box"
    box.Adornee = character:WaitForChild("HumanoidRootPart")
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(3, 5, 1)
    box.Color3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
    box.Transparency = 0.5
    box.Parent = character
    table.insert(ESPObjects[player], box)
    
    -- Line ESP
    local line = Instance.new("LineHandleAdornment")
    line.Name = player.Name .. "_Line"
    line.Adornee = character:WaitForChild("HumanoidRootPart")
    line.AlwaysOnTop = true
    line.ZIndex = 10
    line.Length = (character:WaitForChild("HumanoidRootPart").Position.Y
    line.Thickness = 1
    line.Color3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
    line.Parent = character
    table.insert(ESPObjects[player], line)
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if obj then obj:Destroy() end
        end
        ESPObjects[player] = nil
    end
end

-- Conexões
local function OnPlayerAdded(player)
    player.CharacterAdded:Connect(function(character)
        if ESPEnabled then
            CreateESP(player)
        end
    end)
    
    player.CharacterRemoving:Connect(function()
        RemoveESP(player)
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    OnPlayerAdded(player)
end

Players.PlayerAdded:Connect(OnPlayerAdded)

-- Loop principal
RunService.RenderStepped:Connect(function()
    -- Aimbot quando atirando
    if AimbotEnabled and UserInputService:IsMouseButtonPressed(ShootKey) then
        local closestPlayer = FindClosestPlayer()
        if closestPlayer and closestPlayer.Character then
            local head = closestPlayer.Character:FindFirstChild(TargetPart)
            if head then
                SmoothAim(head.Position)
            end
        end
    end
    
    -- No Recoil
    if NoRecoilEnabled then
        ApplyNoRecoil()
    end
    
    -- Atualizar ESP
    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                CreateESP(player)
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            RemoveESP(player)
        end
    end
end)

print("Script ativado para PCX DO TG!")
print("Aimbot: Ativado ao atirar")
print("No Recoil: Ativado")
print("ESP: Caixa e Linha ativados")

-- GUI simples para ativar/desativar funções
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local ToggleFrame = Instance.new("Frame")
ToggleFrame.Size = UDim2.new(0, 150, 0, 100)
ToggleFrame.Position = UDim2.new(0, 10, 0, 10)
ToggleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ToggleFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 20)
Title.Text = "PCX DO TG Hacks"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Parent = ToggleFrame

local function CreateToggle(text, yPos, varName)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.9, 0, 0, 20)
    button.Position = UDim2.new(0.05, 0, yPos, 0)
    button.Text = text .. ": " .. (varName and "ON" or "OFF")
    button.TextColor3 = varName and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    button.Parent = ToggleFrame
    
    button.MouseButton1Click:Connect(function()
        _G[text] = not _G[text]
        button.Text = text .. ": " .. (_G[text] and "ON" or "OFF")
        button.TextColor3 = _G[text] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    end)
end

CreateToggle("AimbotEnabled", 0.25, AimbotEnabled)
CreateToggle("ESPEnabled", 0.5, ESPEnabled)
CreateToggle("NoRecoilEnabled", 0.75, NoRecoilEnabled)
