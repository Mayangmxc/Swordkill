-- ======================================
-- SCRIPT COMPLET TOUT-EN-UN
-- GUI + COMMANDES + REACH + HANG + VOID
-- ======================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Config = { Prefix = ":" }
local Commands = {}
local AdminList = {}
local LoopHangActive = false
local LoopHangTargets = {}
local HanggedPlayers = {}
local CharacterAddedConnections = {}
local LoopVoidActive = false
local VoidTargets = {}

-- ===========================
-- GUI RAYFIELD
-- ===========================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "Swordkill v2😎",
    LoadingTitle = "Build and battle☑️",
    LoadingSubtitle = "by Mayan_legameur12",
    ConfigurationSaving = { Enabled = true, FolderName = nil, FileName = "Digital Hub" },
    Discord = { Enabled = true, Invite = "wk2xtM8H", RememberJoins = true },
    KeySystem = false
})

-- Notification via Rayfield
local function notify(msg, dur)
    dur = dur or 3
    pcall(function()
        Rayfield:Notify({
            Title = "Commande",
            Content = tostring(msg),
            Duration = dur
        })
    end)
end

-- ===========================
-- COMMANDES
-- ===========================
local function addCommand(names, desc, fn)
    for _, name in ipairs(names) do
        Commands[name:lower()] = {fn = fn, desc = desc}
    end
end

local function hasAdmin(player)
    return table.find(AdminList, player.UserId) ~= nil
end

local function getTargets(arg)
    local targets = {}
    arg = arg:lower()
    if arg == "me" then
        table.insert(targets, LocalPlayer)
    elseif arg == "all" or arg == "others" then
        for _, p in pairs(Players:GetPlayers()) do
            if arg == "all" or (arg == "others" and p ~= LocalPlayer) then
                table.insert(targets, p)
            end
        end
    else
        for _, p in pairs(Players:GetPlayers()) do
            if p.Name:lower():find(arg) or p.DisplayName:lower():find(arg) then
                table.insert(targets, p)
            end
        end
    end
    return targets
end

