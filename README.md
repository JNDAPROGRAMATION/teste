-- Serverscript (LocalScript no StarterPlayerScripts)
-- ESP/Wallhack para Roblox

-- Configurações
local ESP_SETTINGS = {
    ENABLED = true,
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
local controlGui = nil
local isGUIVisible = false

-- Função para criar a interface básica
local function createBasicGUI()
    -- Criar ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESPBasicGUI"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Frame principal do botão
    local mainButtonFrame = Instance.new("Frame")
    mainButtonFrame.Name = "MainButtonFrame"
    mainButtonFrame.Size = UDim2.new(0, 80, 0, 40)
    mainButtonFrame.Position = UDim2.new(1, -90, 0, 20)
    mainButtonFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainButtonFrame.BackgroundTransparency = 0.3
    mainButtonFrame.BorderSizePixel = 0
    mainButtonFrame.Parent = screenGui
    
    -- Arredondar cantos
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 8)
    uiCorner.Parent = mainButtonFrame
    
    -- Sombra suave
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(100, 100, 100)
    uiStroke.Thickness = 1
    uiStroke.Parent = mainButtonFrame
    
    -- Botão ESP
    local espButton = Instance.new("TextButton")
    espButton.Name = "ESPButton"
    espButton.Size = UDim2.new(1, 0, 1, 0)
    espButton.BackgroundTransparency = 1
    espButton.Text = "ESP"
    espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
    espButton.TextSize = 16
    espButton.Font = Enum.Font.GothamBold
    espButton.Parent = mainButtonFrame
    
    -- Botão de configurações (pequeno)
    local settingsButton = Instance.new("TextButton")
    settingsButton.Name = "SettingsButton"
    settingsButton.Size = UDim2.new(0, 25, 0, 25)
    settingsButton.Position = UDim2.new(0, -30, 0, 8)
    settingsButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    settingsButton.BackgroundTransparency = 0.3
    settingsButton.Text = "⚙"
    settingsButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    settingsButton.TextSize = 14
    settingsButton.Font = Enum.Font.Gotham
    settingsButton.Visible = false
    settingsButton.Parent = mainButtonFrame
    
    local settingsCorner = Instance.new("UICorner")
    settingsCorner.CornerRadius = UDim.new(0, 6)
    settingsCorner.Parent = settingsButton
    
    -- Painel de configurações
    local settingsPanel = Instance.new("Frame")
    settingsPanel.Name = "SettingsPanel"
    settingsPanel.Size = UDim2.new(0, 180, 0, 250)
    settingsPanel.Position = UDim2.new(0, -190, 0, 0)
    settingsPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    settingsPanel.BackgroundTransparency = 0.2
    settingsPanel.BorderSizePixel = 0
    settingsPanel.Visible = false
    settingsPanel.Parent = mainButtonFrame
    
    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 8)
    panelCorner.Parent = settingsPanel
    
    local panelStroke = Instance.new("UIStroke")
    panelStroke.Color = Color3.fromRGB(80, 80, 80)
    panelStroke.Thickness = 1
    panelStroke.Parent = settingsPanel
    
    -- Título do painel
    local panelTitle = Instance.new("TextLabel")
    panelTitle.Name = "Title"
    panelTitle.Size = UDim2.new(1, 0, 0, 30)
    panelTitle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    panelTitle.BackgroundTransparency = 0.5
    panelTitle.Text = "Configurações ESP"
    panelTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    panelTitle.TextSize = 14
    panelTitle.Font = Enum.Font.GothamBold
    panelTitle.Parent = settingsPanel
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 8)
    titleCorner.Parent = panelTitle
    
    -- Container para os toggles
    local toggleContainer = Instance.new("ScrollingFrame")
    toggleContainer.Name = "ToggleContainer"
    toggleContainer.Size = UDim2.new(1, -10, 1, -40)
    toggleContainer.Position = UDim2.new(0, 5, 0, 35)
    toggleContainer.BackgroundTransparency = 1
    toggleContainer.BorderSizePixel = 0
    toggleContainer.ScrollBarThickness = 4
    toggleContainer.CanvasSize = UDim2.new(0, 0, 0, 200)
    toggleContainer.Parent = settingsPanel
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 5)
    layout.Parent = toggleContainer
    
    -- Função para criar um toggle
    local function createToggleOption(text, settingName)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Size = UDim2.new(1, 0, 0, 25)
        toggleFrame.BackgroundTransparency = 1
        toggleFrame.Parent = toggleContainer
        
        local label = Instance.new("TextLabel")
        label.Text = text
        label.Size = UDim2.new(0.7, 0, 1, 0)
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.TextSize = 12
        label.Font = Enum.Font.Gotham
        label.Parent = toggleFrame
        
        local toggle = Instance.new("TextButton")
        toggle.Name = settingName
        toggle.Size = UDim2.new(0.3, 0, 1, 0)
        toggle.Position = UDim2.new(0.7, 0, 0, 0)
        toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
        toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggle.TextSize = 12
        toggle.Font = Enum.Font.GothamBold
        toggle.Parent = toggleFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggle
        
        toggle.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
            toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
            
            -- Se for o toggle principal, atualizar cor do botão
            if settingName == "ENABLED" then
                espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
            end
        end)
    end
    
    -- Criar toggles
    createToggleOption("ESP", "ENABLED")
    createToggleOption("Wallhack", "WALLHACK")
    createToggleOption("Nomes", "SHOW_NAMES")
    createToggleOption("Distância", "SHOW_DISTANCE")
    createToggleOption("Vida", "SHOW_HEALTH")
    createToggleOption("Caixa", "SHOW_BOX")
    createToggleOption("Linha", "SHOW_TRACER")
    
    -- Atualizar tamanho do canvas
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        toggleContainer.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y)
    end)
    
    -- Conectar eventos dos botões
    espButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        
        -- Atualizar toggle no painel
        local toggleBtn = settingsPanel:FindFirstChild("ToggleContainer"):FindFirstChild("ENABLED")
        if toggleBtn then
            toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
            toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        end
    end)
    
    settingsButton.MouseButton1Click:Connect(function()
        settingsPanel.Visible = not settingsPanel.Visible
        settingsButton.Text = settingsPanel.Visible and "✕" or "⚙"
    end)
    
    -- Mostrar/ocultar botão de configurações ao passar o mouse
    mainButtonFrame.MouseEnter:Connect(function()
        settingsButton.Visible = true
        local tween = TweenService:Create(settingsButton, TweenInfo.new(0.2), {Position = UDim2.new(0, -30, 0, 8)})
        tween:Play()
    end)
    
    mainButtonFrame.MouseLeave:Connect(function()
        if not settingsPanel.Visible then
            settingsButton.Visible = false
        end
    end)
    
    -- Fechar painel ao clicar fora
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local mousePos = UserInputService:GetMouseLocation()
            local panelAbsolutePos = settingsPanel.AbsolutePosition
            local panelAbsoluteSize = settingsPanel.AbsoluteSize
            
            -- Verificar se clique foi fora do painel
            if settingsPanel.Visible then
                if not (mousePos.X >= panelAbsolutePos.X and mousePos.X <= panelAbsolutePos.X + panelAbsoluteSize.X and
                       mousePos.Y >= panelAbsolutePos.Y and mousePos.Y <= panelAbsolutePos.Y + panelAbsoluteSize.Y) then
                    
                    -- Verificar se não foi no botão de configurações
                    local buttonAbsolutePos = settingsButton.AbsolutePosition
                    local buttonAbsoluteSize = settingsButton.AbsoluteSize
                    
                    if not (mousePos.X >= buttonAbsolutePos.X and mousePos.X <= buttonAbsolutePos.X + buttonAbsoluteSize.X and
                           mousePos.Y >= buttonAbsolutePos.Y and mousePos.Y <= buttonAbsolutePos.Y + buttonAbsoluteSize.Y) then
                        
                        settingsPanel.Visible = false
                        settingsButton.Text = "⚙"
                    end
                end
            end
        end
    end)
    
    return screenGui
