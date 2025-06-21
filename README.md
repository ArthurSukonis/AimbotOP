local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

local localPlayer = Players.LocalPlayer
local aimbotEnabled = false
local aimbotConnection

-- Função para pegar o player inimigo mais próximo de você
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge

    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("Head") then return nil end
    local myPosition = localPlayer.Character.Head.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer
            and player.Team ~= localPlayer.Team
            and player.Character
            and player.Character:FindFirstChild("Head")
            and player.Character:FindFirstChildOfClass("Humanoid")
            and player.Character.Humanoid.Health > 0 then

            local targetPosition = player.Character.Head.Position
            local distance = (myPosition - targetPosition).Magnitude

            if distance < shortestDistance then
                shortestDistance = distance
                closestPlayer = player
            end
        end
    end

    return closestPlayer
end

-- Ativar/desativar aimbot via botão
local function toggleAimbot(button)
    aimbotEnabled = not aimbotEnabled
    button.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")

    if aimbotEnabled then
        aimbotConnection = RunService.RenderStepped:Connect(function()
            local target = getClosestTarget()
            if target and target.Character and target.Character:FindFirstChild("Head") then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
            end
        end)
    elseif aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
end

-- GUI compatível pra qualquer executor
local playerGui = localPlayer:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimGui"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 130, 0, 45)
button.Position = UDim2.new(1, -140, 1, -60)
button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 20
button.Text = "Aimbot: OFF"
button.Parent = screenGui
button.BorderSizePixel = 0

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = button

button.MouseButton1Click:Connect(function()
    toggleAimbot(button)
end)
