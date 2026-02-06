-- Tic-Tac-Toe Smart & Colored (X - Blue, O - Red)
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local MainFrame = Instance.new("Frame", ScreenGui)
local TitleBar = Instance.new("Frame", MainFrame)
local GridFrame = Instance.new("Frame", MainFrame)
local Grid = Instance.new("UIGridLayout", GridFrame)
local Status = Instance.new("TextLabel", MainFrame)
local CloseBtn = Instance.new("TextButton", TitleBar)
local ResetBtn = Instance.new("TextButton", MainFrame)

-- Настройки окна
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Position = UDim2.new(0.5, -110, 0.5, -130)
MainFrame.Size = UDim2.new(0, 220, 0, 300)
MainFrame.BorderSizePixel = 0

-- Полоска для перетаскивания
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
TitleBar.BorderSizePixel = 0

local dragging, dragInput, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
TitleBar.InputEnded:Connect(function(input) dragging = false end)

-- Кнопка закрытия
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -35, 0, 0)
CloseBtn.Text = "X"
CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.BorderSizePixel = 0
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

-- Статус игры
Status.Size = UDim2.new(1, 0, 0, 30)
Status.Position = UDim2.new(0, 0, 0, 40)
Status.Text = "Твой ход (X)"
Status.TextColor3 = Color3.new(1, 1, 1)
Status.BackgroundTransparency = 1
Status.TextSize = 20

-- Игровое поле
GridFrame.Size = UDim2.new(0, 190, 0, 190)
GridFrame.Position = UDim2.new(0, 15, 0, 75)
GridFrame.BackgroundTransparency = 1
Grid.CellSize = UDim2.new(0, 60, 0, 60)
Grid.CellPadding = UDim2.new(0, 5, 0, 5)

-- Кнопка Заново
ResetBtn.Size = UDim2.new(0, 120, 0, 30)
ResetBtn.Position = UDim2.new(0.5, -60, 1, -30)
ResetBtn.Text = "ИГРАТЬ ЗАНОВО"
ResetBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
ResetBtn.TextColor3 = Color3.new(1, 1, 1)
ResetBtn.BorderSizePixel = 0

local board = {}
local gameActive = true
local combos = {{1,2,3},{4,5,6},{7,8,9},{1,4,7},{2,5,8},{3,6,9},{1,5,9},{3,5,7}}

function checkWin(m)
    for _, c in pairs(combos) do
        if board[c[1]].Text == m and board[c[2]].Text == m and board[c[3]].Text == m then return true end
    end
    return false
end

function isFull()
    for i=1,9 do if board[i].Text == "" then return false end end
    return true
end

function botMove()
    if not gameActive then return end
    local move = nil
    
    -- Пытаемся выиграть
    for _, c in pairs(combos) do
        local countO, empty = 0, nil
        for _, p in pairs(c) do
            if board[p].Text == "O" then countO = countO + 1 elseif board[p].Text == "" then empty = p end
        end
        if countO == 2 and empty then move = empty break end
    end
    
    -- Блокируем игрока
    if not move then
        for _, c in pairs(combos) do
            local countX, empty = 0, nil
            for _, p in pairs(c) do
                if board[p].Text == "X" then countX = countX + 1 elseif board[p].Text == "" then empty = p end
            end
            if countX == 2 and empty then move = empty break end
        end
    end
    
    -- Центр или случайно
    if not move then
        if board[5].Text == "" then move = 5
        else
            local avail = {}
            for i=1,9 do if board[i].Text == "" then table.insert(avail, i) end end
            if #avail > 0 then move = avail[math.random(1, #avail)] end
        end
    end

    if move then
        board[move].Text = "O"
        board[move].TextColor3 = Color3.fromRGB(255, 50, 50) -- Красный О
        if checkWin("O") then Status.Text = "БОТ ВЫИГРАЛ!"; gameActive = false
        elseif isFull() then Status.Text = "НИЧЬЯ!"; gameActive = false
        else Status.Text = "Твой ход (X)" end
    end
end

ResetBtn.MouseButton1Click:Connect(function()
    for i=1,9 do board[i].Text = "" end
    gameActive = true; Status.Text = "Твой ход (X)"
end)

for i = 1, 9 do
    local btn = Instance.new("TextButton", GridFrame)
    btn.Text = ""
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextSize = 40
    btn.Font = Enum.Font.SourceSansBold
    board[i] = btn
    
    btn.MouseButton1Click:Connect(function()
        if btn.Text == "" and gameActive then
            btn.Text = "X"
            btn.TextColor3 = Color3.fromRGB(0, 150, 255) -- Синий X
            if checkWin("X") then Status.Text = "ТЫ ПОБЕДИЛ!"; gameActive = false
            elseif isFull() then Status.Text = "НИЧЬЯ!"; gameActive = false
            else
                Status.Text = "Бот думает..."
                task.wait(0.2)
                botMove()
            end
        end
    end)
end
