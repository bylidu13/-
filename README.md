local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- 清除舊 UI
if PlayerGui:FindFirstChild("PinkTabUI_Final") then
    PlayerGui.PinkTabUI_Final:Destroy()
end

local THEME = {
    Main   = Color3.fromRGB(255,182,193), 
    Dark   = Color3.fromRGB(35,35,45),    
    Stroke = Color3.fromRGB(255,200,210), 
    Text   = Color3.fromRGB(255,240,245)  
}

-- 全局開關變數
_G.FastAttackEnabled = false
_G.NoClipEnabled = false -- 
_G.AutoHakiEnabled = false
_G.AutoV3Enabled = false
_G.AutoV4Enabled = false
_G.VelocityValue = 0

local ui = Instance.new("ScreenGui")
ui.Name = "PinkTabUI_Final"
ui.ResetOnSpawn = false
ui.Parent = PlayerGui

-- 面板 (尺寸 400x320)
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 400, 0, 320) 
panel.Position = UDim2.new(0.5, -200, 0.5, -160)
panel.BackgroundColor3 = THEME.Dark
panel.BackgroundTransparency = 0.15
panel.Active = true
panel.Draggable = true
panel.Parent = ui

local panelCorner = Instance.new("UICorner", panel)
panelCorner.CornerRadius = UDim.new(0, 14)

local panelStroke = Instance.new("UIStroke", panel)
panelStroke.Color = THEME.Stroke
panelStroke.Thickness = 1.5
panelStroke.Transparency = 0.35

-- 標題
local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.Text = "媽的楊改瑞"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = THEME.Text

-- 關閉鈕
local closeBtn = Instance.new("TextButton", panel)
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -32, 0, 6)
closeBtn.Text = "×"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 80, 90)
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(1, 0)
closeBtn.MouseButton1Click:Connect(function() ui:Destroy() end)

-- ==========================================
local floatBall = Instance.new("TextButton")
floatBall.Name = "FloatBall"
floatBall.Size = UDim2.new(0, 50, 0, 50)
floatBall.Position = UDim2.new(0, 20, 0, 20) -- 確保在左上角看得見的地方
floatBall.BackgroundColor3 = THEME.Dark
floatBall.BackgroundTransparency = 0.2
floatBall.Text = "🥰"
floatBall.TextSize = 30
floatBall.TextColor3 = Color3.new(1, 1, 1)
floatBall.Font = Enum.Font.GothamBold
floatBall.Parent = ui
floatBall.ZIndex = 10 -- 確保在最上層

Instance.new("UICorner", floatBall).CornerRadius = UDim.new(0, 12)
local bStroke = Instance.new("UIStroke", floatBall)
bStroke.Thickness = 2
bStroke.Color = THEME.Main

-- ==========================================
-- 4. 懸浮球邏輯
-- ==========================================
-- ==========================================
-- 4. 懸浮球邏輯 (修正版：載入即打開，圖示同步)
-- ==========================================
local isUIVisible = true -- 設定初始變數為打開
panel.Visible = true     -- 設定面板初始為顯示

-- 同步函數：確保球的圖示跟面板狀態永遠一致
local function updateUIState()
    -- true (打開) -> 😘
    -- false (關閉) -> 🥰
    floatBall.Text = isUIVisible and "😘" or "🥰"
    panel.Visible = isUIVisible
end

-- 切換函數
local function toggleUI()
    isUIVisible = not isUIVisible
    updateUIState()
end

-- 關鍵：腳本執行到這裡時，立刻跑一次同步，確保球顯示 😘
updateUIState()

-- 拖動與點擊邏輯
local dragging = false
local dragStart, startPos
local dragThreshold = 8 -- 點擊靈敏度

floatBall.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = floatBall.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        floatBall.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X, 
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if dragging then
            dragging = false
            -- 如果手指/滑鼠移動距離很小，判定為「點擊」而非「拖動」
            if (input.Position - dragStart).Magnitude < dragThreshold then
                toggleUI()
            end
        end
    end
end)

-- ==========================================

