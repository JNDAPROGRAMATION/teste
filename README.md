-- Serverscript (LocalScript no StarterPlayerScripts)
-- ESP/Wallhack para Roblox com Painel de Controle

-- Configurações
local ESP_SETTINGS = {
    ENABLED = false, -- Começa desativado
    WALLHACK = true, -- Ver através das paredes
    SHOW_NAMES = true,
    SHOW_DISTANCE = true,
    SHOW_HEALTH = true,
    SHOW_BOX = true,
    SHOW_TRACER = true, -- Linha do pé do jogador até o alvo
    
    MAX_DISTANCE = 1000, -- Distância máxima
    UPDATE_INTERVAL = 0.1, -- Atualização a cada 0.1 segundos
    
    -- Cores
    TEAM_COLOR = true, -- Usar cor do time
    FRIENDLY_COLOR = Color3.fromRGB(0, 255, 0), -- Verde para aliados
    ENEMY_COLOR = Color3.fromRGB(255, 0, 0), -- Vermelho para inimigos
    NEUTRAL_COLOR = Color3.fromRGB(255, 255, 0), -- Amarelo para neutros
    
    -- Tamanhos
    BOX_THICKNESS = 1,
    TEXT_SIZE = 18,
    TRACER_THICKNESS = 1,
    
    -- Posições
    TEXT_OFFSET = Vector2.new(0, -40), -- Offset do texto acima da cabeça
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}
local controlPanel = nil
local isPanelVisible = true -- Começa visível

