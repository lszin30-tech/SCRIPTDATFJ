--[[
    TFJ · Hitbox Universal
    Roblox / qualquer jogo player vs player
    H            -> liga/desliga a hitbox
    RightShift   -> esconde/mostra o menu
]]

local cloneref = cloneref or function(i) return i end
local function svc(name)
    return cloneref(game:GetService(name))
end

local Players = svc("Players")
local RunService = svc("RunService")
local UserInputService = svc("UserInputService")
local TweenService = svc("TweenService")
local okCore, CoreGui = pcall(svc, "CoreGui")

local LocalPlayer = Players.LocalPlayer
local genv = (getgenv and getgenv()) or _G

-- recarregar o script limpa a instancia anterior
if genv.NexusHitboxUnload then
    pcall(genv.NexusHitboxUnload)
end

----------------------------------------------------------------------
-- config / tema
----------------------------------------------------------------------
local Config = {
    Enabled = false,
    Size = 12,
    Transparency = 0.65,
    Part = "HumanoidRootPart",
    TeamCheck = true,
    IgnoreDead = true,
    Color = Color3.fromRGB(255, 80, 80),
}

local Theme = {
    Bg = Color3.fromRGB(16, 17, 21),
    Panel = Color3.fromRGB(22, 23, 29),
    Element = Color3.fromRGB(30, 32, 40),
    Stroke = Color3.fromRGB(46, 49, 61),
    Text = Color3.fromRGB(235, 237, 242),
    Muted = Color3.fromRGB(140, 146, 160),
    Accent = Color3.fromRGB(94, 129, 255),
}

local Connections = {}
local function track(conn)
    table.insert(Connections, conn)
    return conn
end

----------------------------------------------------------------------
-- nucleo da hitbox
----------------------------------------------------------------------
local Originals = setmetatable({}, { __mode = "k" })

local function getPart(char)
    local p = Config.Part
    if p == "Torso" then
        return char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
    end
    return char:FindFirstChild(p)
end

local function applyHitbox(part)
    if not Originals[part] then
        Originals[part] = {
            Size = part.Size,
            Transparency = part.Transparency,
            CanCollide = part.CanCollide,
            Massless = part.Massless,
            Color = part.Color,
            Material = part.Material,
        }
    end
    local s = Config.Size
    part.Size = Vector3.new(s, s, s)
    part.Transparency = Config.Transparency
    part.Color = Config.Color
    part.Material = Enum.Material.Neon
    part.CanCollide = false
    part.Massless = true
end

local function restoreHitbox(part)
    local o = Originals[part]
    Originals[part] = nil
    if o and part.Parent then
        pcall(function()
            part.Size = o.Size
            part.Transparency = o.Transparency
            part.CanCollide = o.CanCollide
            part.Massless = o.Massless
            part.Color = o.Color
            part.Material = o.Material
        end)
    end
end

local function restoreAll()
    for part in pairs(Originals) do
        restoreHitbox(part)
    end
end

local function isValidTarget(plr, hum)
    if Config.IgnoreDead and hum.Health <= 0 then
        return false
    end
    if Config.TeamCheck and plr.Team ~= nil and plr.Team == LocalPlayer.Team then
        return false
    end
    return true
end

----------------------------------------------------------------------
-- helpers de ui
----------------------------------------------------------------------
local function create(class, props, parent)
    local inst = Instance.new(class)
    for k, v in pairs(props) do
        inst[k] = v
    end
    if parent then
        inst.Parent = parent
    end
    return inst
end

local function corner(inst, r)
    return create("UICorner", { CornerRadius = UDim.new(0, r) }, inst)
end

local function stroke(inst, color, thickness)
    return create("UIStroke", {
        Color = color or Theme.Stroke,
        Thickness = thickness or 1,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
    }, inst)
end