-- 頁籤區
local tabs, pages = {}, {}
local tabNames = {"功能", "透視", "傳送", "設定", "其他",}
local tabHolder = Instance.new("Frame", panel)
tabHolder.Size = UDim2.new(1, -20, 0, 32)
tabHolder.Position = UDim2.new(0, 10, 0, 40)
tabHolder.BackgroundTransparency = 1

local tabSpacing = 6
local tabWidth = (380 - (#tabNames - 1) * tabSpacing) / #tabNames

for i, name in ipairs(tabNames) do
    local tabBtn = Instance.new("TextButton", tabHolder)
    tabBtn.Size = UDim2.new(0, tabWidth, 1, 0)
    tabBtn.Position = UDim2.new(0, (tabWidth + tabSpacing) * (i - 1), 0, 0)
    tabBtn.Text = name
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 14
    tabBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
    tabBtn.TextColor3 = (i == 1) and THEME.Main or THEME.Text
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)

    local page = Instance.new("ScrollingFrame", panel)
    page.Size = UDim2.new(1, -20, 1, -95)
    page.Position = UDim2.new(0, 10, 0, 85)
    page.BackgroundTransparency = 1
    page.Visible = (i == 1)
    page.ScrollBarThickness = 4
    page.ScrollBarImageColor3 = THEME.Main
    page.CanvasSize = UDim2.new(0, 0, 0, 0) 
    page.ScrollingDirection = Enum.ScrollingDirection.Y
    
    local layout = Instance.new("UIListLayout", page)
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder

    -- 動態 CanvasSize 確保不擠壓
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
    end)

    tabBtn.MouseButton1Click:Connect(function()
        for _, p in ipairs(pages) do p.Visible = false end
        page.Visible = true
        for _, t in ipairs(tabs) do t.TextColor3 = THEME.Text end
        tabBtn.TextColor3 = THEME.Main
    end)
    tabs[i], pages[i] = tabBtn, page
end

-- [組件函式]

-- 分隔標籤
local function CreateSection(parent, text)
    local label = Instance.new("TextLabel", parent)
    label.Size = UDim2.new(1, -5, 0, 25)
    label.BackgroundTransparency = 1
    label.Text = "--- " .. text .. " ---"
    label.Font = Enum.Font.GothamBold
    label.TextSize = 13
    label.TextColor3 = THEME.Main
end

-- 帶確認通知的跨海傳送 (一二三海)
local function CreateTravelButton(parent, worldName, remoteArg)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -5, 0, 38)
    b.Text = "🚢 傳送至" .. worldName
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BackgroundColor3 = Color3.fromRGB(60, 60, 75)
    b.TextColor3 = THEME.Text
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

    b.MouseButton1Click:Connect(function()
        local bindable = Instance.new("BindableFunction")
        bindable.OnInvoke = function(answer)
            if answer == "確定" then
                pcall(function()
                    ReplicatedStorage.Remotes.CommF_:InvokeServer(remoteArg)
                end)
            end
        end

        StarterGui:SetCore("SendNotification", {
            Title = "傳送" .. worldName,
            Text = "確定要前往嗎？",
            Icon = "rbxassetid://0",
            Duration = 5,
            Callback = bindable,
            Button1 = "確定",
            Button2 = "取消"
        })
    end)
end

-- 普通座標傳送按鈕
local function CreateTPButton(parent, text, color, cf)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -5, 0, 38)
    b.Text = "📍 " .. text
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BackgroundColor3 = color
    b.TextColor3 = THEME.Text
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
    b.MouseButton1Click:Connect(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = cf
        end
    end)
end

-- 開關組件
local function CreateToggle(parent, text, globalVar)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(1, -5, 0, 35)
    frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -60, 1, 0)
    label.Text = text
    label.TextColor3 = THEME.Text
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    local toggle = Instance.new("Frame", frame)
    toggle.Size = UDim2.new(0, 44, 0, 22)
    toggle.Position = UDim2.new(1, -50, 0.5, -11)
    toggle.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
    Instance.new("UICorner", toggle).CornerRadius = UDim.new(1, 0)
    local knob = Instance.new("Frame", toggle)
    knob.Size = UDim2.new(0, 18, 0, 18)
    knob.Position = UDim2.new(0, 2, 0.5, -9)
    knob.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    toggle.InputBegan:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
        _G[globalVar] = not _G[globalVar]
        local active = _G[globalVar]
        TweenService:Create(toggle, TweenInfo.new(0.2), {BackgroundColor3 = active and THEME.Main or Color3.fromRGB(70, 70, 80)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.2), {Position = active and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)}):Play()
    end)