-- Função para criar o painel de controle
local function createControlPanel()
    -- Criar ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESPControlPanel"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Frame principal do painel
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 250, 0, 400)
    mainFrame.Position = UDim2.new(0, 20, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BackgroundTransparency = 0.2
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Arredondar cantos
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 12)
    uiCorner.Parent = mainFrame
    
    -- Sombra
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(60, 60, 60)
    uiStroke.Thickness = 2
    uiStroke.Parent = mainFrame
    
    -- Cabeçalho do painel
    local headerFrame = Instance.new("Frame")
    headerFrame.Name = "Header"
    headerFrame.Size = UDim2.new(1, 0, 0, 50)
    headerFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    headerFrame.BorderSizePixel = 0
    headerFrame.Parent = mainFrame
    
    local headerCorner = Instance.new("UICorner")
    headerCorner.CornerRadius = UDim.new(0, 12)
    headerCorner.Parent = headerFrame
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = "Painel ESP v1.0"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 20
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.BackgroundTransparency = 1
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = headerFrame
    
    -- Botão de minimizar/maximizar
    local minimizeButton = Instance.new("TextButton")
    minimizeButton.Name = "MinimizeButton"
    minimizeButton.Size = UDim2.new(0, 30, 0, 30)
    minimizeButton.Position = UDim2.new(1, -35, 0.5, -15)
    minimizeButton.Text = "-"
    minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeButton.TextSize = 24
    minimizeButton.Font = Enum.Font.GothamBold
    minimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    minimizeButton.BackgroundTransparency = 0.5
    minimizeButton.Parent = headerFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 8)
    buttonCorner.Parent = minimizeButton
    
    -- Área de conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, 0, 1, -50)
    contentFrame.Position = UDim2.new(0, 0, 0, 50)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Container para botões principais
    local mainButtonsContainer = Instance.new("Frame")
    mainButtonsContainer.Name = "MainButtons"
    mainButtonsContainer.Size = UDim2.new(1, 0, 0, 120)
    mainButtonsContainer.BackgroundTransparency = 1
    mainButtonsContainer.Parent = contentFrame
    
    -- Botão principal ESP (grande)
    local espMainButton = Instance.new("TextButton")
    espMainButton.Name = "ESPMainButton"
    espMainButton.Size = UDim2.new(1, -20, 0, 60)
    espMainButton.Position = UDim2.new(0, 10, 0, 10)
    espMainButton.Text = "ESP: DESATIVADO"
    espMainButton.TextColor3 = Color3.fromRGB(255, 50, 50)
    espMainButton.TextSize = 18
    espMainButton.Font = Enum.Font.GothamBold
    espMainButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    espMainButton.Parent = mainButtonsContainer
    
    local espButtonCorner = Instance.new("UICorner")
    espButtonCorner.CornerRadius = UDim.new(0, 10)
    espButtonCorner.Parent = espMainButton
    
    local espButtonStroke = Instance.new("UIStroke")
    espButtonStroke.Color = Color3.fromRGB(100, 100, 100)
    espButtonStroke.Thickness = 2
    espButtonStroke.Parent = espMainButton
    
    -- Status do ESP
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Size = UDim2.new(1, -20, 0, 30)
    statusLabel.Position = UDim2.new(0, 10, 0, 80)
    statusLabel.Text = "Status: Aguardando ativação..."
    statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusLabel.TextSize = 14
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = mainButtonsContainer
    
    -- Separador
    local separator = Instance.new("Frame")
    separator.Name = "Separator"
    separator.Size = UDim2.new(1, -20, 0, 2)
    separator.Position = UDim2.new(0, 10, 0, 140)
    separator.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    separator.BorderSizePixel = 0
    separator.Parent = contentFrame
    
    -- Container de funcionalidades
    local featuresContainer = Instance.new("ScrollingFrame")
    featuresContainer.Name = "FeaturesContainer"
    featuresContainer.Size = UDim2.new(1, 0, 1, -150)
    featuresContainer.Position = UDim2.new(0, 0, 0, 150)
    featuresContainer.BackgroundTransparency = 1
    featuresContainer.BorderSizePixel = 0
    featuresContainer.ScrollBarThickness = 4
    featuresContainer.CanvasSize = UDim2.new(0, 0, 0, 300)
    featuresContainer.Parent = contentFrame
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 10)
    layout.Parent = featuresContainer
    
    -- Função para criar botão de funcionalidade
    local function createFeatureButton(text, settingName, description)
        local featureFrame = Instance.new("Frame")
        featureFrame.Size = UDim2.new(1, -20, 0, 60)
        featureFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        featureFrame.BackgroundTransparency = 0.3
        featureFrame.Parent = featuresContainer
        
        local featureCorner = Instance.new("UICorner")
        featureCorner.CornerRadius = UDim.new(0, 8)
        featureCorner.Parent = featureFrame
        
        -- Texto principal
        local featureText = Instance.new("TextLabel")
        featureText.Name = "FeatureText"
        featureText.Size = UDim2.new(0.6, 0, 0.5, 0)
        featureText.Position = UDim2.new(0, 10, 0, 5)
        featureText.Text = text
        featureText.TextColor3 = Color3.fromRGB(255, 255, 255)
        featureText.TextSize = 16
        featureText.Font = Enum.Font.GothamBold
        featureText.BackgroundTransparency = 1
        featureText.TextXAlignment = Enum.TextXAlignment.Left
        featureText.Parent = featureFrame
        
        -- Descrição
        local descLabel = Instance.new("TextLabel")
        descLabel.Name = "Description"
        descLabel.Size = UDim2.new(0.6, 0, 0.5, 0)
        descLabel.Position = UDim2.new(0, 10, 0.5, 0)
        descLabel.Text = description
        descLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
        descLabel.TextSize = 12
        descLabel.Font = Enum.Font.Gotham
        descLabel.BackgroundTransparency = 1
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.Parent = featureFrame
        
        -- Botão de toggle
        local toggleButton = Instance.new("TextButton")
        toggleButton.Name = "Toggle"
        toggleButton.Size = UDim2.new(0.3, 0, 0.7, 0)
        toggleButton.Position = UDim2.new(0.65, 0, 0.15, 0)
        toggleButton.Text = ESP_SETTINGS[settingName] and "ATIVADO" or "DESATIVADO"
        toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        toggleButton.TextSize = 14
        toggleButton.Font = Enum.Font.GothamBold
        toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 80, 0) or Color3.fromRGB(80, 0, 0)
        toggleButton.Parent = featureFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggleButton
        
        toggleButton.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggleButton.Text = ESP_SETTINGS[settingName] and "ATIVADO" or "DESATIVADO"
            toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
            toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 80, 0) or Color3.fromRGB(80, 0, 0)
            
            -- Atualizar status se for o ESP principal
            if settingName == "ENABLED" then
                espMainButton.Text = ESP_SETTINGS.ENABLED and "ESP: ATIVADO" or "ESP: DESATIVADO"
                espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                statusLabel.Text = ESP_SETTINGS.ENABLED and "Status: ESP ATIVADO com sucesso!" or "Status: ESP DESATIVADO"
            end
        end)
        
        return featureFrame
    end
    
    -- Criar botões de funcionalidades
    createFeatureButton("Wallhack", "WALLHACK", "Ver através das paredes")
    createFeatureButton("Mostrar Nomes", "SHOW_NAMES", "Exibir nomes dos jogadores")
    createFeatureButton("Mostrar Distância", "SHOW_DISTANCE", "Distância até os jogadores")
    createFeatureButton("Mostrar Vida", "SHOW_HEALTH", "Barra de vida dos jogadores")
    createFeatureButton("Caixa de Destaque", "SHOW_BOX", "Caixa ao redor dos jogadores")
    createFeatureButton("Linha Guia", "SHOW_TRACER", "Linha da base até o jogador")
    
    -- Atualizar tamanho do canvas
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        featuresContainer.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20)
    end)
    
    -- Botão de reset
    local resetButton = Instance.new("TextButton")
    resetButton.Name = "ResetButton"
    resetButton.Size = UDim2.new(1, -20, 0, 40)
    resetButton.Position = UDim2.new(0, 10, 1, -50)
    resetButton.Text = "Resetar Configurações"
    resetButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    resetButton.TextSize = 14
    resetButton.Font = Enum.Font.GothamBold
    resetButton.BackgroundColor3 = Color3.fromRGB(80, 0, 0)
    resetButton.BackgroundTransparency = 0.3
    resetButton.Parent = contentFrame
    
    local resetCorner = Instance.new("UICorner")
    resetCorner.CornerRadius = UDim.new(0, 8)
    resetCorner.Parent = resetButton
    
    -- Conectar eventos
    -- Botão principal ESP
    espMainButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        espMainButton.Text = ESP_SETTINGS.ENABLED and "ESP: ATIVADO" or "ESP: DESATIVADO"
        espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        statusLabel.Text = ESP_SETTINGS.ENABLED and "Status: ESP ATIVADO com sucesso!" or "Status: ESP DESATIVADO"
        
        -- Atualizar toggle nas funcionalidades
        for _, child in ipairs(featuresContainer:GetChildren()) do
            if child:IsA("Frame") and child.Name ~= "Layout" then
                local toggleBtn = child:FindFirstChild("Toggle")
                if toggleBtn and toggleBtn.Name == "Toggle" and child:FindFirstChild("FeatureText").Text == "ESP" then
                    toggleBtn.Text = ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO"
                    toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                    toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 80, 0) or Color3.fromRGB(80, 0, 0)
                end
            end
        end
    end)
    
    -- Botão de minimizar
    minimizeButton.MouseButton1Click:Connect(function()
        if isPanelVisible then
            -- Minimizar
            local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 50, 0, 50),
                Position = UDim2.new(0, 20, 0.5, -25)
            })
            tween:Play()
            
            contentFrame.Visible = false
            minimizeButton.Text = "+"
            isPanelVisible = false
        else
            -- Maximizar
            local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 250, 0, 400),
                Position = UDim2.new(0, 20, 0.5, -200)
            })
            tween:Play()
            
            contentFrame.Visible = true
            minimizeButton.Text = "-"
            isPanelVisible = true
        end
    end)
    
    -- Botão de reset
    resetButton.MouseButton1Click:Connect(function()
        -- Resetar todas as configurações
        ESP_SETTINGS.ENABLED = false
        ESP_SETTINGS.WALLHACK = true
        ESP_SETTINGS.SHOW_NAMES = true
        ESP_SETTINGS.SHOW_DISTANCE = true
        ESP_SETTINGS.SHOW_HEALTH = true
        ESP_SETTINGS.SHOW_BOX = true
        ESP_SETTINGS.SHOW_TRACER = true
        
        -- Atualizar botão principal
        espMainButton.Text = "ESP: DESATIVADO"
        espMainButton.TextColor3 = Color3.fromRGB(255, 50, 50)
        statusLabel.Text = "Status: Configurações resetadas"
        
        -- Atualizar todos os toggles
        for _, child in ipairs(featuresContainer:GetChildren()) do
            if child:IsA("Frame") then
                local toggleBtn = child:FindFirstChild("Toggle")
                local featureText = child:FindFirstChild("FeatureText")
                
                if toggleBtn and featureText then
                    local settingName = ""
                    if featureText.Text == "Wallhack" then settingName = "WALLHACK"
                    elseif featureText.Text == "Mostrar Nomes" then settingName = "SHOW_NAMES"
                    elseif featureText.Text == "Mostrar Distância" then settingName = "SHOW_DISTANCE"
                    elseif featureText.Text == "Mostrar Vida" then settingName = "SHOW_HEALTH"
                    elseif featureText.Text == "Caixa de Destaque" then settingName = "SHOW_BOX"
                    elseif featureText.Text == "Linha Guia" then settingName = "SHOW_TRACER" end
                    
                    if settingName ~= "" then
                        local isEnabled = ESP_SETTINGS[settingName]
                        toggleBtn.Text = isEnabled and "ATIVADO" or "DESATIVADO"
                        toggleBtn.TextColor3 = isEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                        toggleBtn.BackgroundColor3 = isEnabled and Color3.fromRGB(0, 80, 0) or Color3.fromRGB(80, 0, 0)
                    end
                end
            end
        end
    end)
    
    -- Tornar o painel arrastável
    local dragging = false
    local dragStart
    local startPos
    
    headerFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
        end
    end)
    
    headerFrame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    return screenGui
