-- NekoHub - DOORS: Aviso de Entidades & Itens (The Hotel)
-- Painel vermelho, multi-aba, notificação de créditos.
-- Somente para estudos! Não use para vantagem injusta.
-- github.com/ivosavmad-png & anonixzin

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Mensagem de créditos ao abrir o painel
Rayfield:Notify({
    Title = "NekoHub",
    Content = "Criado por anonixzin",
    Duration = 5,
    Image = 4483362458,
})

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

-- Itens para aviso
local ItensImportantes = {
    Crucifix = true,        -- Crucifixo
    Vitamins = true,        -- Vitaminas
    Lockpick = true,        -- Chave mestra
    Lighter = true,         -- Isqueiro
    Candle = true,          -- Vela
    Flashlight = true,      -- Lanterna
    ["Skeleton Key"] = true,-- Chave Esqueleto
    Key = true,             -- Chave de porta
    Gold = true,            -- Ouro
    Battery = true,         -- Bateria (usada em alguns updates)
    ["Holy Grenade"] = true -- Easter egg (se existir)
}

local window = Rayfield:CreateWindow({
    Name = "NekoHub - DOORS",
    LoadingTitle = "NekoHub Loader",
    LoadingSubtitle = "Aviso de Entidades & Itens (The Hotel)",
    ConfigurationSaving = {
        Enabled = false,
    },
    Discord = {
        Enabled = false,
    },
    KeySystem = false,
    TitleColor = Color3.fromRGB(200,50,70),
})

local entidadeTab = window:CreateTab("Entidades", Color3.fromRGB(200,50,70))
local itensTab = window:CreateTab("Itens", Color3.fromRGB(200,50,70))

-- Aba de entidades:
for entidade, _ in pairs(Entidades) do
    entidadeTab:CreateToggle({
        Name = "Avisar sobre " .. entidade,
        CurrentValue = false,
        Flag = entidade.."Alert",
        Callback = function(Value)
            Entidades[entidade] = Value
        end
    })
end

-- Aba de itens:
local avisoItens = false
itensTab:CreateToggle({
    Name = "Avisar itens na sala",
    CurrentValue = false,
    Flag = "ItensAlerta",
    Callback = function(Value)
        avisoItens = Value
    end
})

-- Função notificação Rayfield
local function Alert(title, msg)
    Rayfield:Notify({
        Title = title,
        Content = msg,
        Duration = 5,
        Image = 4483362458,
    })
end

-- Função de aviso de entidades
local function WatchEntidades()
    -- Rush e Ambush
    workspace.ChildAdded:Connect(function(child)
        if child.Name == "RushMoving" and Entidades.Rush then
            Alert("NekoHub - AVISO!", "Entidade detectada: Rush")
        elseif child.Name == "AmbushMoving" and Entidades.Ambush then
            Alert("NekoHub - AVISO!", "Entidade detectada: Ambush")
        end
    end)

 -- Seek
    game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
        if value and Entidades.Seek then
            Alert("NekoHub - AVISO!", "Entidade detectada: Seek")
        end
    end)

-- Screech (no Character)
    game.Players.LocalPlayer.Character.ChildAdded:Connect(function(child)
        if child.Name == "Screech" and Entidades.Screech then
            Alert("NekoHub - AVISO!", "Entidade detectada: Screech")
        end
    end)

 -- Eyes (na sala)
    workspace.ChildAdded:Connect(function(child)
        if child.Name == "Eyes" and Entidades.Eyes then
            Alert("NekoHub - AVISO!", "Entidade detectada: Eyes")
        end
    end)

-- Halt (sala)
    game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
        local room = workspace.CurrentRooms:FindFirstChild(tostring(game.ReplicatedStorage.GameData.LatestRoom.Value))
        if room and room:FindFirstChild("Halt") and Entidades.Halt then
            Alert("NekoHub - AVISO!", "Entidade detectada: Halt")
        end
    end)

 -- Figure (sala 50)
    workspace.CurrentRooms.ChildAdded:Connect(function(child)
        if child.Name == "50" and Entidades.Figure then
            local figure = child:FindFirstChild("Figure") or (child:WaitForChild("Figure", 7))
            if figure then
                Alert("NekoHub - AVISO!", "Entidade detectada: Figure (Sala 50)")
            end
        end
    end)

 -- Jack (porta/sala)
    workspace.CurrentRooms.ChildAdded:Connect(function(room)
        if room:FindFirstChild("Jack") and Entidades.Jack then
            Alert("NekoHub - AVISO!", "Entidade detectada: Jack (Armário/Porta)")
        end
    end)
end

-- Função de aviso de itens na sala
local function WatchItens()
    -- Ao entrar numa nova sala, verifica itens
    game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
        if not avisoItens then return end
        -- Busca pela sala atual
        local salaAtual = workspace.CurrentRooms:FindFirstChild(tostring(game.ReplicatedStorage.GameData.LatestRoom.Value))
        if salaAtual then
            for _, obj in pairs(salaAtual:GetDescendants()) do
                if ItensImportantes[obj.Name] then
                    Alert("NekoHub - ITEM!", "Encontrado: " .. obj.Name)
                end
            end
            -- Detecta spawn ao vivo
            salaAtual.DescendantAdded:Connect(function(child)
                if avisoItens and ItensImportantes[child.Name] then
                    Alert("NekoHub - ITEM!", "Encontrado: " .. child.Name)
                end
            end)
        end
    end)
end

WatchEntidades()
WatchItens()
