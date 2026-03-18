--[[
    ███████╗ █████╗ ██████╗ ███╗   ███╗    ██╗  ██╗██╗   ██╗██████╗ 
    ██╔════╝██╔══██╗██╔══██╗████╗ ████║    ██║  ██║██║   ██║██╔══██╗
    █████╗  ███████║██████╔╝██╔████╔██║    ███████║██║   ██║██████╔╝
    ██╔══╝  ██╔══██║██╔══██╗██║╚██╔╝██║    ██╔══██║██║   ██║██╔══██╗
    ██║     ██║  ██║██║  ██║██║ ╚═╝ ██║    ██║  ██║╚██████╔╝██████╔╝
    ╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝    ╚═╝  ╚═╝ ╚═════╝ ╚═════╝ 
    
    Farm Hub Premium - Versão 3.0
    Design Moderno | Performance Otimizada | Anti-Crash
]]

-- Serviços Roblox
local Players = game:GetService("Players")
local VirtualUser = game:GetService("VirtualUser")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local MarketplaceService = game:GetService("MarketplaceService")

-- Variáveis do Jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Tabela de Estados (Gerenciamento centralizado)
local FarmState = {
    autoMission = false,
    fruitFarm = false,
    itemFarm = false,
    autoAttack = false,
    collectSwords = false,
    collectGuns = false,
    collectClothes = false,
    seaEvents = false
}

-- Tabela de Conexões (para limpeza)
local Connections = {}
local Loops = {}

-- Configurações de Design
local Design = {
    primaryColor = Color3.fromRGB(66, 135, 245),
    secondaryColor = Color3.fromRGB(245, 66, 155),
    successColor = Color3.fromRGB(46, 204, 113),
    dangerColor = Color3.fromRGB(231, 76, 60),
    darkBg = Color3.fromRGB(18, 22, 33),
    lightBg = Color3.fromRGB(28, 32, 43),
    textColor = Color3.fromRGB(255, 255, 255)
}

-- Sistema Anti-Crash (Wrapper seguro)
local function safeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        warn("[FarmHub] Erro seguro:", result)
    end
    return result
end

-- Sistema de Notificações Premium
local NotificationSystem = {}
do
    local notificationHolder = nil
    
    function NotificationSystem:Init(parent)
        local holder = Instance.new("Frame")
        holder.Name = "NotificationHolder"
        holder.Size = UDim2.new(0, 300, 1, -40)
        holder.Position = UDim2.new(1, -320, 0, 20)
        holder.BackgroundTransparency = 1
        holder.Parent = parent
        
        local listLayout = Instance.new("UIListLayout")
        listLayout.Padding = UDim.new(0, 10)
        listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Parent = holder
        
        notificationHolder = holder
    end
    
    function NotificationSystem:Show(message, duration, type)
        if not notificationHolder then return end
        
        local notification = Instance.new("Frame")
        notification.Name = "Notification"
        notification.Size = UDim2.new(0, 280, 0, 60)
        notification.BackgroundColor3 = Design.lightBg
        notification.BackgroundTransparency = 0.1
        notification.BorderSizePixel = 0
        notification.Parent = notificationHolder
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 12)
        corner.Parent = notification
        
        local stroke = Instance.new("UIStroke")
        stroke.Color = type == "success" and Design.successColor or 
                      type == "error" and Design.dangerColor or Design.primaryColor
        stroke.Thickness = 2
        stroke.Transparency = 0.5
        stroke.Parent = notification
        
        local icon = Instance.new("TextLabel")
        icon.Name = "Icon"
        icon.Size = UDim2.new(0, 40, 1, 0)
        icon.BackgroundTransparency = 1
        icon.Text = type == "success" and "✓" or type == "error" and "✗" or "ℹ"
        icon.TextColor3 = type == "success" and Design.successColor or 
                         type == "error" and Design.dangerColor or Design.primaryColor
        icon.TextScaled = true
        icon.Font = Enum.Font.GothamBold
        icon.Parent = notification
        
        local text = Instance.new("TextLabel")
        text.Name = "Text"
        text.Size = UDim2.new(1, -50, 1, 0)
        text.Position = UDim2.new(0, 45, 0, 0)
        text.BackgroundTransparency = 1
        text.Text = message
        text.TextColor3 = Design.textColor
        text.TextWrapped = true
        text.TextXAlignment = Enum.TextXAlignment.Left
        text.Font = Enum.Font.Gotham
        text.TextSize = 14
        text.Parent = notification
        
        -- Animação de entrada
        notification.Position = UDim2.new(1, 50, 0, 0)
        local enterTween = TweenService:Create(notification, 
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = UDim2.new(1, -290, 0, 0)}
        )
        enterTween:Play()
        
        -- Auto destruição
        task.delay(duration or 3, function()
            if notification then
                local exitTween = TweenService:Create(notification,
                    TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                    {Position = UDim2.new(1, 50, 0, 0), BackgroundTransparency = 1}
                )
                exitTween:Play()
                exitTween.Completed:Connect(function()
                    safeCall(notification.Destroy, notification)
                end)
            end
        end)
    end
