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
    SHOW_SKELETON = false, -- Esqueleto (bones)
    
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

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}

-- Função para criar um objeto ESP
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
        box.Adornee = nil -- Será definido depois
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Size = Vector3.new(4, 6, 2) -- Tamanho aproximado de um personagem
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
    
    -- Criar tracer (linha até o jogador)
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
        
        -- Atualizar saúde
        if humanoid and espObject.HealthLabel then
            local function updateHealth()
                local healthPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
                espObject.HealthLabel.Text = healthPercent .. "%"
                
                -- Mudar cor baseada na saúde
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
    
    -- Verificar se é aliado
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
        -- Raycast para verificar obstáculos
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
        espObject.Box.Visible = ESP_SETTINGS.ENABLED and isVisible
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
            -- Se não estiver na tela, esconder labels
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
        -- Desconectar conexões
        for _, connection in ipairs(espObject.Connections) do
            connection:Disconnect()
        end
        
        -- Remover objetos
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
        
        espCache[player] = nil
    end
end))

-- Loop de atualização
table.insert(connections, RunService.RenderStepped:Connect(function()
    if ESP_SETTINGS.ENABLED then
        updateAllESP()
    end
end))

-- Interface de controle (opcional)
local function createControlGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESPControl"
    screenGui.Parent = playerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 200, 0, 300)
    frame.Position = UDim2.new(0, 10, 0, 10)
    frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    frame.BackgroundTransparency = 0.3
    frame.BorderSizePixel = 0
    frame.Parent = screenGui
    
    local title = Instance.new("TextLabel")
    title.Text = "ESP Controls"
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    title.TextColor3 = Color3.new(1, 1, 1)
    title.Parent = frame
    
    local yOffset = 35
    
    local function createToggle(text, settingName)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Size = UDim2.new(1, -10, 0, 25)
        toggleFrame.Position = UDim2.new(0, 5, 0, yOffset)
        toggleFrame.BackgroundTransparency = 1
        toggleFrame.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Text = text
        label.Size = UDim2.new(0.7, 0, 1, 0)
        label.TextColor3 = Color3.new(1, 1, 1)
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.Parent = toggleFrame
        
        local toggle = Instance.new("TextButton")
        toggle.Size = UDim2.new(0.3, 0, 1, 0)
        toggle.Position = UDim2.new(0.7, 0, 0, 0)
        toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
        toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        toggle.Parent = toggleFrame
        
        toggle.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
            toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        end)
        
        yOffset = yOffset + 30
    end
    
    createToggle("ESP Enabled", "ENABLED")
    createToggle("Wallhack", "WALLHACK")
    createToggle("Show Names", "SHOW_NAMES")
    createToggle("Show Distance", "SHOW_DISTANCE")
    createToggle("Show Health", "SHOW_HEALTH")
    createToggle("Show Box", "SHOW_BOX")
    createToggle("Show Tracer", "SHOW_TRACER")
    
    -- Botão para fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Text = "X"
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -30, 0, 0)
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    closeButton.TextColor3 = Color3.new(1, 1, 1)
    closeButton.Parent = frame
    
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    -- Botão para mostrar/ocultar
    local toggleButton = Instance.new("TextButton")
    toggleButton.Text = "ESP"
    toggleButton.Size = UDim2.new(0, 50, 0, 50)
    toggleButton.Position = UDim2.new(1, -60, 0, 10)
    toggleButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
    toggleButton.TextColor3 = Color3.new(1, 1, 1)
    toggleButton.Parent = playerGui
    
    toggleButton.MouseButton1Click:Connect(function()
        frame.Visible = not frame.Visible
    end)
end

-- Tecla para ativar/desativar (opcional)
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Insert then
            ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
            print("ESP " .. (ESP_SETTINGS.ENABLED and "ativado" or "desativado"))
        elseif input.KeyCode == Enum.KeyCode.Home then
            createControlGUI()
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
    
    espFolder:Destroy()
end

-- Conectar evento de saída
game:GetService("Players").LocalPlayer.AncestryChanged:Connect(function()
    cleanup()
end)

print("ESP/Wallhack script carregado!")
print("Teclas:")
print("Insert - Ativar/Desativar ESP")
print("Home - Abrir menu de controle")
