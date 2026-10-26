--[[
    SCRIPT: Reroll Cego (Auto-Kick Seguro)
    FUNÇÃO: Fecha o jogo no exato milissegundo do clique para evitar o salvamento.
]]

local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

local isSaveActive = false
local originalNamecall = nil

-- ==========================================
-- 1. INTERFACE
-- ==========================================
local oldGui = CoreGui:FindFirstChild("RerollManagerUI")
if oldGui then oldGui:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RerollManagerUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
ScreenGui.DisplayOrder = 999

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 280, 0, 220)
MainFrame.Position = UDim2.new(0.5, -140, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
TitleLabel.BorderSizePixel = 0
TitleLabel.Text = "REROLL CEGO (AUTO-KICK)"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleLabel

local ToggleSaveBtn = Instance.new("TextButton")
ToggleSaveBtn.Size = UDim2.new(1, -20, 0, 45)
ToggleSaveBtn.Position = UDim2.new(0, 10, 0, 45)
ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
ToggleSaveBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleSaveBtn.Text = "🔴 Modo Seguro: DESLIGADO"
ToggleSaveBtn.Font = Enum.Font.GothamBold
ToggleSaveBtn.TextSize = 14
ToggleSaveBtn.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleSaveBtn

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 100)
StatusLabel.Position = UDim2.new(0, 10, 0, 100)
StatusLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.Text = "Status: Aguardando..."
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 12
StatusLabel.TextWrapped = true
StatusLabel.Parent = MainFrame

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 6)
StatusCorner.Parent = StatusLabel

-- ==========================================
-- 2. FUNÇÃO DE CRASH INSTANTÂNEO
-- ==========================================
local function crashGame()
    -- Tenta Kick normal primeiro
    pcall(function() LocalPlayer:Kick("Auto-Kick Ativado") end)
    
    -- Destruir o CoreGui principal para forçar erro no celular
    pcall(function() 
        local core = game:GetService("CoreGui")
        for _, v in pairs(core:GetChildren()) do
            if v.Name ~= "RerollManagerUI" then v:Destroy() end
        end
    end)
    
    -- Fecha o jogo na força (Shutdown)
    task.wait(0.05)
    pcall(function() game:Shutdown() end)
end

-- ==========================================
-- 3. LÓGICA DO MODO SEGURO (AUTO-KICK)
-- ==========================================
ToggleSaveBtn.MouseButton1Click:Connect(function()
    if not isSaveActive then
        isSaveActive = true
        ToggleSaveBtn.Text = "🟢 Modo Seguro: LIGADO"
        ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
        StatusLabel.Text = "Status: LIGADO. Vá no NPC e clique em Reroll no jogo. O script vai fechar o jogo sozinho."
        StatusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        
        -- O HOOK MÁGICO
        if not originalNamecall then
            originalNamecall = hookmetamethod(game, "__namecall", function(self, ...)
                local method = getnamecallmethod()
                
                if isSaveActive and method == "FireServer" then
                    local remoteName = string.lower(tostring(self.Name))
                    
                    -- Detecta a tentativa de reroll
                    if remoteName:find("roll") or remoteName:find("trait") or remoteName:find("gem") or remoteName:find("crystal") then
                        
                        -- ATUALIZA A TELA ANTES DE MORRER
                        StatusLabel.Text = "Status: REROLL DETECTADO! FECHANDO JOGO..."
                        StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
                        
                        -- BLOQUEIA O ENVIO PRO SERVIDOR
                        print("[AUTO-KICK] Tentativa de envio bloqueada. Fechando...")
                        
                        -- EXECUTA O CRASH EM OUTRA THREAD (para não travar o jogo antes de fechar)
                        task.spawn(function()
                            crashGame()
                        end)
                        
                        return nil -- Impede o pacote de ir pro servidor
                    end
                end
                
                return originalNamecall(self, ...)
            end)
        end
    else
        isSaveActive = false
        ToggleSaveBtn.Text = "🔴 Modo Seguro: DESLIGADO"
        ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
        StatusLabel.Text = "Status: Desligado. O jogo vai salvar normal."
        StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    end
end)
