-- 안전한 서비스 불러오기
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()
local mouse = localPlayer:GetMouse()

local playerGui = localPlayer:WaitForChild("PlayerGui", 5) or localPlayer:FindFirstChildOfClass("PlayerGui")

-- 기존 GUI 제거 (중복 방지)
pcall(function()
    if CoreGui:FindFirstChild("WordGameHelperUI") then
        CoreGui.WordGameHelperUI:Destroy()
    end
    if playerGui and playerGui:FindFirstChild("WordGameHelperUI") then
        playerGui.WordGameHelperUI:Destroy()
    end
end)

-- ScreenGui 생성
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "WordGameHelperUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true

local success = pcall(function()
    if syn and syn.protect_gui then
        syn.protect_gui(screenGui)
        screenGui.Parent = CoreGui
    elseif gethui then
        screenGui.Parent = gethui()
    else
        screenGui.Parent = CoreGui
    end
end)

if not success or not screenGui.Parent then
    screenGui.Parent = playerGui
end

-- ==========================================
-- [유저별 맞춤형 키 및 프리미엄 키 시스템 설정]
-- ==========================================
local userKeys = {
    ["dambii522"] = "no.1keyap191929",
    ["zxxdaswo"] = "no.1keyap19293949",
    ["1CasaNova6974"] = "no.1keyap172737",
    ["dohunpoop"] = "dohunpoop_key12",
    ["yfsm_31"] = "yfsm_31.key199"
}

local premiumKeys = {
    ["zxxdaswo"] = "zxxdaswo.key.pro"
}

_G.WordHelperAuthenticated = _G.WordHelperAuthenticated or false
_G.WordHelperPremiumAuthenticated = _G.WordHelperPremiumAuthenticated or false

local function checkSavedAuth()
    return _G.WordHelperAuthenticated
end

local function checkSavedPremiumAuthenticated()
    return _G.WordHelperPremiumAuthenticated
end

-- ==========================================
-- [메인 헬퍼 UI 생성]
-- ==========================================
local titleFrame = Instance.new("TextButton")
titleFrame.Name = "TitleFrame"
titleFrame.Size = UDim2.new(0, 240, 0, 75)
titleFrame.Position = UDim2.new(0.73, 0, 0.1, 0)
titleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleFrame.TextColor3 = Color3.fromRGB(255, 255, 255)
titleFrame.TextSize = 15
titleFrame.Font = Enum.Font.SourceSansBold
titleFrame.Text = "단어 맞히기 헬퍼"
titleFrame.TextYAlignment = Enum.TextYAlignment.Top
titleFrame.AutoButtonColor = false
titleFrame.Visible = checkSavedAuth() or checkSavedPremiumAuthenticated()
titleFrame.Parent = screenGui

local uiCornerBtn = Instance.new("UICorner")
uiCornerBtn.CornerRadius = UDim.new(0, 8)
uiCornerBtn.Parent = titleFrame

local devLabel = Instance.new("TextLabel")
devLabel.Name = "DevLabel"
devLabel.Size = UDim2.new(1, 0, 0, 20)
devLabel.Position = UDim2.new(0, 0, 0, 22)
devLabel.BackgroundTransparency = 1
devLabel.TextColor3 = Color3.fromRGB(170, 170, 170)
devLabel.TextSize = 12
devLabel.Font = Enum.Font.SourceSansItalic
devLabel.Text = "스크립트 개발자 : 지환"
devLabel.Parent = titleFrame

local autoAnswerEnabled = false
local autoBtn = Instance.new("TextButton")
autoBtn.Name = "AutoAnswerButton"
autoBtn.Size = UDim2.new(0, 85, 0, 24)
autoBtn.Position = UDim2.new(1, -90, 0, 45)
autoBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
autoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
autoBtn.TextSize = 11
autoBtn.Font = Enum.Font.SourceSansBold
autoBtn.Text = "자동정답: OFF"
autoBtn.Parent = titleFrame

local uiCornerAuto = Instance.new("UICorner")
uiCornerAuto.CornerRadius = UDim.new(0, 5)
uiCornerAuto.Parent = autoBtn

autoBtn.MouseButton1Click:Connect(function()
    autoAnswerEnabled = not autoAnswerEnabled
    if autoAnswerEnabled then
        autoBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
        autoBtn.Text = "자동정답: ON"
    else
        autoBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        autoBtn.Text = "자동정답: OFF"
    end
end)

