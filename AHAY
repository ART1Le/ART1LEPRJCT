--==================================================
-- WINDUI LIBRARY
--==================================================
local WindUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/ART1Le/ART1LEL1B/refs/heads/main/dist/main.lua"))()

local Window = WindUI:CreateWindow({
    Title = "ART1LE PROJECT",
    Icon = "rbxassetid://10747363809",
    Author = "by Artile",
    Transparent = true,
    Folder = "ART1LEHUB",
    User = {
        Enabled = true,
        Anonymous = false,
        Callback = function()
            print("clicked")
        end,
    },
})

local function Notify(t, c, d)
    WindUI:Notify({
        Title = t,
        Content = c,
        Duration = d or 3
    })
end

--==================================================
-- SERVICES & GLOBALS
--==================================================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local StarterGui = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local BloxbizRemotes = ReplicatedStorage:WaitForChild("BloxbizRemotes", 5)
local CatalogRemote = BloxbizRemotes and BloxbizRemotes:WaitForChild("CatalogOnApplyOutfit", 5)

-- State Globals
local SelectedPlayerName = nil
local MainTargetPlayer = nil
local SearchInputVal = ""

-- Follow Globals
local FollowTargetEnabled = false
local FollowConnection = nil
local FollowAnimTrack = nil

-- Movement States
_G.InfJump = false
_G.NoClip = false
_G.Fly = false
local FlySpeed = 50
local FlyGyro = nil
local FlyVelocity = nil

-- ESP Globals
local ESP_Enabled = false
local ESP_Mode = "Username"
local ESP_Objects = {}
local ESP_Color = Color3.fromRGB(25, 212, 209) 

-- Freecam Globals
local FreecamEnabled = false
local FreecamPart = nil
local FreecamSpeed = 1
local TargetFOV = 70
local RotationX = 0
local RotationY = 0
local SmoothRotX = 0
local SmoothRotY = 0
local CinematicSmoothness = 0.08 -- Professional Damping

-- Advanced Lock Globals (NEW FEATURES)
local LockFree_Enabled = false
local LockFree_Target = nil
local LockFree_Mode = "OFF"
local LockFree_Offset = 10

local CenterFree_Enabled = false
local CenterFree_A = nil
local CenterFree_B = nil
local CenterFree_Mode = "OFF"
local CenterFree_Offset = 15

--==================================================
-- 🛠️ UTILS & MOVEMENT LOGIC
--==================================================
local function GetPlayerList(filter)
    local t = {}
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer then
            if not filter or v.Name:lower():find(filter:lower()) or v.DisplayName:lower():find(filter:lower()) then
                table.insert(t, v.Name)
            end
        end
    end
    return t
end

-- Fly Logic (KAKU & TEGAK)
RunService.RenderStepped:Connect(function()
    if _G.Fly and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = LocalPlayer.Character.HumanoidRootPart
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")

        if not FlyGyro then
            FlyGyro = Instance.new("BodyGyro", hrp)
            FlyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            FlyGyro.P = 9000
        end
        FlyGyro.CFrame = Camera.CFrame

        if not FlyVelocity then
            FlyVelocity = Instance.new("BodyVelocity", hrp)
            FlyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            FlyVelocity.Velocity = Vector3.new(0, 0, 0)
        end

        local moveDir = hum.MoveDirection
        local flyVec = moveDir * FlySpeed
        local verticalValue = 0

        if UserInputService:IsKeyDown(Enum.KeyCode.E) then verticalValue = FlySpeed end
        if UserInputService:IsKeyDown(Enum.KeyCode.Q) then verticalValue = -FlySpeed end

        FlyVelocity.Velocity = Vector3.new(flyVec.X, verticalValue, flyVec.Z)
        hum.PlatformStand = true
    else
        if FlyGyro then FlyGyro:Destroy() FlyGyro = nil end
        if FlyVelocity then FlyVelocity:Destroy() FlyVelocity = nil end
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").PlatformStand = false
        end
    end
end)

