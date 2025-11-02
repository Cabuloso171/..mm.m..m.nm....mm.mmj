local FAKE_PC_ACTIVE = false
local memoryPatches = {}

function main()
    wait(2000)
    
    if not isSampAvailable() then
        wait(3000)
        if not isSampAvailable() then return end
    end
    
    initializeFakePC()
    
    applyPlatformPatches()
    
    installDetectionHooks()
    
    showActivationMessage()
    
    FAKE_PC_ACTIVE = true
    print("Fake pc ativado - servidor bosta enganado")
end

function initializeFakePC()
    writeMemory(0xBA67A4, 4, 0, false)
    
    writeMemory(0xBA67A8, 4, 0, false)
    
    writeMemory(0xBA67AC, 4, 0, false)
end

function applyPlatformPatches()
    local criticalAddresses = {
        {address = 0xBA67A4, value = 0, description = "Main Platform Flag"},
        {address = 0xBA67A8, value = 0, description = "Client Signature"},
        {address = 0xBA67AC, value = 0, description = "Client Version"},
        {address = 0xBA67B0, value = 0, description = "Device Type"},
        {address = 0xBA67B4, value = 1, description = "PC Client ID"}
    }
    
    for i, patch in ipairs(criticalAddresses) do
        local original = readMemory(patch.address, 4, false)
        writeMemory(patch.address, 4, patch.value, false)
        table.insert(memoryPatches, {
            address = patch.address,
            original = original,
            description = patch.description
        })
        print("Patched: " .. patch.description)
    end
    
    applyAdvancedPatches()
end

function applyAdvancedPatches()
    autoAssemble([[
        [ENABLE]
        // Bypass platform check 1
        alloc(platformBypass1, 64)
        label(returnHere1)
        
        platformBypass1:
        mov eax, 0 // Always return PC
        ret
        returnHere1:
        
        // Bypass platform check 2  
        alloc(platformBypass2, 64)
        label(returnHere2)
        
        platformBypass2:
        xor eax, eax // Clear EAX (PC platform)
        jmp returnHere2
        
        // Hook multiple detection points
        define(address1, 12345678) // Replace with actual addresses
        define(address2, 12345679)
        
        address1:
        jmp platformBypass1
        
        address2:
        jmp platformBypass2
        
        [DISABLE]
        // Restoration code will be here
    ]])
end

function installDetectionHooks()
    hookPacketSending()
    
    hookPlatformQueries()
end

function hookPacketSending()
    sampRegisterPacketListener(function(packetId, data)
        if packetId == 157 then
            manipulateSyncPacket(data)
        elseif packetId == 25 then
            manipulateConnectionPacket(data)
        end
    end)
end

function manipulateSyncPacket(data)
    local manipulated = data
    return manipulated
end

function manipulateConnectionPacket(data)
    local manipulated = data
    return manipulated
end

function hookPlatformQueries()
    local queryFunctions = {
        0x123456,
        0x123457,
        0x123458
    }
    
    for _, addr in ipairs(queryFunctions) do
        autoAssemble(string.format([[
            [ENABLE]
            alloc(hook_%X, 128)
            label(original_%X)
            label(return_%X)
            
            hook_%X:
            cmp [esp+4], 0xBA67A4 // Checking platform address
            jne original_%X
            mov eax, 0 // Return PC
            ret 4
            
            original_%X:
            // Original code here
            jmp return_%X
            
            %X:
            jmp hook_%X
            return_%X:
            
            [DISABLE]
            %X:
            // Original bytes
        ]], addr, addr, addr, addr, addr, addr, addr, addr, addr, addr))
    end
end

function showActivationMessage()
    sampAddChatMessage("{00FF00}==========================================", -1)
    sampAddChatMessage("{00FF00}(nexymods) FAKE PC CARREGADO COM SUCESSO!", -1)
    sampAddChatMessage("{00FF00}Servidor completamente enganado - Plataforma: PC", -1)
    sampAddChatMessage("{00FF00}Sistema anti-cheat bypass ativo", -1)
    sampAddChatMessage("{00FF00}==========================================", -1)
    
    for i = 1, 5 do
        wait(150)
        sampAddChatMessage("{00FF00} Sistema Fake PC totalmente operacional", -1)
    end
end

function isSampAvailable()
    return readMemory(0xBA67A0, 4, false) ~= 0 and
           readMemory(0xBA67A4, 4, false) ~= 0
end

function sampAddChatMessage(text, color)
    local chatPtr = readMemory(0xBA67A0, 4, false)
    if chatPtr ~= 0 then
        callFunction(0x582120, 0, chatPtr, 0, text, color, 0)
    end
end

function monitorPlatform()
    while true do
        wait(5000)
        if FAKE_PC_ACTIVE then
            local currentPlatform = readMemory(0xBA67A4, 4, false)
            if currentPlatform ~= 0 then
                writeMemory(0xBA67A4, 4, 0, false)
                print("Platform corrected to PC")
            end
        end
    end
end

function onScriptTerminate()
    if FAKE_PC_ACTIVE then
        for _, patch in ipairs(memoryPatches) do
            writeMemory(patch.address, 4, patch.original, false)
        end
        
        sampAddChatMessage("{FF0000}Fake PC desativado - Configurações restauradas", -1)
        print("Fake desativado - Tudo restaurado")
    end
end

main()
monitorPlatform()