local function tween(inst, t, props)
    TweenService:Create(inst, TweenInfo.new(t, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props):Play()
end

local function isPointerBegin(input)
    return input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch
end

local function isPointerMove(input)
    return input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
end

----------------------------------------------------------------------
-- gui raiz
----------------------------------------------------------------------
local Gui = create("ScreenGui", {
    Name = "Nexus" .. tostring(math.random(10000, 99999)),
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    IgnoreGuiInset = true,
})

if syn and syn.protect_gui then
    pcall(syn.protect_gui, Gui)
end

local parented = false
if gethui then
    parented = pcall(function()
        Gui.Parent = gethui()
    end)
end
if not parented or not Gui.Parent then
    local ok = okCore and pcall(function()
        Gui.Parent = CoreGui
    end)
    if not ok or not Gui.Parent then
        Gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    end
end

local function Unload()
    Config.Enabled = false
    restoreAll()
    for _, c in ipairs(Connections) do
        pcall(function() c:Disconnect() end)
    end
    table.clear(Connections)
    pcall(function() Gui:Destroy() end)
    genv.NexusHitboxUnload = nil
end
genv.NexusHitboxUnload = Unload

----------------------------------------------------------------------
-- janela principal
----------------------------------------------------------------------
local WIDTH, HEIGHT, HEADER = 540, 360, 40

local Main = create("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    Size = UDim2.fromOffset(WIDTH, HEIGHT),
    BackgroundColor3 = Theme.Bg,
    BorderSizePixel = 0,
    ClipsDescendants = true,
}, Gui)
corner(Main, 10)
stroke(Main)

local Scale = create("UIScale", {}, Main)
local function fit()
    local cam = workspace.CurrentCamera
    if not cam then return end
    local vp = cam.ViewportSize
    Scale.Scale = math.clamp(math.min(vp.X / (WIDTH + 80), vp.Y / (HEIGHT + 60)), 0.6, 1)
end
fit()

local Header = create("Frame", {
    Size = UDim2.new(1, 0, 0, HEADER),
    BackgroundColor3 = Theme.Panel,
    BorderSizePixel = 0,
}, Main)
corner(Header, 10)
create("Frame", {
    Position = UDim2.new(0, 0, 1, -10),
    Size = UDim2.new(1, 0, 0, 10),
    BackgroundColor3 = Theme.Panel,
    BorderSizePixel = 0,
}, Header)

create("TextLabel", {
    Text = 'NEXUS  <font color="#8C92A0">Hitbox · Prime Dash</font>',
    RichText = true,
    Font = Enum.Font.GothamBold,
    TextSize = 14,
    TextColor3 = Theme.Text,
    BackgroundTransparency = 1,
    Position = UDim2.fromOffset(16, 0),
    Size = UDim2.new(0.7, 0, 1, 0),
    TextXAlignment = Enum.TextXAlignment.Left,
}, Header)

create("Frame", {
    Position = UDim2.new(0, 0, 1, -1),
    Size = UDim2.new(1, 0, 0, 1),
    BackgroundColor3 = Theme.Stroke,
    BorderSizePixel = 0,
}, Header)

local function headerButton(text, xOffset)
    local b = create("TextButton", {
        Text = text,
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        TextColor3 = Theme.Muted,
        AutoButtonColor = false,
        BackgroundColor3 = Theme.Element,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, xOffset, 0.5, -14),
        Size = UDim2.fromOffset(28, 28),
        BorderSizePixel = 0,
    }, Header)
    corner(b, 6)
    track(b.MouseEnter:Connect(function()
        tween(b, 0.12, { BackgroundTransparency = 0, TextColor3 = Theme.Text })
    end))
    track(b.MouseLeave:Connect(function()
        tween(b, 0.12, { BackgroundTransparency = 1, TextColor3 = Theme.Muted })
    end))
    return b
end

local CloseBtn = headerButton("X", -38)
local MinBtn = headerButton("-", -70)

local Body = create("Frame", {
    Position = UDim2.fromOffset(0, HEADER),
    Size = UDim2.new(1, 0, 1, -HEADER),
    BackgroundTransparency = 1,
}, Main)

-- arrastar
local dragging, dragStart, startPos = false, nil, nil
track(Header.InputBegan:Connect(function(input)
    if isPointerBegin(input) then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
    end
end))
track(UserInputService.InputChanged:Connect(function(input)
    if dragging and isPointerMove(input) then
        local d = (input.Position - dragStart) / Scale.Scale
        Main.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y
        )
    end
end))
track(UserInputService.InputEnded:Connect(function(input)
    if isPointerBegin(input) then
        dragging = false
    end
end))

local minimized = false
track(MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    Body.Visible = not minimized
    Main.Size = minimized and UDim2.fromOffset(WIDTH, HEADER) or UDim2.fromOffset(WIDTH, HEIGHT)
end))
track(CloseBtn.MouseButton1Click:Connect(Unload))

