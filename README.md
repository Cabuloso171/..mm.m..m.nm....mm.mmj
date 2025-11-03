local NexyCheats_Loaded = false
local NexyCheats_PlatformAddr = 0xBA67A4
local NexyCheats_SignatureAddr = 0xBA67A8
local NexyCheats_ChatPtr = 0xBA67A0

function NexyCheats_Main()
    wait(8000)
    
    -- Verificar se os endereços de memória são válidos
    if not NexyCheats_ValidateAddresses() then
        print("Erro: Endereços de memória inválidos")
        return
    end
    
    if NexyCheats_ApplyFakePC() then
        NexyCheats_ShowSuccess()
        NexyCheats_Loaded = true
        NexyCheats_StartMonitor()
    else
        print("Erro: Falha ao aplicar Fake PC")
    end
end

function NexyCheats_ValidateAddresses()
    -- Tentativa segura de leitura para verificar se os endereços são válidos
    local success, result = pcall(function()
        return readMemory(NexyCheats_PlatformAddr, 4, false)
    end)
    return success and result ~= nil
end

function NexyCheats_ApplyFakePC()
    local success = pcall(function()
        writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
        writeMemory(NexyCheats_SignatureAddr, 4, 0, false)
    end)
    
    if not success then
        return false
    end
    
    wait(1000)
    
    local success, currentPlatform = pcall(function()
        return readMemory(NexyCheats_PlatformAddr, 4, false)
    end)
    
    return success and currentPlatform == 0
end

function NexyCheats_ShowSuccess()
    local success, chat = pcall(function()
        return readMemory(NexyCheats_ChatPtr, 4, false)
    end)
    
    if success and chat ~= 0 then
        local callSuccess = pcall(function()
            lua_call(0x582120, 3, 0, chat, 0, "{00FF00}(NexyCheats) Fake PC carregado com sucesso!", -1, 0)
        end)
        
        if not callSuccess then
            print("Aviso: Não foi possível exibir mensagem no chat")
        end
    end
end

function NexyCheats_StartMonitor()
    lua_thread.create(function()
        while true do
            wait(10000)
            if NexyCheats_Loaded then
                local success, platform = pcall(function()
                    return readMemory(NexyCheats_PlatformAddr, 4, false)
                end)
                
                if success and platform ~= 0 then
                    local writeSuccess = pcall(function()
                        writeMemory(NexyCheats_PlatformAddr, 4, 0, false)
                    end)
                    
                    if not writeSuccess then
                        print("Erro no monitor: Falha ao escrever na memória")
                        break
                    end
                elseif not success then
                    print("Erro no monitor: Falha ao ler memória")
                    break
                end
            else
                break
            end
        end
    end)
end

function onScriptTerminate(script, quitGame)
    if script == thisScript() and NexyCheats_Loaded then
        pcall(function()
            writeMemory(NexyCheats_PlatformAddr, 4, 1, false)
        end)
    end
end

-- Inicialização segura
local mainSuccess, mainError = pcall(function()
    lua_thread.create(function()
        NexyCheats_Main()
    end)
end)

if not mainSuccess then
    print("Erro crítico na inicialização: " .. tostring(mainError))
end
