local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local root = char:WaitForChild("HumanoidRootPart")
local playerGui = player:WaitForChild("PlayerGui")

-- Configurações de Caminho (Baseado no seu Rastreador)
-- Importante: O script tenta encontrar o seu Plot automaticamente
local myPlot = nil
for _, plot in pairs(workspace.Game.Plots:GetChildren()) do
    if plot:FindFirstChild("Owner") and plot.Owner.Value == player.Name then
        myPlot = plot
    elseif plot.Name == "Plot6" then -- Fallback para o Plot das fotos
        myPlot = plot
    end
end

-- Variáveis de Estado
local autoFarmActive = false

-- 1. CRIAÇÃO DA INTERFACE (GUI)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CyberAutofarm_V4"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 150)
mainFrame.Position = UDim2.new(0.5, -110, 0.5, -75)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true -- Permite mover a tela
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

-- Título e Créditos
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "CYBERNODRY - TYCOON"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.BackgroundTransparency = 1
title.Parent = mainFrame

-- Botão de Ativar/Desativar
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 180, 0, 40)
toggleBtn.Position = UDim2.new(0, 20, 0, 45)
toggleBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
toggleBtn.Text = "AUTOFARM: OFF"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.Parent = mainFrame
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 6)

-- Botões de Controle (Fechar/Minimizar)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.Text = "X"; closeBtn.TextColor3 = Color3.fromRGB(255, 50, 50)
closeBtn.BackgroundTransparency = 1; closeBtn.Parent = mainFrame

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 25, 0, 25)
minBtn.Position = UDim2.new(1, -55, 0, 5)
minBtn.Text = "-"; minBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minBtn.BackgroundTransparency = 1; minBtn.Parent = mainFrame

-- 2. LÓGICA DO AUTOFARM
local function startAutofarm()
    task.spawn(function()
        while autoFarmActive do
            if myPlot then
                -- A. Coletar e Depositar Bostas
                local poopsFolder = myPlot:FindFirstChild("Poops")
                local depositPart = myPlot.Buttons:FindFirstChild("Deposit") and myPlot.Buttons.Deposit:FindFirstChild("Main")
                
                if poopsFolder then
                    for _, poop in pairs(poopsFolder:GetDescendants()) do
                        if poop:IsA("BasePart") and poop.Name == "Main" then
                            -- Teleporta a bosta para o jogador (coleta)
                            poop.CFrame = root.CFrame
                        end
                    end
                end
                
                -- B. Ir até o Deposit (opcional, se não for automático por proximidade)
                if depositPart then
                    firetouchinterest(root, depositPart, 0)
                    firetouchinterest(root, depositPart, 1)
                end

                -- C. Auto Buy (Melhorias)
                -- Tenta comprar o primeiro botão disponível que você tiver dinheiro
                local buttonsFolder = myPlot:FindFirstChild("Buttons")
                if buttonsFolder then
                    for _, btn in pairs(buttonsFolder:GetChildren()) do
                        local mainPart = btn:FindFirstChild("Main")
                        if mainPart and btn.Name ~= "Deposit" then
                            -- Simula o toque no botão de compra
                            firetouchinterest(root, mainPart, 0)
                            firetouchinterest(root, mainPart, 1)
                        end
                    end
                end
            end
            task.wait(0.5) -- Delay para não crashar e simular cliques
        end
    end)
end

-- 3. INTERAÇÕES DA GUI
toggleBtn.MouseButton1Click:Connect(function()
    autoFarmActive = not autoFarmActive
    if autoFarmActive then
        toggleBtn.Text = "AUTOFARM: ON"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
        startAutofarm()
    else
        toggleBtn.Text = "AUTOFARM: OFF"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
    end
end)

closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

local minimized = false
minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        mainFrame:TweenSize(UDim2.new(0, 220, 0, 35), "Out", "Quad", 0.3, true)
        toggleBtn.Visible = false
    else
        mainFrame:TweenSize(UDim2.new(0, 220, 0, 150), "Out", "Quad", 0.3, true)
        toggleBtn.Visible = true
    end
end)
