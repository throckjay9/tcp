-- ✅ Versão Final com GUI (Tecla F) + Aimbot ao Atirar + NoRecoil
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- ⚙️ Configurações Atualizadas
local Settings = {
    Aimbot = {
        ActiveOnFire = true,
        Smoothness = 0.15,
        FOV = 90,
        TargetPart = "Head",
        VisibleCheck = true,
        TeamCheck = true
    },
    NoRecoil = {
        Stabilize = true,
        Power = 0.7
    },
    GUI = {
        Key = Enum.KeyCode.F, -- Tecla F para abrir/fechar
        Enabled = true,
        Visible = false
    }
}

-- 🖼️ Criação da GUI
local function CreateGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "AimTrainerGUI"
    ScreenGui.Parent = game.CoreGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 300, 0, 250)
    MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    MainFrame.BackgroundTransparency = 0.15
    MainFrame.BorderSizePixel = 0
    MainFrame.Visible = Settings.GUI.Visible
    MainFrame.Parent = ScreenGui

    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = MainFrame

    -- Título
    local Title = Instance.new("TextLabel")
    Title.Text = "TRAINER MENU (F)"
    Title.Size = UDim2.new(1, 0, 0, 40)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 18
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Parent = MainFrame

    -- Função para criar toggles
    local function CreateToggle(text, yPos, config, key)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
        ToggleFrame.Position = UDim2.new(0.05, 0, yPos, 0)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = MainFrame

        local Label = Instance.new("TextLabel")
        Label.Text = text
        Label.Size = UDim2.new(0.7, 0, 1, 0)
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.Gotham
        Label.TextSize = 14
        Label.TextColor3 = Color3.new(1, 1, 1)
        Label.BackgroundTransparency = 1
        Label.Parent = ToggleFrame

        local Toggle = Instance.new("TextButton")
        Toggle.Size = UDim2.new(0.25, 0, 0.8, 0)
        Toggle.Position = UDim2.new(0.75, 0, 0.1, 0)
        Toggle.Text = config[key] and "ON" or "OFF"
        Toggle.TextColor3 = Color3.new(1, 1, 1)
        Toggle.BackgroundColor3 = config[key] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
        Toggle.Font = Enum.Font.GothamBold
        Toggle.Parent = ToggleFrame

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 4)
        Corner.Parent = Toggle

        Toggle.MouseButton1Click:Connect(function()
            config[key] = not config[key]
            Toggle.Text = config[key] and "ON" or "OFF"
            Toggle.BackgroundColor3 = config[key] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
        end)
    end

    -- Criar toggles
    CreateToggle("Aimbot ao Atirar", 0.15, Settings.Aimbot, "ActiveOnFire")
    CreateToggle("Verificação de Time", 0.25, Settings.Aimbot, "TeamCheck")
    CreateToggle("NoRecoil", 0.35, Settings.NoRecoil, "Stabilize")

    -- Controle de Suavidade
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(0.9, 0, 0, 50)
    SliderFrame.Position = UDim2.new(0.05, 0, 0.55, 0)
    SliderFrame.BackgroundTransparency = 1
    SliderFrame.Parent = MainFrame

    local SliderLabel = Instance.new("TextLabel")
    SliderLabel.Text = "Suavidade: " .. string.format("%.2f", Settings.Aimbot.Smoothness)
    SliderLabel.Size = UDim2.new(1, 0, 0, 20)
    SliderLabel.Font = Enum.Font.Gotham
    SliderLabel.TextSize = 14
    SliderLabel.TextColor3 = Color3.new(1, 1, 1)
    SliderLabel.BackgroundTransparency = 1
    SliderLabel.Parent = SliderFrame

    local Slider = Instance.new("Frame")
    Slider.Size = UDim2.new(1, 0, 0, 5)
    Slider.Position = UDim2.new(0, 0, 0.5, 0)
    Slider.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    Slider.Parent = SliderFrame

    local SliderCorner = Instance.new("UICorner")
    SliderCorner.Parent = Slider

    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new(Settings.Aimbot.Smoothness, 0, 1, 0)
    Fill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    Fill.Parent = Slider

    local FillCorner = Instance.new("UICorner")
    FillCorner.Parent = Fill

    Slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local connection
            connection = input.Changed:Connect(function()
                local posX = math.clamp((input.Position.X - Slider.AbsolutePosition.X) / Slider.AbsoluteSize.X, 0, 1)
                Settings.Aimbot.Smoothness = posX
                Fill.Size = UDim2.new(posX, 0, 1, 0)
                SliderLabel.Text = "Suavidade: " .. string.format("%.2f", posX)
                
                if input.UserInputState == Enum.UserInputState.End then
                    connection:Disconnect()
                end
            end)
        end
    end)

    return ScreenGui
end

-- 🎮 Controle da GUI
local GUI = CreateGUI()
local function ToggleGUI()
    Settings.GUI.Visible = not Settings.GUI.Visible
    GUI.MainFrame.Visible = Settings.GUI.Visible
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Settings.GUI.Key and not gameProcessed then
        ToggleGUI()
    end
end)

-- 🎯 Sistema de Mira (Mesmo código anterior)
local function FindTarget()
    -- ... (Manter mesma função do script anterior)
end

local function SmoothAim(targetPos)
    -- ... (Manter mesma função do script anterior)
end

-- 🔫 NoRecoil (Mesmo código anterior)
local function StabilizeCamera()
    -- ... (Manter mesma função do script anterior)
end

-- 🔄 Conexões de Controle
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 and Settings.Aimbot.ActiveOnFire then
        local target = FindTarget()
        if target and target.Character then
            SmoothAim(target.Character:FindFirstChild(Settings.Aimbot.TargetPart).Position)
        end
    end
end)

game:GetService("RunService").RenderStepped:Connect(function()
    StabilizeCamera()
end)

print("Sistema completo carregado | Pressione F para abrir o menu")
