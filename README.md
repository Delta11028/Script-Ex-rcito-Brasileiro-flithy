--[[
    DEEPHAT ULTIMATE: MULTI-LEVEL MENU EDITION (FULL VERSION)
    Developer: DeepHat (Kindo)
    Mod: Correção de Indexação de Dados e Sistema de Navegação
]]
 
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
 
-- [[ 🔗 CONFIGURAÇÃO DE LINKS (CORRIGIDOS PARA /RAW/) ]]
local Links = {
    ["P1_Esquerda"] = "https://pastebin.com/raw/xTjgHar7", 
    ["P1_Direita"]  = "https://pastebin.com/raw/z8meYsjY",
    ["P2_1"] = "https://pastebin.com/raw/jHrh82rp", ["P2_2"] = "https://pastebin.com/raw/PSthPfUG",
    ["P2_3"] = "https://pastebin.com/raw/C0RDa1K0", ["P2_4"] = "https://pastebin.com/raw/hvA1hHGr",
    ["P2_5"] = "https://pastebin.com/raw/6vKjSy6m", ["P2_6"] = "https://pastebin.com/raw/MRxuJyRq",
    ["P3_Esquerda"] = "https://pastebin.com/raw/89BtCK46", ["P3_Direita"] = "https://pastebin.com/raw/FfAQs9xy",
    ["P4_Esquerda"] = "https://pastebin.com/raw/8caEQibu", ["P4_Direita"] = "https://pastebin.com/raw/3G3gbrzV",
    ["P5_Esquerda"] = "https://pastebin.com/raw/dmfiYh95", ["P5_Direita"] = "https://pastebin.com/raw/FrvKkj1H",
    ["P6_Esquerda"] = "https://pastebin.com/raw/6huwYpyV", ["P6_Direita"] = "https://pastebin.com/raw/ZGCL9V6e",
    ["P7_Esquerda"] = "https://pastebin.com/raw/R9wGEtZJ", ["P7_Meio"] = "https://pastebin.com/raw/dgzZjnju", ["P7_Direita"] = "https://pastebin.com/raw/nmV2CjhB",
    ["P8_Esquerda"] = "https://pastebin.com/raw/tb7sycVR", ["P8_Meio"] = "https://pastebin.com/raw/BCFEQeYJ", ["P8_Direita"] = "https://pastebin.com/raw/5DqEWkaq",
    ["P9_Esquerda"] = "https://pastebin.com/raw/uLAgDkE4", ["P9_Meio"] = "https://pastebin.com/raw/DM7QGbjR", ["P9_Direita"] = "https://pastebin.com/raw/FuzhCjZh",
    ["Torre_Esquerda"] = "https://pastebin.com/raw/kscykq8r",
    ["Torre_Direita"]  = "https://pastebin.com/raw/W2WJRiWd"
}
 
local MenuConfig = {
    ["Parkour 1"] = {"Esquerda", "Direita"},
    ["Parkour 2"] = {"1", "2", "3", "4", "5", "6"},
    ["Parkour 3"] = {"Esquerda", "Direita"},
    ["Parkour 4"] = {"Esquerda", "Direita"},
    ["Parkour 5"] = {"Esquerda", "Direita"},
    ["Parkour 6"] = {"Esquerda", "Direita"},
    ["Parkour 7"] = {"Esquerda", "Meio", "Direita"},
    ["Parkour 8"] = {"Esquerda", "Meio", "Direita"},
    ["Parkour 9"] = {"Esquerda", "Meio", "Direita"},
    ["Torre"]     = {"Esquerda", "Direita"}
}
 
local ParkourDatabase = {}
local SelectedMapName = nil
local Settings = { IsReplaying = false, Smoothness = 0.4 }
 
