local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

local VxnityUI = {}
VxnityUI.__index = VxnityUI

local function getGuiParent()
    if gethui then return gethui() end
    local ok, cg = pcall(function() return game:GetService("CoreGui") end)
    if ok and cg then return cg end
    return LocalPlayer:WaitForChild("PlayerGui")
end

-- ============================================================
-- SNOWFLAKE THEME COLORS
-- ============================================================
local ACCENT = Color3.fromRGB(100, 180, 255)
local ACCENT2 = Color3.fromRGB(70, 150, 235)
local BG_DARK = Color3.fromRGB(240, 245, 255)
local BG_FRAME = Color3.fromRGB(255, 255, 255)
local BG_ELEM = Color3.fromRGB(245, 248, 255)
local TEXT_WHITE = Color3.fromRGB(20, 40, 80)
local TEXT_GRAY = Color3.fromRGB(80, 100, 140)
local OUTLINE = Color3.fromRGB(180, 210, 240)
local ACCENT_GLOW = Color3.fromRGB(120, 200, 255)

-- ============================================================
-- SNOWFLAKE EFFECT
-- ============================================================
local snowflakes = {}
local snowflakeGui

local function createSnowflakeEffect(parent)
    if snowflakeGui then snowflakeGui:Destroy() end
    snowflakeGui = Instance.new("ScreenGui")
    snowflakeGui.Name = "SnowFlakes"
    snowflakeGui.ResetOnSpawn = false
    snowflakeGui.IgnoreGuiInset = true
    snowflakeGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    snowflakeGui.Parent = parent

    for i = 1, 25 do
        local flake = Instance.new("TextLabel")
        flake.Size = UDim2.fromOffset(10, 10)
        flake.BackgroundTransparency = 1
        flake.Text = "â„"
        flake.TextColor3 = Color3.fromRGB(150, 220, 255)
        flake.TextSize = math.random(8, 16)
        flake.TextTransparency = math.random(10, 40) / 100
        flake.Position = UDim2.new(math.random() * 1, 0, math.random() * 1, 0)
        flake.Rotation = math.random(0, 360)
        flake.Font = Enum.Font.GothamBold
        flake.Parent = snowflakeGui

        table.insert(snowflakes, {
            label = flake,
            speed = 0.1 + math.random() * 0.3,
            drift = (math.random() - 0.5) * 0.1,
            rotationSpeed = (math.random() - 0.5) * 20,
            x = flake.Position.X.Scale,
            y = flake.Position.Y.Scale
        })
    end

    task.spawn(function()
        while snowflakeGui and snowflakeGui.Parent do
            for _, flake in ipairs(snowflakes) do
                if flake.label and flake.label.Parent then
                    flake.y = flake.y + flake.speed * 0.002
                    flake.x = flake.x + flake.drift * 0.002
                    flake.label.Rotation = flake.label.Rotation + flake.rotationSpeed * 0.004

                    if flake.y > 1.1 then
                        flake.y = -0.1
                        flake.x = math.random() * 1
                        flake.label.TextSize = math.random(8, 16)
                    end
                    if flake.x > 1.05 then flake.x = -0.05 end
                    if flake.x < -0.05 then flake.x = 1.05 end

                    flake.label.Position = UDim2.new(flake.x, 0, flake.y, 0)
                    local glow = math.sin(tick() * 1.5 + flake.label.TextSize) * 0.3 + 0.7
                    flake.label.TextColor3 = Color3.fromRGB(150 + glow * 80, 200 + glow * 55, 255)
                end
            end
            task.wait(0.016)
        end
    end)
end

-- ============================================================
-- BALL CANCOLLIDE GUARDIAN
-- ============================================================
local _ballGuardConn
local function startBallGuard()
    if _ballGuardConn then return end
    local function guardBall(ball)
        if not ball or not ball:IsA("BasePart") then return end
        pcall(function() ball.CanCollide = false end)
        ball:GetPropertyChangedSignal("CanCollide"):Connect(function()
            if ball.CanCollide then
                pcall(function() ball.CanCollide = false end)
            end
        end)
    end

    local function findAndGuard()
        local tps = Workspace:FindFirstChild("TPSSystem")
        if tps then
            local ball = tps:FindFirstChild("TPS")
            if ball then guardBall(ball) end
            tps.ChildAdded:Connect(function(child)
                if child.Name == "TPS" then guardBall(child) end
            end)
        end
    end

    findAndGuard()
    Workspace.ChildAdded:Connect(function(child)
        if child.Name == "TPSSystem" then
            task.wait(0.05)
            findAndGuard()
        end
    end)
end
pcall(startBallGuard)

-- ============================================================
-- TWEEN HELPERS
-- ============================================================
local function tweenQuint(obj, t, p) return TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), p) end
local function tweenBack(obj, t, p) return TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Back, Enum.EasingDirection.Out), p) end
local function tweenExpo(obj, t, p) return TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Exponential, Enum.EasingDirection.Out), p) end
local function tweenSine(obj, t, p) return TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), p) end
local function tweenCirc(obj, t, p) return TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Circular, Enum.EasingDirection.Out), p) end

-- ============================================================
-- NOTIFICATIONS
-- ============================================================
function VxnityUI:Notify(opts)
    local title = opts.Title or ""
    local desc = opts.Desc or ""
    local duration = opts.Duration or 3

    local parent = getGuiParent()
    local existing = parent:FindFirstChild("VxnityNotifGui")
    if existing then existing:Destroy() end

    local NotifGui = Instance.new("ScreenGui")
    NotifGui.Name = "VxnityNotifGui"
    NotifGui.ResetOnSpawn = false
    NotifGui.IgnoreGuiInset = true
    NotifGui.Parent = parent

    local frame = Instance.new("Frame")
    frame.Size = UDim2.fromOffset(290, 64)
    frame.Position = UDim2.new(1, 20, 1, -80)
    frame.BackgroundColor3 = BG_FRAME
    frame.BorderSizePixel = 0
    frame.BackgroundTransparency = 1
    frame.Parent = NotifGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame

    local snowIcon = Instance.new("TextLabel")
    snowIcon.Size = UDim2.fromOffset(20, 20)
    snowIcon.Position = UDim2.new(0, 8, 0.5, -10)
    snowIcon.BackgroundTransparency = 1
    snowIcon.Text = "â„"
    snowIcon.TextColor3 = ACCENT_GLOW
    snowIcon.Font = Enum.Font.GothamBold
    snowIcon.TextSize = 14
    snowIcon.TextTransparency = 1
    snowIcon.Parent = frame

    local accentBar = Instance.new("Frame")
    accentBar.Size = UDim2.fromOffset(3, 44)
    accentBar.Position = UDim2.new(0, 0, 0.5, -22)
    accentBar.BackgroundColor3 = ACCENT
    accentBar.BorderSizePixel = 0
    accentBar.Parent = frame
    local abC = Instance.new("UICorner")
    abC.CornerRadius = UDim.new(1,0)
    abC.Parent = accentBar

    local stroke = Instance.new("UIStroke")
    stroke.Color = OUTLINE
    stroke.Thickness = 1.5
    stroke.Parent = frame

    local titleLbl = Instance.new("TextLabel")
    titleLbl.Size = UDim2.new(1, -30, 0.5, 0)
    titleLbl.Position = UDim2.fromOffset(32, 4)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Text = title
    titleLbl.TextColor3 = TEXT_WHITE
    titleLbl.Font = Enum.Font.GothamBold
    titleLbl.TextSize = 13
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.TextTransparency = 1
    titleLbl.Parent = frame

    local descLbl = Instance.new("TextLabel")
    descLbl.Size = UDim2.new(1, -30, 0.5, 0)
    descLbl.Position = UDim2.new(0, 32, 0.5, 0)
    descLbl.BackgroundTransparency = 1
    descLbl.Text = desc
    descLbl.TextColor3 = TEXT_GRAY
    descLbl.Font = Enum.Font.Gotham
    descLbl.TextSize = 11
    descLbl.TextXAlignment = Enum.TextXAlignment.Left
    descLbl.TextTransparency = 1
    descLbl.Parent = frame

    tweenExpo(frame, 0.45, { Position = UDim2.new(1, -305, 1, -80), BackgroundTransparency = 0 }):Play()
    task.delay(0.1, function()
        tweenSine(titleLbl, 0.3, { TextTransparency = 0 }):Play()
        tweenSine(snowIcon, 0.3, { TextTransparency = 0 }):Play()
        task.delay(0.08, function() tweenSine(descLbl, 0.3, { TextTransparency = 0 }):Play() end)
    end)

    task.delay(duration, function()
        if NotifGui and NotifGui.Parent then
            tweenQuint(frame, 0.35, { Position = UDim2.new(1, 20, 1, -80), BackgroundTransparency = 1 }):Play()
            task.wait(0.4)
            NotifGui:Destroy()
        end
    end)
end

-- ============================================================
-- PERSISTENCE SYSTEM
-- ============================================================
if not _G._SnowPersist then
    _G._SnowPersist = {
        reachEnabled = false,
        reachDistance = 1,
        reactPower = 0,
        ballSpeedMult = 1.0,
        reactHookOn = false,
        helperActive = false,
        helperEnabled = false,
        magnetMode = true,
        predictMode = true,
        spaceLock = false,
        pullStrength = 1.0,
    }
end
local P = _G._SnowPersist

if not _G._VxRBall then _G._VxRBall = nil end
if not _G._VxRHRP then _G._VxRHRP = nil end

if not _G._VxCacheWorker then
    _G._VxCacheWorker = RunService.RenderStepped:Connect(function()
        local sys = Workspace:FindFirstChild("TPSSystem")
        _G._VxRBall = sys and sys:FindFirstChild("TPS")
        local ch = LocalPlayer.Character
        _G._VxRHRP = ch and ch:FindFirstChild("HumanoidRootPart")
    end)
end

-- ============================================================
-- FIXED REACH SYSTEM - WORKS WITH FIRE_TOUCH_INTEREST
-- ============================================================
local reachEnabled = P.reachEnabled
local reachDistance = P.reachDistance
local reachConnection

