local library = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/Library.lua"
))()
local ThemeManager = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/addons/ThemeManager.lua"
))()
local SaveManager = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/addons/SaveManager.lua"
))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local SilentAim = {
    Enabled = false,
    FOV = 150,
    TargetPart = "Head",
    ShowFOV = true,
}

local BulletSpeed = {
    Enabled = false,
    Speed = 1000,
}

local NoSpread = {
    Enabled = false,
}

local Spinbot = {
    Enabled = false,
    Speed = 10,
}

local FOVField = {
    Enabled = false,
    Value = 70,
}

local ESP = {
    Box = false,
    HealthBar = false,
    Name = false,
    Distance = false,
}

local espObjects = {}

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.NumSides = 50
FOVCircle.Filled = false
FOVCircle.Visible = true
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Transparency = 1

local function isPlayerValid(player)
    if not player or player == LocalPlayer then return false end
    local char = player.Character
    if not char then return false end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return false end
    return true
end

local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = SilentAim.FOV

    for _, player in pairs(Players:GetPlayers()) do
        if isPlayerValid(player) then
            local char = player.Character
            local targetPart = char:FindFirstChild(SilentAim.TargetPart) or char:FindFirstChild("HumanoidRootPart")

            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local mousePos = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end

    return closestPlayer
end

local function createESP(player)
    local obj = {}

    obj.Box = Drawing.new("Square")
    obj.Box.Thickness = 1
    obj.Box.Filled = false
    obj.Box.Color = Color3.fromRGB(255, 255, 255)
    obj.Box.Visible = false

    obj.HealthBarBG = Drawing.new("Square")
    obj.HealthBarBG.Thickness = 1
    obj.HealthBarBG.Filled = true
    obj.HealthBarBG.Color = Color3.fromRGB(0, 0, 0)
    obj.HealthBarBG.Visible = false

    obj.HealthBar = Drawing.new("Square")
    obj.HealthBar.Thickness = 1
    obj.HealthBar.Filled = true
    obj.HealthBar.Color = Color3.fromRGB(0, 255, 0)
    obj.HealthBar.Visible = false

    obj.Name = Drawing.new("Text")
    obj.Name.Size = 14
    obj.Name.Center = true
    obj.Name.Outline = true
    obj.Name.Color = Color3.fromRGB(255, 255, 255)
    obj.Name.Visible = false

    obj.Distance = Drawing.new("Text")
    obj.Distance.Size = 14
    obj.Distance.Center = true
    obj.Distance.Outline = true
    obj.Distance.Color = Color3.fromRGB(255, 255, 255)
    obj.Distance.Visible = false

    espObjects[player] = obj
end

local function removeESP(player)
    local obj = espObjects[player]
    if not obj then return end
    for _, v in pairs(obj) do
        v:Remove()
    end
    espObjects[player] = nil
end

