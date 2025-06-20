-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local plr = Players.LocalPlayer

-- Limpeza de GUI duplicada
if game.CoreGui:FindFirstChild("WallHopGui") then
    game.CoreGui:FindFirstChild("WallHopGui"):Destroy()
end

-- GUI Principal
local gui = Instance.new("ScreenGui")
gui.Name = "WallHopGui"
gui.ResetOnSpawn = false
gui.Parent = game.CoreGui

-- Frame principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0.125, 0, 0.075, 0)
frame.Position = UDim2.new(0.02, 0, 0.15, 0) -- posição melhor para não colidir com UI padrão
frame.BackgroundColor3 = Color3.new(1, 1, 1)
frame.BackgroundTransparency = 0.85
Instance.new("UICorner", frame)

-- Botão de toggle
local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(0.82, 0, 0.65, 0)
button.Position = UDim2.new(0.5, 0, 0.5, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.new(1, 1, 1)
button.BackgroundTransparency = 0.85
button.BorderSizePixel = 0
button.Text = "Walking"
button.TextScaled = true
button.Font = Enum.Font.Gotham
button.TextColor3 = Color3.new(0, 0, 0)
Instance.new("UICorner", button)

-- Sistema de arrastar
do
    local dragging, dragInput, dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Toggle de WallHop
local wallhopActive = false
button.MouseButton1Click:Connect(function()
    wallhopActive = not wallhopActive
    button.Text = wallhopActive and "WallHop" or "Walking"
end)

-- Giro suave
local function smoothRotate(root, angleRad)
    local steps = 3
    local delayTime = 0.007
    local stepAngle = angleRad / steps
    for i = 1, steps do
        root.CFrame *= CFrame.Angles(0, stepAngle, 0)
        task.wait(delayTime)
    end
end

-- Sistema de WallHop
local InfiniteJumpEnabled = true
UserInputService.JumpRequest:Connect(function()
    if not wallhopActive or not InfiniteJumpEnabled then return end

    local char = plr.Character
    if not char then return end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not root then return end

    InfiniteJumpEnabled = false
    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    smoothRotate(root, -1.3) -- aumento sutil para ~75 graus
    task.wait(0.05)
    smoothRotate(root, 1.3)
    InfiniteJumpEnabled = true
end)