----------------------------------------------------------------------
-- sidebar (perfil + abas)
----------------------------------------------------------------------
local Sidebar = create("Frame", {
    Size = UDim2.new(0, 150, 1, 0),
    BackgroundColor3 = Theme.Panel,
    BorderSizePixel = 0,
}, Body)
corner(Sidebar, 10)
create("Frame", {
    Size = UDim2.new(1, 0, 0, 10),
    BackgroundColor3 = Theme.Panel,
    BorderSizePixel = 0,
}, Sidebar)
create("Frame", {
    Position = UDim2.new(1, -10, 0, 0),
    Size = UDim2.new(0, 10, 1, 0),
    BackgroundColor3 = Theme.Panel,
    BorderSizePixel = 0,
}, Sidebar)

create("Frame", {
    Position = UDim2.new(1, -1, 0, 0),
    Size = UDim2.new(0, 1, 1, 0),
    BackgroundColor3 = Theme.Stroke,
    BorderSizePixel = 0,
}, Sidebar)

local Avatar = create("ImageLabel", {
    Position = UDim2.new(0.5, -28, 0, 16),
    Size = UDim2.fromOffset(56, 56),
    BackgroundColor3 = Theme.Element,
    BorderSizePixel = 0,
    Image = "",
}, Sidebar)
corner(Avatar, 28)
stroke(Avatar, Theme.Accent, 2)

create("TextLabel", {
    Text = LocalPlayer.DisplayName,
    Font = Enum.Font.GothamBold,
    TextSize = 13,
    TextColor3 = Theme.Text,
    TextTruncate = Enum.TextTruncate.AtEnd,
    BackgroundTransparency = 1,
    Position = UDim2.fromOffset(8, 78),
    Size = UDim2.new(1, -16, 0, 18),
}, Sidebar)

create("TextLabel", {
    Text = "@" .. LocalPlayer.Name,
    Font = Enum.Font.Gotham,
    TextSize = 11,
    TextColor3 = Theme.Muted,
    TextTruncate = Enum.TextTruncate.AtEnd,
    BackgroundTransparency = 1,
    Position = UDim2.fromOffset(8, 96),
    Size = UDim2.new(1, -16, 0, 14),
}, Sidebar)

task.spawn(function()
    local ok, img = pcall(function()
        return Players:GetUserThumbnailAsync(
            LocalPlayer.UserId,
            Enum.ThumbnailType.HeadShot,
            Enum.ThumbnailSize.Size100x100
        )
    end)
    if ok and img and Avatar.Parent then
        Avatar.Image = img
    end
end)

local TabList = create("Frame", {
    Position = UDim2.fromOffset(0, 128),
    Size = UDim2.new(1, 0, 1, -128),
    BackgroundTransparency = 1,
}, Sidebar)
create("UIListLayout", {
    Padding = UDim.new(0, 4),
    SortOrder = Enum.SortOrder.LayoutOrder,
}, TabList)
create("UIPadding", {
    PaddingLeft = UDim.new(0, 10),
    PaddingRight = UDim.new(0, 10),
}, TabList)

local Content = create("Frame", {
    Position = UDim2.fromOffset(150, 0),
    Size = UDim2.new(1, -150, 1, 0),
    BackgroundTransparency = 1,
}, Body)

local Pages, TabButtons, Orders = {}, {}, setmetatable({}, { __mode = "k" })

local function selectTab(name)
    for n, page in pairs(Pages) do
        page.Visible = (n == name)
    end
    for n, b in pairs(TabButtons) do
        local on = (n == name)
        tween(b, 0.15, {
            BackgroundTransparency = on and 0 or 1,
            TextColor3 = on and Theme.Text or Theme.Muted,
        })
    end
end