local function updateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end

        if not espObjects[player] then
            createESP(player)
        end

        local obj = espObjects[player]
        local char = player.Character
        local humanoid = char and char:FindFirstChild("Humanoid")
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")

        local anyEnabled = ESP.Box or ESP.HealthBar or ESP.Name or ESP.Distance

        if not anyEnabled or not hrp or not head or not humanoid or humanoid.Health <= 0 then
            obj.Box.Visible = false
            obj.HealthBar.Visible = false
            obj.HealthBarBG.Visible = false
            obj.Name.Visible = false
            obj.Distance.Visible = false
            continue
        end

        local hrpScreen, hrpOnScreen = Camera:WorldToViewportPoint(hrp.Position)
        local headScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, head.Size.Y / 2, 0))
        local footScreen = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))

        if not hrpOnScreen then
            obj.Box.Visible = false
            obj.HealthBar.Visible = false
            obj.HealthBarBG.Visible = false
            obj.Name.Visible = false
            obj.Distance.Visible = false
            continue
        end

        local height = math.abs(headScreen.Y - footScreen.Y)
        local width = height * 0.6
        local x = hrpScreen.X - width / 2
        local y = headScreen.Y

        obj.Box.Visible = ESP.Box
        obj.Box.Position = Vector2.new(x, y)
        obj.Box.Size = Vector2.new(width, height)

        local healthPct = humanoid.Health / humanoid.MaxHealth
        local barWidth = 4
        local barX = x - barWidth - 2

        obj.HealthBarBG.Visible = ESP.HealthBar
        obj.HealthBarBG.Position = Vector2.new(barX, y)
        obj.HealthBarBG.Size = Vector2.new(barWidth, height)

        obj.HealthBar.Visible = ESP.HealthBar
        obj.HealthBar.Position = Vector2.new(barX, y + height * (1 - healthPct))
        obj.HealthBar.Size = Vector2.new(barWidth, height * healthPct)
        obj.HealthBar.Color = Color3.fromRGB(255 * (1 - healthPct), 255 * healthPct, 0)

        obj.Name.Visible = ESP.Name
        obj.Name.Text = player.Name
        obj.Name.Position = Vector2.new(hrpScreen.X, y - 16)

        local localChar = LocalPlayer.Character
        local localHRP = localChar and localChar:FindFirstChild("HumanoidRootPart")
        local dist = localHRP and math.floor((hrp.Position - localHRP.Position).Magnitude) or 0
        obj.Distance.Visible = ESP.Distance
        obj.Distance.Text = dist .. " studs"
        obj.Distance.Position = Vector2.new(hrpScreen.X, y + height + 2)
    end
end

local function applySkybox(id)
    local sky = Lighting:FindFirstChildOfClass("Sky")
    if not sky then
        sky = Instance.new("Sky")
        sky.Parent = Lighting
    end
    local url = "rbxassetid://" .. id
    sky.SkyboxBk = url
    sky.SkyboxDn = url
    sky.SkyboxFt = url
    sky.SkyboxLf = url
    sky.SkyboxRt = url
    sky.SkyboxUp = url
end

local function removeSkybox()
    local sky = Lighting:FindFirstChildOfClass("Sky")
    if sky then sky:Destroy() end
end

Players.PlayerRemoving:Connect(removeESP)

local Projectile = require(ReplicatedStorage.ModuleScripts.GunModules.Projectile)
local oldCast = Projectile.Cast

Projectile.Cast = function(self, bulletId, origin, endpoint, force)
    if NoSpread.Enabled then
        endpoint = origin + Camera.CFrame.LookVector * 800
    end
    if SilentAim.Enabled then
        local target = getClosestPlayer()
        if target then
            local char = target.Character
            local targetPart = char:FindFirstChild(SilentAim.TargetPart) or char:FindFirstChild("HumanoidRootPart")
            if targetPart then
                endpoint = targetPart.Position
            end
        end
    end
    if BulletSpeed.Enabled then
        force = BulletSpeed.Speed
    end
    return oldCast(self, bulletId, origin, endpoint, force)
end

local spinAngle = 0

RunService.RenderStepped:Connect(function(dt)
    if FOVField.Enabled then
        Camera.FieldOfView = FOVField.Value
    end

    if Spinbot.Enabled then
        local char = LocalPlayer.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                spinAngle = spinAngle + Spinbot.Speed * dt * 60
                local pos = hrp.CFrame.Position
                hrp.CFrame = CFrame.new(pos) * CFrame.Angles(0, math.rad(spinAngle), 0)
            end
        end
    end

    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Radius = SilentAim.FOV
    FOVCircle.Visible = SilentAim.ShowFOV and SilentAim.Enabled
    FOVCircle.Color = Color3.fromRGB(255, 255, 255)

    updateESP()
end)

local window = library:CreateWindow({
    Title = "Dask | Menu",
    Center = true,
    AutoShow = true
})

local MainTab = window:AddTab("Main")
local ESPTab = window:AddTab("ESP")
local ConfigTab = window:AddTab("Config")

