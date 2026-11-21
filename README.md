--=============================================================
--  BLOX FRUITS • FARM HUB
--  Executar no jogo (Sea 1 / 2 / 3)
--=============================================================
local Players   = game:GetService("Players")
local RS        = game:GetService("ReplicatedStorage")
local RunService= game:GetService("RunService")
local LP        = Players.LocalPlayer

local VALID = {2753915549, 4442272183, 7449423635}
local ok = false
for _, id in ipairs(VALID) do
    if game.PlaceId == id then ok = true end
end
if not ok then return warn("[BF] Execute dentro do Blox Fruits.") end

---------------------------------------------------------------
-- CONFIG
---------------------------------------------------------------
local cfg = {
    AutoFarm   = false,
    AutoQuest  = true,
    AutoChest  = false,
    AutoFruit  = false,
    BringMobs  = true,
    KillAura   = false,
    ESP        = false,
    Height     = 25,
    Delay      = 0.12,
}

---------------------------------------------------------------
-- QUESTS (Sea 1) -> amplie livremente
---------------------------------------------------------------
local QUESTS = {
    {Lvl=1,    Q="Bandit"},
    {Lvl=10,   Q="Monkey"},
    {Lvl=20,   Q="Blade"},
    {Lvl=30,   Q="Brawler"},
    {Lvl=40,   Q="Pirate"},
    {Lvl=55,   Q="Brute"},
    {Lvl=75,   Q="Swift Ninja"},
    {Lvl=90,   Q="Sky Bandit"},
    {Lvl=105,  Q="Dark Master"},
    {Lvl=120,  Q="Toga Warrior"},
    {Lvl=135,  Q="Gladiator"},
    {Lvl=150,  Q="Military Soldier"},
    {Lvl=165,  Q="Military Spy"},
    {Lvl=190,  Q="Fishman Warrior"},
    {Lvl=225,  Q="Fishman Commando"},
    {Lvl=255,  Q="God's Guard"},
    {Lvl=290,  Q="Shanda"},
    {Lvl=320,  Q="Royal Squad"},
    {Lvl=350,  Q="Royal Soldier"},
    {Lvl=385,  Q="Galley Pirate"},
    {Lvl=420,  Q="Galley Captain"},
    {Lvl=475,  Q="Raider"},
    {Lvl=525,  Q="Mercenary"},
    {Lvl=600,  Q="Frog"},
    {Lvl=655,  Q="Snow Bandit"},
    {Lvl=675,  Q="Snowman"},
    {Lvl=725,  Q="Yeti"},
    {Lvl=775,  Q="Ice"},
    {Lvl=800,  Q="Chief"},
    {Lvl=825,  Q="Lab"},
    {Lvl=850,  Q="Reborn Skeleton"},
    {Lvl=875,  Q="Living Zombie"},
    {Lvl=900,  Q="Demonic Soul"},
    {Lvl=925,  Q="Possessed Mummy"},
    {Lvl=950,  Q="Frost"},
    {Lvl=975,  Q="Snow Lurker"},
    {Lvl=1000, Q="Arctic Warrior"},
    {Lvl=1025, Q="Mystery Man"},
    {Lvl=1050, Q="Snow Trooper"},
    {Lvl=1075, Q="Winter Warrior"},
}

---------------------------------------------------------------
-- UTIL
---------------------------------------------------------------
local function char()
    return LP.Character or LP.CharacterAdded:Wait()
end

local function root()
    local c = char()
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function tp(cf)
    local hrp = root()
    if hrp then hrp.CFrame = cf end
end

local function notify(txt)
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Farm Hub", Text = txt, Duration = 3
        })
    end)
end

local function bestQuest()
    local lvl = LP.Data.Level.Value
    local pick = nil
    for _, q in ipairs(QUESTS) do
        if lvl >= q.Lvl then pick = q end
    end
    return pick
end

local function findNPC(name)
    for _, m in ipairs(workspace:GetDescendants()) do
        if m:IsA("Model") and m.Name == name and m:FindFirstChild("Humanoid") then
            return m
        end
    end
end

local function questActive()
    local main = LP.PlayerGui:FindFirstChild("Main")
    local qf = main and main:FindFirstChild("Quest")
    if qf and qf.Visible then
        local title = qf:FindFirstChild("Title")
        return title and title.Text or ""
    end
    return nil
end

