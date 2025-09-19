# BakiHub
--================================================--
-- BakiHub Universal
-- Suporte: Blox Fruits | Slayer Online | Default
--================================================--

-- // Serviços
local Players = game:GetService("Players")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer

--================================================--
-- Anti Kick / Anti AFK
--================================================--
for _, c in pairs(getconnections or get_signal_cons or {}) do
    if typeof(c) == "function" then
        c:Disable()
    end
end

player.Idled:Connect(function()
    VirtualUser:ClickButton2(Vector2.new())
end)

local mt = getrawmetatable(game)
setreadonly(mt,false)
local old = mt.__namecall
mt.__namecall = newcclosure(function(self,...)
    local method = getnamecallmethod()
    if method == "Kick" then
        return nil
    end
    return old(self,...)
end)

--================================================--
-- GUI Principal
--================================================--
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BakiHub"
screenGui.Parent = game:GetService("CoreGui")
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 500, 0, 300)
mainFrame.Position = UDim2.new(0.25, 0, 0.25, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
mainFrame.BorderColor3 = Color3.fromRGB(120,120,120)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local UICorner = Instance.new("UICorner", mainFrame)
UICorner.CornerRadius = UDim.new(0,10)

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "bakiHub"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(200,200,200)

-- Painéis
local leftFrame = Instance.new("ScrollingFrame", mainFrame)
leftFrame.Size = UDim2.new(0.3,0,0.85,0)
leftFrame.Position = UDim2.new(0,0,0.15,0)
leftFrame.BackgroundColor3 = Color3.fromRGB(35,35,35)
leftFrame.ScrollBarThickness = 4
Instance.new("UIListLayout", leftFrame).Padding = UDim.new(0,5)

local rightFrame = Instance.new("ScrollingFrame", mainFrame)
rightFrame.Size = UDim2.new(0.7,0,0.85,0)
rightFrame.Position = UDim2.new(0.3,0,0.15,0)
rightFrame.BackgroundColor3 = Color3.fromRGB(40,40,40)
rightFrame.ScrollBarThickness = 4
Instance.new("UIListLayout", rightFrame).Padding = UDim.new(0,5)

-- Minimizar
local minimizedButton = Instance.new("ImageButton", screenGui)
minimizedButton.Size = UDim2.new(0,60,0,60)
minimizedButton.Position = UDim2.new(0.05,0,0.8,0)
minimizedButton.Image = "rbxassetid://1234567890" -- troque pelo ID da sua imagem
minimizedButton.BackgroundTransparency = 1
minimizedButton.Visible = false
minimizedButton.Draggable = true
minimizedButton.ZIndex = 9999

local function toggleMinimize()
    if mainFrame.Visible then
        mainFrame.Visible = false
        minimizedButton.Visible = true
    else
        minimizedButton.Visible = false
        mainFrame.Visible = true
    end
end
minimizedButton.MouseButton1Click:Connect(toggleMinimize)

--================================================--
-- Detectar Jogo
--================================================--
local GAMES = {
    ["Blox Fruits"] = {2753915549, 4442272183, 7449423635},
    ["Slayer Online"] = {123456789}, -- coloque o PlaceId real
}
local function detectGame()
    for name, ids in pairs(GAMES) do
        for _, id in ipairs(ids) do
            if game.PlaceId == id then return name end
        end
    end
    return "Default"
end
local detectedGame = detectGame()
print("Jogo detectado:", detectedGame)

--================================================--
-- Configurações
--================================================--
local config = {
    alturaVoo = 15,
    delayAtaque = 0.2
}

--================================================--
-- Funções Extras
--================================================--
local state = {autoFarmBlox=false, autoFarmSlayer=false}

local function pegarMissaoBlox()
    if workspace:FindFirstChild("NPCs") then
        for _, npc in ipairs(workspace.NPCs:GetChildren()) do
            if npc:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame * CFrame.new(0,0,3)
                task.wait(0.5)
                VirtualUser:ClickButton1(Vector2.new())
                break
            end
        end
    end
end

local function pegarMissaoSlayer()
    if workspace:FindFirstChild("Quests") then
        for _, npc in ipairs(workspace.Quests:GetChildren()) do
            if npc:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame * CFrame.new(0,0,3)
                task.wait(0.5)
                VirtualUser:ClickButton1(Vector2.new())
                break
            end
        end
    end
end

local function autoFarmBloxLoop()
    while state.autoFarmBlox do
        task.wait(config.delayAtaque)
        local char = player.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then continue end
        local hrp = char.HumanoidRootPart
        local inimigo
        if workspace:FindFirstChild("Enemies") then
            for _, mob in ipairs(workspace.Enemies:GetChildren()) do
                if mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                    inimigo = mob break
                end
            end
        end
        if inimigo then
            hrp.CFrame = CFrame.new(inimigo.HumanoidRootPart.Position + Vector3.new(0,config.alturaVoo,0))
            hrp.Velocity = Vector3.new(0,0,0)
            VirtualUser:ClickButton1(Vector2.new())
        end
    end
end

local function autoFarmSlayerLoop()
    while state.autoFarmSlayer do
        task.wait(config.delayAtaque)
        local char = player.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then continue end
        local hrp = char.HumanoidRootPart
        local mob
        if workspace:FindFirstChild("Mobs") then
            for _, npc in ipairs(workspace.Mobs:GetChildren()) do
                if npc:FindFirstChild("HumanoidRootPart") and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                    mob = npc break
                end
            end
        end
        if mob then
            hrp.CFrame = CFrame.new(mob.HumanoidRootPart.Position + Vector3.new(0,config.alturaVoo,0))
            hrp.Velocity = Vector3.new(0,0,0)
            VirtualUser:ClickButton1(Vector2.new())
        end
    end
end

--================================================--
-- Criar Botões
--================================================--
local function createButton(parent,text,callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1,-10,0,30)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.Text = text
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.MouseButton1Click:Connect(callback)
end

-- Categorias dinâmicas
local categorias = {
    ["Blox Fruits"] = {"Player","Auto Farm","Configurações","Minimizar"},
    ["Slayer Online"] = {"Player","Auto Farm","Configurações","Minimizar"},
    ["Default"] = {"Player","Menu","Configurações","Minimizar"}
}
local acoes = {
    ["Blox Fruits"] = {
        ["Auto Farm"] = {"Ativar Auto Farm"},
        ["Configurações"] = {"Altura +5","Altura -5","Atk mais rápido","Atk mais lento"},
        ["Minimizar"] = {"Minimizar GUI"}
    },
    ["Slayer Online"] = {
        ["Auto Farm"] = {"Ativar Auto Farm"},
        ["Configurações"] = {"Altura +5","Altura -5","Atk mais rápido","Atk mais lento"},
        ["Minimizar"] = {"Minimizar GUI"}
    },
    ["Default"] = {
        ["Menu"] = {"Abrir Configurações"},
        ["Configurações"] = {"Altura +5","Altura -5","Atk mais rápido","Atk mais lento"},
        ["Minimizar"] = {"Minimizar GUI"}
    }
}

-- Criar botões laterais
for _, cat in ipairs(categorias[detectedGame]) do
    createButton(leftFrame, cat, function()
        rightFrame:ClearAllChildren()
        Instance.new("UIListLayout", rightFrame).Padding = UDim.new(0,5)
        for _, acao in ipairs(acoes[detectedGame][cat] or {}) do
            if acao == "Ativar Auto Farm" then
                createButton(rightFrame, acao, function()
                    if detectedGame == "Blox Fruits" then
                        pegarMissaoBlox()
                        state.autoFarmBlox = not state.autoFarmBlox
                        if state.autoFarmBlox then
                            task.spawn(autoFarmBloxLoop)
                        end
                    elseif detectedGame == "Slayer Online" then
                        pegarMissaoSlayer()
                        state.autoFarmSlayer = not state.autoFarmSlayer
                        if state.autoFarmSlayer then
                            task.spawn(autoFarmSlayerLoop)
                        end
                    end
                end)
            elseif acao == "Altura +5" then
                createButton(rightFrame, acao, function() config.alturaVoo = config.alturaVoo + 5 end)
            elseif acao == "Altura -5" then
                createButton(rightFrame, acao, function() config.alturaVoo = math.max(5, config.alturaVoo - 5) end)
            elseif acao == "Atk mais rápido" then
                createButton(rightFrame, acao, function() config.delayAtaque = math.max(0.05, config.delayAtaque - 0.05) end)
            elseif acao == "Atk mais lento" then
                createButton(rightFrame, acao, function() config.delayAtaque = config.delayAtaque + 0.05 end)
            elseif acao == "Minimizar GUI" then
                createButton(rightFrame, acao, toggleMinimize)
            else
                createButton(rightFrame, acao, function() print("Executado:", acao) end)
            end
        end
    end)
end
