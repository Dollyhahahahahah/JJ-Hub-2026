--[[
	JUJU DO PIX HUB - V42 + KEY SYSTEM + DRAG SYSTEM
	KEY: JJHUB2026
]]

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-------------------------------------------------------------------------
-- FUNÇÃO DE ARRASTAR (DRAG SYSTEM)
-------------------------------------------------------------------------
local function MakeDraggable(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-------------------------------------------------------------------------
-- INTERFACE DA KEY
-------------------------------------------------------------------------
local KeyGui = Instance.new("ScreenGui", player.PlayerGui)
KeyGui.Name = "JujuKeySystem"

local BG = Instance.new("Frame", KeyGui); BG.Size = UDim2.new(1, 0, 1, 0); BG.BackgroundColor3 = Color3.fromRGB(5, 5, 5); BG.BackgroundTransparency = 1
local MainKey = Instance.new("Frame", BG); MainKey.Size = UDim2.new(0, 320, 0, 190); MainKey.Position = UDim2.new(0.5, -160, 0.5, -95); MainKey.BackgroundColor3 = Color3.fromRGB(15, 15, 18); MainKey.BackgroundTransparency = 1; Instance.new("UICorner", MainKey); local StrokeK = Instance.new("UIStroke", MainKey); StrokeK.Color = Color3.fromRGB(0, 255, 150); StrokeK.Thickness = 2; StrokeK.Transparency = 1
local TitleK = Instance.new("TextLabel", MainKey); TitleK.Size = UDim2.new(1, 0, 0, 60); TitleK.Text = "JUJU DO PIX HUB V42"; TitleK.TextColor3 = Color3.fromRGB(0, 255, 150); TitleK.Font = "GothamBlack"; TitleK.TextSize = 20; TitleK.BackgroundTransparency = 1; TitleK.TextTransparency = 1
local Box = Instance.new("TextBox", MainKey); Box.Size = UDim2.new(0.85, 0, 0, 45); Box.Position = UDim2.new(0.075, 0, 0.35, 0); Box.PlaceholderText = "DIGITE A KEY..."; Box.Text = ""; Box.BackgroundColor3 = Color3.fromRGB(25, 25, 30); Box.TextColor3 = Color3.new(1, 1, 1); Box.Font = "GothamBold"; Box.TextSize = 14; Box.TextTransparency = 1; Box.BackgroundTransparency = 1; Instance.new("UICorner", Box)
local Verify = Instance.new("TextButton", MainKey); Verify.Size = UDim2.new(0.85, 0, 0, 45); Verify.Position = UDim2.new(0.075, 0, 0.65, 0); Verify.Text = "ATIVAR SCRIPT"; Verify.BackgroundColor3 = Color3.fromRGB(0, 255, 150); Verify.TextColor3 = Color3.fromRGB(10, 10, 10); Verify.Font = "GothamBlack"; Verify.TextSize = 14; Verify.TextTransparency = 1; Verify.BackgroundTransparency = 1; Instance.new("UICorner", Verify)

local info = TweenInfo.new(0.8, Enum.EasingStyle.Quint)
TweenService:Create(BG, info, {BackgroundTransparency = 0.3}):Play()
TweenService:Create(MainKey, info, {BackgroundTransparency = 0}):Play()
TweenService:Create(StrokeK, info, {Transparency = 0}):Play()
TweenService:Create(TitleK, info, {TextTransparency = 0}):Play()
TweenService:Create(Box, info, {TextTransparency = 0, BackgroundTransparency = 0}):Play()
TweenService:Create(Verify, info, {TextTransparency = 0, BackgroundTransparency = 0}):Play()

-------------------------------------------------------------------------
-- SCRIPT PRINCIPAL
-------------------------------------------------------------------------
local function CarregarScriptPrincipal()
    -- BYPASS
    local ACtable = {player.PlayerScripts:FindFirstChild("QuitsAntiCheatChecker"), player.PlayerScripts:FindFirstChild("QuitsAntiCheatLocal")}
    for _, v in pairs(ACtable) do if v then v:Destroy() end end

    local RunService = game:GetService("RunService")
    local Lighting = game:GetService("Lighting")
    local Camera = workspace.CurrentCamera
    local WallHopEnabled, SpeedEnabled = false, false
    local lastJump = 0
    local SETTINGS = {JumpHeight = 36, SpeedMult = 1.4, DetectDist = 3.2, FlickAngle = 45, RecoverySpeed = 0.15, FarmPos = CFrame.new(-32.1, 212.6, 111.0)}

    -- Interface do HUB
    local sg = Instance.new("ScreenGui", player.PlayerGui); sg.Name = "JujuV42"; sg.ResetOnSpawn = false 
    
    -- BOTÃO CIRCULAR (Bolinha) - COM DRAG
    local minBtn = Instance.new("TextButton", sg); minBtn.Size = UDim2.new(0, 45, 0, 45); minBtn.Position = UDim2.new(0.02, 0, 0.1, 0); minBtn.Text = "JP"; minBtn.BackgroundColor3 = Color3.new(0,0,0); minBtn.TextColor3 = Color3.fromRGB(0, 255, 150); minBtn.Font = "GothamBlack"; Instance.new("UICorner", minBtn).CornerRadius = UDim.new(1,0)
    MakeDraggable(minBtn)

    -- JANELA PRINCIPAL - COM DRAG
    local main = Instance.new("Frame", sg); main.Size = UDim2.new(0, 190, 0, 250); main.Position = UDim2.new(0.5, -95, 0.3, 0); main.BackgroundColor3 = Color3.fromRGB(15, 15, 18); main.Visible = true; Instance.new("UICorner", main); Instance.new("UIStroke", main).Color = Color3.fromRGB(0, 255, 150)
    MakeDraggable(main)

    -- Funções e Tabs
    local function ExecutarRitual()
        local char = player.Character; local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            task.spawn(function()
                local bg = Instance.new("BodyAngularVelocity", root); bg.AngularVelocity = Vector3.new(0, 10, 0); bg.MaxTorque = Vector3.new(0, 9e9, 0)
                task.wait(1); bg.AngularVelocity = Vector3.new(0, 50, 0)
                local bv = Instance.new("BodyVelocity", root); bv.Velocity = Vector3.new(0, 15, 0); bv.MaxForce = Vector3.new(0, 9e9, 0)
                task.wait(2); bv:Destroy(); bg:Destroy(); char.Humanoid.Health = 0
            end)
        end
    end

    local function CreateVisualESP(targetTeam, color)
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local isTarget = (p.Team and string.find(p.Team.Name:lower(), targetTeam:lower())) or string.find(p.Name:lower(), targetTeam:lower())
                if isTarget then
                    if p.Character:FindFirstChild("JujuFinalESP") then p.Character.JujuFinalESP:Destroy() end
                    local bg = Instance.new("BillboardGui", p.Character); bg.Name = "JujuFinalESP"; bg.AlwaysOnTop = true; bg.Size = UDim2.new(4.2, 0, 5.8, 0); bg.Adornee = p.Character.HumanoidRootPart
                    local box = Instance.new("Frame", bg); box.Size = UDim2.new(1, 0, 1, 0); box.BackgroundTransparency = 0.7; box.BackgroundColor3 = color; box.BorderSizePixel = 1; box.BorderColor3 = Color3.new(1,1,1)
                    local label = Instance.new("TextLabel", bg); label.Size = UDim2.new(1.5, 0, 0.2, 0); label.Position = UDim2.new(-0.25, 0, -0.22, 0); label.BackgroundTransparency = 1; label.TextColor3 = Color3.new(1,1,1); label.Font = "GothamBold"; label.TextSize = 11
                    task.spawn(function()
                        while p.Character and p.Character:FindFirstChild("HumanoidRootPart") and bg.Parent do
                            local dist = math.floor((player.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude)
                            label.Text = p.Name .. " M:" .. dist; task.wait(0.3)
                        end
                    end)
                end
            end
        end
    end

    local tabs = {}
    local tabBtns = Instance.new("Frame", main); tabBtns.Size = UDim2.new(1, 0, 0, 30); tabBtns.BackgroundTransparency = 1
    local function CreateTab(name, x)
        local b = Instance.new("TextButton", tabBtns); b.Size = UDim2.new(0.33, 0, 1, 0); b.Position = UDim2.new(x, 0, 0, 0); b.Text = name; b.BackgroundColor3 = Color3.fromRGB(20, 20, 25); b.TextColor3 = Color3.new(1,1,1); b.Font = "GothamBold"; b.TextSize = 8
        local f = Instance.new("ScrollingFrame", main); f.Size = UDim2.new(1, -10, 1, -40); f.Position = UDim2.new(0, 5, 0, 35); f.BackgroundTransparency = 1; f.Visible = (name == "Barts"); f.CanvasSize = UDim2.new(0, 0, 3.5, 0); f.ScrollBarThickness = 1; Instance.new("UIListLayout", f).Padding = UDim.new(0, 4)
        tabs[name] = f
        b.MouseButton1Click:Connect(function() for _, v in pairs(tabs) do v.Visible = false end f.Visible = true end)
    end
    CreateTab("Barts", 0); CreateTab("Homers", 0.33); CreateTab("ESP", 0.66)

    local function Btn(p, t, cb)
        local b = Instance.new("TextButton", p); b.Size = UDim2.new(0.95, 0, 0, 30); b.Text = t; b.BackgroundColor3 = Color3.fromRGB(25, 25, 30); b.TextColor3 = Color3.new(0.8, 0.8, 0.8); b.Font = "GothamBold"; b.TextSize = 9; Instance.new("UICorner", b); b.MouseButton1Click:Connect(function() cb(b) end)
    end

    Btn(tabs.Barts, "WALL HOP: OFF", function(b) WallHopEnabled = not WallHopEnabled; b.Text = "WALL HOP: "..(WallHopEnabled and "ON" or "OFF") end)
    Btn(tabs.Barts, "SPEED 1.4X", function(b) SpeedEnabled = not SpeedEnabled; b.Text = "SPEED: "..(SpeedEnabled and "ON" or "OFF") end)
    Btn(tabs.Barts, "FARM COIN", function() if player.Character then player.Character.HumanoidRootPart.CFrame = SETTINGS.FarmPos end end)
    Btn(tabs.Homers, "KILL ALL (0.5s)", function() local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart"); if root then local old = root.CFrame; for _, pl in pairs(Players:GetPlayers()) do if pl ~= player and pl.Character then root.CFrame = pl.Character.HumanoidRootPart.CFrame * CFrame.new(0,0,2); task.wait(0.5) end end; root.CFrame = old end end)
    Btn(tabs.Homers, "RITUAL HOMER", function() ExecutarRitual() end)
    Btn(tabs.ESP, "ESP HOMER", function() CreateVisualESP("homer", Color3.new(1,1,0)) end)
    Btn(tabs.ESP, "ESP BART", function() CreateVisualESP("bart", Color3.new(0,1,0)) end)
    Btn(tabs.ESP, "FULL BRIGHT", function() Lighting.Brightness = 2; Lighting.GlobalShadows = false; Lighting.ClockTime = 14 end)
    Btn(tabs.ESP, "FPS BOOST", function() for _,v in pairs(workspace:GetDescendants()) do if v:IsA("BasePart") then v.Material = "SmoothPlastic" end if v:IsA("Decal") or v:IsA("Texture") then v:Destroy() end end end)
    Btn(tabs.ESP, "LIMPAR ESP", function() for _,p in pairs(Players:GetPlayers()) do if p.Character and p.Character:FindFirstChild("JujuFinalESP") then p.Character.JujuFinalESP:Destroy() end end end)

    minBtn.MouseButton1Click:Connect(function() main.Visible = not main.Visible end)

    RunService.Heartbeat:Connect(function()
        local hum = player.Character and player.Character:FindFirstChild("Humanoid")
        if hum and SpeedEnabled then hum.WalkSpeed = 16 * SETTINGS.SpeedMult end
    end)
end

-- Lógica de Verificação da Key
Verify.MouseButton1Click:Connect(function()
    if Box.Text == "JJHUB2026" then
        Verify.Text = "ACESSO LIBERADO!"
        Verify.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        local fadeOut = TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
        TweenService:Create(MainKey, fadeOut, {BackgroundTransparency = 1}):Play()
        TweenService:Create(BG, fadeOut, {BackgroundTransparency = 1}):Play()
        task.wait(0.6)
        KeyGui:Destroy()
        CarregarScriptPrincipal()
    else
        Verify.Text = "KEY INVÁLIDA!"
        Verify.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        task.wait(1)
        Verify.Text = "ATIVAR SCRIPT"
        Verify.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
    end
end)
