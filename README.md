repeat wait(0.25) until game:IsLoaded()
local player = game.Players.LocalPlayer
repeat wait() until player.Character
repeat wait() until player.Character:FindFirstChild("HumanoidRootPart")
repeat wait() until player.Character:FindFirstChild("Humanoid")
repeat wait() until workspace:FindFirstChild("__THINGS")
local function formatNumber(num)
    local formatted = tostring(num)
    while true do  
        formatted, k = string.gsub(formatted, "^(-?%d+)(%d%d%d)", '%1,%2')
        if (k == 0) then
            break
        end
    end
    return formatted
end
local function refreshLevel()
    return game:GetService("Players").LocalPlayer
        :WaitForChild("PlayerGui")
        :WaitForChild("Main")
        :WaitForChild("Bottom")
        :WaitForChild("Levels")
        :WaitForChild("RankHolder")
        :WaitForChild("RankName").Text
end
local function refreshData()
    local SaveModule = require(game.ReplicatedStorage.Library.Client.Save)
    local PetModule = require(game.ReplicatedStorage.Library.Directory.Pets)
    local saves = SaveModule:GetSaves()
    local Coins, Diamonds, TotalRolls, petList = 0, 0, 0, {}

    if type(saves) == "table" and saves[player] then
        local PlayerData = saves[player]
        
        if type(PlayerData) == "table" then
            if PlayerData.UpgradesPurchased then
                local upgradeCount = 0
		if #PlayerData.UpgradesPurchased > 0 then
		   upgradeCount = #PlayerData.UpgradesPurchased
		else
		   for _ in pairs(PlayerData.UpgradesPurchased) do
			upgradeCount = upgradeCount + 1
		   end
		end
		Upgrage = "Upgrade(" .. upgradeCount .. ")"
		table.insert(petList, Upgrage)
            end
            if PlayerData.TotalRolls then
                TotalRolls = PlayerData.TotalRolls
            end
            if PlayerData.Inventory then
                if PlayerData.Inventory.Currency then
                    for _, currency in pairs(PlayerData.Inventory.Currency) do
                        if currency.id == "Coins" then
                            Coins = currency._am
                        elseif currency.id == "Diamonds" then
                            Diamonds = currency._am
                        end
                    end
		    for _, pet in pairs(PlayerData.Inventory.Pet) do
			local petId = pet.id
			local quantity = pet._am or 1 
			local petData = PetModule[petId]
                        if petData then
                            local petDifficulty = petData.difficulty or 0
                            if string.find(petId, "Huge") or petDifficulty >= 1000000 then
                                table.insert(petList, petId .. " (" .. tostring(quantity) .. "/" .. formatNumber(petDifficulty) .. ")")
                            end
                        end
                    end
		    if #petList == 0 then
                        table.insert(petList, "0 pet ngon")
                    end
                end
            end
        end
    end

    return Coins, Diamonds, TotalRolls, petList
end

spawn(function()
    while true do
        local Coins, Diamonds, TotalRolls, petList = refreshData()
        local lvl = refreshLevel()
        local data = {
            ['Basic Data'] = {
                ['Level'] = lvl,
                ['Beli'] = Coins or 0,
                ['Fragments'] = Diamonds or 0,
                ['DevilFruit'] = 0,
                ['Race'] = TotalRolls or 0,
                ['Fighting Style'] = "Godhuman"
            },
            ['Items Inventory'] = petList or {},
            ['Fruits Inventory'] = {}
        }
        local a, b = pcall(function()
            writefile(string.format("%sData.json", player.Name), game:GetService("HttpService"):JSONEncode(data)) 
        end)

        if a then
            print(string.format("The file with name %sData.json has been written", player.Name))
        else
            warn("got error:", b)
        end
        wait(30)
    end
end)