local answerLabel = Instance.new("TextLabel")
answerLabel.Name = "AnswerLabel"
answerLabel.Size = UDim2.new(0, 240, 0, 45)
answerLabel.Position = UDim2.new(0, 0, 1, 5)
answerLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
answerLabel.BackgroundTransparency = 0.2
answerLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
answerLabel.TextSize = 18
answerLabel.Font = Enum.Font.SourceSansBold
answerLabel.Text = "정답: 라운드 대기 중..."
answerLabel.Visible = true
answerLabel.Parent = titleFrame

local uiCornerLbl = Instance.new("UICorner")
uiCornerLbl.CornerRadius = UDim.new(0, 8)
uiCornerLbl.Parent = answerLabel

local resetKeyBtn = Instance.new("TextButton")
resetKeyBtn.Name = "ResetKeyButton"
resetKeyBtn.Size = UDim2.new(0, 240, 0, 30)
resetKeyBtn.Position = UDim2.new(0, 0, 1, 10)
resetKeyBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
resetKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
resetKeyBtn.TextSize = 13
resetKeyBtn.Font = Enum.Font.SourceSansBold
resetKeyBtn.Text = "키 시스템 초기화"
resetKeyBtn.Visible = true
resetKeyBtn.Parent = answerLabel

local uiCornerReset = Instance.new("UICorner")
uiCornerReset.CornerRadius = UDim.new(0, 6)
uiCornerReset.Parent = resetKeyBtn

local destroyScriptBtn = Instance.new("TextButton")
destroyScriptBtn.Name = "DestroyScriptButton"
destroyScriptBtn.Size = UDim2.new(0, 240, 0, 30)
destroyScriptBtn.Position = UDim2.new(0, 0, 1, 8)
destroyScriptBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
destroyScriptBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
destroyScriptBtn.TextSize = 13
destroyScriptBtn.Font = Enum.Font.SourceSansBold
destroyScriptBtn.Text = "스크립트 삭제"
destroyScriptBtn.Visible = true
destroyScriptBtn.Parent = resetKeyBtn

local uiCornerDestroy = Instance.new("UICorner")
uiCornerDestroy.CornerRadius = UDim.new(0, 6)
uiCornerDestroy.Parent = destroyScriptBtn

destroyScriptBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if screenGui then
            screenGui:Destroy()
        end
    end)
end)

-- ==========================================
-- [프리미엄 전용: 트롤링 전송 UI 및 토글 시스템]
-- ==========================================
local remoteInputBox = Instance.new("TextBox")
remoteInputBox.Name = "RemoteInputBox"
remoteInputBox.Size = UDim2.new(0, 150, 0, 30)
remoteInputBox.Position = UDim2.new(0, 0, 1, 8)
remoteInputBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
remoteInputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
remoteInputBox.PlaceholderColor3 = Color3.fromRGB(160, 160, 160)
remoteInputBox.PlaceholderText = "이상한 가짜 답 입력..."
remoteInputBox.TextSize = 12
remoteInputBox.Font = Enum.Font.SourceSansBold
remoteInputBox.Text = ""
remoteInputBox.Visible = checkSavedPremiumAuthenticated()
remoteInputBox.Parent = destroyScriptBtn

local uiCornerRemote = Instance.new("UICorner")
uiCornerRemote.CornerRadius = UDim.new(0, 6)
uiCornerRemote.Parent = remoteInputBox

local targetToggleBtn = Instance.new("TextButton")
targetToggleBtn.Name = "TargetToggleBtn"
targetToggleBtn.Size = UDim2.new(0, 85, 0, 30)
targetToggleBtn.Position = UDim2.new(1, 5, 0, 0)
targetToggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
targetToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
targetToggleBtn.TextSize = 12
targetToggleBtn.Font = Enum.Font.SourceSansBold
targetToggleBtn.Text = "전송: OFF"
targetToggleBtn.Parent = remoteInputBox

local uiCornerToggle = Instance.new("UICorner")
uiCornerToggle.CornerRadius = UDim.new(0, 6)
uiCornerToggle.Parent = targetToggleBtn

local function updatePremiumUIVisibility(isVisible)
    remoteInputBox.Visible = isVisible
end

-- ==========================================
-- [순수 클라이언트 통신망 설정]
-- ==========================================
local remoteFolderName = "WordHelperSyncNetwork"
local syncFolder = ReplicatedStorage:FindFirstChild(remoteFolderName)
if not syncFolder then
    pcall(function()
        syncFolder = Instance.new("Folder")
        syncFolder.Name = remoteFolderName
        syncFolder.Parent = ReplicatedStorage
    end)
end

local remoteEvent = syncFolder:FindFirstChild("RemoteWordEvent")
if not remoteEvent then
    pcall(function()
        remoteEvent = Instance.new("UnreliableRemoteEvent")
        remoteEvent.Name = "RemoteWordEvent"
        remoteEvent.Parent = syncFolder
    end)
