repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local lp = Players.LocalPlayer

-- ================= SETTINGS (ULTIMATE OPTIMIZED) =================
getgenv().Setting = {
    Bounty = { Min = 0, Max = 30000000 },
    
    Orbit = {
        Enabled = true,
        Radius = 10,       -- Khoảng cách quay quanh đối thủ
        Speed = 7,        -- Tốc độ quay (an toàn)
        Height = 5         -- Độ cao so với mục tiêu
    },

    AutoBuff = {
        Haki = true,        -- Tự bật J
        KenHaki = true,     -- Tự bật E
        RaceV3 = true,      -- Tự bật T
        RaceV4 = true       -- Tự bật Y
    },

    SafeMode = {
        SpamM1 = true,      -- Spam chuột trái
        AutoEquipFruit = true, -- Tự cầm Trái Ác Quỷ (Blox Fruit)
        Noclip = true,      -- Đi xuyên tường
        Platform = true     -- Nền đứng chống rớt biển
    }
}

getgenv().Target = nil
local angle = 0

-- ================= HÀM GIẢ LẬP NHẤN PHÍM =================
local function PressKey(key)
    pcall(function()
        VirtualInputManager:SendKeyEvent(true, key, false, game)
        task.wait(0.05)
        VirtualInputManager:SendKeyEvent(false, key, false, game)
    end)
end

-- ================= TỰ ĐỘNG CẦM TRÁI ÁC QUỶ =================
local function EquipFruit()
    if not getgenv().Setting.SafeMode.AutoEquipFruit then return end
    local char = lp.Character
    if char and not char:FindFirstChildOfClass("Tool") then
        for _, tool in pairs(lp.Backpack:GetChildren()) do
            -- Kiểm tra ToolTip hoặc tên chứa "Fruit"
            if tool:IsA("Tool") and (tool.ToolTip == "Blox Fruit" or tool.Name:find("Fruit")) then
                lp.Humanoid:EquipTool(tool)
                break
            end
        end
    end
end

-- ================= HỆ THỐNG AUTO BUFF (TỐI ƯU) =================
task.spawn(function()
    while task.wait(1.5) do 
        if getgenv().Target and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local char = lp.Character
            
            -- Bật Haki Vũ Trang (J)
            if getgenv().Setting.AutoBuff.Haki then
                if not char:FindFirstChild("HasBuso") and not char:FindFirstChild("SlayerVisual") then
                    PressKey("J")
                end
            end
            
            -- Bật Haki Quan Sát (E)
            if getgenv().Setting.AutoBuff.KenHaki then
                if not char:FindFirstChild("HasKen") then
                    PressKey("E")
                end
            end

            -- Spam Tộc V3 (T) và V4 (Y) khi có nộ/hồi chiêu
            if getgenv().Setting.AutoBuff.RaceV3 then PressKey("T") end
            if getgenv().Setting.AutoBuff.RaceV4 then PressKey("Y") end
        end
    end
end)

-- ================= TẠO NỀN TẢNG CHỐNG RỚT BIỂN =================
local plat = Instance.new("Part")
plat.Size = Vector3.new(12, 1, 12)
plat.Transparency = 1
plat.Anchored = true
plat.Parent = workspace

-- ================= NOCLIP & PLATFORM LOGIC =================
RunService.Stepped:Connect(function()
    local char = lp.Character
    if char then
        -- Noclip xuyên tường
        if getgenv().Setting.SafeMode.Noclip then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") and v.CanCollide then 
                    v.CanCollide = false 
                end
            end
        end
        -- Platform bám theo chân
        if getgenv().Target and getgenv().Setting.SafeMode.Platform then
            local root = char:FindFirstChild("HumanoidRootPart")
            if root then plat.CFrame = root.CFrame * CFrame.new(0, -3.5, 0) end
        else
            plat.CFrame = CFrame.new(0, -1000, 0)
        end
    end
end)

-- ================= TÌM KIẾM MỤC TIÊU GẦN NHẤT =================
task.spawn(function()
    while task.wait(1) do
        local closest, shortest = nil, math.huge
        local myRoot = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
        
        if myRoot then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= lp and plr.Team ~= lp.Team then
                    local char = plr.Character
                    local hum = char and char:FindFirstChildOfClass("Humanoid")
                    local root = char and char:FindFirstChild("HumanoidRootPart")
                    
                    if hum and hum.Health > 0 and root then
                        local b = (char:FindFirstChild("leaderstats") and (char.leaderstats:FindFirstChild("Bounty") or char.leaderstats:FindFirstChild("Honor")))
                        if b and b.Value >= getgenv().Setting.Bounty.Min then
                            local dist = (root.Position - myRoot.Position).Magnitude
                            if dist < shortest then
                                shortest = dist
                                closest = plr
                            end
                        end
                    end
                end
            end
        end
        getgenv().Target = closest
    end
end)

-- ================= VÒNG LẶP CHIẾN ĐẤU (ORBIT + M1) =================
task.spawn(function()
    while true do
        task.wait() 
        if getgenv().Target and getgenv().Target.Character then
            local tRoot = getgenv().Target.Character:FindFirstChild("HumanoidRootPart")
            local myRoot = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
            
            if tRoot and myRoot then
                -- Tính toán vị trí Orbit (Quay vòng tròn)
                angle = angle + (getgenv().Setting.Orbit.Speed / 100)
                local offset = Vector3.new(
                    math.cos(angle) * getgenv().Setting.Orbit.Radius,
                    getgenv().Setting.Orbit.Height,
                    math.sin(angle) * getgenv().Setting.Orbit.Radius
                )
                
                -- Khóa CFrame vào mục tiêu
                myRoot.CFrame = CFrame.new(tRoot.Position + offset, tRoot.Position)
                
                -- Tự cầm Trái Ác Quỷ
                EquipFruit()
                
                -- Siêu Spam chuột trái (M1)
                if getgenv().Setting.SafeMode.SpamM1 then
                    game:GetService("VirtualUser"):CaptureController()
                    game:GetService("VirtualUser"):Button1Down(Vector2.new(0,0))
                end
            end
        end
    end
end)

print("--- Ultimate Fruit Hunter v3.0 Optimized ---")# hihi
Bountyhunter 
