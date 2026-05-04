-- NekoHub - DOORS: Aviso de Entidades & Itens (The Hotel) - SEM nil
-- github.com/ivosavmad-png & anonixzin

local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

---------------------------
-- VARIÁVEIS GLOBAIS
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
local salaAtualCon -- conexão para DescendantAdded

---------------------------
-- FUNÇÕES DE INTERFACE
---------------------------
local function Alert(txt, tempo)
    OrionLib:MakeNotification({
        Name = "NekoHub",
        Content = txt or "",
        Time = tempo or 5
    })
end

---------------------------
-- CRIANDO PAINEL
---------------------------
local Janela = OrionLib:MakeWindow({
    Name = "NekoHub - DOORS",
    HidePremium = true,
    SaveConfig = false,
    IntroText = "NekoHub Loaded",
})

-- Mensagem de créditos ao iniciar
Alert("Criado por anonixzin", 6)

-- ABA ENTIDADES
local entidadeAba = Janela:MakeTab({
    Name = "Entidades",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})
for entidade, _ in pairs(Entidades) do
    entidadeAba:AddToggle({
        Name = "Avisar sobre " .. entidade,
        Default = false,
        Callback = function(state)
            Entidades[entidade] = state == true
        end
    })
end

-- ABA ITENS
local itensAba = Janela:MakeTab({
    Name = "Itens",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})
itensAba:AddToggle({
    Name = "Avisar itens na sala",
    Default = false,
    Callback = function(state)
        avisoItens = state == true
    end
})

---------------------------
-- AVISOS DE ENTIDADES
---------------------------
workspace.ChildAdded:Connect(function(child)
    if typeof(child) == "Instance" then
        if child.Name == "RushMoving" and Entidades.Rush then Alert("Entidade: Rush") end
        if child.Name == "AmbushMoving" and Entidades.Ambush then Alert("Entidade: Ambush") end
        if child.Name == "Eyes" and Entidades.Eyes then Alert("Entidade: Eyes") end
    end
end)

game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
    if value == true and Entidades.Seek then
        Alert("Entidade: Seek")
    end
end)

game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local room = workspace:FindFirstChild("CurrentRooms") and workspace.CurrentRooms:FindFirstChild(tostring(v))
    if room then
        if room:FindFirstChild("Halt") and Entidades.Halt then
            Alert("Entidade: Halt")
        end
        if Entidades.Figure and tostring(v) == "50" and room:FindFirstChild("Figure") then
            Alert("Entidade: Figure (Sala 50)")
        end
        if Entidades.Jack and room:FindFirstChild("Jack") then
            Alert("Entidade: Jack (Armário/Porta)")
        end
    end
end)

-- Screech (suporta respawn)
local function listenScreech(character)
    if character and character:IsA("Model") then
        character.ChildAdded:Connect(function(child)
            if child.Name == "Screech" and Entidades.Screech then
                Alert("Entidade: Screech")
            end
        end)
    end
end
local plr = game.Players.LocalPlayer
if plr.Character then listenScreech(plr.Character) end
plr.CharacterAdded:Connect(listenScreech)

---------------------------
-- AVISOS DE ITENS NA SALA
---------------------------
game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    if not avisoItens then return end
    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local sala = workspace:FindFirstChild("CurrentRooms") and workspace.CurrentRooms:FindFirstChild(tostring(v))
    if salaAtualCon then
        pcall(function() salaAtualCon:Disconnect() end)
        salaAtualCon = nil
    end
    if sala then
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
    end
end)

OrionLib:Init()