end

local targetPlayer = nil
local targetToggleOn = false
local scriptUsers = {}

targetToggleBtn.MouseButton1Click:Connect(function()
    targetToggleOn = not targetToggleOn
    if targetToggleOn then
        targetToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
        targetToggleBtn.Text = "전송: ON"
    else
        targetToggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        targetToggleBtn.Text = "전송: OFF"
        targetPlayer = nil
    end
end)

-- 주기적으로 내가 스크립트를 사용 중임을 브로드캐스트
task.spawn(function()
    while true do
        task.wait(2)
        pcall(function()
            if remoteEvent then
                remoteEvent:FireServer("PING", localPlayer.Name)
            end
        end)
    end
end)

-- 플레이어 클릭하여 타겟 지정 (전송 ON일 때)
mouse.Button1Down:Connect(function()
    if not targetToggleOn or not checkSavedPremiumAuthenticated() then return end
    local hitTarget = mouse.Target
    if hitTarget and hitTarget.Parent then
        local character = hitTarget.Parent
        local p = Players:GetPlayerFromCharacter(character)
        if not p then
            character = character.Parent
            p = Players:GetPlayerFromCharacter(character)
        end
        if p and p ~= localPlayer then
            targetPlayer = p
            pcall(function()
                for _, otherP in ipairs(Players:GetPlayers()) do
                    if otherP.Character and otherP.Character:FindFirstChild("HumanoidRootPart") then
                        local hl = otherP.Character:FindFirstChild("WordHelperHighlight")
                        if hl then hl:Destroy() end
                    end
                end
                if p.Character then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "WordHelperHighlight"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.Parent = p.Character
                end
            end)
        end
    end
end)

-- 머리 위에 스크립트 사용 여부 표시기 관리
local function updateScriptUserBillboard(p, isUsing)
    if not p.Character then return end
    local head = p.Character:FindFirstChild("Head")
    if not head then return end
    
    local guiName = "WordScriptStatusTag"
    local billboard = head:FindFirstChild(guiName)
    if not billboard then
        billboard = Instance.new("BillboardGui")
        billboard.Name = guiName
        billboard.Size = UDim2.new(0, 130, 0, 30)
        billboard.StudsOffset = Vector3.new(0, 2.5, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = head
        
        local textLbl = Instance.new("TextLabel")
        textLbl.Name = "StatusText"
        textLbl.Size = UDim2.new(1, 0, 1, 0)
        textLbl.BackgroundTransparency = 1
        textLbl.TextSize = 13
        textLbl.Font = Enum.Font.SourceSansBold
        textLbl.Parent = billboard
    end
    
    local txtLabel = billboard:FindFirstChild("StatusText")
    if txtLabel then
        if isUsing then
            txtLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
            txtLabel.Text = "[스크립트 사용 중]"
        else
            txtLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
            txtLabel.Text = "[미사용]"
        end
    end
end

-- 프리미엄 입력창에서 엔터 쳤을 때 타겟에게 가짜 정답 발사
remoteInputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed and checkSavedPremiumAuthenticated() then
        local fakeWord = remoteInputBox.Text:gsub("^%s*(.-)%s*$", "%1")
        if fakeWord ~= "" and targetPlayer and remoteEvent then
            pcall(function()
                remoteEvent:FireServer("TROLL", targetPlayer.Name, fakeWord)
            end)
            remoteInputBox.Text = ""
            remoteInputBox.PlaceholderText = "[전송 완료!] 타겟 조작됨"
            task.delay(1.5, function()
                if remoteInputBox and remoteInputBox.Parent then
                    remoteInputBox.PlaceholderText = "이상한 가짜 답 입력..."
                end
            end)
        end
    end
end)

-- 네트워크 수신 이벤트 처리
if remoteEvent then
    remoteEvent.OnClientEvent:Connect(function(actionType, p1, p2)
        if actionType == "PING" and p1 then
            scriptUsers[p1] = true
            local p = Players:FindFirstChild(p1)
            if p then updateScriptUserBillboard(p, true) end
        elseif actionType == "TROLL" and p1 == localPlayer.Name and p2 then
            answerLabel.Text = "정답: " .. p2
            triggerAutoInput(p2)
        end
    end)
end

task.spawn(function()
    while true do
        task.wait(2)
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= localPlayer then
                local isUsing = scriptUsers[p.Name] or false
                updateScriptUserBillboard(p, isUsing)
            end
        end
    end
end)