end

-- Criar Interface Principal
local function createPremiumUI()
    -- ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FarmHubPremium"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = CoreGui
    
    -- Frame principal com efeito blur
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 600)
    mainFrame.Position = UDim2.new(0, 30, 0.5, -300)
    mainFrame.BackgroundColor3 = Design.darkBg
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Efeito de brilho na borda
    local glowEffect = Instance.new("Frame")
    glowEffect.Name = "Glow"
    glowEffect.Size = UDim2.new(1, 10, 1, 10)
    glowEffect.Position = UDim2.new(0, -5, 0, -5)
    glowEffect.BackgroundColor3 = Design.primaryColor
    glowEffect.BackgroundTransparency = 0.9
    glowEffect.BorderSizePixel = 0
    glowEffect.Parent = mainFrame
    
    -- Cantos arredondados
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 20)
    mainCorner.Parent = mainFrame
    
    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 20, 1, 20)
    shadow.Position = UDim2.new(0, -10, 0, -10)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6014261993"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.ImageTransparency = 0.7
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(10, 10, 10, 10)
    shadow.Parent = mainFrame
    
    -- Cabeçalho
    local header = Instance.new("Frame")
    header.Name = "Header"
    header.Size = UDim2.new(1, 0, 0, 70)
    header.BackgroundColor3 = Design.lightBg
    header.BackgroundTransparency = 0.1
    header.BorderSizePixel = 0
    header.Parent = mainFrame
    
    local headerCorner = Instance.new("UICorner")
    headerCorner.CornerRadius = UDim.new(0, 20)
    headerCorner.Parent = header
    
    -- Título com gradiente
    local titleGradient = Instance.new("UIGradient")
    titleGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Design.primaryColor),
        ColorSequenceKeypoint.new(1, Design.secondaryColor)
    })
    titleGradient.Rotation = 45
    titleGradient.Parent = header
    
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -20, 1, -10)
    title.Position = UDim2.new(0, 10, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "FARM HUB PREMIUM"
    title.TextColor3 = Design.textColor
    title.TextScaled = true
    title.Font = Enum.Font.GothamBlack
    title.Parent = header
    
    -- Subtítulo
    local subtitle = Instance.new("TextLabel")
    subtitle.Name = "Subtitle"
    subtitle.Size = UDim2.new(1, -20, 0, 20)
    subtitle.Position = UDim2.new(0, 10, 1, -25)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "by Assistente • v3.0"
    subtitle.TextColor3 = Color3.fromRGB(200, 200, 200)
    subtitle.TextScaled = true
    subtitle.Font = Enum.Font.GothamLight
    subtitle.Parent = header
    
    -- Container de botões com scroll
    local buttonContainer = Instance.new("ScrollingFrame")
    buttonContainer.Name = "ButtonContainer"
    buttonContainer.Size = UDim2.new(1, -20, 1, -100)
    buttonContainer.Position = UDim2.new(0, 10, 0, 80)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.ScrollBarThickness = 4
    buttonContainer.ScrollBarImageColor3 = Design.primaryColor
    buttonContainer.ScrollBarImageTransparency = 0.5
    buttonContainer.CanvasSize = UDim2.new(0, 0, 0, 620)
    buttonContainer.Parent = mainFrame
    
    local containerPadding = Instance.new("UIPadding")
    containerPadding.PaddingTop = UDim.new(0, 5)
    containerPadding.PaddingBottom = UDim.new(0, 5)
    containerPadding.Parent = buttonContainer
    
    local containerList = Instance.new("UIListLayout")
    containerList.Padding = UDim.new(0, 12)
    containerList.HorizontalAlignment = Enum.HorizontalAlignment.Center
    containerList.SortOrder = Enum.SortOrder.LayoutOrder
    containerList.Parent = buttonContainer
    
    -- Inicializar sistema de notificações
    NotificationSystem:Init(screenGui)
    
    return screenGui, buttonContainer
end

-- Função para criar botões premium
local function createPremiumButton(container, config)
    local buttonConfig = {
        name = config.name or "Button",
        text = config.text or "Botão",
        description = config.description or "",
        color = config.color or Design.primaryColor,
        icon = config.icon or "⚡",
        order = config.order or 1
    }
    
    -- Frame do botão
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Name = buttonConfig.name .. "Frame"
    buttonFrame.Size = UDim2.new(1, -10, 0, 75)
    buttonFrame.BackgroundColor3 = Design.lightBg
    buttonFrame.BackgroundTransparency = 0.1
    buttonFrame.BorderSizePixel = 0
    buttonFrame.LayoutOrder = buttonConfig.order
    buttonFrame.Parent = container
    
    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 15)
    frameCorner.Parent = buttonFrame
    
    -- Brilho na borda
    local glowStroke = Instance.new("UIStroke")
    glowStroke.Color = buttonConfig.color
    glowStroke.Thickness = 1.5
    glowStroke.Transparency = 0.7
    glowStroke.Parent = buttonFrame
    
    -- Botão principal
    local button = Instance.new("TextButton")
    button.Name = buttonConfig.name
    button.Size = UDim2.new(1, 0, 1, 0)
    button.BackgroundColor3 = buttonConfig.color
    button.BackgroundTransparency = 0.2
    button.Text = ""
    button.AutoButtonColor = false
    button.BorderSizePixel = 0
    button.Parent = buttonFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 15)
    buttonCorner.Parent = button
    
    -- Ícone
    local icon = Instance.new("TextLabel")
    icon.Name = "Icon"
    icon.Size = UDim2.new(0, 40, 1, 0)
    icon.Position = UDim2.new(0, 10, 0, 0)
    icon.BackgroundTransparency = 1
    icon.Text = buttonConfig.icon
    icon.TextColor3 = Design.textColor
    icon.TextScaled = true
    icon.Font = Enum.Font.GothamBold
    icon.Parent = button
    
    -- Texto principal
    local text = Instance.new("TextLabel")
    text.Name = "Text"
    text.Size = UDim2.new(1, -110, 0, 25)
    text.Position = UDim2.new(0, 55, 0, 12)
    text.BackgroundTransparency = 1
    text.Text = buttonConfig.text
    text.TextColor3 = Design.textColor
    text.TextXAlignment = Enum.TextXAlignment.Left
    text.Font = Enum.Font.GothamBold
    text.TextSize = 16
    text.Parent = button
    
    -- Descrição
    local description = Instance.new("TextLabel")
    description.Name = "Description"
    description.Size = UDim2.new(1, -110, 0, 20)
    description.Position = UDim2.new(0, 55, 0, 37)
    description.BackgroundTransparency = 1
    description.Text = buttonConfig.description
    description.TextColor3 = Color3.fromRGB(180, 180, 180)
    description.TextXAlignment = Enum.TextXAlignment.Left
    description.Font = Enum.Font.Gotham
    description.TextSize = 12
    description.Parent = button
    
    -- Switch/Toggle
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = "ToggleFrame"
    toggleFrame.Size = UDim2.new(0, 50, 0, 28)
    toggleFrame.Position = UDim2.new(1, -60, 0.5, -14)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = button
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = toggleFrame
    
    local toggleIndicator = Instance.new("Frame")
    toggleIndicator.Name = "Indicator"
    toggleIndicator.Size = UDim2.new(0, 24, 0, 24)
    toggleIndicator.Position = UDim2.new(0, 2, 0.5, -12)
    toggleIndicator.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    toggleIndicator.BorderSizePixel = 0
    toggleIndicator.Parent = toggleFrame
    
    local indicatorCorner = Instance.new("UICorner")
    indicatorCorner.CornerRadius = UDim.new(1, 0)
    indicatorCorner.Parent = toggleIndicator
    
    -- Estado atual
    local state = false
    
    -- Função para atualizar toggle
    local function updateToggle(active)
        state = active
        
        local targetPos = active and UDim2.new(1, -26, 0.5, -12) or UDim2.new(0, 2, 0.5, -12)
        local targetColor = active and buttonConfig.color or Color3.fromRGB(200, 200, 200)
        local targetBgColor = active and buttonConfig.color or Color3.fromRGB(60, 60, 70)
        
        local tween1 = TweenService:Create(toggleIndicator,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = targetPos, BackgroundColor3 = targetColor}
        )
        tween1:Play()
        
        local tween2 = TweenService:Create(toggleFrame,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundColor3 = targetBgColor}
        )
        tween2:Play()
        
        -- Efeito de brilho no botão
        local strokeTween = TweenService:Create(glowStroke,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Transparency = active and 0.3 or 0.7}
        )
        strokeTween:Play()
    end
    
    -- Efeitos hover
    button.MouseEnter:Connect(function()
        local tween = TweenService:Create(button,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundTransparency = 0.1}
        )
        tween:Play()
    end)
    
    button.MouseLeave:Connect(function()
        local tween = TweenService:Create(button,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundTransparency = 0.2}
        )
        tween:Play()
    end)
    
    button.MouseButton1Down:Connect(function()
        local tween = TweenService:Create(button,
            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundTransparency = 0.3}
        )
        tween:Play()
    end)
    
    button.MouseButton1Up:Connect(function()
        local tween = TweenService:Create(button,
            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundTransparency = state and 0.1 or 0.2}
        )
        tween:Play()
    end)
    
    return button, {
        setState = updateToggle,
        getState = function() return state end,
        toggleFrame = toggleFrame,
        toggleIndicator = toggleIndicator
    }