end

-- [分配內容]

-- 功能
CreateToggle(pages[1], "⚡ 快速攻擊", "FastAttackEnabled")
CreateToggle(pages[1], "🧱 穿牆模式", "NoClipEnabled")
CreateToggle(pages[1], "🛡️ Auto Haki", "AutoHakiEnabled")
CreateToggle(pages[1], "🧬 Auto Race V3", "AutoV3Enabled")
CreateToggle(pages[1], "🔥 Auto Race V4", "AutoV4Enabled")


-- 🚀 整合飛行按鈕在此
local flyBtn = Instance.new("TextButton", pages[1])
flyBtn.Size = UDim2.new(1, -5, 0, 38)
flyBtn.Text = "✈️ 飛行腳本 (復活不了就把飛行關掉)"
flyBtn.Font = Enum.Font.GothamBold
flyBtn.TextSize = 14
flyBtn.BackgroundColor3 = Color3.fromRGB(80, 100, 120)
flyBtn.TextColor3 = THEME.Text
Instance.new("UICorner", flyBtn).CornerRadius = UDim.new(0, 8)
flyBtn.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/ectom8810/lid12/refs/heads/main/fly"))()
end)

-- 傳送頁
local tpPage = pages[3]
local color2 = Color3.fromRGB(80, 70, 90)
local color3 = Color3.fromRGB(70, 85, 90)

CreateSection(tpPage, "世界")
CreateTravelButton(tpPage, "一海", "TravelMain")
CreateTravelButton(tpPage, "二海", "TravelDressrosa")
CreateTravelButton(tpPage, "三海", "TravelZou")

CreateSection(tpPage, "二海")
CreateTPButton(tpPage, "天鵝房", color2, CFrame.new(-287, 305, 592))
CreateTPButton(tpPage, "  豪宅", color2, CFrame.new(2287, 15, 910))
CreateTPButton(tpPage, "鬼船裡", color2, CFrame.new(-6502, 83, -123))
CreateTPButton(tpPage, "鬼船外", color2, CFrame.new(922, 124, 32842))
CreateTPButton(tpPage, "鬼船(傷害)", color2, CFrame.new(916, 252.3, 32910))

CreateSection(tpPage, "三海")
CreateTPButton(tpPage, "海上城堡", color3, CFrame.new(-12464, 376, -7567))
CreateTPButton(tpPage, "海德拉島", color3, CFrame.new(-5028, 316, -3207))
CreateTPButton(tpPage, "烏龜屋", color3, CFrame.new(-5061, 316, -3194))
CreateTPButton(tpPage, "司法島", color3, CFrame.new(-5098, 316, -3178))

