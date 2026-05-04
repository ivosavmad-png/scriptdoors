-- NekoHub - DOORS: Aviso de Entidades & Itens (The Hotel)
-- Corrigido, compatível, com créditos, multi-aba!
-- github.com/ivosavmad-png & anonixzin

-- Carregando OrionLib (painel e notificações super estáveis)
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

-- Referência de entidades e itens
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
    Crucifix = true, Vitamins = true, Lockpick = true, Lighter = true,
    Candle = true, Flashlight = true, ["Skeleton Key"] = true, Key = true,
    Gold = true, Battery = true, ["Holy Grenade"] = true
}

-- Função notificação Orion padrão
local function Alert(title, txt, tempo)
    OrionLib:MakeNotification({
        Name = title or "NekoHub",
        Content = txt or "",
        Time = tempo or 5
    })
end

-- Cria a janela principal
local Janela = OrionLib:MakeWindow({
    Name = "NekoHub - DOORS",
    HidePremium = true,
    IntroText = "NekoHub Loaded",
    SaveConfig = false,
    ConfigFolder = "NekoHubDOORS",
})

-- Notificação de créditos ao iniciar
Alert("NekoHub", "Criado por anonixzin", 6)

--//////////// ABA DE ENTIDADES ////////////
local entidadeAba = Janela:MakeTab({
    Name = "Entidades",
    Icon = "rbxassetid://4483345998", -- ícone de olho
    PremiumOnly = false
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

--//////////// ABA DE ITENS ////////////
local itensAba = Janela:MakeTab({
    Name = "Itens",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local avisoItens = false
itensAba:AddToggle({
    Name = "Avisar itens na sala",
    Default = false,
    Callback = function(state)
        avisoItens = state
    end
})

--///////// FUNÇÕES DE DETECÇÃO /////////

-- ENTIDADES: conexões únicas, listeners robustos
workspace.ChildAdded:Connect(function(child)
    if child.Name == "RushMoving" and Entidades.Rush then Alert("NekoHub - AVISO!", "Entidade: Rush") end
    if child.Name == "AmbushMoving" and Entidades.Ambush then Alert("NekoHub - AVISO!", "Entidade: Ambush") end
    if child.Name == "Eyes" and Entidades.Eyes then Alert("NekoHub - AVISO!", "Entidade: Eyes") end
end)

game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
    if value and Entidades.Seek then Alert("NekoHub - AVISO!", "Entidade: Seek") end
end)

game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local room = workspace.CurrentRooms:FindFirstChild(tostring(v))
    if room then
        if room:FindFirstChild("Halt") and Entidades.Halt then
            Alert("NekoHub - AVISO!", "Entidade: Halt")
        end
        if Entidades.Figure and tostring(v) == "50" then
            local ok, figure = pcall(function() return room:FindFirstChild("Figure") or room:WaitForChild("Figure",7) end)
            if ok and figure then Alert("NekoHub - AVISO!", "Entidade: Figure (Sala 50)") end
        end
        if Entidades.Jack and room:FindFirstChild("Jack") then
            Alert("NekoHub - AVISO!", "Entidade: Jack (Armário/Porta)")
        end
    end
end)

-- Screech no Character (tolerante ao respawn)
local function listenScreech(character)
    if not character then return end
    character.ChildAdded:Connect(function(child)
        if child.Name == "Screech" and Entidades.Screech then
            Alert("NekoHub - AVISO!", "Entidade: Screech")
        end
    end)
end
local plr = game.Players.LocalPlayer
listenScreech(plr.Character or plr.CharacterAdded:Wait())
plr.CharacterAdded:Connect(listenScreech)

-- ITENS: checagem na entrada da sala e ao spawnar coisa nova
local salaAtualCon
game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    local v = game.ReplicatedStorage.GameData.LatestRoom.Value
    local sala = workspace.CurrentRooms:FindFirstChild(tostring(v))
    if salaAtualCon then pcall(function() salaAtualCon:Disconnect() end) end
    if sala and avisoItens then
        for _, obj in ipairs(sala:GetDescendants()) do
            if ItensImportantes[obj.Name] then
                Alert("NekoHub - ITEM!", "Encontrado: " .. obj.Name)
            end
        end
        salaAtualCon = sala.DescendantAdded:Connect(function(child)
            if avisoItens and ItensImportantes[child.Name] then
                Alert("NekoHub - ITEM!", "Encontrado: " .. child.Name)
            end
        end)
    end
end)

-- OrionLib required loop to keep GUI active
OrionLib:Init()