local function startReach()
    if reachConnection then reachConnection:Disconnect() end
    
    local char, root, hum, tps, limb = nil, nil, nil, nil, nil
    local frameSkip = 0
    local lastLimb = nil

    reachConnection = RunService.RenderStepped:Connect(function()
        local character = LocalPlayer.Character
        if not character then return end
        
        -- Update character references
        if character ~= char then
            char = character
            root = character:FindFirstChild("HumanoidRootPart")
            hum = character:FindFirstChild("Humanoid")
            limb = nil
            lastLimb = nil
        end
        
        if not (root and hum) then return end
        
        -- Get TPS ball
        frameSkip = frameSkip + 1
        if frameSkip >= 2 then
            frameSkip = 0
            local sys = Workspace:FindFirstChild("TPSSystem")
            tps = sys and sys:FindFirstChild("TPS")
        end
        
        if not tps or not tps.Parent then return end
        
        -- Check distance
        local dist = (root.Position - tps.Position).Magnitude
        if dist > reachDistance then return end
        
        -- Make sure ball can be touched
        if tps.CanCollide then
            pcall(function() tps.CanCollide = false end)
        end
        
        -- Get the right limb based on rig type
        local rig = hum.RigType
        if rig ~= lastLimb or not limb or not limb.Parent then
            lastLimb = rig
            
            -- Find preferred foot
            local pf = Lighting:FindFirstChild(LocalPlayer.Name)
            local foot = pf and pf:FindFirstChild("PreferredFoot")
            
            if rig == Enum.HumanoidRigType.R6 then
                -- R6: Use legs
                local legName = "Right Leg"
                if foot and foot.Value == 0 then
                    legName = "Left Leg"
                end
                limb = char:FindFirstChild(legName)
                
                -- Also try both legs
                local rightLeg = char:FindFirstChild("Right Leg")
                local leftLeg = char:FindFirstChild("Left Leg")
                
                -- Fire touch on both legs for better detection
                if rightLeg then
                    pcall(function()
                        firetouchinterest(rightLeg, tps, 0)
                        firetouchinterest(rightLeg, tps, 1)
                    end)
                end
                if leftLeg then
                    pcall(function()
                        firetouchinterest(leftLeg, tps, 0)
                        firetouchinterest(leftLeg, tps, 1)
                    end)
                end
                
            elseif rig == Enum.HumanoidRigType.R15 then
                -- R15: Use lower legs
                local legName = "RightLowerLeg"
                if foot and foot.Value == 0 then
                    legName = "LeftLowerLeg"
                end
                limb = char:FindFirstChild(legName)
                
                -- Fire touch on both lower legs
                local rightLeg = char:FindFirstChild("RightLowerLeg")
                local leftLeg = char:FindFirstChild("LeftLowerLeg")
                
                if rightLeg then
                    pcall(function()
                        firetouchinterest(rightLeg, tps, 0)
                        firetouchinterest(rightLeg, tps, 1)
                    end)
                end
                if leftLeg then
                    pcall(function()
                        firetouchinterest(leftLeg, tps, 0)
                        firetouchinterest(leftLeg, tps, 1)
                    end)
                end
            end
        end
        
        -- If we have a specific limb, fire touch on it too
        if limb then
            pcall(function()
                firetouchinterest(limb, tps, 0)
                firetouchinterest(limb, tps, 1)
            end)
        end
        
        -- Additional: Fire touch on torso/root for better contact
        if root then
            pcall(function()
                firetouchinterest(root, tps, 0)
                firetouchinterest(root, tps, 1)
            end)
        end
        
        local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
        if torso then
            pcall(function()
                firetouchinterest(torso, tps, 0)
                firetouchinterest(torso, tps, 1)
            end)
        end
    end)
end

_G._VxReachRestart = function()
    if P.reachEnabled then startReach() end
end

-- ============================================================
-- STRONG BALL MAGNET - GLUED TO CHARACTER
-- ============================================================
local ballMagnetEnabled = false
local magnetStrength = 1.0
local magnetConnection
local gluedMode = false
local gluedPosition = nil
local gluedCFrame = nil

local function startBallMagnet()
    if magnetConnection then magnetConnection:Disconnect() end
    
    magnetConnection = RunService.RenderStepped:Connect(function()
        if not ballMagnetEnabled then return end
        
        local ball = _G._VxRBall
        local hrp = _G._VxRHRP
        if not (ball and ball.Parent and hrp) then return end
        
        if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
        pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
        
        if gluedMode then
            -- GLUED MODE - Ball sticks to character like it's glued
            local char = LocalPlayer.Character
            if not char then return end
            
            local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
            if not torso then return end
            
            -- Calculate position in front of character
            local targetPos = torso.Position + torso.CFrame.LookVector * 0.3 + Vector3.new(0, 0.05, 0)
            local diff = targetPos - ball.Position
            local dist = diff.Magnitude
            
            -- Strong glue effect
            if dist > 0.02 then
                local force = math.clamp(dist * 800 * magnetStrength, 50, 10000)
                ball.AssemblyLinearVelocity = diff.Unit * force
                ball.AssemblyAngularVelocity = Vector3.zero
                
                -- If close enough, snap perfectly
                if dist < 0.2 then
                    ball.CFrame = CFrame.new(targetPos)
                    ball.AssemblyLinearVelocity = torso.CFrame.LookVector * 30
                end
            else
                ball.CFrame = CFrame.new(targetPos)
                ball.AssemblyLinearVelocity = torso.CFrame.LookVector * 30
            end
            
            -- Keep ball in sync
            ball.AssemblyAngularVelocity = Vector3.zero
            
        else
            -- MAGNET MODE - Strong pull towards player
            local direction = (hrp.Position - ball.Position)
            local distance = direction.Magnitude
            
            if distance > 0.5 then
                local force = math.clamp((distance ^ 1.5) * 30 * magnetStrength, 10, 8000)
                local velocity = direction.Unit * force
                ball.AssemblyLinearVelocity = velocity
                ball.AssemblyAngularVelocity = Vector3.zero
                
                -- Extra snap when close
                if distance < 3 then
                    local targetPos = hrp.Position + hrp.CFrame.LookVector * 0.5 + Vector3.new(0, 0.05, 0)
                    if distance < 1.5 then
                        ball.CFrame = CFrame.new(targetPos)
                        ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * 50
                    else
                        local pull = (targetPos - ball.Position).Unit * force * 1.5
                        ball.AssemblyLinearVelocity = pull
                    end
                end
            end
        end
    end)
end

-- ============================================================
-- VISUAL AVATAR COPIER - Type any username
-- ============================================================
local avatarCopierEnabled = false
local avatarCopyConnection = nil
local targetUsername = ""
local avatarClone = nil

