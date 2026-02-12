-- CONFIGURAÇÕES GLOBAIS
getgenv().HBE_Enabled = false
getgenv().ESP_Enabled = false
getgenv().WallCheck = true
getgenv().Noclip_Enabled = false
getgenv().Underground_Enabled = false -- Nova função
getgenv().HitboxSize = 15

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local fakeBody = nil

-- LIMPEZA DE GUIS ANTIGAS
if player.PlayerGui:FindFirstChild("HRZ_COMPATIBLE_V5") then 
    player.PlayerGui.HRZ_COMPATIBLE_V5:Destroy() 
end

-- PLATAFORMA INVISÍVEL PARA GHOST UNDER
local plat = Instance.new("Part")
plat.Size = Vector3.new(20, 1, 20)
plat.Anchored = true
plat.Transparency = 1
plat.Name = "HRZ_SafePlate"
plat.Parent = workspace

-- INTERFACE
local sg = Instance.new("ScreenGui", player.PlayerGui)
sg.Name = "HRZ_COMPATIBLE_V5"
sg.ResetOnSpawn = false

-- CONTAINER MOVÍVEL
local SideFrame = Instance.new("Frame", sg)
SideFrame.Size = UDim2.new(0, 55, 0, 310) 
SideFrame.Position = UDim2.new(0, 10, 0.3, 0)
SideFrame.BackgroundTransparency = 0.8
SideFrame.BackgroundColor3 = Color3.new(0,0,0)
SideFrame.Active = true
SideFrame.Draggable = true 

Instance.new("UICorner", SideFrame).CornerRadius = UDim.new(0, 10)

local layout = Instance.new("UIListLayout", SideFrame)
layout.Padding = UDim.new(0, 8)
layout.HorizontalAlignment = "Center"
layout.VerticalAlignment = "Center"

-- FUNÇÃO PARA CRIAR OS TOGGLES
local function CreateToggle(name, varName, colorOn, specialCallback)
    local btn = Instance.new("TextButton", SideFrame)
    btn.Size = UDim2.new(0, 45, 0, 45)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Text = name
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 10)

    local function UpdateVisual()
        btn.BackgroundColor3 = getgenv()[varName] and colorOn or Color3.fromRGB(40, 40, 40)
        btn.Text = name .. "\n" .. (getgenv()[varName] and "ON" or "OFF")
    end
    
    UpdateVisual()

    btn.MouseButton1Click:Connect(function()
        getgenv()[varName] = not getgenv()[varName]
        UpdateVisual()
        if specialCallback then specialCallback() end
    end)
end

-- CALLBACK ESPECIAL PARA O GHOST UNDER (ATUALIZADO)
local function ToggleGhost()
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local hrp = char.HumanoidRootPart
        if getgenv().Underground_Enabled then
            getgenv().HBE_Enabled = true -- Ativa a hitbox automaticamente
            getgenv().WallCheck = false  -- Permite hit através do chão
            
            char.Archivable = true
            fakeBody = char:Clone()
            fakeBody.Parent = workspace
            fakeBody:MoveTo(hrp.Position)
            for _, p in pairs(fakeBody:GetChildren()) do
                if p:IsA("BasePart") then p.CanCollide = false p.Transparency = 0.5 end
            end
            hrp.CFrame = hrp.CFrame * CFrame.new(0, -15, 0)
        else
            getgenv().WallCheck = true
            if fakeBody then fakeBody:Destroy() end
            hrp.CFrame = hrp.CFrame * CFrame.new(0, 16, 0)
        end
    end
end

-- CRIANDO OS BOTÕES
CreateToggle("ESP", "ESP_Enabled", Color3.fromRGB(0, 170, 255))
CreateToggle("HBX", "HBE_Enabled", Color3.fromRGB(255, 170, 0))
CreateToggle("WALL", "WallCheck", Color3.fromRGB(170, 0, 255))
CreateToggle("NOCP", "Noclip_Enabled", Color3.fromRGB(255, 0, 80))
CreateToggle("GST", "Underground_Enabled", Color3.fromRGB(0, 255, 150), ToggleGhost)

-- FUNÇÃO WALL CHECK
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
RunService.Stepped:Connect(function()
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local hrp = char.HumanoidRootPart
        
        if (getgenv().Noclip_Enabled or getgenv().Underground_Enabled) then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end

        if getgenv().Underground_Enabled then
            plat.CanCollide = true
            plat.CFrame = hrp.CFrame * CFrame.new(0, -3.2, 0)
        else
            plat.CanCollide = false
            plat.Position = Vector3.new(0, -5000, 0)
        end
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local root = p.Character.HumanoidRootPart
            local hum = p.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then
                local visible = IsVisible(root)
                
                -- LÓGICA DE HITBOX ATUALIZADA (FORÇA 30 NO GHOST)
                if getgenv().HBE_Enabled and (visible or getgenv().Underground_Enabled) then
                    local size = getgenv().Underground_Enabled and 30 or getgenv().HitboxSize
                    root.Size = Vector3.new(size, size, size)
                    root.Transparency = 1
                    root.CanCollide = false
                else
                    root.Size = Vector3.new(2, 2, 1)
                    root.Transparency = 1
                end
                
                local hl = p.Character:FindFirstChild("HRZ_Highlight")
                if getgenv().ESP_Enabled then
                    if not hl then hl = Instance.new("Highlight", p.Character); hl.Name = "HRZ_Highlight" end
                    hl.FillColor = visible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                elseif hl then hl:Destroy() end
            end
        end
    end
end)
