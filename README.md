-- ==========================================
-- [단어 맞히기 헬퍼 - 패치노트 포함 통합 버전]
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
-- [키 모음 정보 데이터 설정]
-- ==========================================
local specialBypassCode = "지환존잘7011" -- 개발자/친구 전용 입력 코드

local savedKeyVault = {
    zxxdaswoNormalKey = "no.1keyap19293949", -- zxxdaswo 기본(일반) 키
    zxxdaswoPremiumKey = "zxxdaswo.key.pro", -- zxxdaswo 프리미엄 키
    casaNovaPremiumKey = "1CasaNova6974_keyesi", -- 1CasaNova6974 프리미엄 키
    masterKeyText = "MASTER_KEY_2026"         -- 공개 마스터 키
}

local userKeys = {
    ["dambii522"] = "no.1keyap191929",
    ["zxxdaswo"] = savedKeyVault.zxxdaswoNormalKey,
    ["1CasaNova6974"] = savedKeyVault.casaNovaPremiumKey,
    ["dohunpoop"] = "dohunpoop_key12",
    ["yfsm_31"] = "yfsm_31.key199"
}

local premiumKeys = {
    ["zxxdaswo"] = savedKeyVault.zxxdaswoPremiumKey,
    ["1CasaNova6974"] = savedKeyVault.casaNovaPremiumKey
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
-- [자동 정답 입력 및 게임 정답 로직]
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

local currentAnswer = ""

local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    if txt:find("#") or txt:find("_") then return false end
    if txt:find("%s") then return false end
    if #txt < 2 or #txt > 20 then return false end
    if tonumber(txt) ~= nil or txt:match("%d") then return false end
    local lowerTxt = txt:lower()
    if lowerTxt == "total" or lowerTxt:find("total") or lowerTxt == "설정" or lowerTxt == "옵션" or lowerTxt == "메뉴" or lowerTxt == "상점" or lowerTxt == "정보" or lowerTxt == "선택됨" then
        return false
    end
    if lowerTxt:match("^cl") or lowerTxt:match("^gui") or lowerTxt:match("^rem") or lowerTxt:match("^http") then
        return false
    end
    if txt:match("[a-zA-Z]") then return false end
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
                    if type(arg) == "string" then processValue(arg)
                    elseif type(arg) == "table" then
                        for _, subArg in pairs(arg) do
                            if type(subArg) == "string" then processValue(subArg) end
                        end
                    end
                end
            end)
        end
    end
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do hookEvent(v) end
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
-- [패치노트 UI 창 생성 함수]
-- ==========================================
local function createPatchNotesUI(keyFrame)
    local patchFrame = Instance.new("Frame")
    patchFrame.Size = UDim2.new(0, 320, 0, 320)
    patchFrame.Position = UDim2.new(0.5, -160, 0.4, -160)
    patchFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    patchFrame.BorderSizePixel = 0
    patchFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = patchFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.SourceSansBold
    title.Text = "📜 스크립트 패치노트 & 업데이트"
    title.Parent = patchFrame

    -- 패치노트 내용 스크롤/텍스트 영역
    local contentBox = Instance.new("TextLabel")
    contentBox.Size = UDim2.new(0, 280, 0, 210)
    contentBox.Position = UDim2.new(0.5, -140, 0, 45)
    contentBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    contentBox.TextColor3 = Color3.fromRGB(220, 220, 220)
    contentBox.TextSize = 12
    contentBox.Font = Enum.Font.SourceSans
    contentBox.TextXAlignment = Enum.TextXAlignment.Left
    contentBox.TextYAlignment = Enum.TextYAlignment.Top
    contentBox.TextWrapped = true
    contentBox.Text = [[
[ v1.3 업데이트 내역 ]
• 1CasaNova6974 프리미엄 키 추가 완료 (전용 키 연동)
• 프리미엄 전용 [자동정답 ON/OFF] 기능 탑재
• 리모트 이벤트 및 StringValue 실시간 단어 추출 로직 강화

[ v1.2 업데이트 내역 ]
• 2단계 본인 확인 인증 시스템(Username & Display Name) 적용
• 개발자 및 친구 전용 바이패스 코드 기능 추가
• UI 드래그 이동 기능 및 깔끔한 모서리 디자인 적용

[ v1.1 업데이트 내역 ]
• 불필요한 시스템 텍스트 및 숫자/특수문자 필터링 정교화
• 키 시스템 초기화 및 스크립트 완전 삭제 버튼 추가
]]
    contentBox.Parent = patchFrame

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 6)
    boxCorner.Parent = contentBox

    -- 닫기 버튼
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 280, 0, 32)
    closeBtn.Position = UDim2.new(0.5, -140, 0, 268)
    closeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 13
    closeBtn.Font = Enum.Font.SourceSansBold
    closeBtn.Text = "닫기"
    closeBtn.Parent = patchFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = closeBtn

    closeBtn.MouseButton1Click:Connect(function()
        patchFrame:Destroy()
        if keyFrame then
            keyFrame.Visible = true
        end
    end)