-- [[ ENGINE DE MOVIMENTO ]]
local function StartReplay(Character, PathData)
    local RootPart = Character:WaitForChild("HumanoidRootPart", 5)
    local Humanoid = Character:WaitForChild("Humanoid", 5)
    if not RootPart or not Humanoid then return end
 
    Settings.IsReplaying = true
 
    for i = 1, #PathData do
        if not Settings.IsReplaying then break end
        local d = PathData[i]
        if d.P and d.R then
            local targetPos = Vector3.new(d.P[1], d.P[2], d.P[3])
            local targetLook = Vector3.new(d.R[1], d.R[2], d.R[3])
            local targetVel = d.V and Vector3.new(d.V[1], d.V[2], d.V[3]) or Vector3.new(0, 0, 0)
 
            RootPart.CFrame = RootPart.CFrame:Lerp(CFrame.new(targetPos, targetPos + targetLook), Settings.Smoothness)
            RootPart.AssemblyLinearVelocity = targetVel
 
            if targetVel.Y > 5 then
                Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            else
                Humanoid:ChangeState(Enum.HumanoidStateType.Running)
            end
        end
        task.wait(0.015) 
    end
    Settings.IsReplaying = false
end
 
-- [[ DOWNLOADER ]]
local function SafeRequest(url)
    local reqFunc = request or http_request or syn.request
    if reqFunc then
        local success, result = pcall(function()
            local response = reqFunc({ Url = url, Method = "GET" })
            return response.Body
        end)
        return success and result or nil
    else
        local success, result = pcall(function() return HttpService:GetAsync(url) end)
        return success and result or nil
    end
end
 
local function GetMapData(name)
    local url = Links[name]
    if not url then return nil end
    local body = SafeRequest(url)
    if body and body ~= "" then
        local success, decoded = pcall(function() return HttpService:JSONDecode(body) end)
        if success and decoded then 
            return decoded.Data or decoded 
        end
    end
    return nil
end
 
-- [[ INTERFACE GRÁFICA ]]
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 450, 0, 300)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
 
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 140, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Sidebar.BorderSizePixel = 0
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 12)
 
local LevelList = Instance.new("ScrollingFrame", Sidebar)
LevelList.Size = UDim2.new(1, -10, 0.9, 0)
LevelList.Position = UDim2.new(0, 5, 0.05, 0)
LevelList.BackgroundTransparency = 1
LevelList.ScrollBarThickness = 4
LevelList.ScrollBarImageColor3 = Color3.fromRGB(0, 255, 150)
LevelList.CanvasSize = UDim2.new(0, 0, 0, 0)
LevelList.ZIndex = 3
 
local LevelLayout = Instance.new("UIListLayout", LevelList)
LevelLayout.Padding = UDim.new(0, 8)
LevelLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
 
local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Size = UDim2.new(0.6, 0, 1, 0)
ContentFrame.Position = UDim2.new(0.35, 0, 0, 0)
ContentFrame.BackgroundTransparency = 1
 
local TitleLabel = Instance.new("TextLabel", ContentFrame)
TitleLabel.Size = UDim2.new(1, 0, 0, 50)
TitleLabel.Text = "MDC CLOUD"
TitleLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 24
TitleLabel.BackgroundTransparency = 1
 
local InfoLabel = Instance.new("TextLabel", ContentFrame)
InfoLabel.Size = UDim2.new(1, 0, 0, 30)
InfoLabel.Position = UDim2.new(0, 0, 0, 60)
InfoLabel.Text = "Selecione um mapa"
InfoLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.TextSize = 14
InfoLabel.BackgroundTransparency = 1
 
local ReplayBtn = Instance.new("TextButton", ContentFrame)
ReplayBtn.Size = UDim2.new(0.8, 0, 0, 50)
ReplayBtn.Position = UDim2.new(0.1, 0, 0.5, 0)
ReplayBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 150)
ReplayBtn.Text = "REPRODUZIR"
ReplayBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ReplayBtn.Font = Enum.Font.GothamBold
ReplayBtn.TextSize = 16
Instance.new("UICorner", ReplayBtn).CornerRadius = UDim.new(0, 8)
 