end

-- Função para criar um objeto ESP (igual ao anterior)
local function createESPObject(player)
    local espObject = {
        Player = player,
        Box = nil,
        NameLabel = nil,
        DistanceLabel = nil,
        HealthLabel = nil,
        Tracer = nil,
        Connections = {}
    }
    
    if ESP_SETTINGS.SHOW_BOX then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_ESPBox"
        box.Adornee = nil
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Size = Vector3.new(4, 6, 2)
        box.Transparency = 0.3
        box.Visible = false
        box.Parent = espFolder
        
        espObject.Box = box
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = player.Name .. "_ESPGui"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = espFolder
    
    if ESP_SETTINGS.SHOW_NAMES then
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "Name"
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        nameLabel.TextStrokeTransparency = 0.5
        nameLabel.TextSize = ESP_SETTINGS.TEXT_SIZE
        nameLabel.Visible = false
        nameLabel.Parent = screenGui
        
        espObject.NameLabel = nameLabel
    end
    
    if ESP_SETTINGS.SHOW_DISTANCE then
        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Name = "Distance"
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.Text = "0m"
        distanceLabel.TextColor3 = Color3.new(1, 1, 1)
        distanceLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        distanceLabel.TextStrokeTransparency = 0.5
        distanceLabel.TextSize = ESP_SETTINGS.TEXT_SIZE - 2
        distanceLabel.Visible = false
        distanceLabel.Parent = screenGui
        
        espObject.DistanceLabel = distanceLabel
    end
    
    if ESP_SETTINGS.SHOW_HEALTH then
        local healthLabel = Instance.new("TextLabel")
        healthLabel.Name = "Health"
        healthLabel.BackgroundTransparency = 1
        healthLabel.Text = "100%"
        healthLabel.TextColor3 = Color3.new(0, 1, 0)
        healthLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        healthLabel.TextStrokeTransparency = 0.5
        healthLabel.TextSize = ESP_SETTINGS.TEXT_SIZE - 2
        healthLabel.Visible = false
        healthLabel.Parent = screenGui
        
        espObject.HealthLabel = healthLabel
    end
    
    if ESP_SETTINGS.SHOW_TRACER then
        local tracer = Instance.new("Frame")
        tracer.Name = "Tracer"
        tracer.BackgroundColor3 = Color3.new(1, 1, 1)
        tracer.BorderSizePixel = 0
        tracer.Size = UDim2.new(0, 2, 0, 100)
        tracer.Rotation = 0
        tracer.Visible = false
        tracer.Parent = screenGui
        
        espObject.Tracer = tracer
    end
    
    local function characterAdded(character)
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart", 5)
        local humanoid = character:WaitForChild("Humanoid", 5)
        
        if espObject.Box then
            espObject.Box.Adornee = humanoidRootPart
        end
        
        if humanoid and espObject.HealthLabel then
            local function updateHealth()
                local healthPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
                espObject.HealthLabel.Text = healthPercent .. "%"
                
                if healthPercent > 70 then
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                elseif healthPercent > 30 then
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
                else
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                end
            end
            
            table.insert(espObject.Connections, humanoid.HealthChanged:Connect(updateHealth))
            updateHealth()
        end
    end
    
    if player.Character then
        characterAdded(player.Character)
    end
    
    table.insert(espObject.Connections, player.CharacterAdded:Connect(characterAdded))
    
    espCache[player] = espObject
    return espObject
