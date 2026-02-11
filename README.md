-- CONFIGURAÇÕES GLOBAIS
getgenv().HBE_Enabled = false
getgenv().ESP_Enabled = false
getgenv().Skeleton_Enabled = false -- NOVO
getgenv().WallCheck = true
getgenv().Noclip_Enabled = false
getgenv().HitboxSize = 15

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- LIMPEZA
if player.PlayerGui:FindFirstChild("HRZ_V5_NOCLIP") then 
    player.PlayerGui.HRZ_V5_NOCLIP:Destroy() 
end

-- INTERFACE
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HRZ_V5_NOCLIP"
ScreenGui.Parent = player.PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 200, 0, 360) -- Aumentado para caber o novo botão
MainFrame.Position = UDim2.new(0.5, -100, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.Active = true
MainFrame.Draggable = true 
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(160, 100, 255)
Title.Text = "HRZ SCRIPT(VIP)"
Title.TextColor3 = Color3.new(1,1,1)
Title.Parent = MainFrame

local function CreateButton(text, pos, toggleVar)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 40) -- Ajuste fino no tamanho
    btn.Position = pos
    btn.BackgroundColor3 = getgenv()[toggleVar] and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(35, 35, 35)
    btn.Text = text .. ": " .. (getgenv()[toggleVar] and "ON" or "OFF")
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Parent = MainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

    btn.MouseButton1Click:Connect(function()
        getgenv()[toggleVar] = not getgenv()[toggleVar]
        btn.Text = text .. ": " .. (getgenv()[toggleVar] and "ON" or "OFF")
        btn.BackgroundColor3 = getgenv()[toggleVar] and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(35, 35, 35)
    end)
    return btn
end

-- BOTOES (Posições ajustadas)
CreateButton("ESP", UDim2.new(0.05, 0, 0.12, 0), "ESP_Enabled")
CreateButton("ESP ESQUELETO", UDim2.new(0.05, 0, 0.28, 0), "Skeleton_Enabled")
CreateButton("HITBOX", UDim2.new(0.05, 0, 0.44, 0), "HBE_Enabled")
CreateButton("WALL CHECK", UDim2.new(0.05, 0, 0.60, 0), "WallCheck")
CreateButton("ATRAVESSAR PAREDE", UDim2.new(0.05, 0, 0.76, 0), "Noclip_Enabled")

-- LÓGICA DO ESQUELETO
local SkeletonParts = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
}

local function CreateLine()
    local l = Drawing.new("Line")
    l.Thickness = 1.5
    l.Transparency = 1
    return l
end

local PlayerLines = {}

-- FUNÇÃO DE VISIBILIDADE
local function IsVisible(targetPart)
    if not getgenv().WallCheck then return true end
    local origin = camera.CFrame.Position
    local direction = (targetPart.Position - origin)
    local raycastParams = RaycastParams.new()
    local ignoreList = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character then table.insert(ignoreList, p.Character) end
    end
    raycastParams.FilterDescendantsInstances = ignoreList
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    local result = workspace:Raycast(origin, direction, raycastParams)
    return result == nil
end

-- LOOP PRINCIPAL
RunService.RenderStepped:Connect(function()
    if getgenv().Noclip_Enabled and player.Character then
        for _, part in pairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player then
            local char = p.Character
            if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
                local root = char.HumanoidRootPart
                local visible = IsVisible(root)

                -- HITBOX
                if getgenv().HBE_Enabled and visible then
                    root.Size = Vector3.new(getgenv().HitboxSize, getgenv().HitboxSize, getgenv().HitboxSize)
                    root.Transparency = 1
                    root.CanCollide = false
                else
                    root.Size = Vector3.new(2, 2, 1)
                end

                -- ESP HIGHLIGHT
                local hl = char:FindFirstChild("HRZ_Highlight")
                if getgenv().ESP_Enabled then
                    if not hl then hl = Instance.new("Highlight", char); hl.Name = "HRZ_Highlight" end
                    hl.FillColor = visible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                elseif hl then hl:Destroy() end

                -- ESP ESQUELETO
                if getgenv().Skeleton_Enabled then
                    if not PlayerLines[p.Name] then
                        PlayerLines[p.Name] = {}
                        for i=1, #SkeletonParts do table.insert(PlayerLines[p.Name], CreateLine()) end
                    end

                    for i, bone in pairs(SkeletonParts) do
                        local p1, p2 = char:FindFirstChild(bone[1]), char:FindFirstChild(bone[2])
                        local line = PlayerLines[p.Name][i]
                        if p1 and p2 then
                            local pos1, onScreen1 = camera:WorldToViewportPoint(p1.Position)
                            local pos2, onScreen2 = camera:WorldToViewportPoint(p2.Position)
                            if onScreen1 and onScreen2 then
                                line.From = Vector2.new(pos1.X, pos1.Y)
                                line.To = Vector2.new(pos2.X, pos2.Y)
                                line.Color = visible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                                line.Visible = true
                            else line.Visible = false end
                        else line.Visible = false end
                    end
                else
                    if PlayerLines[p.Name] then
                        for _, l in pairs(PlayerLines[p.Name]) do l.Visible = false end
                    end
                end
            else
                if PlayerLines[p.Name] then
                    for _, l in pairs(PlayerLines[p.Name]) do l.Visible = false end
                end
            end
        end
    end
end)

-- FECHAR COM F8
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.F8 then
        MainFrame.Visible = not MainFrame.Visible
    end
end)