-- NoClip Logic
RunService.Stepped:Connect(function()
    if (_G.NoClip or FollowTargetEnabled) and LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

-- InfJump Logic
UserInputService.JumpRequest:Connect(function()
    if _G.InfJump and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

--==================================================
-- 👁️ ESP LOGIC
--==================================================
local function ClearESP()
    for _, obj in pairs(ESP_Objects) do if obj then obj:Destroy() end end
    ESP_Objects = {}
end

local function CreateESP(plr)
    if plr == LocalPlayer or not plr.Character then return end
    local head = plr.Character:FindFirstChild("Head")
    if not head then return end

    local bill = Instance.new("BillboardGui", head)
    bill.Name = "ART1LE_ESP"
    bill.AlwaysOnTop = true
    bill.Size = UDim2.new(0, 100, 0, 30)
    bill.StudsOffset = Vector3.new(0, 2.5, 0)

    local txt = Instance.new("TextLabel", bill)
    txt.BackgroundTransparency = 1
    txt.Size = UDim2.new(1, 0, 1, 0)
    txt.Font = Enum.Font.GothamBold
    txt.TextSize = 13
    txt.TextStrokeTransparency = 0.5
    txt.TextColor3 = ESP_Color

    if ESP_Mode == "Username" then
        txt.Text = plr.Name
    elseif ESP_Mode == "Display" then
        txt.Text = plr.DisplayName
    elseif ESP_Mode == "Both" then
        txt.Text = plr.DisplayName .. " (@" .. plr.Name .. ")"
    end
    ESP_Objects[plr] = bill
end

local function RefreshESP()
    ClearESP()
    if not ESP_Enabled then return end
    for _, p in ipairs(Players:GetPlayers()) do CreateESP(p) end
end

--==================================================
-- 🎥 FREECAM LOGIC
--==================================================
local function ToggleFreecam(state)
    FreecamEnabled = state
    StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.All, not state)

    for _, gui in pairs(LocalPlayer:WaitForChild("PlayerGui"):GetChildren()) do
        if gui:IsA("ScreenGui") and gui.Name ~= "WindUI" then
            gui.Enabled = not state
        end
    end
    
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.Anchored = state 
    end

    if state then
        local camCFrame = Camera.CFrame
        local x, y, z = camCFrame:ToEulerAnglesYXZ()
        RotationX = math.deg(x)
        RotationY = math.deg(y)
        SmoothRotX = RotationX
        SmoothRotY = RotationY
        TargetFOV = 70
        
        FreecamPart = Instance.new("Part")
        FreecamPart.Name = "FreecamPart"
        FreecamPart.Transparency = 1
        FreecamPart.CanCollide = false
        FreecamPart.Anchored = true
        FreecamPart.CFrame = camCFrame
        FreecamPart.Parent = workspace
        
        Camera.CameraType = Enum.CameraType.Scriptable
    else
        if FreecamPart then FreecamPart:Destroy() FreecamPart = nil end
        Camera.CameraType = Enum.CameraType.Custom
        Camera.FieldOfView = 70
        Camera.CameraSubject = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    end
    UserInputService.MouseBehavior = Enum.MouseBehavior.Default
end

UserInputService.InputChanged:Connect(function(input)
    if FreecamEnabled then
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                UserInputService.MouseBehavior = Enum.MouseBehavior.LockCurrentPosition
                local delta = input.Delta
                RotationX = math.clamp(RotationX - delta.Y * 0.25, -85, 85)
                RotationY = RotationY - delta.X * 0.25
            else
                UserInputService.MouseBehavior = Enum.MouseBehavior.Default
            end
        elseif input.UserInputType == Enum.UserInputType.MouseWheel then
            TargetFOV = math.clamp(TargetFOV - (input.Position.Z * 5), 5, 120)
        end
    end
end)

UserInputService.InputBegan:Connect(function(input)
    if FreecamEnabled and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        if input.KeyCode == Enum.KeyCode.Up then
            FreecamSpeed = math.clamp(FreecamSpeed + 0.2, 0.1, 15)
            Notify("Speed Update", "Freecam Speed: " .. string.format("%.1f", FreecamSpeed), 1)
        elseif input.KeyCode == Enum.KeyCode.Down then
            FreecamSpeed = math.clamp(FreecamSpeed - 0.2, 0.1, 15)
            Notify("Speed Update", "Freecam Speed: " .. string.format("%.1f", FreecamSpeed), 1)
        end
    end
end)

--==================================================
-- 🔥 CORE LOGIC: AVATAR STEALER
--==================================================
local function ApplyAvatarFromDescription(desc, targetName)
    if not CatalogRemote or not desc then return end
    local args = {{ 
        Accessories = {}, Head = desc.Head, LeftArm = desc.LeftArm, RightArm = desc.RightArm,
        LeftLeg = desc.LeftLeg, RightLeg = desc.RightLeg, Torso = desc.Torso, Face = desc.Face,
        Shirt = desc.Shirt, Pants = desc.Pants, GraphicTShirt = desc.GraphicTShirt,
        BodyTypeScale = desc.BodyTypeScale, DepthScale = desc.DepthScale, HeightScale = desc.HeightScale,
        WidthScale = desc.WidthScale, ProportionScale = desc.ProportionScale, HeadScale = desc.HeadScale,
        LeftArmColor = desc.LeftArmColor, RightArmColor = desc.RightArmColor, LeftLegColor = desc.LeftLegColor,
        RightLegColor = desc.RightLegColor, TorsoColor = desc.TorsoColor, HeadColor = desc.HeadColor,
        IdleAnimation = desc.IdleAnimation, RunAnimation = desc.RunAnimation, WalkAnimation = desc.WalkAnimation,
        JumpAnimation = desc.JumpAnimation, ClimbAnimation = desc.ClimbAnimation, FallAnimation = desc.FallAnimation,
        SwimAnimation = desc.SwimAnimation, MoodAnimation = desc.MoodAnimation
    }}
    pcall(function() 
        local accs = desc:GetAccessories(true)
        for _, v in ipairs(accs) do 
            table.insert(args[1].Accessories, {
                AssetId = v.AssetId, 
                AccessoryType = v.AccessoryType, 
                IsLayered = v.IsLayered, 
                Order = v.Order, 
                Puffiness = v.Puffiness, 
                Position = v.Position or Vector3.zero, 
                Rotation = v.Rotation or Vector3.zero, 
                Scale = v.Scale or Vector3.one
            }) 
        end 
    end)
    pcall(function() CatalogRemote:FireServer(unpack(args)) end)
    Notify("Sukses", "Avatar copy dari: " .. tostring(targetName))
end

--==================================================
-- 🏠 MAIN TAB
--==================================================
local MainTab = Window:Tab({ Title = "MAIN", Icon = "rbxassetid://10723407389" })
MainTab:Section({ Title = "Player Selector" })
local MainPlayerDrop
MainTab:Input({
    Title = "Search Player",
    Placeholder = "Type username...",
    Callback = function(t)
        if MainPlayerDrop then MainPlayerDrop:Refresh(GetPlayerList(t)) end
    end
})
MainPlayerDrop = MainTab:Dropdown({
    Title = "Select Target Player",
    Values = GetPlayerList(),
    Callback = function(v) MainTargetPlayer = v end
})
MainTab:Button({
    Title = "Teleport to Target",
    Callback = function()
        if MainTargetPlayer then
            local target = Players:FindFirstChild(MainTargetPlayer)
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                LocalPlayer.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame
                Notify("Teleport", "Teleported to " .. MainTargetPlayer)
            end
        end
    end
})

-- FITUR FOLLOW TARGET
MainTab:Toggle({
    Title = "Follow Target (Back Position)",
    Callback = function(v)
        FollowTargetEnabled = v
        if FollowConnection then FollowConnection:Disconnect() FollowConnection = nil end
        if FollowAnimTrack then FollowAnimTrack:Stop() FollowAnimTrack = nil end
        
        if v then
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            local anim = Instance.new("Animation")
            anim.AnimationId = "rbxassetid://74146582057902"
            FollowAnimTrack = hum:LoadAnimation(anim)
            FollowAnimTrack.Looped = true
            FollowAnimTrack:Play()

            FollowConnection = RunService.RenderStepped:Connect(function()
                if not FollowTargetEnabled then return end
                local target = Players:FindFirstChild(MainTargetPlayer)
                if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local targetHRP = target.Character.HumanoidRootPart
                    local myHRP = LocalPlayer.Character.HumanoidRootPart
                    myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 1.8, 0.5)
                    myHRP.Velocity = Vector3.new(0, 0, 0)
                end
            end)
        end
    end
})