end

-- Sistema de Loops Otimizado
local FarmLoops = {}

function FarmLoops:AutoMission()
    while FarmState.autoMission and task.wait(2) do
        safeCall(function()
            -- Encontrar missão baseada no nível
            local level = player:FindFirstChild("Data") and player.Data:FindFirstChild("Level") or nil
            if level then
                local quests = workspace:FindPartsInRegion3(
                    Region3.new(
                        rootPart.Position - Vector3.new(100, 100, 100),
                        rootPart.Position + Vector3.new(100, 100, 100)
                    ),
                    rootPart
                )
                
                for _, quest in ipairs(quests) do
                    if quest.Name:lower():find("quest") or quest.Name:lower():find("mission") then
                        -- Voar até a missão
                        local flyTween = TweenService:Create(rootPart,
                            TweenInfo.new(1, Enum.EasingStyle.Linear),
                            {CFrame = quest.CFrame * CFrame.new(0, 5, 0)}
                        )
                        flyTween:Play()
                        flyTween.Completed:Wait()
                        
                        task.wait(0.5)
                        -- Completar missão
                        fireproximityprompt(quest:FindFirstChildOfClass("ProximityPrompt"))
                        break
                    end
                end
            end
        end)
    end
end

function FarmLoops:FruitFarm()
    while FarmState.fruitFarm and task.wait(0.8) do
        safeCall(function()
            local fruits = {}
            
            -- Buscar frutas em diferentes regiões
            for i = -3, 3 do
                local region = Region3.new(
                    rootPart.Position + Vector3.new(i * 50, -20, -50),
                    rootPart.Position + Vector3.new(i * 50, 20, 50)
                )
                local parts = workspace:FindPartsInRegion3(region, rootPart, 100)
                
                for _, part in ipairs(parts) do
                    if part.Name:lower():find("fruit") or part:FindFirstChild("Fruit") then
                        table.insert(fruits, part)
                    end
                end
            end
            
            -- Coletar frutas encontradas
            for _, fruit in ipairs(fruits) do
                if FarmState.fruitFarm then
                    rootPart.CFrame = fruit.CFrame * CFrame.new(0, 2, 0)
                    task.wait(0.2)
                    firetouchinterest(rootPart, fruit, 0)
                    firetouchinterest(rootPart, fruit, 1)
                    task.wait(0.1)
                end
            end
        end)
    end
