-- NekoHub - DOORS: Aviso de Entidades (The Hotel)
-- Tema vermelho, seleção das entidades que deseja alerta.
-- Apenas fins educacionais! Não use para vantagem injusta.

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

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

local window = Rayfield:CreateWindow({
    Name = "NekoHub - DOORS",
    LoadingTitle = "NekoHub Loader",
    LoadingSubtitle = "Aviso de Entidades (The Hotel)",
    ConfigurationSaving = {
        Enabled = false,
    },
    Discord = {
        Enabled = false,
    },
    KeySystem = false,
    TitleColor = Color3.fromRGB(200,50,70),
})

local mainTab = window:CreateTab("Entidades", Color3.fromRGB(200, 50, 70))

-- Criando toggles para seleção das entidades desejadas
for entidade, _ in pairs(Entidades) do
    mainTab:CreateToggle({
        Name = "Avisar sobre " .. entidade,
        CurrentValue = false,
        Flag = entidade.."Alert",
        Callback = function(Value)
            Entidades[entidade] = Value
        end
    })
end

-- Função de alerta
local function AlertEntidade(nome)
    Rayfield:Notify({
        Title = "NekoHub - AVISO!",
        Content = "Entidade detectada: " .. nome,
        Duration = 5,
        Image = 4483362458, -- Ícone Roblox alert
    })
end

-- Checagem de entidades do The Hotel (Rush, Ambush, Seek, etc.)
local function WatchEntidades()
    -- Rush e Ambush
    workspace.ChildAdded:Connect(function(child)
        if child.Name == "RushMoving" and Entidades.Rush then
            AlertEntidade("Rush")
        elseif child.Name == "AmbushMoving" and Entidades.Ambush then
            AlertEntidade("Ambush")
        end
    end)

    -- seek
    game.ReplicatedStorage.GameData.SeekerMoving.Changed:Connect(function(value)
        if value and Entidades.Seek then
            AlertEntidade("Seek")
        end
    end)

    -- Screech (geralmente é um som do player, limitação: alerta ao som gerar no Character)
    game.Players.LocalPlayer.Character.ChildAdded:Connect(function(child)
        if child.Name == "Screech" and Entidades.Screech then
            AlertEntidade("Screech")
        end
    end)

    -- Eyes (ao spawnar na sala)
    workspace.ChildAdded:Connect(function(child)
        if child.Name == "Eyes" and Entidades.Eyes then
            AlertEntidade("Eyes")
        end
    end)

    -- Halt (detectado no GameData)
    game.ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
        local room = workspace.CurrentRooms:FindFirstChild(tostring(game.ReplicatedStorage.GameData.LatestRoom.Value))
        if room and room:FindFirstChild("Halt") and Entidades.Halt then
            AlertEntidade("Halt")
        end
    end)

    -- Figure (Sala 50, Library)
    workspace.CurrentRooms.ChildAdded:Connect(function(child)
        if child.Name == "50" and Entidades.Figure then
            local figure = child:FindFirstChild("Figure") or (child:WaitForChild("Figure", 7))
            if figure then
                AlertEntidade("Figure (Sala 50)")
            end
        end
    end)

    -- Jack (quando porta contém "Jack")
    workspace.CurrentRooms.ChildAdded:Connect(function(room)
        if room:FindFirstChild("Jack") and Entidades.Jack then
            AlertEntidade("Jack (Armário/Porta)")
        end
    end)
end

WatchEntidades()
