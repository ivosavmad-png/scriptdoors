-- NekoHub - DOORS: Aviso de Entidades & Marcação Visual dos Itens - 100% compatível

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

-- Simples painel de seleção (pode editar manualmente aqui)
Entidades.Rush = true
Entidades.Ambush = true
Entidades.Seek = true
Entidades.Screech = true
Entidades.Eyes = true
Entidades.Halt = true
Entidades.Figure = true
Entidades.Jack = true
local avisoItens = true -- TRUE para mostrar/marcar itens, FALSE para não mostrar

-- Função de notificação padrão Roblox
local function Notificar(txt)
    game.StarterGui:SetCore("SendNotification", {
        Title = "NekoHub",
        Text = txt,
        Duration = 5
    })
end

-- Função que adiciona marcador visual ao item
local function MarcarItem(item)
    if item:FindFirstChild("NekoHub_ESP") then
        item.NekoHub_ESP:Destroy()
    end
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

-- Remove todos marcadores antigos
local function LimparMarcadores(sala)
    if not sala then return end
    for _, obj in pairs(sala:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("NekoHub_ESP") then
            obj.NekoHub_ESP:Destroy()
        end
    end
end

-- ENTIDADES global
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
    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local rooms = workspace:FindFirstChild("CurrentRooms")
    local sala = rooms and rooms:FindFirstChild(tostring(v)) or nil

    LimparMarcadores(sala)

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

        if avisoItens then
            for _, obj in ipairs(sala:GetDescendants()) do
                if ItensImportantes[obj.Name] then
                    Notificar("Encontrado: " .. obj.Name)
                    MarcarItem(obj)
                end
            end
            sala.DescendantAdded:Connect(function(child)
                if avisoItens and ItensImportantes[child.Name] then
                    Notificar("Encontrado: " .. child.Name)
                    MarcarItem(child)
                end
            end)
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