-- 穿牆
RunService.Stepped:Connect(function()
    if _G.NoClipEnabled and player.Character then
        for _, v in ipairs(player.Character:GetChildren()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

-- 快速攻擊邏輯
task.spawn(function()
    local scanTick = 0
    local targets = {}
    local LastAttack = 0 -- 引入 crack.txt 的冷卻計時器

    while true do
        task.wait(0.01) -- 高頻檢測
        
        -- 檢查開關與角色狀態 (比照 crack.txt 的嚴謹判斷)
        if _G.FastAttackEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            
            pcall(function()
                local Net = ReplicatedStorage:WaitForChild("Modules"):WaitForChild("Net")
                local RegAttack = Net:FindFirstChild("RE/RegisterAttack")
                local RegHit = Net:FindFirstChild("RE/RegisterHit")
                
                -- 1. 冷卻檢查 (類似 crack.txt 的 tick 判斷)
                -- 如果距離上次攻擊小於 0.1 秒則跳過 (數值可依需求微調，0.1-0.15 最穩)
                if tick() - LastAttack < 0.1 then 
                    return 
                end

                -- 2. 目標掃描 (每 0.4 秒更新一次列表)
                if tick() - scanTick > 0.4 then
                    scanTick = tick()
                    targets = {}
                    local myPos = player.Character.HumanoidRootPart.Position
                    
                    -- 掃描怪物
                    if workspace:FindFirstChild("Enemies") then
                        for _, v in ipairs(workspace.Enemies:GetChildren()) do
                            local hrp = v:FindFirstChild("HumanoidRootPart")
                            local hum = v:FindFirstChild("Humanoid")
                            if hrp and hum and hum.Health > 0 and (hrp.Position - myPos).Magnitude < 5000 then
                                table.insert(targets, {v, hrp})
                            end
                        end
                    end
                    
                    -- 掃描玩家
                    for _, p in ipairs(Players:GetPlayers()) do
                        if p ~= player and p.Character then
                            local pRoot = p.Character:FindFirstChild("HumanoidRootPart")
                            local pHum = p.Character:FindFirstChild("Humanoid")
                            if pRoot and pHum and pHum.Health > 0 and (pRoot.Position - myPos).Magnitude < 5000 then
                                table.insert(targets, {p.Character, pRoot})
                            end
                        end
                    end
                end

                -- 3. 執行打擊 (加入 crack.txt 式的驗證)
                if #targets > 0 then
                    LastAttack = tick() -- 更新最後攻擊時間
                    
                    -- 模仿 crack.txt 的循環打擊，但加入存活檢查
                    for i = 1, 4 do
                        -- 檢查第一個目標是否還活著，死了就跳出
                        if targets[1] and targets[1][1] and targets[1][1]:FindFirstChild("Humanoid") and targets[1][1].Humanoid.Health > 0 then
                            RegAttack:FireServer(0)
                            RegHit:FireServer(targets[1][2], targets)
                        else
                            break -- 目標死亡，停止本次循環
                        end
                        task.wait(0.01) -- 防爆 Ping
                    end
                end
            end)
        end
    end
end)

-- 秒開邏輯
task.spawn(function()
    while true do
        RunService.Heartbeat:Wait()
        local char = player.Character
        if char and char:FindFirstChildOfClass("Humanoid") and char:FindFirstChildOfClass("Humanoid").Health > 0 then
            if _G.AutoHakiEnabled and not char:FindFirstChild("HasBuso") then
                pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso") end)
            end
            if _G.AutoV3Enabled then pcall(function() ReplicatedStorage.Remotes.CommE:FireServer("ActivateAbility") end) end
            if _G.AutoV4Enabled then
                local awk = player.Backpack:FindFirstChild("Awakening") or char:FindFirstChild("Awakening")
                if awk and awk:FindFirstChild("RemoteFunction") then pcall(function() awk.RemoteFunction:InvokeServer(true) end) end
            end
        end
    end
end)

-- 其他頁 (兩個腳本合一)
local otherPage = pages[5]
CreateSection(otherPage, "畫質(開啟後不能關閉 ! ! ! )")
local optBtn = Instance.new("TextButton", otherPage)
optBtn.Size = UDim2.new(1, -5, 0,45) -- 加高按鈕
optBtn.Text = "✨ 減畫質 (點了需等一下,視野變小請到設定調圓形畫質)"
optBtn.Font = Enum.Font.GothamBold
optBtn.TextSize = 14 -- 文字加大
optBtn.BackgroundColor3 = Color3.fromRGB(60, 80, 75)
optBtn.TextColor3 = THEME.Text
Instance.new("UICorner", optBtn).CornerRadius = UDim.new(0, 8)

optBtn.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/ectom8810/lid12/refs/heads/main/graphics'))()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/ectom8810/lid12/refs/heads/main/graphics2'))()
end)

-- 移除霧按鈕
local fogBtn = Instance.new("TextButton", otherPage)
fogBtn.Size = UDim2.new(1, -5, 0, 45)
fogBtn.Text = "🌫️ 移除霧"
fogBtn.Font = Enum.Font.GothamBold
fogBtn.TextSize = 16
fogBtn.BackgroundColor3 = Color3.fromRGB(70, 60, 85)
fogBtn.TextColor3 = THEME.Text
Instance.new("UICorner", fogBtn).CornerRadius = UDim.new(0, 8)
fogBtn.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/ectom8810/lid12/refs/heads/main/nofog'))()
end)