-- ==========================================
-- [자동 정답 입력 로직 (개선됨)]
-- ==========================================
function triggerAutoInput(word)
    if not autoAnswerEnabled then return end
    pcall(function()
        local targetBox = nil
        local focusedGui = UserInputService:GetFocusedTextBox()
        if focusedGui and focusedGui:IsA("TextBox") then
            targetBox = focusedGui
        else
            if playerGui then
                for _, descendant in ipairs(playerGui:GetDescendants()) do
                    if descendant:IsA("TextBox") and descendant.Visible and descendant.AbsoluteSize.X > 0 then
                        local phText = (descendant.PlaceholderText or ""):lower()
                        local txt = (descendant.Text or ""):lower()
                        if phText:find("입력") or phText:find("단어") or phText:find("여기에") or 
                           txt:find("입력") or txt:find("단어") or txt:find("여기에") then
                            targetBox = descendant
                            break
                        end
                    end
                end
            end
            if not targetBox and playerGui then
                for _, descendant in ipairs(playerGui:GetDescendants()) do
                    if descendant:IsA("TextBox") and descendant.Visible and descendant.AbsoluteSize.X > 50 then
                        targetBox = descendant
                        break
                    end
                end
            end
        end

        if targetBox then
            targetBox.Text = word
            task.spawn(function()
                targetBox:CaptureFocus()
                task.wait(0.05)
                if VirtualInputManager then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Return, false, game)
                    task.wait(0.03)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Return, false, game)
                end
            end)
        end
    end)
end

-- ==========================================
-- [키 인증 프레임 생성 함수]
-- ==========================================
local function createKeySystemUI()
    local keyFrame = Instance.new("Frame")
    keyFrame.Name = "KeySystemFrame"
    keyFrame.Size = UDim2.new(0, 300, 0, 260)
    keyFrame.Position = UDim2.new(0.5, -150, 0.4, -130)
    keyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    keyFrame.BorderSizePixel = 0
    keyFrame.Visible = true
    keyFrame.Parent = screenGui

    local uiCornerKey = Instance.new("UICorner")
    uiCornerKey.CornerRadius = UDim.new(0, 10)
    uiCornerKey.Parent = keyFrame

    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 35)
    keyTitle.BackgroundTransparency = 1
    keyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyTitle.TextSize = 16
    keyTitle.Font = Enum.Font.SourceSansBold
    keyTitle.Text = "단어 헬퍼 전용 인증"
    keyTitle.Parent = keyFrame

    local keyBox = Instance.new("TextBox")
    keyBox.Size = UDim2.new(0, 260, 0, 32)
    keyBox.Position = UDim2.new(0.5, -130, 0, 38)
    keyBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    keyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    keyBox.PlaceholderText = "비밀 키를 입력하세요..."
    keyBox.TextSize = 13
    keyBox.Font = Enum.Font.SourceSans
    keyBox.Text = ""
    keyBox.Parent = keyFrame

    local uiCornerBox = Instance.new("UICorner")
    uiCornerBox.CornerRadius = UDim.new(0, 6)
    uiCornerBox.Parent = keyBox

    local submitBtn = Instance.new("TextButton")
    submitBtn.Size = UDim2.new(0, 260, 0, 30)
    submitBtn.Position = UDim2.new(0.5, -130, 0, 76)
    submitBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    submitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitBtn.TextSize = 13
    submitBtn.Font = Enum.Font.SourceSansBold
    submitBtn.Text = "인증하기"
    submitBtn.Parent = keyFrame

    local uiCornerSub = Instance.new("UICorner")
    uiCornerSub.CornerRadius = UDim.new(0, 6)
    uiCornerSub.Parent = submitBtn

    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0, 260, 0, 28)
    buyBtn.Position = UDim2.new(0.5, -130, 0, 112)
    buyBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    buyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyBtn.TextSize = 12
    buyBtn.Font = Enum.Font.SourceSansBold
    buyBtn.Text = "전용 키 구매"
    buyBtn.Parent = keyFrame

    local uiCornerBuy = Instance.new("UICorner")
    uiCornerBuy.CornerRadius = UDim.new(0, 6)
    uiCornerBuy.Parent = buyBtn

    local buyProBtn = Instance.new("TextButton")
    buyProBtn.Size = UDim2.new(0, 260, 0, 28)
    buyProBtn.Position = UDim2.new(0.5, -130, 0, 146)
    buyProBtn.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
    buyProBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyProBtn.TextSize = 12
    buyProBtn.Font = Enum.Font.SourceSansBold
    buyProBtn.Text = "프리미엄 전용 키 구매"
    buyProBtn.Parent = keyFrame

    local uiCornerBuyPro = Instance.new("UICorner")
    uiCornerBuyPro.CornerRadius = UDim.new(0, 6)
    uiCornerBuyPro.Parent = buyProBtn

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 25)
    statusLabel.Position = UDim2.new(0, 0, 0, 180)
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLabel.TextSize = 12
    statusLabel.Font = Enum.Font.SourceSansItalic
    statusLabel.Text = ""
    statusLabel.Parent = keyFrame

    local function copyDiscordLink()
        local discordLink = "https://discord.gg/ZKenYVezV"
        pcall(function()
            if setclipboard then
                setclipboard(discordLink)
            elseif toclipboard then
                toclipboard(discordLink)
            end
        end)
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLabel.Text = "디스코드 링크가 복사되었습니다!"
    end

    buyBtn.MouseButton1Click:Connect(copyDiscordLink)
    buyProBtn.MouseButton1Click:Connect(copyDiscordLink)

    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        if premiumKeys[playerName] and premiumKeys[playerName] == enteredKey then
            _G.WordHelperPremiumAuthenticated = true
            _G.WordHelperAuthenticated = true
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "프리미엄 인증 성공!"
            task.wait(0.8)
            keyFrame:Destroy()
            titleFrame.Visible = true
            updatePremiumUIVisibility(true)
        elseif userKeys[playerName] and userKeys[playerName] == enteredKey then
            _G.WordHelperAuthenticated = true
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "인증 성공!"
            task.wait(0.8)
            keyFrame:Destroy()
            titleFrame.Visible = true
            updatePremiumUIVisibility(false)
        else
            statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLabel.Text = "권한이 없거나 잘못된 키입니다."
        end
    end)
