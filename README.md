-- Spawn de armas no jogador
-- A arma precisa estar em ServerStorage > Items

local ServerStorage = game:GetService("ServerStorage")
local Players = game:GetService("Players")

local pastaDeItens = ServerStorage:WaitForChild("Items") -- crie essa pasta e coloque suas Tools (armas)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        local backpack = player:WaitForChild("Backpack")

        -- armas para dar
        local armasParaDar = {"StarterPistol"} -- adicione mais nomes aqui
        for _, armaNome in ipairs(armasParaDar) do
            local arma = pastaDeItens:FindFirstChild(armaNome)
            if arma then
                arma:Clone().Parent = backpack
            end
        end
    end)
end)-- ItemManager (ModuleScript)
-- Gerencia itens do jogador

local ItemManager = {}
ItemManager.__index = ItemManager

local playerItems = {}

function ItemManager:InitPlayer(player)
    playerItems[player.UserId] = playerItems[player.UserId] or {}
    local starter = {"StarterPistol", "StarterCar"}
    for _, id in ipairs(starter) do
        playerItems[player.UserId][id] = true
    end
end

function ItemManager:HasItem(player, itemId)
    local t = playerItems[player.UserId]
    if not t then return false end
    return t[itemId] == true
end

function ItemManager:GiveItem(player, itemId)
    playerItems[player.UserId] = playerItems[player.UserId] or {}
    playerItems[player.UserId][itemId] = true
end

function ItemManager:RemoveItem(player, itemId)
    local t = playerItems[player.UserId]
    if not t then return false end
    t[itemId] = nil
    return true
end

function ItemManager:GetAll(player)
    return playerItems[player.UserId] or {}
end

game.Players.PlayerRemoving:Connect(function(plr)
    playerItems[plr.UserId] = nil
end)

return ItemManager-- CarSpawner (ModuleScript)
-- Spawn de carros no jogo

local CarSpawner = {}
CarSpawner.__index = CarSpawner

local ServerStorage = game:GetService("ServerStorage")
local carsFolder = ServerStorage:WaitForChild("Cars")

function CarSpawner:CarExists(carName)
    return carsFolder:FindFirstChild(carName) ~= nil
end

function CarSpawner:SpawnCar(carName, position)
    local model = carsFolder:FindFirstChild(carName)
    if not model then return nil, "Car not found" end
    local clone = model:Clone()
    clone.PrimaryPart = clone.PrimaryPart or clone:FindFirstChildWhichIsA("BasePart")
    if not clone.PrimaryPart then
        return nil, "Car model has no PrimaryPart"
    end
    clone:SetPrimaryPartCFrame(CFrame.new(position) * CFrame.Angles(0, math.rad(math.random(0,360)), 0))
    clone.Parent = workspace
    return clone
end

return CarSpawner-- GameServer (Script)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Modules = ReplicatedStorage:WaitForChild("Modules")
local ItemManager = require(Modules:WaitForChild("ItemManager"))
local CarSpawner = require(Modules:WaitForChild("CarSpawner"))

local RemoteEvents = ReplicatedStorage:WaitForChild("RemoteEvents")
local RE_SpawnCar = RemoteEvents:WaitForChild("RequestSpawnCar")
local RE_Equip = RemoteEvents:WaitForChild("RequestEquipItem")
local RE_Use = RemoteEvents:WaitForChild("RequestUseItem")

local CAR_SPAWN_DISTANCE = 10

-- inicializa jogador
Players.PlayerAdded:Connect(function(player)
    ItemManager:InitPlayer(player)
end)

-- spawn de carro
RE_SpawnCar.OnServerEvent:Connect(function(player, carName)
    if type(carName) ~= "string" then return end
    if not ItemManager:HasItem(player, carName) then return end
    local character = player.Character
    if not character or not character.PrimaryPart then ret-- LocalScript para interface simples
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local RemoteEvents = ReplicatedStorage:WaitForChild("RemoteEvents")
local RE_SpawnCar = RemoteEvents:WaitForChild("RequestSpawnCar")
local RE_Equip = RemoteEvents:WaitForChild("RequestEquipItem")
local RE_Use = RemoteEvents:WaitForChild("RequestUseItem")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MiniCityGUI"
screenGui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,220,0,90)
frame.Position = UDim2.new(0,10,0,10)
frame.BackgroundTransparency = 0.3
frame.Parent = screenGui

local spawnCarBtn = Instance.new("TextButton")
spawnCarBtn.Parent = frame
spawnCarBtn.Size = UDim2.new(1, -10, 0, 30)
spawnCarBtn.Position = UDim2.new(0,5,0,5)
spawnCarBtn.Text = "Spawn Car (StarterCar)"
spawnCarBtn.MouseButton1Click:Conn-- Script dentro da Tool (StarterPistol)
local tool = script.Parent
local player = nil
local mouse = nil

local DAMAGE = 20
local BULLET_SPEED = 200
local BULLET_LIFETIME = 3

tool.Equipped:Connect(function(m)
	player = game.Players:GetPlayerFromCharacter(tool.Parent)
	mouse = m
end)

tool.Activated:Connect(function()
	if not player or not mouse then return end
	local char = player.Character
	if not char or not char.PrimaryPart then return end

	local bullet = Instance.new("Part")
	bullet.Shape = Enum.PartType.Ball
	bullet.Material = Enum.Material.Neon
	bullet.Color = Color3.new(1, 1, 0)
	bullet.Size = Vector3.new(0.3, 0.3, 0.3)
	bullet.CFrame = char.PrimaryPart.CFrame * CFrame.new(0, 1, -2)
	bullet.CanCollide = false
	bullet.Anchored = false
	bullet.Parent = workspace

	local bv = Instance.new("BodyVelocity")
	bv.Velocity = (mouse.Hit.Position - bullet.Position).Unit * BULLET_SPEED
	bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
	bv.Parent = bullet

	bullet.Touched:Connect(function(hit)
