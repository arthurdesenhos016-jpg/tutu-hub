-- LocalScript em StarterPlayerScripts
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- Configurações iniciais
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")
local defaultSpeed, defaultJump = humanoid.WalkSpeed, humanoid.JumpPower
local savedConfig = {Speed=defaultSpeed, Jump=defaultJump, Fly=50, Hitbox=5}

local isNoClip, isFly, isESP, isHitbox, isSpeed, isJump = false, false, false, false, false, false
local flyBodyVelocity = nil

-----------------------------------------------------
-- Funções de cheat
-----------------------------------------------------
local function applyNoClip()
    if character then
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = isNoClip and false or true
            end
        end
    end
end

local function applyFly()
    if isFly and character then
        if not flyBodyVelocity then
            flyBodyVelocity = Instance.new("BodyVelocity")
            flyBodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBodyVelocity.Velocity = Vector3.zero
            flyBodyVelocity.Parent = hrp
        end
    elseif flyBodyVelocity then
        flyBodyVelocity:Destroy()
        flyBodyVelocity = nil
    end
end

local function toggleESP(state)
    isESP = state
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local function applyToChar(char)
                local hrpChar = char:WaitForChild("HumanoidRootPart")
                if state then
                    if not hrpChar:FindFirstChild("ESP_Box") then
                        local box = Instance.new("BoxHandleAdornment")
                        box.Name = "ESP_Box"
                        box.Adornee = hrpChar
                        box.Size = Vector3.new(2,5,1)
                        box.Color3 = Color3.fromRGB(255,0,0)
                        box.Transparency = 0.5
                        box.AlwaysOnTop = true
                        box.ZIndex = 10
                        box.Parent = hrpChar
                    end
                    if not hrpChar:FindFirstChild("ESP_Name") then
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = "ESP_Name"
                        billboard.Adornee = hrpChar
                        billboard.Size = UDim2.new(0,100,0,50)
                        billboard.AlwaysOnTop = true
                        billboard.Parent = hrpChar

                        local label = Instance.new("TextLabel")
                        label.Size = UDim2.new(1,0,1,0)
                        label.BackgroundTransparency = 1
                        label.TextColor3 = Color3.fromRGB(255,255,0)
                        label.Font = Enum.Font.GothamBold
                        label.TextScaled = true
                        label.Text = plr.Name
                        label.Parent = billboard
                    end
                else
                    if hrpChar:FindFirstChild("ESP_Box") then hrpChar.ESP_Box:Destroy() end
                    if hrpChar:FindFirstChild("ESP_Name") then hrpChar.ESP_Name:Destroy() end
                end
            end

            if plr.Character then
                applyToChar(plr.Character)
            end
            plr.CharacterAdded:Connect(applyToChar)
        end
    end
end

local function applyHitbox(sizeVal)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local function applyToChar(char)
                local hrpChar = char:WaitForChild("HumanoidRootPart")

                -- Remove hitbox antiga
                if hrpChar:FindFirstChild("CustomHitbox") then
                    hrpChar.CustomHitbox:Destroy()
                end

                -- Cria nova hitbox invisível
                local hitboxPart = Instance.new("Part")
                hitboxPart.Name = "CustomHitbox"
                hitboxPart.Size = Vector3.new(sizeVal, sizeVal, sizeVal)
                hitboxPart.Transparency = 1
                hitboxPart.CanCollide = false
                hitboxPart.Anchored = false
                hitboxPart.Massless = true
                hitboxPart.CFrame = hrpChar.CFrame
                hitboxPart.Parent = workspace

                -- Mantém a hitbox sempre seguindo o personagem
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = hitboxPart
                weld.Part1 = hrpChar
                weld.Parent = hitboxPart

                -- Adorn visual opcional
                if isHitbox then
                    local adorn = hrpChar:FindFirstChild("HitboxAdornee")
                    if not adorn then
                        adorn = Instance.new("BoxHandleAdornment")
                        adorn.Name = "HitboxAdornee"
                        adorn.Adornee = hitboxPart
                        adorn.Color3 = Color3.fromRGB(255,0,0)
                        adorn.Transparency = 0.5
                        adorn.AlwaysOnTop = true
                        adorn.ZIndex = 5
                        adorn.Parent = hitboxPart
                    end
                    adorn.Size = hitboxPart.Size
                end
            end

            if plr.Character then
                applyToChar(plr.Character)
            end
            plr.CharacterAdded:Connect(applyToChar)
        end
    end
end

-----------------------------------------------------
-- Loop principal
-----------------------------------------------------
RunService.RenderStepped:Connect(function()
    applyNoClip()
    applyFly()

    if isFly and flyBodyVelocity then
        local camCF = workspace.CurrentCamera.CFrame
        local moveVec = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVec += camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVec -= camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVec -= camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVec += camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVec += Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec -= Vector3.new(0,1,0) end
        if moveVec.Magnitude > 0 then
            flyBodyVelocity.Velocity = moveVec.Unit * savedConfig.Fly
        else
            flyBodyVelocity.Velocity = Vector3.zero
        end
    end
end)

-----------------------------------------------------
-- GUI
-----------------------------------------------------
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "AizenHub"
ScreenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 350)
mainFrame.Position = UDim2.new(0,20,0,100)
mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = ScreenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Aizen Hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Parent = mainFrame

local optionsFrame = Instance.new("Frame")
optionsFrame.Size = UDim2.new(1,0,1,-30)
optionsFrame.Position = UDim2.new(0,0,0,30)
optionsFrame.BackgroundTransparency = 1
optionsFrame.Parent = mainFrame

local layout = Instance.new("UIListLayout")
layout.Parent = optionsFrame
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0,5)

local function createOptionRow(name, default, callback)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1,0,0,35)
    row.BackgroundTransparency = 1
    row.LayoutOrder = #optionsFrame:GetChildren()

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0,120,1,0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.Font = Enum.Font.Gotham
    label.TextScaled = true
    label.Parent = row

    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0,80,1,0)
    input.Position = UDim2.new(0,130,0,0)
    input.Text = tostring(default)
    input.TextColor3 = Color3.fromRGB(255,255,255)
    input.BackgroundColor3 = Color3.fromRGB(50,50,50)
    input.ClearTextOnFocus = false
    input.Parent = row

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,80,1,0)
    btn.Position = UDim2.new(0,220,0,0)
    btn.Text = "OFF"
    btn.BackgroundColor3 = Color3.fromRGB(150,0,0)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Parent = row

    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and Color3.fromRGB(0,150,0) or Color3.fromRGB(150,0,0)
        btn.Text = state and "ON" or "OFF"
        callback(state, tonumber(input.Text) or default)
    end)

    row.Parent = optionsFrame
end

-- Opções
createOptionRow("Speed", savedConfig.Speed, function(state,val)
    isSpeed = state
    humanoid.WalkSpeed = state and val or defaultSpeed
    savedConfig.Speed = val
end)

createOptionRow("Jump", savedConfig.Jump, function(state,val)
    isJump = state
    humanoid.JumpPower = state and val or defaultJump
    savedConfig.Jump = val
end)

createOptionRow("Fly", savedConfig.Fly, function(state,val)
    isFly = state
    savedConfig.Fly = val
    applyFly()
end)

createOptionRow("Hit
