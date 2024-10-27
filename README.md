-- Interface GUI para o Script de Farm até Nível Máximo

-- Cria a interface
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local ToggleButton = Instance.new("TextButton")
local StatusLabel = Instance.new("TextLabel")

-- Configurações de GUI
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Frame.Position = UDim2.new(0.1, 0, 0.8, 0)
Frame.Size = UDim2.new(0, 200, 0, 100)

-- Botão de alternar farm
ToggleButton.Parent = Frame
ToggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
ToggleButton.Size = UDim2.new(0, 180, 0, 40)
ToggleButton.Position = UDim2.new(0, 10, 0, 10)
ToggleButton.Text = "Iniciar Farm"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 20

-- Label de status
StatusLabel.Parent = Frame
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0, 10, 0, 60)
StatusLabel.Size = UDim2.new(0, 180, 0, 30)
StatusLabel.Text = "Status: Inativo"
StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusLabel.TextSize = 18

-- Variável para controle do farm
local farming = false

-- Dados de quests para cada nível (exemplo)
local questData = {
    {level = 0, questName = "Bandit Quest", enemyName = "Bandit", npcName = "BanditQuestGiverNPC"},
    {level = 10, questName = "Monkey Quest", enemyName = "Monkey", npcName = "MonkeyQuestGiverNPC"},
    {level = 30, questName = "Gorilla Quest", enemyName = "Gorilla", npcName = "GorillaQuestGiverNPC"},
    {level = 50, questName = "Pirate Quest", enemyName = "Pirate", npcName = "PirateQuestGiverNPC"},
    -- Adicione mais quests conforme necessário
}

local attackRange = 50 -- Distância de ataque
local refreshDelay = 2 -- Tempo entre verificações

-- Função para mover-se até o NPC
function moveTo(position)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    character.Humanoid:MoveTo(position)
end

-- Função para pegar missão do NPC
function startQuest(npcName)
    local npc = game.Workspace:FindFirstChild(npcName)
    if npc then
        moveTo(npc.Position)
        wait(1)
        fireproximityprompt(npc.ProximityPrompt)
    end
end

-- Função para atacar inimigos
function attackEnemy(enemy)
    if enemy and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        
        character.HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5) -- Fica a uma distância segura
        wait(0.5)
        
        -- Realiza o ataque
        character:FindFirstChildOfClass("Tool"):Activate()
    end
end

-- Função principal para farmar level até o máximo
function farmToMaxLevel()
    while farming do
        local player = game.Players.LocalPlayer
        local playerLevel = player.Data.Level.Value
        local quest = nil

        -- Determina a quest com base no nível do jogador
        for i = #questData, 1, -1 do
            if playerLevel >= questData[i].level then
                quest = questData[i]
                break
            end
        end

        -- Checa se a missão está ativa
        local questActive = player.PlayerGui.QuestTracker:FindFirstChild(quest.questName)
        if not questActive then
            startQuest(quest.npcName)
        end

        -- Encontra e ataca inimigos
        for _, enemy in pairs(game.Workspace.Enemies:GetChildren()) do
            if enemy.Name == quest.enemyName and (enemy.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).magnitude <= attackRange then
                attackEnemy(enemy)
                wait(1) -- Intervalo entre os ataques
            end
        end

        wait(refreshDelay)
    end
end

-- Função para iniciar/parar o farm
ToggleButton.MouseButton1Click:Connect(function()
    farming = not farming
    if farming then
        StatusLabel.Text = "Status: Ativo"
        ToggleButton.Text = "Parar Farm"
        farmToMaxLevel()
    else
        StatusLabel.Text = "Status: Inativo"
        ToggleButton.Text = "Iniciar Farm"
    end
end)

