-- Script para PCX DO TG com Aimbot, No Recoil, ESP Linha e Box
-- ATENÇÃO: Use apenas em servidores privados com permissão

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Configurações
local settings = {
    Aimbot = {
        Enabled = true,
        Key = Enum.UserInputType.MouseButton2,
        Smoothness = 0.25,
        FOV = 90,
        TargetPart = "Head",
        VisibleCheck = true
    },
    ESP = {
        Enabled = true,
        Box = true,
        Line = true,
        TeamCheck = true,
        TeamColor = Color3.fromRGB(0, 255, 0),
        EnemyColor = Color3.fromRGB(255, 50, 50)
    },
    NoRecoil = {
        Enabled = true
    },
    GUI = {
        Key = Enum.KeyCode.F,
        Visible = false
    }
}

-- Variáveis
local espObjects = {}
local connections = {}

-- Função para criar a GUI
local function CreateGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PCXHackGUI"
    ScreenGui.Parent = game.CoreGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 250, 0, 300)
    MainFrame.Position = UDim2.new(0.5, -125, 0.5, -150)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    MainFrame.BackgroundTransparency = 0.2
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = settings.GUI.Visible
    MainFrame.Parent = ScreenGui

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = MainFrame

    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Size = UDim2.new(1, 0, 0, 40)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    Title.BackgroundTransparency = 0.3
    Title.Text = "PCX DO TG HACKS"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16
    Title.Parent = MainFrame

    local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 6)
    TitleCorner.Parent = Title

    local function CreateToggle(text, posY, configTable, configKey)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
        ToggleFrame.Position = UDim2.new(0.05, 0, posY, 0)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = MainFrame

        local ToggleText = Instance.new("TextLabel")
        ToggleText.Size = UDim2.new(0.7, 0, 1, 0)
        ToggleText.Position = UDim2.new(0, 0, 0, 0)
        ToggleText.BackgroundTransparency = 1
        ToggleText.Text = text
        ToggleText.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleText.Font = Enum.Font.Gotham
        ToggleText.TextXAlignment = Enum.TextXAlignment.Left
        ToggleText.Parent = ToggleFrame

        local ToggleButton = Instance.new("TextButton")
        ToggleButton.Size = UDim2.new(0.25, 0, 0.8, 0)
        ToggleButton.Position = UDim2.new(0.75, 0, 0.1, 0)
        ToggleButton.Text = configTable[configKey] and "ON" or "OFF"
        ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleButton.BackgroundColor3 = configTable[configKey] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
        ToggleButton.Font = Enum.Font.GothamBold
        ToggleButton.Parent = ToggleFrame

        local ButtonCorner = Instance.new("UICorner")
        ButtonCorner.CornerRadius = UDim.new(0, 4)
        ButtonCorner.Parent = ToggleButton

        ToggleButton.MouseButton1Click:Connect(function()
            configTable[configKey] = not configTable[configKey]
            ToggleButton.Text = configTable[configKey] and "ON" or "OFF"
            ToggleButton.BackgroundColor3 = configTable[configKey] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
            
            -- Atualizar ESP se necessário
            if text == "ESP" then
                if not configTable[configKey] then
                    for _, player in pairs(Players:GetPlayers()) do
                        if player ~= LocalPlayer then
                            RemoveESP(player)
                        end
                    end
                end
            end
        end)
    end

    -- Criar toggles
    CreateToggle("Aimbot", 0.15, settings.Aimbot, "Enabled")
    CreateToggle("ESP", 0.25, settings.ESP, "Enabled")
    CreateToggle("No Recoil", 0.35, settings.NoRecoil, "Enabled")
    CreateToggle("Team Check", 0.45, settings.ESP, "TeamCheck")

    return ScreenGui
end

-- Função para alternar a GUI
local function ToggleGUI()
    settings.GUI.Visible = not settings.GUI.Visible
    if game.CoreGui:FindFirstChild("PCXHackGUI") then
        game.CoreGui.PCXHackGUI.MainFrame.Visible = settings.GUI.Visible
    end
end

-- Configurar tecla F para GUI
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == settings.GUI.Key and not gameProcessed then
        ToggleGUI()
    end
end)