end

-- Função para obter cor baseada no time
local function getPlayerColor(player)
    if ESP_SETTINGS.TEAM_COLOR and player.Team then
        return player.Team.TeamColor.Color
    end
    
    if localPlayer.Team and player.Team then
        if localPlayer.Team == player.Team then
            return ESP_SETTINGS.FRIENDLY_COLOR
        else
            return ESP_SETTINGS.ENEMY_COLOR
        end
    end
    
    return ESP_SETTINGS.NEUTRAL_COLOR
end

-- Função para atualizar o ESP de um jogador
local function updateESP(player, espObject)
    if not ESP_SETTINGS.ENABLED then 
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return 
    end
    
    if not player or not player.Character then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    
    if not humanoidRootPart or not humanoid then return end
    
    local localCharacter = localPlayer.Character
    local localRootPart = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
    
    if not localRootPart then return end
    
    local distance = (humanoidRootPart.Position - localRootPart.Position).Magnitude
    
    if distance > ESP_SETTINGS.MAX_DISTANCE then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    local isVisible = true
    if not ESP_SETTINGS.WALLHACK then
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {localCharacter, character}
        raycastParams.IgnoreWater = true
        
        local raycastResult = workspace:Raycast(
            localRootPart.Position,
            (humanoidRootPart.Position - localRootPart.Position).Unit * distance,
            raycastParams
        )
        
        isVisible = not raycastResult
    end
    
    local color = getPlayerColor(player)
    
    if espObject.Box then
        espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX and isVisible
        espObject.Box.Color3 = color
        espObject.Box.Size = Vector3.new(4, humanoid.HipHeight * 2, 2)
    end
    
    local head = character:FindFirstChild("Head")
    if head then
        local screenPosition, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position + Vector3.new(0, 2, 0))
        
        if onScreen then
            if espObject.NameLabel then
                espObject.NameLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_NAMES and isVisible
                espObject.NameLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                         0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y)
                espObject.NameLabel.TextColor3 = color
            end
            
            if espObject.DistanceLabel then
                espObject.DistanceLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_DISTANCE and isVisible
                espObject.DistanceLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                             0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 20)
                espObject.DistanceLabel.Text = math.floor(distance) .. "m"
                espObject.DistanceLabel.TextColor3 = color
            end
            
            if espObject.HealthLabel then
                espObject.HealthLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_HEALTH and isVisible
                espObject.HealthLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                           0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 40)
            end
            
            if espObject.Tracer then
                local rootScreenPosition = workspace.CurrentCamera:WorldToViewportPoint(humanoidRootPart.Position)
                local bottomScreenPosition = Vector2.new(rootScreenPosition.X, rootScreenPosition.Y)
                
                espObject.Tracer.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_TRACER and isVisible
                espObject.Tracer.BackgroundColor3 = color
                espObject.Tracer.Position = UDim2.new(0, bottomScreenPosition.X, 0, bottomScreenPosition.Y)
                espObject.Tracer.Size = UDim2.new(0, ESP_SETTINGS.TRACER_THICKNESS, 0, workspace.CurrentCamera.ViewportSize.Y - bottomScreenPosition.Y)
            end
        else
            if espObject.NameLabel then espObject.NameLabel.Visible = false end
            if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
            if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
            if espObject.Tracer then espObject.Tracer.Visible = false end
        end
    end