MainTab:Toggle({
    Title = "SPY Player",
    Callback = function(v)
        if v and MainTargetPlayer then
            local target = Players:FindFirstChild(MainTargetPlayer)
            if target and target.Character then Camera.CameraSubject = target.Character:FindFirstChildOfClass("Humanoid") end
        else
            Camera.CameraSubject = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        end
    end
})
MainTab:Section({ Title = "Movement & Visual" })
MainTab:Toggle({
    Title = "Enable Name ESP",
    Callback = function(v) ESP_Enabled = v RefreshESP() end
})
MainTab:Dropdown({
    Title = "ESP Mode",
    Values = {"Username", "Display", "Both"},
    Value = "Display",
    Callback = function(v) ESP_Mode = v RefreshESP() end
})
MainTab:Colorpicker({
    Title = "ESP Text Color",
    Default = Color3.fromRGB(25, 212, 209),
    Callback = function(color)
        ESP_Color = color
        RefreshESP()
    end
})
MainTab:Toggle({ Title = "Infinity Jump", Callback = function(v) _G.InfJump = v end })
MainTab:Toggle({ Title = "No Clip", Callback = function(v) _G.NoClip = v end })
MainTab:Toggle({ Title = "FLY", Callback = function(v) _G.Fly = v end })
MainTab:Dropdown({
    Title = "Mode FLY Speed",
    Values = {"SLOW", "NORMAL", "SPEED"},
    Value = "NORMAL",
    Callback = function(v)
        if v == "SLOW" then FlySpeed = 25
        elseif v == "NORMAL" then FlySpeed = 50
        elseif v == "SPEED" then FlySpeed = 150
        end
    end
})