local tabCount = 0
local function newTab(name)
    tabCount = tabCount + 1
    local page = create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Theme.Stroke,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new(),
        Visible = false,
    }, Content)
    create("UIListLayout", {
        Padding = UDim.new(0, 8),
        SortOrder = Enum.SortOrder.LayoutOrder,
    }, page)
    create("UIPadding", {
        PaddingTop = UDim.new(0, 14),
        PaddingBottom = UDim.new(0, 14),
        PaddingLeft = UDim.new(0, 14),
        PaddingRight = UDim.new(0, 14),
    }, page)

    local btn = create("TextButton", {
        Text = name,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        TextColor3 = Theme.Muted,
        TextXAlignment = Enum.TextXAlignment.Left,
        AutoButtonColor = false,
        BackgroundColor3 = Theme.Element,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 32),
        BorderSizePixel = 0,
        LayoutOrder = tabCount,
    }, TabList)
    corner(btn, 6)
    create("UIPadding", { PaddingLeft = UDim.new(0, 12) }, btn)

    Pages[name] = page
    TabButtons[name] = btn
    track(btn.MouseButton1Click:Connect(function()
        selectTab(name)
    end))
    return page
end

----------------------------------------------------------------------
-- componentes
----------------------------------------------------------------------
local function nextOrder(page)
    local n = (Orders[page] or 0) + 1
    Orders[page] = n
    return n
end

local function newRow(page, height)
    local f = create("Frame", {
        Size = UDim2.new(1, 0, 0, height),
        BackgroundColor3 = Theme.Element,
        BorderSizePixel = 0,
        LayoutOrder = nextOrder(page),
    }, page)
    corner(f, 8)
    stroke(f)
    return f
end

local function rowLabel(parent, text, centered)
    return create("TextLabel", {
        Text = text,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        TextColor3 = Theme.Text,
        BackgroundTransparency = 1,
        Position = centered and UDim2.fromOffset(14, 0) or UDim2.fromOffset(14, 8),
        Size = centered and UDim2.new(1, -70, 1, 0) or UDim2.new(1, -90, 0, 20),
        TextXAlignment = Enum.TextXAlignment.Left,
    }, parent)
end

local function addSection(page, text)
    create("TextLabel", {
        Text = string.upper(text),
        Font = Enum.Font.GothamBold,
        TextSize = 11,
        TextColor3 = Theme.Muted,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 16),
        TextXAlignment = Enum.TextXAlignment.Left,
        LayoutOrder = nextOrder(page),
    }, page)
end

local function addToggle(page, label, default, callback)
    local r = newRow(page, 38)
    rowLabel(r, label, true)

    local pill = create("TextButton", {
        Text = "",
        AutoButtonColor = false,
        Size = UDim2.fromOffset(40, 20),
        Position = UDim2.new(1, -54, 0.5, -10),
        BackgroundColor3 = default and Theme.Accent or Theme.Stroke,
        BorderSizePixel = 0,
    }, r)
    corner(pill, 10)

    local knob = create("Frame", {
        Size = UDim2.fromOffset(14, 14),
        Position = default and UDim2.fromOffset(23, 3) or UDim2.fromOffset(3, 3),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0,
    }, pill)
    corner(knob, 7)

    local state = default
    local function set(v)
        state = v and true or false
        tween(pill, 0.15, { BackgroundColor3 = state and Theme.Accent or Theme.Stroke })
        tween(knob, 0.15, { Position = state and UDim2.fromOffset(23, 3) or UDim2.fromOffset(3, 3) })
        pcall(callback, state)
    end

    track(pill.MouseButton1Click:Connect(function()
        set(not state)
    end))

    return { Set = set, Get = function() return state end }
end

