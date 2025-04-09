-- Configurações
local FLY_KEY = Enum.KeyCode.F -- Tecla para ativar/desativar voo
local MENU_KEY = Enum.KeyCode.RightShift -- Tecla para abrir/fechar menu
local DEFAULT_SPEED = 50 -- Velocidade padrão de voo

-- Variáveis
local player = game:GetService("Players").LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")
local flying = false
local flySpeed = DEFAULT_SPEED
local flyDebounce = false

-- Cria a GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyGUI"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Menu principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0.2, 0, 0.25, 0)
mainFrame.Position = UDim2.new(0.78, 0, 0.3, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0.15, 0)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
title.BorderSizePixel = 0
title.Text = "Configurações de Voo"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.Parent = mainFrame

-- Controle de velocidade
local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.8, 0, 0.15, 0)
speedLabel.Position = UDim2.new(0.1, 0, 0.2, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade: " .. flySpeed
speedLabel.TextColor3 = Color3.new(1, 1, 1)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 12
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = mainFrame

local speedSlider = Instance.new("TextBox")
speedSlider.Name = "SpeedSlider"
speedSlider.Size = UDim2.new(0.8, 0, 0.1, 0)
speedSlider.Position = UDim2.new(0.1, 0, 0.35, 0)
speedSlider.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
speedSlider.BorderSizePixel = 0
speedSlider.Text = tostring(flySpeed)
speedSlider.TextColor3 = Color3.new(1, 1, 1)
speedSlider.Font = Enum.Font.Gotham
speedSlider.TextSize = 12
speedSlider.Parent = mainFrame

-- Botão de aplicar
local applyButton = Instance.new("TextButton")
applyButton.Name = "ApplyButton"
applyButton.Size = UDim2.new(0.6, 0, 0.15, 0)
applyButton.Position = UDim2.new(0.2, 0, 0.5, 0)
applyButton.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
applyButton.BorderSizePixel = 0
applyButton.Text = "Aplicar"
applyButton.TextColor3 = Color3.new(1, 1, 1)
applyButton.Font = Enum.Font.GothamBold
applyButton.TextSize = 12
applyButton.Parent = mainFrame

-- Status do voo
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Size = UDim2.new(1, 0, 0.15, 0)
statusLabel.Position = UDim2.new(0, 0, 0.7, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Voo: DESATIVADO"
statusLabel.TextColor3 = Color3.new(1, 0.5, 0.5)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 14
statusLabel.Parent = mainFrame

-- Notificação
local notifyLabel = Instance.new("TextLabel")
notifyLabel.Name = "NotifyLabel"
notifyLabel.Size = UDim2.new(0.4, 0, 0.05, 0)
notifyLabel.Position = UDim2.new(0.3, 0, 0.9, 0)
notifyLabel.BackgroundTransparency = 1
notifyLabel.Text = "Pressione "..tostring(FLY_KEY):sub(14).." para voar"
notifyLabel.TextColor3 = Color3.new(1, 1, 1)
notifyLabel.Font = Enum.Font.Gotham
notifyLabel.TextSize = 12
notifyLabel.Visible = false
notifyLabel.Parent = screenGui

-- Função para mostrar notificação
local function showNotification(text, duration)
    notifyLabel.Text = text
    notifyLabel.Visible = true
    wait(duration or 2)
    notifyLabel.Visible = false
end

-- Função para atualizar status
local function updateStatus()
    statusLabel.Text = "Voo: " .. (flying and "ATIVADO" or "DESATIVADO")
    statusLabel.TextColor3 = flying and Color3.new(0.5, 1, 0.5) or Color3.new(1, 0.5, 0.5)
end

-- Função principal de voo
local function fly()
    if flyDebounce then return end
    flyDebounce = true
    
    flying = not flying
    updateStatus()
    
    if flying then
        showNotification("Voo ativado! Use WASD + Espaço/Shift")
        
        -- Cria os controles de voo
        local bodyGyro = Instance.new("BodyGyro")
        bodyGyro.P = 10000
        bodyGyro.D = 100
        bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyGyro.CFrame = rootPart.CFrame
        bodyGyro.Parent = rootPart
        
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVelocity.Parent = rootPart
        
        -- Controles
        local cam = workspace.CurrentCamera
        local control = {
            forward = 0,
            backward = 0,
            left = 0,
            right = 0,
            up = 0,
            down = 0
        }
        
        -- Conexões de input
        local forwardConnection
        local backwardConnection
        local leftConnection
        local rightConnection
        local upConnection
        local downConnection
        
        local function disconnectControls()
            if forwardConnection then forwardConnection:Disconnect() end
            if backwardConnection then backwardConnection:Disconnect() end
            if leftConnection then leftConnection:Disconnect() end
            if rightConnection then rightConnection:Disconnect() end
            if upConnection then upConnection:Disconnect() end
            if downConnection then downConnection:Disconnect() end
        end
        
        forwardConnection = game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == Enum.KeyCode.W then
                control.forward = 1
            elseif input.KeyCode == Enum.KeyCode.S then
                control.backward = 1
            elseif input.KeyCode == Enum.KeyCode.A then
                control.left = 1
            elseif input.KeyCode == Enum.KeyCode.D then
                control.right = 1
            elseif input.KeyCode == Enum.KeyCode.Space then
                control.up = 1
            elseif input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift then
                control.down = 1
            end
        end)
        
        backwardConnection = game:GetService("UserInputService").InputEnded:Connect(function(input, gameProcessed)
            if input.KeyCode == Enum.KeyCode.W then
                control.forward = 0
            elseif input.KeyCode == Enum.KeyCode.S then
                control.backward = 0
            elseif input.KeyCode == Enum.KeyCode.A then
                control.left = 0
            elseif input.KeyCode == Enum.KeyCode.D then
                control.right = 0
            elseif input.KeyCode == Enum.KeyCode.Space then
                control.up = 0
            elseif input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift then
                control.down = 0
            end
        end)
        
        -- Loop de voo
        while flying and character and rootPart and bodyGyro and bodyVelocity do
            local direction = (control.forward - control.backward) * cam.CFrame.LookVector + 
                            (control.right - control.left) * cam.CFrame.RightVector + 
                            (control.up - control.down) * Vector3.new(0, 1, 0)
            
            if direction.Magnitude > 0 then
                direction = direction.Unit * flySpeed
            end
            
            bodyVelocity.Velocity = direction
            bodyGyro.CFrame = cam.CFrame
            
            game:GetService("RunService").Heartbeat:Wait()
        end
        
        -- Limpeza
        disconnectControls()
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVelocity then bodyVelocity:Destroy() end
    else
        showNotification("Voo desativado")
    end
    
    flyDebounce = false
end

-- Eventos de teclado
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == FLY_KEY then
        fly()
    elseif input.KeyCode == MENU_KEY then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

-- Configuração de velocidade
speedSlider.FocusLost:Connect(function(enterPressed)
    local newSpeed = tonumber(speedSlider.Text)
    if newSpeed and newSpeed > 0 and newSpeed <= 200 then
        flySpeed = newSpeed
        speedLabel.Text = "Velocidade: " .. flySpeed
        showNotification("Velocidade ajustada para " .. flySpeed)
    else
        speedSlider.Text = tostring(flySpeed)
        showNotification("Velocidade inválida! Use valores entre 1-200", 3)
    end
end)

applyButton.MouseButton1Click:Connect(function()
    local newSpeed = tonumber(speedSlider.Text)
    if newSpeed and newSpeed > 0 and newSpeed <= 200 then
        flySpeed = newSpeed
        speedLabel.Text = "Velocidade: " .. flySpeed
        showNotification("Velocidade ajustada para " .. flySpeed)
    else
        speedSlider.Text = tostring(flySpeed)
        showNotification("Velocidade inválida! Use valores entre 1-200", 3)
    end
end)

-- Atualizações iniciais
updateStatus()
showNotification("Pressione "..tostring(FLY_KEY):sub(14).." para voar", 5)
showNotification("Pressione "..tostring(MENU_KEY):sub(14).." para o menu", 5)

-- Reset ao respawn
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    flying = false
    updateStatus()
end)