local function attack()
    local c = char()
    if not c then return end
    for _, t in ipairs(c:GetChildren()) do
        if t:IsA("Tool") and t:FindFirstChild("Handle") then
            pcall(function() t:Activate() end)
        end
    end
end

---------------------------------------------------------------
-- ESP
---------------------------------------------------------------
local espFolder
local function buildESP()
    espFolder = Instance.new("Folder")
    espFolder.Name = "BF_ESP"
    espFolder.Parent = game.CoreGui
end
if not espFolder then buildESP() end

local function addESP(obj, color, label)
    if obj:FindFirstChild("BF_Tag") then return end
    local bb = Instance.new("BillboardGui")
    bb.Name = "BF_Tag"
    bb.Size = UDim2.new(0, 120, 0, 34)
    bb.AlwaysOnTop = true
    bb.ExtentsOffset = Vector3.new(0, 3, 0)
    bb.Parent = obj
    local tx = Instance.new("TextLabel")
    tx.Size = UDim2.fromScale(1, 1)
    tx.BackgroundTransparency = 1
    tx.TextColor3 = color
    tx.TextStrokeTransparency = 0
    tx.TextScaled = true
    tx.Text = label
    tx.Parent = bb
end

RunService.RenderStepped:Connect(function()
    if not cfg.ESP then return end
    for _, f in ipairs(workspace:GetChildren()) do
        if f.Name == "Fruits" or f.Name == "_Fruits" then
            for _, fr in ipairs(f:GetChildren()) do
                if fr:IsA("Tool") then addESP(fr, Color3.fromRGB(0, 255, 120), fr.Name) end
            end
        end
        if f.Name == "_Chests" then
            for _, ch in ipairs(f:GetChildren()) do
                addESP(ch, Color3.fromRGB(255, 210, 0), "BAÚ")
            end
        end
    end
end)

---------------------------------------------------------------
-- LOOP PRINCIPAL
---------------------------------------------------------------
task.spawn(function()
    while task.wait(cfg.Delay) do
        local hrp = root()
        if not hrp then continue end

        if cfg.AutoFarm then
            -- 1) pegar quest
            if cfg.AutoQuest then
                local title = questActive()
                if not title then
                    local q = bestQuest()
                    local npc = q and findNPC(q.Q)
                    if npc and npc:FindFirstChild("HumanoidRootPart") then
                        tp(npc.HumanoidRootPart.CFrame * CFrame.new(0, 3, 3))
                        task.wait(0.35)
                        pcall(function()
                            RS.Remotes.CommF_:InvokeServer("StartQuest", q.Q, LP.Data.Level.Value)
                        end)
                    end
                end
            end

            -- 2) achar alvo
            local title = questActive()
            local target
            if title then
                local qname
                for _, q in ipairs(QUESTS) do
                    if string.find(title, q.Q, 1, true) then qname = q.Q break end
                end
                if qname then
                    local best, dist = nil, math.huge
                    for _, e in ipairs(workspace:GetChildren()) do
                        if e.Name == "Enemies" then
                            for _, m in ipairs(e:GetChildren()) do
                                if m.Name:find(qname, 1, true) and m:FindFirstChild("HumanoidRootPart")
                                   and m.Humanoid.Health > 0 then
                                    local d = (m.HumanoidRootPart.Position - hrp.Position).Magnitude
                                    if d < dist then best, dist = m, d end
                                end
                            end
                        end
                    end
                    target = best
                end
            end

            -- 3) farmar
            if target then
                tp(target.HumanoidRootPart.CFrame * CFrame.new(0, cfg.Height, 0))
                if cfg.BringMobs then
                    for _, e in ipairs(workspace:GetChildren()) do
                        if e.Name == "Enemies" then
                            for _, m in ipairs(e:GetChildren()) do
                                if m:FindFirstChild("HumanoidRootPart") and m.Humanoid.Health > 0 then
                                    if (m.HumanoidRootPart.Position - hrp.Position).Magnitude < 300 then
                                        m.HumanoidRootPart.CFrame =
                                            hrp.CFrame * CFrame.new(math.random(-8, 8), 0, math.random(-8, 8))
                                    end
                                end
                            end
                        end
                    end
                end
                attack()
            end
        end

        if cfg.KillAura then
            for _, e in ipairs(workspace:GetChildren()) do
                if e.Name == "Enemies" then
                    for _, m in ipairs(e:GetChildren()) do
                        if m:FindFirstChild("HumanoidRootPart") and m.Humanoid.Health > 0
                           and (m.HumanoidRootPart.Position - hrp.Position).Magnitude < 60 then
                            attack()
                        end
                    end
                end
            end
        end

        if cfg.AutoChest then
            for _, f in ipairs(workspace:GetChildren()) do
                if f.Name == "_Chests" then
                    for _, ch in ipairs(f:GetChildren()) do
                        if ch:FindFirstChild("HumanoidRootPart") then
                            tp(ch.HumanoidRootPart.CFrame * CFrame.new(0, 3, 0))
                            task.wait(0.25)
                        end
                    end
                end
            end
        end

        if cfg.AutoFruit then
            for _, f in ipairs(workspace:GetChildren()) do
                if f.Name == "Fruits" or f.Name == "_Fruits" then
                    for _, fr in ipairs(f:GetChildren()) do
                        if fr:IsA("Tool") and fr:FindFirstChild("Handle") then
                            tp(fr.Handle.CFrame * CFrame.new(0, 3, 0))
                            task.wait(0.3)
                        end
                    end
                end
            end
        end
    end
