local Players = game:GetService('Players')
local Client = Players.LocalPlayer

_G.EnableFriendRequest = true
_G.EnableAutoAcceptFriendRequest = true
local interval = 60

local function SendFriendRequests()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Client then  
            Client:RequestFriendship(player)
            warn("Đã gửi yêu cầu kết bạn tới:", player.Name)
        end
        wait(10) 
    end
end

local function AutoAcceptFriendRequests()

    Client.FriendRequestEvent:Connect(function(requestingPlayer)
        if _G.EnableAutoAcceptFriendRequest then
            Client:RequestFriendship(requestingPlayer) 
            warn("Đã chấp nhận yêu cầu kết bạn từ:", requestingPlayer.Name)
        end
    end)
end

local function MainLoop()
    while true do
        if _G.EnableFriendRequest then
            SendFriendRequests()
        end
        wait(interval)
    end
end

task.spawn(function()
    AutoAcceptFriendRequests()
end)

task.spawn(function()
    MainLoop()
end)

Client.Idled:connect(function()
    local VirtualUser = game:GetService('VirtualUser')
    VirtualUser:Button2Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
    wait(1)
    VirtualUser:Button2Up(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
end)
