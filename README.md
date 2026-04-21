--[[
    Script: RoRo Hub para King Legacy
    Funcionalidades:
    - Interface Bonita e Moderna
    - Auto Roletar Frutas (Gacha)
    - Auto Comprar Frutas na Loja
    - ESP de Frutas no Chão
    - ESP de Players
    - Tecla de toggle (Insert) para abrir/fechar
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- // Verifica se já existe para não duplicar
if _G.RoRoHubLoaded then
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "RoRo Hub";
        Text = "O Hub já está carregado!";
        Duration = 2;
    })
    return
end
_G.RoRoHubLoaded = true

-- // Biblioteca de UI (DLL Simples e Moderna)
local Library = {
    colors = {
        primary = Color3.fromRGB(255, 85, 85);
        secondary = Color3.fromRGB(30, 30, 40);
        background = Color3.fromRGB(20, 20, 25);
        text = Color3.fromRGB(255, 255, 255);
        accent = Color3.fromRGB(255, 170, 85);
    }
}

-- // Criação da ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "RoRoHub"
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false
gui.Parent = CoreGui

-- // Função para criar botões arredondados e elegantes
local function createButton(parent, text, callback, color)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 180, 0, 35)
    button.Position = UDim2.new(0.5, -90, 0, 0)
    button.BackgroundColor3 = color or Library.colors.primary
    button.BackgroundTransparency = 0.15
    button.TextColor3 = Library.colors.text
    button.Text = text
    button.TextSize = 14
    button.Font = Enum.Font.GothamSemibold
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.Parent = parent
    
    -- Efeito Hover
    local hovering = false
    button.MouseEnter:Connect(function()
        hovering = true
        while hovering and button.Parent do
            button.BackgroundTransparency = math.max(0, button.BackgroundTransparency - 0.05)
            button.Size = UDim2.new(0, 185, 0, 38)
            task.wait(0.05)
        end
    end)
    button.MouseLeave:Connect(function()
        hovering = false
        button.BackgroundTransparency = 0.15
        button.Size = UDim2.new(0, 180, 0, 35)
    end)
    button.MouseButton1Click:Connect(callback)
    
    return button
end

-- // Frame Principal com estilo "Neumorphism"
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 500)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
mainFrame.BackgroundColor3 = Library.colors.background
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = gui

-- Sombras e cantos arredondados
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

local shadow = Instance.new("UIStroke")
shadow.Color = Color3.fromRGB(0, 0, 0)
shadow.Thickness = 2
shadow.Transparency = 0.8
shadow.Parent = mainFrame

-- // Top Bar (Draggable)
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 45)
topBar.BackgroundColor3 = Library.colors.secondary
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 1, 0)
title.BackgroundTransparency = 1
title.Text = "RoRo Hub ✦ King Legacy"
title.TextColor3 = Library.colors.text
title.TextSize = 18
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Center
title.Parent = topBar

-- Botão Fechar (X)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 1, 0)
closeBtn.Position = UDim2.new(1, -35, 0, 0)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "✕"
closeBtn.TextColor3 = Library.colors.text
closeBtn.TextSize = 20
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = topBar
closeBtn.MouseButton1Click:Connect(function()
    gui.Enabled = false
end)

-- Drag logic
local dragging = false
local dragInput, dragStart, startPos

topBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

topBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- // Conteúdo (Scrolling Frame)
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, 0, 1, -45)
scrollFrame.Position = UDim2.new(0, 0, 0, 45)
scrollFrame.BackgroundTransparency = 1
scrollFrame.ScrollBarThickness = 6
scrollFrame.ScrollBarImageColor3 = Library.colors.primary
scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 320)
scrollFrame.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 15)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = scrollFrame

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 20)
padding.PaddingBottom = UDim.new(0, 20)
padding.Parent = scrollFrame

-- // Seção de Frutas
local fruitSection = Instance.new("Frame")
fruitSection.Size = UDim2.new(1, -30, 0, 140)
fruitSection.BackgroundColor3 = Library.colors.secondary
fruitSection.BackgroundTransparency = 0.5
fruitSection.BorderSizePixel = 0
fruitSection.Parent = scrollFrame