-- Funções do Aimbot
local function FindClosestPlayer()
    local closestPlayer, closestDistance = nil, settings.Aimbot.FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local head = player.Character:FindFirstChild(settings.Aimbot.TargetPart)
            
            if humanoid and humanoid.Health > 0 and head then
                if settings.ESP.TeamCheck and player.Team == LocalPlayer.Team then
                    continue
                end
                
                -- Verificação de visibilidade
                if settings.Aimbot.VisibleCheck then
                    local raycastParams = RaycastParams.new()
                    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, player.Character}
                    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                    
                    local origin = Camera.CFrame.Position
                    local direction = (head.Position - origin).Unit * 1000
                    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
                    
                    if raycastResult and raycastResult.Instance:IsDescendantOf(player.Character) then
                        -- Não está visível
                        continue
                    end
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

local function SmoothAim(targetPosition)
    local currentPosition = Camera.CFrame.Position
    local direction = (targetPosition - currentPosition).Unit
    local newPosition = currentPosition + (direction * settings.Aimbot.Smoothness * 10)
    Camera.CFrame = CFrame.new(newPosition, targetPosition)
end

-- Funções do No Recoil
local function ApplyNoRecoil()
    if settings.NoRecoil.Enabled and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.AutoRotate = false
            humanoid.CameraOffset = Vector3.new(0, 0, 0)
        end
    end
end

-- Funções do ESP
local function CreateESP(player)
    if espObjects[player] then return end
    
    local character = player.Character
    if not character then return end
    
    espObjects[player] = {}
    
    -- Box ESP
    if settings.ESP.Box then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_Box"
        box.Adornee = character:WaitForChild("HumanoidRootPart")
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = Vector3.new(3, 5, 1)
        box.Color3 = settings.ESP.TeamCheck and player.Team == LocalPlayer.Team and settings.ESP.TeamColor or settings.ESP.EnemyColor
        box.Transparency = 0.5
        box.Parent = character
        table.insert(espObjects[player], box)
    end
    
    -- Line ESP
    if settings.ESP.Line then
        local line = Instance.new("LineHandleAdornment")
        line.Name = player.Name .. "_Line"
        line.Adornee = character:WaitForChild("HumanoidRootPart")
        line.AlwaysOnTop = true
        line.ZIndex = 10
        line.Length = (character:WaitForChild("HumanoidRootPart").Position.Y
        line.Thickness = 1
        line.Color3 = settings.ESP.TeamCheck and player.Team == LocalPlayer.Team and settings.ESP.TeamColor or settings.ESP.EnemyColor
        line.Parent = character
        table.insert(espObjects[player], line)
    end
end

local function RemoveESP(player)
    if espObjects[player] then
        for _, obj in pairs(espObjects[player]) do
            if obj then obj:Destroy() end
        end
        espObjects[player] = nil
    end
end

-- Conexões de eventos
local function OnPlayerAdded(player)
    connections[player] = {}
    
    table.insert(connections[player], player.CharacterAdded:Connect(function(character)
        if settings.ESP.Enabled then
            CreateESP(player)
        end
    end))
    
    table.insert(connections[player], player.CharacterRemoving:Connect(function()
        RemoveESP(player)
    end))
    
    if player.Character and settings.ESP.Enabled then
        CreateESP(player)
    end
end

local function Initialize()
    -- Criar GUI
    CreateGUI()
    
    -- Conectar jogadores existentes
    for _, player in pairs(Players:GetPlayers()) do
        OnPlayerAdded(player)
    end
    
    -- Conectar novos jogadores
    Players.PlayerAdded:Connect(OnPlayerAdded)
    
    -- Limpar conexões quando o jogador sair
    Players.PlayerRemoving:Connect(function(player)
        if connections[player] then
            for _, connection in pairs(connections[player]) do
                connection:Disconnect()
            end
            connections[player] = nil
        end
        RemoveESP(player)
    end)
    
    -- Loop principal
    table.insert(connections, RunService.RenderStepped:Connect(function()
        -- Aimbot quando atirando
        if settings.Aimbot.Enabled and UserInputService:IsMouseButtonPressed(settings.Aimbot.Key) then
            local closestPlayer = FindClosestPlayer()
            if closestPlayer and closestPlayer.Character then
                local head = closestPlayer.Character:FindFirstChild(settings.Aimbot.TargetPart)
                if head then
                    SmoothAim(head.Position)
                end
            end
        end
        
        -- No Recoil
        if settings.NoRecoil.Enabled then
            ApplyNoRecoil()
        end
        
        -- Atualizar ESP
        if settings.ESP.Enabled then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    CreateESP(player)
                end
            end
        end
    end))
end

-- Inicializar
Initialize()

print("Script para PCX DO TG carregado!")
print("Pressione F para abrir/fechar o menu")
print("Aimbot: Ativado com botão direito do mouse")
print("ESP: Caixa e Linha ativados")
print("No Recoil: Ativado")
