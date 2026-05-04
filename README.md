-- NekoHub - DOORS: Aviso de Entidades & Itens no Rayfield (com marcador visual)
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Entidades = {
    Rush = false,
    Ambush = false,
    Seek = false,
    Screech = false,
    Eyes = false,
    Halt = false,
    Figure = false,
    Jack = false
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

-- Notificação Rayfield
local function Alert(title, msg)
    Rayfield:Notify({
        Title = title,
        Content = msg,
        Duration = 5,
        Image = 4483362458
    })
end

-- Marca item com BillboardGui
local function MarcarItem(item)
    if not item:IsA("BasePart") and not (item:IsA("Model") and item.PrimaryPart) then return end
    local adornee = item:IsA("BasePart") and item or item.PrimaryPart
    if adornee:FindFirstChild("NekoHub_ESP") then adornee.NekoHub_ESP:Destroy() end
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
    lbl.TextStrokeTransparency = 0.3
    lbl.TextStrokeColor3 = Color3.new(0,0,0)
    lbl.Font = Enum.Font.FredokaOne
    lbl.TextScaled = true
    gui.Parent = adornee
end

-- Limpar marcadores antigos
local function LimparMarcadores(sala)
    if not sala then return end
    for _, obj in ipairs(sala:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("NekoHub_ESP") then
            obj.NekoHub_ESP:Destroy()
        end
    end
end

-- UI Principal
local window = Rayfield:CreateWindow({
    Name = "NekoHub - DOORS",
    LoadingTitle = "NekoHub Loader",
    LoadingSubtitle = "Rayfield (DOORS)",
    ConfigurationSaving = {Enabled = false},
    Discord = {Enabled = false},
    KeySystem = false,
    TitleColor = Color3.fromRGB(200,50,70),
})

local entidadeTab = window:CreateTab("Entidades", Color3.fromRGB(200,50,70))
local itensTab = window:CreateTab("Itens", Color3.fromRGB(200,50,70))

-- Entidades toggles
for entidade, _ in pairs(Entidades) do
    entidadeTab:CreateToggle({
        Name = "Avisar sobre " .. entidade,
        CurrentValue = false,
        Flag = entidade.."Alert",
        Callback = function(Value) Entidades[entidade] = Value end
    })
end

-- Itens toggle
itensTab:CreateToggle({
    Name = "Avisar e marcar itens na sala",
    CurrentValue = false,
    Flag = "ItensAlerta",
    Callback = function(Value) avisoItens = Value end
})

-- Entidades
workspace.ChildAdded:Connect(function(child)
    if typeof(child) == "Instance" then
        if child.Name == "RushMoving" and Entidades.Rush then Alert("NekoHub - AVISO!", "Entidade: Rush") end
        if child.Name == "AmbushMoving" and Entidades.Ambush then Alert("NekoHub - AVISO!", "Entidade: Ambush") end
        if child.Name == "Eyes" and Entidades.Eyes then Alert("NekoHub - AVISO!", "Entidade: Eyes") end
    end
end)
game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
    if value and Entidades.Seek then Alert("NekoHub - AVISO!", "Entidade: Seek") end
end)

game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    -- Limpa marcadores velhos
    if salaAtualCon then
        salaAtualCon:Disconnect()
        salaAtualCon = nil
    end

local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local rooms = workspace:FindFirstChild("CurrentRooms")
    local sala = rooms and rooms:FindFirstChild(tostring(v)) or nil

LimparMarcadores(sala)
    
-- Itens
if avisoItens and sala then
        for _, obj in ipairs(sala:GetDescendants()) do
            if ItensImportantes[obj.Name] then
                Alert("NekoHub - ITEM!", "Encontrado: " .. obj.Name)
                MarcarItem(obj)
            end
        end
        salaAtualCon = sala.DescendantAdded:Connect(function(child)
            if avisoItens and ItensImportantes[child.Name] then
                Alert("NekoHub - ITEM!", "Encontrado: " .. child.Name)
                MarcarItem(child)
            end
        end)
    end

-- Entidades dependentes da sala
if sala then
        if sala:FindFirstChild("Halt") and Entidades.Halt then
            Alert("NekoHub - AVISO!", "Entidade: Halt")
        end
        if tostring(v) == "50" and sala:FindFirstChild("Figure") and Entidades.Figure then
            Alert("NekoHub - AVISO!", "Entidade: Figure (Sala 50)")
        end
        if sala:FindFirstChild("Jack") and Entidades.Jack then
            Alert("NekoHub - AVISO!", "Entidade: Jack (Armário/Porta)")
        end
    end
end)

-- Screech
local function conectarScreech(character)
    if character and typeof(character) == "Instance" then
        character.ChildAdded:Connect(function(child)
            if child.Name == "Screech" and Entidades.Screech then
                Alert("NekoHub - AVISO!", "Entidade: Screech")
            end
        end)
    end
end

local plr = game.Players.LocalPlayer
if plr.Character then conectarScreech(plr.Character) end
plr.CharacterAdded:Connect(conectarScreech)
