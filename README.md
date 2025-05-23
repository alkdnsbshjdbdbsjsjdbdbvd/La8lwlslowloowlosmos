-- Services
local replicatedStorage = game:GetService("ReplicatedStorage")
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local workspace = game:GetService("Workspace")
local vim = game:GetService("VirtualInputManager")
local uis = game:GetService("UserInputService")

-- === Funktionen ===

local function unequipAllPets()
    local petFolder = localPlayer:WaitForChild("petsFolder")
    for _, petSet in pairs(petFolder:GetChildren()) do
        if petSet:IsA("Folder") then
            for _, pet in pairs(petSet:GetChildren()) do
                replicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.1)
end

local function equipPetByName(petName)
    unequipAllPets()
    task.wait(0.01)
    for _, pet in pairs(localPlayer.petsFolder.Unique:GetChildren()) do
        if pet.Name == petName then
            replicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
        end
    end
end

local function findMachine(machineName)
    local machine = workspace.machinesFolder:FindFirstChild(machineName)
    if not machine then
        for _, folder in pairs(workspace:GetChildren()) do
            if folder:IsA("Folder") and folder.Name:find("machines") then
                machine = folder:FindFirstChild(machineName)
                if machine then break end
            end
        end
    end
    return machine
end

local function pressEKey()
    vim:SendKeyEvent(true, "E", false, game)
    task.wait(0.1)
    vim:SendKeyEvent(false, "E", false, game)
end

local function autoPunchRock(rockName, stopCondition)
    local function punch()
        replicatedStorage.rEvents.punchEvent:FireServer(true)
    end

    while not stopCondition() do
        local rock = workspace:FindFirstChild(rockName)
        if rock then
            localPlayer.Character.HumanoidRootPart.CFrame = rock.CFrame + Vector3.new(0, 3, 0)
            punch()
        end
        task.wait(0.2)
    end
end

-- === UI ===

local screenGui = Instance.new("ScreenGui", localPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "StrengthFarmPanel"
screenGui.ResetOnSpawn = false

-- Main Frame
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 600, 0, 400)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = false

-- Custom Dragging
local dragging, dragInput, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
uis.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Sidebar Tabs
local sidebar = Instance.new("Frame", mainFrame)
sidebar.Size = UDim2.new(0, 120, 1, 0)
sidebar.Position = UDim2.new(0, 0, 0, 0)
sidebar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local function createTabButton(name, yPos)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    return btn
end

local mainTabBtn = createTabButton("Main", 20)
local rockTabBtn = createTabButton("Rocks", 70)

-- Content Frames
local mainTab = Instance.new("Frame", mainFrame)
mainTab.Size = UDim2.new(1, -130, 1, -10)
mainTab.Position = UDim2.new(0, 130, 0, 5)
mainTab.BackgroundColor3 = Color3.fromRGB(45, 45, 45)

local rockTab = mainTab:Clone()
rockTab.Parent = mainFrame
rockTab.Visible = false

-- Tab switching
mainTabBtn.MouseButton1Click:Connect(function()
    mainTab.Visible = true
    rockTab.Visible = false
end)
rockTabBtn.MouseButton1Click:Connect(function()
    mainTab.Visible = false
    rockTab.Visible = true
end)

-- === Main Tab Inhalte ===

