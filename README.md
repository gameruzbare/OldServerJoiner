
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local placeId = game.PlaceId

local function getOldServer()
    local cursor = ""
    while true do
        local success, response = pcall(function()
            return syn.request({
                Url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100&cursor=" .. cursor,
                Method = "GET"
            })
        end)
        if not success or not response or response.StatusCode ~= 200 then break end

        local data = HttpService:JSONDecode(response.Body)
        for _, server in ipairs(data.data) do
            if server.playing < server.maxPlayers and (not server.ping or server.ping > 100) then
                return server.id
            end
        end

        if data.nextPageCursor and type(data.nextPageCursor) == "string" then
            cursor = data.nextPageCursor
        else
            break
        end
    end
    return nil
end

local oldServerId = getOldServer()
if oldServerId then
    TeleportService:TeleportToPlaceInstance(placeId, oldServerId, player)
else
    warn("[OldServerJoiner] No suitable server found.")
end
