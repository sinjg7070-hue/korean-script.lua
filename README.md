-- ==========================================
-- [단어 맞히기 헬퍼 - 마스터 키 2단계 인증 생략 버전]
-- ==========================================
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

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
-- [유저별 맞춤형 키 및 마스터 키 설정]
-- ==========================================
local masterKey = "MASTER_KEY_2026" -- 마스터 키 (누구나 2단계 없이 사용 가능)

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

-- [프리미엄 전용] 자동 정답 버튼
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
autoBtn.Visible = checkSavedPremiumAuthenticated()
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

local function updatePremiumUIVisibility(isVisible)
    autoBtn.Visible = isVisible
end

-- ==========================================
-- [자동 정답 입력 로직]
-- ==========================================
function triggerAutoInput(word)
    if not checkSavedPremiumAuthenticated() or not autoAnswerEnabled then return end
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
                        if phText:find("입력") or phText:find("단어") or phText:find("여기에") or phText:find("chat") or
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
                task.wait(0.04)
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
-- [게임 정답 감지 및 원본 추출 로직]
-- ==========================================
local currentAnswer = ""

local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if txt:find("#") or txt:find("_") then return false end
    if txt:find("%s") then return false end
    if #txt < 2 or #txt > 20 then return false end
    
    if tonumber(txt) ~= nil or txt:match("%d") then 
        return false 
    end
    
    local lowerTxt = txt:lower()
    
    if lowerTxt == "total" or lowerTxt:find("total") or lowerTxt == "설정" or lowerTxt == "옵션" or lowerTxt == "메뉴" or lowerTxt == "상점" or lowerTxt == "정보" or lowerTxt == "선택됨" then
        return false
    end
    
    if lowerTxt:match("^cl") or lowerTxt:match("^gui") or lowerTxt:match("^rem") or lowerTxt:match("^http") then
        return false
    end
    
    if txt:match("[a-zA-Z]") then
        return false
    end
    
    return true
end

local function checkRoundReset(txt)
    if not txt or type(txt) ~= "string" then return false end
    local low = txt:lower()
    if low:find("대기") or low:find("라운드") or low:find("시작") or low:find("끝") or low:find("종료") or low:find("ready") or low:find("wait") or low:find("end") or low:find("over") or low:find("finish") then
        return true
    end
    return false
end

local function processValue(txt)
    if not txt or type(txt) ~= "string" then return end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if checkRoundReset(txt) then
        if currentAnswer ~= "RESET" then
            currentAnswer = "RESET"
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
    local function hookEvent(v)
        if v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent") then
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

    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        hookEvent(v)
    end
    ReplicatedStorage.DescendantAdded:Connect(hookEvent)
end)

pcall(function()
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("StringValue") or obj:IsA("TextValue") then
            processValue(obj.Value)
            obj.Changed:Connect(processValue)
        end
    end
    
    ReplicatedStorage.DescendantAdded:Connect(function(obj)
        if obj:IsA("StringValue") or obj:IsA("TextValue") then
            obj.Changed:Connect(processValue)
        end
    end)
end)

