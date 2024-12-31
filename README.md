local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")
local runService = game:GetService("RunService")
local workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local teleportService = game:GetService("TeleportService")

local placeId = game.PlaceId -- Current game Place ID

-- Create ScreenGui and Frame
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "GiftPartGui"
screenGui.Parent = player:WaitForChild("PlayerGui")

local blackFrame = Instance.new("Frame")
blackFrame.Name = "BlackFrame"
blackFrame.Size = UDim2.new(0.3, 0, 0.2, 0)
blackFrame.Position = UDim2.new(0.35, 0, 0.75, 0)
blackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
blackFrame.Parent = screenGui

-- Buttons
local autoButton = Instance.new("TextButton")
autoButton.Name = "AutoButton"
autoButton.Size = UDim2.new(0.3, 0, 1, 0)
autoButton.Position = UDim2.new(0, 0, 0, 0)
autoButton.BackgroundColor3 = Color3.new(0.2, 0.8, 0.2)
autoButton.Text = "Auto: OFF"
autoButton.TextColor3 = Color3.new(1, 1, 1)
autoButton.Font = Enum.Font.SourceSansBold
autoButton.TextScaled = true
autoButton.Parent = blackFrame

local pickButton = Instance.new("TextButton")
pickButton.Name = "PickButton"
pickButton.Size = UDim2.new(0.3, 0, 1, 0)
pickButton.Position = UDim2.new(0.35, 0, 0, 0)
pickButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
pickButton.Text = "Pick: OFF"
pickButton.TextColor3 = Color3.new(1, 1, 1)
pickButton.Font = Enum.Font.SourceSansBold
pickButton.TextScaled = true
pickButton.Parent = blackFrame

local giftButton = Instance.new("TextButton")
giftButton.Name = "GiftButton"
giftButton.Size = UDim2.new(0.3, 0, 1, 0)
giftButton.Position = UDim2.new(0.7, 0, 0, 0)
giftButton.BackgroundColor3 = Color3.new(0.8, 0.8, 0.2)
giftButton.Text = "Teleport to Gift"
giftButton.TextColor3 = Color3.new(1, 1, 1)
giftButton.Font = Enum.Font.SourceSansBold
giftButton.TextScaled = true
giftButton.Parent = blackFrame

local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0.3, 0, 1, 0)
closeButton.Position = UDim2.new(0.7, 0, 0, 0)
closeButton.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2)
closeButton.Text = "Close"
closeButton.TextColor3 = Color3.new(1, 1, 1)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextScaled = true
closeButton.Parent = blackFrame

-- Variables
local autoTeleportEnabled = false
local pickEnabled = false
local teleportInterval = 0.5
local gifts = {}

-- Update gift parts
local function updateGiftParts()
    gifts = {}
    for _, model in ipairs(workspace:GetChildren()) do
        if model:IsA("Model") then
            local gift = model:FindFirstChild("GiftPart")
            if gift and gift.Transparency == 0 then
                table.insert(gifts, gift)
            end
        end
    end
end

-- Auto teleport logic
local function autoTeleport()
    while autoTeleportEnabled do
        updateGiftParts()
        for _, gift in ipairs(gifts) do
            if gift.Transparency == 0 then
                rootPart.CFrame = gift.CFrame + Vector3.new(0, 3, 0)
                break
            end
        end
        task.wait(teleportInterval)
    end
end

-- Pick functionality
local function togglePickScript()
    pickEnabled = not pickEnabled
    pickButton.Text = "Pick: " .. (pickEnabled and "ON" or "OFF")

    if pickEnabled then
        _G.ProximityPromptCountdown = 1
        _G.AutoProximityPrompt = true -- Start with AutoProximityPrompt enabled

        local function fireproximityprompt(Obj, Amount, Skip)
            if Obj.ClassName == "ProximityPrompt" then
                Amount = Amount or 1
                local PromptTime = Obj.HoldDuration
                if Skip then
                    Obj.HoldDuration = 0
                end
                for i = 1, Amount do
                    Obj:InputHoldBegin()
                    if not Skip then
                        task.wait(Obj.HoldDuration)
                    end
                    Obj:InputHoldEnd()
                end
                Obj.HoldDuration = PromptTime
            else
                error("userdata<ProximityPrompt> expected")
            end
        end

        local function ProximityPromptCountdown()
            while pickEnabled do
                if _G.AutoProximityPrompt then
                    for _, v in pairs(workspace:GetDescendants()) do
                        if v:IsA("ProximityPrompt") then
                            fireproximityprompt(v, 1, true)
                        end
                    end
                end
                wait(_G.ProximityPromptCountdown)
            end
        end

        coroutine.wrap(ProximityPromptCountdown)()
    else
        _G.AutoProximityPrompt = false -- Stop the ProximityPrompt script
    end
end

-- Gift Teleport functionality
local function teleportToGift()
    updateGiftParts()
    if #gifts > 0 then
        local gift = gifts[math.random(1, #gifts)]
        rootPart.CFrame = gift.CFrame + Vector3.new(0, 3, 0)
    else
        print("No gift parts available")
    end
end

-- Button functionality
autoButton.MouseButton1Click:Connect(function()
    autoTeleportEnabled = not autoTeleportEnabled
    autoButton.Text = "Auto: " .. (autoTeleportEnabled and "ON" or "OFF")
    if autoTeleportEnabled then
        coroutine.wrap(autoTeleport)()
    end
end)

pickButton.MouseButton1Click:Connect(togglePickScript)

giftButton.MouseButton1Click:Connect(teleportToGift)

closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)
