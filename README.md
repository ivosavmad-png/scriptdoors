-- NekoHub Autofarm Script para DOORS
-- Tema vermelho, interface simples
-- Feito apenas para fins educacionais! Não use para trapacear.

-- Carregar UI Rayfield ou outro UI-framework
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local window = Rayfield:CreateWindow({
    Name = "NekoHub - DOORS",
    LoadingTitle = "NekoHub Loader",
    LoadingSubtitle = "by github.com/ivosavmad-png",
    ConfigurationSaving = {
        Enabled = false,
    },
    Discord = {
        Enabled = false,
    },
    KeySystem = false,
    TitleColor = Color3.fromRGB(200, 50, 70) -- Vermelho
})

local mainTab = window:CreateTab("Autofarm", Color3.fromRGB(200, 50, 70))

local autofarmEnabled = false

mainTab:CreateToggle({
    Name = "Ativar Autofarm",
    CurrentValue = false,
    Flag = "AutoFarmToggle",
    Callback = function(Value)
        autofarmEnabled = Value
    end
})

-- Autofarm: atravessa as salas automaticamente (simples)
spawn(function()
    while true do
        task.wait(0.5)
        if autofarmEnabled and game.PlaceId == 6516141723 then -- DOORS
            local plr = game.Players.LocalPlayer
            local chr = plr.Character
            local root = chr and chr:FindFirstChild("HumanoidRootPart")
            local currentRoom = workspace.CurrentRooms:FindFirstChild(tostring(game.ReplicatedStorage.GameData.LatestRoom.Value))
            local nextRoom = workspace.CurrentRooms:FindFirstChild(tostring(game.ReplicatedStorage.GameData.LatestRoom.Value + 1))
            if root then
                if nextRoom and nextRoom:FindFirstChild("Door") then
                    root.CFrame = nextRoom.Door.CFrame + Vector3.new(0,0,2)
                elseif currentRoom and currentRoom:FindFirstChild("Door") then
                    -- Teleporta para o final da sala atual
                    root.CFrame = currentRoom.Door.CFrame + Vector3.new(0,0,2)
                end
            end
        end
    end
end)
