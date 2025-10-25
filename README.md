--// Variables
local player = game:GetService("Players").LocalPlayer
local runService = game:GetService("RunService")
local players = game:GetService("Players")

-- Config
local enabled = true
local range = 8
local ignoreFriends = true
local targetVictim = nil

-- SafeList (jamais attaqués)
local safeList = {
    ["Mayan_legameur12"] = true,
    ["Mayan_legameur14"] = true,
}

-- Supprimer ancienne GUI
if player.PlayerGui:FindFirstChild("KillAuraGUI") then
    player.PlayerGui.KillAuraGUI:Destroy()
end

--// GUI principale
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KillAuraGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:FindFirstChildOfClass("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 270)
mainFrame.Position = UDim2.new(0.5, -125, 0.2, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BackgroundTransparency = 0.2
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

-- Titre principal
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, -30, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "⚔ Kill Aura"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true

-- Sous-titre (créateur)
local subtitle = Instance.new("TextLabel", mainFrame)
subtitle.Size = UDim2.new(1, -30, 0, 20)
subtitle.Position = UDim2.new(0, 0, 0, 30)
subtitle.BackgroundTransparency = 1
subtitle.Text = "Created by Mayan_legameur12"
subtitle.Font = Enum.Font.Gotham
subtitle.TextColor3 = Color3.fromRGB(180, 180, 180)
subtitle.TextScaled = true

-- Bouton fermer principal
local closeBtn = Instance.new("TextButton", mainFrame)
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -30, 0, 0)
closeBtn.Text = "❌"
closeBtn.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
closeBtn.TextScaled = true
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

local openBtn = Instance.new("TextButton", screenGui)
openBtn.Size = UDim2.new(0, 60, 0, 60)
openBtn.Position = UDim2.new(0, 20, 1, -80)
openBtn.Text = "📂"
openBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
openBtn.TextScaled = true
openBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
openBtn.Font = Enum.Font.GothamBold
openBtn.Visible = false
openBtn.Active = true
openBtn.Draggable = true
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1, 0)

-- Rayon
local rangeBox = Instance.new("TextBox", mainFrame)
rangeBox.Size = UDim2.new(1, -20, 0, 30)
rangeBox.Position = UDim2.new(0, 10, 0, 60)
rangeBox.PlaceholderText = "Rayon (1 à 1e17)"
rangeBox.Text = tostring(range)
rangeBox.TextScaled = true
rangeBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
rangeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
rangeBox.Font = Enum.Font.Gotham
Instance.new("UICorner", rangeBox).CornerRadius = UDim.new(0, 8)

-- ON/OFF
local toggleBtn = Instance.new("TextButton", mainFrame)
toggleBtn.Size = UDim2.new(0, 100, 0, 30)
toggleBtn.Position = UDim2.new(0, 10, 0, 100)
toggleBtn.Text = "ON"
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
toggleBtn.TextScaled = true
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 8)

-- AutoKill Button (à côté du ON/OFF)
local autoKillBtn = Instance.new("TextButton", mainFrame)
autoKillBtn.Size = UDim2.new(0, 100, 0, 30)
autoKillBtn.Position = UDim2.new(0, 130, 0, 100)
autoKillBtn.Text = "AutoKill"
autoKillBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
autoKillBtn.TextScaled = true
autoKillBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
autoKillBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", autoKillBtn).CornerRadius = UDim.new(0, 8)

-- Ignore friends
local friendBtn = Instance.new("TextButton", mainFrame)
friendBtn.Size = UDim2.new(1, -20, 0, 30)
friendBtn.Position = UDim2.new(0, 10, 0, 140)
friendBtn.Text = "👥 Ignore Friends: ON"
friendBtn.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
friendBtn.TextScaled = true
friendBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
friendBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", friendBtn).CornerRadius = UDim.new(0, 8)