-- [[ LÓGICA DE NAVEGAÇÃO E MENU ]]
 
local function UpdateCanvasSize()
    task.wait(0.1)
    LevelList.CanvasSize = UDim2.new(0, 0, 0, LevelLayout.AbsoluteContentSize.Y + 20)
end
 
local function ClearList()
    for _, child in pairs(LevelList:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
end
 
local function CreateMenuButton(text, callback, isBackBtn)
    local btn = Instance.new("TextButton", LevelList)
    btn.Size = UDim2.new(0.9, 0, 0, 35)
    btn.BackgroundColor3 = isBackBtn and Color3.fromRGB(80, 30, 30) or Color3.fromRGB(35, 35, 35)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(220, 220, 220)
    btn.Font = isBackBtn and Enum.Font.GothamBold or Enum.Font.GothamMedium
    btn.TextSize = 13
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
 
    btn.MouseButton1Click:Connect(callback)
    return btn
end
 
local function LoadMainMenu()
    ClearList()
    InfoLabel.Text = "Menu Principal"
    InfoLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    
    local MainCategories = {"Parkour 1", "Parkour 2", "Parkour 3", "Parkour 4", "Parkour 5", "Parkour 6", "Parkour 7", "Parkour 8", "Parkour 9", "Torre"}
    
    for _, name in ipairs(MainCategories) do
        CreateMenuButton(name, function()
            ClearList()
            
            CreateMenuButton("⬅ Voltar", function()
                LoadMainMenu()
            end, true)
            
            InfoLabel.Text = name .. " -> Escolha:"
            InfoLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            
            local subOptions = MenuConfig[name] or {}
            
            for _, opt in ipairs(subOptions) do
                CreateMenuButton(opt, function()
                    local finalKey = ""
                    
                    if name == "Torre" then
                        finalKey = "Torre_" .. opt
                    elseif name == "Parkour 2" then
                        finalKey = "P2_" .. opt
                    elseif name:find("Parkour") then
                        local num = name:match("%d+") 
                        if num then
                            finalKey = "P" .. num .. "_" .. opt
                        end
                    end
                    
                    SelectedMapName = finalKey
                    InfoLabel.Text = "Carregando " .. name .. " (" .. opt .. ")..."
                    
                    local data = GetMapData(finalKey)
                    if data then
                        ParkourDatabase[finalKey] = data
                        InfoLabel.Text = name .. " [" .. opt .. "] OK!"
                        InfoLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
                    else
                        InfoLabel.Text = "Erro ao carregar: " .. finalKey
                        InfoLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
                    end
                    UpdateCanvasSize()
                end)
            end
            UpdateCanvasSize()
        end)
    end
    UpdateCanvasSize()
end
 
LoadMainMenu()
 
ReplayBtn.MouseButton1Click:Connect(function()
    if not SelectedMapName then 
        InfoLabel.Text = "SELECIONE UM MAPA!"
        return 
    end
 
    local data = ParkourDatabase[SelectedMapName]
    if not data or #data == 0 then 
        InfoLabel.Text = "DADOS NÃO ENCONTRADOS!"
        return 
    end
 
    local char = Players.LocalPlayer.Character
    if not char or Settings.IsReplaying then return end
 
    ReplayBtn.Text = "EXECUTANDO..."
    ReplayBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 150)
 
    StartReplay(char, data)
 
    ReplayBtn.Text = "REPRODUZIR"
    ReplayBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 150)
end)
 
local function MakeDraggable(guiElement)
    local dragging, dragStart, startPos
    guiElement.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = guiElement.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            guiElement.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
end
MakeDraggable(MainFrame)
 
print("------------------------------------------")
print("DEEPHAT ULTIMATE: FULL VERSION READY")
print("------------------------------------------")
