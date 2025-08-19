--// PvP Toolbar (Rayfield UI + Show Rayfield) — by BEBAO
--// Gói sẵn: Lock target, Spin mượt, Auto Jump, Auto Attack
--// Dán file này lên GitHub/Pastebin, rồi chạy:
--// loadstring(game:HttpGet("https://raw.githubusercontent.com/<user>/<repo>/main/pvp_toolbar_rayfield.lua"))()

--===== Load Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

--===== Window
local Window = Rayfield:CreateWindow({
   Name = "⚔️ PvP Toolbar",
   LoadingTitle = "PvP Panel Loading...",
   LoadingSubtitle = "by BEBAO",
   ConfigurationSaving = { Enabled = false },
   Discord = { Enabled = false },
   KeySystem = false
})

--===== Services & Vars
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local Char, Hum, HRP
local target = nil
local spinOn, jumpOn, attackOn = false, false, false

--===== Setup character
local function setupChar(c)
    Char = c
    Hum = Char:WaitForChild("Humanoid")
    HRP = Char:WaitForChild("HumanoidRootPart")
end
setupChar(LP.Character or LP.CharacterAdded:Wait())
LP.CharacterAdded:Connect(setupChar)

--===== Find nearest enemy (giữ đúng logic bản cũ)
local function getNearestEnemy()
    if not HRP then return nil end
    local best, dist = nil, math.huge
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local d = (HRP.Position - plr.Character.HumanoidRootPart.Position).Magnitude
            if d < dist then
                best, dist = plr, d
            end
        end
    end
    return best
end

--===== Lấy tool đang cầm hoặc tự equip
local function getTool()
    if not Char then return nil end
    local tool = Char:FindFirstChildOfClass("Tool")
    if tool then return tool end
    for _,t in ipairs(LP.Backpack:GetChildren()) do
        if t:IsA("Tool") then
            Hum:EquipTool(t)
            return t
        end
    end
end

--===== UI: Tabs & Controls
local mainTab = Window:CreateTab("Main", 4483362458)

local statusLabel = mainTab:CreateLabel("Status: Idle")

local lockBtn = mainTab:CreateButton({
    Name = "🎯 Lock Nearest / Unlock",
    Callback = function()
        if target then
            target = nil
            statusLabel:Set("Status: Unlock target")
            warn("Unlock target")
        else
            target = getNearestEnemy()
            if target then
                statusLabel:Set("Status: Locked -> "..target.Name)
                warn("Locked: "..target.Name)
            else
                statusLabel:Set("Status: No target")
            end
        end
    end
})

local spinToggle = mainTab:CreateToggle({
    Name = "🌀 Spin",
    CurrentValue = false,
    Flag = "spin_toggle",
    Callback = function(val)
        spinOn = val
        statusLabel:Set("Status: Spin "..tostring(val))
        warn("Spin: "..tostring(val))
    end
})

local jumpToggle = mainTab:CreateToggle({
    Name = "🦵 Auto Jump",
    CurrentValue = false,
    Flag = "jump_toggle",
    Callback = function(val)
        jumpOn = val
        statusLabel:Set("Status: Jump "..tostring(val))
        warn("Jump: "..tostring(val))
    end
})

local attackToggle = mainTab:CreateToggle({
    Name = "🗡️ Auto Attack",
    CurrentValue = false,
    Flag = "attack_toggle",
    Callback = function(val)
        attackOn = val
        statusLabel:Set("Status: Attack "..tostring(val))
        warn("Attack: "..tostring(val))
    end
})

mainTab:CreateDivider()
mainTab:CreateLabel("Tip: Nhấn nút Show Rayfield để hiện/ẩn menu")

--===== Main loop (kết hợp Lock + Spin mượt + Auto Jump + Auto Attack)
RunService.RenderStepped:Connect(function()
    if HRP and Hum then
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local enemyHRP = target.Character.HumanoidRootPart

            -- Nhìn về phía target (Lock nhìn)
            local look = (enemyHRP.Position - HRP.Position).Unit
            HRP.CFrame = CFrame.new(HRP.Position, HRP.Position + Vector3.new(look.X,0,look.Z))

            -- Auto Attack khi đủ gần
            if attackOn and (HRP.Position - enemyHRP.Position).Magnitude < 8 then
                local tool = getTool()
                if tool and tool:FindFirstChild("Activate") then
                    tool:Activate()
                end
            end

            -- Spin mượt quanh target
            if spinOn then
                local t = tick() * 4   -- tốc độ quay
                local r = 7            -- bán kính quay
                local offset = Vector3.new(math.cos(t),0,math.sin(t)) * r
                HRP.CFrame = CFrame.new(enemyHRP.Position + offset, enemyHRP.Position)
            end
        else
            -- Không có target mà vẫn bật Spin thì quay tại chỗ
            if spinOn then
                HRP.CFrame = HRP.CFrame * CFrame.Angles(0, math.rad(10), 0)
            end
        end

        -- Auto Jump khi đứng đất
        if jumpOn and Hum.FloorMaterial ~= Enum.Material.Air then
            Hum.Jump = true
        end
    end
end)

Rayfield:Notify({
    Title = "PvP Toolbar",
    Content = "Loaded • Lock • Spin • Jump • Attack",
    Duration = 4
})