end)

---------------------------------------------------------------
-- TELEPORTES
---------------------------------------------------------------
local TP_LIST = {
    ["Café"]        = CFrame.new(-250, 20, -25),
    ["Bandit"]      = CFrame.new(-1400, 60, -1450),
    ["Jungle"]      = CFrame.new(-1450, 40, -300),
    ["Vila Pirata"] = CFrame.new(-1250, 40, 700),
    ["Deserto"]     = CFrame.new(1000, 50, 1000),
    ["Vila Gelo"]   = CFrame.new(-1200, 30, -8000),
    ["Marine Fort"] = CFrame.new(-5000, 30, -8000),
    ["Skylands"]    = CFrame.new(-5000, 300, -8000),
    ["Coliseu"]     = CFrame.new(-1000, 30, 6000),
    ["Magma"]       = CFrame.new(-5000, 50, 3000),
    ["Cidade Sub"]  = CFrame.new(-4000, 40, 5000),
    ["Fountain"]    = CFrame.new(1000, 60, 3000),
}

---------------------------------------------------------------
-- UI
---------------------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "BF_FarmHub"
gui.ResetOnSpawn = false
gui.Parent = game.CoreGui

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 210, 0, 400)
main.Position = UDim2.new(0, 15, 0, 100)
main.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0, 8)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 34)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
title.BorderSizePixel = 0
title.Text = "FARM HUB"
title.TextColor3 = Color3.fromRGB(0, 200, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 15
title.Parent = main
Instance.new("UICorner", title).CornerRadius = UDim.new(0, 8)

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -10, 1, -44)
scroll.Position = UDim2.new(0, 5, 0, 39)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 4
scroll.CanvasSize = UDim2.new(0, 0, 0, 700)
scroll.Parent = main

local y = 0
local function button(text, cb, color)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 30)
    b.Position = UDim2.new(0, 0, 0, y)
    b.BackgroundColor3 = color or Color3.fromRGB(35, 35, 44)
    b.BorderSizePixel = 0
    b.Text = text
    b.TextColor3 = Color3.fromRGB(235, 235, 235)
    b.Font = Enum.Font.Gotham
    b.TextSize = 12
    b.Parent = scroll
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    b.MouseButton1Click:Connect(cb)
    y = y + 34
    return b
end

local function toggle(label, key)
    local btn = button(label .. ": OFF", function() end, Color3.fromRGB(35, 35, 44))
    btn.MouseButton1Click:Connect(function()
        cfg[key] = not cfg[key]
        btn.Text = label .. ": " .. (cfg[key] and "ON" or "OFF")
        btn.BackgroundColor3 = cfg[key] and Color3.fromRGB(0, 130, 90) or Color3.fromRGB(35, 35, 44)
    end)
end

toggle("Auto Farm", "AutoFarm")
toggle("Auto Quest", "AutoQuest")
toggle("Auto Baú", "AutoChest")
toggle("Auto Fruta", "AutoFruit")
toggle("Bring Mobs", "BringMobs")
toggle("Kill Aura", "KillAura")
toggle("ESP Fruta/Baú", "ESP")

button("--- TELEPORTES ---", function() end, Color3.fromRGB(25, 25, 32))

for nome, cf in pairs(TP_LIST) do
    button("TP " .. nome, function()
        tp(cf)
    end)
end

scroll.CanvasSize = UDim2.new(0, 0, 0, y + 10)

notify("Farm Hub carregado")