local function addSlider(page, label, min, max, default, step, callback)
    local r = newRow(page, 56)
    rowLabel(r, label)

    local valueLabel = create("TextLabel", {
        Font = Enum.Font.GothamBold,
        TextSize = 12,
        TextColor3 = Theme.Accent,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -74, 0, 8),
        Size = UDim2.fromOffset(60, 20),
        TextXAlignment = Enum.TextXAlignment.Right,
        Text = "",
    }, r)

    local bar = create("Frame", {
        Position = UDim2.new(0, 14, 0, 38),
        Size = UDim2.new(1, -28, 0, 6),
        BackgroundColor3 = Theme.Stroke,
        BorderSizePixel = 0,
    }, r)
    corner(bar, 3)

    local fill = create("Frame", {
        Size = UDim2.new(0, 0, 1, 0),
        BackgroundColor3 = Theme.Accent,
        BorderSizePixel = 0,
    }, bar)
    corner(fill, 3)

    local knob = create("Frame", {
        Size = UDim2.fromOffset(14, 14),
        Position = UDim2.new(0, -7, 0.5, -7),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0,
    }, bar)
    corner(knob, 7)

    local hit = create("TextButton", {
        Text = "",
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 0.5, -12),
        Size = UDim2.new(1, 0, 0, 24),
    }, bar)

    local function setValue(v)
        v = math.clamp(math.floor(v / step + 0.5) * step, min, max)
        local a = (v - min) / (max - min)
        fill.Size = UDim2.new(a, 0, 1, 0)
        knob.Position = UDim2.new(a, -7, 0.5, -7)
        valueLabel.Text = tostring(math.floor(v * 100 + 0.5) / 100)
        pcall(callback, v)
    end

    local function fromInput(input)
        local a = (input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X
        setValue(min + (max - min) * math.clamp(a, 0, 1))
    end

    local sliding = false
    track(hit.InputBegan:Connect(function(input)
        if isPointerBegin(input) then
            sliding = true
            fromInput(input)
        end
    end))
    track(UserInputService.InputChanged:Connect(function(input)
        if sliding and isPointerMove(input) then
            fromInput(input)
        end
    end))
    track(UserInputService.InputEnded:Connect(function(input)
        if isPointerBegin(input) then
            sliding = false
        end
    end))

    setValue(default)
end

local function addSegment(page, label, options, defaultIndex, callback)
    local r = newRow(page, 70)
    rowLabel(r, label)

    local holder = create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(14, 34),
        Size = UDim2.new(1, -28, 0, 26),
    }, r)
    create("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        Padding = UDim.new(0, 6),
    }, holder)

    local buttons = {}
    local function pick(i)
        for idx, b in ipairs(buttons) do
            local on = (idx == i)
            tween(b, 0.15, {
                BackgroundColor3 = on and Theme.Accent or Theme.Bg,
                TextColor3 = on and Color3.new(1, 1, 1) or Theme.Muted,
            })
        end
        pcall(callback, options[i][2])
    end

    local n = #options
    for i, opt in ipairs(options) do
        local b = create("TextButton", {
            Text = opt[1],
            Font = Enum.Font.GothamMedium,
            TextSize = 12,
            TextColor3 = Theme.Muted,
            AutoButtonColor = false,
            BackgroundColor3 = Theme.Bg,
            BorderSizePixel = 0,
            Size = UDim2.new(1 / n, -(6 * (n - 1)) / n, 1, 0),
        }, holder)
        corner(b, 6)
        buttons[i] = b
        track(b.MouseButton1Click:Connect(function()
            pick(i)
        end))
    end
    pick(defaultIndex)
end

local function addColors(page, label, colors, defaultIndex, callback)
    local r = newRow(page, 66)
    rowLabel(r, label)

    local holder = create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(14, 32),
        Size = UDim2.new(1, -28, 0, 26),
    }, r)
    create("UIListLayout", {
        FillDirection = Enum.FillDirection.Horizontal,
        Padding = UDim.new(0, 10),
        VerticalAlignment = Enum.VerticalAlignment.Center,
    }, holder)

    local ring = {}
    local function pick(i)
        for idx, st in ipairs(ring) do
            st.Transparency = (idx == i) and 0 or 1
        end
        pcall(callback, colors[i])
    end

    for i, c in ipairs(colors) do
        local b = create("TextButton", {
            Text = "",
            AutoButtonColor = false,
            Size = UDim2.fromOffset(24, 24),
            BackgroundColor3 = c,
            BorderSizePixel = 0,
        }, holder)
        corner(b, 12)
        ring[i] = create("UIStroke", {
            Color = Color3.new(1, 1, 1),
            Thickness = 2,
            Transparency = 1,
        }, b)
        track(b.MouseButton1Click:Connect(function()
            pick(i)
        end))
    end
    pick(defaultIndex)
end

local function addButton(page, text, callback)
    local b = create("TextButton", {
        Text = text,
        Font = Enum.Font.GothamBold,
        TextSize = 13,
        TextColor3 = Theme.Text,
        AutoButtonColor = false,
        BackgroundColor3 = Theme.Element,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 36),
        LayoutOrder = nextOrder(page),
    }, page)
    corner(b, 8)
    stroke(b)
    track(b.MouseEnter:Connect(function()
        tween(b, 0.12, { BackgroundColor3 = Theme.Accent })
    end))
    track(b.MouseLeave:Connect(function()
        tween(b, 0.12, { BackgroundColor3 = Theme.Element })
    end))
    track(b.MouseButton1Click:Connect(function()
        pcall(callback)
    end))
    return b
