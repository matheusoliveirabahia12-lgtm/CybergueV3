--==================================================
-- hog blue | DEV PANEL (JOGO PRÓPRIO / TESTE)
--==================================================

--================= SERVIÇOS =================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

--================= CONFIG ===================
local SETTINGS = {
	SpeedMult = 10,
	FlyForce = 60,
	AimbotRange = 120,
	XrayTransparency = 0.5
}

--================= CORES ====================
local YELLOW = Color3.fromRGB(255,210,0)
local BLACK  = Color3.fromRGB(18,18,18)
local DARK   = Color3.fromRGB(30,30,30)
local RED    = Color3.fromRGB(255,60,60)
local BLUE   = Color3.fromRGB(80,160,255)

--================= ESTADOS ==================
local TOGGLE = {
	ESP = false,
	Hitbox = false,
	Aimbot = false,
	Xray = false,
	ShowNames = false,
	ShowDistance = false,
	ShowTeam = false
}

--================= CHAR =====================
local function getChar()
	return player.Character or player.CharacterAdded:Wait()
end
local function getHumanoid()
	return getChar():WaitForChild("Humanoid")
end
local function getHRP(char)
	return char:FindFirstChild("HumanoidRootPart")
end

--================= GUI ======================
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "hog_blue_gui"
gui.ResetOnSpawn = false

-- 🔘 BOLINHA (ABRE/FECHA)
local ball = Instance.new("TextButton", gui)
ball.Size = UDim2.fromScale(0.09,0.09)
ball.Position = UDim2.fromScale(0.88,0.75)
ball.Text = ""
ball.BackgroundColor3 = YELLOW
ball.BorderSizePixel = 0
ball.Active = true
ball.Draggable = true
Instance.new("UICorner", ball).CornerRadius = UDim.new(1,0)

-- PAINEL
local panel = Instance.new("Frame", gui)
panel.Size = UDim2.fromScale(0.62,0.6)
panel.Position = UDim2.fromScale(0.19,0.2)
panel.BackgroundColor3 = BLACK
panel.Visible = false
panel.BorderSizePixel = 3
panel.BorderColor3 = YELLOW
Instance.new("UICorner", panel).CornerRadius = UDim.new(0.05,0)

-- TOPO
local top = Instance.new("Frame", panel)
top.Size = UDim2.fromScale(1,0.15)
top.BackgroundColor3 = DARK
top.BorderSizePixel = 0
local title = Instance.new("TextLabel", top)
title.Size = UDim2.fromScale(1,1)
title.Text = "hog blue"
title.TextColor3 = YELLOW
title.TextScaled = true
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold

-- LISTA
local list = Instance.new("Frame", panel)
list.Position = UDim2.fromScale(0,0.16)
list.Size = UDim2.fromScale(1,0.84)
list.BackgroundTransparency = 1
local layout = Instance.new("UIListLayout", list)
layout.Padding = UDim.new(0,8)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function Button(text, cb, tColor)
	local b = Instance.new("TextButton", list)
	b.Size = UDim2.fromScale(0.9,0.1)
	b.Text = text
	b.TextScaled = true
	b.Font = Enum.Font.Gotham
	b.BackgroundColor3 = DARK
	b.TextColor3 = tColor or YELLOW
	b.BorderSizePixel = 1
	b.BorderColor3 = YELLOW
	Instance.new("UICorner", b).CornerRadius = UDim.new(0.15,0)
	b.MouseButton1Click:Connect(cb)
end

local function ToggleBtn(text, key)
	Button(text.." [OFF]", function(btn)
		TOGGLE[key] = not TOGGLE[key]
		btn.Text = text.." ["..(TOGGLE[key] and "ON" or "OFF").."]"
	end)
end

--================= GAMEPLAY =================
Button("⚡ Speed 10x", function()
	getHumanoid().WalkSpeed = 16 * SETTINGS.SpeedMult
end)

