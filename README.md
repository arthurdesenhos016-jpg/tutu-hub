-- LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local guiEnabled = true
local isSpeedOn, isJumpOn, isNoClipOn = false, false, false
local savedConfig = {}
local defaultSpeed, defaultJump
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
defaultSpeed, defaultJump = humanoid.WalkSpeed, humanoid.JumpPower

-- Função para aplicar NoClip
local function applyNoClip()
    if isNoClipOn and character then
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

-- Loop do NoClip contínuo
RunService.Stepped:Connect(applyNoClip)

-- Função que cria a GUI
local function createGUI()
    if player.PlayerGui:FindFirstChild("TutuHubGUI") then
        return player.PlayerGui.TutuHubGUI
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "TutuHubGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 280, 0, 200)
    mainFrame.Position = UDim2.new(0, 20, 0, 100)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.Text = "Tutu Hub"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 20
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Parent = mainFrame

    -- Botão fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -35, 0, 0)
    closeButton.Text = "X"
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 18
    closeButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.Parent = mainFrame
    closeButton.MouseButton1Click:Connect(function()
        screenGui.Enabled = false
        guiEnabled = false
    end)

    -- Botão Velocidade
    local speedBtn = Instance.new("TextButton")
    speedBtn.Size = UDim2.new(0, 120, 0, 30)
    speedBtn.Position = UDim2.new(0, 10, 0, 50)
    speedBtn.Text = "Velocidade OFF"
    speedBtn.Font = Enum.Font.Gotham
    speedBtn.TextSize = 14
    speedBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    speedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedBtn.Parent = mainFrame

    local speedInput = Instance.new("TextBox")
    speedInput.Size = UDim2.new(0, 100, 0, 30)
    speedInput.Position = UDim2.new(0, 150, 0, 50)
    speedInput.PlaceholderText = "Velocidade"
    speedInput.Text = ""
    speedInput.Font = Enum.Font.Gotham
    speedInput.TextSize = 14
    speedInput.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedInput.ClearTextOnFocus = false
    speedInput.Parent = mainFrame

    speedBtn.MouseButton1Click:Connect(function()
        local inputSpeed = tonumber(speedInput.Text)
        if not inputSpeed then
            speedInput.PlaceholderText = "Número inválido"
            return
        end
        isSpeedOn = not isSpeedOn
        if isSpeedOn then
            humanoid.WalkSpeed = inputSpeed
            speedBtn.Text = "Velocidade ON"
            speedBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        else
            humanoid.WalkSpeed = defaultSpeed
            speedBtn.Text = "Velocidade OFF"
            speedBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        end
        savedConfig.speed = inputSpeed
    end)

    -- Botão Jump
    local jumpBtn = Instance.new("TextButton")
    jumpBtn.Size = UDim2.new(0, 120, 0, 30)
    jumpBtn.Position = UDim2.new(0, 10, 0, 90)
    jumpBtn.Text = "Jump OFF"
    jumpBtn.Font = Enum.Font.Gotham
    jumpBtn.TextSize = 14
    jumpBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    jumpBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    jumpBtn.Parent = mainFrame

    local jumpInput = Instance.new("TextBox")
    jumpInput.Size = UDim2.new(0, 100, 0, 30)
    jumpInput.Position = UDim2.new(0, 150, 0, 90)
    jumpInput.PlaceholderText = "JumpPower"
    jumpInput.Text = ""
    jumpInput.Font = Enum.Font.Gotham
    jumpInput.TextSize = 14
    jumpInput.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    jumpInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    jumpInput.ClearTextOnFocus = false
    jumpInput.Parent = mainFrame

    jumpBtn.MouseButton1Click:Connect(function()
        local inputJump = tonumber(jumpInput.Text)
        if not inputJump then
            jumpInput.PlaceholderText = "Número inválido"
            return
        end
        isJumpOn = not isJumpOn
        if isJumpOn then
            humanoid.JumpPower = inputJump
            jumpBtn.Text = "Jump ON"
            jumpBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        else
            humanoid.JumpPower = defaultJump
            jumpBtn.Text = "Jump OFF"
            jumpBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        end
        savedConfig.jump = inputJump
    end)

    -- Botão NoClip
    local noclipBtn = Instance.new("TextButton")
    noclipBtn.Size = UDim2.new(0, 120, 0, 30)
    noclipBtn.Position = UDim2.new(0, 10, 0, 130)
    noclipBtn.Text = "NoClip OFF"
    noclipBtn.Font = Enum.Font.Gotham
    noclipBtn.TextSize = 14
    noclipBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    noclipBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    noclipBtn.Parent = mainFrame

    noclipBtn.MouseButton1Click:Connect(function()
        isNoClipOn = not isNoClipOn
        if isNoClipOn then
            noclipBtn.Text = "NoClip ON"
            noclipBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        else
            noclipBtn.Text = "NoClip OFF"
            noclipBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        end
    end)

    return screenGui
end

-- Cria GUI
local gui = createGUI()

-- Reaplica GUI se removida
player.PlayerGui.ChildRemoved:Connect(function(child)
    if child.Name == "TutuHubGUI" then
        task.wait(0.5)
        gui = createGUI()
    end
end)

-- Tecla K para abrir/fechar
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.K and gui then
        gui.Enabled = not gui.Enabled
        guiEnabled = gui.Enabled
    end
end)

-- Reaplicar humanoid após respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    humanoid.WalkSpeed = savedConfig.speed or defaultSpeed
    humanoid.JumpPower = savedConfig.jump or defaultJump
end)