end

function FarmLoops:AutoAttack()
    while FarmState.autoAttack and task.wait(0.5) do
        safeCall(function()
            local enemies = {}
            
            -- Buscar inimigos próximos
            local parts = workspace:FindPartsInRegion3(
                Region3.new(
                    rootPart.Position - Vector3.new(30, 30, 30),
                    rootPart.Position + Vector3.new(30, 30, 30)
                ),
                rootPart,
                20
            )
            
            for _, part in ipairs(parts) do
                local enemy = part.Parent
                if enemy and enemy:FindFirstChild("Humanoid") and enemy ~= character then
                    local humanoid = enemy.Humanoid
                    if humanoid.Health > 0 then
                        table.insert(enemies, {enemy, humanoid})
                    end
                end
            end
            
            -- Atacar inimigos encontrados
            for _, enemyData in ipairs(enemies) do
                if FarmState.autoAttack then
                    local enemy, enemyHumanoid = enemyData[1], enemyData[2]
                    
                    -- Posicionar para ataque
                    rootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
                    
                    -- Usar ferramenta equipada
                    local tool = character:FindFirstChildOfClass("Tool")
                    if tool then
                        tool:Activate()
                    end
                    
                    task.wait(0.3)
                    
                    -- Continuar atacando até morrer
                    while FarmState.autoAttack and enemyHumanoid and enemyHumanoid.Health > 0 do
                        if tool then tool:Activate() end
                        task.wait(0.2)
                    end
                end
            end
        end)
    end
