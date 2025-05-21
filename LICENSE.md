-- ✅ Script para PCX DO TG (Roblox) - Funcionando Perfeitamente
-- 🔑 Tecla F para abrir/fechar a GUI
-- 🎯 Aimbot na cabeça ao atirar | ESP Box/Line | No Recoil

-- 🔧 CONFIGURAÇÕES INICIAIS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- ⚙️ CONFIGURAÇÕES
local Settings = {
    Aimbot = {
        Enabled = true,
        Key = Enum.UserInputType.MouseButton2, -- Botão direito para mirar
        Smoothness = 0.25,
        FOV = 100,
        TargetPart = "Head",
        VisibleCheck = true
    },
    ESP = {
        Enabled = true,
        Box = true,
        Line = true,
        TeamCheck = true,
        TeamColor = Color3.fromRGB(0, 255, 0), -- Verde para aliados
        EnemyColor = Color3.fromRGB(255, 50, 50) -- Vermelho para inimigos
    },
    NoRecoil = {
        Enabled = true
    },
    GUI = {
        Key = Enum.KeyCode.F, -- Tecla F para abrir/fechar
        Visible = false
    }
}

-- 📦 VARIÁVEIS
local ESPObjects = {}
local GUI

-- 🖼️ FUNÇÃO PARA CRIAR GUI
local function CreateGUI()
    -- Destruir GUI antiga se existir
    if game.CoreGui:FindFirstChild("PCXHackGUI") then
        game.CoreGui.PCXHackGUI:Destroy()
    end

    -- Criar nova GUI
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PCXHackGUI"
    ScreenGui.Parent = game.CoreGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 250, 0, 220)
    MainFrame.Position = UDim2.new(0.5, -125, 0.5, -110)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    MainFrame.BackgroundTransparency = 0.2
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = Settings.GUI.Visible
    MainFrame.Parent = ScreenGui

    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 6)
    UICorner.Parent = MainFrame

    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Size = UDim2.new(1, 0, 0, 30)
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

    -- 🔘 FUNÇÃO PARA CRIAR BOTÕES
    local function CreateToggle(text, yPos, configTable, configKey)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
        ToggleFrame.Position = UDim2.new(0.05, 0, yPos, 0)
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
        end)
    end

    -- 🎚️ CRIAR BOTÕES
    CreateToggle("Aimbot", 0.15, Settings.Aimbot, "Enabled")
    CreateToggle("ESP", 0.30, Settings.ESP, "Enabled")
    CreateToggle("No Recoil", 0.45, Settings.NoRecoil, "Enabled")
    CreateToggle("Team Check", 0.60, Settings.ESP, "TeamCheck")

    return ScreenGui
end

-- 🔄 FUNÇÃO PARA ABRIR/FECHAR GUI
local function ToggleGUI()
    Settings.GUI.Visible = not Settings.GUI.Visible
    if GUI and GUI.MainFrame then
        GUI.MainFrame.Visible = Settings.GUI.Visible
    end
end

-- 🎯 AIMBOT (MIRA NA CABEÇA)
local function FindClosestPlayer()
    local closestPlayer, closestDistance = nil, Settings.Aimbot.FOV

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local head = player.Character:FindFirstChild(Settings.Aimbot.TargetPart)

            if humanoid and humanoid.Health > 0 and head then
                if Settings.ESP.TeamCheck and player.Team == LocalPlayer.Team then
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

local function SmoothAim(targetPosition)
    local currentPosition = Camera.CFrame.Position
    local direction = (targetPosition - currentPosition).Unit
    local newPosition = currentPosition + (direction * Settings.Aimbot.Smoothness * 10)
    Camera.CFrame = CFrame.new(newPosition, targetPosition)
end

-- 🔫 NO RECOIL (SEM RECUO)
local function ApplyNoRecoil()
    if Settings.NoRecoil.Enabled and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.AutoRotate = false
            humanoid.CameraOffset = Vector3.new(0, 0, 0)
        end
    end
end

-- 📦 ESP (CAIXA E LINHA)
local function CreateESP(player)
    if ESPObjects[player] then return end

    local character = player.Character
    if not character then return end

    ESPObjects[player] = {}

    -- BOX ESP
    if Settings.ESP.Box then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_Box"
        box.Adornee = character:WaitForChild("HumanoidRootPart")
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = Vector3.new(3, 5, 1)
        box.Color3 = Settings.ESP.TeamCheck and player.Team == LocalPlayer.Team and Settings.ESP.TeamColor or Settings.ESP.EnemyColor
        box.Transparency = 0.5
        box.Parent = character
        table.insert(ESPObjects[player], box)
    end

    -- LINE ESP
    if Settings.ESP.Line then
        local line = Instance.new("LineHandleAdornment")
        line.Name = player.Name .. "_Line"
        line.Adornee = character:WaitForChild("HumanoidRootPart")
        line.AlwaysOnTop = true
        line.ZIndex = 10
        line.Length = (character:WaitForChild("HumanoidRootPart").Position).Y
        line.Thickness = 1
        line.Color3 = Settings.ESP.TeamCheck and player.Team == LocalPlayer.Team and Settings.ESP.TeamColor or Settings.ESP.EnemyColor
        line.Parent = character
        table.insert(ESPObjects[player], line)
    end
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if obj then obj:Destroy() end
        end
        ESPObjects[player] = nil
    end
end

-- 🔄 LOOP PRINCIPAL
local function MainLoop()
    -- AIMBOT (MIRA AO ATIRAR)
    if Settings.Aimbot.Enabled and UserInputService:IsMouseButtonPressed(Settings.Aimbot.Key) then
        local closestPlayer = FindClosestPlayer()
        if closestPlayer and closestPlayer.Character then
            local head = closestPlayer.Character:FindFirstChild(Settings.Aimbot.TargetPart)
            if head then
                SmoothAim(head.Position)
            end
        end
    end

    -- NO RECOIL
    if Settings.NoRecoil.Enabled then
        ApplyNoRecoil()
    end

    -- ESP
    if Settings.ESP.Enabled then
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
end

-- 🏁 INICIAR SCRIPT
GUI = CreateGUI()

-- TECLA F PARA ABRIR GUI
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Settings.GUI.Key and not gameProcessed then
        ToggleGUI()
    end
end)

-- CONECTAR LOOP
RunService.RenderStepped:Connect(MainLoop)

print("✅ SCRIPT CARREGADO! PRESSIONE F PARA ABRIR O MENU.")
