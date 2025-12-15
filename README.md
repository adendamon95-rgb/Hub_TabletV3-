-- Hub_TabletV3 ONE-SCRIPT INSTALLER (NO ADMINS)
-- RUN ONCE IN ROBLOX STUDIO

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterPack = game:GetService("StarterPack")
local ServerScriptService = game:GetService("ServerScriptService")

-- =====================
-- RemoteEvent
-- =====================
local remote = Instance.new("RemoteEvent")
remote.Name = "HubTabletAction"
remote.Parent = ReplicatedStorage

-- =====================
-- Server Logic (NO ADMIN CHECK)
-- =====================
local server = Instance.new("Script")
server.Name = "HubTabletServer"
server.Parent = ServerScriptService

server.Source = [[
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remote = ReplicatedStorage:WaitForChild("HubTabletAction")

local function ragdoll(character)
    local hum = character:FindFirstChildOfClass("Humanoid")
    if not hum then return end

    hum:ChangeState(Enum.HumanoidStateType.Physics)

    for _, j in ipairs(character:GetDescendants()) do
        if j:IsA("Motor6D") then
            local a0 = Instance.new("Attachment", j.Part0)
            local a1 = Instance.new("Attachment", j.Part1)
            a0.CFrame = j.C0
            a1.CFrame = j.C1

            local socket = Instance.new("BallSocketConstraint")
            socket.Attachment0 = a0
            socket.Attachment1 = a1
            socket.Parent = j.Part0

            j:Destroy()
        end
    end
end

local function fling(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.AssemblyLinearVelocity = Vector3.new(
            math.random(-300,300),
            250,
            math.random(-300,300)
        )
    end
end

local ACTIONS = {
    Ragdoll = ragdoll,
    Fling = fling
}

remote.OnServerEvent:Connect(function(sender, action, targetUserId)
    local fn = ACTIONS[action]
    if not fn then return end

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.UserId == targetUserId and plr.Character then
            fn(plr.Character)
        end
    end
end)
]]

-- =====================
-- Tool (GIVEN TO EVERY PLAYER)
-- =====================
local tool = Instance.new("Tool")
tool.Name = "Hub_TabletV3"
tool.RequiresHandle = true
tool.Parent = StarterPack

local handle = Instance.new("Part")
handle.Name = "Handle"
handle.Size = Vector3.new(1,1,1)
handle.Parent = tool

-- =====================
-- GUI
-- =====================
local gui = Instance.new("ScreenGui")
gui.Name = "TabletGui"
gui.ResetOnSpawn = false
gui.Parent = tool

local frame = Instance.new("Frame", gui)
frame.Name = "MainFrame"
frame.Size = UDim2.fromScale(0.4,0.4)
frame.Position = UDim2.fromScale(0.3,0.3)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)

local box = Instance.new("TextBox", frame)
box.Name = "PlayerIdBox"
box.PlaceholderText = "Target UserId"
box.Size = UDim2.fromScale(1,0.15)
box.BackgroundColor3 = Color3.fromRGB(50,50,50)
box.TextColor3 = Color3.new(1,1,1)

local rag = Instance.new("TextButton", frame)
rag.Name = "RagdollBtn"
rag.Text = "Ragdoll"
rag.Position = UDim2.fromScale(0,0.25)
rag.Size = UDim2.fromScale(1,0.15)

local fling = Instance.new("TextButton", frame)
fling.Name = "FlingBtn"
fling.Text = "Fling"
fling.Position = UDim2.fromScale(0,0.45)
fling.Size = UDim2.fromScale(1,0.15)

-- =====================
-- Client Script
-- =====================
local client = Instance.new("LocalScript")
client.Parent = tool

client.Source = [[
local player = game.Players.LocalPlayer
local tool = script.Parent
local gui = tool:WaitForChild("TabletGui")
local remote = game.ReplicatedStorage:WaitForChild("HubTabletAction")

tool.Equipped:Connect(function()
    gui.Parent = player.PlayerGui
end)

tool.Unequipped:Connect(function()
    gui.Parent = tool
end)

local frame = gui.MainFrame
local box = frame.PlayerIdBox

frame.RagdollBtn.MouseButton1Click:Connect(function()
    local id = tonumber(box.Text)
    if id then
        remote:FireServer("Ragdoll", id)
    end
end)

frame.FlingBtn.MouseButton1Click:Connect(function()
    local id = tonumber(box.Text)
    if id then
        remote:FireServer("Fling", id)
    end
end)
]]

print("Hub_TabletV3 (NO ADMIN) INSTALLED")