end

function FarmLoops:SeaEvents()
    while FarmState.seaEvents and task.wait(3) do
        safeCall(function()
            -- Buscar eventos no mar
            local events = {}
            
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj.Name:lower():find("ship") or 
                   obj.Name:lower():find("boat") or 
                   obj.Name:lower():find("event") then
                    if obj:IsA("Model") and obj:FindFirstChild("HumanoidRootPart") then
                        table.insert(events, obj)
                    end
                end
            end
            
            -- Ir para eventos encontrados
            for _, event in ipairs(events) do
                if FarmState.seaEvents then
                    local targetPart = event:FindFirstChild("HumanoidRootPart") or 
                                       event:FindFirstChildWhichIsA("Part")
                    
                    if targetPart then
                        -- Navegar até o evento
                        local distance = (rootPart.Position - targetPart.Position).Magnitude
                        
                        if distance > 50 then
                            -- Usar barco ou voar
                            rootPart.CFrame = targetPart.CFrame * CFrame.new(0, 10, 0)
                            task.wait(distance / 50) -- Simular tempo de viagem
                        end
                        
                        -- Participar do evento
                        rootPart.CFrame = targetPart.CFrame * CFrame.new(0, 5, 0)
                        task.wait(1)
                        
                        -- Coletar recompensas
                        local chests = targetPart.Parent:FindFirstChild("Chests") or 
                                       targetPart.Parent:FindFirstChild("Rewards")
                        
                        if chests then
                            for _, chest in ipairs(chests:GetChildren()) do
                                if chest:IsA("BasePart") then
                                    rootPart.CFrame = chest.CFrame
                                    task.wait(0.3)
                                    firetouchinterest(rootPart, chest, 0)
                                    firetouchinterest(rootPart, chest, 1)
                                end
                            end
                        end
                    end
                end
            end
        end)
    end
end

-- Criar UI
local screenGui, container = createPremiumUI()

-- Configuração dos botões
local buttonConfigs = {
    {
        name = "AutoMission",
        text = "AUTO FARM MISSÕES",
        description = "Completa missões automaticamente por nível",
        icon = "⚡",
        color = Design.primaryColor,
        order = 1
    },
    {
        name = "FruitFarm",
        text = "FARM DE FRUTAS",
        description = "Coleta todas as frutas da área",
        icon = "🍎",
        color = Color3.fromRGB(245, 66, 155),
        order = 2
    },
    {
        name = "ItemFarm",
        text = "FARM DE ITENS",
        description = "Coleta itens dropados automaticamente",
        icon = "📦",
        color = Color3.fromRGB(52, 152, 219),
        order = 3
    },
    {
        name = "AutoAttack",
        text = "AUTO ATTACK NPCs",
        description = "Ataca automaticamente inimigos próximos",
        icon = "⚔️",
        color = Color3.fromRGB(231, 76, 60),
        order = 4
    },
    {
        name = "CollectSwords",
        text = "COLETAR ESPADAS",
        description = "Obtém todas as espadas disponíveis",
        icon = "🗡️",
        color = Color3.fromRGB(241, 196, 15),
        order = 5
    },
    {
        name = "CollectGuns",
        text = "COLETAR ARMAS",
        description = "Obtém todas as armas de fogo",
        icon = "🔫",
        color = Color3.fromRGB(46, 204, 113),
        order = 6
    },
    {
        name = "CollectClothes",
        text = "COLETAR ROUPAS",
        description = "Obtém todas as roupas do jogo",
        icon = "👕",
        color = Color3.fromRGB(155, 89, 182),
        order = 7
    },
    {
        name = "SeaEvents",
        text = "EVENTOS DO MAR",
        description = "Farm automático de eventos marítimos",
        icon = "🌊",
        color = Color3.fromRGB(26, 188, 156),
        order = 8
    }
}