--==================================================
-- 📸 FREECAM TAB 
--==================================================
local FreecamTab = Window:Tab({ Title = "FREECAM", Icon = "camera" })

local function GetFullPlayerList(filter)
    local t = {}
    for _, v in pairs(Players:GetPlayers()) do
        if not filter or v.DisplayName:lower():find(filter:lower()) or v.Name:lower():find(filter:lower()) then
            table.insert(t, v.DisplayName .. " (@" .. v.Name .. ")")
        end
    end
    return t
end

local function GetPlrFromText(txt)
    if not txt then return nil end
    local name = txt:match("@(%w+)")
    return Players:FindFirstChild(name)
end

FreecamTab:Section({ Title = "Freecam Controller" })
FreecamTab:Toggle({
    Title = "Enable Freecam",
    Callback = function(v) ToggleFreecam(v) end
})
FreecamTab:Slider({
    Title = "Freecam Speed",
    Min = 0.1, Max = 10, Step = 0.1, Default = 1,
    Callback = function(v) FreecamSpeed = v end
})

FreecamTab:Section({ Title = "Solo Freecam" })
local LockDrop
FreecamTab:Input({
    Title = "Search Player",
    Placeholder = "Search DisplayName...",
    Callback = function(t) if LockDrop then LockDrop:Refresh(GetFullPlayerList(t)) end end
})
LockDrop = FreecamTab:Dropdown({
    Title = "Select Target",
    Values = GetFullPlayerList(),
    Callback = function(v) LockFree_Target = GetPlrFromText(v) end
})
FreecamTab:Dropdown({
    Title = "Lock Mode",
    Values = {"OFF", "FOLLOW HEAD", "FOLLOW BODY"},
    Value = "OFF",
    Callback = function(v) LockFree_Mode = v end
})
FreecamTab:Toggle({ Title = "Enable Lock Position", Callback = function(v) LockFree_Enabled = v end })

FreecamTab:Section({ Title = "Duo Center Freecam" })
local CDropA, CDropB
FreecamTab:Input({
    Title = "Search Player A/B",
    Placeholder = "Filter...",
    Callback = function(t) 
        local l = GetFullPlayerList(t)
        if CDropA then CDropA:Refresh(l) end
        if CDropB then CDropB:Refresh(l) end
    end
})
CDropA = FreecamTab:Dropdown({ Title = "Target A", Values = GetFullPlayerList(), Callback = function(v) CenterFree_A = GetPlrFromText(v) end })
CDropB = FreecamTab:Dropdown({ Title = "Target B", Values = GetFullPlayerList(), Callback = function(v) CenterFree_B = GetPlrFromText(v) end })
FreecamTab:Dropdown({
    Title = "Center Mode",
    Values = {"OFF", "FOLLOW HEAD", "FOLLOW BODY"},
    Value = "OFF",
    Callback = function(v) CenterFree_Mode = v end
})
FreecamTab:Toggle({ Title = "Enable Center Freecam", Callback = function(v) CenterFree_Enabled = v end })