end

-- Função principal de atualização
local function updateAllESP()
    for player, espObject in pairs(espCache) do
        if player ~= localPlayer then
            updateESP(player, espObject)
        end
    end
end

-- Inicializar ESP para jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer then
        createESPObject(player)
    end
end

-- Conectar para novos jogadores
table.insert(connections, Players.PlayerAdded:Connect(function(player)
    if player ~= localPlayer then
        createESPObject(player)
    end
end))

-- Remover ESP quando jogador sair
table.insert(connections, Players.PlayerRemoving:Connect(function(player)
    local espObject = espCache[player]
    if espObject then
        for _, connection in ipairs(espObject.Connections) do
            connection:Disconnect()
        end
        
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
        
        espCache[player] = nil
    end
end))

-- Loop de atualização
table.insert(connections, RunService.RenderStepped:Connect(function()
    updateAllESP()
end))

-- Criar painel de controle
controlPanel = createControlPanel()

-- Tecla para abrir/fechar o painel (F5)
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.F5 then
            local mainFrame = controlPanel:FindFirstChild("MainFrame")
            if mainFrame then
                if isPanelVisible then
                    -- Minimizar
                    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3), {
                        Size = UDim2.new(0, 50, 0, 50),
                        Position = UDim2.new(0, 20, 0.5, -25)
                    })
                    tween:Play()
                    
                    mainFrame:FindFirstChild("Content").Visible = false
                    mainFrame:FindFirstChild("Header"):FindFirstChild("MinimizeButton").Text = "+"
                    isPanelVisible = false
                else
                    -- Maximizar
                    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3), {
                        Size = UDim2.new(0, 250, 0, 400),
                        Position = UDim2.new(0, 20, 0.5, -200)
                    })
                    tween:Play()
                    
                    mainFrame:FindFirstChild("Content").Visible = true
                    mainFrame:FindFirstChild("Header"):FindFirstChild("MinimizeButton").Text = "-"
                    isPanelVisible = true
                end
            end
        elseif input.KeyCode == Enum.KeyCode.Insert then
            -- Ativar