CreateSection(otherPage, "效能")

local statsLabel = Instance.new("TextLabel", otherPage)
statsLabel.Size = UDim2.new(1, -5, 0, 40)
statsLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
statsLabel.TextColor3 = Color3.new(1, 1, 1)
statsLabel.TextSize = 14
statsLabel.Font = Enum.Font.Code
statsLabel.Text = "  數據計算中..."
Instance.new("UICorner", statsLabel).CornerRadius = UDim.new(0, 8)

local toggleFloat = Instance.new("TextButton", otherPage)
toggleFloat.Size = UDim2.new(1, -5, 0, 40)
toggleFloat.Text = "📊 開啟/關閉 小字懸浮窗 (可拖動)"
toggleFloat.Font = Enum.Font.GothamBold
toggleFloat.TextSize = 14
toggleFloat.BackgroundColor3 = Color3.fromRGB(80, 60, 85)
toggleFloat.TextColor3 = THEME.Text
Instance.new("UICorner", toggleFloat).CornerRadius = UDim.new(0, 8)

-- 縮小版懸浮窗 
local floatFrame = Instance.new("Frame", ui)
floatFrame.Size = UDim2.new(0, 130, 0, 45) -- 尺寸縮小 
floatFrame.Position = UDim2.new(0.85, 0, 0.15, 0)
floatFrame.BackgroundColor3 = THEME.Dark
floatFrame.BackgroundTransparency = 0.3
floatFrame.Visible = false
floatFrame.Active = true
floatFrame.Draggable = true 
Instance.new("UICorner", floatFrame).CornerRadius = UDim.new(0, 8)
local fStroke = Instance.new("UIStroke", floatFrame)
fStroke.Color = THEME.Main
fStroke.Thickness = 1.5

local floatLabel = Instance.new("TextLabel", floatFrame)
floatLabel.Size = UDim2.new(1, 0, 1, 0)
floatLabel.BackgroundTransparency = 1
floatLabel.TextColor3 = Color3.new(1, 1, 1) -- 白色
floatLabel.Font = Enum.Font.GothamBold
floatLabel.TextSize = 15 -- 字體縮小 
floatLabel.Text = "..."

toggleFloat.MouseButton1Click:Connect(function()
    floatFrame.Visible = not floatFrame.Visible
end)

-- 核心：一秒平均 FPS 更新邏輯 
local frameCount = 0
local lastUpdate = tick()

RunService.RenderStepped:Connect(function()
    frameCount = frameCount + 1
    local now = tick()
    local delta = now - lastUpdate
    
    if delta >= 1 then -- 每一秒刷新一次數據 
        local avgFps = math.floor(frameCount / delta)
        local ping = math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
        local str = string.format("FPS: %d | Ping: %dms", avgFps, ping)
        
        statsLabel.Text = "  " .. str
        floatLabel.Text = string.format("FPS: %d\nPing: %dms", avgFps, ping)
        
        frameCount = 0
        lastUpdate = now
    end
end)

-- ✨ 依照你要求的樣式製作的視距解鎖按鈕
local zoomBtn = Instance.new("TextButton", pages[4])
zoomBtn.Size = UDim2.new(1, -5, 0, 45) -- 你要求的 45 高度
zoomBtn.Text = "🔭 解鎖視野距離"
zoomBtn.Font = Enum.Font.GothamBold
zoomBtn.TextSize = 14
zoomBtn.BackgroundColor3 = Color3.fromRGB(60, 80, 75) -- 你指定的顏色
zoomBtn.TextColor3 = THEME.Text
Instance.new("UICorner", zoomBtn).CornerRadius = UDim.new(0, 8)

zoomBtn.MouseButton1Click:Connect(function()
    pcall(function()
        player.CameraMaxZoomDistance = 5000
        -- 強制將 FOV 鎖定在一個較穩定的數值，減少滑動視覺誤差
        workspace.CurrentCamera.FieldOfView = 60
    end)
end)

