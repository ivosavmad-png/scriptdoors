-- NekoHub - DOORS: Aviso de Entidades & Itens (The Hotel) - FIXED
-- github.com/ivosavmad-png & anonixzin

local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

---------------------------
-- SERVIÇOS
---------------------------
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

local plr = Players.LocalPlayer

---------------------------
-- SAFE GET (ANTI NIL)
---------------------------
local function WaitFor(path, name)
    local obj = path:FindFirstChild(name)
    while not obj do
        task.wait()
        obj = path:FindFirstChild(name)
    end
    return obj
end

local GameData = WaitFor(ReplicatedStorage, "GameData")
local LatestRoom = WaitFor(GameData, "LatestRoom")
local SeekerMoving = WaitFor(GameData, "SeekerMoving")

---------------------------
-- VARIÁVEIS
---------------------------
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
    ["Holy Grenade"] = true,
}

local avisoItens = false
local salaAtualCon

---------------------------
-- UI
---------------------------
local function Alert(txt, tempo)
    OrionLib:MakeNotification({
        Name = "NekoHub",
        Content = txt or "",
        Time = tempo or 5
    })
end

local Janela = OrionLib:MakeWindow({
    Name = "NekoHub - DOORS",
    HidePremium = true,
    SaveConfig = false,
    IntroText = "NekoHub Loaded",
})

Alert("Criado por anonixzin", 6)

---------------------------
-- ABA ENTIDADES
---------------------------
local entidadeAba = Janela:MakeTab({
    Name = "Entidades",
    Icon = "rbxassetid://4483345998",
})

for entidade, _ in pairs(Entidades) do
    entidadeAba:AddToggle({
        Name = "Avisar sobre " .. entidade,
        Default = false,
        Callback = function(state)
            Entidades[entidade] = state
        end
    })
end

---------------------------
-- ABA ITENS
---------------------------
local itensAba = Janela:MakeTab({
    Name = "Itens",
    Icon = "rbxassetid://4483345998",
})

itensAba:AddToggle({
    Name = "Avisar itens na sala",
    Default = false,
    Callback = function(state)
        avisoItens = state
    end
})

---------------------------
-- ENTIDADES (SEGURO)
---------------------------
Workspace.ChildAdded:Connect(function(child)
    if not child then return end

    if child.Name == "RushMoving" and Entidades.Rush then
        Alert("Entidade: Rush")
    elseif child.Name == "AmbushMoving" and Entidades.Ambush then
        Alert("Entidade: Ambush")
    elseif child.Name == "Eyes" and Entidades.Eyes then
        Alert("Entidade: Eyes")
    end
end)

---------------------------
-- SEEK
---------------------------
SeekerMoving.Changed:Connect(function(value)
    if value == true and Entidades.Seek then
        Alert("Entidade: Seek")
    end
end)

---------------------------
-- ROOMS / ENTIDADES
---------------------------
LatestRoom.Changed:Connect(function()
    local v = LatestRoom.Value

    local currentRooms = Workspace:FindFirstChild("CurrentRooms")
    if not currentRooms then return end

    local room = currentRooms:FindFirstChild(tostring(v))
    if not room then return end

    if room:FindFirstChild("Halt") and Entidades.Halt then
        Alert("Entidade: Halt")
    end

    if Entidades.Figure and v == 50 and room:FindFirstChild("Figure") then
        Alert("Entidade: Figure (Sala 50)")
    end

    if Entidades.Jack and room:FindFirstChild("Jack") then
        Alert("Entidade: Jack")
    end
end)

---------------------------
-- SCREECH (ANTI BUG)
---------------------------
local function listenScreech(character)
    if not character then return end

    character.ChildAdded:Connect(function(child)
        if child and child.Name == "Screech" and Entidades.Screech then
            Alert("Entidade: Screech")
        end
    end)
end

if plr.Character then
    listenScreech(plr.Character)
end

plr.CharacterAdded:Connect(listenScreech)

---------------------------
-- ITENS (MELHORADO)
---------------------------
LatestRoom.Changed:Connect(function()
    if not avisoItens then return end

    local v = LatestRoom.Value
    local currentRooms = Workspace:FindFirstChild("CurrentRooms")
    if not currentRooms then return end

    local sala = currentRooms:FindFirstChild(tostring(v))
    if not sala then return end

    if salaAtualCon then
        pcall(function()
            salaAtualCon:Disconnect()
        end)
        salaAtualCon = nil
    end

    for _, obj in ipairs(sala:GetDescendants()) do
        if ItensImportantes[obj.Name] then
            Alert("Encontrado: " .. obj.Name)
        end
    end

    salaAtualCon = sala.DescendantAdded:Connect(function(child)
        if avisoItens and child and ItensImportantes[child.Name] then
            Alert("Encontrado: " .. child.Name)
        end
    end)
end)

---------------------------
-- INIT
---------------------------
OrionLib:Init()