-- Choisir victime
local chooseBtn = Instance.new("TextButton", mainFrame)
chooseBtn.Size = UDim2.new(1, -20, 0, 30)
chooseBtn.Position = UDim2.new(0, 10, 0, 180)
chooseBtn.Text = "🎯 Choisir Victime"
chooseBtn.BackgroundColor3 = Color3.fromRGB(120, 85, 0)
chooseBtn.TextScaled = true
chooseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
chooseBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", chooseBtn).CornerRadius = UDim.new(0, 8)

-- Bouton Discord
local discordBtn = Instance.new("TextButton", mainFrame)
discordBtn.Size = UDim2.new(1, -20, 0, 30)
discordBtn.Position = UDim2.new(0, 10, 0, 220)
discordBtn.Text = "💬 Rejoindre le Discord"
discordBtn.BackgroundColor3 = Color3.fromRGB(85, 110, 230)
discordBtn.TextScaled = true
discordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
discordBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 8)

discordBtn.MouseButton1Click:Connect(function()
    setclipboard("https://discord.gg/nehc227wW")
end)

-- Frame Choisir Victime
local victimFrame = Instance.new("Frame", screenGui)
victimFrame.Size = UDim2.new(0, 200, 0, 300)
victimFrame.Position = UDim2.new(0.5, -100, 0.5, -150)
victimFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
victimFrame.Visible = false
victimFrame.Active = true
victimFrame.Draggable = true
Instance.new("UICorner", victimFrame).CornerRadius = UDim.new(0, 12)

-- Bouton fermer victime
local victimClose = Instance.new("TextButton", victimFrame)
victimClose.Size = UDim2.new(0, 30, 0, 30)
victimClose.Position = UDim2.new(1, -30, 0, 0)
victimClose.Text = "❌"
victimClose.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
victimClose.TextScaled = true
victimClose.TextColor3 = Color3.fromRGB(255, 255, 255)
victimClose.Font = Enum.Font.GothamBold
Instance.new("UICorner", victimClose).CornerRadius = UDim.new(0, 6)

victimClose.MouseButton1Click:Connect(function()
    victimFrame.Visible = false
end)

local victimTitle = Instance.new("TextLabel", victimFrame)
victimTitle.Size = UDim2.new(1, -30, 0, 30)
victimTitle.Text = "🎯 Choisir Victime"
victimTitle.BackgroundTransparency = 1
victimTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
victimTitle.Font = Enum.Font.GothamBold
victimTitle.TextScaled = true

local scroll = Instance.new("ScrollingFrame", victimFrame)
scroll.Size = UDim2.new(1, -10, 1, -70)
scroll.Position = UDim2.new(0, 5, 0, 35)
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.ScrollBarThickness = 6
scroll.BackgroundTransparency = 1

local infoText = Instance.new("TextLabel", victimFrame)
infoText.Size = UDim2.new(1, -10, 0, 40)
infoText.Position = UDim2.new(0, 5, 1, -40)
infoText.BackgroundTransparency = 1
infoText.Text = "For it to work press once the username\nPress again to revert to everyone"
infoText.TextColor3 = Color3.fromRGB(255, 255, 255)
infoText.Font = Enum.Font.Gotham
infoText.TextScaled = true

-- Fonction refresh joueurs
local function refreshPlayers()
    scroll:ClearAllChildren()
    local y = 0
    for _, plr in ipairs(players:GetPlayers()) do
        if plr ~= player then
            local btn = Instance.new("TextButton", scroll)
            btn.Size = UDim2.new(1, -10, 0, 30)
            btn.Position = UDim2.new(0, 5, 0, y)
            btn.Text = plr.Name
            btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.Gotham
            btn.TextScaled = true
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

            btn.MouseButton1Click:Connect(function()
                if targetVictim == plr.Name then
                    targetVictim = nil
                    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
                else
                    targetVictim = plr.Name
                    for _, child in ipairs(scroll:GetChildren()) do
                        if child:IsA("TextButton") then
                            child.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
                        end
                    end
                    btn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
                end
            end)
            y += 35
        end
    end
    scroll.CanvasSize = UDim2.new(0, 0, 0, y)