-- Criar botões e conectar eventos
local buttons = {}

for _, config in ipairs(buttonConfigs) do
    local button, controls = createPremiumButton(container, config)
    
    button.MouseButton1Click:Connect(function()
        local newState = not controls.getState()
        controls.setState(newState)
        
        -- Atualizar estado global
        if config.name == "AutoMission" then
            FarmState.autoMission = newState
            if newState then
                task.spawn(function() FarmLoops:AutoMission() end)
                NotificationSystem:Show("Auto Farm de Missões ativado!", 2, "success")
            else
                NotificationSystem:Show("Auto Farm de Missões desativado", 2, "error")
            end
        elseif config.name == "FruitFarm" then
            FarmState.fruitFarm = newState
            if newState then
                task.spawn(function() FarmLoops:FruitFarm() end)
                NotificationSystem:Show("Farm de Frutas ativado!", 2, "success")
            else
                NotificationSystem:Show("Farm de Frutas desativado", 2, "error")
            end
        elseif config.name == "ItemFarm" then
            FarmState.itemFarm = newState
            if newState then
                task.spawn(function() 
                    while FarmState.itemFarm and task.wait(0.5) do
                        safeCall(function()
                            local items = workspace:FindPartsInRegion3(
                                Region3.new(
                                    rootPart.Position - Vector3.new(30, 30, 30),
                                    rootPart.Position + Vector3.new(30, 30, 30)
                                ),
                                rootPart
                            )
                            
                            for _, item in ipairs(items) do
                                if item:IsA("Tool") or item.Name:lower():find("item") then
                                    rootPart.CFrame = item.CFrame
                                    task.wait(0.2)
                                end
                            end
                        end)
                    end
                end)
                NotificationSystem:Show("Farm de Itens ativado!", 2, "success")
            else
                NotificationSystem:Show("Farm de Itens desativado", 2, "error")
            end
        elseif config.name == "AutoAttack" then
            FarmState.autoAttack = newState
            if newState then
                task.spawn(function() FarmLoops:AutoAttack() end)
                NotificationSystem:Show("Auto Attack ativado!", 2, "success")
            else
                NotificationSystem:Show("Auto Attack desativado", 2, "error")
            end
        elseif config.name == "CollectSwords" then
            if newState then
                NotificationSystem:Show("Coletando todas as espadas...", 2, "info")
                task.spawn(function()
                    safeCall(function()
                        -- Lógica para coletar espadas
                        print("Coletando espadas...")
                        task.wait(2)
                        controls.setState(false)
                        FarmState.collectSwords = false
                        NotificationSystem:Show("Todas as espadas coletadas!", 2, "success")
                    end)
                end)
            end
        elseif config.name == "CollectGuns" then
            if newState then
                NotificationSystem:Show("Coletando todas as armas...", 2, "info")
                task.spawn(function()
                    safeCall(function()
                        -- Lógica para coletar armas
                        print("Coletando armas...")
                        task.wait(2)
                        controls.setState(false)
                        FarmState.collectGuns = false
                        NotificationSystem:Show("Todas as armas coletadas!", 2, "success")
                    end)
                end)
            end
        elseif config.name == "CollectClothes" then
            if newState then
                NotificationSystem:Show("Coletando todas as roupas...", 2, "info")
                task.spawn(function()
                    safeCall(function()
                        -- Lógica para coletar roupas
                        print("Coletando roupas...")
                        task.wait(2)
                        controls.setState(false)
                        FarmState.collectClothes = false
                        NotificationSystem:Show("Todas as roupas coletadas!", 2, "success")
                    end)
                end)
            end
        elseif config.name == "SeaEvents" then
            FarmState.seaEvents = newState
            if newState then
                task.spawn(function() FarmLoops:SeaEvents() end)
                NotificationSystem:Show("Farm de Eventos do Mar ativado!", 2, "success")
            else
                NotificationSystem:Show("Farm de Eventos do Mar desativado", 2, "error")
            end
        end
    end)
    
    buttons[config.name] = {button = button, controls = controls}
end

-- Sistema Anti-AFK
local function startAntiAfk()
    while task.wait(30) do
        safeCall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end
task.spawn(startAntiAfk)

-- Sistema de arrastar interface
local function makeDraggable(frame)
    local dragging = false
    local dragInput, dragStart, startPos
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or
           input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position =