end

-- Função para criar um objeto ESP (mantida igual)
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
    
    -- Criar caixa (Outline)
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
    
    -- Criar labels para informações
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
    
    -- Criar tracer
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
    
    -- Conectar eventos de mudança de personagem
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
    if not ESP_SETTINGS.ENABLED then return end
    
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
    
    -- Calcular distância
    local distance = (humanoidRootPart.Position - localRootPart.Position).Magnitude
    
    if distance > ESP_SETTINGS.MAX_DISTANCE then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    -- Verificar se está visível (para wallhack)
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
    
    -- Obter cor
    local color = getPlayerColor(player)
    
    -- Atualizar caixa
    if espObject.Box then
        espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX and isVisible
        espObject.Box.Color3 = color
        espObject.Box.Size = Vector3.new(4, humanoid.HipHeight * 2, 2)
    end
    
    -- Atualizar labels na tela
    local head = character:FindFirstChild("Head")
    if head then
        local screenPosition, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position + Vector3.new(0, 2, 0))
        
        if onScreen then
            -- Nome
            if espObject.NameLabel then
                espObject.NameLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_NAMES and isVisible
                espObject.NameLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                         0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y)
                espObject.NameLabel.TextColor3 = color
            end
            
            -- Distância
            if espObject.DistanceLabel then
                espObject.DistanceLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_DISTANCE and isVisible
                espObject.DistanceLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                             0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 20)
                espObject.DistanceLabel.Text = math.floor(distance) .. "m"
                espObject.DistanceLabel.TextColor3 = color
            end
            
            -- Saúde
            if espObject.HealthLabel then
                espObject.HealthLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_HEALTH and isVisible
                espObject.HealthLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                           0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 40)
            end
            
            -- Tracer
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
    if not ESP_SETTINGS.ENABLED then return end
    
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

-- Criar interface
controlGui = createBasicGUI()

-- Tecla para ativar/desativar (Insert)
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Insert then
            ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
            
            -- Atualizar botão na interface
            local espButton = controlGui:FindFirstChild("MainButtonFrame"):FindFirstChild("ESPButton")
            if espButton then
                espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
            end
            
            -- Atualizar toggle no painel
            local settingsPanel = controlGui:FindFirstChild("MainButtonFrame"):FindFirstChild("SettingsPanel")
            if settingsPanel and settingsPanel.Visible then
                local toggleBtn = settingsPanel:FindFirstChild("ToggleContainer"):FindFirstChild("ENABLED")
                if toggleBtn then
                    toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
                    toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
                end
            end
            
            print("ESP " .. (ESP_SETTINGS.ENABLED and "ativado" or "desativado"))
        end
    end
end))

-- Limpeza quando o script for destruído
local function cleanup()
    for _, connection in ipairs(connections) do
        connection:Disconnect()
    end
    
    for _, espObject in pairs(espCache) do
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
    end
    
    if controlGui then controlGui:Destroy() end
    espFolder:Destroy()
end

-- Conectar evento de saída
game:GetService("Players").LocalPlayer.AncestryChanged:Connect(function()
    cleanup()
end)

print("ESP/Wallhack script carregado!")
print("Interface criada - Botão ESP no canto superior direito")
print("Tecla Insert - Ativar/Desativar ESP")
print("Passe o mouse sobre o botão para ver opções de configuração")