-- Velocity Slide (滑桿)
local setPage = pages[4]

local speedFrame = Instance.new("Frame", setPage)
speedFrame.Size = UDim2.new(1, -5, 0, 60)
speedFrame.BackgroundTransparency = 1

local speedTitle = Instance.new("TextLabel", speedFrame)
speedTitle.Size = UDim2.new(0, 120, 0, 25)
speedTitle.Text = "🚀 移動速度 (會改變飛行速度)"
speedTitle.TextColor3 = THEME.Text
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextSize = 14
speedTitle.BackgroundTransparency = 1
speedTitle.TextXAlignment = Enum.TextXAlignment.Left

local speedBox = Instance.new("TextLabel", speedFrame)
speedBox.Size = UDim2.new(0, 55, 0, 22)
speedBox.Position = UDim2.new(1, -60, 0, 0)
speedBox.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.Text = "0.0"
speedBox.Font = Enum.Font.Code
speedBox.TextSize = 12
Instance.new("UICorner", speedBox)

local bar = Instance.new("Frame", speedFrame)
bar.Size = UDim2.new(1, -20, 0, 6)
bar.Position = UDim2.new(0, 10, 0, 45)
bar.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
Instance.new("UICorner", bar)

local handle = Instance.new("TextButton", bar)
handle.Size = UDim2.new(0, 30, 0, 18)
handle.AnchorPoint = Vector2.new(0.5, 0.5)
handle.Position = UDim2.new(0, 0, 0.5, 0)
handle.BackgroundColor3 = Color3.new(1, 1, 1)
handle.Text = ""
Instance.new("UICorner", handle)

