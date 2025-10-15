-- UGG Scripts - MultiJump + Save/Return Location (LocalScript)
-- Colar em StarterPlayerScripts. Use em servidores privados/Testes.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- espera o personagem carregar
local function getChar()
    local char = player.Character or player.CharacterAdded:Wait()
    return char
end

local function waitForParts(char)
    local hrp = char:WaitForChild("HumanoidRootPart", 5) or char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:WaitForChild("Humanoid", 5) or char:FindFirstChildWhichIsA("Humanoid")
    return hrp, humanoid
end

-- ===== Config / Estado =====
local multiJumpEnabled = false
local savedPosition = nil -- Vector3 (pos) or nil
local savedOrientation = nil -- optional CFrame orientation

-- ===== UI =====
local playerGui = player:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UGG_MultiJump_SaveLoc"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 220, 0, 220)
panel.Position = UDim2.new(0.02, 0, 0.08, 0)
panel.BackgroundColor3 = Color3.fromRGB(28,28,28)
panel.BorderSizePixel = 0
panel.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -10, 0, 32)
title.Position = UDim2.new(0, 5, 0, 6)
title.BackgroundTransparency = 1
title.Text = "UGG Scripts — MultiJump + SaveLoc"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 16
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextWrapped = true
title.Parent = panel

local function makeButton(text, y)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.9, 0, 0, 36)
    b.Position = UDim2.new(0.05, 0, 0, y)
    b.BackgroundColor3 = Color3.fromRGB(60,60,60)
    b.TextColor3 = Color3.fromRGB(255,255,255)
    b.Font = Enum.Font.SourceSans
    b.TextSize = 14
    b.Text = text
    b.Parent = panel
    return b
end

local toggleJumpBtn = makeButton("MultiJump: Desligado", 48)
local saveBtn = makeButton("Salvar Posição", 96)
local gotoBtn = makeButton("Voltar para a Posição Salva", 144)
local clearBtn = makeButton("Apagar Posição Salva", 184)

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -10, 0, 20)
statusLabel.Position = UDim2.new(0, 5, 0, 136)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 12
statusLabel.TextColor3 = Color3.fromRGB(200,200,200)
statusLabel.Text = "Posição salva: (nenhuma)"
statusLabel.TextWrapped = true
statusLabel.Parent = panel

-- ajuste positions for buttons to avoid overlap with statusLabel
saveBtn.Position = UDim2.new(0.05, 0, 0, 72)
gotoBtn.Position = UDim2.new(0.05, 0, 0, 108)
clearBtn.Position = UDim2.new(0.05, 0, 0, 144)

-- ===== MultiJump logic =====
-- usa JumpRequest que funciona em mobile e PC
local function onJumpRequest()
    if not multiJumpEnabled then return end
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    -- força o Humanoid para o estado de Jumping
    pcall(function()
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end)
end

-- conecta apenas uma vez (não duplicar)
local jumpConn = UserInputService.JumpRequest:Connect(onJumpRequest)

-- ===== Save / GoTo location =====
local function formatPos(pos)
    return string.format("X:%.1f Y:%.1f Z:%.1f", pos.X, pos.Y, pos.Z)
end

local function saveCurrentPosition()
    local char = getChar()
    local hrp, humanoid = waitForParts(char)
    if not hrp then
        statusLabel.Text = "Falha ao salvar: HRP não encontrado."
        return
    end
    savedPosition = hrp.Position
    savedOrientation = hrp.CFrame - hrp.CFrame.Position -- orientation (rotation)
    statusLabel.Text = "Posição salva: "..formatPos(savedPosition)
end

local function gotoSavedPosition()
    if not savedPosition then
        statusLabel.Text = "Nenhuma posição salva."
        return
    end
    local char = getChar()
    local hrp, humanoid = waitForParts(char)
    if not hrp then
        statusLabel.Text = "Falha ao teleportar: HRP não encontrado."
        return
    end

    -- tenta teletransportar de forma segura
    local ok, err = pcall(function()
        -- usa CFrame com orientação salva se existir
        if savedOrientation then
            hrp.CFrame = CFrame.new(savedPosition) * savedOrientation
        else
            hrp.CFrame = CFrame.new(savedPosition)
        end
    end)
    if not ok then
        statusLabel.Text = "Erro ao teleportar: "..tostring(err)
    else
        statusLabel.Text = "Teleportado para: "..formatPos(savedPosition)
    end
end

local function clearSaved()
    savedPosition = nil
    savedOrientation = nil
    statusLabel.Text = "Posição salva: (nenhuma)"
end

-- ===== Button connections =====
toggleJumpBtn.MouseButton1Click:Connect(function()
    multiJumpEnabled = not multiJumpEnabled
    toggleJumpBtn.Text = "MultiJump: "..(multiJumpEnabled and "Ligado" or "Desligado")
end)

saveBtn.MouseButton1Click:Connect(function()
    saveCurrentPosition()
end)

gotoBtn.MouseButton1Click:Connect(function()
    gotoSavedPosition()
end)

clearBtn.MouseButton1Click:Connect(function()
    clearSaved()
end)

-- Limpeza caso o jogador respawn (mantém posição salva mesmo após respawn)
player.CharacterAdded:Connect(function(char)
    -- opcional: atualizar status se houver posição salva
    if savedPosition then
        statusLabel.Text = "Posição salva: "..formatPos(savedPosition)
    else
        statusLabel.Text = "Posição salva: (nenhuma)"
    end
end)

-- Mensagem inicial
print("UGG Scripts - MultiJump + SaveLoc carregado!")