local toggleButton = Instance.new("TextButton", mainTab)
toggleButton.Size = UDim2.new(0, 280, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.Text = "Start Strength Farm"
toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 20

local statusLabel = Instance.new("TextLabel", mainTab)
statusLabel.Size = UDim2.new(0, 280, 0, 30)
statusLabel.Position = UDim2.new(0, 20, 0, 80)
statusLabel.Text = "Status: Stopped"
statusLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 18

local rebirthTargetInput = Instance.new("TextBox", mainTab)
rebirthTargetInput.Size = UDim2.new(0, 280, 0, 30)
rebirthTargetInput.Position = UDim2.new(0, 20, 0, 120)
rebirthTargetInput.PlaceholderText = "Enter target rebirths"
rebirthTargetInput.Text = ""
rebirthTargetInput.ClearTextOnFocus = false
rebirthTargetInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
rebirthTargetInput.TextColor3 = Color3.new(1, 1, 1)
rebirthTargetInput.Font = Enum.Font.SourceSans
rebirthTargetInput.TextSize = 18

local rockNameInput = Instance.new("TextBox", mainTab)
rockNameInput.Size = UDim2.new(0, 280, 0, 30)
rockNameInput.Position = UDim2.new(0, 20, 0, 160)
rockNameInput.PlaceholderText = "Enter Legend Rock name"
rockNameInput.Text = ""
rockNameInput.ClearTextOnFocus = false
rockNameInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
rockNameInput.TextColor3 = Color3.new(1, 1, 1)
rockNameInput.Font = Enum.Font.SourceSans
rockNameInput.TextSize = 18

local rebirthDisplay = Instance.new("TextLabel", mainTab)
rebirthDisplay.Size = UDim2.new(0, 280, 0, 30)
rebirthDisplay.Position = UDim2.new(0, 20, 0, 200)
rebirthDisplay.Text = "Current Rebirths: 0"
rebirthDisplay.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
rebirthDisplay.TextColor3 = Color3.new(1, 1, 1)
rebirthDisplay.Font = Enum.Font.SourceSans
rebirthDisplay.TextSize = 18

-- Live Rebirth Update
task.spawn(function()
    while true do
        pcall(function()
            rebirthDisplay.Text = "Current Rebirths: " .. localPlayer.leaderstats.Rebirths.Value
        end)
        task.wait(1)
    end
end)

-- === Farming Logic ===

local farmingEnabled = false
toggleButton.MouseButton1Click:Connect(function()
    farmingEnabled = not farmingEnabled
    if farmingEnabled then
        toggleButton.Text = "Stop Strength Farm"
        statusLabel.Text = "Status: Running"

        local targetRebirths = tonumber(rebirthTargetInput.Text) or 0
        local rockName = rockNameInput.Text

        task.spawn(function()
            while farmingEnabled do
                local rebirths = localPlayer.leaderstats.Rebirths.Value
                local requiredStrength = 10000 + (5000 * rebirths)

                if localPlayer.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                    local gr = localPlayer.ultimatesFolder["Golden Rebirth"].Value
                    requiredStrength = math.floor(requiredStrength * (1 - (gr * 0.1)))
                end

                unequipAllPets()
                task.wait(0.1)
                equipPetByName("Swift Samurai")

                while localPlayer.leaderstats.Strength.Value < requiredStrength and farmingEnabled do
                    for _ = 1, 10 do
                        localPlayer.muscleEvent:FireServer("rep")
                    end
                    task.wait()
                end

                if not farmingEnabled then return end

                unequipAllPets()
                task.wait(0.1)
                equipPetByName("Tribal Overlord")

                local machine = findMachine("Jungle Bar Lift")
                if machine and machine:FindFirstChild("interactSeat") then
                    localPlayer.Character.HumanoidRootPart.CFrame = machine.interactSeat.CFrame * CFrame.new(0, 3, 0)
                    repeat task.wait(0.1) pressEKey() until localPlayer.Character.Humanoid.Sit or not farmingEnabled
                end

                if not farmingEnabled then return end

                local beforeRebirths = localPlayer.leaderstats.Rebirths.Value
                repeat
                    replicatedStorage.rEvents.rebirthRemote:InvokeServer("rebirthRequest")
                    task.wait(0.1)
                until localPlayer.leaderstats.Rebirths.Value > beforeRebirths or not farmingEnabled

                if rockName ~= "" then
                    autoPunchRock(rockName, function()
                        return not farmingEnabled or (targetRebirths > 0 and localPlayer.leaderstats.Rebirths.Value >= targetRebirths)
                    end)
                end

                if targetRebirths > 0 and localPlayer.leaderstats.Rebirths.Value >= targetRebirths then
                    farmingEnabled = false
                    toggleButton.Text = "Start Strength Farm"
                    statusLabel.Text = "Status: Stopped"
                    break
                end
                task.wait()
            end
        end)
    else
        toggleButton.Text = "Start Strength Farm"
        statusLabel.Text = "Status: Stopped"
    end
end)-- Services
local replicatedStorage = game:GetService("ReplicatedStorage")
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local workspace = game:GetService("Workspace")
local vim = game:GetService("VirtualInputManager")
local uis = game:GetService("UserInputService")

-- === Funktionen ===

local function unequipAllPets()
    local petFolder = localPlayer:WaitForChild("petsFolder")
    for _, petSet in pairs(petFolder:GetChildren()) do
        if petSet:IsA("Folder") then
            for _, pet in pairs(petSet:GetChildren()) do
                replicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.1)