local SilentAimBox = MainTab:AddLeftGroupbox("Silent Aim")
local RightBox = MainTab:AddRightGroupbox("Misc")
local ESPBox = ESPTab:AddLeftGroupbox("ESP")
local ThemeBox = ConfigTab:AddLeftGroupbox("Theme")
local SaveBox = ConfigTab:AddRightGroupbox("Config")

SilentAimBox:AddToggle("EnableSilentAim", {
    Text = "Enable Silent Aim",
    Default = false,
    Callback = function(value)
        SilentAim.Enabled = value
    end
})

SilentAimBox:AddSlider("FOVSlider", {
    Text = "FOV Size",
    Default = 150,
    Min = 10,
    Max = 500,
    Rounding = 0,
    Callback = function(value)
        SilentAim.FOV = value
    end
})

SilentAimBox:AddToggle("ShowFOV", {
    Text = "Show FOV Circle",
    Default = true,
    Callback = function(value)
        SilentAim.ShowFOV = value
    end
})

SilentAimBox:AddDropdown("TargetPart", {
    Values = {"Head", "HumanoidRootPart", "Torso", "UpperTorso"},
    Default = 1,
    Multi = false,
    Text = "Target Part",
    Callback = function(value)
        SilentAim.TargetPart = value
    end
})

SilentAimBox:AddToggle("EnableFOVField", {
    Text = "Custom FOV",
    Default = false,
    Callback = function(value)
        FOVField.Enabled = value
        if not value then
            Camera.FieldOfView = 70
        end
    end
})

SilentAimBox:AddSlider("FOVField", {
    Text = "FOV Field",
    Default = 70,
    Min = 70,
    Max = 120,
    Rounding = 0,
    Callback = function(value)
        FOVField.Value = value
    end
})

SilentAimBox:AddInput("SkyboxID", {
    Default = "",
    Numeric = true,
    Finished = true,
    Text = "Skybox ID",
    Callback = function(value)
        if value and value ~= "" then
            applySkybox(value)
        else
            removeSkybox()
        end
    end
})

RightBox:AddToggle("EnableBulletSpeed", {
    Text = "Enable Bullet Speed",
    Default = false,
    Callback = function(value)
        BulletSpeed.Enabled = value
    end
})

RightBox:AddSlider("BulletSpeedSlider", {
    Text = "Speed",
    Default = 1000,
    Min = 0,
    Max = 3000,
    Rounding = 0,
    Callback = function(value)
        BulletSpeed.Speed = value
    end
})

RightBox:AddToggle("NoSpread", {
    Text = "No Spread",
    Default = false,
    Callback = function(value)
        NoSpread.Enabled = value
    end
})

RightBox:AddToggle("EnableSpinbot", {
    Text = "Spinbot",
    Default = false,
    Callback = function(value)
        Spinbot.Enabled = value
        spinAngle = 0
    end
})

RightBox:AddSlider("SpinbotSpeed", {
    Text = "Spin Speed",
    Default = 10,
    Min = 1,
    Max = 50,
    Rounding = 0,
    Callback = function(value)
        Spinbot.Speed = value
    end
})

ESPBox:AddToggle("ESPBox", {
    Text = "Box",
    Default = false,
    Callback = function(value)
        ESP.Box = value
    end
})

ESPBox:AddToggle("ESPHealthBar", {
    Text = "Health Bar",
    Default = false,
    Callback = function(value)
        ESP.HealthBar = value
    end
})

ESPBox:AddToggle("ESPName", {
    Text = "Name",
    Default = false,
    Callback = function(value)
        ESP.Name = value
    end
})

ESPBox:AddToggle("ESPDistance", {
    Text = "Distance",
    Default = false,
    Callback = function(value)
        ESP.Distance = value
    end
})

ThemeManager:SetLibrary(library)
SaveManager:SetLibrary(library)
SaveManager:SetFolder("SilentAim")
ThemeManager:ApplyToGroupbox(ThemeBox)
SaveManager:ApplyToGroupbox(SaveBox)
SaveManager:LoadAutoLoad()

library:Notify("Dask | Menu loaded!", 3)