-- ==========================================
-- [인증창(1단계) 및 드래그 UI 시스템]
-- ==========================================
local function createSecondStepUI(isPremium)
    -- 일반 키는 2단계 인증 진행
    local secondFrame = Instance.new("Frame")
    secondFrame.Size = UDim2.new(0, 320, 0, 250)
    secondFrame.Position = UDim2.new(0.5, -160, 0.4, -125)
    secondFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    secondFrame.BorderSizePixel = 0
    secondFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = secondFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.SourceSansBold
    title.Text = "2단계 본인 확인 인증"
    title.Parent = secondFrame

    local usernameBox = Instance.new("TextBox")
    usernameBox.Size = UDim2.new(0, 280, 0, 32)
    usernameBox.Position = UDim2.new(0.5, -140, 0, 50)
    usernameBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    usernameBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    usernameBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    usernameBox.PlaceholderText = "실제 닉네임 (Username) 입력..."
    usernameBox.TextSize = 13
    usernameBox.Parent = secondFrame

    local corner1 = Instance.new("UICorner")
    corner1.CornerRadius = UDim.new(0, 6)
    corner1.Parent = usernameBox

    local displayBox = Instance.new("TextBox")
    displayBox.Size = UDim2.new(0, 280, 0, 32)
    displayBox.Position = UDim2.new(0.5, -140, 0, 92)
    displayBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    displayBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    displayBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    displayBox.PlaceholderText = "표시 닉네임 (Display Name) 입력..."
    displayBox.TextSize = 13
    displayBox.Parent = secondFrame

    local corner2 = Instance.new("UICorner")
    corner2.CornerRadius = UDim.new(0, 6)
    corner2.Parent = displayBox

    local statusLbl = Instance.new("TextLabel")
    statusLbl.Size = UDim2.new(1, 0, 0, 25)
    statusLbl.Position = UDim2.new(0, 0, 0, 135)
    statusLbl.BackgroundTransparency = 1
    statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLbl.TextSize = 12
    statusLbl.Font = Enum.Font.SourceSansItalic
    statusLbl.Text = "본인의 계정 정보를 정확히 입력해주세요."
    statusLbl.Parent = secondFrame

    local confirmBtn = Instance.new("TextButton")
    confirmBtn.Size = UDim2.new(0, 280, 0, 35)
    confirmBtn.Position = UDim2.new(0.5, -140, 0, 175)
    confirmBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
    confirmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    confirmBtn.TextSize = 14
    confirmBtn.Font = Enum.Font.SourceSansBold
    confirmBtn.Text = "최종 인증 완료"
    confirmBtn.Parent = secondFrame

    local cornerBtn = Instance.new("UICorner")
    cornerBtn.CornerRadius = UDim.new(0, 6)
    cornerBtn.Parent = confirmBtn

    confirmBtn.MouseButton1Click:Connect(function()
        local enteredUser = usernameBox.Text:gsub("^%s*(.-)%s*$", "%1")
        local enteredDisplay = displayBox.Text:gsub("^%s*(.-)%s*$", "%1")

        if enteredUser == localPlayer.Name and enteredDisplay == localPlayer.DisplayName then
            if isPremium then
                _G.WordHelperPremiumAuthenticated = true
            end
            _G.WordHelperAuthenticated = true
            statusLbl.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLbl.Text = "2단계 인증 성공! 환영합니다."
            task.wait(0.8)
            secondFrame:Destroy()
            titleFrame.Visible = true
            updatePremiumUIVisibility(isPremium)
        else
            statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLbl.Text = "실제 닉네임 또는 표시 닉네임이 일치하지 않습니다."
        end
    end)
end

local function createKeySystemUI()
    local keyFrame = Instance.new("Frame")
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
            if setclipboard then setclipboard(discordLink) end
        end)
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLabel.Text = "디스코드 링크가 복사되었습니다!"
    end

    buyBtn.MouseButton1Click:Connect(copyDiscordLink)
    buyProBtn.MouseButton1Click:Connect(copyDiscordLink)

    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        -- 마스터 키 입력 시 2단계 인증 없이 곧바로 메인 화면 및 프리미엄 활성화
        if enteredKey == masterKey then
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "마스터 키 인증 성공! 바로 실행됩니다."
            _G.WordHelperAuthenticated = true
            _G.WordHelperPremiumAuthenticated = true
            task.wait(0.6)
            keyFrame:Destroy()
            titleFrame.Visible = true
            updatePremiumUIVisibility(true)
        elseif premiumKeys[playerName] and premiumKeys[playerName] == enteredKey then
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "프리미엄 키 인증 성공!"
            task.wait(0.6)
            keyFrame:Destroy()
            createSecondStepUI(true)
        elseif userKeys[playerName] and userKeys[playerName] == enteredKey then
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "일반 키 인증 성공!"
            task.wait(0.6)
            keyFrame:Destroy()
            createSecondStepUI(false)
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

-- 드래그 이동 로직
local dragging, dragStart, startPos = false, nil, nil
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
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        titleFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
