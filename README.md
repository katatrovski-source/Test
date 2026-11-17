-- Goated MM2 Script - Lightweight & Stable
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService)
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ==========================================
-- CONFIGURATION
-- ==========================================
local Config = {
    ESPEnabled = false,
    GodMode = false,
    SpeedHack = false,
    SpeedValue = 25,
    Theme = {
        Background = Color3.fromRGB(20, 20, 25),
        TextColor = Color3.fromRGB(255, 255, 255),
        Accent = Color3.fromRGB(0, 255, 200), -- Cyan
        Murderer = Color3.fromRGB(255, 50, 50),
        Sheriff = Color3.fromRGB(50, 255, 50),
        Innocent = Color3.fromRGB(150, 150, 150)
    }
}

-- ==========================================
-- UI CREATION
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GoatedMM2"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 300)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -150)
MainFrame.BackgroundColor3 = Config.Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Header
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleText = Instance.new("TextLabel")
TitleText.Name = "TitleText"
TitleText.Size = UDim2.new(1, -20, 1, 0)
TitleText.Position = UDim2.new(0, 10, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "🔪 Goated MM2"
TitleText.Font = Enum.Font.GothinBold
TitleText.TextSize = 18
TitleText.TextColor3 = Config.Theme.Accent
TitleText.Parent = TitleBar

-- Buttons Helper
function CreateButton(Text, YPos, Callback)
    local Btn = Instance.new("TextButton")
    Btn.Name = Text
    Btn.Size = UDim2.new(1, -20, 0, 30)
    Btn.Position = UDim2.new(0, 10, 0, YPos)
    Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    Btn.BorderSizePixel = 0
    Btn.Text = Text
    Btn.TextColor3 = Config.Theme.TextColor
    Btn.Font = Enum.Font.Gothin
    Btn.TextSize = 14
    Btn.Parent = MainFrame
    
    Btn.MouseButton1Click:Connect(Callback)
    return Btn
end

-- Content Container
local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -20, 1, -50)
ContentFrame.Position = UDim2.new(0, 10, 0, 50)
ContentFrame.BackgroundColor3 = Config.Theme.Background
ContentFrame.BorderSizePixel = 0
ContentFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ContentFrame.ScrollBarThickness = 2
ContentFrame.Parent = MainFrame

CreateButton("ESP: OFF", 0, function()
    Config.ESPEnabled = not Config.ESPEnabled
    UpdateButtons()
end)

CreateButton("God Mode: OFF", 40, function()
    Config.GodMode = not Config.GodMode
    UpdateButtons()
end)

CreateButton("Speed Hack: OFF", 80, function()
    Config.SpeedHack = not Config.SpeedHack
    UpdateButtons()
end)

-- Info Label
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Name = "InfoLabel"
InfoLabel.Size = UDim2.new(1, -20, 0, 20)
InfoLabel.Position = UDim2.new(0, 10, 0, 120)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text = "Drag me around!"
InfoLabel.TextColor3 = Config.Theme.Accent
InfoLabel.Font = Enum.Font.SourceHint
InfoLabel.TextSize = 12
InfoLabel.Parent = ContentFrame

-- Dragging Logic
local dragging = false
local dragStart = nil
local startPos = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragStart = input.Position
    end
end)

Players.LocalPlayer:GetMouse().Moved:Connect(function()
    if dragging then
        local delta = Vector2.new(input.Position.X - dragStart.X, input.Position.Y - dragStart.Y)
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

function UpdateButtons()
    local children = ContentFrame:GetChildren()
    for _, child in pairs(children) do
        if child:IsA("TextButton") then
            if child.Name == "ESP: " then
                child.Text = "ESP: " .. (Config.ESPEnabled and "ON" or "OFF")
            elseif child.Name == "God Mode: " then
                child.Text = "God Mode: " .. (Config.GodMode and "ON" or "OFF")
            elseif child.Name == "Speed Hack: " then
                child.Text = "Speed Hack: " .. (Config.SpeedHack and "ON" or "OFF")
            end
        end
    end
end

-- ==========================================
-- ESP LOGIC (Using Drawing API for Performance)
-- ==========================================
local Draw = {}
local DrawingObjects = {}

function Draw.new(Type, Properties)
    local d = drawing.new(Type)
    for k, v in pairs(Properties) do
        pcall(function() d[k] = v end)
    end
    table.insert(DrawingObjects, d)
    return d
end

function Draw.clear()
    for i, d in ipairs(DrawingObjects) do
        pcall(function() d:Remove() end)
    end
    DrawingObjects = {}
end

local PlayerESP = {}

function UpdateESP()
    if not Config.ESPEnabled then 
        Draw.clear()
        PlayerESP = {}
        return 
    end
    
    local players = Players:GetPlayers()
    local rootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    if not rootPart then return end

    for _, player in ipairs(players) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local role = "Innocent"
            
            -- Simple Role Detection
            local char = player.Character
            local tool = char:FindFirstChildOfClass("Tool")
            if tool then
                if string.find(tool.Name, "Knife") or string.find(tool.Name, "Katana") then
                    role = "Murderer"
                elseif string.find(tool.Name, "Revolver") or string.find(tool.Name, "Pistol") then
                    role = "Sheriff"
                end
            end
            
            local color = Config.Theme.InnocentColor
            if role == "Murderer" then color = Config.Theme.Murderer
            elseif role == "Sheriff" then color = Config.Theme.Sheriff end
            
            local espData = PlayerESP[player]
            if not espData then
                espData = {}
                espData.Box = Draw.new("Square")
                espData.Box.Filled = false
                espData.Box.Thickness = 2
                espData.Box.Color = color
                
                espData.Line = Draw.new("Line")
                espData.Line.From = rootPart.WorldPosition
                espData.Line.To = rootPart.WorldPosition
                espData.Line.Color = Color3.fromRGB(100, 100, 100)
                
                espData.Name = Draw.new("Text")
                espData.Name.Text = player.Name
                espData.Name.Color = color
                espData.Name.Outline = true
                espData.Name.OutlineColor = Color3.fromRGB(0,0,0)
                espData.Name.Size = 14
                espData.Name.Centered = true
                
                PlayerESP[player] = espData
            else
                -- Update positions
                local head = player.Character.Head
                local targetPos = head.Position
                local dist = (rootPart.WorldPosition - targetPos).Magnitude
                
                -- Scale box based on distance
                local size = math.clamp(dist * 0.05, 20, 100)
                
                espData.Box.Position = targetPos
                espData.Box.Size = Vector2.new(size, size * 1.8)
                
                espData.Line.From = rootPart.WorldPosition
                espData.Line.To = targetPos
                
                espData.Name.Position = targetPos + Vector3.new(0, 1.8, 0)
            end
        else
            if PlayerESP[player] then
                for _, obj in pairs(PlayerESP[player]) do
                    pcall(function() obj:Remove() end)
                end
                PlayerESP[player] = nil
            end
        end
    end
end

-- ==========================================
-- MAIN LOOPS
-- ==========================================
RunService.Heartbeat:Connect(function()
    -- Speed Hack
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
    if hum then
        if Config.SpeedHack then
            hum.WalkSpeed = Config.SpeedValue
        else
            hum.WalkSpeed = 16
        end
        
        -- God Mode (Simple Ragdoll Fix)
        if Config.GodMode and hum.State == Enum.HumanoidStateType.Ragdoll then
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    end
    
    UpdateESP()
end)

print("✅ Goated MM2 Loaded!")
