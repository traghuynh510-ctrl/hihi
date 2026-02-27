-- ==========================================
-- HỆ THỐNG TỰ ĐỘNG KHỞI CHẠY (OPTIMIZED)
-- ==========================================
repeat task.wait() until game:IsLoaded()

-- Tự động chọn phe Hải tặc
local function JoinTeam()
    local player = game:GetService("Players").LocalPlayer
    if player.Team == nil then
        local success, err = pcall(function()
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("SetTeam", "Pirates")
        end)
    end
end
JoinTeam()

-- Anti-AFK (Chống bị kick khi treo máy)
local VirtualUser = game:GetService("VirtualUser")
game:GetService("Players").LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- ==========================================
-- KHAI BÁO BIẾN (FIXED SYNTAX FOR DELTA)
-- ==========================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local lp = Players.LocalPlayer

-- ==========================================
-- CÀI ĐẶT TREO MÁY (ULTIMATE OPTIMIZED)
-- ==========================================
getgenv().Setting = {
    Bounty = { Min = 0, Max = 30000000 },
    Orbit = {
        Enabled = true,
        Radius = 10,
        Speed = 7,
        Height = 5
    },
    AutoBuff = {
        Haki = true,
        KenHaki = true,
        RaceV3 = true,
        RaceV4 = true
    },
    SafeMode = {
        SpamM1 = true,
        AutoEquipFruit = true,
        Noclip = true,
        Platform = true
    }
}

getgenv().Target = nil
local angle = 0

-- ==========================================
-- HÀM GIẢ LẬP NHẤN PHÍM
-- ==========================================
local function PressKey(key)
    pcall(function()
        VirtualInputManager:SendKeyEvent(true, key, false, game)
        task.wait(0.05)
        VirtualInputManager:SendKeyEvent(false, key, false, game)
    end)
end

-- ==========================================
-- TỰ ĐỘNG CẦM TRÁI ÁC QUỶ
-- ==========================================
local function EquipFruit()
    if not getgenv().Setting.SafeMode.AutoEquipFruit then return end
    local char = lp.Character
    if char and not char:FindFirstChildOfClass("Tool") then
        for _, tool in pairs(lp.Backpack:GetChildren()) do
            if tool:IsA("Tool") and (tool.ToolTip == "Blox Fruit" or tool.Name:find("Fruit")) then
                lp.Humanoid:EquipTool(tool)
                break
            end
        end
    end
end

-- ==========================================
-- HỆ THỐNG AUTO BUFF & TREO MÁY
-- ==========================================
task.spawn(function()
    while task.wait(1.5) do
        if getgenv().Target and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local char = lp.Character
            
            -- Tự bật Haki Vũ Trang (J)
            if getgenv().Setting.AutoBuff.Haki then
                if not char:FindFirstChild("HasBuso") and not char:FindFirstChild("SlayerVisual") then
                    PressKey("J")
                end
            end
            
            -- Tự bật Haki Quan Sát (E)
            if getgenv().Setting.AutoBuff.KenHaki then
                if not char:FindFirstChild("HasKen") then
                    PressKey("E")
                end
            end
            
            -- Tự bật Tộc V3 (T) và V4 (Y)
            if getgenv().Setting.AutoBuff.RaceV3 then PressKey("T") end
            if getgenv().Setting.AutoBuff.RaceV4 then PressKey("Y") end
        end
    end
end)

-- ==========================================
-- NỀN TẢNG CHỐNG RỚT BIỂN & NOCLIP
-- ==========================================
local plat = Instance.new("Part")
plat.Size = Vector3.new(12, 1, 12)
plat.Transparency = 1
plat.Anchored = true
plat.Parent = workspace

RunService.Stepped:Connect(function()
    local char = lp.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        -- Noclip xuyên tường
        if getgenv().Setting.SafeMode.Noclip then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") and v.CanCollide then
                    v.CanCollide = false
                end
            end
        end
        
        -- Giữ platform dưới chân
        if getgenv().Target and getgenv().Setting.SafeMode.Platform then
            local root = char:FindFirstChild("HumanoidRootPart")
            if root then 
                plat.CFrame = root.CFrame * CFrame.new(0, -3.5, 0) 
            end
        else
            plat.CFrame = CFrame.new(0, -1000, 0)
        end
    end
end)

print("--- SCRIPT HOÀN TẤT: ĐÃ SẴN SÀNG TREO FARM BOUNTY ---")
