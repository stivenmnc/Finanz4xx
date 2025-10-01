local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local function waitForPlayerGui()
    local pg = player:FindFirstChild("PlayerGui")
    while not pg do
        task.wait(0.1)
        pg = player:FindFirstChild("PlayerGui")
    end
    return pg
end

local PlayerGui = waitForPlayerGui()

-- Estados
local state = {
    invisible = false,
    noclip = false,
    locked = false,
    speedOn = false,
    originalWalkSpeed = nil
}

-- Helper: limpiar conexiones repetidas
local noclipConnection = nil

-- Crear GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "H4X_RBX_V1"
ScreenGui.ResetOnSpawn = true
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 260)
MainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
MainFrame.AnchorPoint = Vector2.new(0,0)

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 32)
Title.BackgroundTransparency = 1
Title.Text = "H4X RBX V1"
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.TextColor3 = Color3.fromRGB(255,255,255)

local UIList = Instance.new("UIListLayout")
UIList.Parent = MainFrame
UIList.Padding = UDim.new(0,8)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.VerticalAlignment = Enum.VerticalAlignment.Top

local function makeButton(text)
    local btn = Instance.new("TextButton")
    btn.Parent = MainFrame
    btn.Size = UDim2.new(1, -12, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(45,45,45)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 16
    btn.Text = text
    btn.LayoutOrder = 2
    btn.BorderSizePixel = 0
    return btn
end

-- Util: get character safely
local function GetCharacter()
    local ch = player.Character or player.CharacterAdded:Wait()
    return ch
end

-- Funciones de las opciones

-- 1) Invisible local (solo visual para el cliente)
local function toggleInvisible()
    local ch = player.Character
    if not ch then return end
    state.invisible = not state.invisible

    for _, obj in pairs(ch:GetDescendants()) do
        if obj:IsA("BasePart") then
            if state.invisible then
                -- guardamos un valor en un atributo para restaurar si falta
                if obj:GetAttribute("H4X_OrigTransparency") == nil then
                    obj:SetAttribute("H4X_OrigTransparency", obj.Transparency)
                end
                obj.LocalTransparencyModifier = 1
            else
                obj.LocalTransparencyModifier = 0
                local orig = obj:GetAttribute("H4X_OrigTransparency")
                if orig then
                    obj.Transparency = orig
                    obj:SetAttribute("H4X_OrigTransparency", nil)
                end
            end
        elseif obj:IsA("Decal") then
            if state.invisible then
                if obj:GetAttribute("H4X_OrigDecalTrans") == nil then
                    obj:SetAttribute("H4X_OrigDecalTrans", obj.Transparency)
                end
                obj.Transparency = 1
            else
                local orig = obj:GetAttribute("H4X_OrigDecalTrans")
                if orig then
                    obj.Transparency = orig
                    obj:SetAttribute("H4X_OrigDecalTrans", nil)
                end
            end
        end
    end
end

-- 2) Noclip básico (quita colisiones de partes del personaje)
local function setNoclip(enabled)
    local ch = player.Character
    if not ch then return end

    state.noclip = enabled

    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end

    if enabled then
        -- cada frame asegura que CanCollide = false
        noclipConnection = RunService.Stepped:Connect(function()
            local character = player.Character
            if not character then return end
            for _, p in pairs(character:GetDescendants()) do
                if p:IsA("BasePart") then
                    p.CanCollide = false
                end
            end
        end)
    else
        -- restaurar CanCollide (lo dejamos true por defecto)
        local character = player.Character
        if character then
            for _, p in pairs(character:GetDescendants()) do
                if p:IsA("BasePart") then
                    p.CanCollide = true
                end
            end
        end
    end
end

-- 3) Bloqueo de base automática (ancla en posición actual)
local function toggleLock()
    local ch = player.Character
    if not ch then return end
    local hrp = ch:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    state.locked = not state.locked
    if state.locked then
        -- guardar pos para mover y anclar
        hrp:SetAttribute("H4X_OrigAnchored", hrp.Anchored)
        hrp:SetAttribute("H4X_LockPos", tostring(hrp.Position))
        hrp.Anchored = true
    else
        local orig = hrp:GetAttribute("H4X_OrigAnchored")
        if orig ~= nil then
            hrp.Anchored = orig
            hrp:SetAttribute("H4X_OrigAnchored", nil)
            hrp:SetAttribute("H4X_LockPos", nil)
        else
            hrp.Anchored = false
        end
    end
end

-- 4) Speed safe (aumenta WalkSpeed temporalmente)
local function toggleSpeed()
    local ch = player.Character
    if not ch then return end
    local humanoid = ch:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if state.originalWalkSpeed == nil then
        state.originalWalkSpeed = humanoid.WalkSpeed
    end

    state.speedOn = not state.speedOn
    if state.speedOn then
        humanoid.WalkSpeed = 60 -- ajusta si quieres menos
    else
        humanoid.WalkSpeed = state.originalWalkSpeed or 16
    end
end

-- Crear botones y conectarlos
local b1 = makeButton("TP 1 - Invisible")
b1.MouseButton1Click:Connect(function()
    toggleInvisible()
    b1.Text = state.invisible and "TP 1 - Invisible (ON)" or "TP 1 - Invisible (OFF)"
end)

local b2 = makeButton("TP 2 - Noclip")
b2.MouseButton1Click:Connect(function()
    setNoclip(not state.noclip)
    b2.Text = state.noclip and "TP 2 - Noclip (ON)" or "TP 2 - Noclip (OFF)"
end)

local b3 = makeButton("TP 3 - Bloqueo Base")
b3.MouseButton1Click:Connect(function()
    toggleLock()
    b3.Text = state.locked and "TP 3 - Bloqueo (ON)" or "TP 3 - Bloqueo (OFF)"
end)

local b4 = makeButton("TP 4 - Speed")
b4.MouseButton1Click:Connect(function()
    toggleSpeed()
    b4.Text = state.speedOn and "TP 4 - Speed (ON)" or "TP 4 - Speed (OFF)"
end)

-- Restaurar en respawn para evitar estado roto
player.CharacterAdded:Connect(function(char)
    -- pequeña espera para que partes existan
    task.wait(0.2)
    -- restablecer invis y speed y noclip si estaban activados
    if state.invisible then
        toggleInvisible() -- esto alterna, así que lo llamamos para forzar el efecto en el nuevo char
        toggleInvisible() -- y lo volvemos a aplicar para mantener el estado
    end

    if state.noclip then
        setNoclip(true)
    end

    if state.speedOn then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid and state.originalWalkSpeed then
            humanoid.WalkSpeed = 60
        end
    end
end)