local dragging = false
handle.MouseButton1Down:Connect(function() dragging = true end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

RunService.RenderStepped:Connect(function()
    if dragging then
        local mouseX = UserInputService:GetMouseLocation().X
        local relative = math.clamp((mouseX - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
        _G.VelocityValue = relative * 200
        handle.Position = UDim2.new(relative, 0, 0.5, 0)
        speedBox.Text = string.format("%.1f", _G.VelocityValue)
    end
end)

RunService.Heartbeat:Connect(function()
    if _G.VelocityValue > 0 then
        pcall(function()
            local char = player.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChild("Humanoid")
            if root and hum and hum.MoveDirection.Magnitude > 0 then
                root.CFrame = root.CFrame + (hum.MoveDirection * (_G.VelocityValue / 28))
            end
        end)
    end
end)

-- JumpPower Slide (跳躍力滑桿) - 加入在移動速度下方

local setPage = pages[4]

local OFFSET_JUMP = 50 -- 隱形的基礎強度
_G.JumpPowerValue = 0  -- UI 顯示的數值

local jumpFrame = Instance.new("Frame", setPage)
jumpFrame.Size = UDim2.new(1, -5, 0, 60)
jumpFrame.BackgroundTransparency = 1

local jumpTitle = Instance.new("TextLabel", jumpFrame)
jumpTitle.Size = UDim2.new(0, 120, 0, 25)
jumpTitle.Text = "🚀 跳躍力量 (開佛需改變數值)"
jumpTitle.TextColor3 = THEME.Text
jumpTitle.Font = Enum.Font.GothamBold
jumpTitle.TextSize = 14
jumpTitle.BackgroundTransparency = 1
jumpTitle.TextXAlignment = Enum.TextXAlignment.Left

local jumpBox = Instance.new("TextLabel", jumpFrame)
jumpBox.Size = UDim2.new(0, 55, 0, 22)
jumpBox.Position = UDim2.new(1, -60, 0, 0)
jumpBox.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
jumpBox.TextColor3 = Color3.new(1, 1, 1)
jumpBox.Text = "0.0" -- 初始顯示 0
jumpBox.Font = Enum.Font.Code
jumpBox.TextSize = 12
Instance.new("UICorner", jumpBox)

local jBar = Instance.new("Frame", jumpFrame)
jBar.Size = UDim2.new(1, -20, 0, 6)
jBar.Position = UDim2.new(0, 10, 0, 45)
jBar.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
Instance.new("UICorner", jBar)

local jHandle = Instance.new("TextButton", jBar)
jHandle.Size = UDim2.new(0, 30, 0, 18)
jHandle.AnchorPoint = Vector2.new(0.5, 0.5)
jHandle.Position = UDim2.new(0, 0, 0.5, 0) -- 滑桿初始在 0
jHandle.BackgroundColor3 = Color3.new(1, 1, 1)
jHandle.Text = ""
Instance.new("UICorner", jHandle)

local jDragging = false
jHandle.MouseButton1Down:Connect(function() jDragging = true end)
UserInputService.InputEnded:Connect(function(i) 
    if i.UserInputType == Enum.UserInputType.MouseButton1 then jDragging = false end 
end)

RunService.RenderStepped:Connect(function()
    if jDragging then
        local mouseX = UserInputService:GetMouseLocation().X
        local relative = math.clamp((mouseX - jBar.AbsolutePosition.X) / jBar.AbsoluteSize.X, 0, 1)
        -- 滑桿控制 0-700
        _G.JumpPowerValue = relative * 450 
        jHandle.Position = UDim2.new(relative, 0, 0.5, 0)
        -- UI 顯示：拉到多少就顯示多少
        jumpBox.Text = string.format("%.1f", _G.JumpPowerValue)
    end
end)

-- 實時應用邏輯
RunService.Heartbeat:Connect(function()
    pcall(function()
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then
            -- 關鍵在這裡：實際強度 = UI數值 + 50
            -- 這樣當 UI 是 0 時，實際就是 50 強度
            -- 當 UI 是 30 時，實際就是 80 強度
            local realPower = _G.JumpPowerValue + OFFSET_JUMP
            
            if hum.UseJumpPower then
                hum.JumpPower = realPower
            else
                hum.JumpHeight = realPower
            end
        end
    end)
end)

-- ==========================================
-- ESP Pro 核心邏輯 (DisplayName + @Username)
-- ==========================================
local espEnabled = false
local espObjects = {}
local RandomID = math.random(1, 1000000)

local function Round(num)
    return math.floor(tonumber(num) + 0.5)
end

local function getTeamColor(otherP)
    return (otherP.Team ~= player.Team) and Color3.fromRGB(255, 60, 60) or Color3.fromRGB(60, 160, 255)
end

local function removeESP(otherP)
    if espObjects[otherP] then
        for _, v in pairs(espObjects[otherP]) do
            if typeof(v) == "Instance" then v:Destroy() end
        end
        espObjects[otherP] = nil
    end
end

local function createESP(otherP)
    if otherP == player or not otherP.Character then return end
    removeESP(otherP)
    
    local char = otherP.Character
    local head = char:WaitForChild("Head", 5)
    if not head then return end

    -- 發光外框
    local highlight = Instance.new("Highlight")
    highlight.Adornee = char
    highlight.FillTransparency = 1
    highlight.OutlineColor = getTeamColor(player)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = char

    -- 資訊看板
    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 200, 0, 60)
    billboard.AlwaysOnTop = true
    billboard.StudsOffset = Vector3.new(0, 4, 0)
    billboard.Parent = head

    -- 頭像
    local avatar = Instance.new("ImageLabel", billboard)
    avatar.Size = UDim2.new(0, 40, 0, 40)
    avatar.Position = UDim2.new(0, 0, 0, 0)
    avatar.BackgroundTransparency = 1
    avatar.Image = "https://www.roblox.com/headshot-thumbnail/image?userId="..otherP.UserId.."&width=150&height=150&format=png"

    -- 文字
    local text = Instance.new("TextLabel", billboard)
    text.Size = UDim2.new(1, -45, 1, 0)
    text.Position = UDim2.new(0, 45, 0, 0)
    text.BackgroundTransparency = 1
    text.RichText = true
    text.TextSize = 12
    text.Font = Enum.Font.GothamBold
    text.TextColor3 = Color3.new(1, 1, 1)
    text.TextXAlignment = Enum.TextXAlignment.Left

    espObjects[otherP] = { highlight = highlight, billboard = billboard, text = text }

    task.spawn(function()
        while espEnabled and otherP and otherP.Parent and espObjects[otherP] do
            if otherP.Character and otherP.Character:FindFirstChild("HumanoidRootPart") then
                local root = otherP.Character.HumanoidRootPart
                local lroot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                if root and lroot then
                    local dist = Round((root.Position - lroot.Position).Magnitude)
                    local hum = otherP.Character:FindFirstChildOfClass("Humanoid")
                    local hp = hum and Round(hum.Health) or 0
                    
                    text.Text = string.format(
                        "%s\n<font color='rgb(200,200,200)'>@%s</font>\n<font color='rgb(255,182,193)'>%d M</font> | HP: %d",
                        otherP.DisplayName, otherP.Name, dist, hp
                    )
                end
            else break end
            task.wait(0.2)
        end
    end)
end

local function toggleESP(state)
    espEnabled = state
    if state then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character then createESP(p) end
        end
    else
        for p, _ in pairs(espObjects) do removeESP(p) end
        espObjects = {}
    end
end

-- ==========================================
-- 在 Page 2 建立開關 UI
-- ==========================================
local espPage = pages[2] -- 指定放在第二頁

local espFrame = Instance.new("Frame", espPage)
espFrame.Size = UDim2.new(1, -5, 0, 40)
espFrame.BackgroundTransparency = 1

local espLabel = Instance.new("TextLabel", espFrame)
espLabel.Size = UDim2.new(1, -60, 1, 0)
espLabel.Text = "👁️ 玩家透視"
espLabel.TextColor3 = THEME.Text
espLabel.Font = Enum.Font.GothamBold
espLabel.TextSize = 14
espLabel.BackgroundTransparency = 1
espLabel.TextXAlignment = Enum.TextXAlignment.Left

local espBtn = Instance.new("TextButton", espFrame)
espBtn.Size = UDim2.new(0, 44, 0, 22)
espBtn.Position = UDim2.new(1, -50, 0.5, -11)
espBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
espBtn.Text = ""
Instance.new("UICorner", espBtn).CornerRadius = UDim.new(1, 0)

local espKnob = Instance.new("Frame", espBtn)
espKnob.Size = UDim2.new(0, 18, 0, 18)
espKnob.Position = UDim2.new(0, 2, 0.5, -9)
espKnob.BackgroundColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", espKnob).CornerRadius = UDim.new(1, 0)

espBtn.MouseButton1Click:Connect(function()
    _G.ESP_Enabled = not _G.ESP_Enabled
    local state = _G.ESP_Enabled
    
    -- 動畫效果
    TweenService:Create(espBtn, TweenInfo.new(0.2), {BackgroundColor3 = state and THEME.Main or Color3.fromRGB(70, 70, 80)}):Play()
    TweenService:Create(espKnob, TweenInfo.new(0.2), {Position = state and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)}):Play()
    
    -- 啟動/關閉邏輯
    toggleESP(state)
end)

-- 自動監聽新玩家
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function()
        if espEnabled then task.wait(1) createESP(p) end
    end)
end)

local function PureCleanupESP()
    -- 1. 停止舊腳本的循環開關
    _G.ESP_Enabled = false
    
    -- 2. 地毯式搜索所有玩家角色
    for _, p in pairs(game:GetService("Players"):GetPlayers()) do
        if p.Character then
            -- 使用 GetDescendants 才能抓到躲在 Head 裡的看板
            for _, child in pairs(p.Character:GetDescendants()) do
                -- 只要是 Highlight (框) 或 BillboardGui (文字/距離/頭像)
                if child:IsA("Highlight") or child:IsA("BillboardGui") then
                    -- 檢查名稱關鍵字，避免刪掉遊戲原生的 UI
                    local name = child.Name:lower()
                    if name:find("esp") or name:find("name") or name:find("ui") or name:find("highlight") then
                        child:Destroy()
                    end
                end
            end
        end
    end
end

-- 腳本一執行立刻跑一次清理
PureCleanupESP()

loadstring(game:HttpGet("https://raw.githubusercontent.com/ectom8810/lid12/refs/heads/main/obver"))()
loadstring(game:HttpGet("https://raw.githubusercontent.com/ectom8810/script/refs/heads/main/removeLava"))()