local function copyPlayerAvatar(username)
    if avatarClone then 
        avatarClone:Destroy() 
        avatarClone = nil
    end
    
    -- Find the player
    local targetPlayer = nil
    for _, player in ipairs(Players:GetPlayers()) do
        if string.lower(player.Name) == string.lower(username) or string.lower(player.DisplayName) == string.lower(username) then
            targetPlayer = player
            break
        end
    end
    
    if not targetPlayer then
        VxnityUI:Notify({ Title = "â„ Avatar Copy", Desc = "Player not found: " .. username, Duration = 3 })
        return
    end
    
    if not targetPlayer.Character then
        VxnityUI:Notify({ Title = "â„ Avatar Copy", Desc = "Player has no character", Duration = 3 })
        return
    end
    
    -- Copy the avatar visually
    local sourceChar = targetPlayer.Character
    local myChar = LocalPlayer.Character
    
    if not myChar then
        VxnityUI:Notify({ Title = "â„ Avatar Copy", Desc = "You don't have a character", Duration = 3 })
        return
    end
    
    -- Create a clone of the target's avatar parts
    local cloneGroup = Instance.new("Model")
    cloneGroup.Name = "AvatarClone_" .. targetPlayer.Name
    cloneGroup.Parent = Workspace
    
    for _, part in ipairs(sourceChar:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            local clone = part:Clone()
            clone.CanCollide = false
            clone.Transparency = 0.2
            clone.Material = Enum.Material.Neon
            clone.Anchored = false
            clone.Parent = cloneGroup
            
            -- Add glow
            local glow = Instance.new("PointLight")
            glow.Name = "AvatarGlow"
            glow.Color = ACCENT
            glow.Brightness = 3
            glow.Range = 10
            glow.Parent = clone
        end
    end
    
    -- Copy accessories if possible
    for _, acc in ipairs(sourceChar:GetChildren()) do
        if acc:IsA("Accessory") or acc:IsA("Hat") or acc:IsA("Tool") then
            local cloneAcc = acc:Clone()
            cloneAcc.CanCollide = false
            cloneAcc.Parent = cloneGroup
        end
    end
    
    -- Add a glowing aura
    local aura = Instance.new("Part")
    aura.Name = "CopyAura"
    aura.Size = Vector3.new(6, 6, 6)
    aura.Shape = Enum.PartType.Ball
    aura.CanCollide = false
    aura.Anchored = true
    aura.Transparency = 0.7
    aura.Material = Enum.Material.Neon
    aura.Color = ACCENT
    aura.Parent = cloneGroup
    
    local auraGlow = Instance.new("PointLight")
    auraGlow.Color = ACCENT
    auraGlow.Brightness = 2
    auraGlow.Range = 15
    auraGlow.Parent = aura
    
    avatarClone = cloneGroup
    
    -- Follow the player
    if avatarCopyConnection then avatarCopyConnection:Disconnect() end
    
    local hrp = myChar:FindFirstChild("HumanoidRootPart")
    
    avatarCopyConnection = RunService.RenderStepped:Connect(function()
        local myCharNow = LocalPlayer.Character
        if not myCharNow then 
            if avatarClone then avatarClone:Destroy() end
            return 
        end
        
        local root = myCharNow:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        if avatarClone and avatarClone.Parent then
            avatarClone:PivotTo(root.CFrame * CFrame.new(0, 0, -2))
            
            -- Animate aura
            local auraPart = avatarClone:FindFirstChild("CopyAura")
            if auraPart then
                auraPart.Size = Vector3.new(4 + math.sin(tick() * 2) * 0.5, 4 + math.sin(tick() * 2) * 0.5, 4 + math.sin(tick() * 2) * 0.5)
            end
        end
    end)
    
    VxnityUI:Notify({ Title = "â„ Avatar Copy", Desc = "Copied " .. targetPlayer.Name .. "'s avatar!", Duration = 3 })
end

-- ============================================================
-- REACTS SYSTEM
-- ============================================================
local REACT_ACTIONS = {
    Kick=true, KickC1=true, Tackle=true, Header=true,
    SaveRA=true, SaveLA=true, SaveRL=true, SaveLL=true, SaveT=true
}

local currentReactPower = P.reactPower
local ballSpeedMult = P.ballSpeedMult

local _reactBallCache = nil
local _reactHRPCache = nil
local _reactCacheTick = 0

local function getReactTargets()
    _reactCacheTick = _reactCacheTick + 1
    if _reactCacheTick >= 2 then
        _reactCacheTick = 0
        _reactBallCache = _G._VxRBall
        _reactHRPCache = _G._VxRHRP
    end
    return _reactBallCache, _reactHRPCache
end

local function applyReactInstant(power)
    local ball, hrp = getReactTargets()
    if not (ball and ball.Parent and hrp) then return end
    if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
    pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
    ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * (power * ballSpeedMult)
end

local function enableReactHook()
    if _G._VxReactHookInstalled then return end
    _G._VxReactHookInstalled = true
    P.reactHookOn = true
    local meta = getrawmetatable(game)
    local oldNamecall = meta.namecall
    setreadonly(meta, false)
    meta.namecall = newcclosure(function(self, ...)
        if getnamecallmethod() == "FireServer"
            and currentReactPower > 0
            and REACT_ACTIONS[tostring(self)] then
            local ball, hrp = getReactTargets()
            if ball and ball.Parent and hrp then
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * (currentReactPower * ballSpeedMult)
            end
        end
        return oldNamecall(self, ...)
    end)
    setreadonly(meta, true)
end

-- ============================================================
-- INF HELPER SYSTEM
-- ============================================================
local CONFIG = {
    FOLLOW_DISTANCE = 0.08,
    FOLLOW_SPEED = 30000,
    DEAD_ZONE = 0.03,
    MAX_DISTANCE = 0.2,
    STRONG_PULL = 50000,
    SOFT_PULL = 35000,
    MAGNET_PULL = 80000,
    PREDICT_OFFSET = 0.04,
    VERTICAL_OFFSET = -0.05,
    LOCK_RADIUS = 0.01,
    ANGULAR_KILL = true,
}

local function getOrCreateAtt(ball)
    local att = ball:FindFirstChild("_infAtt")
    if not att then att = Instance.new("Attachment"); att.Name = "_infAtt"; att.Parent = ball end
    return att
end

local function getOrCreateLV(ball, att)
    local lv = ball:FindFirstChild("_infLV")
    if not lv then
        lv = Instance.new("LinearVelocity"); lv.Name = "_infLV"; lv.Attachment0 = att
        lv.MaxForce = math.huge; lv.RelativeTo = Enum.ActuatorRelativeTo.World
        lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
        lv.VectorVelocity = Vector3.zero; lv.Parent = ball
    end
    return lv
end

local function getOrCreateAV(ball, att)
    local av = ball:FindFirstChild("_infAV")
    if not av then
        av = Instance.new("AngularVelocity"); av.Name = "_infAV"; av.Attachment0 = att
        av.MaxTorque = math.huge; av.RelativeTo = Enum.ActuatorRelativeTo.World
        av.AngularVelocity = Vector3.zero; av.Parent = ball
    end
    return av
end

local function cleanupBall(ball)
    if not ball then return end
    pcall(function()
        local lv = ball:FindFirstChild("_infLV"); if lv then lv.VectorVelocity = Vector3.zero end
        local av = ball:FindFirstChild("_infAV"); if av then av.AngularVelocity = Vector3.zero end
    end)
end

local lockedPos = nil
local lastHRPPos = nil
local lastTick = tick()

local function getPredictedTarget(hrp)
    local now = tick(); local dt = now - lastTick; lastTick = now
    local currentPos = hrp.Position
    if lastHRPPos and dt > 0 and dt < 0.1 then
        local velocity = (currentPos - lastHRPPos) / dt
        lastHRPPos = currentPos
        return currentPos + velocity * CONFIG.PREDICT_OFFSET + Vector3.new(0, CONFIG.VERTICAL_OFFSET, 0)
    end
    lastHRPPos = currentPos
    return currentPos + hrp.CFrame.LookVector * CONFIG.FOLLOW_DISTANCE + Vector3.new(0, CONFIG.VERTICAL_OFFSET, 0)
end

-- ============================================================
-- CREATE WINDOW
-- ============================================================
function VxnityUI:CreateWindow(opts)
    local isMobile = UserInputService.TouchEnabled
    local winW = isMobile and 480 or 600
    local winH = isMobile and 380 or 520
    local guiParent = getGuiParent()

    createSnowflakeEffect(guiParent)

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SnowHubGui"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.DisplayOrder = 999
    ScreenGui.Parent = guiParent

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    local newSize = 350
    MainFrame.Size = UDim2.fromOffset(newSize * 0.6, newSize * 0.6)
    MainFrame.Position = UDim2.new(0.5, -newSize/2, 0.5, -newSize/2)
    MainFrame.BackgroundColor3 = BG_DARK
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.BackgroundTransparency = 1
    MainFrame.Parent = ScreenGui

    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 10)
    mainCorner.Parent = MainFrame

    local mainStroke = Instance.new("UIStroke")
    mainStroke.Color = OUTLINE
    mainStroke.Thickness = 1.5
    mainStroke.Parent = MainFrame

    -- Topbar
    local Topbar = Instance.new("Frame")
    Topbar.Name = "Topbar"
    Topbar.Size = UDim2.new(1, 0, 0, isMobile and 40 or 48)
    Topbar.BackgroundColor3 = Color3.fromRGB(245, 248, 255)
    Topbar.BorderSizePixel = 0
    Topbar.BackgroundTransparency = 1
    Topbar.Parent = MainFrame
    Topbar.ZIndex = 2

    local topCorner = Instance.new("UICorner")
    topCorner.CornerRadius = UDim.new(0, 10)
    topCorner.Parent = Topbar

    local topSnow = Instance.new("TextLabel")
    topSnow.Size = UDim2.fromOffset(20, 20)
    topSnow.Position = UDim2.new(0, 6, 0.5, -10)
    topSnow.BackgroundTransparency = 1
    topSnow.Text = "â„"
    topSnow.TextColor3 = ACCENT_GLOW
    topSnow.Font = Enum.Font.GothamBold
    topSnow.TextSize = 14
    topSnow.TextTransparency = 1
    topSnow.Parent = Topbar
    topSnow.ZIndex = 2

    local topAccentLine = Instance.new("Frame")
    topAccentLine.Size = UDim2.new(0, 0, 0, 2)
    topAccentLine.Position = UDim2.new(0, 0, 1, -2)
    topAccentLine.BackgroundColor3 = ACCENT
    topAccentLine.BorderSizePixel = 0
    topAccentLine.Parent = Topbar
    topAccentLine.ZIndex = 2

    local titleLbl = Instance.new("TextLabel")
    titleLbl.Size = UDim2.new(0, 200, 0.5, 0)
    titleLbl.Position = UDim2.fromOffset(30, 0)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Text = "â„ Snow Hub"
    titleLbl.TextColor3 = TEXT_WHITE
    titleLbl.Font = Enum.Font.GothamBold
    titleLbl.TextSize = isMobile and 14 or 16
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.TextTransparency = 1
    titleLbl.Parent = Topbar
    titleLbl.ZIndex = 2

    local authorLbl = Instance.new("TextLabel")
    authorLbl.Size = UDim2.new(0, 200, 0.5, 0)
    authorLbl.Position = UDim2.new(0, 30, 0.5, 0)
    authorLbl.BackgroundTransparency = 1
    authorLbl.Text = opts.Author or "snow edition"
    authorLbl.TextColor3 = ACCENT
    authorLbl.Font = Enum.Font.Gotham
    authorLbl.TextSize = isMobile and 11 or 12
    authorLbl.TextXAlignment = Enum.TextXAlignment.Left
    authorLbl.TextTransparency = 1
    authorLbl.Parent = Topbar
    authorLbl.ZIndex = 2

    -- Minimize Button
    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.fromOffset(22, 22)
    MinBtn.Position = UDim2.new(1, -70, 0.5, -11)
    MinBtn.BackgroundColor3 = Color3.fromRGB(200, 220, 255)
    MinBtn.Text = "âˆ’"
    MinBtn.TextColor3 = Color3.fromRGB(20, 40, 80)
    MinBtn.Font = Enum.Font.GothamBold
    MinBtn.TextSize = 16
    MinBtn.BorderSizePixel = 0
    MinBtn.BackgroundTransparency = 0
    MinBtn.Parent = Topbar
    MinBtn.ZIndex = 3
    local minC = Instance.new("UICorner")
    minC.CornerRadius = UDim.new(1,0)
    minC.Parent = MinBtn

    -- Close Button
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.fromOffset(22, 22)
    CloseBtn.Position = UDim2.new(1, -40, 0.5, -11)
    CloseBtn.BackgroundColor3 = ACCENT
    CloseBtn.Text = "âœ•"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 12
    CloseBtn.BorderSizePixel = 0
    CloseBtn.BackgroundTransparency = 0
    CloseBtn.Parent = Topbar
    CloseBtn.ZIndex = 3
    local clsC = Instance.new("UICorner")
    clsC.CornerRadius = UDim.new(1,0)
    clsC.Parent = CloseBtn

    local minimized = false

    local ContentFrame = Instance.new("Frame")
    ContentFrame.Name = "ContentFrame"
    ContentFrame.Size = UDim2.new(1, 0, 1, -(isMobile and 40 or 48))
    ContentFrame.Position = UDim2.new(0, 0, 0, isMobile and 40 or 48)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Parent = MainFrame
    ContentFrame.ZIndex = 1

    local TabPanel = Instance.new("ScrollingFrame")
    TabPanel.Name = "TabPanel"
    TabPanel.Size = UDim2.new(0, 160, 1, 0)
    TabPanel.BackgroundColor3 = Color3.fromRGB(235, 240, 255)
    TabPanel.BorderSizePixel = 0
    TabPanel.ScrollBarThickness = 0
    TabPanel.CanvasSize = UDim2.new(0, 0, 0, 0)
    TabPanel.AutomaticCanvasSize = Enum.AutomaticSize.Y
    TabPanel.BackgroundTransparency = 1
    TabPanel.Parent = ContentFrame
    TabPanel.ZIndex = 1

    local tabListLayout = Instance.new("UIListLayout")
    tabListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabListLayout.Padding = UDim.new(0, 2)
    tabListLayout.Parent = TabPanel

    local Separator = Instance.new("Frame")
    Separator.Size = UDim2.new(0, 1, 1, 0)
    Separator.Position = UDim2.fromOffset(160, 0)
    Separator.BackgroundColor3 = OUTLINE
    Separator.BorderSizePixel = 0
    Separator.BackgroundTransparency = 1
    Separator.Parent = ContentFrame
    Separator.ZIndex = 1

    local PageHolder = Instance.new("Frame")
    PageHolder.Name = "PageHolder"
    PageHolder.Size = UDim2.new(1, -161, 1, 0)
    PageHolder.Position = UDim2.fromOffset(161, 0)
    PageHolder.BackgroundTransparency = 1
    PageHolder.ClipsDescendants = true
    PageHolder.Parent = ContentFrame
    PageHolder.ZIndex = 1

    -- OPEN ANIMATION
    task.spawn(function()
        tweenBack(MainFrame, 0.35, { Size = UDim2.fromOffset(newSize, newSize), BackgroundTransparency = 0 }):Play()
        task.wait(0.08)
        tweenExpo(Topbar, 0.2, { BackgroundTransparency = 0 }):Play()
        tweenCirc(topAccentLine, 0.35, { Size = UDim2.new(1, 0, 0, 2) }):Play()
        task.wait(0.05)
        tweenSine(titleLbl, 0.18, { TextTransparency = 0 }):Play()
        tweenSine(topSnow, 0.18, { TextTransparency = 0 }):Play()
        task.delay(0.03, function() tweenSine(authorLbl, 0.18, { TextTransparency = 0 }):Play() end)
        task.delay(0.04, function() tweenBack(MinBtn, 0.15, { BackgroundTransparency = 0 }):Play() end)
        task.delay(0.06, function() tweenBack(CloseBtn, 0.15, { BackgroundTransparency = 0 }):Play() end)
        task.delay(0.05, function() tweenExpo(TabPanel, 0.2, { BackgroundTransparency = 0 }):Play() end)
    end)

    -- DRAG
    local dragging, dragStart, startPos = false, nil, nil
    Topbar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    MinBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        tweenQuint(MainFrame, 0.3, {
            Size = minimized and UDim2.fromOffset(winW, isMobile and 40 or 48) or UDim2.fromOffset(winW, winH)
        }):Play()
    end)

    CloseBtn.MouseButton1Click:Connect(function()
        tweenQuint(MainFrame, 0.25, { Size = UDim2.fromOffset(winW, 0), BackgroundTransparency = 1 }):Play()
        task.wait(0.3)
        ScreenGui:Destroy()
    end)

    -- FLOATING OPEN BUTTON
    local OpenBtn = Instance.new("TextButton")
    OpenBtn.Name = "SnowOpenBtn"
    OpenBtn.Size = UDim2.fromOffset(80, 30)
    OpenBtn.Position = UDim2.new(0, 10, 0.5, -15)
    OpenBtn.BackgroundColor3 = ACCENT2
    OpenBtn.Text = "â„ Snow"
    OpenBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    OpenBtn.Font = Enum.Font.GothamBold
    OpenBtn.TextSize = 13
    OpenBtn.BorderSizePixel = 0
    OpenBtn.Visible = false
    OpenBtn.Parent = ScreenGui
    OpenBtn.ZIndex = 999
    local obC = Instance.new("UICorner")
    obC.CornerRadius = UDim.new(0,8)
    obC.Parent = OpenBtn

    CloseBtn.MouseButton1Click:Connect(function() OpenBtn.Visible = true end)
    OpenBtn.MouseButton1Click:Connect(function()
        OpenBtn.Visible = false
        MainFrame.Size = UDim2.fromOffset(winW, winH)
        MainFrame.Parent = ScreenGui
    end)

    local windowObj = {}
    local currentTab = nil
    local sectionOrder = 0

    local function setActiveTab(tabPage, tabBtn)
        if currentTab then
            currentTab.Page.Visible = false
            TweenService:Create(currentTab.Btn, TweenInfo.new(0.18, Enum.EasingStyle.Sine), {
                BackgroundColor3 = Color3.fromRGB(230, 235, 250),
                BackgroundTransparency = 0
            }):Play()
        end
        tabPage.Visible = true
        TweenService:Create(tabBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quint), {
            BackgroundColor3 = Color3.fromRGB(215, 225, 245),
            BackgroundTransparency = 0
        }):Play()
        currentTab = {Page = tabPage, Btn = tabBtn}
    end

    local function makeElementContainer(parent)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, -8, 0, 52)
        frame.BackgroundColor3 = BG_ELEM
        frame.BorderSizePixel = 0
        frame.Parent = parent
        local fCorner = Instance.new("UICorner")
        fCorner.CornerRadius = UDim.new(0, 7)
        fCorner.Parent = frame
        return frame
    end

    local function buildTabAPI(page)
        local tabAPI = {}

        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.BorderSizePixel = 0
        scroll.ScrollBarThickness = 3
        scroll.ScrollBarImageColor3 = ACCENT
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
        scroll.Parent = page

        local listLayout = Instance.new("UIListLayout")
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Padding = UDim.new(0, 5)
        listLayout.Parent = scroll

        local elemOrder = 0
        local function nextOrder() elemOrder = elemOrder + 1 return elemOrder end

        function tabAPI:Section(opts2)
            local sFrame = Instance.new("Frame")
            sFrame.Size = UDim2.new(1, -8, 0, 22)
            sFrame.BackgroundTransparency = 1
            sFrame.LayoutOrder = nextOrder()
            sFrame.Parent = scroll

            local sLine = Instance.new("Frame")
            sLine.Size = UDim2.new(1, 0, 0, 1)
            sLine.Position = UDim2.new(0, 0, 0.5, 0)
            sLine.BackgroundColor3 = ACCENT
            sLine.BackgroundTransparency = 0.3
            sLine.BorderSizePixel = 0
            sLine.Parent = sFrame

            local sTitle = Instance.new("TextLabel")
            sTitle.Size = UDim2.new(0, 0, 1, 0)
            sTitle.AutomaticSize = Enum.AutomaticSize.X
            sTitle.BackgroundColor3 = BG_DARK
            sTitle.BorderSizePixel = 0
            sTitle.Position = UDim2.new(0, 8, 0, 0)
            sTitle.Text = " " .. (opts2.Title or "") .. " "
            sTitle.TextColor3 = ACCENT
            sTitle.Font = Enum.Font.GothamBold
            sTitle.TextSize = 11
            sTitle.Parent = sFrame
        end

        function tabAPI:Paragraph(opts2)
            local f = makeElementContainer(scroll)
            f.Size = UDim2.new(1, -8, 0, 48)
            f.LayoutOrder = nextOrder()

            local t = Instance.new("TextLabel")
            t.Size = UDim2.new(1, -12, 0.5, 0)
            t.Position = UDim2.fromOffset(10, 4)
            t.BackgroundTransparency = 1
            t.Text = opts2.Title or ""
            t.TextColor3 = TEXT_WHITE
            t.Font = Enum.Font.GothamBold
            t.TextSize = 13
            t.TextXAlignment = Enum.TextXAlignment.Left
            t.Parent = f

            local d = Instance.new("TextLabel")
            d.Size = UDim2.new(1, -12, 0.5, 0)
            d.Position = UDim2.new(0, 10, 0.5, 0)
            d.BackgroundTransparency = 1
            d.Text = opts2.Desc or ""
            d.TextColor3 = TEXT_GRAY
            d.Font = Enum.Font.Gotham
            d.TextSize = 11
            d.TextXAlignment = Enum.TextXAlignment.Left
            d.Parent = f
        end

        function tabAPI:Toggle(opts2)
            local f = makeElementContainer(scroll)
            f.LayoutOrder = nextOrder()

            local titleLb = Instance.new("TextLabel")
            titleLb.Size = UDim2.new(1, -58, 0.5, 0)
            titleLb.Position = UDim2.fromOffset(10, 5)
            titleLb.BackgroundTransparency = 1
            titleLb.Text = opts2.Title or ""
            titleLb.TextColor3 = TEXT_WHITE
            titleLb.Font = Enum.Font.GothamBold
            titleLb.TextSize = 13
            titleLb.TextXAlignment = Enum.TextXAlignment.Left
            titleLb.Parent = f

            if opts2.Desc and opts2.Desc ~= "" then
                local descLb = Instance.new("TextLabel")
                descLb.Size = UDim2.new(1, -58, 0.5, 0)
                descLb.Position = UDim2.new(0, 10, 0.5, 0)
                descLb.BackgroundTransparency = 1
                descLb.Text = opts2.Desc
                descLb.TextColor3 = TEXT_GRAY
                descLb.Font = Enum.Font.Gotham
                descLb.TextSize = 11
                descLb.TextXAlignment = Enum.TextXAlignment.Left
                descLb.Parent = f
            end

            local switchBG = Instance.new("Frame")
            switchBG.Size = UDim2.fromOffset(36, 20)
            switchBG.Position = UDim2.new(1, -46, 0.5, -10)
            switchBG.BackgroundColor3 = Color3.fromRGB(200, 210, 230)
            switchBG.BorderSizePixel = 0
            switchBG.Parent = f
            local swC = Instance.new("UICorner")
            swC.CornerRadius = UDim.new(1,0)
            swC.Parent = switchBG

            local knob = Instance.new("Frame")
            knob.Size = UDim2.fromOffset(14, 14)
            knob.Position = UDim2.fromOffset(3, 3)
            knob.BackgroundColor3 = Color3.fromRGB(150, 170, 200)
            knob.BorderSizePixel = 0
            knob.Parent = switchBG
            local kC = Instance.new("UICorner")
            kC.CornerRadius = UDim.new(1,0)
            kC.Parent = knob

            local value = false
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, 0, 1, 0)
            btn.BackgroundTransparency = 1
            btn.Text = ""
            btn.Parent = f

            local toggleObj = {}
            function toggleObj:Set(v)
                value = v
                TweenService:Create(switchBG, TweenInfo.new(0.2, Enum.EasingStyle.Quint), {
                    BackgroundColor3 = v and ACCENT or Color3.fromRGB(200, 210, 230)
                }):Play()
                TweenService:Create(knob, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                    Position = v and UDim2.fromOffset(19, 3) or UDim2.fromOffset(3, 3),
                    BackgroundColor3 = v and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(150, 170, 200)
                }):Play()
                if opts2.Callback then opts2.Callback(v) end
            end

            btn.MouseButton1Click:Connect(function() toggleObj:Set(not value) end)
            return toggleObj
        end

        function tabAPI:AddToggle(opts2)
            return tabAPI:Toggle(opts2)
        end

        function tabAPI:Button(opts2)
            local f = makeElementContainer(scroll)
            f.Size = UDim2.new(1, -8, 0, 40)
            f.LayoutOrder = nextOrder()

            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, 0, 1, 0)
            btn.BackgroundTransparency = 1
            btn.Text = ""
            btn.Parent = f

            local titleLb = Instance.new("TextLabel")
            titleLb.Size = UDim2.new(1, -12, 1, 0)
            titleLb.Position = UDim2.fromOffset(10, 0)
            titleLb.BackgroundTransparency = 1
            titleLb.Text = opts2.Title or ""
            titleLb.TextColor3 = TEXT_WHITE
            titleLb.Font = Enum.Font.GothamBold
            titleLb.TextSize = 13
            titleLb.TextXAlignment = Enum.TextXAlignment.Left
            titleLb.Parent = f

            if opts2.Desc and opts2.Desc ~= "" then
                titleLb.Size = UDim2.new(1, -12, 0.5, 0)
                titleLb.Position = UDim2.fromOffset(10, 4)
                local descLb = Instance.new("TextLabel")
                descLb.Size = UDim2.new(1, -12, 0.5, 0)
                descLb.Position = UDim2.new(0, 10, 0.5, 0)
                descLb.BackgroundTransparency = 1
                descLb.Text = opts2.Desc
                descLb.TextColor3 = TEXT_GRAY
                descLb.Font = Enum.Font.Gotham
                descLb.TextSize = 11
                descLb.TextXAlignment = Enum.TextXAlignment.Left
                descLb.Parent = f
            end

            btn.MouseButton1Click:Connect(function()
                if opts2.Callback then opts2.Callback() end
                tweenSine(f, 0.08, { BackgroundColor3 = Color3.fromRGB(220, 230, 250) }):Play()
                task.wait(0.1)
                tweenSine(f, 0.1, { BackgroundColor3 = BG_ELEM }):Play()
            end)
        end

        function tabAPI:Slider(opts2)
            local minV = opts2.Min or 0
            local maxV = opts2.Max or 100
            local defV = opts2.Default or minV
            local currentVal = defV

            local f = makeElementContainer(scroll)
            f.Size = UDim2.new(1, -8, 0, 64)
            f.LayoutOrder = nextOrder()

            local titleLb = Instance.new("TextLabel")
            titleLb.Size = UDim2.new(1, -60, 0.5, 0)
            titleLb.Position = UDim2.fromOffset(10, 4)
            titleLb.BackgroundTransparency = 1
            titleLb.Text = opts2.Title or ""
            titleLb.TextColor3 = TEXT_WHITE
            titleLb.Font = Enum.Font.GothamBold
            titleLb.TextSize = 13
            titleLb.TextXAlignment = Enum.TextXAlignment.Left
            titleLb.Parent = f

            if opts2.Desc and opts2.Desc ~= "" then
                local descLb = Instance.new("TextLabel")
                descLb.Size = UDim2.new(1, -60, 0, 14)
                descLb.Position = UDim2.new(0, 10, 0, 22)
                descLb.BackgroundTransparency = 1
                descLb.Text = opts2.Desc
                descLb.TextColor3 = TEXT_GRAY
                descLb.Font = Enum.Font.Gotham
                descLb.TextSize = 11
                descLb.TextXAlignment = Enum.TextXAlignment.Left
                descLb.Parent = f
            end

            local valLbl = Instance.new("TextLabel")
            valLbl.Size = UDim2.fromOffset(50, 20)
            valLbl.Position = UDim2.new(1, -58, 0, 4)
            valLbl.BackgroundTransparency = 1
            valLbl.Text = tostring(defV)
            valLbl.TextColor3 = ACCENT_GLOW
            valLbl.Font = Enum.Font.GothamBold
            valLbl.TextSize = 12
            valLbl.TextXAlignment = Enum.TextXAlignment.Right
            valLbl.Parent = f

            local trackBG = Instance.new("Frame")
            trackBG.Size = UDim2.new(1, -20, 0, 5)
            trackBG.Position = UDim2.new(0, 10, 1, -14)
            trackBG.BackgroundColor3 = Color3.fromRGB(200, 210, 230)
            trackBG.BorderSizePixel = 0
            trackBG.Parent = f
            local trC = Instance.new("UICorner")
            trC.CornerRadius = UDim.new(1,0)
            trC.Parent = trackBG

            local fill = Instance.new("Frame")
            fill.Size = UDim2.new((defV - minV) / (maxV - minV), 0, 1, 0)
            fill.BackgroundColor3 = ACCENT
            fill.BorderSizePixel = 0
            fill.Parent = trackBG
            local fC = Instance.new("UICorner")
            fC.CornerRadius = UDim.new(1,0)
            fC.Parent = fill

            local sliderBtn = Instance.new("TextButton")
            sliderBtn.Size = UDim2.new(1, 0, 0, 18)
            sliderBtn.Position = UDim2.new(0, 0, 1, -18)
            sliderBtn.BackgroundTransparency = 1
            sliderBtn.Text = ""
            sliderBtn.Parent = f

            local sliding = false
            local function updateSlider(inputX)
                local absPos = trackBG.AbsolutePosition.X
                local absSize = trackBG.AbsoluteSize.X
                local rel = math.clamp((inputX - absPos) / absSize, 0, 1)
                local rawVal = minV + rel * (maxV - minV)
                currentVal = math.floor(rawVal * 100 + 0.5) / 100
                fill.Size = UDim2.new(rel, 0, 1, 0)
                valLbl.Text = tostring(math.floor(currentVal * 10 + 0.5) / 10)
                if opts2.Callback then opts2.Callback(currentVal) end
            end

            sliderBtn.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    sliding = true
                    updateSlider(input.Position.X)
                end
            end)
            UserInputService.InputChanged:Connect(function(input)
                if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                    updateSlider(input.Position.X)
                end
            end)
            UserInputService.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    sliding = false
                end
            end)
        end

        function tabAPI:Input(opts2)
            local f = makeElementContainer(scroll)
            f.Size = UDim2.new(1, -8, 0, 62)
            f.LayoutOrder = nextOrder()

            local titleLb = Instance.new("TextLabel")
            titleLb.Size = UDim2.new(1, -12, 0, 18)
            titleLb.Position = UDim2.fromOffset(10, 5)
            titleLb.BackgroundTransparency = 1
            titleLb.Text = opts2.Title or ""
            titleLb.TextColor3 = TEXT_WHITE
            titleLb.Font = Enum.Font.GothamBold
            titleLb.TextSize = 13
            titleLb.TextXAlignment = Enum.TextXAlignment.Left
            titleLb.Parent = f

            if opts2.Desc and opts2.Desc ~= "" then
                local descLb = Instance.new("TextLabel")
                descLb.Size = UDim2.new(1, -12, 0, 13)
                descLb.Position = UDim2.fromOffset(10, 23)
                descLb.BackgroundTransparency = 1
                descLb.Text = opts2.Desc
                descLb.TextColor3 = TEXT_GRAY
                descLb.Font = Enum.Font.Gotham
                descLb.TextSize = 11
                descLb.TextXAlignment = Enum.TextXAlignment.Left
                descLb.Parent = f
            end

            local inputBG = Instance.new("Frame")
            inputBG.Size = UDim2.new(1, -20, 0, 22)
            inputBG.Position = UDim2.new(0, 10, 1, -26)
            inputBG.BackgroundColor3 = Color3.fromRGB(235, 240, 255)
            inputBG.BorderSizePixel = 0
            inputBG.Parent = f
            local inC = Instance.new("UICorner")
            inC.CornerRadius = UDim.new(0,5)
            inC.Parent = inputBG
            local inStr = Instance.new("UIStroke")
            inStr.Color = OUTLINE
            inStr.Thickness = 1
            inStr.Parent = inputBG

            local textBox = Instance.new("TextBox")
            textBox.Size = UDim2.new(1, -10, 1, 0)
            textBox.Position = UDim2.fromOffset(5, 0)
            textBox.BackgroundTransparency = 1
            textBox.Text = opts2.Value or ""
            textBox.PlaceholderText = "Enter value..."
            textBox.TextColor3 = TEXT_WHITE
            textBox.PlaceholderColor3 = TEXT_GRAY
            textBox.Font = Enum.Font.Gotham
            textBox.TextSize = 12
            textBox.TextXAlignment = Enum.TextXAlignment.Left
            textBox.ClearTextOnFocus = false
            textBox.Parent = inputBG

            textBox.FocusLost:Connect(function()
                if opts2.Callback then opts2.Callback(textBox.Text) end
                tweenSine(inStr, 0.18, { Color = OUTLINE }):Play()
            end)
            textBox:GetPropertyChangedSignal("Text"):Connect(function()
                tweenSine(inStr, 0.18, { Color = ACCENT }):Play()
            end)
        end

        function tabAPI:Keybind(opts2)
            local f = makeElementContainer(scroll)
            f.LayoutOrder = nextOrder()

            local titleLb = Instance.new("TextLabel")
            titleLb.Size = UDim2.new(1, -80, 1, 0)
            titleLb.Position = UDim2.fromOffset(10, 0)
            titleLb.BackgroundTransparency = 1
            titleLb.Text = opts2.Title or ""
            titleLb.TextColor3 = TEXT_WHITE
            titleLb.Font = Enum.Font.GothamBold
            titleLb.TextSize = 13
            titleLb.TextXAlignment = Enum.TextXAlignment.Left
            titleLb.Parent = f

            local keyBG = Instance.new("Frame")
            keyBG.Size = UDim2.fromOffset(54, 26)
            keyBG.Position = UDim2.new(1, -62, 0.5, -13)
            keyBG.BackgroundColor3 = Color3.fromRGB(235, 240, 255)
            keyBG.BorderSizePixel = 0
            keyBG.Parent = f
            local kbC = Instance.new("UICorner")
            kbC.CornerRadius = UDim.new(0,5)
            kbC.Parent = keyBG
            local kbStr = Instance.new("UIStroke")
            kbStr.Color = OUTLINE
            kbStr.Thickness = 1
            kbStr.Parent = keyBG

            local currentKey = opts2.Default or Enum.KeyCode.Unknown
            local keyLbl = Instance.new("TextLabel")
            keyLbl.Size = UDim2.new(1, 0, 1, 0)
            keyLbl.BackgroundTransparency = 1
            keyLbl.Text = tostring(currentKey.Name or currentKey)
            keyLbl.TextColor3 = ACCENT_GLOW
            keyLbl.Font = Enum.Font.GothamBold
            keyLbl.TextSize = 11
            keyLbl.Parent = keyBG

            local listening = false
            local keyBtn = Instance.new("TextButton")
            keyBtn.Size = UDim2.new(1, 0, 1, 0)
            keyBtn.BackgroundTransparency = 1
            keyBtn.Text = ""
            keyBtn.Parent = keyBG

            keyBtn.MouseButton1Click:Connect(function()
                listening = true
                keyLbl.Text = "..."
                tweenSine(kbStr, 0.15, { Color = ACCENT }):Play()
            end)

            UserInputService.InputBegan:Connect(function(input, gp)
                if listening and not gp then
                    if input.UserInputType == Enum.UserInputType.Keyboard then
                        currentKey = input.KeyCode
                        keyLbl.Text = tostring(input.KeyCode.Name)
                        listening = false
                        tweenSine(kbStr, 0.15, { Color = OUTLINE }):Play()
                    end
                elseif not gp and input.KeyCode == currentKey then
                    if opts2.Callback then opts2.Callback() end
                end
            end)
        end

        return tabAPI
    end

    function windowObj:Section(opts2)
        sectionOrder = sectionOrder + 1
        local sectionObj = {}

        local sLabel = Instance.new("TextLabel")
        sLabel.Size = UDim2.new(1, 0, 0, 18)
        sLabel.BackgroundTransparency = 1
        sLabel.Text = string.upper(opts2.Title or "")
        sLabel.TextColor3 = ACCENT
        sLabel.Font = Enum.Font.GothamBold
        sLabel.TextSize = 10
        sLabel.TextXAlignment = Enum.TextXAlignment.Left
        sLabel.LayoutOrder = sectionOrder * 1000
        sLabel.Parent = TabPanel

        local tabOrder2 = 0
        function sectionObj:Tab(tabOpts)
            tabOrder2 = tabOrder2 + 1

            local tabBtn = Instance.new("Frame")
            tabBtn.Name = tabOpts.Title or "Tab"
            tabBtn.Size = UDim2.new(1, 0, 0, 32)
            tabBtn.BackgroundColor3 = Color3.fromRGB(230, 235, 250)
            tabBtn.BorderSizePixel = 0
            tabBtn.LayoutOrder = sectionOrder * 1000 + tabOrder2
            tabBtn.Parent = TabPanel
            local tbC = Instance.new("UICorner")
            tbC.CornerRadius = UDim.new(0,7)
            tbC.Parent = tabBtn

            local tabTitleLbl = Instance.new("TextLabel")
            tabTitleLbl.Name = "TextLabel"
            tabTitleLbl.Size = UDim2.new(1, -10, 1, 0)
            tabTitleLbl.Position = UDim2.fromOffset(10, 0)
            tabTitleLbl.BackgroundTransparency = 1
            tabTitleLbl.Text = tabOpts.Title or "Tab"
            tabTitleLbl.TextColor3 = TEXT_GRAY
            tabTitleLbl.Font = Enum.Font.Gotham
            tabTitleLbl.TextSize = 13
            tabTitleLbl.TextXAlignment = Enum.TextXAlignment.Left
            tabTitleLbl.Parent = tabBtn

            local tabPage = Instance.new("Frame")
            tabPage.Name = (tabOpts.Title or "Tab") .. "Page"
            tabPage.Size = UDim2.new(1, 0, 1, 0)
            tabPage.BackgroundTransparency = 1
            tabPage.Visible = false
            tabPage.Parent = PageHolder

            local tabClickBtn = Instance.new("TextButton")
            tabClickBtn.Size = UDim2.new(1, 0, 1, 0)
            tabClickBtn.BackgroundTransparency = 1
            tabClickBtn.Text = ""
            tabClickBtn.Parent = tabBtn

            tabClickBtn.MouseButton1Click:Connect(function()
                setActiveTab(tabPage, tabBtn)
            end)

            if currentTab == nil then
                setActiveTab(tabPage, tabBtn)
            end

            local api = buildTabAPI(tabPage)
            return api
        end

        return sectionObj
    end

    return windowObj
