-- NekoHub - DOORS: Aviso de Entidades & Itens - Mostra a posição dos itens (com marcador visual)
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

local Entidades = {
    Rush = false,
    Ambush = false,
    Seek = false,
    Screech = false,
    Eyes = false,
    Halt = false,
    Figure = false,
    Jack = false,
}

local ItensImportantes = {
    Crucifix = true,
    Vitamins = true,
    Lockpick = true,
    Lighter = true,
    Candle = true,
    Flashlight = true,
    ["Skeleton Key"] = true,
    Key = true,
    Gold = true,
    Battery = true,
    ["Holy Grenade"] = true
}

local avisoItens = false
local salaAtualCon = nil

-- Função de notificação
local function Notificar(txt, tempo)
    OrionLib:MakeNotification({
        Name = "NekoHub",
        Content = txt,
        Time = tempo or 5
    })
end

-- Função que adiciona BillboardGui (marcador) no item
local function MarcarItem(item)
    -- Remove marcador antigo se existir
    if item:FindFirstChild("NekoHub_ESP") then
        item.NekoHub_ESP:Destroy()
    end
    -- Só marca se for BasePart ou Model com PrimaryPart
    local adornee
    if item:IsA("BasePart") then
        adornee = item
    elseif item:IsA("Model") and item.PrimaryPart then
        adornee = item.PrimaryPart
    end
    if adornee then
        local gui = Instance.new("BillboardGui")
        gui.Name = "NekoHub_ESP"
        gui.Size = UDim2.new(0, 100, 0, 30)
        gui.AlwaysOnTop = true
        gui.Adornee = adornee
        gui.StudsOffset = Vector3.new(0, 2, 0)
        local lbl = Instance.new("TextLabel", gui)
        lbl.Size = UDim2.new(1, 0, 1, 0)
        lbl.Text = "🔴 " .. item.Name
        lbl.TextColor3 = Color3.fromRGB(220,30,30)
        lbl.BackgroundTransparency = 1
        lbl.TextScaled = true
        gui.Parent = adornee
    end
end

-- Remove todos marcadores de sala anterior
local function LimpaMarcadores(sala)
    if not sala then return end
    for _, obj in pairs(sala:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("NekoHub_ESP") then
            obj.NekoHub_ESP:Destroy()
        end
    end
end

-- Criação do painel principal
local Janela = OrionLib:MakeWindow({
    Name = "NekoHub - DOORS",
    HidePremium = true,
    SaveConfig = false
})

-- Aba de Entidades
local abaEntidade = Janela:MakeTab({
    Name = "Entidades",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

for entidade, _ in pairs(Entidades) do
    abaEntidade:AddToggle({
        Name = "Avisar sobre " .. entidade,
        Default = false,
        Callback = function(state)
            Entidades[entidade] = state
        end
    })
end

-- Aba de Itens
local abaItens = Janela:MakeTab({
    Name = "Itens",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

abaItens:AddToggle({
    Name = "Avisar e mostrar itens na sala",
    Default = false,
    Callback = function(state)
        avisoItens = state
    end
})

-- Detecção de entidades (listeners globais)
workspace.ChildAdded:Connect(function(child)
    if typeof(child) == "Instance" then
        if child.Name == "RushMoving" and Entidades.Rush then Notificar("Entidade: Rush") end
        if child.Name == "AmbushMoving" and Entidades.Ambush then Notificar("Entidade: Ambush") end
        if child.Name == "Eyes" and Entidades.Eyes then Notificar("Entidade: Eyes") end
    end
end)

game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
    if value and Entidades.Seek then Notificar("Entidade: Seek") end
end)

game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    -- Limpa marcadores antigos (se algum)
    if salaAtualCon then
        salaAtualCon:Disconnect()
        salaAtualCon = nil
    end

    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local rooms = workspace:FindFirstChild("CurrentRooms")
    local sala = rooms and rooms:FindFirstChild(tostring(v)) or nil

    if sala then
        LimpaMarcadores(sala)
        if avisoItens then
            for _, obj in ipairs(sala:GetDescendants()) do
                if ItensImportantes[obj.Name] then
                    Notificar("Encontrado: " .. obj.Name)
                    MarcarItem(obj)
                end
            end
            salaAtualCon = sala.DescendantAdded:Connect(function(child)
                if avisoItens and ItensImportantes[child.Name] then
                    Notificar("Encontrado: " .. child.Name)
                    MarcarItem(child)
                end
            end)
        end
    end

    -- Entidades que dependem da sala
    if sala then
        if sala:FindFirstChild("Halt") and Entidades.Halt then
            Notificar("Entidade: Halt")
        end
        if Entidades.Figure and tostring(v) == "50" and sala:FindFirstChild("Figure") then
            Notificar("Entidade: Figure (Sala 50)")
        end
        if Entidades.Jack and sala:FindFirstChild("Jack") then
            Notificar("Entidade: Jack (Armário/Porta)")
        end
    end
end)

-- Screech (reconecta ao respawn)
local function conectarScreech(character)
    if character and typeof(character) == "Instance" then
        character.ChildAdded:Connect(function(child)
            if child.Name == "Screech" and Entidades.Screech then
                Notificar("Entidade: Screech")
            end
        end)
    end
end
local plr = game.Players.LocalPlayer
if plr.Character then conectarScreech(plr.Character) end
plr.CharacterAdded:Connect(conectarScreech)

OrionLib:Init()
