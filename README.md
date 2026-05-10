-- =========================
-- LÓGICA DAS FUNCIONALIDADES
-- (Cole isso no final do script, substituindo os hooks dos botões)
-- =========================

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- ========================= 
-- TELEPORT
-- =========================
btnTeleport.MouseButton1Click:Connect(function()
    -- Teleporta para o spawn principal do mapa
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local spawnLocations = workspace:GetDescendants()
        for _, obj in ipairs(spawnLocations) do
            if obj:IsA("SpawnLocation") then
                char.HumanoidRootPart.CFrame = obj.CFrame + Vector3.new(0, 5, 0)
                return
            end
        end
        -- Se não achar spawn, vai para 0,100,0
        char.HumanoidRootPart.CFrame = CFrame.new(0, 100, 0)
    end
end)

-- =========================
-- PEGA CAIXA (coleta partes próximas chamadas de "Caixa" ou "Box")
-- =========================
btnCaixa.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    local found = false
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("Model") then
            local name = obj.Name:lower()
            if name:find("caixa") or name:find("box") or name:find("chest") then
                local pos = obj:IsA("Model") 
                    and (obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart)
                    or obj
                if pos then
                    local dist = (rootPart.Position - pos.Position).Magnitude
                    if dist < 200 then
                        -- Teleporta até a caixa
                        rootPart.CFrame = CFrame.new(pos.Position + Vector3.new(0, 5, 0))
                        -- Tenta destruir/tocar a caixa
                        if obj:IsA("BasePart") then
                            obj:Destroy()
                        elseif obj:IsA("Model") then
                            obj:Destroy()
                        end
                        found = true
                        break
                    end
                end
            end
        end
    end
    if not found then
        print("[RENAN FRX] Nenhuma caixa encontrada próxima.")
    end
end)

-- =========================
-- NOCLIP
-- =========================
local noclipConnection = nil
local _, getNoclip = toggleNoclip  -- estado do toggle

-- Recria o toggle com callback
local noclipFrame = toggleNoclip
local noclipActive = false

-- Sobrescreve o switch do noclip
local noclipSwitch = noclipFrame:FindFirstChildWhichIsA("TextButton")
if noclipSwitch then
    noclipSwitch.MouseButton1Click:Connect(function()
        noclipActive = not noclipActive
        if noclipActive then
            noclipConnection = RunService.Stepped:Connect(function()
                local char = LocalPlayer.Character
                if char then
                    for _, part in ipairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end
                end
            end)
        else
            if noclipConnection then
                noclipConnection:Disconnect()
                noclipConnection = nil
            end
            -- Reativa colisão
            local char = LocalPlayer.Character
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end)
end

-- =========================
-- FLY
-- =========================
local flyConnection = nil
local flyActive = false
local flyFrame = toggleFly

local flySwitch = flyFrame:FindFirstChildWhichIsA("TextButton")
if flySwitch then
    flySwitch.MouseButton1Click:Connect(function()
        flyActive = not flyActive
        if flyActive then
            local char = LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildWhichIsA("Humanoid")
            if not hrp or not hum then flyActive = false return end

            hum.PlatformStand = true

            local bodyVel = Instance.new("BodyVelocity")
            bodyVel.Velocity = Vector3.zero
            bodyVel.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            bodyVel.Parent = hrp

            local bodyGyro = Instance.new("BodyGyro")
            bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
            bodyGyro.P = 1e4
            bodyGyro.CFrame = hrp.CFrame
            bodyGyro.Parent = hrp

            flyConnection = RunService.RenderStepped:Connect(function()
                if not flyActive then
                    bodyVel:Destroy()
                    bodyGyro:Destroy()
                    if hum then hum.PlatformStand = false end
                    flyConnection:Disconnect()
                    return
                end

                local cam = workspace.CurrentCamera
                local dir = Vector3.zero

                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    dir = dir + cam.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    dir = dir - cam.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    dir = dir - cam.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    dir = dir + cam.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    dir = dir + Vector3.new(0, 1, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    dir = dir - Vector3.new(0, 1, 0)
                end

                bodyVel.Velocity = dir.Magnitude > 0 
                    and dir.Unit * flySpeed 
                    or Vector3.zero

                bodyGyro.CFrame = cam.CFrame
            end)
        else
            flyActive = false
        end
    end)
end

print("[RENAN FRX] Lógica carregada com sucesso.")