end

-- ============================================================
-- SYSTEM LOADER
-- ============================================================
local function ShowSystemLoader(onFinished)
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SnowSystemLoader"
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = getGuiParent()

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = BG_DARK
    bg.BackgroundTransparency = 0
    bg.Parent = ScreenGui

    local accentLine = Instance.new("Frame")
    accentLine.Size = UDim2.new(0, 0, 0, 3)
    accentLine.Position = UDim2.new(0.5, 0, 0.5, 30)
    accentLine.BackgroundColor3 = ACCENT
    accentLine.BorderSizePixel = 0
    accentLine.Parent = bg
    local alC = Instance.new("UICorner")
    alC.CornerRadius = UDim.new(1,0)
    alC.Parent = accentLine

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0.2, 0)
    title.Position = UDim2.new(0, 0, 0.4, 0)
    title.BackgroundTransparency = 1
    title.Text = "â„ SNOW HUB"
    title.TextColor3 = ACCENT
    title.Font = Enum.Font.GothamBold
    title.TextScaled = true
    title.TextTransparency = 1
    title.Parent = bg

    local subtitle = Instance.new("TextLabel")
    subtitle.Size = UDim2.new(1, 0, 0.08, 0)
    subtitle.Position = UDim2.new(0, 0, 0.55, 0)
    subtitle.BackgroundTransparency = 1
    subtitle.TextColor3 = TEXT_GRAY
    subtitle.Font = Enum.Font.Gotham
    subtitle.TextScaled = true
    subtitle.TextTransparency = 1
    subtitle.Parent = bg

    tweenExpo(accentLine, 0.3, { Size = UDim2.new(0.3, 0, 0, 3), Position = UDim2.new(0.35, 0, 0.5, 30) }):Play()
    task.wait(0.15)
    tweenBack(title, 0.35, { TextTransparency = 0 }):Play()
    task.wait(0.2)

    local steps = {"Initializing", "Loading modules", "Creating snowflakes", "Ready!"}
    for _, text in ipairs(steps) do
        subtitle.Text = text
        tweenSine(subtitle, 0.15, { TextTransparency = 0 }):Play()
        task.wait(0.4)
        tweenSine(subtitle, 0.15, { TextTransparency = 1 }):Play()
        task.wait(0.08)
    end

    tweenExpo(accentLine, 0.2, { Size = UDim2.new(1, 0, 0, 3), Position = UDim2.new(0, 0, 0.5, 30) }):Play()
    task.wait(0.1)
    TweenService:Create(bg, TweenInfo.new(0.25, Enum.EasingStyle.Quint), { BackgroundTransparency = 1 }):Play()
    task.wait(0.3)
    ScreenGui:Destroy()
    if onFinished then onFinished() end
