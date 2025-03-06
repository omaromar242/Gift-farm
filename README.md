local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local function ensureHoldDuration(playerModel)
    local humanoidRootPart = playerModel:FindFirstChild("HumanoidRootPart")
    if humanoidRootPart then
        local arrestPrompt = humanoidRootPart:FindFirstChild("ArrestPrompt")
        if arrestPrompt and arrestPrompt:IsA("ProximityPrompt") then
            arrestPrompt.HoldDuration = 1.2
        end
    end
end

local function monitorPlayers()
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if character then
            ensureHoldDuration(character)
        end
    end
end

RunService.RenderStepped:Connect(monitorPlayers)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        ensureHoldDuration(character)
    end)
end)