FreecamTab:Section({ Title = "Other" })
FreecamTab:Button({
    Title = "Refresh Player List",
    Callback = function()
        local l = GetFullPlayerList()
        if LockDrop then LockDrop:Refresh(l) end
        if CDropA then CDropA:Refresh(l) end
        if CDropB then CDropB:Refresh(l) end
        Notify("Updated", "Player list refreshed!")
    end
})

--==================================================
-- 🎥 RENDERSTEPPED: ALL LOGIC (ASLI + LOCK + CENTER + SMOOTHING)
--==================================================
RunService.RenderStepped:Connect(function(dt)
    if not FreecamEnabled or not FreecamPart then return end

    Camera.FieldOfView = Camera.FieldOfView + (TargetFOV - Camera.FieldOfView) * CinematicSmoothness
    SmoothRotX = SmoothRotX + (RotationX - SmoothRotX) * CinematicSmoothness
    SmoothRotY = SmoothRotY + (RotationY - SmoothRotY) * CinematicSmoothness
    local lookRotation = CFrame.Angles(0, math.rad(SmoothRotY), 0) * CFrame.Angles(math.rad(SmoothRotX), 0, 0)

    if LockFree_Enabled or CenterFree_Enabled then
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            if LockFree_Enabled then LockFree_Offset = math.max(LockFree_Offset - FreecamSpeed, 0)
            else CenterFree_Offset = math.max(CenterFree_Offset - FreecamSpeed, 0) end
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            if LockFree_Enabled then LockFree_Offset = LockFree_Offset + FreecamSpeed
            else CenterFree_Offset = CenterFree_Offset + FreecamSpeed end
        end
    end

    local targetPosition = FreecamPart.Position

    if LockFree_Enabled and LockFree_Target and LockFree_Mode ~= "OFF" then
        local char = LockFree_Target.Character
        if char then
            local p = (LockFree_Mode == "FOLLOW HEAD") and char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
            if p then
                targetPosition = p.Position + (lookRotation.LookVector * -LockFree_Offset)
            end
        end
    elseif CenterFree_Enabled and CenterFree_A and CenterFree_B and CenterFree_Mode ~= "OFF" then
        local cA, cB = CenterFree_A.Character, CenterFree_B.Character
        if cA and cB then
            local pName = (CenterFree_Mode == "FOLLOW HEAD") and "Head" or "HumanoidRootPart"
            local pA, pB = cA:FindFirstChild(pName), cB:FindFirstChild(pName)
            if pA and pB then
                local centerPos = (pA.Position + pB.Position) / 2
                targetPosition = centerPos + (lookRotation.LookVector * -CenterFree_Offset)
            end
        end
    else
        local moveInput = Vector3.new(0,0,0)
        local lookV = Camera.CFrame.LookVector
        local rightV = Camera.CFrame.RightVector
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveInput = moveInput + lookV end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveInput = moveInput - lookV end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveInput = moveInput + rightV end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveInput = moveInput - rightV end
        if UserInputService:IsKeyDown(Enum.KeyCode.E) then moveInput = moveInput + Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.Q) then moveInput = moveInput - Vector3.new(0,1,0) end
        
        targetPosition = FreecamPart.Position + (moveInput * FreecamSpeed)
    end

    FreecamPart.CFrame = FreecamPart.CFrame:Lerp(CFrame.new(targetPosition) * lookRotation, CinematicSmoothness)
    Camera.CFrame = FreecamPart.CFrame
end)

