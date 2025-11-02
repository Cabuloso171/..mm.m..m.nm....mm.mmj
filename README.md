local NexyCheats_Loaded = false
local NexyCheats_PlatformAddr = 0xBA67A4
local NexyCheats_SignatureAddr = 0xBA67A8
local NexyCheats_ChatPtr = 0xBA67A0

function NexyCheats_Main()
    local waitCount = 0
    while waitCount < 50 do
        if readMemory(NexyCheats_ChatPtr, 4, false) ~= 0 then
            break
        end
        wait(100)
        waitCount = waitCount + 1
    end
    
    if NexyCheats_ApplyFakePC() then
        NexyCheats_ShowSuccess()
        NexyCheats_Loaded = true
        NexyCheats_StartMonitor()
    end
end

function NexyCheats_ApplyFakePC()
    writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
    writeMemory(NexyCheats_SignatureAddr, 4, 0, false)
    
    wait(500)
    
    local currentPlatform = readMemory(NexyCheats_PlatformAddr, 4, false)
    if currentPlatform == 0 then
        return true
    end
    
    return false
end

function NexyCheats_ShowSuccess()
    local chat = readMemory(NexyCheats_ChatPtr, 4, false)
    if chat ~= 0 then
        lua_call(0x582120, 3, 0, chat, 0, "{00FF00}(NexyCheats) Fake PC carregado com sucesso!", -1, 0)
        wait(200)
        lua_call(0x582120, 3, 0, chat, 0, "{00FF00}Arenas PC liberadas - Conectado como PC", -1, 0)
    end
end

function NexyCheats_StartMonitor()
    lua_thread.create(function()
        while true do
            wait(8000)
            if NexyCheats_Loaded then
                local platform = readMemory(NexyCheats_PlatformAddr, 4, false)
                if platform ~= 0 then
                    writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
                end
            end
        end
    end)
end

function NexyCheats_OnTerminate()
    if NexyCheats_Loaded then
        writeMemory(NexyCheats_PlatformAddr, 4, 1, false)
        local chat = readMemory(NexyCheats_ChatPtr, 4, false)
        if chat ~= 0 then
            lua_call(0x582120, 3, 0, chat, 0, "{FF0000}Fake PC NexyCheats desativado", -1, 0)
        end
    end
end

lua_thread.create(function()
    wait(3500)
    NexyCheats_Main()
end)