end

local function addNote(page, text)
    create("TextLabel", {
        Text = text,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        TextColor3 = Theme.Muted,
        TextWrapped = true,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 34),
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        LayoutOrder = nextOrder(page),
    }, page)
end

----------------------------------------------------------------------
-- paginas
----------------------------------------------------------------------
local HitboxPage = newTab("Hitbox")
local SettingsPage = newTab("Ajustes")

-- hitbox
addSection(HitboxPage, "Status")
local statusRow = newRow(HitboxPage, 38)
rowLabel(statusRow, "Alvos ativos", true)
local StatusLabel = create("TextLabel", {
    Text = "0",
    Font = Enum.Font.GothamBold,
    TextSize = 13,
    TextColor3 = Theme.Accent,
    BackgroundTransparency = 1,
    Position = UDim2.new(1, -74, 0, 0),
    Size = UDim2.fromOffset(60, 38),
    TextXAlignment = Enum.TextXAlignment.Right,
}, statusRow)

addSection(HitboxPage, "Hitbox")
local HitboxToggle = addToggle(HitboxPage, "Ativar hitbox  [H]", Config.Enabled, function(v)
    Config.Enabled = v
    if not v then
        restoreAll()
    end
end)

addSlider(HitboxPage, "Tamanho", 2, 40, Config.Size, 1, function(v)
    Config.Size = v
end)

addSlider(HitboxPage, "Transparência", 0, 1, Config.Transparency, 0.05, function(v)
    Config.Transparency = v
end)

addSegment(HitboxPage, "Parte alvo", {
    { "Root", "HumanoidRootPart" },
    { "Cabeça", "Head" },
    { "Torso", "Torso" },
}, 1, function(v)
    if Config.Part ~= v then
        restoreAll()
        Config.Part = v
    end
end)

addColors(HitboxPage, "Cor", {
    Color3.fromRGB(255, 80, 80),
    Color3.fromRGB(255, 170, 60),
    Color3.fromRGB(90, 220, 120),
    Color3.fromRGB(80, 170, 255),
    Color3.fromRGB(170, 110, 255),
    Color3.fromRGB(240, 240, 245),
}, 1, function(c)
    Config.Color = c
end)

-- ajustes
addSection(SettingsPage, "Filtros")
addToggle(SettingsPage, "Ignorar mesmo time", Config.TeamCheck, function(v)
    Config.TeamCheck = v
end)
addToggle(SettingsPage, "Ignorar jogadores mortos", Config.IgnoreDead, function(v)
    Config.IgnoreDead = v
end)

addSection(SettingsPage, "Atalhos")
addNote(SettingsPage, "H liga/desliga a hitbox. RightShift esconde ou mostra o menu.")

addSection(SettingsPage, "Script")
addButton(SettingsPage, "Descarregar script", Unload)

selectTab("Hitbox")

----------------------------------------------------------------------
-- loop principal
----------------------------------------------------------------------
local acc = 0
track(RunService.Heartbeat:Connect(function(dt)
    if Config.Enabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then
                local char = plr.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                local part = char and getPart(char)
                if hum and part and part:IsA("BasePart") then
                    if isValidTarget(plr, hum) then
                        applyHitbox(part)
                    elseif Originals[part] then
                        restoreHitbox(part)
                    end
                end
            end
        end
    end

    acc = acc + dt
    if acc >= 0.25 then
        acc = 0
        local n = 0
        for part in pairs(Originals) do
            if part.Parent then
                n = n + 1
            else
                Originals[part] = nil
            end
        end
        StatusLabel.Text = tostring(n)
    end
end))

track(UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        Main.Visible = not Main.Visible
    elseif input.KeyCode == Enum.KeyCode.H then
        HitboxToggle.Set(not HitboxToggle.Get())
    end
end))

track(Players.PlayerRemoving:Connect(function(plr)
    local char = plr.Character
    if char then
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("BasePart") and Originals[d] then
                Originals[d] = nil
            end
        end
    end
end))

local cam = workspace.CurrentCamera
if cam then
    track(cam:GetPropertyChangedSignal("ViewportSize"):Connect(fit))
end
