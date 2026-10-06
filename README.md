-- ============================================================
-- [디스코드 웹훅 경고 및 강력한 닉네임 검증 시스템]
-- ============================================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- 영구 허용된 플레이어 목록 및 전용 키 매핑
local allowedPlayers = {
    ["zxxdaswo"] = "zxxdaswo_key.pro",
    ["yw62su"] = "yw62su_key_pro",
    ["5ee566"] = "5ee566_key_pro",
    ["dohunpoop"] = "dohunpoop_key_pro",
    ["jihoo215500_b"] = "jihoo215500_b_key_pro"
}

-- 디스코드 보안 경고 웹훅 주소 설정
local WEBHOOK_URL = "https://discord.com/api/webhooks/1556945712967716944/nLL6WGPq61Ob2aeqpP0LeQky87vL_abEBhJK20r80zdLUM3ujCO1IQRYGwIzdsYDggYR"

-- 허용되지 않은 유저일 경우 즉시 킥 및 웹훅 전송
if not allowedPlayers[LocalPlayer.Name] then
    pcall(function()
        local thumbUrl = string.format("https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=420&height=420&format=png", LocalPlayer.UserId)
        local data = {
            ["content"] = "🚨 **[AXR 보안 경고] 무단 접속자 차단됨**",
            ["embeds"] = {
                {
                    ["title"] = "⚠️ 허용되지 않은 유저 실행 감지",
                    ["description"] = "스크립트 권한이 없는 사용자가 실행을 시도하여 차단되었습니다.",
                    ["color"] = 16711680, -- 빨간색
                    ["fields"] = {
                        {
                            ["name"] = "👤 로블록스 닉네임",
                            ["value"] = LocalPlayer.DisplayName .. " (@" .. LocalPlayer.Name .. ")",
                            ["inline"] = true
                        },
                        {
                            ["name"] = "🆔 사용자 ID",
                            ["value"] = tostring(LocalPlayer.UserId),
                            ["inline"] = true
                        }
                    },
                    ["thumbnail"] = {
                        ["url"] = thumbUrl
                    },
                    ["footer"] = {
                        ["text"] = "AXR 포세이큰 보안 시스템"
                    },
                    ["timestamp"] = DateTime.now():ToIsoDate()
                }
            }
        }
        
        local encodedData = HttpService:JSONEncode(data)
        if syn and syn.request then
            syn.request({Url = WEBHOOK_URL, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        elseif http_request then
            http_request({Url = WEBHOOK_URL, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        elseif request then
            request({Url = WEBHOOK_URL, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        else
            HttpService:PostAsync(WEBHOOK_URL, encodedData)
        end
    end)
    
    LocalPlayer:Kick("[AXR 보안 시스템] 허용되지 않은 계정입니다. 스크립트를 사용할 수 없습니다.")
    return
end

-- 시간제 공용 키 설정 (10분)
local SHARED_TIME_KEY = "shared_time_key20"
local TIME_LIMIT_DURATION = 10 * 60 -- 10분 (초 단위)

local isPermanentUser = allowedPlayers[LocalPlayer.Name] ~= nil

-- ============================================================
-- [개인 블랙리스트 및 오입력 횟수 검증 로직]
-- ============================================================
local userIdStr = tostring(LocalPlayer.UserId)
local blacklistFileName = "AXR_Blacklist_" .. userIdStr .. ".txt"
local failCountFileName = "AXR_FailCount_" .. userIdStr .. ".txt"
local timeExpiryFileName = "AXR_TimeExpiry_" .. userIdStr .. ".txt"
local keyBlacklistFileName = "AXR_KeyBlacklist.txt"

local isBlacklisted = false

pcall(function()
    if isfile and isfile(blacklistFileName) then
        isBlacklisted = true
    end
end)

if isBlacklisted then
    LocalPlayer:Kick("[AXR 보안 시스템] 블랙리스트에 등록되어 스크립트를 사용할 수 없습니다.")
    return
end

local CoreGui = game:GetService("CoreGui")

local KeyGui = Instance.new("ScreenGui")
KeyGui.Name = "AXRKeySystem"
KeyGui.Parent = CoreGui
KeyGui.IgnoreGuiInset = true

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 260)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -130)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = KeyGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = isPermanentUser and "AXR 포세이큰 보안 인증 (영구)" or "AXR 포세이큰 공용 시간제 인증"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 18
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

local Subtitle = Instance.new("TextLabel")
Subtitle.Size = UDim2.new(1, 0, 0, 25)
Subtitle.Position = UDim2.new(0, 0, 0, 35)
Subtitle.BackgroundTransparency = 1
Subtitle.Text = "기종을 선택하고 보안 키를 입력해주세요."
Subtitle.TextColor3 = Color3.fromRGB(180, 180, 180)
Subtitle.TextSize = 13
Subtitle.Font = Enum.Font.SourceSans
Subtitle.Parent = MainFrame

local selectedPlatform = "PC"

local PcBtn = Instance.new("TextButton")
PcBtn.Size = UDim2.new(0, 145, 0, 30)
PcBtn.Position = UDim2.new(0, 25, 0, 65)
PcBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
PcBtn.Text = "💻 컴퓨터 (PC)"
PcBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
PcBtn.TextSize = 14
PcBtn.Font = Enum.Font.SourceSansBold
PcBtn.Parent = MainFrame
Instance.new("UICorner", PcBtn).CornerRadius = UDim.new(0, 6)

local MobileBtn = Instance.new("TextButton")
MobileBtn.Size = UDim2.new(0, 145, 0, 30)
MobileBtn.Position = UDim2.new(0, 180, 0, 65)
MobileBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MobileBtn.Text = "📱 모바일"
MobileBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
MobileBtn.TextSize = 14
MobileBtn.Font = Enum.Font.SourceSansBold
MobileBtn.Parent = MainFrame
Instance.new("UICorner", MobileBtn).CornerRadius = UDim.new(0, 6)

PcBtn.MouseButton1Click:Connect(function()
    selectedPlatform = "PC"
    PcBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    PcBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    MobileBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    MobileBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
end)

MobileBtn.MouseButton1Click:Connect(function()
    selectedPlatform = "Mobile"
    MobileBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    MobileBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    PcBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    PcBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
end)

local WarningLabel = Instance.new("TextLabel")
WarningLabel.Size = UDim2.new(0, 300, 0, 20)
WarningLabel.Position = UDim2.new(0.5, -150, 0, 105)
WarningLabel.BackgroundTransparency = 1
WarningLabel.Text = "⚠ 5번을 틀리면 블랙리스트에 올릅니다."
WarningLabel.TextColor3 = Color3.fromRGB(255, 170, 0)
WarningLabel.TextSize = 12
WarningLabel.Font = Enum.Font.SourceSansBold
WarningLabel.Parent = MainFrame

local TextBox = Instance.new("TextBox")
TextBox.Size = UDim2.new(0, 300, 0, 35)
TextBox.Position = UDim2.new(0.5, -150, 0, 128)
TextBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
TextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
TextBox.PlaceholderText = "여기에 키를 입력하세요..."
TextBox.Text = ""
TextBox.TextSize = 14
TextBox.Font = Enum.Font.SourceSans
TextBox.ClearTextOnFocus = false
TextBox.Parent = MainFrame
Instance.new("UICorner", TextBox).CornerRadius = UDim.new(0, 6)

local SubmitBtn = Instance.new("TextButton")
SubmitBtn.Size = UDim2.new(0, 300, 0, 35)
SubmitBtn.Position = UDim2.new(0.5, -150, 0, 173)
SubmitBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
SubmitBtn.Text = "인증 확인"
SubmitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SubmitBtn.TextSize = 15
SubmitBtn.Font = Enum.Font.SourceSansBold
SubmitBtn.Parent = MainFrame
Instance.new("UICorner", SubmitBtn).CornerRadius = UDim.new(0, 6)

local NoticeLabel = Instance.new("TextLabel")
NoticeLabel.Size = UDim2.new(1, 0, 0, 20)
NoticeLabel.Position = UDim2.new(0, 0, 0, 213)
NoticeLabel.BackgroundTransparency = 1
NoticeLabel.Text = ""
NoticeLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
NoticeLabel.TextSize = 12
NoticeLabel.Font = Enum.Font.SourceSans
NoticeLabel.Parent = MainFrame

local authenticated = false
local isUsingSharedTimeKey = false

local isKeyBlacklisted = false
pcall(function()
    if isfile and isfile(keyBlacklistFileName) then
        local content = readfile(keyBlacklistFileName)
        if content:find(SHARED_TIME_KEY) then
            isKeyBlacklisted = true
        end
    end
end)

if isKeyBlacklisted and not isPermanentUser then
    KeyGui:Destroy()
    LocalPlayer:Kick("[AXR 보안 시스템] 해당 공용 시간제 키는 기간이 만료되어 사용할 수 없습니다.")
    return
end

local existingExpiryTime = nil
pcall(function()
    if isfile and isfile(timeExpiryFileName) then
        local content = readfile(timeExpiryFileName)
        existingExpiryTime = tonumber(content)
    end
end)

if existingExpiryTime and not isPermanentUser then
    if os.time() >= existingExpiryTime then
        pcall(function()
            if writefile then writefile(keyBlacklistFileName, SHARED_TIME_KEY) end
        end)
        KeyGui:Destroy()
        LocalPlayer:Kick("[AXR 보안 시스템] 공용 시간제 키 시간이 만료되어 키가 차단되었습니다.")
        return
    else
        isUsingSharedTimeKey = true
    end
end

local function getFailCount()
    local count = 0
    pcall(function()
        if isfile and isfile(failCountFileName) then
            count = tonumber(readfile(failCountFileName)) or 0
        end
    end)
    return count
end

local function addFailCount()
    local current = getFailCount() + 1
    pcall(function()
        if writefile then writefile(failCountFileName, tostring(current)) end
    end)
    return current
end

SubmitBtn.MouseButton1Click:Connect(function()
    local inputKey = TextBox.Text
    
    if isPermanentUser and inputKey == allowedPlayers[LocalPlayer.Name] then
        authenticated = true
        KeyGui:Destroy()
    elseif inputKey == SHARED_TIME_KEY then
        if isKeyBlacklisted then
            NoticeLabel.Text = "만료된 공용 키입니다."
            return
        end
        
        isUsingSharedTimeKey = true
        authenticated = true
        
        if not existingExpiryTime then
            pcall(function()
                if writefile then
                    local expiry = os.time() + TIME_LIMIT_DURATION
                    writefile(timeExpiryFileName, tostring(expiry))
                end
            end)
        end
        
        NoticeLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
        NoticeLabel.Text = "시간제 키 사용 됨! 로딩 중..."
        task.wait(0.5)
        KeyGui:Destroy()
    else
        TextBox.Text = ""
        local fails = addFailCount()
        if fails >= 5 then
            pcall(function()
                if writefile then writefile(blacklistFileName, "BLACKLISTED_WRONG_KEY") end
            end)
            KeyGui:Destroy()
            LocalPlayer:Kick("[AXR 보안 시스템] 키를 5회 이상 틀려 블랙리스트에 등록되었습니다.")
        else
            NoticeLabel.Text = string.format("키가 틀렸습니다! (오입력: %d/5회)", fails)
        end
    end
end)

repeat task.wait() until authenticated

-- ============================================================
-- [공용 시간제 사용자 만료 관리 루프]
-- ============================================================
if isUsingSharedTimeKey then
    task.spawn(function()
        local targetExpiryTime = existingExpiryTime
        if not targetExpiryTime then
            targetExpiryTime = os.time() + TIME_LIMIT_DURATION
        end
        
        local timerGui = Instance.new("ScreenGui")
        timerGui.Name = "AXRTimeLimitGui"
        timerGui.Parent = CoreGui
        timerGui.IgnoreGuiInset = true
        
        local timerLabel = Instance.new("TextLabel")
        timerLabel.Size = UDim2.new(0, 250, 0, 35)
        timerLabel.Position = UDim2.new(0.5, -125, 0, 10)
        timerLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        timerLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        timerLabel.TextSize = 14
        timerLabel.Font = Enum.Font.SourceSansBold
        timerLabel.Parent = timerGui
        Instance.new("UICorner", timerLabel).CornerRadius = UDim.new(0, 6)
        
        while true do
            local leftTime = targetExpiryTime - os.time()
            if leftTime <= 0 then
                timerLabel.Text = "⚠️ [AXR] 공용 시간제 키 기간 만료됨!"
                
                pcall(function()
                    if writefile then
                        writefile(keyBlacklistFileName, SHARED_TIME_KEY)
                    end
                end)
                
                task.wait(1)
                LocalPlayer:Kick("[AXR 보안 시스템] 공용 시간제 키(10분)가 만료되어 해당 키가 차단되었습니다.")
                break
            else
                local mins = math.floor(leftTime / 60)
                local secs = math.floor(leftTime % 60)
                timerLabel.Text = string.format("⏳ [공용 시간제] 남은 시간: %02d분 %02d초", mins, secs)
            end
            task.wait(1)
        end
    end)
end

-- ============================================================
-- [Rayfield UI 및 메인 스크립트 로드]
-- ============================================================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

local C = {
    SpeedEnabled = false, CurrentSpeed = 37,
    JumpEnabled = false, CurrentJumpPower = 65,
    FlyEnabled = false, FlySpeed = 50,
    NoclipEnabled = false,
    Keys = {W = false, A = false, S = false, D = false, Space = false, Shift = false},
    EspEnabled = false,
    GeneratorEspEnabled = false,
    GeneratorWirePreviewEnabled = false, -- 발전기 줄 미리보기 기능 상태 추가
    AimbotEnabled = false,
    AimbotRadius = 150,
    HitboxExpandEnabled = false,
}

local BODY_GYRO_NAME = "AXRForsakenGyro"
local BODY_VELOCITY_NAME = "AXRForsakenVelocity"
local PlayerEspFolder = "AXRForsakenPlayerEsp"
local GenEspFolder = "AXRForsakenGeneratorEsp"
local GenWirePreviewFolder = "AXRForsakenGenWirePreview"
local AimbotGuiFolder = "AXRForsakenAimbotGui"

local generatorHighlights = {}
local wirePreviewHighlights = {}
local isAutoClearing = false
local aimbotCircle = nil
local lockedAimbotTarget = nil

local MainWindow = Rayfield:CreateWindow({
   Name = "AXR 포세이큰 스크립트 (" .. selectedPlatform .. " 모드)",
   LoadingTitle = "AXR 포세이큰 로딩 중...",
   LoadingSubtitle = "by zxxdaswo, yw62su, 5ee566, dohunpoop, jihoo215500_b",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "AXRForsakenHub",
      FileName = "AXRForsakenConfig"
   },
   KeySystem = false,
})

task.spawn(function()
   task.wait(1)
   Rayfield:Notify({
      Title = "⚠️ [안내] 권장 설정 및 주의사항",
      Content = "스피드 ~37 / 점프력 ~65 권장\n플라이 및 노클립 사용 자제",
      Duration = 6,
      Image = 4483362458,
   })
end)

local MainTab = MainWindow:CreateTab("메인 기능", 4483362458)
local ParticipantTab = MainWindow:CreateTab("참가자 전용", 4483362458)
local HunterTab = MainWindow:CreateTab("술래 전용", 4483362458)
local InquiryTab = MainWindow:CreateTab("문의", 4483362458)

-- ============================================================
-- [MainTab 내용: 메인 기능]
-- ============================================================
MainTab:CreateSection("스피드 설정 (권장: 37)")
MainTab:CreateToggle({
   Name = "스피드 ON/OFF", CurrentValue = C.SpeedEnabled,
   Callback = function(Value) C.SpeedEnabled = Value end,
})
MainTab:CreateSlider({
   Name = "이동 속도 조절 (권장 37)", Range = {16, 250}, Increment = 1, CurrentValue = C.CurrentSpeed,
   Callback = function(Value) C.CurrentSpeed = Value end,
})

MainTab:CreateSection("점프력 설정 (권장: 65)")
MainTab:CreateToggle({
   Name = "점프력 ON/OFF", CurrentValue = C.JumpEnabled,
   Callback = function(Value)
      C.JumpEnabled = Value
      local char = LocalPlayer.Character
      if char and char:FindFirstChild("Humanoid") then
         if not Value then
            char.Humanoid.UseJumpPower = true
            char.Humanoid.JumpPower = 50
         end
      end
   end,
})
MainTab:CreateSlider({
   Name = "점프력 조절 (권장 65)", Range = {50, 300}, Increment = 5, CurrentValue = C.CurrentJumpPower,
   Callback = function(Value) C.CurrentJumpPower = Value end,
})

MainTab:CreateSection("플라이 설정 [사용 금지]")
MainTab:CreateToggle({
   Name = "플라이 ON/OFF (사용 금지)", CurrentValue = C.FlyEnabled,
   Callback = function(Value)
      if Value then
         Rayfield:Notify({
            Title = "🚨 경고",
            Content = "플라이 기능은 정지 위험이 있으므로 사용하지 않는 것을 권장합니다!",
            Duration = 3,
            Image = 4483362458,
         })
      end
      C.FlyEnabled = Value
      local char = LocalPlayer.Character
      if char and char:FindFirstChild("HumanoidRootPart") then
         local rootPart = char.HumanoidRootPart
         if Value then
            if not rootPart:FindFirstChild(BODY_GYRO_NAME) then
               local bg = Instance.new("BodyGyro") bg.Name = BODY_GYRO_NAME
               bg.P = 9e4 bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
               bg.CFrame = rootPart.CFrame bg.Parent = rootPart
            end
            if not rootPart:FindFirstChild(BODY_VELOCITY_NAME) then
               local bv = Instance.new("BodyVelocity") bv.Name = BODY_VELOCITY_NAME
               bv.Velocity = Vector3.new(0, 0, 0) bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
               bv.Parent = rootPart
            end
         else
            local bg, bv = rootPart:FindFirstChild(BODY_GYRO_NAME), rootPart:FindFirstChild(BODY_VELOCITY_NAME)
            if bg then bg:Destroy() end if bv then bv:Destroy() end
         end
      end
   end,
})
MainTab:CreateSlider({
   Name = "플라이 속도 조절", Range = {10, 300}, Increment = 5, CurrentValue = C.FlySpeed,
   Callback = function(Value) C.FlySpeed = Value end,
})

MainTab:CreateSection("노클립 설정 [사용 자제]")
MainTab:CreateToggle({
   Name = "노클립(벽 통과) ON/OFF [사용 자제]", CurrentValue = C.NoclipEnabled,
   Callback = function(Value)
      if Value then
         Rayfield:Notify({
            Title = "⚠️ 주의",
            Content = "노클립 기능은 정지 위험이 있으므로 가급적 사용을 자제해주세요!",
            Duration = 3,
            Image = 4483362458,
         })
      end
      C.NoclipEnabled = Value
   end,
})

MainTab:CreateSection("ESP 설정")
MainTab:CreateToggle({
   Name = "플레이어 ESP ON/OFF", CurrentValue = C.EspEnabled,
   Callback = function(Value)
      C.EspEnabled = Value
      local container = CoreGui:FindFirstChild(PlayerEspFolder)
      if Value then
         if not container then
            container = Instance.new("ScreenGui", CoreGui)
            container.Name = PlayerEspFolder
         end
         
         local function applyEsp(targetPlayer)
            if targetPlayer == LocalPlayer then return end
            local function setup()
               if not targetPlayer.Character then return end
               for _, v in ipairs(container:GetChildren()) do
                  if v.Name == targetPlayer.Name .. "_EspObj" then v:Destroy() end
               end
               
               local folder = Instance.new("Folder")
               folder.Name = targetPlayer.Name .. "_EspObj"
               folder.Parent = container
               
               local hl = Instance.new("Highlight")
               hl.Adornee = targetPlayer.Character
               hl.FillColor = Color3.fromRGB(0, 255, 200)
               hl.OutlineColor = Color3.fromRGB(0, 0, 0)
               hl.FillTransparency = 0.5
               hl.OutlineTransparency = 0
               hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
               hl.Parent = folder
               
               local head = targetPlayer.Character:FindFirstChild("Head")
               if head then
                  local bill = Instance.new("BillboardGui")
                  bill.Size = UDim2.new(0, 200, 0, 40)
                  bill.StudsOffset = Vector3.new(0, 2.5, 0)
                  bill.AlwaysOnTop = true
                  bill.Parent = folder
                  bill.Adornee = head
                  
                  local textLabel = Instance.new("TextLabel")
                  textLabel.Size = UDim2.new(1, 0, 1, 0)
                  textLabel.BackgroundTransparency = 1
                  textLabel.Font = Enum.Font.SourceSansBold
                  textLabel.TextSize = 14
                  textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                  textLabel.TextStrokeTransparency = 0
                  textLabel.Text = targetPlayer.DisplayName .. " (@" .. targetPlayer.Name .. ")"
                  textLabel.Parent = bill
               end
            end
            
            targetPlayer.CharacterAdded:Connect(function()
               task.wait(1)
               if C.EspEnabled then setup() end
            end)
            setup()
         end
         
         for _, p in ipairs(Players:GetPlayers()) do applyEsp(p) end
         Players.PlayerAdded:Connect(applyEsp)
      else
         if container then container:Destroy() end
      end
   end,
})

-- ============================================================
-- [ParticipantTab 내용: 참가자 전용]
-- ============================================================
ParticipantTab:CreateSection("참가자 전용 발전기 기능")

ParticipantTab:CreateSection("발전기 ESP")
ParticipantTab:CreateToggle({
   Name = "발전기 ESP ON/OFF", CurrentValue = C.GeneratorEspEnabled,
   Callback = function(Value)
      C.GeneratorEspEnabled = Value
      local container = CoreGui:FindFirstChild(GenEspFolder)
      if Value then
         if not container then
            container = Instance.new("ScreenGui", CoreGui)
            container.Name = GenEspFolder
         end
         
         generatorHighlights = {}
         local scannedModels = {}
         
         for _, obj in ipairs(Workspace:GetDescendants()) do
            local nameLower = obj.Name:lower()
            if nameLower == "generator" or nameLower == "발전기" or nameLower:find("generator") or nameLower:find("발전기") then
               local model = obj:IsA("Model") and obj or obj:FindFirstAncestorOfClass("Model") or obj
               local parentPath = model:GetFullName():lower()
               
               if not parentPath:find("lobby") and not parentPath:find("spawn") and not parentPath:find("waiting") then
                  if model and not scannedModels[model] then
                     local okToAdd = true
                     if model:IsA("Model") then
                        local size = model:GetExtentsSize()
                        if size.X > 80 or size.Z > 80 then okToAdd = false end
                     end
                     
                     if okToAdd then
                        scannedModels[model] = true
                        local hl = Instance.new("Highlight")
                        hl.Name = "GenHighlight"
                        hl.Adornee = model
                        hl.FillColor = Color3.fromRGB(255, 170, 0)
                        hl.OutlineColor = Color3.fromRGB(0, 0, 0)
                        hl.FillTransparency = 0.4
                        hl.OutlineTransparency = 0
                        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        hl.Parent = container
                        
                        table.insert(generatorHighlights, {model = model, highlight = hl})
                     end
                  end
               end
            end
         end
      else
         if container then container:Destroy() end
         generatorHighlights = {}
      end
   end,
})

ParticipantTab:CreateSection("자동 발전기 & 퍼즐 연타 시스템")

local function toggleAutoClear(state)
   isAutoClearing = state
   if isAutoClearing then
      task.spawn(function()
         while isAutoClearing and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") do
            for _, obj in ipairs(Workspace:GetDescendants()) do
               local nameLower = obj.Name:lower()
               if nameLower:find("generator") or nameLower:find("발전기") or nameLower:find("puzzle") or nameLower:find("퍼즐") then
                  for _, child in ipairs(obj:GetDescendants()) do
                     if child:IsA("RemoteEvent") then
                        pcall(function() child:FireServer(true) end)
                     elseif child:IsA("RemoteFunction") then
                        pcall(function() child:InvokeServer(true) end)
                     end
                  end
               end
            end
            
            local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
            if playerGui then
               for _, gui in ipairs(playerGui:GetDescendants()) do
                  if gui:IsA("RemoteEvent") then
                     pcall(function() gui:FireServer(true) end)
                  end
               end
            end
            
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
               local rootPart = char.HumanoidRootPart
               for _, prompt in ipairs(Workspace:GetDescendants()) do
                  if prompt:IsA("ProximityPrompt") then
                     local parentPart = prompt.Parent
                     if parentPart and parentPart:IsA("BasePart") and (parentPart.Position - rootPart.Position).Magnitude <= 15 then
                        pcall(function() fireproximityprompt(prompt) end)
                     end
                  end
               end
            end
            
            if selectedPlatform == "PC" then
               VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
               task.wait(0.02)
               VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
               task.wait(0.03)
            else
               task.wait(0.05)
            end
         end
      end)
      
      Rayfield:Notify({
         Title = "AXR 포세이큰",
         Content = "자동 발전기 및 퍼즐 연타가 시작되었습니다! (" .. selectedPlatform .. ")",
         Duration = 1.5,
         Image = 4483362458,
      })
   else
      Rayfield:Notify({
         Title = "AXR 포세이큰",
         Content = "자동 발전기 연타가 중지되었습니다.",
         Duration = 1.5,
         Image = 4483362458,
      })
   end
end

ParticipantTab:CreateButton({
   Name = "자동 발전기 & 퍼즐 연타 ON/OFF (토글)",
   Callback = function()
      toggleAutoClear(not isAutoClearing)
   end,
})

-- [요청하신 기능 추가] 자동 발전기 클리어 버튼 바로 아래에 배치
ParticipantTab:CreateToggle({
   Name = "발전기 줄 미리보기 ON/OFF (불투명도 조절)",
   CurrentValue = C.GeneratorWirePreviewEnabled,
   Callback = function(Value)
      C.GeneratorWirePreviewEnabled = Value
      local container = CoreGui:FindFirstChild(GenWirePreviewFolder)
      
      if Value then
         if not container then
            container = Instance.new("Folder", CoreGui)
            container.Name = GenWirePreviewFolder
         end
         
         wirePreviewHighlights = {}
         local scannedWires = {}
         
         for _, obj in ipairs(Workspace:GetDescendants()) do
            local nameLower = obj.Name:lower()
            -- 발전기 전기선, 케이블, 와이어 등을 탐색
            if nameLower:find("wire") or nameLower:find("cable") or nameLower:find("line") or nameLower:find("전선") or nameLower:find("줄") then
               if obj:IsA("BasePart") and not scannedWires[obj] then
                  scannedWires[obj] = true
                  
                  -- 기존 속성 백업 및 불투명도 적용 (Highlight 또는 투명도 조절)
                  local hl = Instance.new("Highlight")
                  hl.Adornee = obj
                  hl.FillColor = Color3.fromRGB(0, 170, 255)
                  hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                  hl.FillTransparency = 0.3 -- 불투명도 조절
                  hl.OutlineTransparency = 0.1
                  hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                  hl.Parent = container
                  
                  table.insert(wirePreviewHighlights, {part = obj, highlight = hl})
               end
            end
         end
         
         Rayfield:Notify({
            Title = "AXR 포세이큰",
            Content = "발전기 줄 미리보기가 활성화되었습니다.",
            Duration = 1.5,
            Image = 4483362458,
         })
      else
         if container then container:Destroy() end
         wirePreviewHighlights = {}
         
         Rayfield:Notify({
            Title = "AXR 포세이큰",
            Content = "발전기 줄 미리보기가 비활성화되었습니다.",
            Duration = 1.5,
            Image = 4483362458,
         })
      end
   end,
})

ParticipantTab:CreateSection("발전기 텔레포트 [가급적 사용 자제]")

local generatorOptions = {"발전기 스캔 중..."}
local generatorInstances = {}
local selectedGeneratorOption = nil

local function scanGenerators()
   generatorOptions = {}
   generatorInstances = {}
   local scannedModels = {}
   local scannedPositions = {}
   local count = 1
   
   for _, obj in ipairs(Workspace:GetDescendants()) do
      local nameLower = obj.Name:lower()
      if nameLower == "generator" or nameLower == "발전기" or nameLower:find("generator") or nameLower:find("발전기") then
         local model = obj:IsA("Model") and obj or obj:FindFirstAncestorOfClass("Model") or obj
         local parentPath = model:GetFullName():lower()
         
         if not parentPath:find("lobby") and not parentPath:find("spawn") and not parentPath:find("waiting") then
            if model and not scannedModels[model] then
               local okToAdd = true
               if model:IsA("Model") then
                  local size = model:GetExtentsSize()
                  if size.X > 80 or size.Z > 80 then okToAdd = false end
               end
               
               if okToAdd then
                  local interactionPart = nil
                  local promptObj = nil
                  for _, desc in ipairs(model:GetDescendants()) do
                     if desc:IsA("ProximityPrompt") and desc.Parent and desc.Parent:IsA("BasePart") then
                        interactionPart = desc.Parent
                        promptObj = desc
                        break
                     end
                  end
                  
                  if not interactionPart then
                     for _, desc in ipairs(model:GetDescendants()) do
                        if desc:IsA("BasePart") then
                           local dName = desc.Name:lower()
                           if dName:find("interact") or dName:find("touch") or dName:find("hitbox") or dName:find("prompt") then
                              interactionPart = desc
                              break
                           end
                        end
                     end
                  end
                  
                  local targetPart = interactionPart or model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
                  if targetPart then
                     local pos = targetPart.Position
                     
                     local isDuplicate = false
                     for _, savedPos in ipairs(scannedPositions) do
                        if (Vector3.new(pos.X, 0, pos.Z) - Vector3.new(savedPos.X, 0, savedPos.Z)).Magnitude < 5 then
                           isDuplicate = true
                           break
                        end
                     end
                     
                     if not isDuplicate then
                        scannedModels[model] = true
                        table.insert(scannedPositions, pos)
                        
                        local optionName = string.format("발전기 #%d (X:%.0f, Y:%.0f, Z:%.0f)", count, pos.X, pos.Y, pos.Z)
                        table.insert(generatorOptions, optionName)
                        generatorInstances[optionName] = {part = targetPart, model = model, prompt = promptObj}
                        count = count + 1
                     end
                  end
               end
            end
         end
      end
   end
   
   if #generatorOptions == 0 then
      table.insert(generatorOptions, "발전기를 찾을 수 없음")
   end
end

scanGenerators()

local GeneratorDropdown = ParticipantTab:CreateDropdown({
   Name = "발전기 선택",
   Options = generatorOptions,
   CurrentOption = generatorOptions[1],
   MultipleOptions = false,
   Flag = "GenTeleportDropdown",
   Callback = function(Option)
      selectedGeneratorOption = type(Option) == "table" and Option[1] or Option
   end,
})
selectedGeneratorOption = generatorOptions[1]

ParticipantTab:CreateButton({
   Name = "선택한 발전기로 텔레포트 [사용 자제]",
   Callback = function()
      Rayfield:Notify({
         Title = "⚠️ 주의",
         Content = "텔레포트는 웬만하면 사용하지 않는 것을 권장합니다!",
         Duration = 2,
         Image = 4483362458,
      })
      local genData = generatorInstances[selectedGeneratorOption]
      if genData and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
         local rootPart = LocalPlayer.Character.HumanoidRootPart
         
         local targetPos = genData.part.Position
         local offsetPos = targetPos + (genData.part.CFrame.LookVector * 3)
         rootPart.CFrame = CFrame.new(Vector3.new(offsetPos.X, targetPos.Y, offsetPos.Z), targetPos)
         rootPart.Velocity = Vector3.new(0, 0, 0)
         
         for _, item in ipairs(generatorHighlights) do
            if item.model == genData.model then
               if item.highlight and item.highlight.Parent then
                  item.highlight.FillColor = Color3.fromRGB(0, 255, 0)
               end
            end
         end
      else
         Rayfield:Notify({
            Title = "AXR 포세이큰",
            Content = "유효한 발전기를 선택해주세요!",
            Duration = 2,
            Image = 4483362458,
         })
      end
   end,
})

ParticipantTab:CreateButton({
   Name = "발전기 목록 새로고침",
   Callback = function()
      scanGenerators()
      GeneratorDropdown:Refresh(generatorOptions, true)
      Rayfield:Notify({
         Title = "AXR 포세이큰",
         Content = "발전기 목록을 최신화했습니다!",
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

-- ============================================================
-- [HunterTab 내용: 술래 전용]
-- ============================================================
HunterTab:CreateSection("술래 전용 전투 기능")

HunterTab:CreateToggle({
   Name = "플레이어 에임봇 ON/OFF", CurrentValue = C.AimbotEnabled,
   Callback = function(Value)
      C.AimbotEnabled = Value
      lockedAimbotTarget = nil
      local container = CoreGui:FindFirstChild(AimbotGuiFolder)
      if Value then
         if not container then
            container = Instance.new("ScreenGui", CoreGui)
            container.Name = AimbotGuiFolder
         end
         
         if not aimbotCircle then
            aimbotCircle = Instance.new("Frame")
            aimbotCircle.Name = "AimbotCircle"
            aimbotCircle.BackgroundTransparency = 1
            aimbotCircle.AnchorPoint = Vector2.new(0.5, 0.5)
            aimbotCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
            aimbotCircle.Size = UDim2.new(0, C.AimbotRadius * 2, 0, C.AimbotRadius * 2)
            aimbotCircle.Parent = container
            
            local corner = Instance.new("UICorner")
            corner.CornerRadius = UDim.new(1, 0)
            corner.Parent = aimbotCircle
            
            local stroke = Instance.new("UIStroke")
            stroke.Color = Color3.fromRGB(255, 255, 255)
            stroke.Thickness = 1.5
            stroke.Parent = aimbotCircle
         end
      else
         if container then container:Destroy() end
         aimbotCircle = nil
      end
      
      Rayfield:Notify({
         Title = "AXR 포세이큰 술래",
         Content = "플레이어 에임봇이 " .. (Value and "활성화" or "비활성화") .. " 되었습니다.",
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

HunterTab:CreateSlider({
   Name = "에임봇 원 크기(반경) 조절",
   Range = {50, 400},
   Increment = 5,
   CurrentValue = C.AimbotRadius,
   Callback = function(Value)
      C.AimbotRadius = Value
      if aimbotCircle then
         aimbotCircle.Size = UDim2.new(0, Value * 2, 0, Value * 2)
      end
   end,
})

HunterTab:CreateToggle({
   Name = "평타 및 스킬 히트박스 최대화 ON/OFF", CurrentValue = C.HitboxExpandEnabled,
   Callback = function(Value)
      C.HitboxExpandEnabled = Value
      Rayfield:Notify({
         Title = "AXR 포세이큰 술래",
         Content = "히트박스 확장 기능이 " .. (Value and "활성화" or "비활성화") .. " 되었습니다.",
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

HunterTab:CreateSection("플레이어 텔레포트 [가급적 사용 자제]")

local playerOptions = {"플레이어 스캔 중..."}
local playerInstances = {}
local selectedPlayerOption = nil

local function scanPlayers()
   playerOptions = {}
   playerInstances = {}
   
   for _, p in ipairs(Players:GetPlayers()) do
      if p ~= LocalPlayer then
         local pName = p.DisplayName .. " (@" .. p.Name .. ")"
         table.insert(playerOptions, pName)
         playerInstances[pName] = p
      end
   end
   
   if #playerOptions == 0 then
      table.insert(playerOptions, "다른 플레이어가 없음")
   end
end

scanPlayers()

Players.PlayerAdded:Connect(scanPlayers)
Players.PlayerRemoving:Connect(scanPlayers)

local PlayerDropdown = HunterTab:CreateDropdown({
   Name = "플레이어 선택",
   Options = playerOptions,
   CurrentOption = playerOptions[1],
   MultipleOptions = false,
   Flag = "PlayerTeleportDropdown",
   Callback = function(Option)
      selectedPlayerOption = type(Option) == "table" and Option[1] or Option
   end,
})
selectedPlayerOption = playerOptions[1]

HunterTab:CreateButton({
   Name = "선택한 플레이어로 텔레포트 [사용 자제]",
   Callback = function()
      Rayfield:Notify({
         Title = "⚠️ 주의",
         Content = "텔레포트는 웬만하면 사용하지 않는 것을 권장합니다!",
         Duration = 2,
         Image = 4483362458,
      })
      local targetPlayer = playerInstances[selectedPlayerOption]
      if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
         local targetPart = targetPlayer.Character.HumanoidRootPart
         if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = targetPart.CFrame + Vector3.new(0, 3, 0)
         end
      else
         Rayfield:Notify({
            Title = "AXR 포세이큰 술래",
            Content = "유효한 플레이어를 선택해주세요!",
            Duration = 2,
            Image = 4483362458,
         })
      end
   end,
})

HunterTab:CreateButton({
   Name = "플레이어 목록 새로고침",
   Callback = function()
      scanPlayers()
      PlayerDropdown:Refresh(playerOptions, true)
      Rayfield:Notify({
         Title = "AXR 포세이큰 술래",
         Content = "플레이어 목록을 갱신했습니다!",
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

-- ============================================================
-- [InquiryTab 내용: 문의 카테고리 (새로운 문의 웹훅 적용)]
-- ============================================================
InquiryTab:CreateSection("개발자에게 문의하기")
InquiryTab:CreateSection("⚠️ 주의: 장난 및 도배성 문의는 개발자에게 실시간 알림이 가므로 자제해 주세요!")

local inquiryMessage = ""
local discordNameInput = ""

InquiryTab:CreateInput({
   Name = "문의 내용 입력",
   PlaceholderText = "개발자에게 전달할 메시지를 입력하세요...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      inquiryMessage = Text
   end,
})

InquiryTab:CreateInput({
   Name = "디스코드 표시 닉네임 입력",
   PlaceholderText = "예: 0000#0 또는 본인 디스코드 닉네임",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      discordNameInput = Text
   end,
})

InquiryTab:CreateButton({
   Name = "문의 내용 보내기",
   Callback = function()
      if inquiryMessage == "" or inquiryMessage:gsub("%s+", "") == "" then
         Rayfield:Notify({
            Title = "⚠️ 오류",
            Content = "보낼 문의 내용을 입력해주세요!",
            Duration = 2,
            Image = 4483362458,
         })
         return
      end

      local finalDiscordName = (discordNameInput == "" or discordNameInput:gsub("%s+", "") == "") and "입력 안 함" or discordNameInput

      -- 문의 전용 웹훅 주소 적용
      local inquiryWebhookUrl = "https://discord.com/api/webhooks/1556961378953330762/G1saYuhrhmJsYRbeJJwAKrG_A7KRWXJeBpTul7ouEpNJhPWJZEr_VV20qb4aeepcDuDT"
      local thumbUrl = string.format("https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=420&height=420&format=png", LocalPlayer.UserId)
      
      local data = {
         ["content"] = "📬 **[AXR 포세이큰 새로운 문의 도착]**",
         ["embeds"] = {
            {
               ["title"] = "💬 유저 문의 내용",
               ["description"] = inquiryMessage,
               ["color"] = 3447003,
               ["fields"] = {
                  {
                     ["name"] = "💬 디스코드 표시 닉네임",
                     ["value"] = finalDiscordName,
                     ["inline"] = false
                  },
                  {
                     ["name"] = "👤 로블록스 닉네임",
                     ["value"] = LocalPlayer.DisplayName .. " (@" .. LocalPlayer.Name .. ")",
                     ["inline"] = true
                  },
                  {
                     ["name"] = "🆔 사용자 ID",
                     ["value"] = tostring(LocalPlayer.UserId),
                     ["inline"] = true
                  },
                  {
                     ["name"] = "💻 사용 모드",
                     ["value"] = selectedPlatform,
                     ["inline"] = true
                  }
               },
               ["thumbnail"] = {
                  ["url"] = thumbUrl
               },
               ["footer"] = {
                  ["text"] = "AXR 포세이큰 자동 문의 시스템"
               },
               ["timestamp"] = DateTime.now():ToIsoDate()
            }
         }
      }

      pcall(function()
         local encodedData = HttpService:JSONEncode(data)
         if syn and syn.request then
            syn.request({Url = inquiryWebhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
         elseif http_request then
            http_request({Url = inquiryWebhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
         elseif request then
            request({Url = inquiryWebhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
         else
            HttpService:PostAsync(inquiryWebhookUrl, encodedData)
         end
      end)

      Rayfield:Notify({
         Title = "✅ 전송 완료",
         Content = "개발자에게 문의 메시지가 성공적으로 전송되었습니다!",
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

-- ============================================================
-- [키 입력 및 물리 연산 / 루프]
-- ============================================================
UserInputService.InputBegan:Connect(function(input, gp)
   if gp then return end
   
   if input.KeyCode == Enum.KeyCode.W then C.Keys.W = true
   elseif input.KeyCode == Enum.KeyCode.A then C.Keys.A = true
   elseif input.KeyCode == Enum.KeyCode.S then C.Keys.S = true
   elseif input.KeyCode == Enum.KeyCode.D then C.Keys.D = true
   elseif input.KeyCode == Enum.KeyCode.Space then C.Keys.Space = true
   elseif input.KeyCode == Enum.KeyCode.LeftShift then C.Keys.Shift = true end
end)

UserInputService.InputEnded:Connect(function(input)
   if input.KeyCode == Enum.KeyCode.W then C.Keys.W = false
   elseif input.KeyCode == Enum.KeyCode.A then C.Keys.A = false
   elseif input.KeyCode == Enum.KeyCode.S then C.Keys.S = false
   elseif input.KeyCode == Enum.KeyCode.D then C.Keys.D = false
   elseif input.KeyCode == Enum.KeyCode.Space then C.Keys.Space = false
   elseif input.KeyCode == Enum.KeyCode.LeftShift then C.Keys.Shift = false end
end)

RunService.RenderStepped:Connect(function()
   local char = LocalPlayer.Character
   if not char then return end
   
   local hum = char:FindFirstChild("Humanoid")
   local rootPart = char:FindFirstChild("HumanoidRootPart")
   
   if C.HitboxExpandEnabled then
      for _, tool in ipairs(char:GetChildren()) do
         if tool:IsA("Tool") then
            for _, part in ipairs(tool:GetDescendants()) do
               if part:IsA("BasePart") then
                  part.Size = Vector3.new(15, 15, 15)
                  part.Transparency = 0.8
                  part.CanCollide = false
               end
            end
         end
      end
   end

   if C.AimbotEnabled then
      local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
      
      local isTargetValid = false
      if lockedAimbotTarget and lockedAimbotTarget.Character then
         local targetHum = lockedAimbotTarget.Character:FindFirstChild("Humanoid")
         local targetPart = lockedAimbotTarget.Character:FindFirstChild("HumanoidRootPart") or lockedAimbotTarget.Character:FindFirstChild("LowerTorso")
         if targetHum and targetHum.Health > 0 and targetPart then
            isTargetValid = true
         end
      end
      
      if not isTargetValid then
         lockedAimbotTarget = nil
         local shortestDistance = math.huge
         
         for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
               local humanoid = p.Character:FindFirstChild("Humanoid")
               local targetPart = p.Character:FindFirstChild("HumanoidRootPart") or p.Character:FindFirstChild("LowerTorso")
               
               if humanoid and humanoid.Health > 0 and targetPart then
                  local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                  if onScreen then
                     local screenPos2D = Vector2.new(screenPos.X, screenPos.Y)
                     local distFromCenter = (screenPos2D - screenCenter).Magnitude
                     
                     if distFromCenter <= C.AimbotRadius and distFromCenter < shortestDistance then
                        shortestDistance = distFromCenter
                        lockedAimbotTarget = p
                     end
                  end
               end
            end
         end
      end
      
      if lockedAimbotTarget and lockedAimbotTarget.Character then
         local targetPart = lockedAimbotTarget.Character:FindFirstChild("HumanoidRootPart") or lockedAimbotTarget.Character:FindFirstChild("LowerTorsu") or lockedAimbotTarget.Character:FindFirstChild("LowerTorso")
         if targetPart then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
         end
      end
   else
      lockedAimbotTarget = nil
   end
   
   if C.NoclipEnabled then
      for _, part in ipairs(char:GetDescendants()) do
         if part:IsA("BasePart") and part.CanCollide then
            part.CanCollide = false
         end
      end
   end

   if hum and C.JumpEnabled then 
      hum.UseJumpPower = true 
      hum.JumpPower = C.CurrentJumpPower 
   end

   if C.SpeedEnabled and rootPart and hum then
      local moveDir = hum.MoveDirection
      if moveDir.Magnitude > 0 then
         local currentVelocity = rootPart.Velocity
         rootPart.Velocity = Vector3.new(moveDir.X * C.CurrentSpeed, currentVelocity.Y, moveDir.Z * C.CurrentSpeed)
      end
   end

   if C.FlyEnabled and rootPart then
      local bg = rootPart:FindFirstChild(BODY_GYRO_NAME)
      local bv = rootPart:FindFirstChild(BODY_VELOCITY_NAME)
      
      if bg and bv then
         bg.CFrame = Camera.CFrame
         local moveDir = Vector3.new(0, 0, 0)
         if C.Keys.W then moveDir += Camera.CFrame.LookVector end
         if C.Keys.S then moveDir -= Camera.CFrame.LookVector end
         if C.Keys.A then moveDir -= Camera.CFrame.RightVector end
         if C.Keys.D then moveDir += Camera.CFrame.RightVector end
         if C.Keys.Space then moveDir += Vector3.new(0, 1, 0) end
         if C.Keys.Shift then moveDir -= Vector3.new(0, 1, 0) end
         
         if moveDir.Magnitude > 0 then
            bv.Velocity = moveDir.Unit * C.FlySpeed
         else
            bv.Velocity = Vector3.new(0, 0.01, 0)
         end
      end
   end
end)