--==================================================
-- 🎭 TROLL TAB
--==================================================
local TrollTab = Window:Tab({ Title = "TROLL", Icon = "zap" })
TrollTab:Section({ Title = "WORK ONLY HANGOUT!" })
local CopyDropdown
TrollTab:Input({
    Title = "Search Player",
    Placeholder = "Type name...",
    Callback = function(t)
        if CopyDropdown then CopyDropdown:Refresh(GetPlayerList(t)) end
    end
})
CopyDropdown = TrollTab:Dropdown({
    Title = "Select Player",
    Values = GetPlayerList(),
    Callback = function(val) SelectedPlayerName = val end
})
TrollTab:Button({
    Title = "Copy Ava",
    Callback = function()
        if not SelectedPlayerName then return end
        local target = Players:FindFirstChild(SelectedPlayerName)
        if target and target.Character then
            local success, desc = pcall(function() return target.Character:FindFirstChildOfClass("Humanoid"):GetAppliedDescription() end)
            if success and desc then ApplyAvatarFromDescription(desc, SelectedPlayerName) end
        end
    end
})
TrollTab:Button({
    Title = "Refresh Player List",
    Callback = function() if CopyDropdown then CopyDropdown:Refresh(GetPlayerList()) end end
})
TrollTab:Section({ Title = "Global Search (API)" })
TrollTab:Input({
    Title = "Search Username",
    Placeholder = "e.g. Roblox",
    Callback = function(t) SearchInputVal = t end
})
TrollTab:Button({
    Title = "Copy By Search USN",
    Callback = function()
        if SearchInputVal == "" then return end
        local uid = tonumber(SearchInputVal)
        if not uid then
            local s, id = pcall(function() return Players:GetUserIdFromNameAsync(SearchInputVal) end)
            if s then uid = id end
        end
        if uid then
            local s, desc = pcall(function() return Players:GetHumanoidDescriptionFromUserId(uid) end)
            if s and desc then ApplyAvatarFromDescription(desc, SearchInputVal) end
        end
    end
})
TrollTab:Button({
    Title = "Reset Avatar",
    Callback = function()
        local s, desc = pcall(function() return Players:GetHumanoidDescriptionFromUserId(LocalPlayer.UserId) end)
        if s and desc then ApplyAvatarFromDescription(desc, "Original") end
    end
})

--==================================================
-- 🕺 EMOTE & ANIMASI TAB (FIXED & OVERRIDE READY)
--==================================================
local EmoteTab = Window:Tab({ Title = "EMOTE", Icon = "smile" })

-- 📌 TAMBAHKAN ID EMOTE KAMU DI SINI
local EmotesData = {
    ["SHAKE THAT THANG"] = "rbxassetid://118364690209655",
    ["Dougie"] = "rbxassetid://93650537970037",
    ["stretch"] = "rbxassetid://119377401608190"
}

-- 📌 TAMBAHKAN ID ANIMASI KAMU DI SINI (Ubah Gaya Berjalan/Lari)
local AnimationsData = {
    ["Ninja Run"] = "rbxassetid://656118852",
    ["Zombie Walk"] = "rbxassetid://616163682",
    ["Mage Idle"] = "rbxassetid://656117400",
    ["Toy Walk"] = "rbxassetid://782843345"
}

local function GetKeys(t)
    local keys = {}
    for k, v in pairs(t) do table.insert(keys, k) end
    return keys
end

--================ LOGIC EMOTE ================--
local SelectedEmoteId = nil
local CurrentEmoteTrack = nil

local function PlayEmote(animId)
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local animator = hum:FindFirstChildOfClass("Animator") or hum

    if CurrentEmoteTrack then CurrentEmoteTrack:Stop() end

    local anim = Instance.new("Animation")
    anim.AnimationId = animId
    CurrentEmoteTrack = animator:LoadAnimation(anim)
    CurrentEmoteTrack.Looped = true
    CurrentEmoteTrack:Play()
end

local function StopEmote()
    if CurrentEmoteTrack then CurrentEmoteTrack:Stop() end
    CurrentEmoteTrack = nil
    Notify("Berhasil", "Emote dihentikan.")
end

--================ LOGIC ANIMASI OVERRIDE ================--
local SelectedAnimName = nil
local SelectedAnimId = nil
local DefaultAnimIds = {}

