script_name("FAKE PC")
script_author("Shellder")

require "lib.samp.events"
require "moonloader"

local b = "0.3.7"
local PC_HASH = "3917818323CD3E248B7722EB5CBCED04CE5B4DBF67E"

function main()
    while not isSampAvailable() do
        wait(100)
    end
    
    while not sampIsLocalPlayerSpawned() do
        wait(100)
    end
    
    checkScriptName()
    sampAddChatMessage('[FAKE PC] | by Shellder', 0xFFFFFFFF)
    
    while true do
        wait(100)
    end
end

function onSendClientJoin(c, d, e, f, g, h, i)
    h = b
    g = PC_HASH
    d = 1
    return {c, d, e, f, g, h, i}
end

function onSendPlayerSync(k)
    if k.weapon == 44 or k.weapon == 45 or k.weapon == 46 then
        k.weapon = 0
    end

    if k.leftRightKeys ~= 0 and k.leftRightKeys ~= 128 and k.leftRightKeys ~= 65408 then
        if k.leftRightKeys > 0 then
            k.leftRightKeys = 128
        else
            k.leftRightKeys = 65408
        end
    end

    if k.upDownKeys ~= 0 and k.upDownKeys ~= 128 and k.upDownKeys ~= 65408 then
        if k.upDownKeys > 0 then
            k.upDownKeys = 128
        else
            k.upDownKeys = 65408
        end
    end

    if k.upDownKeys == 65408 or k.keysData == 4 then
        k.animationId = 0
    end

    if k.specialAction ~= nil and k.specialAction ~= 0 and k.specialAction ~= 1 then
        k.specialAction = 0
    end

    return k
end

function onSendVehicleSync(k)
    if k.leftRightKeys ~= 0 and k.leftRightKeys ~= 128 and k.leftRightKeys ~= 65408 then
        if k.leftRightKeys > 0 then
            k.leftRightKeys = 128
        else
            k.leftRightKeys = 65408
        end
    end
    return k
end

function onSendPassengerSync(k)
    if k.leftRightKeys ~= 0 and k.leftRightKeys ~= 128 and k.leftRightKeys ~= 65408 then
        if k.leftRightKeys > 0 then
            k.leftRightKeys = 128
        else
            k.leftRightKeys = 65408
        end
    end
    return k
end

function checkScriptName()
    local q = "FAKE PC by Shellder.lua"
    local r = thisScript().filename

    if r ~= q then
        sampAddChatMessage("NÃO RENOMEIE O SCRIPT! Nome correto: " .. q, 0xFFFF0000)
        thisScript():unload()
        return false
    end
    return true
end
