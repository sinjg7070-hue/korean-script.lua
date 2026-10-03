-- ============================================================
-- [디스코드 웹훅 경고 및 강력한 닉네임 검증 시스템]
-- ============================================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- 허용된 플레이어 목록 및 전용 키 매핑
local allowedPlayers = {
    ["zxxdaswo"] = "zxxdaswo_key.pro",
    ["yw62su"] = "yw62su_key_pro",
    ["5ee566"] = "5ee566_key_pro"
}

-- 지정된 플레이어가 아닐 경우
if not allowedPlayers[LocalPlayer.Name] then
    local thumbUrl = string.format("https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=420&height=420&format=png", LocalPlayer.UserId)
    local webhookUrl = "https://discord.com/api/webhooks/1554402747841773622/up3pj44KILozThMY1klzJfXbl6ED8-U9MFa6Sur3KUsTNLu8oFal2joOAIUi4pLUfWhE"
    
    local data = {
        ["content"] = "@here **[AXR 보안 시스템 경고]** 허용되지 않은 사용자가 스크립트 실행을 시도했습니다!",
        ["embeds"] = {
            {
                ["title"] = "🚨 무단 실행 차단 및 경고 발생",
                ["color"] = 16711680,
                ["fields"] = {
                    {
                        ["name"] = "👤 표시 닉네임 (Display Name)",
                        ["value"] = LocalPlayer.DisplayName,
                        ["inline"] = true
                    },
                    {
                        ["name"] = "🆔 진짜 닉네임 (Username)",
                        ["value"] = "@" .. LocalPlayer.Name,
                        ["inline"] = true
                    },
                    {
                        ["name"] = "🔢 고유 ID (User ID)",
                        ["value"] = tostring(LocalPlayer.UserId),
                        ["inline"] = true
                    }
                },
                ["thumbnail"] = {
                    ["url"] = thumbUrl
                },
                ["image"] = {
                    ["url"] = thumbUrl
                },
                ["footer"] = {
                    ["text"] = "AXR 보안 자동화 시스템 • Target: zxxdaswo, yw62su, 5ee566"
                },
                ["timestamp"] = DateTime.now():ToIsoDate()
            }
        }
    }

    pcall(function()
        local encodedData = HttpService:JSONEncode(data)
        if syn and syn.request then
            syn.request({Url = webhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        elseif http_request then
            http_request({Url = webhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        elseif request then
            request({Url = webhookUrl, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = encodedData})
        else
            HttpService:PostAsync(webhookUrl, encodedData)
        end
    end)

    LocalPlayer:Kick("[AXR 보안 시스템] 허용되지 않은 사용자입니다.")
    return
end

local CoreGui = game:GetService("CoreGui")

local KeyGui = Instance.new("ScreenGui")
KeyGui.Name = "AXRKeySystem"
KeyGui.Parent = CoreGui
KeyGui.IgnoreGuiInset = true

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 240)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -120)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = KeyGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "AXR 포세이큰 보안 인증"
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

-- 모바일 / 컴퓨터 선택 버튼
local selectedPlatform = "PC" -- 기본값 PC

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

local TextBox = Instance.new("TextBox")
TextBox.Size = UDim2.new(0, 300, 0, 35)
TextBox.Position = UDim2.new(0.5, -150, 0, 110)
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
SubmitBtn.Position = UDim2.new(0.5, -150, 0, 155)
SubmitBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
SubmitBtn.Text = "인증 확인"
SubmitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SubmitBtn.TextSize = 15
SubmitBtn.Font = Enum.Font.SourceSansBold
SubmitBtn.Parent = MainFrame
Instance.new("UICorner", SubmitBtn).CornerRadius = UDim.new(0, 6)

local NoticeLabel = Instance.new("TextLabel")
NoticeLabel.Size = UDim2.new(1, 0, 0, 20)
NoticeLabel.Position = UDim2.new(0, 0, 0, 195)
NoticeLabel.BackgroundTransparency = 1
NoticeLabel.Text = ""
NoticeLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
NoticeLabel.TextSize = 12
NoticeLabel.Font = Enum.Font.SourceSans
NoticeLabel.Parent = MainFrame

local authenticated = false

SubmitBtn.MouseButton1Click:Connect(function()
    local correctKey = allowedPlayers[LocalPlayer.Name]
    if TextBox.Text == correctKey then
        authenticated = true
        KeyGui:Destroy()
    else
        TextBox.Text = ""
        NoticeLabel.Text = "틀렸습니다! 다시 입력하세요."
    end
end)

repeat task.wait() until authenticated

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
    AimbotEnabled = false,
    AimbotRadius = 150,
    HitboxExpandEnabled = false,
}

local BODY_GYRO_NAME = "AXRForsakenGyro"
local BODY_VELOCITY_NAME = "AXRForsakenVelocity"
local PlayerEspFolder = "AXRForsakenPlayerEsp"
local GenEspFolder = "AXRForsakenGeneratorEsp"
local AimbotGuiFolder = "AXRForsakenAimbotGui"

local generatorHighlights = {}
local isAutoClearing = false
local aimbotCircle = nil
local lockedAimbotTarget = nil

local MainWindow = Rayfield:CreateWindow({
   Name = "AXR 포세이큰 스크립트 (" .. selectedPlatform .. " 모드)",
   LoadingTitle = "AXR 포세이큰 로딩 중...",
   LoadingSubtitle = "by zxxdaswo & yw62su & 5ee566",
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
      Content = "스피드 ~37 / 점프력 ~65 권장\n플라이 사용 금지 / 텔레포트 자제",
      Duration = 6,
      Image = 4483362458,
   })
end)

local MainTab = MainWindow:CreateTab("메인 기능", 4483362458)
local ParticipantTab = MainWindow:CreateTab("참가자 전용", 4483362458)
local HunterTab = MainWindow:CreateTab("술래 전용", 4483362458)

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

MainTab:CreateSection("노클립 설정")
MainTab:CreateToggle({
   Name = "노클립(벽 통과) ON/OFF", CurrentValue = C.NoclipEnabled,
   Callback = function(Value) C.NoclipEnabled = Value end,
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
            
            -- PC 모드일 때만 가상 키보드(F키) 입력 실행 (모바일은 제외하여 굳음 현상 방지)
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
         local targetPart = lockedAimbotTarget.Character:FindFirstChild("HumanoidRootPart") or lockedAimbotTarget.Character:FindFirstChild("LowerTorso")
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
