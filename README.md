-- NekoHub - DOORS: Entidades & Itens Corrigido
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

Rayfield:Notify({
    Title = "NekoHub",
    Content = "Criado por anonixzin",
    Duration = 5,
    Image = 4483362458,
})

local Entidades = {
    Rush = false, Ambush = false, Seek = false, Screech = false,
    Eyes = false, Halt = false, Figure = false, Jack = false,
}
local ItensImportantes = {
    Crucifix = true, Vitamins = true, Lockpick = true, Lighter = true,
    Candle = true, Flashlight = true, ["Skeleton Key"] = true, Key = true,
    Gold = true, Battery = true, ["Holy Grenade"] = true,
}

local window = Rayfield:CreateWindow({
    Name = "NekoHub - DOORS",
    LoadingTitle = "NekoHub Loader",
    LoadingSubtitle = "Aviso de Entidades & Itens (The Hotel)",
    TitleColor = Color3.fromRGB(200,50,70),
})

local entidadeTab = window:CreateTab("Entidades", Color3.fromRGB(200,50,70))
local itensTab = window:CreateTab("Itens", Color3.fromRGB(200,50,70))

for entidade, _ in pairs(Entidades) do
    entidadeTab:CreateToggle({
        Name = "Avisar sobre " .. entidade,
        CurrentValue = false,
        Flag = entidade.."Alert",
        Callback = function(Value) Entidades[entidade] = Value end
    })
end

local avisoItens = false
itensTab:CreateToggle({
    Name = "Avisar itens na sala",
    CurrentValue = false,
    Flag = "ItensAlerta",
    Callback = function(Value) avisoItens = Value end
})

local function Alert(title, msg)
    Rayfield:Notify({ Title = title, Content = msg, Duration = 5, Image = 4483362458 })
end

-- Listener de entidades
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

        if Entidades.Figure and tostring(v) ==_

