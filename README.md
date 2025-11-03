local NexyCheats_Loaded = false
local NexyCheats_PlatformAddr = 0xBA67A4
local NexyCheats_SignatureAddr = 0xBA67A8
local NexyCheats_ChatPtr = 0xBA67A0

function NexyCheats_Main()
    wait(8000)
    
    if NexyCheats_ApplyFakePC() then
        NexyCheats_ShowSuccess()
        NexyCheats_Loaded = true
        NexyCheats_StartMonitor()
    end
end

function NexyCheats_ApplyFakePC()
    writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
    writeMemory(NexyCheats_SignatureAddr, 4, 0, false)
    
    wait(1000)
    
    local currentPlatform = readMemory(NexyCheats_PlatformAddr, 4, false)
    return currentPlatform == 0
end

function NexyCheats_ShowSuccess()
    local chat = readMemory(NexyCheats_ChatPtr, 4, false)
    if chat ~= 0 then
        lua_call(0x582120, 3, 0, chat, 0, "{00FF00}(NexyCheats) Fake PC carregado com sucesso!", -1, 0)
    end
end

function NexyCheats_StartMonitor()
    lua_thread.create(function()
        while true do
            wait(10000)
            if NexyCheats_Loaded then
                local platform = readMemory(NexyCheats_PlatformAddr, 4, false)
                if platform ~= 0 then
                    writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
                end
            end
        end
    end)
end

function onScriptTerminate(script, quitGame)
    if script == thisScript() and NexyCheats_Loaded then
        writeMemory(NexyCheats_PlatformAddr, 4, 1, false)
    end
end

lua_thread.create(function()
    NexyCheats_Main()
end)