end

-- ==========================================
-- [저장된 키 모음 정보 창]
-- ==========================================
local function createKeyInfoResultUI(specialFrame)
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 360, 0, 310)
    infoFrame.Position = UDim2.new(0.5, -180, 0.4, -155)
    infoFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = infoFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.SourceSansBold
    title.Text = "저장된 키 모음 정보"
    title.Parent = infoFrame

    local normalKeyBtn = Instance.new("TextButton")
    normalKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    normalKeyBtn.Position = UDim2.new(0.5, -160, 0, 48)
    normalKeyBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
    normalKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    normalKeyBtn.TextSize = 13
    normalKeyBtn.Font = Enum.Font.SourceSansBold
    normalKeyBtn.Text = "기본 키로 적용 및 실행\n(" .. savedKeyVault.zxxdaswoNormalKey .. ")"
    normalKeyBtn.Parent = infoFrame

    local c1 = Instance.new("UICorner")
    c1.CornerRadius = UDim.new(0, 6)
    c1.Parent = normalKeyBtn

    local premiumKeyBtn = Instance.new("TextButton")
    premiumKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    premiumKeyBtn.Position = UDim2.new(0.5, -160, 0, 98)
    premiumKeyBtn.BackgroundColor3 = Color3.fromRGB(230, 130, 0)
    premiumKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    premiumKeyBtn.TextSize = 13
    premiumKeyBtn.Font = Enum.Font.SourceSansBold
    premiumKeyBtn.Text = "프리미엄 키로 적용 및 실행\n(" .. savedKeyVault.zxxdaswoPremiumKey .. ")"
    premiumKeyBtn.Parent = infoFrame

    local c2 = Instance.new("UICorner")
    c2.CornerRadius = UDim.new(0, 6)
    c2.Parent = premiumKeyBtn

    local masterKeyBtn = Instance.new("TextButton")
    masterKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    masterKeyBtn.Position = UDim2.new(0.5, -160, 0, 148)
    masterKeyBtn.BackgroundColor3 = Color3.fromRGB(219, 112, 147)
    masterKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    masterKeyBtn.TextSize = 13
    masterKeyBtn.Font = Enum.Font.SourceSansBold
    masterKeyBtn.Text = "마스터 키로 적용 및 실행\n(" .. savedKeyVault.masterKeyText .. ")"
    masterKeyBtn.Parent = infoFrame

    local c3 = Instance.new("UICorner")
    c3.CornerRadius = UDim.new(0, 6)
    c3.Parent = masterKeyBtn

    normalKeyBtn.MouseButton1Click:Connect(function()
        _G.WordHelperAuthenticated = true
        _G.WordHelperPremiumAuthenticated = false
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        titleFrame.Visible = true
        updatePremiumUIVisibility(false)
    end)

    premiumKeyBtn.MouseButton1Click:Connect(function()
        _G.WordHelperAuthenticated = true
        _G.WordHelperPremiumAuthenticated = true
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        titleFrame.Visible = true
        updatePremiumUIVisibility(true)
    end)

    masterKeyBtn.MouseButton1Click:Connect(function()
        _G.WordHelperAuthenticated = true
        _G.WordHelperPremiumAuthenticated = true
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        titleFrame.Visible = true
        updatePremiumUIVisibility(true)
    end)
end

