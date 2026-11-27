-- XIT DA FACÇÃO v4.0 | delta | vermelho & preto | TFJ
local cloneref = cloneref or function(o) return o end
local function svc(n) return cloneref(game:GetService(n)) end
local Players, RunService, UIS, TweenService = svc("Players"), svc("RunService"), svc("UserInputService"), svc("TweenService")
local CoreGui, Workspace, VirtualUser = svc("CoreGui"), svc("Workspace"), svc("VirtualUser")
local LP = Players.LocalPlayer
local env = (getgenv and getgenv()) or _G
if env.XitCleanup then pcall(env.XitCleanup) end

local function getCam() return Workspace.CurrentCamera end
local conns = {}
local function track(c) conns[#conns + 1] = c return c end

local pcache = setmetatable({}, {__mode = "k"})
local function P(o, k, v)
	local c = pcache[o]
	if not c then c = {}; pcache[o] = c end
	if c[k] ~= v then c[k] = v; o[k] = v end
end

---------------------------------------------------------------- CONFIG
local KEY = "TFJ-2026-KEY"

local COLORS = {
	{"Vermelho", Color3.fromRGB(255, 35, 55)},
	{"Branco", Color3.fromRGB(255, 255, 255)},
	{"Rosa", Color3.fromRGB(255, 90, 140)},
	{"Laranja", Color3.fromRGB(255, 130, 0)},
	{"Ciano", Color3.fromRGB(0, 220, 255)},
	{"Verde", Color3.fromRGB(60, 255, 100)},
}
local COLOR_NAMES = {}
for i, c in ipairs(COLORS) do COLOR_NAMES[i] = c[1] end
local function col(i) return COLORS[i][2] end
local WHITE, BLACK = Color3.new(1, 1, 1), Color3.new(0, 0, 0)
local RED = Color3.fromRGB(255, 35, 55)

local S = {
	aimOn = false, aimMode = 1, aimPart = 1, aimSmooth = 40, aimFov = 150, aimShowFov = true, aimFovColor = 1,
	aimDist = 1000, aimTeam = true, aimWall = true, aimPredOn = false, aimPred = 0.12,
	hbOn = false, hbW = 8, hbH = 8, hbTrans = 0.7, hbColor = 1, hbTeam = true,
	hhOn = false, hhSize = 5, hhVisual = true, hhTrans = 0.5, hhColor = 1, hhHide = true, hhTeam = true,
	espOn = false, espFps = 90, espMaxDist = 2000, espTeam = true, espTeamColor = false, espColor = 1,
	espBox = true, espBoxStyle = 1, espFill = false, espFillOp = 0.15,
	espName = true, espDist = true, espHealth = true, espWeapon = false, espTextSize = 13,
	espTracer = false, espTracerFrom = 1, espSkeleton = false, espHead = false,
	espChams = false, espChamsFill = 0.5, espArrow = false, espArrowR = 160,
	flyOn = false, flySpeed = 60, speedOn = false, speedMode = 1, speed = 40,
	jumpOn = false, jump = 90, infJump = false, noclip = false,
	spinOn = false, spinSpeed = 720, gravOn = false, gravity = 100,
	fovOn = false, camFov = 90, afk = true, tpName = "—",
	dmgOn = false, dmgMode = 2, dmgWant = 40, dmgBase = 10, dmgReach = 15, dmgTeam = true,
	teamPick = "—", teamLocal = false, plPick = "—",
	fullbright = false, noFog = false, clockOn = false, clockT = 14,
	-- combate v4
	aimTarget = 1, aimSticky = false, aimYOff = 0, aimLine = false, ignFriends = false,
	trigOn = false, trigDelay = 0.12, clickOn = false, clickRate = 0.1, reachOn = false, reachSize = 12,
	-- troll v4
	invisOn = false, invisAlpha = 1, invisName = true, nofall = false, bhop = false, zoomInf = false,
	clickTp = false, orbitOn = false, orbitSpeed = 2, orbitDist = 8, followOn = false, followDist = 5,
	freecam = false, fcSpeed = 60, thirdP = false,
	-- otimizacao
	opLow = false, opQuality = 1, opNoShadow = false, opNoFx = false, opNoPart = false, opNoLight = false,
	opNoTex = false, opPlastic = false, opNoSound = false, opWater = false, opNoGrass = false,
	opHideAcc = false, opFpsCap = false, opCap = 60, opNo3D = false,
	uiAnim = true, uiFps = true, uiScaleMul = 1,
}

---------------------------------------------------------------- HELPERS
local friends = {}
local function isEnemy(p, check)
	if S.ignFriends and friends[p] then return false end
	if not check then return true end
	if not LP.Team or not p.Team then return true end
	return p.Team ~= LP.Team
end
local function new(cls, props, parent)
	local i = Instance.new(cls)
	for k, v in pairs(props) do i[k] = v end
	if parent then i.Parent = parent end
	return i
end
local function corner(i, r) return new("UICorner", {CornerRadius = UDim.new(0, r or 6)}, i) end
local function stroke(i, c, t, tr)
	return new("UIStroke", {Color = c, Thickness = t or 1, Transparency = tr or 0, ApplyStrokeMode = Enum.ApplyStrokeMode.Border}, i)
end
local function tw(i, props, t)
	if not S.uiAnim then
		for k, v in pairs(props) do i[k] = v end
		return
	end
	TweenService:Create(i, TweenInfo.new(t or 0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props):Play()
end

local gui = new("ScreenGui", {
	Name = "XitDaFaccao", ResetOnSpawn = false, IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling, DisplayOrder = 999,
})
local okp = pcall(function() gui.Parent = (gethui and gethui()) or CoreGui end)
if not okp or not gui.Parent then gui.Parent = LP:WaitForChild("PlayerGui") end

local holder = Instance.new("Folder")
holder.Name = "XitHolder"
if not pcall(function() holder.Parent = CoreGui end) or not holder.Parent then holder.Parent = getCam() end

---------------------------------------------------------------- LOGO TFJ (PNG sem fundo, embutido)
local LOGO_B64 = table.concat({
"iVBORw0KGgoAAAANSUhEUgAAAIAAAACACAMAAAD04JH5AAABgFBMVEXio1VdXiLZGxsgTiJwZV2mmVWpnJygVxrgVizUqaijYFilnS7UbFieGx3uzhPDDBHXpDAUJyWOAABijSicBwrjzVrMAADf09IvLEg6DwCgCQodgzNpJyN2AAA/QkV/hIW9xie8wMHIEhIAAAAE",
"AQHLFBfFBAbDjEm0FRe1BAduBwhPCAkwBwiMAwTMlFGRFRe4hkyPZjlOOCZuEhN1VjQwJhYSFQtpSS+PaERTFBOoeUgrGBHMJRpVRSmxAAD/AACIWzJNNRl5AAC0HCHGGyHLSCmZc0msdjbMZzMyNhhJKRTPVy+wWCuxaDFzZi60JhTLOSXMJSSSBAaxCQ2TdjOyCg6t",
"NxixSCbNdjWRJxCvIyU1KSJSVilwNxdQRRdyJhCQRycUJhF0dDK5DBGPOBc4NSdERkhpRhevhDPJNhu0OSgFdzNUAAByAAH7+/uUBAfKhjt0WEWuRhoyAABqNyaThTlGLSLTo1ACajE7RiVQAACTOiizk1DNeUORRBiqBglJ4D1VAAAAgHRSTlP//+//////////////",
"8/+W//8f/53/Bf//G1j//0T/////fgD+/v7//v7+/v7+//7////+///////+/////wMB//8H//7/////////////////Tov/r//////////////////O//////////8OL/9u////Cf///////yj/////dmfNINAAAB9DSURBVHjarXyHY9tGsj7ULFlyi3255N67935F",
"CxCLDhCFJACbFMVe1SWry1JUzkWWe1zyr7+ZXZAiZTlx8g6xaUokMN9O+WZ2dzbC7J+55v6e/Pts6e7dFXl+XsRrYf7J3btLi/efss/+/vDRn3mk8CekP2Svi7+tMLGPi7qSXLpcZL/62/9/wUA8+hMYvhfAIyb92RIIn5eVtEq+urSsYoJGbt9dfAXffDj3bwUwhyO6",
"v3RbFM20Rn7n0rImaGKJY/i3AXgE4ud+ewLS+wPX0opiynhR9qor6ezlZzJgeIa4/y0AcPTPli6la2m9CBLEUVES+9covshmD4UGerj7H9+lBeE7Rr+4Is4nms+aMpcoowuq2ew/s3CBH8q93/dslJXF0e+B8AcA4Pb7K6KcTaQnMpTstfZXFY5C5hhUDuHRo78OAMQ/",
"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"
})

local function b64decode(data)
	local lut = {}
	local alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
	for i = 1, #alphabet do lut[alphabet:byte(i)] = i - 1 end
	local out, buf, bits = {}, 0, 0
	local chunk = {}
	for i = 1, #data do
		local v = lut[data:byte(i)]
		if v then
			buf = buf * 64 + v
			bits = bits + 6
			if bits >= 8 then
				bits = bits - 8
				local div = 2 ^ bits
				local byte = math.floor(buf / div)
				buf = buf - byte * div
				chunk[#chunk + 1] = string.char(byte)
				if #chunk >= 4096 then out[#out + 1] = table.concat(chunk); chunk = {} end
			end
		end
	end
	out[#out + 1] = table.concat(chunk)
	return table.concat(out)
end

local LOGO
do
	local ok = pcall(function()
		if not (writefile and getcustomasset) then return end
		local fname = "tfj_logo_v1.png"
		if not (isfile and isfile(fname)) then writefile(fname, b64decode(LOGO_B64)) end
		LOGO = getcustomasset(fname)
	end)
	if not ok then LOGO = nil end
end

local function logoImage(cls, props, parent)
	if LOGO then
		props.Image = LOGO
		props.BackgroundTransparency = 1
		props.ScaleType = Enum.ScaleType.Fit
		return new(cls, props, parent), true
	end
	return nil, false
end

---------------------------------------------------------------- SISTEMA DE KEY
do
	local passed, closed = false, false
	local KC = {
		bg = Color3.fromRGB(8, 8, 9), panel = Color3.fromRGB(13, 13, 15), row = Color3.fromRGB(26, 26, 29),
		red = Color3.fromRGB(235, 25, 45), dark = Color3.fromRGB(110, 8, 18), dim = Color3.fromRGB(125, 125, 130),
	}
	local kscale = math.clamp(math.min(getCam().ViewportSize.X / 520, getCam().ViewportSize.Y / 360), 0.6, 1.1)
	local kw = new("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
		Size = UDim2.fromOffset(340, 290), BackgroundColor3 = KC.bg, BorderSizePixel = 0, Active = true,
	}, gui)
	corner(kw, 12)
	stroke(kw, KC.red, 1.5, 0.15)
	new("UIScale", {Scale = kscale}, kw)

	local lg = logoImage("ImageLabel", {
		Size = UDim2.fromOffset(96, 96), AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 14),
	}, kw)
	if not lg then
		local fb = new("TextLabel", {
			Text = "TFJ", Size = UDim2.fromOffset(80, 80), AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 20),
			BackgroundColor3 = KC.red, TextColor3 = BLACK, Font = Enum.Font.GothamBlack, TextSize = 26,
		}, kw)
		corner(fb, 40)
	end
	new("TextLabel", {
		Text = "XIT DA FACÇÃO", Size = UDim2.new(1, 0, 0, 20), Position = UDim2.fromOffset(0, 116), BackgroundTransparency = 1,
		TextColor3 = WHITE, Font = Enum.Font.GothamBlack, TextSize = 17,
	}, kw)
	new("TextLabel", {
		Text = "TORCIDA FACÇÃO JOVEM  •  DIGITE A KEY", Size = UDim2.new(1, 0, 0, 14), Position = UDim2.fromOffset(0, 136),
		BackgroundTransparency = 1, TextColor3 = KC.red, Font = Enum.Font.GothamBold, TextSize = 10,
	}, kw)

	local box = new("TextBox", {
		Text = "", PlaceholderText = "TFJ-XXXX-XXX", PlaceholderColor3 = KC.dim, ClearTextOnFocus = false,
		Size = UDim2.new(1, -40, 0, 36), Position = UDim2.fromOffset(20, 162), BackgroundColor3 = KC.row,
		TextColor3 = WHITE, Font = Enum.Font.GothamMedium, TextSize = 14,
	}, kw)
	corner(box, 8)
	local boxStroke = stroke(box, KC.dark, 1)

	local msg = new("TextLabel", {
		Text = "", Size = UDim2.new(1, -40, 0, 14), Position = UDim2.fromOffset(20, 203), BackgroundTransparency = 1,
		TextColor3 = KC.red, Font = Enum.Font.GothamBold, TextSize = 11,
	}, kw)

	local ok = new("TextButton", {
		Text = "ENTRAR", Size = UDim2.new(1, -40, 0, 36), Position = UDim2.fromOffset(20, 224), BackgroundColor3 = KC.red,
		TextColor3 = WHITE, Font = Enum.Font.GothamBlack, TextSize = 14, AutoButtonColor = false,
	}, kw)
	corner(ok, 8)
	new("UIGradient", {Rotation = 90, Color = ColorSequence.new(RED, KC.dark)}, ok)

	local x = new("TextButton", {
		Text = "X", Size = UDim2.fromOffset(26, 24), Position = UDim2.new(1, -34, 0, 8), BackgroundColor3 = KC.dark,
		TextColor3 = WHITE, Font = Enum.Font.GothamBold, TextSize = 13, AutoButtonColor = false,
	}, kw)
	corner(x, 6)
	new("TextLabel", {
		Text = "BONDE BATE E RODA", Size = UDim2.new(1, 0, 0, 14), Position = UDim2.new(0, 0, 1, -17), BackgroundTransparency = 1,
		TextColor3 = KC.dim, Font = Enum.Font.Gotham, TextSize = 10,
	}, kw)

	local busy = false
	local function try()
		if busy then return end
		local typed = box.Text:gsub("^%s+", ""):gsub("%s+$", "")
		if typed == KEY then
			busy = true
			passed = true
			msg.TextColor3 = Color3.fromRGB(60, 255, 100)
			msg.Text = "KEY VÁLIDA"
			task.wait(0.35)
			kw:Destroy()
		else
			msg.TextColor3 = KC.red
			msg.Text = "KEY INVÁLIDA"
			boxStroke.Color = KC.red
			local base = box.Position
			for _ = 1, 3 do
				box.Position = base + UDim2.fromOffset(6, 0); task.wait(0.04)
				box.Position = base - UDim2.fromOffset(6, 0); task.wait(0.04)
			end
			box.Position = base
		end
	end
	ok.Activated:Connect(try)
	box.FocusLost:Connect(function(enter) if enter then try() end end)
	x.Activated:Connect(function() closed = true end)

	while not passed and not closed do task.wait(0.1) end
	if not passed then
		pcall(function() kw:Destroy() end)
		pcall(function() holder:Destroy() end)
		pcall(function() gui:Destroy() end)
		return
	end
end

---------------------------------------------------------------- CACHE DE JOGADORES
local plist, info = {}, {}
local espObjs, highlights = {}, {}

local function build(p)
	local c = p.Character
	if not c then info[p] = nil return end
	task.spawn(function()
		local hum = c:WaitForChild("Humanoid", 6)
		local root = c:WaitForChild("HumanoidRootPart", 6)
		local head = c:WaitForChild("Head", 6)
		if not hum or not root or p.Character ~= c then return end
		info[p] = {
			c = c, hum = hum, root = root, head = head,
			torso = c:FindFirstChild("UpperTorso") or c:FindFirstChild("Torso"),
		}
	end)
end
local function live(p)
	local i = info[p]
	if i and i.c.Parent and i.hum.Health > 0 then return i end
end

local function dropESP(p)
	local o = espObjs[p]
	if o then
		for _, d in ipairs(o.list) do pcall(function() d:Remove() end) end
		espObjs[p] = nil
	end
	if highlights[p] then highlights[p]:Destroy(); highlights[p] = nil end
end

local function addP(p)
	if p == LP or table.find(plist, p) then return end
	plist[#plist + 1] = p
	task.spawn(function()
		local ok, r = pcall(LP.IsFriendsWith, LP, p.UserId)
		if ok and r then friends[p] = true end
	end)
	track(p.CharacterAdded:Connect(function() info[p] = nil; build(p) end))
	track(p.CharacterRemoving:Connect(function() info[p] = nil end))
	build(p)
end
local function remP(p)
	local idx = table.find(plist, p)
	if idx then table.remove(plist, idx) end
	info[p] = nil
	friends[p] = nil
	dropESP(p)
end
track(Players.PlayerAdded:Connect(addP))
track(Players.PlayerRemoving:Connect(remP))
for _, p in ipairs(Players:GetPlayers()) do addP(p) end

---------------------------------------------------------------- HITBOX / HITHEAD
local saved = setmetatable({}, {__mode = "k"})
local adorns = {}
local dirty = false

local function expand(part, size, trans, color)
	if not saved[part] then
		saved[part] = {
			Size = part.Size, Transparency = part.Transparency, CanCollide = part.CanCollide,
			Massless = part.Massless, Color = part.Color, Material = part.Material,
		}
	end
	if part.Size ~= size then part.Size = size end
	if part.CanCollide then part.CanCollide = false end
	if not part.Massless then part.Massless = true end
	if trans and part.Transparency ~= trans then part.Transparency = trans end
	if color then
		if part.Color ~= color then part.Color = color end
		if part.Material ~= Enum.Material.Neon then part.Material = Enum.Material.Neon end
	end
	dirty = true
end
local function restore(part)
	local s = saved[part]
	if not s then return end
	pcall(function()
		part.Size = s.Size; part.Transparency = s.Transparency; part.CanCollide = s.CanCollide
		part.Massless = s.Massless; part.Color = s.Color; part.Material = s.Material
	end)
	saved[part] = nil
end
local function setSphere(head, on, radius, trans, color)
	local a = adorns[head]
	if on then
		if not a or not a.Parent then
			a = Instance.new("SphereHandleAdornment")
			a.AlwaysOnTop = true; a.ZIndex = 1; a.Adornee = head; a.Parent = holder
			adorns[head] = a
		end
		P(a, "Radius", radius); P(a, "Transparency", trans); P(a, "Color3", color)
	elseif a then
		a:Destroy(); adorns[head] = nil
	end
end
local function hitStep()
	local anyOn = S.hbOn or S.hhOn
	if not anyOn and not dirty then return end
	dirty = anyOn
	for _, p in ipairs(plist) do
		local i = info[p]
		if i then
			local alive = i.c.Parent and i.hum.Health > 0
			if S.hbOn and alive and isEnemy(p, S.hbTeam) then
				expand(i.root, Vector3.new(S.hbW, S.hbH, S.hbW), S.hbTrans, col(S.hbColor))
			else
				restore(i.root)
			end
			local head = i.head
			if head then
				if S.hhOn and alive and isEnemy(p, S.hhTeam) then
					local sz = S.hhSize
					expand(head, Vector3.new(sz, sz, sz), S.hhHide and 1 or nil, nil)
					setSphere(head, S.hhVisual, sz / 2, S.hhTrans, col(S.hhColor))
				else
					restore(head)
					setSphere(head, false)
				end
			end
		end
	end
	for head, a in pairs(adorns) do
		if not head.Parent then a:Destroy(); adorns[head] = nil end
	end
end

---------------------------------------------------------------- AIMBOT
local HAS_DRAW = false
pcall(function() local t = Drawing.new("Line"); t:Remove(); HAS_DRAW = true end)

local fovC = new("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false}, gui)
new("UICorner", {CornerRadius = UDim.new(1, 0)}, fovC)
local fovS = stroke(fovC, RED, 1.5)
local rp = RaycastParams.new()
rp.FilterType = Enum.RaycastFilterType.Exclude
local lastFilter = 0
local lockedPart, aimLineD

local function screenCenter(cam)
	if UIS.TouchEnabled and not UIS.MouseEnabled then
		local vp = cam.ViewportSize
		return Vector2.new(vp.X / 2, vp.Y / 2)
	end
	return UIS:GetMouseLocation()
end

local function hasTool()
	local ch = LP.Character
	return ch and ch:FindFirstChildOfClass("Tool")
end

local function aimStep(cam, now)
	local showFov = S.aimOn and S.aimShowFov
	P(fovC, "Visible", showFov)
	if aimLineD and not (S.aimOn and S.aimLine) then P(aimLineD, "Visible", false) end
	if not S.aimOn then lockedPart = nil return end
	local center = screenCenter(cam)
	if showFov then
		P(fovC, "Size", UDim2.fromOffset(S.aimFov * 2, S.aimFov * 2))
		P(fovC, "Position", UDim2.fromOffset(center.X, center.Y))
		P(fovS, "Color", col(S.aimFovColor))
	end
	if S.aimMode == 2 and not UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then lockedPart = nil; if aimLineD then P(aimLineD, "Visible", false) end return end
	if S.aimMode == 3 and not hasTool() then lockedPart = nil; if aimLineD then P(aimLineD, "Visible", false) end return end

	if S.aimWall and now - lastFilter > 0.5 then
		lastFilter = now
		local f = {cam}
		if LP.Character then f[#f + 1] = LP.Character end
		for _, i in pairs(info) do f[#f + 1] = i.c end
		rp.FilterDescendantsInstances = f
	end

	local origin = cam.CFrame.Position
	local best, bestScore = nil, math.huge
	local ap = S.aimPart
	local fov, sticky, tmode = S.aimFov, S.aimSticky, S.aimTarget
	for _, p in ipairs(plist) do
		if isEnemy(p, S.aimTeam) then
			local i = live(p)
			if i then
				local part = (ap == 1 and i.head) or (ap == 2 and i.torso) or i.root
				if part then
					local pos = part.Position
					local d3 = (pos - origin).Magnitude
					if d3 <= S.aimDist then
						local v, on = cam:WorldToViewportPoint(pos)
						if on then
							local d = (Vector2.new(v.X, v.Y) - center).Magnitude
							local isLock = sticky and part == lockedPart
							if d < (isLock and fov * 1.6 or fov) then
								local score = d
								if tmode == 2 then score = d3 elseif tmode == 3 then score = i.hum.Health end
								if isLock then score = -1 end
								if score < bestScore and (not S.aimWall or Workspace:Raycast(origin, pos - origin, rp) == nil) then
									bestScore = score; best = part
								end
							end
						end
					end
				end
			end
		end
	end
	lockedPart = sticky and best or nil
	if best then
		local pos = best.Position + Vector3.new(0, S.aimYOff, 0)
		if S.aimPredOn then pos = pos + best.AssemblyLinearVelocity * S.aimPred end
		cam.CFrame = cam.CFrame:Lerp(CFrame.new(origin, pos), math.clamp(S.aimSmooth / 100, 0.01, 1))
		if S.aimLine and HAS_DRAW then
			if not aimLineD then aimLineD = Drawing.new("Line"); aimLineD.Thickness = 1.5; aimLineD.Visible = false end
			local sp = cam:WorldToViewportPoint(pos)
			P(aimLineD, "From", center); P(aimLineD, "To", Vector2.new(sp.X, sp.Y))
			P(aimLineD, "Color", col(S.aimFovColor)); P(aimLineD, "Visible", sp.Z > 0)
		end
	elseif aimLineD then
		P(aimLineD, "Visible", false)
	end
end

---------------------------------------------------------------- TRIGGER / AUTOCLIQUE / REACH
local lastTrig, lastClick = 0, 0
local rpTrig = RaycastParams.new()
rpTrig.FilterType = Enum.RaycastFilterType.Exclude

local function trigStep(cam, now)
	if not S.trigOn or now - lastTrig < S.trigDelay then return end
	local ch = LP.Character
	local tool = ch and ch:FindFirstChildOfClass("Tool")
	if not tool then return end
	local c = screenCenter(cam)
	local ray = cam:ViewportPointToRay(c.X, c.Y)
	rpTrig.FilterDescendantsInstances = {ch}
	local res = Workspace:Raycast(ray.Origin, ray.Direction * S.aimDist, rpTrig)
	if not res then return end
	local m = res.Instance:FindFirstAncestorOfClass("Model")
	local p = m and Players:GetPlayerFromCharacter(m)
	if p and isEnemy(p, S.aimTeam) and live(p) then
		lastTrig = now
		pcall(tool.Activate, tool)
	end
end

local function clickStep(now)
	if not S.clickOn or now - lastClick < S.clickRate then return end
	local tool = hasTool()
	if tool then
		lastClick = now
		pcall(tool.Activate, tool)
	end
end

local reachH
local function reachStep()
	local tool = hasTool()
	local h = tool and tool:FindFirstChild("Handle")
	if reachH and reachH ~= h then restore(reachH); reachH = nil end
	if S.reachOn and h and h:IsA("BasePart") then
		local s = S.reachSize
		expand(h, Vector3.new(s, s, s), 1, nil)
		reachH = h
	elseif reachH then
		restore(reachH); reachH = nil
	end
end

---------------------------------------------------------------- ESP (objetos só quando usados)

local function obj(o, key, cls, props)
	local d = o[key]
	if not d then
		d = Drawing.new(cls)
		for k, v in pairs(props) do d[k] = v end
		d.Visible = false
		o[key] = d
		o.list[#o.list + 1] = d
	end
	return d
end
local function off1(o, k)
	local d = o[k]
	if d then P(d, "Visible", false) end
end
local function hideAll(o)
	if o.shown then
		o.shown = false
		for _, d in ipairs(o.list) do P(d, "Visible", false) end
	end
end
local function txt(o, key, text, size, color, pos)
	local d = obj(o, key, "Text", {Center = true, Outline = true, Font = 2})
	P(d, "Text", text); P(d, "Size", size); P(d, "Color", color); P(d, "Position", pos); P(d, "Visible", true)
end

local R15 = {
	{"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
	{"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
	{"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
	{"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
	{"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"},
}
local R6 = {
	{"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
	{"Torso", "Left Leg"}, {"Torso", "Right Leg"},
}

local function updateChams(p, i, on, color)
	local hl = highlights[p]
	if on and S.espChams then
		if not hl or not hl.Parent then
			hl = Instance.new("Highlight")
			hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
			hl.OutlineColor = WHITE
			hl.Parent = holder
			highlights[p] = hl
		end
		P(hl, "Adornee", i.c); P(hl, "FillColor", color); P(hl, "FillTransparency", S.espChamsFill)
	elseif hl then
		hl:Destroy(); highlights[p] = nil
	end
end

local function drawPlayer(o, p, i, cam, vp, center, color, dist)
	local root, hum, c = i.root, i.hum, i.c
	local rpos = root.Position
	local rv, on = cam:WorldToViewportPoint(rpos)
	if not on then
		hideAll(o)
		if S.espArrow then
			local pos = Vector2.new(rv.X, rv.Y)
			if rv.Z < 0 then pos = center * 2 - pos end
			local dir = pos - center
			if dir.Magnitude > 1 then
				dir = dir.Unit
				local base = center + dir * S.espArrowR
				local perp = Vector2.new(-dir.Y, dir.X)
				local a = obj(o, "arrow", "Triangle", {Filled = true})
				P(a, "PointA", base + dir * 14); P(a, "PointB", base + perp * 9); P(a, "PointC", base - perp * 9)
				P(a, "Color", color); P(a, "Visible", true)
				o.shown = true
			end
		end
		return
	end
	off1(o, "arrow")
	o.shown = true

	local top = cam:WorldToViewportPoint(rpos + Vector3.new(0, 2.7, 0))
	local bot = cam:WorldToViewportPoint(rpos - Vector3.new(0, 3.1, 0))
	local h = math.abs(top.Y - bot.Y)
	local w = h * 0.55
	local x, y = rv.X - w / 2, math.min(top.Y, bot.Y)
	local pos, size = Vector2.new(x, y), Vector2.new(w, h)

	if S.espFill then
		local d = obj(o, "fill", "Square", {Filled = true})
		P(d, "Position", pos); P(d, "Size", size); P(d, "Color", color); P(d, "Transparency", S.espFillOp); P(d, "Visible", true)
	else off1(o, "fill") end

	if S.espBox and S.espBoxStyle == 1 then
		local bo = obj(o, "boxOut", "Square", {Filled = false, Thickness = 3, Color = BLACK})
		P(bo, "Position", pos); P(bo, "Size", size); P(bo, "Visible", true)
		local b = obj(o, "box", "Square", {Filled = false, Thickness = 1})
		P(b, "Position", pos); P(b, "Size", size); P(b, "Color", color); P(b, "Visible", true)
	else off1(o, "boxOut"); off1(o, "box") end

	if S.espBox and S.espBoxStyle == 2 then
		local len = math.min(w, h) * 0.3
		local x2, y2 = x + w, y + h
		local pts = {
			x, y, x + len, y,   x, y, x, y + len,
			x2, y, x2 - len, y,   x2, y, x2, y + len,
			x, y2, x + len, y2,   x, y2, x, y2 - len,
			x2, y2, x2 - len, y2,   x2, y2, x2, y2 - len,
		}
		for k = 1, 8 do
			local l = obj(o, "c" .. k, "Line", {Thickness = 2})
			local b = (k - 1) * 4
			P(l, "From", Vector2.new(pts[b + 1], pts[b + 2])); P(l, "To", Vector2.new(pts[b + 3], pts[b + 4]))
			P(l, "Color", color); P(l, "Visible", true)
		end
		o.cornMade = true
	elseif o.cornMade then
		for k = 1, 8 do off1(o, "c" .. k) end
		o.cornMade = false
	end

	local ts = S.espTextSize
	if S.espName then txt(o, "name", p.DisplayName, ts, color, Vector2.new(rv.X, y - ts - 3)) else off1(o, "name") end
	local yy = y + h + 2
	if S.espDist then
		local ds = math.floor(dist)
		if o.ld ~= ds then o.ld = ds; o.dtxt = "[" .. ds .. "m]" end
		txt(o, "dist", o.dtxt, ts, WHITE, Vector2.new(rv.X, yy))
		yy = yy + ts + 1
	else off1(o, "dist") end
	if S.espWeapon then
		local tool = c:FindFirstChildOfClass("Tool")
		txt(o, "weapon", tool and tool.Name or "—", ts, Color3.fromRGB(200, 200, 200), Vector2.new(rv.X, yy))
	else off1(o, "weapon") end

	if S.espHealth then
		local pct = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
		local hc = RED:Lerp(WHITE, pct)
		local bg = obj(o, "hpBg", "Square", {Filled = true, Color = BLACK})
		P(bg, "Position", Vector2.new(x - 7, y - 1)); P(bg, "Size", Vector2.new(5, h + 2)); P(bg, "Visible", true)
		local hp = obj(o, "hp", "Square", {Filled = true})
		P(hp, "Position", Vector2.new(x - 6, y + h * (1 - pct))); P(hp, "Size", Vector2.new(3, h * pct))
		P(hp, "Color", hc); P(hp, "Visible", true)
		txt(o, "hpTxt", tostring(math.floor(hum.Health)), math.max(ts - 2, 9), hc, Vector2.new(x - 20, y + h * (1 - pct) - 5))
	else off1(o, "hpBg"); off1(o, "hp"); off1(o, "hpTxt") end

	if S.espTracer then
		local from
		if S.espTracerFrom == 1 then from = Vector2.new(center.X, vp.Y)
		elseif S.espTracerFrom == 2 then from = center
		else from = Vector2.new(center.X, 0) end
		local t = obj(o, "tracer", "Line", {Thickness = 1})
		P(t, "From", from); P(t, "To", Vector2.new(rv.X, y + h)); P(t, "Color", color); P(t, "Visible", true)
	else off1(o, "tracer") end

	if S.espHead and i.head then
		local hp = cam:WorldToViewportPoint(i.head.Position)
		local d = obj(o, "headDot", "Circle", {Thickness = 1, NumSides = 20, Filled = false})
		P(d, "Position", Vector2.new(hp.X, hp.Y)); P(d, "Radius", math.max(h * 0.08, 2)); P(d, "Color", color); P(d, "Visible", true)
	else off1(o, "headDot") end

	if S.espSkeleton then
		if not i.skel then
			local list = (i.torso and i.torso.Name == "UpperTorso") and R15 or R6
			i.skel = {}
			for _, pair in ipairs(list) do
				local a, b = c:FindFirstChild(pair[1]), c:FindFirstChild(pair[2])
				if a and b then i.skel[#i.skel + 1] = {a, b} end
			end
		end
		for k, pr in ipairs(i.skel) do
			local l = obj(o, "s" .. k, "Line", {Thickness = 1})
			local pa = cam:WorldToViewportPoint(pr[1].Position)
			local pb = cam:WorldToViewportPoint(pr[2].Position)
			if pa.Z > 0 and pb.Z > 0 then
				P(l, "From", Vector2.new(pa.X, pa.Y)); P(l, "To", Vector2.new(pb.X, pb.Y))
				P(l, "Color", color); P(l, "Visible", true)
			else P(l, "Visible", false) end
		end
		o.skelMade = #i.skel
	elseif o.skelMade then
		for k = 1, o.skelMade do off1(o, "s" .. k) end
		o.skelMade = nil
	end
end

local espWasOn, lastEsp = false, 0
local function espStep(cam, now)
	if not S.espOn then
		if espWasOn then
			espWasOn = false
			for _, o in pairs(espObjs) do hideAll(o) end
			for p, hl in pairs(highlights) do hl:Destroy(); highlights[p] = nil end
		end
		return
	end
	espWasOn = true
	if now - lastEsp < 1 / S.espFps - 0.002 then return end
	lastEsp = now

	local vp = cam.ViewportSize
	local center = vp / 2
	local camPos = cam.CFrame.Position
	for _, p in ipairs(plist) do
		local i = live(p)
		local show = i ~= nil and isEnemy(p, S.espTeam)
		local dist = 0
		if show then
			dist = (i.root.Position - camPos).Magnitude
			if dist > S.espMaxDist then show = false end
		end
		if show then
			local color = (S.espTeamColor and p.Team) and p.TeamColor.Color or col(S.espColor)
			if HAS_DRAW then
				local o = espObjs[p]
				if not o then o = {list = {}}; espObjs[p] = o end
				drawPlayer(o, p, i, cam, vp, center, color, dist)
			end
			updateChams(p, i, true, color)
		else
			local o = espObjs[p]
			if o then hideAll(o) end
			if highlights[p] then updateChams(p, i, false) end
		end
	end
end

---------------------------------------------------------------- TROLL
local TR = {orbit = 0, nfHum = nil, nfApplied = false, saved = nil}
local flyBV, flyBG
local flyUp, flyDown = false, false
local wsApplied, jpApplied, gravApplied, fovApplied = false, false, false, false
local origWS, origJP
local origGrav = Workspace.Gravity
local lpParts = {}

local function flyClean()
	if flyBV then flyBV:Destroy(); flyBV = nil end
	if flyBG then flyBG:Destroy(); flyBG = nil end
	local ch = LP.Character
	local h = ch and ch:FindFirstChildOfClass("Humanoid")
	if h then h.PlatformStand = false end
end

local function cacheLP(ch)
	lpParts = {}
	for _, v in ipairs(ch:GetDescendants()) do
		if v:IsA("BasePart") then lpParts[#lpParts + 1] = v end
	end
	track(ch.DescendantAdded:Connect(function(v)
		if v:IsA("BasePart") then lpParts[#lpParts + 1] = v end
	end))
end
if LP.Character then cacheLP(LP.Character) end
track(LP.CharacterAdded:Connect(cacheLP))

local function movStep(dt)
	if S.gravOn then
		Workspace.Gravity = S.gravity; gravApplied = true
	elseif gravApplied then
		Workspace.Gravity = origGrav; gravApplied = false
	end

	if not (S.flyOn or S.speedOn or S.jumpOn or S.spinOn or S.bhop or S.orbitOn or S.followOn or S.nofall or TR.nfApplied or flyBV or wsApplied or jpApplied) then return end
	local char = LP.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")
	if not hum or not root then return end
	local cam = getCam()

	if S.flyOn then
		if not flyBV or flyBV.Parent ~= root then
			flyClean()
			flyBV = new("BodyVelocity", {MaxForce = Vector3.new(9e9, 9e9, 9e9), Velocity = Vector3.zero}, root)
			flyBG = new("BodyGyro", {MaxTorque = Vector3.new(9e9, 9e9, 9e9), P = 9e4}, root)
		end
		hum.PlatformStand = true
		local md = hum.MoveDirection
		local look, right = cam.CFrame.LookVector, cam.CFrame.RightVector
		local fl = Vector3.new(look.X, 0, look.Z)
		local fr = Vector3.new(right.X, 0, right.Z)
		fl = fl.Magnitude > 0 and fl.Unit or Vector3.zero
		fr = fr.Magnitude > 0 and fr.Unit or Vector3.zero
		local vel = look * md:Dot(fl) + right * md:Dot(fr)
		local up = flyUp or UIS:IsKeyDown(Enum.KeyCode.Space)
		local down = flyDown or UIS:IsKeyDown(Enum.KeyCode.LeftControl)
		vel = vel + Vector3.new(0, (up and 1 or 0) - (down and 1 or 0), 0)
		flyBV.Velocity = vel * S.flySpeed
		flyBG.CFrame = cam.CFrame
	elseif flyBV then
		flyClean()
	end

	if S.speedOn and not S.flyOn and S.speedMode ~= 3 then
		local md = hum.MoveDirection
		if md.Magnitude > 0 then
			if S.speedMode == 1 then
				local v = root.AssemblyLinearVelocity
				root.AssemblyLinearVelocity = Vector3.new(md.X * S.speed, v.Y, md.Z * S.speed)
			else
				root.CFrame = root.CFrame + md * S.speed * dt
			end
		end
	end
	if S.speedOn and S.speedMode == 3 then
		if not wsApplied then origWS = hum.WalkSpeed; wsApplied = true end
		P(hum, "WalkSpeed", S.speed)
	elseif wsApplied then
		hum.WalkSpeed = origWS or 16; wsApplied = false
	end

	if S.jumpOn then
		if not jpApplied then origJP = hum.JumpPower; jpApplied = true end
		hum.UseJumpPower = true
		P(hum, "JumpPower", S.jump)
	elseif jpApplied then
		hum.JumpPower = origJP or 50; jpApplied = false
	end

	if S.spinOn and not S.flyOn then
		root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(S.spinSpeed * dt), 0)
	end

	if S.bhop and hum.MoveDirection.Magnitude > 0 and hum.FloorMaterial ~= Enum.Material.Air
		and (UIS:IsKeyDown(Enum.KeyCode.Space) or not UIS.KeyboardEnabled) then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end

	if (S.nofall and TR.nfHum ~= hum) or (not S.nofall and TR.nfApplied) then
		TR.nfHum = S.nofall and hum or nil
		TR.nfApplied = S.nofall
		pcall(function()
			hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, not S.nofall)
			hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, not S.nofall)
		end)
	end

	if (S.orbitOn or S.followOn) and not S.flyOn then
		local tp = Players:FindFirstChild(S.tpName)
		local tr = tp and tp.Character and tp.Character:FindFirstChild("HumanoidRootPart")
		if tr then
			if S.orbitOn then
				TR.orbit = TR.orbit + dt * S.orbitSpeed
				local off = Vector3.new(math.cos(TR.orbit), 0, math.sin(TR.orbit)) * S.orbitDist
				root.CFrame = CFrame.new(tr.Position + off, tr.Position)
			else
				root.CFrame = root.CFrame:Lerp(tr.CFrame * CFrame.new(0, 0, S.followDist), math.clamp(dt * 8, 0, 1))
			end
		end
	end
end

track(RunService.Stepped:Connect(function()
	if S.noclip then
		for _, v in ipairs(lpParts) do
			if v.Parent and v.CanCollide then v.CanCollide = false end
		end
	end
end))
track(UIS.JumpRequest:Connect(function()
	if S.infJump then
		local ch = LP.Character
		local h = ch and ch:FindFirstChildOfClass("Humanoid")
		if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end))
track(LP.Idled:Connect(function()
	if S.afk then
		pcall(function()
			VirtualUser:CaptureController()
			VirtualUser:ClickButton2(Vector2.new())
		end)
	end
end))

local function tpNext()
	local list = {}
	for _, p in ipairs(plist) do list[#list + 1] = p.Name end
	table.sort(list)
	if #list == 0 then S.tpName = "—" return end
	local idx = table.find(list, S.tpName) or 0
	S.tpName = list[idx % #list + 1]
end

---------------------------------------------------------------- UI
local C = {
	bg = Color3.fromRGB(8, 8, 9), panel = Color3.fromRGB(13, 13, 15), card = Color3.fromRGB(18, 18, 20),
	row = Color3.fromRGB(26, 26, 29), line = Color3.fromRGB(45, 18, 22),
	red = Color3.fromRGB(235, 25, 45), dark = Color3.fromRGB(110, 8, 18),
	text = Color3.fromRGB(240, 240, 240), dim = Color3.fromRGB(125, 125, 130), off = Color3.fromRGB(48, 48, 52),
}
local uiScale = math.clamp(math.min(getCam().ViewportSize.X / 640, getCam().ViewportSize.Y / 400), 0.55, 1.15)

local win = new("Frame", {
	Name = "Win", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.fromOffset(580, 360), BackgroundColor3 = C.bg, BorderSizePixel = 0,
}, gui)
corner(win, 12)
stroke(win, C.red, 1.5, 0.15)
local winScale = new("UIScale", {Scale = uiScale}, win)
new("UIGradient", {
	Rotation = 90,
	Color = ColorSequence.new(Color3.fromRGB(16, 8, 10), Color3.fromRGB(6, 6, 7)),
}, win)

local top = new("Frame", {Size = UDim2.new(1, 0, 0, 46), BackgroundColor3 = C.panel, BorderSizePixel = 0, Active = true}, win)
corner(top, 12)
new("Frame", {Size = UDim2.new(1, 0, 0, 12), Position = UDim2.new(0, 0, 1, -12), BackgroundColor3 = C.panel, BorderSizePixel = 0}, top)
local accent = new("Frame", {Size = UDim2.new(1, 0, 0, 2), Position = UDim2.fromOffset(0, 46), BackgroundColor3 = C.red, BorderSizePixel = 0}, win)
new("UIGradient", {Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, C.dark), ColorSequenceKeypoint.new(0.5, RED), ColorSequenceKeypoint.new(1, C.dark),
})}, accent)

local logo = logoImage("ImageLabel", {Size = UDim2.fromOffset(38, 38), Position = UDim2.fromOffset(9, 4)}, top)
if not logo then
	logo = new("TextLabel", {
		Text = "X", Size = UDim2.fromOffset(30, 30), Position = UDim2.fromOffset(12, 8), BackgroundColor3 = C.red,
		TextColor3 = BLACK, Font = Enum.Font.GothamBlack, TextSize = 20,
	}, top)
	corner(logo, 8)
	new("UIGradient", {Rotation = 45, Color = ColorSequence.new(RED, C.dark)}, logo)
end
new("TextLabel", {
	Text = "XIT DA FACÇÃO", Size = UDim2.fromOffset(200, 22), Position = UDim2.fromOffset(54, 5),
	BackgroundTransparency = 1, TextColor3 = WHITE, Font = Enum.Font.GothamBlack, TextSize = 17,
	TextXAlignment = Enum.TextXAlignment.Left,
}, top)
new("TextLabel", {
	Text = "TFJ  •  DELTA  •  v4.0", Size = UDim2.fromOffset(200, 14), Position = UDim2.fromOffset(54, 26),
	BackgroundTransparency = 1, TextColor3 = C.red, Font = Enum.Font.GothamBold, TextSize = 10,
	TextXAlignment = Enum.TextXAlignment.Left,
}, top)

local function topBtn(txtv, xoff, bg)
	local b = new("TextButton", {
		Text = txtv, Size = UDim2.fromOffset(30, 26), AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, xoff, 0, 23), BackgroundColor3 = bg, TextColor3 = WHITE,
		Font = Enum.Font.GothamBold, TextSize = 14, AutoButtonColor = false,
	}, top)
	corner(b, 7)
	b.MouseEnter:Connect(function() tw(b, {BackgroundColor3 = C.red}) end)
	b.MouseLeave:Connect(function() tw(b, {BackgroundColor3 = bg}) end)
	return b
end
local minBtn = topBtn("—", -46, C.row)
local closeBtn = topBtn("X", -12, C.dark)

local function draggable(handle, target, onTap)
	local drag, startPos, base, startInput, moved = false, nil, nil, nil, 0
	local function isPress(i) return i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch end
	track(handle.InputBegan:Connect(function(i)
		if isPress(i) then drag = true; moved = 0; startPos = i.Position; base = target.Position; startInput = i end
	end))
	track(UIS.InputChanged:Connect(function(i)
		if not drag then return end
		local mv = i.UserInputType == Enum.UserInputType.MouseMovement
		local tc = i.UserInputType == Enum.UserInputType.Touch and i == startInput
		if mv or tc then
			local d = i.Position - startPos
			moved = math.max(moved, math.sqrt(d.X * d.X + d.Y * d.Y))
			target.Position = UDim2.new(base.X.Scale, base.X.Offset + d.X, base.Y.Scale, base.Y.Offset + d.Y)
		end
	end))
	track(UIS.InputEnded:Connect(function(i)
		if drag and isPress(i) and (i.UserInputType == Enum.UserInputType.MouseButton1 or i == startInput) then
			drag = false
			if moved < 6 and onTap then onTap() end
		end
	end))
end
draggable(top, win)

local float = logoImage("ImageButton", {
	Size = UDim2.fromOffset(54, 54), Position = UDim2.new(0, 14, 0.5, -27), AutoButtonColor = false,
}, gui)
if not float then
	float = new("TextButton", {
		Text = "X", Size = UDim2.fromOffset(46, 46), Position = UDim2.new(0, 14, 0.5, -23),
		BackgroundColor3 = C.bg, TextColor3 = C.red, Font = Enum.Font.GothamBlack, TextSize = 22, AutoButtonColor = false,
	}, gui)
	corner(float, 23)
	stroke(float, C.red, 2)
end
draggable(float, float, function() win.Visible = not win.Visible end)
minBtn.Activated:Connect(function() win.Visible = false end)
track(UIS.InputBegan:Connect(function(i, gp)
	if not gp and (i.KeyCode == Enum.KeyCode.RightShift or i.KeyCode == Enum.KeyCode.Insert) then
		win.Visible = not win.Visible
	end
end))

local flyBtns = new("Frame", {Size = UDim2.fromOffset(56, 120), Position = UDim2.new(1, -76, 0.5, -60), BackgroundTransparency = 1, Visible = false}, gui)
local function holdBtn(lbl, y, setter)
	local b = new("TextButton", {
		Text = lbl, Size = UDim2.fromOffset(56, 56), Position = UDim2.fromOffset(0, y), BackgroundColor3 = C.bg,
		TextColor3 = C.red, Font = Enum.Font.GothamBlack, TextSize = 13, AutoButtonColor = false,
	}, flyBtns)
	corner(b, 28)
	stroke(b, C.red, 2)
	local function press(i) return i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch end
	track(b.InputBegan:Connect(function(i) if press(i) then setter(true); tw(b, {BackgroundColor3 = C.dark}, 0.08) end end))
	track(b.InputEnded:Connect(function(i) if press(i) then setter(false); tw(b, {BackgroundColor3 = C.bg}, 0.08) end end))
end
holdBtn("SOBE", 0, function(v) flyUp = v end)
holdBtn("DESCE", 64, function(v) flyDown = v end)

local sidebar = new("Frame", {Position = UDim2.fromOffset(10, 58), Size = UDim2.new(0, 120, 1, -68), BackgroundColor3 = C.panel, BorderSizePixel = 0}, win)
corner(sidebar, 10)
stroke(sidebar, C.line, 1)
local tabList = new("ScrollingFrame", {Size = UDim2.new(1, 0, 1, -44), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 0, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y}, sidebar)
new("UIListLayout", {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder}, tabList)
new("UIPadding", {PaddingTop = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8)}, tabList)

new("Frame", {Size = UDim2.new(1, -16, 0, 1), Position = UDim2.new(0, 8, 1, -42), BackgroundColor3 = C.line, BorderSizePixel = 0}, sidebar)
local fpsLbl = new("TextLabel", {
	Text = "FPS 60", Size = UDim2.new(1, -16, 0, 16), Position = UDim2.new(0, 8, 1, -38), BackgroundTransparency = 1,
	TextColor3 = C.red, Font = Enum.Font.GothamBold, TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left,
}, sidebar)
new("TextLabel", {
	Text = "made by nexus", Size = UDim2.new(1, -16, 0, 16), Position = UDim2.new(0, 8, 1, -22), BackgroundTransparency = 1,
	TextColor3 = C.dim, Font = Enum.Font.Gotham, TextSize = 10, TextXAlignment = Enum.TextXAlignment.Left,
}, sidebar)

local pageHolder = new("Frame", {Position = UDim2.fromOffset(138, 58), Size = UDim2.new(1, -148, 1, -68), BackgroundTransparency = 1, ClipsDescendants = true}, win)
local pages, tabBtns = {}, {}
local builders, built, refresh = {}, {}, {}
local curPage

local function nextOrder(o)
	local n = (o:GetAttribute("n") or 0) + 1
	o:SetAttribute("n", n)
	return n
end

local function newPage()
	local sf = new("ScrollingFrame", {
		Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3,
		ScrollBarImageColor3 = C.red, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, Visible = false,
	}, pageHolder)
	new("UIListLayout", {Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder}, sf)
	new("UIPadding", {PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6)}, sf)
	return sf
end

local function selectTab(name)
	if builders[name] and not built[name] then
		built[name] = true
		curPage = pages[name]
		local ok, err = pcall(builders[name])
		if not ok then warn("[XIT] aba " .. name .. ": " .. tostring(err)) end
	end
	for n, pg in pairs(pages) do pg.Visible = (n == name) end
	for n, b in pairs(tabBtns) do
		local a = (n == name)
		tw(b, {BackgroundColor3 = a and C.red or C.row, TextColor3 = a and WHITE or C.dim})
		b.Ind.Visible = a
	end
end

local function makeTab(name, order, builder)
	local b = new("TextButton", {
		Text = name, Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = C.row, TextColor3 = C.dim,
		Font = Enum.Font.GothamBlack, TextSize = 13, AutoButtonColor = false, LayoutOrder = order,
	}, tabList)
	corner(b, 8)
	new("Frame", {Name = "Ind", Size = UDim2.new(0, 3, 0.55, 0), Position = UDim2.new(0, 0, 0.225, 0), BackgroundColor3 = WHITE, BorderSizePixel = 0, Visible = false}, b)
	b.Activated:Connect(function() selectTab(name) end)
	tabBtns[name] = b
	pages[name] = newPage()
	builders[name] = builder
	return pages[name]
end

local function card(title)
	local f = new("Frame", {
		Size = UDim2.new(1, -4, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = C.card,
		BorderSizePixel = 0, LayoutOrder = nextOrder(curPage),
	}, curPage)
	corner(f, 10)
	stroke(f, C.line, 1)
	new("UIListLayout", {Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, f)
	new("UIPadding", {PaddingTop = UDim.new(0, 8), PaddingBottom = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8)}, f)
	local hd = new("Frame", {Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1, LayoutOrder = 0}, f)
	new("Frame", {Size = UDim2.fromOffset(3, 14), Position = UDim2.fromOffset(0, 3), BackgroundColor3 = C.red, BorderSizePixel = 0}, hd)
	new("TextLabel", {
		Text = title, Size = UDim2.new(1, -10, 1, 0), Position = UDim2.fromOffset(10, 0), BackgroundTransparency = 1,
		TextColor3 = WHITE, Font = Enum.Font.GothamBlack, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left,
	}, hd)
	return f
end

local function row(parent, h, cls)
	local f = new(cls or "Frame", {Size = UDim2.new(1, 0, 0, h or 34), BackgroundColor3 = C.row, BorderSizePixel = 0, LayoutOrder = nextOrder(parent)}, parent)
	if cls == "TextButton" then f.Text = ""; f.AutoButtonColor = false end
	corner(f, 7)
	return f
end
local function label(parent, text)
	return new("TextLabel", {
		Text = text, Size = UDim2.new(1, -80, 0, 34), Position = UDim2.fromOffset(12, 0), BackgroundTransparency = 1,
		TextColor3 = C.text, Font = Enum.Font.GothamMedium, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left,
	}, parent)
end

local function toggle(parent, name, key, cb)
	local f = row(parent, 34, "TextButton")
	label(f, name)
	local sw = new("Frame", {Size = UDim2.fromOffset(40, 20), AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -10, 0.5, 0), BackgroundColor3 = C.off, BorderSizePixel = 0}, f)
	corner(sw, 10)
	local knob = new("Frame", {Size = UDim2.fromOffset(16, 16), Position = UDim2.fromOffset(2, 2), BackgroundColor3 = WHITE, BorderSizePixel = 0}, sw)
	corner(knob, 8)
	local function render(anim)
		local on = S[key]
		local g1 = {BackgroundColor3 = on and C.red or C.off}
		local g2 = {Position = on and UDim2.fromOffset(22, 2) or UDim2.fromOffset(2, 2)}
		if anim then tw(sw, g1); tw(knob, g2) else sw.BackgroundColor3 = g1.BackgroundColor3; knob.Position = g2.Position end
	end
	render(false)
	refresh[key] = function() render(false) end
	f.MouseEnter:Connect(function() tw(f, {BackgroundColor3 = Color3.fromRGB(34, 34, 38)}) end)
	f.MouseLeave:Connect(function() tw(f, {BackgroundColor3 = C.row}) end)
	f.Activated:Connect(function()
		S[key] = not S[key]
		render(true)
		if cb then cb(S[key]) end
	end)
end

local function slider(parent, name, key, min, max, step, suffix, cb)
	local page = curPage
	local f = row(parent, 50)
	new("TextLabel", {
		Text = name, Size = UDim2.new(1, -90, 0, 26), Position = UDim2.fromOffset(12, 0), BackgroundTransparency = 1,
		TextColor3 = C.text, Font = Enum.Font.GothamMedium, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left,
	}, f)
	local val = new("TextLabel", {
		Text = "", Size = UDim2.new(0, 80, 0, 26), AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -12, 0, 0),
		BackgroundTransparency = 1, TextColor3 = C.red, Font = Enum.Font.GothamBold, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Right,
	}, f)
	local hit = new("TextButton", {Text = "", AutoButtonColor = false, Size = UDim2.new(1, -24, 0, 22), Position = UDim2.fromOffset(12, 24), BackgroundTransparency = 1}, f)
	local bar = new("Frame", {Size = UDim2.new(1, 0, 0, 6), AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 0, 0.5, 0), BackgroundColor3 = C.off, BorderSizePixel = 0}, hit)
	corner(bar, 3)
	local fill = new("Frame", {Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = C.red, BorderSizePixel = 0}, bar)
	corner(fill, 3)
	new("UIGradient", {Color = ColorSequence.new(C.dark, RED)}, fill)
	local knob = new("Frame", {Size = UDim2.fromOffset(14, 14), AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.new(0, 0, 0.5, 0), BackgroundColor3 = WHITE, BorderSizePixel = 0}, bar)
	corner(knob, 7)
	stroke(knob, C.red, 2)
	local dec = step >= 1 and 0 or (step >= 0.1 and 1 or 2)
	local function render()
		local rel = math.clamp((S[key] - min) / (max - min), 0, 1)
		fill.Size = UDim2.new(rel, 0, 1, 0)
		knob.Position = UDim2.new(rel, 0, 0.5, 0)
		val.Text = string.format("%." .. dec .. "f", S[key]) .. (suffix or "")
	end
	render()
	refresh[key] = render
	local function setX(x)
		local rel = math.clamp((x - hit.AbsolutePosition.X) / math.max(hit.AbsoluteSize.X, 1), 0, 1)
		S[key] = math.clamp(math.floor((min + (max - min) * rel) / step + 0.5) * step, min, max)
		render()
		if cb then cb(S[key]) end
	end
	local dragging, dInput = false, nil
	local function isPress(i) return i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch end
	track(hit.InputBegan:Connect(function(i)
		if isPress(i) then dragging = true; dInput = i; page.ScrollingEnabled = false; setX(i.Position.X) end
	end))
	track(UIS.InputChanged:Connect(function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or (i.UserInputType == Enum.UserInputType.Touch and i == dInput)) then
			setX(i.Position.X)
		end
	end))
	track(UIS.InputEnded:Connect(function(i)
		if dragging and isPress(i) and (i.UserInputType == Enum.UserInputType.MouseButton1 or i == dInput) then
			dragging = false; page.ScrollingEnabled = true
		end
	end))
end

local function cycle(parent, name, key, options)
	local f = row(parent, 34)
	label(f, name)
	local b = new("TextButton", {
		Text = options[S[key]], Size = UDim2.fromOffset(120, 24), AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -8, 0.5, 0),
		BackgroundColor3 = C.panel, TextColor3 = C.red, Font = Enum.Font.GothamBold, TextSize = 12, AutoButtonColor = false,
		TextTruncate = Enum.TextTruncate.AtEnd,
	}, f)
	corner(b, 6)
	stroke(b, C.dark, 1)
	refresh[key] = function() b.Text = options[S[key]] end
	b.Activated:Connect(function()
		S[key] = S[key] % #options + 1
		b.Text = options[S[key]]
	end)
	return b
end

local function button(parent, name, cb)
	local b = new("TextButton", {
		Text = name, Size = UDim2.new(1, 0, 0, 34), BackgroundColor3 = C.red, TextColor3 = WHITE,
		Font = Enum.Font.GothamBlack, TextSize = 13, AutoButtonColor = false, LayoutOrder = nextOrder(parent),
	}, parent)
	corner(b, 7)
	new("UIGradient", {Rotation = 90, Color = ColorSequence.new(RED, C.dark)}, b)
	b.Activated:Connect(cb)
	return b
end

---------------------------------------------------------------- v4: SERVIÇOS, HELPERS E MÓDULOS
local Lighting, TeleportService, HttpService, Stats = svc("Lighting"), svc("TeleportService"), svc("HttpService"), svc("Stats")
local Teams = svc("Teams")
local nc = newcclosure or function(f) return f end
local alive = true
local M = {cleanups = {}}
local function onCleanup(f) M.cleanups[#M.cleanups + 1] = f end

local function picker(parent, name, key, getList)
	local f = row(parent, 34)
	label(f, name)
	local b = new("TextButton", {
		Text = S[key], Size = UDim2.fromOffset(120, 24), AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -8, 0.5, 0),
		BackgroundColor3 = C.panel, TextColor3 = C.red, Font = Enum.Font.GothamBold, TextSize = 12, AutoButtonColor = false,
		TextTruncate = Enum.TextTruncate.AtEnd,
	}, f)
	corner(b, 6)
	stroke(b, C.dark, 1)
	refresh[key] = function() b.Text = S[key] end
	b.Activated:Connect(function()
		local list = getList()
		if #list == 0 then S[key] = "—"
		else S[key] = list[((table.find(list, S[key]) or 0) % #list) + 1] end
		b.Text = S[key]
	end)
	return b
end

local function status(parent, text, h)
	local r = row(parent, h or 34)
	return new("TextLabel", {
		Text = text, Size = UDim2.new(1, -20, 1, 0), Position = UDim2.fromOffset(10, 0), BackgroundTransparency = 1,
		TextColor3 = C.dim, Font = Enum.Font.Gotham, TextSize = 12, TextWrapped = true,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, r)
end

local function myRoot()
	local ch = LP.Character
	return ch and ch:FindFirstChild("HumanoidRootPart"), ch and ch:FindFirstChildOfClass("Humanoid")
end

local function playerNames()
	local l = {}
	for _, p in ipairs(plist) do l[#l + 1] = p.Name end
	table.sort(l)
	return l
end

---------------------------------------------------------------- DANO POR HIT (hook só sob demanda)
do
	local armUntil = 0
	local function dmgMult() return math.max(1, math.ceil(S.dmgWant / math.max(S.dmgBase, 1))) end

	local function touchHit(tool)
		local h = tool:FindFirstChild("Handle")
		local root = myRoot()
		if not h or not root or not firetouchinterest then return end
		local best, bd = nil, S.dmgReach
		for _, p in ipairs(plist) do
			local i = live(p)
			if i and isEnemy(p, S.dmgTeam) then
				local d = (i.root.Position - root.Position).Magnitude
				if d <= bd then bd = d; best = i end
			end
		end
		if best then
			task.spawn(function()
				for _ = 1, dmgMult() do
					pcall(firetouchinterest, h, best.root, 0)
					pcall(firetouchinterest, h, best.root, 1)
					task.wait()
				end
			end)
		end
	end

	local oldNC, hooked = nil, false
	function M.dmgHook()
		if hooked or not hookmetamethod then return end
		hooked = true
		pcall(function()
			oldNC = hookmetamethod(game, "__namecall", nc(function(self, ...)
				if alive and S.dmgOn and S.dmgMode == 1 and os.clock() < armUntil and not checkcaller() then
					if getnamecallmethod() == "FireServer" and typeof(self) == "Instance" and self:IsA("RemoteEvent") then
						local args = table.pack(...)
						local extra = dmgMult() - 1
						if extra > 0 then
							task.spawn(function()
								for _ = 1, extra do pcall(self.FireServer, self, table.unpack(args, 1, args.n)) end
							end)
						end
					end
				end
				return oldNC(self, ...)
			end))
		end)
	end

	local watched = setmetatable({}, {__mode = "k"})
	local function watch(t)
		if not t:IsA("Tool") or watched[t] then return end
		watched[t] = true
		track(t.Activated:Connect(function()
			armUntil = os.clock() + 0.35
			if S.dmgOn and S.dmgMode == 1 then M.dmgHook() end
			if S.dmgOn and S.dmgMode == 2 then touchHit(t) end
		end))
	end
	local function watchAll(parent)
		for _, t in ipairs(parent:GetChildren()) do watch(t) end
		track(parent.ChildAdded:Connect(watch))
	end
	watchAll(LP:WaitForChild("Backpack"))
	if LP.Character then watchAll(LP.Character) end
	track(LP.CharacterAdded:Connect(function(ch) watchAll(ch); watchAll(LP:WaitForChild("Backpack")) end))
end

---------------------------------------------------------------- TROLL: invisível, click TP, freecam, waypoint
do
	-- invisível (efeito local: só você vê o personagem sumir)
	local invSaved = setmetatable({}, {__mode = "k"})
	local active, origDisp = false, nil
	local function applyInvis(ch, alpha)
		for _, v in ipairs(ch:GetDescendants()) do
			if v:IsA("BasePart") or v:IsA("Decal") then
				if invSaved[v] == nil then invSaved[v] = v.Transparency end
				local target = math.max(invSaved[v], alpha)
				if v.Transparency ~= target then v.Transparency = target end
			end
		end
	end
	local function restoreInvis()
		for v, o in pairs(invSaved) do
			pcall(function() v.Transparency = o end)
			invSaved[v] = nil
		end
		local _, hum = myRoot()
		if hum and origDisp then pcall(function() hum.DisplayDistanceType = origDisp end) end
		origDisp = nil
	end
	task.spawn(function()
		while gui.Parent do
			local ch = LP.Character
			if S.invisOn and ch then
				active = true
				pcall(applyInvis, ch, S.invisAlpha)
				local hum = ch:FindFirstChildOfClass("Humanoid")
				if hum then
					if S.invisName then
						if hum.DisplayDistanceType ~= Enum.HumanoidDisplayDistanceType.None then
							origDisp = origDisp or hum.DisplayDistanceType
							hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
						end
					elseif origDisp then
						hum.DisplayDistanceType = origDisp; origDisp = nil
					end
				end
			elseif active then
				active = false
				restoreInvis()
			end
			task.wait(0.25)
		end
	end)
	onCleanup(restoreInvis)

	-- teleporte por raycast
	local rpTp = RaycastParams.new()
	rpTp.FilterType = Enum.RaycastFilterType.Exclude
	local function tpTo(x, y)
		local cam, root = getCam(), (myRoot())
		if not cam or not root then return end
		local ray = cam:ViewportPointToRay(x, y)
		rpTp.FilterDescendantsInstances = {LP.Character}
		local res = Workspace:Raycast(ray.Origin, ray.Direction * 2000, rpTp)
		if res then root.CFrame = CFrame.new(res.Position + Vector3.new(0, 3.5, 0)) end
	end
	function M.tpLook()
		local cam = getCam()
		if cam then tpTo(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2) end
	end
	track(UIS.InputBegan:Connect(function(i, gp)
		if gp or not S.clickTp then return end
		if i.UserInputType == Enum.UserInputType.MouseButton1 and UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
			local m = UIS:GetMouseLocation()
			tpTo(m.X, m.Y)
		end
	end))

	function M.savePos()
		local root = myRoot()
		if root then TR.saved = root.CFrame end
	end
	function M.loadPos()
		local root = myRoot()
		if root and TR.saved then root.CFrame = TR.saved end
	end
	function M.sit()
		local _, hum = myRoot()
		if hum then hum.Sit = true end
	end

	-- zoom sem limite
	local zoomOrig
	function M.zoom(on)
		pcall(function()
			if on then
				zoomOrig = zoomOrig or {LP.CameraMaxZoomDistance, LP.CameraMinZoomDistance, LP.CameraMode}
				LP.CameraMaxZoomDistance = 1e5
				LP.CameraMinZoomDistance = 0.5
				LP.CameraMode = Enum.CameraMode.Classic
			elseif zoomOrig then
				LP.CameraMaxZoomDistance = zoomOrig[1]
				LP.CameraMinZoomDistance = zoomOrig[2]
				LP.CameraMode = zoomOrig[3]
				zoomOrig = nil
			end
		end)
	end
	onCleanup(function() M.zoom(false) end)

	-- freecam
	local fc = {on = false, cf = nil, yaw = 0, pitch = 0}
	function M.fcSet(on)
		local cam = getCam()
		if not cam then return end
		if on and not fc.on then
			fc.on = true
			fc.cf = cam.CFrame
			local rx, ry = cam.CFrame:ToOrientation()
			fc.pitch, fc.yaw = rx, ry
			cam.CameraType = Enum.CameraType.Scriptable
			local root = myRoot()
			if root then root.Anchored = true end
		elseif not on and fc.on then
			fc.on = false
			cam.CameraType = Enum.CameraType.Custom
			local root = myRoot()
			if root then root.Anchored = false end
		end
	end
	track(UIS.InputChanged:Connect(function(i, gp)
		if not fc.on then return end
		local mouse = i.UserInputType == Enum.UserInputType.MouseMovement and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
		local touch = i.UserInputType == Enum.UserInputType.Touch and not gp
		if mouse or touch then
			fc.yaw = fc.yaw - i.Delta.X * 0.004
			fc.pitch = math.clamp(fc.pitch - i.Delta.Y * 0.004, -1.5, 1.5)
		end
	end))
	function M.fcStep(cam, dt)
		if not fc.on then return end
		local root, hum = myRoot()
		if root and not root.Anchored then root.Anchored = true end
		local rot = CFrame.fromOrientation(fc.pitch, fc.yaw, 0)
		local md = hum and hum.MoveDirection or Vector3.zero
		local look, right = rot.LookVector, rot.RightVector
		local fl = Vector3.new(look.X, 0, look.Z)
		local fr = Vector3.new(right.X, 0, right.Z)
		fl = fl.Magnitude > 0 and fl.Unit or Vector3.zero
		fr = fr.Magnitude > 0 and fr.Unit or Vector3.zero
		local v = look * md:Dot(fl) + right * md:Dot(fr)
		local up = flyUp or UIS:IsKeyDown(Enum.KeyCode.E) or UIS:IsKeyDown(Enum.KeyCode.Space)
		local down = flyDown or UIS:IsKeyDown(Enum.KeyCode.Q) or UIS:IsKeyDown(Enum.KeyCode.LeftControl)
		v = v + Vector3.new(0, (up and 1 or 0) - (down and 1 or 0), 0)
		fc.cf = CFrame.new(fc.cf.Position + v * S.fcSpeed * dt) * rot
		cam.CFrame = fc.cf
	end
	onCleanup(function() M.fcSet(false) end)
end

---------------------------------------------------------------- TORCIDA / TIMES (sem disparar remotes às cegas)
do
	local lastJoin = 0
	function M.teamNames()
		local l = {}
		for _, t in ipairs(Teams:GetTeams()) do l[#l + 1] = t.Name end
		return l
	end

	local function clickGui(b)
		local fired = false
		if getconnections then
			for _, sig in ipairs({b.MouseButton1Click, b.Activated, b.MouseButton1Down, b.MouseButton1Up}) do
				pcall(function()
					for _, c in ipairs(getconnections(sig)) do
						pcall(function() c:Fire() end)
						fired = true
					end
				end)
			end
		end
		if not fired and firesignal then
			pcall(firesignal, b.MouseButton1Click)
			pcall(firesignal, b.Activated)
			fired = true
		end
		return fired
	end

	-- usa o botão/prompt que o próprio jogo oferece: o código do jogo é quem manda o pedido
	function M.joinTeam(setStatus)
		if os.clock() - lastJoin < 3 then return end
		lastJoin = os.clock()
		local t = Teams:FindFirstChild(S.teamPick)
		if not t then setStatus("Escolha uma torcida primeiro") return end
		local name = t.Name:lower()
		local how
		local pg = LP:FindFirstChildOfClass("PlayerGui")
		if pg then
			for _, b in ipairs(pg:GetDescendants()) do
				if (b:IsA("TextButton") or b:IsA("ImageButton")) and not b:IsDescendantOf(gui) and b.AbsoluteSize.X > 0 then
					local tx = b:IsA("TextButton") and b.Text:lower() or ""
					if (tx ~= "" and tx:find(name, 1, true)) or b.Name:lower():find(name, 1, true) then
						if clickGui(b) then how = "botão do jogo" break end
					end
				end
			end
		end
		local root = myRoot()
		if not how and root and fireproximityprompt then
			for _, v in ipairs(Workspace:GetDescendants()) do
				if v:IsA("ProximityPrompt") then
					local par = v.Parent
					local pn = (v.Name .. " " .. (par and par.Name or "") .. " " .. v.ObjectText .. " " .. v.ActionText):lower()
					local pos = par and par:IsA("BasePart") and par.Position
					if pos and pn:find(name, 1, true) and (pos - root.Position).Magnitude <= v.MaxActivationDistance + 4 then
						pcall(fireproximityprompt, v)
						how = "prompt perto de você"
						break
					end
				end
			end
		end
		if S.teamLocal then
			pcall(function() LP.Team = t end)
			how = (how and (how .. " + ") or "") .. "visual local"
		end
		task.delay(0.8, function()
			local ok = LP.Team == t
			setStatus((how and ("Usei: " .. how) or "Nada do jogo achado (chegue perto do pad/prompt)") .. "  |  Time atual: " .. (LP.Team and LP.Team.Name or "nenhum") .. (ok and "  ✔" or ""))
		end)
	end

	function M.toSpawn()
		local t = Teams:FindFirstChild(S.teamPick)
		local root = myRoot()
		if not t or not root then return end
		for _, v in ipairs(Workspace:GetDescendants()) do
			if v:IsA("SpawnLocation") and v.TeamColor == t.TeamColor then
				root.CFrame = v.CFrame * CFrame.new(0, 4, 0)
				return
			end
		end
	end
end

---------------------------------------------------------------- MUNDO (luz)
do
	local lightOrig
	local function lightRestore()
		if lightOrig then
			pcall(function()
				Lighting.Brightness = lightOrig.B; Lighting.GlobalShadows = lightOrig.G
				Lighting.Ambient = lightOrig.A; Lighting.FogEnd = lightOrig.F; Lighting.ClockTime = lightOrig.T
			end)
			lightOrig = nil
		end
	end
	task.spawn(function()
		while gui.Parent do
			if S.fullbright or S.noFog or S.clockOn then
				if not lightOrig then
					lightOrig = {B = Lighting.Brightness, G = Lighting.GlobalShadows, A = Lighting.Ambient, F = Lighting.FogEnd, T = Lighting.ClockTime}
				end
				if S.fullbright then
					Lighting.Brightness = 2; Lighting.GlobalShadows = false; Lighting.Ambient = Color3.fromRGB(190, 190, 190)
				end
				if S.noFog then Lighting.FogEnd = 1e6 end
				if S.clockOn then Lighting.ClockTime = S.clockT end
			else
				lightRestore()
			end
			task.wait(0.5)
		end
	end)
	onCleanup(lightRestore)
end

---------------------------------------------------------------- JOGADORES (espectar)
function M.spectate(on)
	local cam = getCam()
	local c = on and (Players:FindFirstChild(S.plPick) or {}).Character or LP.Character
	local h = c and c:FindFirstChildOfClass("Humanoid")
	if h and cam then cam.CameraSubject = h end
end
onCleanup(function() M.spectate(false) end)

---------------------------------------------------------------- OTIMIZAÇÃO
do
	local cats = {"part", "tex", "fx", "light", "snd", "acc", "post", "water", "grass"}
	local stores = {}
	for _, k in ipairs(cats) do stores[k] = setmetatable({}, {__mode = "k"}) end

	local function setp(cat, inst, prop, val)
		local st = stores[cat]
		local s = st[inst]
		if not s then s = {}; st[inst] = s end
		if s[prop] == nil then s[prop] = inst[prop] end
		if inst[prop] ~= val then inst[prop] = val end
	end
	local function restoreCat(cat)
		local st = stores[cat]
		for inst, props in pairs(st) do
			for k, v in pairs(props) do pcall(function() inst[k] = v end) end
			st[inst] = nil
		end
	end

	local function applyOne(v)
		if v:IsA("BasePart") then
			if S.opPlastic and not saved[v] and not (LP.Character and v:IsDescendantOf(LP.Character)) then
				setp("part", v, "Material", Enum.Material.SmoothPlastic)
				setp("part", v, "Reflectance", 0)
			end
		elseif v:IsA("Decal") or v:IsA("Texture") then
			if S.opNoTex then setp("tex", v, "Transparency", 1) end
		elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
			if S.opNoPart then setp("fx", v, "Enabled", false) end
		elseif v:IsA("PointLight") or v:IsA("SpotLight") or v:IsA("SurfaceLight") then
			if S.opNoLight then setp("light", v, "Enabled", false) end
		elseif v:IsA("Sound") then
			if S.opNoSound then setp("snd", v, "Volume", 0) end
		end
	end

	local sweeping, again = false, false
	local function sweep()
		if sweeping then again = true return end
		sweeping = true
		task.spawn(function()
			repeat
				again = false
				local list = Workspace:GetDescendants()
				for i = 1, #list do
					local v = list[i]
					if v.Parent then pcall(applyOne, v) end
					if i % 250 == 0 then task.wait() end
				end
			until not again
			sweeping = false
		end)
	end

	track(Workspace.DescendantAdded:Connect(function(v)
		if S.opPlastic or S.opNoTex or S.opNoPart or S.opNoLight or S.opNoSound then
			task.defer(function() if v.Parent then pcall(applyOne, v) end end)
		end
	end))

	local function postOne(v)
		if v:IsA("PostEffect") then
			setp("post", v, "Enabled", false)
		elseif v:IsA("Atmosphere") then
			setp("post", v, "Density", 0); setp("post", v, "Haze", 0); setp("post", v, "Glare", 0)
		end
	end

	local applied = {}
	local catFlag = {part = "opPlastic", tex = "opNoTex", fx = "opNoPart", light = "opNoLight", snd = "opNoSound"}
	local OL = {}
	local function qualityEnum(n)
		for _, e in ipairs(Enum.QualityLevel:GetEnumItems()) do
			if e.Value == n then return e end
		end
	end

	local function tick()
		local need = false
		for cat, key in pairs(catFlag) do
			local on = S[key] and true or false
			if on ~= (applied[cat] or false) then
				applied[cat] = on
				if on then need = true else restoreCat(cat) end
			end
		end
		if need then sweep() end

		-- sombras
		if S.opNoShadow then
			if OL.shadows == nil then OL.shadows = Lighting.GlobalShadows end
			if Lighting.GlobalShadows then Lighting.GlobalShadows = false end
		elseif OL.shadows ~= nil then
			Lighting.GlobalShadows = OL.shadows; OL.shadows = nil
		end

		-- efeitos de pós-processamento
		if S.opNoFx ~= (applied.post or false) then
			applied.post = S.opNoFx
			if not S.opNoFx then restoreCat("post") end
		end
		if S.opNoFx then
			for _, v in ipairs(Lighting:GetChildren()) do pcall(postOne, v) end
			local cam = getCam()
			if cam then for _, v in ipairs(cam:GetChildren()) do pcall(postOne, v) end end
		end

		-- água e grama
		local terr = Workspace.Terrain
		if S.opWater ~= (applied.water or false) then
			applied.water = S.opWater
			if S.opWater then
				pcall(function()
					setp("water", terr, "WaterWaveSize", 0); setp("water", terr, "WaterWaveSpeed", 0)
					setp("water", terr, "WaterReflectance", 0)
				end)
			else restoreCat("water") end
		end
		if S.opNoGrass ~= (applied.grass or false) then
			applied.grass = S.opNoGrass
			if S.opNoGrass then pcall(setp, "grass", terr, "Decoration", false) else restoreCat("grass") end
		end

		-- acessórios dos outros jogadores
		if S.opHideAcc ~= (applied.acc or false) then
			applied.acc = S.opHideAcc
			if not S.opHideAcc then restoreCat("acc") end
		end
		if S.opHideAcc then
			for _, p in ipairs(plist) do
				local c = p.Character
				if c then
					for _, a in ipairs(c:GetChildren()) do
						if a:IsA("Accessory") then
							local h = a:FindFirstChild("Handle")
							if h then pcall(setp, "acc", h, "Transparency", 1) end
						end
					end
				end
			end
		end

		-- qualidade gráfica
		if S.opLow then
			pcall(function()
				local r = settings().Rendering
				if OL.q == nil then OL.q = r.QualityLevel; OL.mesh = r.MeshPartDetailLevel end
				local e = qualityEnum(S.opQuality)
				if e and r.QualityLevel ~= e then r.QualityLevel = e end
				if r.MeshPartDetailLevel ~= Enum.MeshPartDetailLevel.Level04 then r.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level04 end
			end)
		elseif OL.q ~= nil then
			pcall(function()
				local r = settings().Rendering
				r.QualityLevel = OL.q
				if OL.mesh then r.MeshPartDetailLevel = OL.mesh end
			end)
			OL.q, OL.mesh = nil, nil
		end

		-- limite de FPS
		if setfpscap then
			if S.opFpsCap then
				if OL.cap == nil then OL.cap = (getfpscap and getfpscap()) or 60 end
				if OL.capSet ~= S.opCap then OL.capSet = S.opCap; pcall(setfpscap, S.opCap) end
			elseif OL.cap ~= nil then
				pcall(setfpscap, OL.cap); OL.cap, OL.capSet = nil, nil
			end
		end

		-- tela preta (renderização 3D off)
		if S.opNo3D ~= (applied.r3d or false) then
			applied.r3d = S.opNo3D
			pcall(function() RunService:Set3dRenderingEnabled(not S.opNo3D) end)
		end
	end

	task.spawn(function()
		while gui.Parent do
			pcall(tick)
			task.wait(1)
		end
	end)

	local OPT_KEYS = {"opLow", "opNoShadow", "opNoFx", "opNoPart", "opNoLight", "opNoTex", "opPlastic", "opNoSound", "opWater", "opNoGrass", "opHideAcc"}
	function M.preset(name)
		for _, k in ipairs(OPT_KEYS) do S[k] = false end
		if name == "leve" then
			S.opLow = true; S.opQuality = 4
			S.opNoShadow = true; S.opNoFx = true; S.opNoPart = true; S.opWater = true; S.opNoGrass = true
		elseif name == "ultra" then
			S.opLow = true; S.opQuality = 1
			S.opNoShadow = true; S.opNoFx = true; S.opNoPart = true; S.opNoLight = true
			S.opNoTex = true; S.opPlastic = true; S.opWater = true; S.opNoGrass = true; S.opHideAcc = true
		end
		for _, fn in pairs(refresh) do pcall(fn) end
	end

	function M.cleanMem()
		pcall(function() collectgarbage("collect") end)
	end

	function M.statsText(fps)
		local ping, mem = 0, 0
		pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue() + 0.5) end)
		pcall(function() mem = math.floor(Stats:GetTotalMemoryUsageMb() + 0.5) end)
		return string.format("FPS %d   |   Ping %d ms   |   Memória %d MB   |   Jogadores %d", fps, ping, mem, #Players:GetPlayers())
	end

	onCleanup(function()
		for _, k in pairs(catFlag) do S[k] = false end
		S.opNoShadow, S.opNoFx, S.opWater, S.opNoGrass, S.opHideAcc, S.opLow, S.opFpsCap, S.opNo3D = false, false, false, false, false, false, false, false
		pcall(tick)
		for _, c in ipairs(cats) do restoreCat(c) end
	end)
end

---------------------------------------------------------------- ABAS (construídas só quando abertas)
local function buildCombate()
	local cA = card("AIMBOT")
	toggle(cA, "Aimbot", "aimOn")
	cycle(cA, "Ativação", "aimMode", {"Sempre", "Botão dir. (PC)", "Tool na mão"})
	cycle(cA, "Mirar em", "aimPart", {"Cabeça", "Peito", "Raiz"})
	cycle(cA, "Alvo", "aimTarget", {"Perto da mira", "Perto de mim", "Menor vida"})
	slider(cA, "Suavidade (100 = travado)", "aimSmooth", 1, 100, 1)
	slider(cA, "Tamanho do FOV", "aimFov", 20, 600, 5, "px")
	toggle(cA, "Mostrar círculo FOV", "aimShowFov")
	cycle(cA, "Cor do FOV/linha", "aimFovColor", COLOR_NAMES)
	slider(cA, "Distância máxima", "aimDist", 50, 3000, 50, "m")
	toggle(cA, "Ignorar meu time", "aimTeam")
	toggle(cA, "Ignorar amigos (geral)", "ignFriends")
	toggle(cA, "Checar parede", "aimWall")
	toggle(cA, "Travar no alvo (sticky)", "aimSticky")
	toggle(cA, "Linha até o alvo", "aimLine")
	slider(cA, "Ajuste vertical da mira", "aimYOff", -3, 3, 0.1, " st")
	toggle(cA, "Predição de movimento", "aimPredOn")
	slider(cA, "Força da predição", "aimPred", 0, 0.5, 0.01)

	local cT = card("TRIGGERBOT")
	toggle(cT, "Atirar ao mirar no inimigo", "trigOn")
	slider(cT, "Intervalo entre tiros", "trigDelay", 0.02, 0.6, 0.01, "s")

	local cK = card("AUTO CLIQUE")
	toggle(cK, "Clicar com a tool na mão", "clickOn")
	slider(cK, "Intervalo do clique", "clickRate", 0.03, 1, 0.01, "s")

	local cR = card("ALCANCE DA ARMA (REACH)")
	toggle(cR, "Aumentar alcance da tool", "reachOn")
	slider(cR, "Tamanho do alcance", "reachSize", 4, 40, 1, " st")

	local cB = card("HITBOX (CAIXA)")
	toggle(cB, "Hitbox", "hbOn")
	slider(cB, "Largura", "hbW", 2, 40, 0.5)
	slider(cB, "Altura", "hbH", 2, 40, 0.5)
	slider(cB, "Transparência", "hbTrans", 0, 1, 0.05)
	cycle(cB, "Cor da hitbox", "hbColor", COLOR_NAMES)
	toggle(cB, "Ignorar meu time", "hbTeam")

	local cH = card("HITHEAD (CABEÇA)")
	toggle(cH, "HitHead", "hhOn")
	slider(cH, "Tamanho da cabeça", "hhSize", 1.5, 25, 0.5)
	toggle(cH, "Esfera visível", "hhVisual")
	slider(cH, "Transparência da esfera", "hhTrans", 0, 1, 0.05)
	cycle(cH, "Cor da esfera", "hhColor", COLOR_NAMES)
	toggle(cH, "Esconder cabeça original", "hhHide")
	toggle(cH, "Ignorar meu time", "hhTeam")

	local d1 = card("DANO POR HIT")
	toggle(d1, "Dano turbinado", "dmgOn", function(v)
		if v and S.dmgMode == 1 then M.dmgHook() end
	end)
	cycle(d1, "Modo", "dmgMode", {"Remote (arriscado)", "Toque (tool)"})
	slider(d1, "Dano desejado por hit", "dmgWant", 10, 200, 5)
	slider(d1, "Dano base da arma", "dmgBase", 1, 100, 1)
	slider(d1, "Alcance (modo toque)", "dmgReach", 5, 60, 1, "m")
	toggle(d1, "Ignorar meu time", "dmgTeam")
	status(d1, "Toque é o padrão e não usa hook. O modo Remote instala hook e é o mais fácil de ser detectado.", 48)
end

local function buildEsp()
	if not HAS_DRAW then
		local w = card("AVISO")
		local r = row(w, 34)
		label(r, "Sem Drawing neste executor: só chams")
	end
	local e1 = card("GERAL")
	toggle(e1, "ESP ligado", "espOn")
	slider(e1, "Taxa de atualização", "espFps", 15, 240, 5, " fps")
	slider(e1, "Distância máxima", "espMaxDist", 100, 5000, 100, "m")
	toggle(e1, "Só inimigos", "espTeam")
	toggle(e1, "Usar cor do time", "espTeamColor")
	cycle(e1, "Cor do ESP", "espColor", COLOR_NAMES)

	local e2 = card("CAIXA")
	toggle(e2, "Caixa", "espBox")
	cycle(e2, "Estilo", "espBoxStyle", {"Completa", "Cantos"})
	toggle(e2, "Preenchimento", "espFill")
	slider(e2, "Opacidade do preench.", "espFillOp", 0.05, 0.8, 0.05)

	local e3 = card("INFORMAÇÕES")
	toggle(e3, "Nome", "espName")
	toggle(e3, "Distância", "espDist")
	toggle(e3, "Barra de vida", "espHealth")
	toggle(e3, "Arma na mão", "espWeapon")
	slider(e3, "Tamanho do texto", "espTextSize", 10, 22, 1)

	local e4 = card("LINHAS")
	toggle(e4, "Tracer", "espTracer")
	cycle(e4, "Origem do tracer", "espTracerFrom", {"Embaixo", "Centro", "Em cima"})
	toggle(e4, "Esqueleto", "espSkeleton")
	toggle(e4, "Círculo na cabeça", "espHead")

	local e5 = card("EXTRAS")
	toggle(e5, "Chams (brilho)", "espChams")
	slider(e5, "Preench. do chams", "espChamsFill", 0, 1, 0.05)
	toggle(e5, "Setas fora da tela", "espArrow")
	slider(e5, "Raio das setas", "espArrowR", 80, 400, 10, "px")
end

local function buildTroll()
	local t1 = card("VOAR")
	toggle(t1, "Voar", "flyOn", function(v) flyBtns.Visible = v end)
	slider(t1, "Velocidade do voo", "flySpeed", 10, 300, 5)

	local t2 = card("VELOCIDADE")
	toggle(t2, "Velocidade", "speedOn")
	cycle(t2, "Modo", "speedMode", {"Velocity", "CFrame", "WalkSpeed"})
	slider(t2, "Valor", "speed", 16, 300, 2)

	local t3 = card("PULO")
	toggle(t3, "Pulo alto", "jumpOn")
	slider(t3, "Força do pulo", "jump", 50, 300, 5)
	toggle(t3, "Pulo infinito", "infJump")
	toggle(t3, "Bhop (pular andando)", "bhop")

	local tI = card("INVISÍVEL")
	toggle(tI, "Invisível", "invisOn")
	slider(tI, "Nível de invisibilidade", "invisAlpha", 0.1, 1, 0.05)
	toggle(tI, "Esconder meu nome", "invisName")
	status(tI, "Efeito local: some o seu personagem na sua tela. Os outros jogadores só deixam de ver se o jogo replicar isso.", 60)

	local t4 = card("MOVIMENTO EXTRA")
	toggle(t4, "Noclip", "noclip")
	toggle(t4, "Sem dano/ragdoll de queda", "nofall")
	toggle(t4, "Girar", "spinOn")
	slider(t4, "Velocidade do giro", "spinSpeed", 30, 3000, 30, "°/s")
	toggle(t4, "Gravidade custom", "gravOn")
	slider(t4, "Gravidade", "gravity", 0, 300, 5)
	toggle(t4, "Anti-AFK", "afk")

	local tC = card("CÂMERA")
	toggle(tC, "FOV da câmera", "fovOn")
	slider(tC, "Valor do FOV", "camFov", 40, 120, 1)
	toggle(tC, "Freecam", "freecam", function(v) M.fcSet(v); flyBtns.Visible = v or S.flyOn end)
	slider(tC, "Velocidade da freecam", "fcSpeed", 10, 300, 5)
	toggle(tC, "Zoom sem limite", "zoomInf", function(v) M.zoom(v) end)

	local t5 = card("TELEPORTE")
	picker(t5, "Jogador alvo", "tpName", playerNames)
	button(t5, "TELEPORTAR ATÉ O JOGADOR", function()
		local target = Players:FindFirstChild(S.tpName)
		local tr = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
		local mr = myRoot()
		if tr and mr then mr.CFrame = tr.CFrame * CFrame.new(0, 0, 3) end
	end)
	button(t5, "TELEPORTAR PRA ONDE EU OLHO", function() M.tpLook() end)
	toggle(t5, "Ctrl + clique teleporta (PC)", "clickTp")
	button(t5, "SALVAR MINHA POSIÇÃO", function() M.savePos() end)
	button(t5, "VOLTAR PRA POSIÇÃO SALVA", function() M.loadPos() end)

	local t6 = card("SEGUIR JOGADOR ALVO")
	toggle(t6, "Orbitar em volta", "orbitOn", function(v) if v then S.followOn = false; if refresh.followOn then refresh.followOn() end end end)
	slider(t6, "Velocidade da órbita", "orbitSpeed", 0.5, 12, 0.5)
	slider(t6, "Distância da órbita", "orbitDist", 3, 40, 1, " st")
	toggle(t6, "Grudar nas costas", "followOn", function(v) if v then S.orbitOn = false; if refresh.orbitOn then refresh.orbitOn() end end end)
	slider(t6, "Distância ao seguir", "followDist", 2, 30, 1, " st")
	button(t6, "SENTAR", function() M.sit() end)
end

local function buildTorcida()
	local teamStatus
	local m1 = card("ENTRAR NUM TIME")
	picker(m1, "Torcida", "teamPick", M.teamNames)
	teamStatus = status(m1, "Time atual: " .. (LP.Team and LP.Team.Name or "nenhum"), 60)
	button(m1, "ENTRAR NO TIME", function()
		M.joinTeam(function(txtv) teamStatus.Text = txtv end)
	end)
	toggle(m1, "Forçar visual local (só pra mim)", "teamLocal")
	button(m1, "IR PRO SPAWN DO TIME", function() M.toSpawn() end)
	status(m1, "Usa o botão ou prompt que o próprio jogo oferece. Não dispara remotes por conta própria.", 48)

	local m2 = card("TORCIDAS NO SERVIDOR")
	local countLbl = status(m2, "Toque em atualizar", 60)
	button(m2, "ATUALIZAR CONTAGEM", function()
		local out = {}
		for _, t in ipairs(Teams:GetTeams()) do out[#out + 1] = t.Name .. ": " .. #t:GetPlayers() end
		countLbl.Text = #out > 0 and table.concat(out, "  |  ") or "Esse jogo não usa Teams"
	end)
end

local function buildMundo()
	local w1 = card("VISUAL")
	toggle(w1, "Fullbright", "fullbright")
	toggle(w1, "Sem neblina", "noFog")
	toggle(w1, "Hora fixa", "clockOn")
	slider(w1, "Hora do dia", "clockT", 0, 24, 0.5, "h")
	local w2 = card("SERVIDOR")
	button(w2, "REENTRAR NO SERVIDOR", function()
		pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LP) end)
	end)
	button(w2, "TROCAR DE SERVIDOR", function()
		task.spawn(function()
			pcall(function()
				local req = request or http_request or (syn and syn.request)
				if not req then return end
				local res = req({Url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100", Method = "GET"})
				local data = HttpService:JSONDecode(res.Body)
				for _, s in ipairs(data.data or {}) do
					if s.id ~= game.JobId and s.playing < s.maxPlayers then
						TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, LP)
						return
					end
				end
			end)
		end)
	end)
end

local function buildJogadores()
	local j1 = card("ALVO")
	picker(j1, "Jogador", "plPick", playerNames)
	button(j1, "TELEPORTAR ATÉ ELE", function()
		local p = Players:FindFirstChild(S.plPick)
		local tr = p and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
		local root = myRoot()
		if tr and root then root.CFrame = tr.CFrame * CFrame.new(0, 0, 3) end
	end)
	button(j1, "ESPECTAR", function() M.spectate(true) end)
	button(j1, "PARAR DE ESPECTAR", function() M.spectate(false) end)
end

local optStats
local function buildOtim()
	local o0 = card("STATUS")
	optStats = status(o0, "carregando...", 48)

	local o1 = card("PERFIS RÁPIDOS")
	button(o1, "MODO LEVE", function() M.preset("leve") end)
	button(o1, "MODO ULTRA LEVE", function() M.preset("ultra") end)
	button(o1, "RESTAURAR TUDO", function() M.preset("off") end)
	button(o1, "LIMPAR MEMÓRIA", function() M.cleanMem() end)

	local o2 = card("GRÁFICOS")
	toggle(o2, "Qualidade gráfica baixa", "opLow")
	slider(o2, "Nível de qualidade (1 = pior)", "opQuality", 1, 10, 1)
	toggle(o2, "Limitar FPS", "opFpsCap")
	slider(o2, "Limite de FPS", "opCap", 15, 240, 5, " fps")
	toggle(o2, "Tela preta (economiza GPU/bateria)", "opNo3D")

	local o3 = card("MAPA")
	toggle(o3, "Sem sombras", "opNoShadow")
	toggle(o3, "Sem efeitos (bloom, blur, névoa)", "opNoFx")
	toggle(o3, "Sem partículas/fogo/trails", "opNoPart")
	toggle(o3, "Desligar luzes", "opNoLight")
	toggle(o3, "Sem texturas/decals", "opNoTex")
	toggle(o3, "Materiais lisos (plástico)", "opPlastic")
	toggle(o3, "Água simples", "opWater")
	toggle(o3, "Sem grama", "opNoGrass")

	local o4 = card("JOGADORES E SOM")
	toggle(o4, "Esconder acessórios dos outros", "opHideAcc")
	toggle(o4, "Mutar sons do mapa", "opNoSound")

	local o5 = card("PAINEL")
	toggle(o5, "Animações do painel", "uiAnim")
	toggle(o5, "Mostrar FPS no menu", "uiFps")
	slider(o5, "Tamanho do painel", "uiScaleMul", 0.6, 1.4, 0.05, "x", function(v) winScale.Scale = uiScale * v end)
end

makeTab("COMBATE", 1, buildCombate)
makeTab("ESP", 2, buildEsp)
makeTab("TROLL", 3, buildTroll)
makeTab("TORCIDA", 4, buildTorcida)
makeTab("MUNDO", 5, buildMundo)
makeTab("JOGADORES", 6, buildJogadores)
makeTab("OTIMIZAÇÃO", 7, buildOtim)

track({Disconnect = function()
	alive = false
	for _, f in ipairs(M.cleanups) do pcall(f) end
end})

selectTab("COMBATE")

---------------------------------------------------------------- LOOPS
local hitAcc, fpsAcc, fpsFrames, lastFps, statAcc = 0, 0, 0, 60, 0
track(RunService.Heartbeat:Connect(function(dt)
	hitAcc = hitAcc + dt
	if hitAcc >= 0.1 then
		hitAcc = 0
		pcall(hitStep)
		if S.reachOn or reachH then pcall(reachStep) end
	end
	pcall(movStep, dt)
end))

RunService:BindToRenderStep("XitAim", Enum.RenderPriority.Camera.Value + 1, function(dt)
	local cam = getCam()
	if not cam then return end
	local now = os.clock()
	pcall(aimStep, cam, now)
	pcall(espStep, cam, now)
	if S.trigOn then pcall(trigStep, cam, now) end
	if S.clickOn then pcall(clickStep, now) end
	if S.freecam then pcall(M.fcStep, cam, dt) end
	if S.fovOn then
		P(cam, "FieldOfView", S.camFov); fovApplied = true
	elseif fovApplied then
		cam.FieldOfView = 70; fovApplied = false
		pcache[cam] = nil
	end
	if win.Visible then
		fpsAcc = fpsAcc + dt; fpsFrames = fpsFrames + 1
		if fpsAcc >= 0.5 then
			lastFps = math.floor(fpsFrames / fpsAcc + 0.5)
			fpsLbl.Text = "FPS " .. lastFps
			fpsAcc, fpsFrames = 0, 0
		end
		P(fpsLbl, "Visible", S.uiFps)
		if optStats and pages["OTIMIZAÇÃO"].Visible then
			statAcc = statAcc + dt
			if statAcc >= 1 then
				statAcc = 0
				optStats.Text = M.statsText(lastFps)
			end
		end
	end
end)

---------------------------------------------------------------- UNLOAD
local function unload()
	alive = false
	for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
	pcall(function() RunService:UnbindFromRenderStep("XitAim") end)
	for part in pairs(saved) do restore(part) end
	for _, a in pairs(adorns) do pcall(function() a:Destroy() end) end
	for _, hl in pairs(highlights) do pcall(function() hl:Destroy() end) end
	for _, o in pairs(espObjs) do
		for _, d in ipairs(o.list) do pcall(function() d:Remove() end) end
	end
	if aimLineD then pcall(function() aimLineD:Remove() end) end
	flyClean()
	pcall(function()
		local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
		if h then
			if wsApplied then h.WalkSpeed = origWS or 16 end
			if jpApplied then h.JumpPower = origJP or 50 end
			if TR.nfApplied then
				h:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
				h:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
			end
		end
		Workspace.Gravity = origGrav
		if fovApplied then getCam().FieldOfView = 70 end
	end)
	pcall(function() holder:Destroy() end)
	pcall(function() gui:Destroy() end)
	env.XitCleanup = nil
end
env.XitCleanup = unload
closeBtn.Activated:Connect(unload)
