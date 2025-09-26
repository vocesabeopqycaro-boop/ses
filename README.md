-- ServerScriptService/onehit_dev_mode.lua
local Players = game:GetService("Players")
local CollectionService = game:GetService("CollectionService")
local RunService = game:GetService("RunService")

-- IDs dos developers que terão o poder de one-hit (substitua pelos seus UserIds)
local DEV_USER_IDS = {
    12345678, -- seu UserId aqui
    -- adicione outros se quiser
}

-- Tag opcional para marcar NPCs que NÃO queres afetar
local IGNORE_TAG = "IgnoreOneHit"

-- Guarda os MaxHealth originais para poder restaurar (se necessário)
local originalMaxHealth = {}

local function isDevPlayer(player)
    for _, id in ipairs(DEV_USER_IDS) do
        if player.UserId == id then
            return true
        end
    end
    return false
end

-- Checa se o modelo é um NPC (tem Humanoid e não é personagem de jogador)
local function isNPC(model)
    if typeof(model) ~= "Instance" then return false end
    local humanoid = model:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    -- se tiver um Player com esse modelo, então não é NPC
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.Character == model then
            return false
        end
    end
    return true
end

local function setOneHit(humanoid)
    if not humanoid or not humanoid:IsA("Humanoid") then return end
    if humanoid:GetAttribute("OneHitApplied") then return end
    -- salva original (se ainda não salvo)
    if originalMaxHealth[humanoid] == nil then
        originalMaxHealth[humanoid] = humanoid.MaxHealth
    end
    humanoid.MaxHealth = 1
    if humanoid.Health > 1 then
        humanoid.Health = 1
    end
    humanoid:SetAttribute("OneHitApplied", true)
end

local function restoreHumanoid(humanoid)
    if not humanoid or not humanoid:IsA("Humanoid") then return end
    if originalMaxHealth[humanoid] then
        humanoid.MaxHealth = originalMaxHealth[humanoid]
        if humanoid.Health > humanoid.MaxHealth then
            humanoid.Health = humanoid.MaxHealth
        end
        originalMaxHealth[humanoid] = nil
    end
    humanoid:SetAttribute("OneHitApplied", nil)
end

-- Aplica a todos NPCs já existentes (exceto marcados)
local function applyToExistingNPCs()
    for _, inst in ipairs(workspace:GetDescendants()) do
        if isNPC(inst) and not CollectionService:HasTag(inst, IGNORE_TAG) then
            local h = inst:FindFirstChildOfClass("Humanoid")
            if h then setOneHit(h) end
        end
    end
end

-- Monitora novos NPCs que aparecerem
local function monitorNewNPCs()
    workspace.DescendantAdded:Connect(function(desc)
        -- atraso curto para componentes serem montados
        if desc:IsA("Model") then
            if isNPC(desc) and not CollectionService:HasTag(desc, IGNORE_TAG) then
                local h = desc:FindFirstChildOfClass("Humanoid")
                if h then
                    setOneHit(h)
                else
                    -- se o humanoid for adicionado depois
                    desc.ChildAdded:Connect(function(child)
                        if child:IsA("Humanoid") then
                            if isNPC(desc) and not CollectionService:HasTag(desc, IGNORE_TAG) then
                                setOneHit(child)
                            end
                        end
                    end)
                end
            end
        elseif desc:IsA("Humanoid") then
            local model = desc.Parent
            if isNPC(model) and not CollectionService:HasTag(model, IGNORE_TAG) then
                setOneHit(desc)
            end
        end
    end)
end

-- Habilita one-hit mode (chamado quando um dev entra)
local function enableOneHitMode()
    applyToExistingNPCs()
    monitorNewNPCs()
end

-- Quando jogador entra, se for dev, ativa o modo
Players.PlayerAdded:Connect(function(player)
    -- pequeno delay para garantir que a lista de players esteja atualizada
    wait(1)
    if isDevPlayer(player) then
        enableOneHitMode()
    end
end)

-- Se já houver players (com reloads) checa se algum dev já está presente
for _, p in ipairs(Players:GetPlayers()) do
    if isDevPlayer(p) then
        enableOneHitMode()
        break
    end
end