local function parseAndRun(messageText, fromPlayer)
    if type(messageText) ~= "string" then return end
    if messageText:sub(1, #Config.Prefix) ~= Config.Prefix then return end
    if fromPlayer ~= LocalPlayer and not hasAdmin(fromPlayer) then return end

    local payload = messageText:sub(#Config.Prefix + 1)
    local parts = {}
    for w in payload:gmatch("%S+") do table.insert(parts, w) end
    if #parts == 0 then parts[1] = payload end

    local cmd = table.remove(parts, 1):lower()
    if not Commands[cmd] then
        notify("Commande inconnue : " .. cmd)
        return
    end

    local success, err = pcall(function() Commands[cmd].fn(parts, fromPlayer) end)
    if not success then
        warn("Erreur commande: " .. tostring(err))
        notify("Erreur exécution.")
    end
end

-- ===========================
-- COMMANDES ADMIN
-- ===========================
addCommand({"admin"}, "Donner l'admin à un joueur", function(args, fromPlayer)
    if fromPlayer ~= LocalPlayer then notify("Vous n'avez pas la permission.") return end
    local targetArg = args[1]
    if not targetArg then notify("Usage : :admin <nom>") return end
    local targets = getTargets(targetArg)
    if #targets == 0 then notify("Aucun joueur trouvé pour '" .. targetArg .. "'") return end
    for _, player in ipairs(targets) do
        if player == LocalPlayer then
            notify("Vous avez déjà tous les droits !")
        elseif hasAdmin(player) then
            notify(player.Name .. " a déjà l'admin")
        else
            table.insert(AdminList, player.UserId)
            notify(player.Name .. " a reçu l'admin ✅")
        end
    end
end)

addCommand({"unadmin"}, "Retirer l'admin à un joueur", function(args, fromPlayer)
    if fromPlayer ~= LocalPlayer then notify("Vous n'avez pas la permission.") return end
    local targetArg = args[1]
    if not targetArg then notify("Usage : :unadmin <nom>") return end
    local targets = getTargets(targetArg)
    if #targets == 0 then notify("Aucun joueur trouvé pour '" .. targetArg .. "'") return end
    for _, player in ipairs(targets) do
        if player == LocalPlayer then
            notify("Vous ne pouvez pas vous retirer l'admin !")
        else
            local index = table.find(AdminList, player.UserId)
            if not index then
                notify(player.Name .. " n'a pas l'admin")
            else
                table.remove(AdminList, index)
                notify(player.Name .. " a perdu l'admin ❌")
            end
        end
    end
end)

addCommand({"admins", "adminlist"}, "Voir la liste des admins", function(args, fromPlayer)
    if fromPlayer ~= LocalPlayer then return end
    if #AdminList == 0 then notify("Aucun admin actuellement") return end
    local adminNames = {}
    for _, userId in ipairs(AdminList) do
        local player = Players:GetPlayerByUserId(userId)
        if player then table.insert(adminNames, player.Name) end
    end
    notify("Admins: " .. table.concat(adminNames, ", "), 5)
end)

-- ===========================
-- HANG / LOOPHANG
-- ===========================
local function getPlate()
    local Plates = Workspace:FindFirstChild("Plates")
    if not Plates then return nil end
    for _, Plate in pairs(Plates:GetChildren()) do
        if Plate:FindFirstChild("Owner") and Plate.Owner.Value == LocalPlayer then
            return Plate:FindFirstChild("Plate")
        end
    end
    return nil
end

local function hangPlayer(player)
    if not player or not player.Character or not player.Character.PrimaryPart then return false end
    local StampAsset = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("StampAsset")
    local LPlate = getPlate()
    if not StampAsset or not LPlate then return false end
    local success = pcall(function()
        StampAsset:InvokeServer(
            41324945,
            LPlate.CFrame - Vector3.new(0, 2.5, 0),
            "{99ab22df-ca29-4143-a2fd-0a1b79db78c2}",
            {player.Character.PrimaryPart},
            0
        )
    end)
    return success
end

local function setupLoopHangForPlayer(player)
    if CharacterAddedConnections[player.UserId] then CharacterAddedConnections[player.UserId]:Disconnect() end
    if player.Character then task.wait(0.5) if hangPlayer(player) then HanggedPlayers[player.UserId] = true notify(player.Name .. " a été hang 🔒") end end
    CharacterAddedConnections[player.UserId] = player.CharacterAdded:Connect(function(character)
        task.wait(0.5)
        if LoopHangActive and table.find(LoopHangTargets, player) then
            if hangPlayer(player) then HanggedPlayers[player.UserId] = true notify(player.Name .. " re-hang après respawn 🔒") end
        end
    end)
end

addCommand({"hang"}, "Appliquer le hang", function(args, fromPlayer)
    local targetArg = args[1]
    if not targetArg then notify("Usage : :hang <me|nom|all|others>") return end
    local targets = getTargets(targetArg)
    if #targets == 0 then notify("Aucun joueur trouvé pour '" .. targetArg .. "'") return end
    local hangCount = 0
    for _, player in ipairs(targets) do
        if hangPlayer(player) then
            hangCount = hangCount + 1
            notify("Hang appliqué sur " .. player.Name)
        end
        task.wait(0.05)
    end
    if hangCount > 0 then notify("Total: " .. hangCount .. " hang(s) réussi(s)") end
end)

addCommand({"loophang"}, "Activer le loop hang", function(args)
    local targetArg = args[1]
    if not targetArg then notify("Usage : :loophang <me|nom|all|others>") return end
    local targets = getTargets(targetArg)
    if #targets == 0 then notify("Aucun joueur trouvé pour '" .. targetArg .. "'") return end
    LoopHangTargets = targets
    LoopHangActive = true
    for _, player in ipairs(targets) do setupLoopHangForPlayer(player) end
    notify("Loop hang activé sur " .. #targets .. " cible(s) 🔄")
end)

addCommand({"unloophang"}, "Désactiver le loop hang", function()
    LoopHangActive = false
    LoopHangTargets = {}
    HanggedPlayers = {}
    for _, conn in pairs(CharacterAddedConnections) do conn:Disconnect() end
    CharacterAddedConnections = {}
    notify("Loop hang désactivé ❌")
end)

-- ===========================
-- VOID / LOOPVOID
-- ===========================
local function voidPlayer(player)
    if not player or not player.Character or not player.Character.PrimaryPart then return false end
    local StampAsset = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("StampAsset")
    local LPlate = getPlate()
    if not StampAsset or not LPlate then return false end
    local success = pcall(function()
        StampAsset:InvokeServer(
            41324945,
            LPlate.CFrame - Vector3.new(0, 9e9, 0),
            "{99ab22df-ca29-4143-a2fd-0a1b79db78c2}",
            {player.Character.PrimaryPart},
            0
        )
    end)
    return success
end

addCommand({"void"}, "Void un joueur une fois", function(args)
    local targetArg = args[1]
    if not targetArg then notify("Usage : :void <me|nom|all|others>") return end
    local targets = getTargets(targetArg)
    local voidCount = 0
    for _, player in ipairs(targets) do
        if voidPlayer(player) then
            voidCount = voidCount + 1
            notify("Void appliqué sur " .. player.Name)
        end
        task.wait(0.05)
    end
    if voidCount > 0 then notify("Total: " .. voidCount .. " void(s) réussi(s)") end
end)

addCommand({"loopvoid"}, "Activer le loop void", function(args)
    local targetArg = args[1]
    if not targetArg then notify("Usage : :loopvoid <me|nom|all|others>") return end
    local targets = getTargets(targetArg)
    VoidTargets = targets
    LoopVoidActive = true
    task.spawn(function()
        while LoopVoidActive do
            local LPlate = getPlate()
            local StampAsset = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("StampAsset")
            if LPlate and StampAsset then
                for _, player in ipairs(VoidTargets) do
                    if player and player.Character and player.Character.PrimaryPart then
                        voidPlayer(player)
                        task.wait(0.05)
                    end
                end
            end
            task.wait(0.2)
        end
    end)
    notify("Loop void activé sur " .. #targets .. " cible(s) 🌀")
end)

addCommand({"unloopvoid"}, "Désactiver le loop void", function()
    LoopVoidActive = false
    VoidTargets = {}
    notify("Loop void désactivé ❌")
end)

-- ===========================
-- REACH SYSTEM
-- ===========================
local reachEnabled = false
local reachRange = 15
local reachConnection = nil
local showSphere = false
local spherePart = nil

local function createSphere()
    if spherePart then spherePart:Destroy() end
    spherePart = Instance.new("Part")
    spherePart.Shape = Enum.PartType.Ball
    spherePart.Material = Enum.Material.ForceField
    spherePart.Size = Vector3.new(reachRange*2, reachRange*2, reachRange*2)
    spherePart.Anchored = true
    spherePart.CanCollide = false
    spherePart.Transparency = 0.7
    spherePart.Color = Color3.fromRGB(0, 150, 255)
    spherePart.Parent = workspace
end

local function updateSphere()
    if showSphere and spherePart and LocalPlayer.Character then
        local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if tool and tool:FindFirstChild("Handle") then
            spherePart.CFrame = tool.Handle.CFrame
            spherePart.Size = Vector3.new(reachRange*2, reachRange*2, reachRange*2)
            spherePart.Transparency = 0.7
        else
            spherePart.Transparency = 1
        end
    end
end

RunService.RenderStepped:Connect(function()
    updateSphere()
end)

local function startReach()
    if reachConnection then return end
    reachConnection = RunService.RenderStepped:Connect(function()
        if not reachEnabled then return end
        local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if not tool then return end
        for i, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                if (LocalPlayer.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude <= reachRange then
                    tool:Activate()
                    for _, part in pairs(p.Character:GetChildren()) do
                        if part:IsA("BasePart") then
                            firetouchinterest(tool.Handle, part, 0)
                            firetouchinterest(tool.Handle, part, 1)
                        end
                    end
                end
            end
        end
    end)
end

local function stopReach()
    if reachConnection then
        reachConnection:Disconnect()
        reachConnection = nil
    end
end

-- ===========================
-- CHAT CONNECTION
-- ===========================
LocalPlayer.Chatted:Connect(function(msg) parseAndRun(msg, LocalPlayer) end)
Players.PlayerAdded:Connect(function(player)
    player.Chatted:Connect(function(msg) parseAndRun(msg, player) end)
end)

-- ===========================
-- GUI MAIN
-- ===========================
local Tab = Window:CreateTab("Main")
local Tab1 = Window:CreateTab("Cmds🌐")

-- Input reach range
Tab:CreateInput({
    Name = "Reach Range",
    PlaceholderText = "1-1000",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        local value = tonumber(Text)
        if value and value >= 1 then
            reachRange = value
            notify("Reach range défini à "..value)
            if spherePart then spherePart.Size = Vector3.new(reachRange*2, reachRange*2, reachRange*2) end
        else notify("Valeur invalide !") end
    end
})

-- Toggle reach
Tab:CreateToggle({
    Name = "Enable Reach",
    CurrentValue = false,
    Callback = function(Value)
        reachEnabled = Value
        if Value then
            startReach()
            notify("Reach activé")
        else
            stopReach()
            notify("Reach désactivé")
        end
    end
})

-- Toggle sphere
Tab:CreateToggle({
    Name = "Show Reach Sphere",
    CurrentValue = false,
    Callback = function(Value)
        showSphere = Value
        if Value and not spherePart then createSphere() end
        if not Value and spherePart then spherePart.Transparency = 1 end
        notify(Value and "Sphère de reach visible" or "Sphère de reach cachée")
    end
})

-- Player name input for commands
local playerName = ""
Tab1:CreateInput({
    Name = "Player Name",
    PlaceholderText = "Enter player name",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text) playerName = Text end
})

-- Buttons
Tab1:CreateButton({
    Name = "Give Admin",
    Callback = function()
        if playerName ~= "" then parseAndRun(":admin "..playerName, LocalPlayer) end
    end
})

Tab1:CreateButton({
    Name = "Unadmin",
    Callback = function()
        if playerName ~= "" then parseAndRun(":unadmin "..playerName, LocalPlayer) end
    end
})

Tab1:CreateButton({
    Name = "Destroy GUI",
    Callback = function()
        stopReach()
        if spherePart then spherePart:Destroy() end
        Rayfield:Destroy()
    end
})

-- ===========================
-- INIT
-- ===========================
notify("✅ Système complet chargé !")
notify("🔨 Hang: :hang, :loophang, :unloophang")
notify("🌀 Void: :void, :loopvoid, :unloopvoid")
notify("👑 Admin: :admin, :unadmin, :admins")