end

-- ============================================================
-- LOAD HUB WITH ALL FEATURES
-- ============================================================
local function LoadSnowHub()
    VxnityUI:Notify({ Title = "â„ Snow Hub", Desc = "Loading main script...", Duration = 2 })

    local Window = VxnityUI:CreateWindow({
        Title = "â„ Snow Hub",
        Author = "snow edition"
    })

    -- HOME TAB
    local HomeSection = Window:Section({ Title = "Information" })
    local HomeTab = HomeSection:Tab({ Title = "Home" })

    HomeTab:Section({ Title = "â„ Welcome to Snow Hub" })
    HomeTab:Paragraph({ Title = "Script Version: 3.0", Desc = "Snowflake White Edition" })
    HomeTab:Paragraph({ Title = "User: " .. LocalPlayer.Name, Desc = "Rank: Premium User" })
    
    HomeTab:Section({ Title = "Updates" })
    HomeTab:Paragraph({
        Title = "Latest Update: 2026-08-20",
        Desc = "- FIXED Reach System\n- Avatar Copier (Type any username)\n- Glued Ball Magnet Mode"
    })

    -- REACH TAB (FIXED)
    local Main = Window:Section({ Title = "Main" })
    local ReachTab = Main:Tab({ Title = "Reach" })

    ReachTab:Section({ Title = "Leg Reach - FIXED" })
    
    ReachTab:Paragraph({
        Title = "âš ï¸ How it works",
        Desc = "Uses firetouchinterest on all limbs. Keep distance low (1-3) for best results."
    })
    
    local reachToggle = ReachTab:Toggle({
        Title = "Active Reach",
        Desc = "Enables automatic ball contact",
        Callback = function(v)
            reachEnabled = v
            P.reachEnabled = v
            if v then
                startReach()
                VxnityUI:Notify({ Title = "Reach", Desc = "Enabled! Distance: " .. reachDistance, Duration = 2 })
            else
                if reachConnection then reachConnection:Disconnect(); reachConnection = nil end
                VxnityUI:Notify({ Title = "Reach", Desc = "Disabled", Duration = 1 })
            end
        end
    })

    ReachTab:Slider({
        Title = "Reach Distance",
        Desc = "Range 1-10 (Recommended: 1-3)",
        Min = 1,
        Max = 10,
        Default = 1,
        Callback = function(v)
            reachDistance = v
            P.reachDistance = v
            if reachEnabled then
                startReach()
                VxnityUI:Notify({ Title = "Reach", Desc = "Distance set to: " .. v, Duration = 1.5 })
            end
        end
    })

    ReachTab:Section({ Title = "Leg Hitbox Size" })

    ReachTab:Input({
        Title = "Leg Hitbox (R6)",
        Desc = "Increase leg size for better reach",
        Value = "1",
        Callback = function(Value)
            local v = tonumber(Value) or 1
            if LocalPlayer.Character then
                for _, part in ipairs(LocalPlayer.Character:GetChildren()) do
                    if part:IsA("BasePart") and (part.Name:find("Leg") or part.Name:find("Foot")) then
                        part.Size = Vector3.new(v, 2, v)
                        part.CanCollide = false
                    end
                end
            end
        end
    })

    ReachTab:Input({
        Title = "Leg Hitbox (R15)",
        Desc = "Increase leg size for better reach",
        Value = "1",
        Callback = function(Value)
            local v = tonumber(Value) or 1
            if LocalPlayer.Character then
                for _, part in ipairs(LocalPlayer.Character:GetChildren()) do
                    if part:IsA("BasePart") and (part.Name:find("LowerLeg") or part.Name:find("UpperLeg") or part.Name:find("Foot")) then
                        part.Size = Vector3.new(v, 2, v)
                        part.CanCollide = false
                    end
                end
            end
        end
    })

    -- BALL MAGNET TAB (GLUED MODE)
    local MagnetTab = Main:Tab({ Title = "Ball Magnet" })
    
    MagnetTab:Section({ Title = "ðŸ§² Ball Magnet - Glued Mode" })
    
    MagnetTab:Paragraph({
        Title = "Glued Mode",
        Desc = "Ball sticks to your character like it's glued"
    })
    
    local magnetToggle = MagnetTab:Toggle({
        Title = "Enable Ball Magnet",
        Desc = "Pulls ball towards you like a magnet",
        Callback = function(v)
            ballMagnetEnabled = v
            if v then
                startBallMagnet()
                VxnityUI:Notify({ Title = "Magnet", Desc = "Enabled - " .. (gluedMode and "Glued Mode" or "Pull Mode"), Duration = 2 })
            else
                if magnetConnection then magnetConnection:Disconnect(); magnetConnection = nil end
                VxnityUI:Notify({ Title = "Magnet", Desc = "Disabled", Duration = 1 })
            end
        end
    })
    
    MagnetTab:Toggle({
        Title = "Glued Mode",
        Desc = "Ball stays glued to your character",
        Callback = function(v)
            gluedMode = v
            if ballMagnetEnabled then
                startBallMagnet()
                VxnityUI:Notify({ Title = "Mode", Desc = gluedMode and "Glued Mode" or "Pull Mode", Duration = 1.5 })
            end
        end
    })
    
    MagnetTab:Slider({
        Title = "Magnet Strength",
        Desc = "How strong the pull is",
        Min = 0.1,
        Max = 5.0,
        Default = 1.0,
        Callback = function(v)
            magnetStrength = v
            P.pullStrength = v
        end
    })
    
    MagnetTab:Button({
        Title = "ðŸ’¥ ULTRA GLUE",
        Desc = "Maximum glue strength",
        Callback = function()
            magnetStrength = 5.0
            gluedMode = true
            P.pullStrength = 5.0
            if not ballMagnetEnabled then
                ballMagnetEnabled = true
                magnetToggle:Set(true)
                startBallMagnet()
            else
                startBallMagnet()
            end
            VxnityUI:Notify({ Title = "ULTRA GLUE", Desc = "Ball glued to you!", Duration = 2 })
        end
    })
    
    MagnetTab:Button({
        Title = "ðŸŒ€ SNAP TO PLAYER",
        Desc = "Instantly snaps ball to you",
        Callback = function()
            local ball = _G._VxRBall
            local hrp = _G._VxRHRP
            if ball and hrp then
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                ball.CFrame = CFrame.new(hrp.Position + hrp.CFrame.LookVector * 0.3 + Vector3.new(0, 0.1, 0))
                ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * 100
                VxnityUI:Notify({ Title = "SNAP", Desc = "Ball snapped to you!", Duration = 1.5 })
            end
        end
    })

    -- VISUAL AVATAR COPIER TAB
    local AvatarTab = Main:Tab({ Title = "Avatar" })
    
    AvatarTab:Section({ Title = "â„ Visual Avatar Copier" })
    
    AvatarTab:Paragraph({
        Title = "Type any Roblox username",
        Desc = "Their avatar will be copied visually (client-side only)"
    })
    
    AvatarTab:Input({
        Title = "Username",
        Desc = "Enter a Roblox player name",
        Value = "",
        Callback = function(Value)
            targetUsername = Value
        end
    })
    
    AvatarTab:Button({
        Title = "ðŸ“‹ Copy Avatar",
        Desc = "Copies the entered username's avatar",
        Callback = function()
            if targetUsername and targetUsername ~= "" then
                copyPlayerAvatar(targetUsername)
            else
                VxnityUI:Notify({ Title = "â„ Avatar Copy", Desc = "Please enter a username first!", Duration = 2 })
            end
        end
    })
    
    AvatarTab:Button({
        Title = "ðŸ”„ Refresh Avatar",
        Desc = "Refresh the copied avatar",
        Callback = function()
            if targetUsername and targetUsername ~= "" then
                if avatarClone then avatarClone:Destroy(); avatarClone = nil end
                copyPlayerAvatar(targetUsername)
            end
        end
    })
    
    AvatarTab:Button({
        Title = "ðŸ—‘ï¸ Remove Avatar Copy",
        Desc = "Remove the copied avatar",
        Callback = function()
            if avatarClone then 
                avatarClone:Destroy() 
                avatarClone = nil 
            end
            if avatarCopyConnection then 
                avatarCopyConnection:Disconnect() 
                avatarCopyConnection = nil 
            end
            VxnityUI:Notify({ Title = "Avatar Copy", Desc = "Removed!", Duration = 2 })
        end
    })

    -- REACTS TAB (Preserved)
    local ReactTab = Main:Tab({ Title = "Reacts" })

    ReactTab:Section({ Title = "âš¡ Reacts V6 â€” MAXIMUM POWER" })

    ReactTab:Button({ 
        Title = "ðŸ”¥ ULTRA SPEED", 
        Desc = "Maximum illegal speed",
        Callback = function()
            currentReactPower = 5e18; P.reactPower = currentReactPower
            enableReactHook(); applyReactInstant(currentReactPower)
            VxnityUI:Notify({ Title = "ULTRA SPEED", Desc = "Ball at maximum speed!", Duration = 2 })
        end
    })
    
    ReactTab:Button({ 
        Title = "ðŸ’€ MEGA POWER", 
        Desc = "Extreme power",
        Callback = function()
            currentReactPower = 1e25; P.reactPower = currentReactPower
            enableReactHook(); applyReactInstant(currentReactPower)
            VxnityUI:Notify({ Title = "MEGA POWER", Desc = "Extreme power activated", Duration = 2 })
        end
    })
    
    ReactTab:Button({ 
        Title = "âš¡ HYPER VELOCITY", 
        Desc = "Instant hyper velocity",
        Callback = function()
            currentReactPower = 2e22; P.reactPower = currentReactPower
            enableReactHook(); applyReactInstant(currentReactPower)
            VxnityUI:Notify({ Title = "HYPER", Desc = "Max hyper velocity", Duration = 2 })
        end
    })
    
    ReactTab:Button({ 
        Title = "ðŸš€ ULTIMATE KICK", 
        Desc = "Ultimate kick",
        Callback = function()
            currentReactPower = 8e20; P.reactPower = currentReactPower
            enableReactHook(); applyReactInstant(currentReactPower)
            VxnityUI:Notify({ Title = "ULTIMATE", Desc = "Ultimate kick", Duration = 2 })
        end
    })
    
    ReactTab:Button({ 
        Title = "ðŸ’¥ MAX POWER", 
        Desc = "Absolute maximum power",
        Callback = function()
            currentReactPower = 1e28; P.reactPower = currentReactPower
            enableReactHook(); applyReactInstant(currentReactPower)
            VxnityUI:Notify({ Title = "MAX POWER", Desc = "Absolute max power", Duration = 2 })
        end
    })

    ReactTab:Button({ 
        Title = "ðŸŒ€ HYPER SNAP", 
        Desc = "Super fast magnetic snap",
        Callback = function()
            currentReactPower = 3e20; P.reactPower = currentReactPower
            enableReactHook()
            local ball = _G._VxRBall; local hrp = _G._VxRHRP
            if ball and hrp then
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                ball.CFrame = CFrame.new(hrp.Position + hrp.CFrame.LookVector * 0.2 + Vector3.new(0, 0.1, 0))
                ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * (currentReactPower * ballSpeedMult)
                ball.AssemblyAngularVelocity = Vector3.zero
            end
            VxnityUI:Notify({ Title = "HYPER SNAP", Desc = "Ultra fast snap!", Duration = 2 })
        end
    })

    ReactTab:Button({ 
        Title = "ðŸ’€ MEGA LOCK", 
        Desc = "Super heavy lock",
        Callback = function()
            currentReactPower = 5e25; P.reactPower = currentReactPower
            enableReactHook()
            local ball = _G._VxRBall; local hrp = _G._VxRHRP
            if ball and hrp then
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * (currentReactPower * ballSpeedMult)
                ball.AssemblyAngularVelocity = Vector3.zero
            end
            VxnityUI:Notify({ Title = "MEGA LOCK", Desc = "Super heavy lock!", Duration = 2 })
        end
    })

    ReactTab:Button({ 
        Title = "ðŸ”´ ULTRA PIVOT", 
        Desc = "Ultra fast pivot + stick",
        Callback = function()
            currentReactPower = 4e22; P.reactPower = currentReactPower
            enableReactHook()
            local ball = _G._VxRBall; local hrp = _G._VxRHRP
            if ball and hrp then
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                ball.CFrame = CFrame.new(hrp.Position + hrp.CFrame.LookVector * 0.15)
                ball.AssemblyLinearVelocity = hrp.CFrame.LookVector * (currentReactPower * ballSpeedMult)
                ball.AssemblyAngularVelocity = Vector3.zero
            end
            VxnityUI:Notify({ Title = "ULTRA PIVOT", Desc = "Ultra fast pivot!", Duration = 2 })
        end
    })

    ReactTab:Button({ 
        Title = "âœï¸ LUCIO MAX", 
        Desc = "Max power + prediction",
        Callback = function()
            currentReactPower = 6e23; P.reactPower = currentReactPower
            enableReactHook()
            local ball = _G._VxRBall; local hrp = _G._VxRHRP
            if ball and hrp then
                pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                local look = hrp.CFrame.LookVector
                ball.CFrame = CFrame.new(hrp.Position + look * 0.15 + Vector3.new(0, 0.05, 0))
                ball.AssemblyLinearVelocity = look * (currentReactPower * ballSpeedMult)
                ball.AssemblyAngularVelocity = Vector3.zero
            end
            VxnityUI:Notify({ Title = "LUCIO MAX", Desc = "Max power + prediction!", Duration = 2 })
        end
    })

    ReactTab:Section({ Title = "ðŸŽšï¸ Ball Speed Control" })
    
    ReactTab:Slider({
        Title = "Ball Speed Multiplier",
        Desc = "Real-time speed control (up to 50x)",
        Min = 0.1,
        Max = 50,
        Default = 1.0,
        Callback = function(val)
            ballSpeedMult = val
            P.ballSpeedMult = val
        end
    })

    ReactTab:Section({ Title = "ðŸŽ¯ Velocity Presets" })
    
    ReactTab:Button({ 
        Title = "Normal Mode", 
        Desc = "Multiplier x1.0",
        Callback = function()
            ballSpeedMult = 1.0; P.ballSpeedMult = ballSpeedMult
            VxnityUI:Notify({ Title = "Velocity", Desc = "Normal (x1.0)", Duration = 1.5 })
        end
    })
    
    ReactTab:Button({ 
        Title = "RÃ¡pido Mode", 
        Desc = "Multiplier x5.0",
        Callback = function()
            ballSpeedMult = 5.0; P.ballSpeedMult = ballSpeedMult
            VxnityUI:Notify({ Title = "Velocity", Desc = "RÃ¡pido (x5.0)", Duration = 1.5 })
        end
    })
    
    ReactTab:Button({ 
        Title = "ðŸ”¥ Extremo Mode", 
        Desc = "Multiplier x25.0",
        Callback = function()
            ballSpeedMult = 25.0; P.ballSpeedMult = ballSpeedMult
            VxnityUI:Notify({ Title = "Velocity", Desc = "EXTREMO (x25.0)", Duration = 1.5 })
        end
    })

    ReactTab:Section({ Title = "React Power" })
    
    ReactTab:Slider({
        Title = "React Power (base)",
        Desc = "Maximum base power",
        Min = 1e18,
        Max = 1e30,
        Default = 1e22,
        Callback = function(val)
            currentReactPower = val; P.reactPower = val
        end
    })

    -- HELPERS TAB (Preserved)
    local Misc = Window:Section({ Title = "Utility & Extra" })
    local HelpersTab = Misc:Tab({ Title = "Helpers" })

    HelpersTab:Section({ Title = "Ball Visuals" })

    HelpersTab:Toggle({ 
        Title = "ZZZ helper", 
        Desc = "Highlights the ball's position",
        Callback = function(state)
            if state then
                local part = Instance.new("Part")
                part.Name = "TPS1"; part.Size = Vector3.new(9, 0.1, 9)
                part.Anchored = true; part.BrickColor = BrickColor.new("Bright red")
                part.Transparency = 1; part.CanCollide = false; part.Parent = Workspace
                RunService.RenderStepped:Connect(function()
                    local b = _G._VxRBall
                    if b and part.Parent then part.Position = b.Position - Vector3.new(0, 1, 0) end
                end)
            else
                if Workspace:FindFirstChild("TPS1") then Workspace.TPS1:Destroy() end
            end
        end
    })

    -- INF HELPER (Preserved)
    local toggleEnabled = P.helperEnabled
    local helperActive = P.helperActive
    local magnetMode = P.magnetMode
    local predictMode = P.predictMode
    local multiLockActive = P.spaceLock

    HelpersTab:Section({ Title = "Inf Helper Advanced" })

    HelpersTab:Toggle({ 
        Title = "Inf Helper", 
        Desc = "[B] Toggle | Infinite ball control",
        Callback = function(state)
            toggleEnabled = state; P.helperEnabled = state
            if not state then helperActive = false; P.helperActive = false end
        end
    })
    
    HelpersTab:Toggle({ 
        Title = "Magnet Mode", 
        Desc = "Ball sticks instantly to you",
        Callback = function(state) magnetMode = state; P.magnetMode = state end
    })
    
    HelpersTab:Toggle({ 
        Title = "Predict Mode", 
        Desc = "Anticipates your movement",
        Callback = function(state) predictMode = state; P.predictMode = state end
    })
    
    HelpersTab:Toggle({ 
        Title = "Space Lock", 
        Desc = "Freezes ball in space",
        Callback = function(state) multiLockActive = state; P.spaceLock = state end
    })
    
    HelpersTab:Slider({ 
        Title = "Follow Distance", 
        Min = 0, Max = 10, Default = 0.25,
        Callback = function(val) CONFIG.DEAD_ZONE = val end
    })
    
    HelpersTab:Slider({ 
        Title = "Vertical Offset", 
        Min = -5, Max = 5, Default = -0.20,
        Callback = function(val) CONFIG.VERTICAL_OFFSET = val end
    })

    UserInputService.InputBegan:Connect(function(input, gp)
        if input.KeyCode == Enum.KeyCode.B and not gp and toggleEnabled then
            helperActive = not helperActive; P.helperActive = helperActive
        end
    end)

    -- INF HELPER RENDER LOOP
    RunService.RenderStepped:Connect(function()
        if not (helperActive and toggleEnabled) then
            local ball = _G._VxRBall; if ball then cleanupBall(ball) end
            lockedPos = nil; return
        end
        local ball = _G._VxRBall; local hrp = _G._VxRHRP
        local char = LocalPlayer.Character; local hum = char and char:FindFirstChild("Humanoid")
        if not (ball and hrp and hum) then return end
        if hum.Health <= 0 then return end
        if not ball:IsA("BasePart") then return end
        if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
        pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
        
        local ballPos = ball.Position; local hrpPos = hrp.Position
        local dist = (ballPos - hrpPos).Magnitude
        local att = getOrCreateAtt(ball); local lv = getOrCreateLV(ball, att)
        
        if CONFIG.ANGULAR_KILL then
            local av = getOrCreateAV(ball, att)
            av.AngularVelocity = Vector3.zero; ball.AssemblyAngularVelocity = Vector3.zero
        end
        
        if multiLockActive then
            if not lockedPos then lockedPos = ballPos end
            local toLock = lockedPos - ballPos; local lockDist = toLock.Magnitude
            if lockDist > CONFIG.LOCK_RADIUS then
                lv.VectorVelocity = toLock.Unit * CONFIG.MAGNET_PULL
                ball.AssemblyLinearVelocity = toLock.Unit * CONFIG.MAGNET_PULL
            else
                lv.VectorVelocity = Vector3.zero; ball.AssemblyLinearVelocity = Vector3.zero
            end
            return
        else lockedPos = nil end
        
        local targetPos = predictMode and getPredictedTarget(hrp) 
            or (hrpPos + hrp.CFrame.LookVector * CONFIG.FOLLOW_DISTANCE + Vector3.new(0, CONFIG.VERTICAL_OFFSET, 0))
        local toTarget = targetPos - ballPos; local toTargetDist = toTarget.Magnitude
        
        if magnetMode then
            if toTargetDist > 0.04 then
                local speed = math.clamp(toTargetDist * 1200, 80, CONFIG.MAGNET_PULL)
                lv.VectorVelocity = toTarget.Unit * speed; ball.AssemblyLinearVelocity = toTarget.Unit * speed
            else
                ball.CFrame = CFrame.new(targetPos)
                lv.VectorVelocity = Vector3.zero; ball.AssemblyLinearVelocity = Vector3.zero
            end
            return
        end
        
        if dist > CONFIG.MAX_DISTANCE then
            local dir = (targetPos - ballPos).Unit
            lv.VectorVelocity = dir * CONFIG.STRONG_PULL; ball.AssemblyLinearVelocity = dir * CONFIG.STRONG_PULL
        elseif dist > CONFIG.DEAD_ZONE then
            local speed = math.clamp(toTargetDist * CONFIG.SOFT_PULL, 20, CONFIG.FOLLOW_SPEED)
            lv.VectorVelocity = toTarget.Unit * speed
        else
            lv.VectorVelocity = Vector3.zero
        end
    end)

    -- LUCIO INF HELPER
    HelpersTab:Section({ Title = "Lucio Inf Helper" })
    
    HelpersTab:AddToggle({ 
        Title = "Lucio Inf Helper", 
        Desc = "0 reach + ultra stick + super fast",
        Callback = function(state)
            if state then
                _G.AerialInfUltra = RunService.RenderStepped:Connect(function()
                    local ball = _G._VxRBall; local hrp = _G._VxRHRP
                    if not (ball and ball.Parent and hrp) then return end
                    local char = LocalPlayer.Character
                    local torso = char and (char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso"))
                    local base = torso or hrp
                    if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                    pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                    local targetPos = base.Position + base.CFrame.LookVector * 0.06 + Vector3.new(0, 0.02, 0)
                    local diff = targetPos - ball.Position
                    local dist = diff.Magnitude
                    if dist > 0.015 then
                        local spd = math.clamp(dist * 45000, 1000, 80000)
                        ball.AssemblyLinearVelocity = diff.Unit * spd
                        ball.CFrame = CFrame.new(targetPos)
                    else
                        ball.CFrame = CFrame.new(targetPos)
                        ball.AssemblyLinearVelocity = base.CFrame.LookVector * 150
                    end
                    ball.AssemblyAngularVelocity = Vector3.new(80, 0, 80)
                end)
            else
                if _G.AerialInfUltra then _G.AerialInfUltra:Disconnect(); _G.AerialInfUltra = nil end
            end
        end
    })

    HelpersTab:AddToggle({ 
        Title = "Lucio INF TER/AIR", 
        Desc = "0 reach + ultra stick + super fast",
        Callback = function(state)
            if state then
                _G.KenyahINF = RunService.RenderStepped:Connect(function()
                    local ball = _G._VxRBall; local hrp = _G._VxRHRP
                    if not (ball and ball.Parent and hrp) then return end
                    local char = LocalPlayer.Character
                    local torso = char and (char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso"))
                    local base = torso or hrp
                    if ball.CanCollide then pcall(function() ball.CanCollide = false end) end
                    pcall(function() ball:SetNetworkOwner(LocalPlayer) end)
                    local targetPos = base.Position + base.CFrame.LookVector * 0.06 + Vector3.new(0, 0.02, 0)
                    local diff = targetPos - ball.Position
                    local dist = diff.Magnitude
                    if dist > 0.015 then
                        local speed = math.clamp(dist * 40000, 1200, 70000)
                        ball.AssemblyLinearVelocity = diff.Unit * speed
                        ball.CFrame = CFrame.new(targetPos)
                    else
                        ball.CFrame = CFrame.new(targetPos)
                        ball.AssemblyLinearVelocity = base.CFrame.LookVector * 120
                    end
                    ball.AssemblyAngularVelocity = Vector3.new(80, 0, 80)
                end)
            else
                if _G.KenyahINF then _G.KenyahINF:Disconnect(); _G.KenyahINF = nil end
            end
        end
    })

    -- DRIBBLE ASSIST (Preserved)
    local BallControl = Window:Section({ Title = "Ball Control" })
    local DribbleTab = BallControl:Tab({ Title = "Dribble Assist" })

    local dribbleEnabled = false
    local dribbleIntensity = "High"
    local dribbleActivation = "Click"
    local dribbleActive = false
    local dribbleConnection = nil

    local DRIBBLE_CONFIG = {
        Low = { strength = 30000, deadzone = 0.03, offset = 0.08 },
        Medium = { strength = 60000, deadzone = 0.015, offset = 0.05 },
        High = { strength = 100000, deadzone = 0.008, offset = 0.03 }
    }

    DribbleTab:Section({ Title = "ðŸŽ¯ Dribble ULTRA 0-REACH" })

    DribbleTab:Toggle({
        Title = "Enable Dribble Assist",
        Desc = "0 reach visual + ultra stuck ball",
        Callback = function(state)
            dribbleEnabled = state
            if not state and dribbleConnection then
                dribbleConnection:Disconnect()
                dribbleConnection = nil
                dribbleActive = false
            end
        end
    })

    DribbleTab:Slider({
        Title = "Intensity (1=Low, 2=Med, 3=High)",
        Desc = "Adjust dribble power",
        Min = 1,
        Max = 3,
        Default = 3,
        Callback = function(val)
            local levels = {"Low", "Medium", "High"}
            dribbleIntensity = levels[math.floor(val)] or "High"
        end
    })

    DribbleTab:Toggle({
        Title = "Click/Tap Activation",
        Desc = "Activate with click or tap",
        Callback = function(state)
            if state then dribbleActivation = "Click" end
        end
    })

    DribbleTab:Toggle({
        Title = "Hold Activation",
        Desc = "Activate by holding",
        Callback = function(state)
            if state then dribbleActivation = "Hold" end
        end
    })

    local function applyDribbleAssist()
        if not dribbleEnabled then return end
        local ball = _G._VxRBall
        local hrp = _G._VxRHRP
        if not (ball and hrp) then return end

        local config = DRIBBLE_CONFIG[dribbleIntensity] or DRIBBLE_CONFIG.High

        pcall(function() ball.CanCollide = false end)
        pcall(function() ball:SetNetworkOwner(LocalPlayer) end)

        local targetPos = hrp.Position + hrp.CFrame.LookVector * config.offset + Vector3.new(0, 0.02, 0)
        local diff = targetPos - ball.Position
        local dist = diff.Magnitude

        if dist > config.deadzone then
            local speed = math.clamp(dist * config.strength, 800, config.strength)
            ball.AssemblyLinearVelocity = diff.Unit * speed + hrp.CFrame.LookVector * 80
            ball.CFrame = CFrame.new(targetPos)
        else
            ball.CFrame = CFrame.new(targetPos)
            ball.AssemblyLinearVelocity = ball.AssemblyLinearVelocity * 0.85 + hrp.CFrame.LookVector * 60
        end

        ball.AssemblyAngularVelocity = Vector3.new(50, 0, 50)
    end

    dribbleConnection = RunService.RenderStepped:Connect(function()
        if dribbleEnabled and dribbleActive then
            applyDribbleAssist()
        end
    end)

    local function handleDribbleInput(input, gp)
        if not dribbleEnabled then return end
        if dribbleActivation == "Click" then
            if input.UserInputType == Enum.UserInputType.MouseButton1
                or input.UserInputType == Enum.UserInputType.Touch
                or input.UserInputType == Enum.UserInputType.FingerTouch then
                dribbleActive = true
                applyDribbleAssist()
                task.delay(0.05, function() dribbleActive = false end)
            end
        elseif dribbleActivation == "Hold" then
            if (input.UserInputType == Enum.UserInputType.MouseButton1
                or input.UserInputType == Enum.UserInputType.Touch
                or input.UserInputType == Enum.UserInputType.FingerTouch) and not gp then
                dribbleActive = true
            end
        end
    end

    local function handleDribbleEnd(input, gp)
        if dribbleActivation == "Hold" then
            if input.UserInputType == Enum.UserInputType.MouseButton1
                or input.UserInputType == Enum.UserInputType.Touch
                or input.UserInputType == Enum.UserInputType.FingerTouch then
                dribbleActive = false
            end
        end
    end

    UserInputService.InputBegan:Connect(handleDribbleInput)
    UserInputService.InputEnded:Connect(handleDribbleEnd)

    if UserInputService.TouchEnabled then
        UserInputService.TouchTap:Connect(function(touchPos, gp)
            if dribbleEnabled and dribbleActivation == "Click" then
                dribbleActive = true
                applyDribbleAssist()
                task.delay(0.05, function() dribbleActive = false end)
            end
        end)
    end

    -- AIMBOT TAB (Preserved)
    local AimbotTab = Misc:Tab({ Title = "Aimbot" })
    
    local isAimbotEnabled = false
    local aimbotTargetPos = Vector3.new(0, 14, 157)
    local laser = Instance.new("Part")
    laser.Name = "SnowHub Aimbot"
    laser.Anchored = true
    laser.CanCollide = false
    laser.Material = Enum.Material.Neon
    laser.Color = Color3.fromRGB(100, 180, 255)
    laser.Transparency = 1
    laser.Size = Vector3.new(0.05, 0.05, 1)
    laser.Parent = Workspace

    local function toggleAimbot(state)
        isAimbotEnabled = state
        laser.Transparency = isAimbotEnabled and 0.4 or 1
    end

    RunService:BindToRenderStep("SnowAimbotLoop", Enum.RenderPriority.Camera.Value + 1, function()
        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local torso = char and (char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso"))

        if isAimbotEnabled and hrp and torso then
            local hrpPos = hrp.Position
            local lookTarget = Vector3.new(aimbotTargetPos.X, hrpPos.Y, aimbotTargetPos.Z)
            hrp.CFrame = CFrame.lookAt(hrpPos, lookTarget)

            local startPos = torso.Position + Vector3.new(0, 0.8, 0)
            local distance = (aimbotTargetPos - startPos).Magnitude
            laser.Size = Vector3.new(0.05, 0.05, distance)
            laser.CFrame = CFrame.lookAt(startPos, aimbotTargetPos) * CFrame.new(0, 0, -distance/2)
        end
    end)

    AimbotTab:Section({ Title = "Aimbot Settings" })

    local AimbotToggle = AimbotTab:Toggle({
        Title = "Enable / Disable Aimbot",
        Callback = function(state)
            toggleAimbot(state)
        end
    })

    AimbotTab:Keybind({
        Title = "Aimbot Keybind",
        Default = Enum.KeyCode.R,
        Callback = function()
            local newState = not isAimbotEnabled
            AimbotToggle:Set(newState)
        end
    })

    -- OPTIMIZATION TAB (Preserved)
    local Optimization = Window:Section({ Title = "Optimizations" })
    local OptTab = Optimization:Tab({ Title = "Performance" })

    OptTab:Section({ Title = "âš¡ Improve Ping" })

    OptTab:Toggle({
        Title = "Optimize Network Calls",
        Desc = "Reduce network latency",
        Callback = function(state)
            if state then
                VxnityUI:Notify({ Title = "Ping Optimize", Desc = "Network optimization enabled", Duration = 2 })
            end
        end
    })

    OptTab:Toggle({
        Title = "Reduce Packet Loss",
        Desc = "Optimize remote event handling",
        Callback = function(state)
            if state then
                pcall(function() settings().Network.Physics = 30 end)
                VxnityUI:Notify({ Title = "Packet Loss", Desc = "Optimization enabled", Duration = 2 })
            end
        end
    })

    OptTab:Section({ Title = "ðŸ–¥ï¸ Improve CPU" })

    OptTab:Toggle({
        Title = "Optimize Loops",
        Desc = "Reduce render loop iterations",
        Callback = function(state)
            if state then
                VxnityUI:Notify({ Title = "CPU Optimize", Desc = "Loop optimization enabled", Duration = 2 })
            end
        end
    })

    OptTab:Toggle({
        Title = "Optimize Physics",
        Desc = "Reduce physics calculations",
        Callback = function(state)
            if state then
                pcall(function() settings().Physics.PhysicsEngine = "Voxel" end)
                VxnityUI:Notify({ Title = "Physics", Desc = "Physics optimization enabled", Duration = 2 })
            end
        end
    })

    OptTab:Section({ Title = "ðŸŽ¨ Improve GPU" })

    OptTab:Toggle({
        Title = "Optimize UI Rendering",
        Desc = "Reduce UI redraw frequency",
        Callback = function(state)
            if state then
                VxnityUI:Notify({ Title = "GPU Optimize", Desc = "UI optimization enabled", Duration = 2 })
            end
        end
    })

    OptTab:Toggle({
        Title = "Reduce Visual Effects",
        Desc = "Minimize particle effects",
        Callback = function(state)
            if state then
                pcall(function() settings().Rendering.EffectsQuality = 0 end)
                VxnityUI:Notify({ Title = "Effects", Desc = "Effects optimization enabled", Duration = 2 })
            end
        end
    })

    VxnityUI:Notify({ Title = "â„ Snow Hub", Desc = "Welcome! All features loaded.", Duration = 4 })
end

-- ============================================================
-- EXECUTION
-- ============================================================
ShowSystemLoader(function()
    task.wait(0.1)
    LoadSnowHub()
end)
