-- CONFIGURAÇÕES GLOBAIS
getgenv().HBE_Enabled = false
getgenv().ESP_Enabled = false
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
MainFrame.Size = UDim2.new(0, 200, 0, 310)
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
    btn.Size = UDim2.new(0.9, 0, 0, 45)
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

-- BOTOES
CreateButton("ESP", UDim2.new(0.05, 0, 0.15, 0), "ESP_Enabled")
CreateButton("HITBOX", UDim2.new(0.05, 0, 0.35, 0), "HBE_Enabled")
CreateButton("WALL CHECK", UDim2.new(0.05, 0, 0.55, 0), "WallCheck")
CreateButton("ATRAVESSAR PAREDE", UDim2.new(0.05, 0, 0.75, 0), "Noclip_Enabled")

-- FUNÇÃO DE VISIBILIDADE (VERSÃO SIMPLIFICADA QUE IGNORA PLAYERS)
local function IsVisible(targetPart)
    if not getgenv().WallCheck then return true end
    
    local origin = camera.CFrame.Position
    local direction = (targetPart.Position - origin)
    
    local raycastParams = RaycastParams.new()
    
    -- Criamos uma lista para ignorar TODOS os personagens
    local ignoreList = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character then
            table.insert(ignoreList, p.Character)
        end
    end
    
    raycastParams.FilterDescendantsInstances = ignoreList
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(origin, direction, raycastParams)
    
    -- Se o raio não bater em nada (paredes), o player é considerado visível
    return result == nil
end

-- LOOP PRINCIPAL
RunService.Stepped:Connect(function()
    if getgenv().Noclip_Enabled and player.Character then
        for _, part in pairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local root = p.Character.HumanoidRootPart
            local hum = p.Character:FindFirstChild("Humanoid")
            
            -- Otimização: chama a função uma vez por player no loop
            local visible = IsVisible(root)

            if getgenv().HBE_Enabled and hum and hum.Health > 0 and visible then
                root.Size = Vector3.new(getgenv().HitboxSize, getgenv().HitboxSize, getgenv().HitboxSize)
                root.Transparency = 1 -- Mude para 0.7 se quiser ver a hitbox
                root.CanCollide = false
            else
                root.Size = Vector3.new(2, 2, 1)
                root.Transparency = 1
            end
            
            local hl = p.Character:FindFirstChild("HRZ_Highlight")
            if getgenv().ESP_Enabled and hum and hum.Health > 0 then
                if not hl then 
                    hl = Instance.new("Highlight", p.Character)
                    hl.Name = "HRZ_Highlight" 
                end
                hl.FillColor = visible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
            elseif hl then 
                hl:Destroy() 
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