Button("🕊️ Voar (clique sobe)", function()
	local hrp = getHRP(getChar())
	if not hrp then return end
	local bv = Instance.new("BodyVelocity")
	bv.Velocity = Vector3.new(0, SETTINGS.FlyForce, 0)
	bv.MaxForce = Vector3.new(0, math.huge, 0)
	bv.Parent = hrp
	game.Debris:AddItem(bv, 0.25)
end)

--================= TOGGLES ==================
ToggleBtn("ESP", "ESP")
ToggleBtn("Hitbox (DEV)", "Hitbox")
ToggleBtn("Aimbot (DEV)", "Aimbot")
ToggleBtn("Xray Base", "Xray")

ToggleBtn("Mostrar nomes", "ShowNames")
ToggleBtn("Mostrar distância", "ShowDistance")
ToggleBtn("Mostrar time", "ShowTeam")

Button("🧬 Brainrot Secreto", function()
	print("Linha Brainrot Secreto (DEV)")
end, RED)

--================= VISUAL DEV ===============
local espFolder = Instance.new("Folder", gui)
espFolder.Name = "ESP"

local function clearESP()
	espFolder:ClearAllChildren()
end

local function drawESP(target)
	if not TOGGLE.ESP then return end
	local hrp = getHRP(target.Character)
	if not hrp then return end

	local bb = Instance.new("BillboardGui", espFolder)
	bb.Adornee = hrp
	bb.Size = UDim2.fromScale(4,2)
	bb.AlwaysOnTop = true

	local lbl = Instance.new("TextLabel", bb)
	lbl.Size = UDim2.fromScale(1,1)
	lbl.BackgroundTransparency = 1
	lbl.TextScaled = true
	lbl.TextColor3 = RED
	lbl.Font = Enum.Font.GothamBold

	RunService.RenderStepped:Connect(function()
		if not bb.Parent then return end
		local txt = ""
		if TOGGLE.ShowNames then txt ..= target.Name.." " end
		if TOGGLE.ShowDistance then
			local d = (hrp.Position - camera.CFrame.Position).Magnitude
			txt ..= string.format("[%.0fm] ", d)
		end
		if TOGGLE.ShowTeam and target.Team then
			lbl.TextColor3 = target.TeamColor.Color
		else
			lbl.TextColor3 = RED
		end
		lbl.Text = txt
	end)
end

--================= HITBOX DEV ===============
local function applyHitbox(p)
	if not TOGGLE.Hitbox then return end
	local hrp = getHRP(p.Character)
	if hrp then
		hrp.Size = Vector3.new(6,6,6)
		hrp.Transparency = 0.5
	end
end

--================= AIMBOT DEV ===============
local function getClosest()
	local closest, dist = nil, SETTINGS.AimbotRange
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= player and p.Character and getHRP(p.Character) then
			local d = (getHRP(p.Character).Position - camera.CFrame.Position).Magnitude
			if d < dist then
				dist, closest = d, p
			end
		end
	end
	return closest
end

RunService.RenderStepped:Connect(function()
	if TOGGLE.Aimbot then
		local t = getClosest()
		if t and getHRP(t.Character) then
			camera.CFrame = CFrame.new(
				camera.CFrame.Position,
				getHRP(t.Character).Position
			)
		end
	end
end)

--================= XRAY =====================
local function setXray(on)
	for _,p in ipairs(workspace:GetDescendants()) do
		if p:IsA("BasePart") and p.Name == "BaseWall" then
			p.LocalTransparencyModifier = on and SETTINGS.XrayTransparency or 0
		end
	end
end

--================= LOOP =====================
RunService.Heartbeat:Connect(function()
	clearESP()
	if TOGGLE.ESP then
		for _,p in ipairs(Players:GetPlayers()) do
			if p ~= player and p.Character then
				drawESP(p)
				applyHitbox(p)
			end
		end
	end
	setXray(TOGGLE.Xray)
end)

--================= ABRIR/FECHAR ============
ball.MouseButton1Click:Connect(function()
	panel.Visible = not panel.Visible
end)