local function SaveOriginalAnimations()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Animate") then return end
    
    if next(DefaultAnimIds) == nil then
        for _, state in ipairs(char.Animate:GetChildren()) do
            if state:IsA("StringValue") or state:IsA("Folder") then
                DefaultAnimIds[state.Name] = {}
                for _, anim in pairs(state:GetChildren()) do
                    if anim:IsA("Animation") then
                        DefaultAnimIds[state.Name][anim.Name] = anim.AnimationId
                    end
                end
            end
        end
    end
end

local function ApplyMovementAnimation(animName, animId)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Animate") then 
        return Notify("Error", "Script Animate bawaan tidak ditemukan di karaktermu.") 
    end
    
    SaveOriginalAnimations()

    local animate = char.Animate
    local lowerName = string.lower(animName)
    
    -- Auto Deteksi
    local targets = {}
    if string.find(lowerName, "run") then table.insert(targets, "run")
    elseif string.find(lowerName, "walk") then table.insert(targets, "walk")
    elseif string.find(lowerName, "idle") then table.insert(targets, "idle")
    elseif string.find(lowerName, "jump") then table.insert(targets, "jump")
    elseif string.find(lowerName, "fall") then table.insert(targets, "fall")
    else
        table.insert(targets, "walk")
        table.insert(targets, "run")
    end

    for _, stateName in ipairs(targets) do
        local stateObj = animate:FindFirstChild(stateName)
        if stateObj then
            for _, anim in pairs(stateObj:GetChildren()) do
                if anim:IsA("Animation") then
                    anim.AnimationId = animId
                end
            end
        end
    end

    animate.Disabled = true
    task.wait(0.05)
    animate.Disabled = false

    Notify("Sukses", animName .. " berhasil diaplikasikan!")
end

local function ResetMovementDefault()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Animate") then return end
    
    local animate = char.Animate
    for stateName, anims in pairs(DefaultAnimIds) do
        local stateObj = animate:FindFirstChild(stateName)
        if stateObj then
            for animName, origId in pairs(anims) do
                local anim = stateObj:FindFirstChild(animName)
                if anim and anim:IsA("Animation") then
                    anim.AnimationId = origId
                end
            end
        end
    end
    
    animate.Disabled = true
    task.wait(0.05)
    animate.Disabled = false
    Notify("Sukses", "Gerakan dikembalikan ke default.")
end

-- SECTION: Categori Emote
EmoteTab:Section({ Title = "Categori Emote" })

EmoteTab:Dropdown({
    Title = "Pilih Emote",
    Values = GetKeys(EmotesData),
    Callback = function(val)
        SelectedEmoteId = EmotesData[val]
    end
})

EmoteTab:Button({
    Title = "Play Emote",
    Callback = function()
        if SelectedEmoteId then
            PlayEmote(SelectedEmoteId)
        else
            Notify("Error", "Pilih emote dari list terlebih dahulu!")
        end
    end
})

EmoteTab:Button({
    Title = "Stop Emote",
    Callback = function()
        StopEmote()
    end
})

-- SECTION: Categori Animasi
EmoteTab:Section({ Title = "Categori Animasi (Ubah Gaya Jalan)" })

EmoteTab:Dropdown({
    Title = "Pilih Animasi",
    Values = GetKeys(AnimationsData),
    Callback = function(val)
        SelectedAnimName = val
        SelectedAnimId = AnimationsData[val]
    end
})

EmoteTab:Button({
    Title = "Apply Animasi (Override)",
    Callback = function()
        if SelectedAnimId and SelectedAnimName then
            ApplyMovementAnimation(SelectedAnimName, SelectedAnimId)
        else
            Notify("Error", "Pilih animasi dari list terlebih dahulu!")
        end
    end
})

EmoteTab:Button({
    Title = "Reset Default Animasi",
    Callback = function()
        ResetMovementDefault()
    end
})

--==================================================
-- ⚙️ SETTINGS & UI LOGIC
--==================================================
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and (input.KeyCode == Enum.KeyCode.RightControl or input.KeyCode == Enum.KeyCode.Insert) then
        Window:Toggle()
    end
end)
Players.PlayerAdded:Connect(function() task.wait(0.5) RefreshESP() end)
Players.PlayerRemoving:Connect(function() RefreshESP() end)

Notify("SCRIPT BY ART1LE", "INI PRIVATE YA AJG", 5)