local sectionCorner = Instance.new("UICorner")
sectionCorner.CornerRadius = UDim.new(0, 8)
sectionCorner.Parent = fruitSection

local sectionTitle = Instance.new("TextLabel")
sectionTitle.Size = UDim2.new(1, 0, 0, 30)
sectionTitle.BackgroundTransparency = 1
sectionTitle.Text = "🍎 Sistema de Frutas"
sectionTitle.TextColor3 = Library.colors.accent
sectionTitle.TextSize = 16
sectionTitle.Font = Enum.Font.GothamBold
sectionTitle.TextXAlignment = Enum.TextXAlignment.Center
sectionTitle.Parent = fruitSection

-- Botão Auto Roletar
local autoRollBtn = createButton(fruitSection, "🎲 Auto Roletar", function()
    _G.AutoRoll = not _G.AutoRoll
    autoRollBtn.BackgroundColor3 = _G.AutoRoll and Color3.fromRGB(50, 200, 50) or Library.colors.primary
    autoRollBtn.Text = _G.AutoRoll and "🎲 Roletando..." or "🎲 Auto Roletar"
end, Library.colors.primary)
autoRollBtn.Position = UDim2.new(0.5, -95, 0, 45)

-- Botão Auto Comprar
local autoBuyBtn = createButton(fruitSection, "💰 Auto Comprar", function()
    _G.AutoBuy = not _G.AutoBuy
    autoBuyBtn.BackgroundColor3 = _G.AutoBuy and Color3.fromRGB(50, 200, 50) or Library.colors.primary
    autoBuyBtn.Text = _G.AutoBuy and "💰 Comprando..." or "💰 Auto Comprar"
end, Library.colors.primary)
autoBuyBtn.Position = UDim2.new(0.5, -95, 0, 95)

-- // Seção de ESP
local espSection = Instance.new("Frame")
espSection.Size = UDim2.new(1, -30, 0, 140)
espSection.BackgroundColor3 = Library.colors.secondary
espSection.BackgroundTransparency = 0.5
espSection.BorderSizePixel = 0
espSection.Parent = scrollFrame

local espCorner = Instance.new("UICorner")
espCorner.CornerRadius = UDim.new(0, 8)
espCorner.Parent = espSection

local espTitle = Instance.new("TextLabel")
espTitle.Size = UDim2.new(1, 0, 0, 30)
espTitle.BackgroundTransparency = 1
espTitle.Text = "👁️ Visual ESP"
espTitle.TextColor3 = Library.colors.accent
espTitle.TextSize = 16
espTitle.Font = Enum.Font.GothamBold
espTitle.TextXAlignment = Enum.TextXAlignment.Center
espTitle.Parent = espSection

-- Botão ESP Frutas
local espFruitBtn = createButton(espSection, "🍌 ESP Frutas", function()
    _G.ESP_Fruits = not _G.ESP_Fruits
    espFruitBtn.BackgroundColor3 = _G.ESP_Fruits and Color3.fromRGB(50, 200, 50) or Library.colors.primary
    espFruitBtn.Text = _G.ESP_Fruits and "🥝 Frutas Visíveis" or "🍌 ESP Frutas"
end, Library.colors.primary)
espFruitBtn.Position = UDim2.new(0.5, -95, 0, 45)

-- Botão ESP Players
local espPlayerBtn = createButton(espSection, "👤 ESP Players", function()
    _G.ESP_Players = not _G.ESP_Players
    espPlayerBtn.BackgroundColor3 = _G.ESP_Players and Color3.fromRGB(50, 200, 50) or Library.colors.primary
    espPlayerBtn.Text = _G.ESP_Players and "🎯 Players Visíveis" or "👤 ESP Players"
end, Library.colors.primary)
espPlayerBtn.Position = UDim2.new(0.5, -95, 0, 95)

-- Atualiza CanvasSize
listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 20)
end)

-- // Toggle com a tecla Insert
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Insert then
        gui.Enabled = not gui.Enabled
    end
