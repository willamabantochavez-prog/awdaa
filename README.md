-- Eclipse Hub • Realistic cinematic Roblox intro
-- Place this in a LocalScript or run via executor

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local alive = true
local connections = {}
local activeTweens = {}

-- FIX 1: Limpiar instancia anterior si existe
local oldGui = playerGui:FindFirstChild("EclipseHub_CinematicIntro")
if oldGui then oldGui:Destroy() end

local gui -- Declarado antes para referencia en cleanup

local function trackConnection(connection)
    table.insert(connections, connection)
    return connection
end

local function playTween(instance, info, properties)
    if not instance or not alive then return nil end
    local animation = TweenService:Create(instance, info, properties)
    table.insert(activeTweens, animation)
    animation:Play()
    return animation
end

local function make(className, properties, parent)
    local instance = Instance.new(className)
    for property, value in pairs(properties) do
        instance[property] = value
    end
    instance.Parent = parent
    return instance
end

-- FIX 2: Cleanup idempotente con pcall de seguridad
local function cleanup()
    if not alive then return end
    alive = false
    for _, connection in ipairs(connections) do
        pcall(function() connection:Disconnect() end)
    end
    for _, animation in ipairs(activeTweens) do
        pcall(function() animation:Cancel() end)
    end
    table.clear(connections)
    table.clear(activeTweens)
    if gui and gui.Parent then
        gui:Destroy()
    end
end

gui = make("ScreenGui", {
    Name = "EclipseHub_CinematicIntro",
    IgnoreGuiInset = true,
    ResetOnSpawn = false,
    DisplayOrder = 999999,
    ZIndexBehavior = Enum.ZIndexBehavior.Global,
}, playerGui)

-- FIX 3: Reemplazar task.defer (falla en algunos executors) con task.spawn
task.spawn(function()
    task.wait(0.1)
    if not gui.Parent then
        cleanup()
    end
end)

local backdrop = make("Frame", {
    Name = "Backdrop",
    Size = UDim2.fromScale(1, 1),
    BackgroundColor3 = Color3.fromRGB(2, 3, 10),
    BorderSizePixel = 0,
}, gui)

make("UIGradient", {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(1, 2, 8)),
        ColorSequenceKeypoint.new(0.48, Color3.fromRGB(12, 8, 27)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(1, 2, 7)),
    }),
    Rotation = 28,
}, backdrop)

local starLayer = make("Frame", {
    Name = "StarLayer",
    Size = UDim2.fromScale(1, 1),
    BackgroundTransparency = 1,
}, backdrop)

-- FIX 4: Reemplazar Random.new() con math.random() para compatibilidad
local starData = {}
for index = 1, 56 do
    local size = math.random(15, 34) / 10
    local star = make("Frame", {
        Name = "Star_" .. index,
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(math.random(), math.random()),
        Size = UDim2.fromOffset(size, size),
        BackgroundColor3 = index % 4 == 0 and Color3.fromRGB(255, 217, 159) or Color3.fromRGB(191, 211, 255),
        BackgroundTransparency = math.random(18, 78) / 100,
        BorderSizePixel = 0,
        Rotation = 45,
        ZIndex = 2,
    }, starLayer)
    make("UICorner", { CornerRadius = UDim.new(1, 0) }, star)
    starData[index] = {
        frame = star,
        x = star.Position.X.Scale,
        y = star.Position.Y.Scale,
        vx = (math.random() - 0.5) * 0.14,
        vy = math.random(45, 160) / 1000,
    }
end

local intro = make("Frame", {
    Name = "Intro",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.8, 0.28),
    BackgroundTransparency = 1,
    ZIndex = 10,
}, backdrop)

local introGlow = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.52),
    Size = UDim2.fromScale(0.42, 0.05),
    BackgroundColor3 = Color3.fromRGB(101, 74, 255),
    BackgroundTransparency = 0.72,
    BorderSizePixel = 0,
}, intro)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, introGlow)
playTween(introGlow, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
    Size = UDim2.fromScale(0.62, 0.07),
    BackgroundTransparency = 0.45,
})