end

players.PlayerAdded:Connect(refreshPlayers)
players.PlayerRemoving:Connect(refreshPlayers)

-- Frame AutoKill
local autoKillFrame = Instance.new("Frame", screenGui)
autoKillFrame.Size = UDim2.new(0, 250, 0, 120)
autoKillFrame.Position = UDim2.new(0.5, -125, 0.5, -60)
autoKillFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
autoKillFrame.Visible = false
autoKillFrame.Active = true
autoKillFrame.Draggable = true
Instance.new("UICorner", autoKillFrame).CornerRadius = UDim.new(0, 12)

local akTitle = Instance.new("TextLabel", autoKillFrame)
akTitle.Size = UDim2.new(1, -30, 0, 40)
akTitle.BackgroundTransparency = 1
akTitle.Text = "⚔️ AutoKill"
akTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
akTitle.Font = Enum.Font.GothamBold
akTitle.TextScaled = true

local akClose = Instance.new("TextButton", autoKillFrame)
akClose.Size = UDim2.new(0, 30, 0, 30)
akClose.Position = UDim2.new(1, -30, 0, 0)
akClose.Text = "❌"
akClose.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
akClose.TextScaled = true
akClose.TextColor3 = Color3.fromRGB(255, 255, 255)
akClose.Font = Enum.Font.GothamBold
Instance.new("UICorner", akClose).CornerRadius = UDim.new(0, 6)

akClose.MouseButton1Click:Connect(function()
    autoKillFrame.Visible = false
end)

local coming = Instance.new("TextLabel", autoKillFrame)
coming.Size = UDim2.new(1, 0, 0, 60)
coming.Position = UDim2.new(0, 0, 0, 45)
coming.BackgroundTransparency = 1
coming.Text = "Coming soon 😝"
coming.TextColor3 = Color3.fromRGB(255, 255, 255)
coming.Font = Enum.Font.Gotham
coming.TextScaled = true

-- Toggle frames
chooseBtn.MouseButton1Click:Connect(function()
    victimFrame.Visible = not victimFrame.Visible
    if victimFrame.Visible then
        refreshPlayers()
    end
end)

autoKillBtn.MouseButton1Click:Connect(function()
    autoKillFrame.Visible = not autoKillFrame.Visible
end)

-- ON/OFF logic
toggleBtn.MouseButton1Click:Connect(function()
    enabled = not enabled
    toggleBtn.Text = enabled and "ON" or "OFF"
    toggleBtn.BackgroundColor3 = enabled and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
end)

friendBtn.MouseButton1Click:Connect(function()
    ignoreFriends = not ignoreFriends
    friendBtn.Text = ignoreFriends and "👥 Ignore Friends: ON" or "👥 Ignore Friends: OFF"
end)

rangeBox.FocusLost:Connect(function()
    local newRange = tonumber(rangeBox.Text)
    if newRange and newRange >= 1 and newRange <= 1e17 then
        range = newRange
    else
        rangeBox.Text = tostring(range)
    end
end)

closeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    openBtn.Visible = true
end)

openBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    openBtn.Visible = false
end)

-- Kill Aura loop
runService.RenderStepped:Connect(function()
    if not enabled then return end
    for _, plr in ipairs(players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if safeList[plr.Name] then continue end
            if ignoreFriends and player:IsFriendsWith(plr.UserId) then continue end
            if targetVictim and plr.Name ~= targetVictim then continue end
            local v = plr.Character
            if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and player:DistanceFromCharacter(v.HumanoidRootPart.Position) <= range then
                local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
                if tool and tool:FindFirstChild("Handle") then
                    tool:Activate()
                    for _, part in ipairs(v:GetChildren()) do
                        if part:IsA("BasePart") then
                            firetouchinterest(tool.Handle, part, 0)
                            firetouchinterest(tool.Handle, part, 1)
                        end
                    end
                end
            end
        end
    end
end)