end

local function equipPetByName(petName)
    unequipAllPets()
    task.wait(0.01)
    for _, pet in pairs(localPlayer.petsFolder.Unique:GetChildren()) do
        if pet.Name == petName then
            replicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
        end
    end
end

local function findMachine(machineName)
    local machine = workspace.machinesFolder:FindFirstChild(machineName)
    if not machine then
        for _, folder in pairs(workspace:GetChildren()) do
            if folder:IsA("Folder") and folder.Name:find("machines") then
                machine = folder:FindFirstChild(machineName)
                if machine then break end
            end
        end
    end
    return machine
end

local function pressEKey()
    vim:SendKeyEvent(true, "E", false, game)
    task.wait(0.1)
    vim:SendKeyEvent(false, "E", false, game)
end

local function autoPunchRock(rockName, stopCondition)
    local function punch()
        replicatedStorage.rEvents.punchEvent:FireServer(true)
    end

    while not stopCondition() do
        local rock = workspace:FindFirstChild(rockName)
        if rock then
            localPlayer.Character.HumanoidRootPart.CFrame = rock.CFrame + Vector3.new(0, 3, 0)
            punch()
        end
        task.wait(0.2)
    end
end

-- === UI ===

local screenGui = Instance.new("ScreenGui", localPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "StrengthFarmPanel"
screenGui.ResetOnSpawn = false

-- Main Frame
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 600, 0, 400)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = false

-- Custom Dragging
local dragging, dragInput, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
uis.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Sidebar Tabs
local sidebar = Instance.new("Frame", mainFrame)
sidebar.Size = UDim2.new(0, 120, 1, 0)
sidebar.Position = UDim2.new(0, 0, 0, 0)
sidebar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local function createTabButton(name, yPos)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    return btn
end

local mainTabBtn = createTabButton("Main", 20)
local rockTabBtn = createTabButton("Rocks", 70)

-- Content Frames
local mainTab = Instance.new("Frame", mainFrame)
mainTab.Size = UDim2.new(1, -130, 1, -10)
mainTab.Position = UDim2.new(0, 130, 0, 5)
mainTab.BackgroundColor3 = Color3.fromRGB(45, 45, 45)

local rockTab = mainTab:Clone()
rockTab.Parent = mainFrame
rockTab.Visible = false

-- Tab switching
mainTabBtn.MouseButton1Click:Connect(function()
    mainTab.Visible = true
    rockTab.Visible = false
end)
rockTabBtn.MouseButton1Click:Connect(function()
    mainTab.Visible = false
    rockTab.Visible = true
end)

-- === Main Tab Inhalte ===