local kicker = make("TextLabel", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.22),
    Size = UDim2.fromScale(1, 0.18),
    BackgroundTransparency = 1,
    Font = Enum.Font.GothamMedium,
    Text = "By J4kpot MD",
    TextColor3 = Color3.fromRGB(152, 162, 205),
    TextSize = 12,
    TextTransparency = 1,
}, intro)

local title = make("TextLabel", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.43),
    Size = UDim2.fromScale(1, 0.42),
    BackgroundTransparency = 1,
    Font = Enum.Font.GothamBlack,
    Text = "ECLIPSE HUB",
    TextColor3 = Color3.fromRGB(245, 247, 255),
    TextScaled = true,
    TextTransparency = 1,
}, intro)

local rule = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.73),
    Size = UDim2.fromScale(0.28, 0.012),
    BackgroundColor3 = Color3.fromRGB(128, 103, 255),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
}, intro)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, rule)

local subline = make("TextLabel", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.87),
    Size = UDim2.fromScale(1, 0.16),
    BackgroundTransparency = 1,
    Font = Enum.Font.Gotham,
    Text = "https://discord.gg/T7WYrJzU9",
    TextColor3 = Color3.fromRGB(178, 184, 214),
    TextSize = 11,
    TextTransparency = 1,
}, intro)

local fadeIn = TweenInfo.new(0.42, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
playTween(kicker, fadeIn, { TextTransparency = 0 })
playTween(title, fadeIn, { TextTransparency = 0 })
playTween(rule, TweenInfo.new(0.6, Enum.EasingStyle.Quint), { BackgroundTransparency = 0 })
playTween(subline, TweenInfo.new(0.6, Enum.EasingStyle.Quint, Enum.EasingDirection.Out, 0, false, 0.12), { TextTransparency = 0 })

-- Fase 1: 1.5 segundos
task.wait(1.5)
if not alive or not gui.Parent then cleanup() return end

local eclipse = make("Frame", {
    Name = "RealisticEclipse",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.9, 0.9),
    BackgroundTransparency = 1,
    ZIndex = 8,
}, backdrop)
make("UIAspectRatioConstraint", {
    AspectRatio = 1,
    DominantAxis = Enum.DominantAxis.Height,
}, eclipse)

local eclipseScale = make("UIScale", { Scale = 0.22 }, eclipse)

-- Corona exterior
local outerCorona = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.72, 0.72),
    BackgroundTransparency = 1,
}, eclipse)
make("UIAspectRatioConstraint", { AspectRatio = 1 }, outerCorona)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, outerCorona)
local outerStroke = make("UIStroke", {
    Color = Color3.fromRGB(126, 101, 255),
    Thickness = 3,
    Transparency = 0.68,
}, outerCorona)

-- Corona interior
local innerCorona = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.62, 0.62),
    BackgroundTransparency = 1,
}, eclipse)
make("UIAspectRatioConstraint", { AspectRatio = 1 }, innerCorona)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, innerCorona)
local innerStroke = make("UIStroke", {
    Color = Color3.fromRGB(255, 214, 147),
    Thickness = 4,
    Transparency = 0.24,
}, innerCorona)

-- Rayos de la corona
local rays = make("Frame", {
    Size = UDim2.fromScale(1, 1),
    BackgroundTransparency = 1,
}, eclipse)
for index = 1, 18 do
    local ray = make("Frame", {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        Size = UDim2.fromScale(math.random(48, 78) / 100, math.random(2, 6) / 1000),
        Rotation = (index * 20) + math.random(-7, 7),
        BackgroundColor3 = index % 3 == 0 and Color3.fromRGB(255, 202, 124) or Color3.fromRGB(164, 137, 255),
        BackgroundTransparency = math.random(55, 82) / 100,
        BorderSizePixel = 0,
    }, rays)
    make("UIGradient", {
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 1),
            NumberSequenceKeypoint.new(0.35, 0.35),
            NumberSequenceKeypoint.new(0.5, 0),
            NumberSequenceKeypoint.new(0.65, 0.35),
            NumberSequenceKeypoint.new(1, 1),
        }),
    }, ray)
end

