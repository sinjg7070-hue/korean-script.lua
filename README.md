-- 서비스 불러오기
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

-- 기존 GUI 제거 (중복 방지)
if CoreGui:FindFirstChild("WordGameHelperUI") then
    CoreGui.WordGameHelperUI:Destroy()
elseif playerGui:FindFirstChild("WordGameHelperUI") then
    playerGui.WordGameHelperUI:Destroy()
end

-- ScreenGui 생성 (CoreGui에 생성하여 감지 우회)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "WordGameHelperUI"
screenGui.ResetOnSpawn = false
if syn and syn.protect_gui then
    syn.protect_gui(screenGui)
    screenGui.Parent = CoreGui
else
    screenGui.Parent = playerGui
end

-- 타이틀 패널 (드래그 가능한 메인 프레임)
local titleFrame = Instance.new("TextButton")
titleFrame.Name = "TitleFrame"
titleFrame.Size = UDim2.new(0, 220, 0, 70) -- 크기 살짝 조절 (개발자 텍스트 공간 확보)
titleFrame.Position = UDim2.new(0.75, 0, 0.1, 0)
titleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleFrame.TextColor3 = Color3.fromRGB(255, 255, 255)
titleFrame.TextSize = 15
titleFrame.Font = Enum.Font.SourceSansBold
titleFrame.Text = "\n단어 맞히기 헬퍼 🖱️" -- 줄바꿈으로 아래쪽 공간 확보
titleFrame.AutoButtonColor = false
titleFrame.Parent = screenGui

local uiCornerBtn = Instance.new("UICorner")
uiCornerBtn.CornerRadius = UDim.new(0, 8)
uiCornerBtn.Parent = titleFrame

-- 💡 [추가] 개발자 표시 텍스트 라벨
local devLabel = Instance.new("TextLabel")
devLabel.Name = "DevLabel"
devLabel.Size = UDim2.new(1, 0, 0, 20)
devLabel.Position = UDim2.new(0, 0, 0, 5)
devLabel.BackgroundTransparency = 1
devLabel.TextColor3 = Color3.fromRGB(170, 170, 170) -- 연한 회색빛
devLabel.TextSize = 12
devLabel.Font = Enum.Font.SourceSansItalic
devLabel.Text = "스크립트 개발자 : 지환"
devLabel.Parent = titleFrame

-- 정답창 (TextLabel)
local answerLabel = Instance.new("TextLabel")
answerLabel.Name = "AnswerLabel"
answerLabel.Size = UDim2.new(0, 220, 0, 45)
answerLabel.Position = UDim2.new(0, 0, 1, 5) -- 타이틀 바로 아래
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

-- 💡 [마우스 드래그 이동 로직]
local dragging = false
local dragInput, dragStart, startPos

titleFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = titleFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        titleFrame.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X, 
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- 텍스트 순수 단어 검증 함수
local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if #txt < 2 or #txt > 15 then return false end
    if tonumber(txt) ~= nil then return false end
    if txt:find("_") or txt:find("R$") or txt:find("Robux") or txt:find("대기") or txt:find("라운드") or txt:find("Kucing") then return false end
    
    return true
end

-- 서버 통신(RemoteEvent)을 통한 실시간 정답 감지
for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
    if v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent") then
        v.OnClientEvent:Connect(function(...)
            local args = {...}
            for _, arg in ipairs(args) do
                if type(arg) == "string" and isValidWord(arg) then
                    answerLabel.Text = "정답: " .. arg
                elseif type(arg) == "table" then
                    for _, subArg in pairs(arg) do
                        if type(subArg) == "string" and isValidWord(subArg) then
                            answerLabel.Text = "정답: " .. subArg
                        end
                    end
                end
            end
        end)
    end
end

-- 백업 주기적 스캔 (ReplicatedStorage 내부의 값 변동 추적)
task.spawn(function()
    while true do
        task.wait(0.5)
        for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
            if obj:IsA("StringValue") then
                local val = obj.Value
                if isValidWord(val) then
                    answerLabel.Text = "정답: " .. val
                    break
                end
            end
        end
    end
end)

print("개발자 표시형 정답 헬퍼 로드 완료!")