local toggleButton = Instance.new("TextButton", mainTab)
toggleButton.Size = UDim2.new(0, 280, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.Text = "Start Strength Farm"
toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 20

local statusLabel = Instance.new("TextLabel", mainTab)
statusLabel.Size = UDim2.new(0, 280, 0, 30)
statusLabel.Position = UDim2.new(0, 20, 0, 80)
statusLabel.Text = "Status: Stopped"
statusLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 18

local rebirthTargetInput = Instance.new("TextBox", mainTab)
rebirthTargetInput.Size = UDim2.new(0, 280, 0, 30)
rebirthTargetInput.Position = UDim2.new(0, 20, 0, 120)
rebirthTargetInput.PlaceholderText = "Enter target rebirths"
rebirthTargetInput.Text = ""
rebirthTargetInput.ClearTextOnFocus = false
rebirthTargetInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
rebirthTargetInput.TextColor3 = Color3.new(1, 1, 1)
rebirthTargetInput.Font = Enum.Font.SourceSans
rebirthTargetInput.TextSize = 18

local rockNameInput = Instance.new("TextBox", mainTab)
rockNameInput.Size = UDim2.new(0, 280, 0, 30)
rockNameInput.Position = UDim2.new(0, 20, 0, 160)
rockNameInput.PlaceholderText = "Enter Legend Rock name"
rockNameInput.Text = ""
rockNameInput.ClearTextOnFocus = false
rockNameInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
rockNameInput.TextColor3 = Color3.new(1, 1, 1)
rockNameInput.Font = Enum.Font.SourceSans
rockNameInput.TextSize = 18

local rebirthDisplay = Instance.new("TextLabel", mainTab)
rebirthDisplay.Size = UDim2.new(0, 280, 0, 30)
rebirthDisplay.Position = UDim2.new(0, 20, 0, 200)
rebirthDisplay.Text = "Current Rebirths: 0"
rebirthDisplay.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
rebirthDisplay.TextColor3 = Color3.new(1, 1, 1)
rebirthDisplay.Font = Enum.Font.SourceSans
rebirthDisplay.TextSize = 18

-- Live Rebirth Update
task.spawn(function()
    while true do
        pcall(function()
            rebirthDisplay.Text = "Current Rebirths: " .. localPlayer.leaderstats.Rebirths.Value
        end)
        task.wait(1)
    end
end)

-- === Farming Logic ===

local farmingEnabled = false
toggleButton.MouseButton1Click:Connect(function()
    farmingEnabled = not farmingEnabled
    if farmingEnabled then
        toggleButton.Text = "Stop Strength Farm"
        statusLabel.Text = "Status: Running"

        local targetRebirths = tonumber(rebirthTargetInput.Text) or 0
        local rockName = rockNameInput.Text

        task.spawn(function()
            while farmingEnabled do
                local rebirths = localPlayer.leaderstats.Rebirths.Value
                local requiredStrength = 10000 + (5000 * rebirths)

                if localPlayer.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                    local gr = localPlayer.ultimatesFolder["Golden Rebirth"].Value
                    requiredStrength = math.floor(requiredStrength * (1 - (gr * 0.1)))
                end

                unequipAllPets()
                task.wait(0.1)
                equipPetByName("Swift Samurai")

                while localPlayer.leaderstats.Strength.Value < requiredStrength and farmingEnabled do
                    for _ = 1, 10 do
                        localPlayer.muscleEvent:FireServer("rep")
                    end
                    task.wait()
                end

                if not farmingEnabled then return end

                unequipAllPets()
                task.wait(0.1)
                equipPetByName("Tribal Overlord")

                local machine = findMachine("Jungle Bar Lift")
                if machine and machine:FindFirstChild("interactSeat") then
                    localPlayer.Character.HumanoidRootPart.CFrame = machine.interactSeat.CFrame * CFrame.new(0, 3, 0)
                    repeat task.wait(0.1) pressEKey() until localPlayer.Character.Humanoid.Sit or not farmingEnabled
                end

                if not farmingEnabled then return end

                local beforeRebirths = localPlayer.leaderstats.Rebirths.Value
                repeat
                    replicatedStorage.rEvents.rebirthRemote:InvokeServer("rebirthRequest")
                    task.wait(0.1)
                until localPlayer.leaderstats.Rebirths.Value > beforeRebirths or not farmingEnabled

                if rockName ~= "" then
                    autoPunchRock(rockName, function()
                        return not farmingEnabled or (targetRebirths > 0 and localPlayer.leaderstats.Rebirths.Value >= targetRebirths)
                    end)
                end

                if targetRebirths > 0 and localPlayer.leaderstats.Rebirths.Value >= targetRebirths then
                    farmingEnabled = false
                    toggleButton.Text = "Start Strength Farm"
                    statusLabel.Text = "Status: Stopped"
                    break
                end
                task.wait()
            end
        end)
    else
        toggleButton.Text = "Start Strength Farm"
        statusLabel.Text = "Status: Stopped"
    end
end)