end

if not titleFrame.Visible then
    createKeySystemUI()
end

resetKeyBtn.MouseButton1Click:Connect(function()
    _G.WordHelperAuthenticated = false
    _G.WordHelperPremiumAuthenticated = false
    titleFrame.Visible = false
    updatePremiumUIVisibility(false)
    createKeySystemUI()
end)

-- ==========================================
-- [마우스 드래그 이동 로직]
-- ==========================================
local dragging = false
local dragStart, startPos

titleFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = titleFrame.Position
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

titleFrame.MouseButton1Up:Connect(function()
    dragging = false
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        titleFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- ==========================================
-- [단어 검증 및 정답 추출 로직 (중복 및 버그 수정)]
-- ==========================================
local currentAnswer = ""

local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if txt:find("%s") then return false end
    if #txt < 2 or #txt > 12 then return false end
    if tonumber(txt) ~= nil then return false end
    
    local lowerTxt = txt:lower()
    if lowerTxt:match("^cl") or lowerTxt:match("^gui") or lowerTxt:match("^rem") or lowerTxt:match("^http") then
        return false
    end
    
    if lowerTxt:match("^[a-z]+$") then
        return false
    end
    
    return true
end

local function checkRoundReset(txt)
    if not txt or type(txt) ~= "string" then return false end
    local low = txt:lower()
    if low:find("대기") or low:find("라운드") or low:find("시작") or low:find("끝") or low:find("종료") or low:find("ready") or low:find("wait") or low:find("end") or low:find("over") or low:find("finish") or low:find("win") then
        return true
    end
    return false
end

local function processValue(txt)
    if not txt or type(txt) ~= "string" then return end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if checkRoundReset(txt) then
        if currentAnswer ~= "RESET" then
            currentAnswer = "RESET" -- 라운드 변경 시 기존 단어 기억 초기화!
            answerLabel.Text = "정답: 라운드 대기 중..."
        end
    elseif isValidWord(txt) then
        if txt ~= currentAnswer then
            currentAnswer = txt
            answerLabel.Text = "정답: " .. txt
            triggerAutoInput(txt)
        end
    end
end

pcall(function()
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        if (v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent")) and v.Name ~= "RemoteWordEvent" then
            v.OnClientEvent:Connect(function(...)
                local args = {...}
                for _, arg in ipairs(args) do
                    if type(arg) == "string" then
                        processValue(arg)
                    elseif type(arg) == "table" then
                        for _, subArg in pairs(arg) do
                            if type(subArg) == "string" then
                                processValue(subArg)
                            end
                        end
                    end
                end
            end)
        end
    end
end)

pcall(function()
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("StringValue") then
            processValue(obj.Value)
            obj.Changed:Connect(function(val)
                processValue(val)
            end)
        end
    end
    
    ReplicatedStorage.DescendantAdded:Connect(function(obj)
        if obj:IsA("StringValue") then
            obj.Changed:Connect(function(val)
                processValue(val)
            end)
        end
    end)
end)
