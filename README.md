local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local plr = Players.LocalPlayer

-- Remove GUI duplicada
local existingGui = CoreGui:FindFirstChild("WallHopGui")
if existingGui then existingGui:Destroy() end

-- GUI
local guiy = Instance.new("ScreenGui")
guiy.Name = "WallHopGui"
guiy.Parent = CoreGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0.125, 0, 0.075, 0)
frame.Position = UDim2.new(0.02, 0, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
frame.BackgroundTransparency = 0.85
frame.Parent = guiy

Instance.new("UICorner", frame)

local button = Instance.new("TextButton")
button.Size = UDim2.new(0.82, 0, 0.65, 0)
button.Position = UDim2.new(0.5, 0, 0.5, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
button.BackgroundTransparency = 0.85
button.BorderSizePixel = 0
button.Text = "Walking"
button.TextScaled = true
button.Font = Enum.Font.Gotham
button.TextColor3 = Color3.new(0, 0, 0)
button.Parent = frame

Instance.new("UICorner", button)

-- Drag funcional
do
    local dragging, dragInput, dragStart, startPos

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
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
        if input.UserInputType == Enum.UserInputType.MouseMovement then
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

-- Toggle
local wallhopActive = false
button.MouseButton1Click:Connect(function()
    wallhopActive = not wallhopActive
    button.Text = wallhopActive and "WallHop" or "Walking"
end)

-- Giro com leve suavidade
local function smoothQuickRotate(root, angle)
    local steps = 2
    local stepAngle = angle / steps
    for _ = 1, steps do
        root.CFrame *= CFrame.Angles(0, stepAngle, 0)
        task.wait(0.02) -- giro mais lento, ainda leve
    end
end

-- Sistema de WallHop
local debounce = true
UserInputService.JumpRequest:Connect(function()
    if not wallhopActive or not debounce then return end

    local char = plr.Character
    if not char then return end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not root then return end

    debounce = false
    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)

    smoothQuickRotate(root, -1)
    task.wait(0.02)
    smoothQuickRotate(root, 1)

    debounce = true
end)