-- ==========================================
-- [개발자 전용 코드 입력 UI]
-- ==========================================
local function createSpecialCodeUI(keyFrame)
    local specialFrame = Instance.new("Frame")
    specialFrame.Size = UDim2.new(0, 300, 0, 180)
    specialFrame.Position = UDim2.new(0.5, -150, 0.4, -90)
    specialFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    specialFrame.BorderSizePixel = 0
    specialFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = specialFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 35)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 15
    title.Font = Enum.Font.SourceSansBold
    title.Text = "개발자 / 허용한 친구 코드 입력"
    title.Parent = specialFrame

    local codeBox = Instance.new("TextBox")
    codeBox.Size = UDim2.new(0, 260, 0, 32)
    codeBox.Position = UDim2.new(0.5, -130, 0, 45)
    codeBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    codeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    codeBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    codeBox.PlaceholderText = "코드를 입력하세요..."
    codeBox.TextSize = 13
    codeBox.Text = ""
    codeBox.Parent = specialFrame

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 6)
    boxCorner.Parent = codeBox

    local statusLbl = Instance.new("TextLabel")
    statusLbl.Size = UDim2.new(1, 0, 0, 25)
    statusLbl.Position = UDim2.new(0, 0, 0, 85)
    statusLbl.BackgroundTransparency = 1
    statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLbl.TextSize = 12
    statusLbl.Font = Enum.Font.SourceSansItalic
    statusLbl.Text = ""
    statusLbl.Parent = specialFrame

    local submitCodeBtn = Instance.new("TextButton")
    submitCodeBtn.Size = UDim2.new(0, 125, 0, 32)
    submitCodeBtn.Position = UDim2.new(0.5, -130, 0, 120)
    submitCodeBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
    submitCodeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitCodeBtn.TextSize = 13
    submitCodeBtn.Font = Enum.Font.SourceSansBold
    submitCodeBtn.Text = "확인"
    submitCodeBtn.Parent = specialFrame

    local btnCorner1 = Instance.new("UICorner")
    btnCorner1.CornerRadius = UDim.new(0, 6)
    btnCorner1.Parent = submitCodeBtn

    local cancelCodeBtn = Instance.new("TextButton")
    cancelCodeBtn.Size = UDim2.new(0, 125, 0, 32)
    cancelCodeBtn.Position = UDim2.new(0.5, 5, 0, 120)
    cancelCodeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    cancelCodeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    cancelCodeBtn.TextSize = 13
    cancelCodeBtn.Font = Enum.Font.SourceSansBold
    cancelCodeBtn.Text = "취소"
    cancelCodeBtn.Parent = specialFrame

    local btnCorner2 = Instance.new("UICorner")
    btnCorner2.CornerRadius = UDim.new(0, 6)
    btnCorner2.Parent = cancelCodeBtn

    cancelCodeBtn.MouseButton1Click:Connect(function()
        specialFrame:Destroy()
        keyFrame.Visible = true
    end)

    submitCodeBtn.MouseButton1Click:Connect(function()
        local entered = codeBox.Text:gsub("^%s*(.-)%s*$", "%1")
        if entered == specialBypassCode then
            statusLbl.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLbl.Text = "인증 성공!"
            task.wait(0.4)
            createKeyInfoResultUI(specialFrame)
            if keyFrame then keyFrame:Destroy() end
        else
            statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLbl.Text = "코드가 일치하지 않습니다."
        end
    end)
end

-- ==========================================
-- [인증창 및 드래그 UI 시스템]
-- ==========================================
local function createSecondStepUI(isPremium)
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
    keyFrame.Size = UDim2.new(0, 300, 0, 325)
    keyFrame.Position = UDim2.new(0.5, -150, 0.4, -162)
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

    local devFriendBtn = Instance.new("TextButton")
    devFriendBtn.Size = UDim2.new(0, 260, 0, 28)
    devFriendBtn.Position = UDim2.new(0.5, -130, 0, 180)
    devFriendBtn.BackgroundColor3 = Color3.fromRGB(120, 60, 180)
    devFriendBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    devFriendBtn.TextSize = 12
    devFriendBtn.Font = Enum.Font.SourceSansBold
    devFriendBtn.Text = "스크 개발자 전용 또는 허용한 친구"
    devFriendBtn.Parent = keyFrame

    local uiCornerDevFriend = Instance.new("UICorner")
    uiCornerDevFriend.CornerRadius = UDim.new(0, 6)
    uiCornerDevFriend.Parent = devFriendBtn

    -- [패치노트 버튼 추가]
    local patchNoteBtn = Instance.new("TextButton")
    patchNoteBtn.Size = UDim2.new(0, 260, 0, 28)
    patchNoteBtn.Position = UDim2.new(0.5, -130, 0, 214)
    patchNoteBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    patchNoteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    patchNoteBtn.TextSize = 12
    patchNoteBtn.Font = Enum.Font.SourceSansBold
    patchNoteBtn.Text = "📜 패치노트 및 업데이트 확인"
    patchNoteBtn.Parent = keyFrame

    local uiCornerPatch = Instance.new("UICorner")
    uiCornerPatch.CornerRadius = UDim.new(0, 6)
    uiCornerPatch.Parent = patchNoteBtn

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 25)
    statusLabel.Position = UDim2.new(0, 0, 0, 248)
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

    devFriendBtn.MouseButton1Click:Connect(function()
        keyFrame.Visible = false
        createSpecialCodeUI(keyFrame)
    end)

    patchNoteBtn.MouseButton1Click:Connect(function()
        keyFrame.Visible = false
        createPatchNotesUI(keyFrame)
    end)

    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        if premiumKeys[playerName] and premiumKeys[playerName] == enteredKey then
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
end)