end)

-- // CORES DE NOTIFICAÇÃO ESTILIZADAS
local function notify(title, text, duration)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = title;
        Text = text;
        Duration = duration or 2;
    })
end

notify("RoRo Hub", "Carregado com sucesso! Pressione Insert para abrir.", 3)

-- ===========================================
-- // LÓGICA DAS FUNÇÕES (Auto Roll, Auto Buy e ESPs)
-- ===========================================

-- // Função para roletar fruta (simula clique no botão de rolar)
local function rollFruit()
    -- Procura pelo botão de roletar fruta (Gacha)
    local buttons = game:GetService("CoreGui"):FindFirstChild("RobloxGui", true)
    if not buttons then
        -- Tenta achar pela tela principal
        local screenGui = LocalPlayer.PlayerGui:FindFirstChild("Main") or LocalPlayer.PlayerGui:FindFirstChild("ScreenGui")
        if screenGui then
            local rollBtn = screenGui:FindFirstChild("RollButton") or screenGui:FindFirstChild("GachaButton")
            if rollBtn and rollBtn:IsA("TextButton") then
                rollBtn:FireServer() or rollBtn:Click()
                return true
            end
        end
        -- Fallback: tenta via RemoteEvent (mais confiável)
        local remote = game:GetService("ReplicatedStorage"):FindFirstChild("RollFruit") or game:GetService("ReplicatedStorage"):FindFirstChild("RequestRoll")
        if remote then
            remote:FireServer()
            return true
        end
        return false
    end
end

-- // Função para comprar fruta na loja (Belí)
local function buyFruitFromShop()
    -- Simula clique para comprar fruta da loja (Belí)
    local remote = game:GetService("ReplicatedStorage"):FindFirstChild("BuyFruit") or game:GetService("ReplicatedStorage"):FindFirstChild("PurchaseFruit")
    if remote then
        remote:FireServer()
        return true
    end
    -- Fallback para botão na UI
    local playerGui = LocalPlayer.PlayerGui
    if playerGui then
        local shopFrame = playerGui:FindFirstChild("Shop") or playerGui:FindFirstChild("Store")
        if shopFrame then
            local buyBtn = shopFrame:FindFirstChild("BuyButton") or shopFrame:FindFirstChild("Purchase")
            if buyBtn and buyBtn:IsA("TextButton") then
                buyBtn:Click()
                return true
            end
        end
    end
    return false
end

-- // Loop principal das ações automáticas
task.spawn(function()
    while task.wait(1) do -- Executa a cada 1 segundo
        pcall(function()
            if _G.AutoRoll then
                local success = rollFruit()
                if success then
                    notify("Auto Roll", "Roletando fruta...", 0.5)
                end
            end
            
            if _G.AutoBuy then
                local success = buyFruitFromShop()
                if success then
                    notify("Auto Buy", "Comprando fruta na loja...", 0.5)
                end
            end
        end)
    end
end)