-- Anillo solar
local sun = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.515, 0.5),
    Size = UDim2.fromScale(0.54, 0.54),
    BackgroundColor3 = Color3.fromRGB(255, 183, 91),
    BorderSizePixel = 0,
}, eclipse)
make("UIAspectRatioConstraint", { AspectRatio = 1 }, sun)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, sun)
local sunStroke = make("UIStroke", {
    Color = Color3.fromRGB(255, 231, 178),
    Thickness = 5,
    Transparency = 0.12,
}, sun)

-- Sombra lunar
local moon = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.49, 0.5),
    Size = UDim2.fromScale(0.48, 0.48),
    BackgroundColor3 = Color3.fromRGB(1, 2, 7),
    BorderSizePixel = 0,
    ZIndex = 4,
}, eclipse)
make("UIAspectRatioConstraint", { AspectRatio = 1 }, moon)
make("UICorner", { CornerRadius = UDim.new(1, 0) }, moon)
make("UIStroke", {
    Color = Color3.fromRGB(23, 22, 43),
    Thickness = 2,
    Transparency = 0.05,
}, moon)

-- Flare horizontal
local horizontalFlare = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.96, 0.006),
    BackgroundColor3 = Color3.fromRGB(194, 169, 255),
    BackgroundTransparency = 0.62,
    BorderSizePixel = 0,
    ZIndex = 6,
}, eclipse)
make("UIGradient", {
    Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.34, 0.55),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(0.66, 0.55),
        NumberSequenceKeypoint.new(1, 1),
    }),
}, horizontalFlare)

-- Flare vertical
local verticalFlare = make("Frame", {
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.006, 0.8),
    BackgroundColor3 = Color3.fromRGB(170, 145, 255),
    BackgroundTransparency = 0.72,
    BorderSizePixel = 0,
    ZIndex = 6,
}, eclipse)
make("UIGradient", {
    Rotation = 90,
    Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.35, 0.65),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(0.65, 0.65),
        NumberSequenceKeypoint.new(1, 1),
    }),
}, verticalFlare)

-- Overlay de fade
local fadeOverlay = make("Frame", {
    Size = UDim2.fromScale(1, 1),
    BackgroundColor3 = Color3.fromRGB(0, 0, 3),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ZIndex = 50,
}, gui)

-- Fase 2: Fade out texto, expandir eclipse
local fadeOut = TweenInfo.new(0.28, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
playTween(kicker, fadeOut, { TextTransparency = 1 })
playTween(title, fadeOut, { TextTransparency = 1 })
playTween(rule, fadeOut, { BackgroundTransparency = 1 })
playTween(subline, fadeOut, { TextTransparency = 1 })
playTween(eclipseScale, TweenInfo.new(0.72, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
    Scale = 1,
})
playTween(outerStroke, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
    Transparency = 0.38,
    Thickness = 6,
})
playTween(innerStroke, TweenInfo.new(1.05, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
    Transparency = 0.02,
    Thickness = 7,
})
playTween(sunStroke, TweenInfo.new(0.9, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
    Transparency = 0.5,
    Thickness = 9,
})

-- FIX 5: Reemplazar os.clock() con tick() para compatibilidad
local phaseStarted = tick()
local fadeStarted = false

trackConnection(RunService.Heartbeat:Connect(function(deltaTime)
    if not alive then return end
    for _, star in ipairs(starData) do
        star.x += star.vx * deltaTime
        star.y += star.vy * deltaTime
        if star.y > 1.08 then star.y = -0.08 end
        if star.x > 1.08 then star.x = -0.08 end
        if star.x < -0.08 then star.x = 1.08 end
        star.frame.Position = UDim2.fromScale(star.x, star.y)
    end
    -- FIX 6: Evitar crear múltiples tweens del fadeOverlay
    if not fadeStarted and tick() - phaseStarted >= 2.15 then
        fadeStarted = true
        playTween(fadeOverlay, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            BackgroundTransparency = 0,
        })
    end
end))

-- Fase 2: 2.5 segundos
task.wait(2.5)
cleanup()

print("⚡ Eclipse Hub Intro completada")