-- // ESP de Frutas (Destaca frutas no chão)
if _G.ESP_Fruits == nil then _G.ESP_Fruits = false end
task.spawn(function()
    local fruitEspFolder = Instance.new("Folder")
    fruitEspFolder.Name = "RoRo_FruitESP"
    fruitEspFolder.Parent = gui
    
    while task.wait(0.2) do
        pcall(function()
            -- Limpa ESPs antigos
            for _, v in pairs(fruitEspFolder:GetChildren()) do
                if v:IsA("BillboardGui") then
                    v:Destroy()
                end
            end
            
            if _G.ESP_Fruits then
                -- Procura por frutas no chão (Model com nome de fruta)
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj.Name:find("Fruit") or obj.Name:find("Fruits") or obj:FindFirstChild("Handle") then
                        local isFruit = false
                        local fruitName = obj.Name
                        -- Lista de nomes de frutas conhecidas
                        local fruitKeywords = {"Gomu", "Magma", "Ice", "Flame", "Quake", "Dark", "Light", "String", "Dough", "Dragon", "Leopard", "Phoenix", "Buddha"}
                        for _, kw in pairs(fruitKeywords) do
                            if fruitName:find(kw) then
                                isFruit = true
                                break
                            end
                        end
                        
                        if isFruit or fruitName:lower():find("fruit") then
                            local esp = Instance.new("BillboardGui")
                            esp.Name = "FruitESP"
                            esp.Adornee = obj
                            esp.Size = UDim2.new(0, 120, 0, 40)
                            esp.StudsOffset = Vector3.new(0, 2, 0)
                            esp.Parent = fruitEspFolder
                            
                            local label = Instance.new("TextLabel")
                            label.Size = UDim2.new(1, 0, 1, 0)
                            label.BackgroundTransparency = 0.3
                            label.BackgroundColor3 = Color3.fromRGB(255, 170, 85)
                            label.TextColor3 = Color3.fromRGB(255, 255, 255)
                            label.Text = "🍎 " .. fruitName .. " 🍎"
                            label.TextSize = 14
                            label.Font = Enum.Font.GothamBold
                            label.TextStrokeTransparency = 0.5
                            label.Parent = esp
                        end
                    end
                end
            end
        end)
    end
end)

-- // ESP de Players (Destaca outros jogadores)
if _G.ESP_Players == nil then _G.ESP_Players = false end
task.spawn(function()
    local playerEspFolder = Instance.new("Folder")
    playerEspFolder.Name = "RoRo_PlayerESP"
    playerEspFolder.Parent = gui
    
    while task.wait(0.2) do
        pcall(function()
            -- Limpa ESPs antigos
            for _, v in pairs(playerEspFolder:GetChildren()) do
                if v:IsA("BillboardGui") then
                    v:Destroy()
                end
            end
            
            if _G.ESP_Players then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local esp = Instance.new("BillboardGui")
                        esp.Name = "PlayerESP"
                        esp.Adornee = player.Character.HumanoidRootPart
                        esp.Size = UDim2.new(0, 150, 0, 50)
                        esp.StudsOffset = Vector3.new(0, 2.5, 0)
                        esp.Parent = playerEspFolder
                        
                        local frame = Instance.new("Frame")
                        frame.Size = UDim2.new(1, 0, 1, 0)
                        frame.BackgroundTransparency = 0.5
                        frame.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
                        frame.BorderSizePixel = 0
                        frame.Parent = esp
                        
                        local nameLabel = Instance.new("TextLabel")
                        nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
                        nameLabel.Position = UDim2.new(0, 0, 0.25, 0)
                        nameLabel.BackgroundTransparency = 1
                        nameLabel.Text = player.Name
                        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                        nameLabel.TextSize = 12
                        nameLabel.Font = Enum.Font.GothamBold
                        nameLabel.TextStrokeTransparency = 0.3
                        nameLabel.Parent = frame
                        
                        local healthLabel = Instance.new("TextLabel")
                        healthLabel.Size = UDim2.new(1, 0, 0.5, 0)
                        healthLabel.Position = UDim2.new(0, 0, 0.6, 0)
                        healthLabel.BackgroundTransparency = 1
                        healthLabel.Text = "❤️ " .. math.floor((player.Character.Humanoid.Health / player.Character.Humanoid.MaxHealth) * 100) .. "%"
                        healthLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
                        healthLabel.TextSize = 11
                        healthLabel.Font = Enum.Font.Gotham
                        healthLabel.Parent = frame
                    end
                end
            end
        end)
    end
end)

-- // Mensagem final no console
print([[

   ██████╗  ██████╗ ██████╗  ██████╗ 
   ██╔══██╗██╔═══██╗██╔══██╗██╔═══██╗
   ██████╔╝██║   ██║██████╔╝██║   ██║
   ██╔══██╗██║   ██║██╔══██╗██║   ██║
   ██║  ██║╚██████╔╝██║  ██║╚██████╔╝
   ╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═╝ ╚═════╝ 
                                      
         RoRo Hub - King Legacy
         Carregado com sucesso!
         
         [Insert] para abrir/fechar
]])
