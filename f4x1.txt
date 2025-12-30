--==================================================
-- F4X - OBJECT TOOL (EXECUTOR SAFE / GITHUB READY)
--==================================================

-- ===== SAFE START (CRITICAL) =====
repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local lp
repeat
	lp = Players.LocalPlayer
	task.wait()
until lp

local cam
repeat
	cam = workspace.CurrentCamera
	task.wait()
until cam

local mouse
repeat
	mouse = lp:GetMouse()
	task.wait()
until mouse

-- ===== CLEAN PREVIOUS GUI =====
local old = CoreGui:FindFirstChild("F4X_GUI")
if old then old:Destroy() end

--================ GUI =================
local gui = Instance.new("ScreenGui")
gui.Name = "F4X_GUI"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = CoreGui

local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 320, 0, 520)
panel.Position = UDim2.new(1, -340, 0.5, -260)
panel.BackgroundColor3 = Color3.fromRGB(18,18,20)
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true
panel.ZIndex = 10
Instance.new("UICorner", panel).CornerRadius = UDim.new(0,18)

local grad = Instance.new("UIGradient", panel)
grad.Color = ColorSequence.new{
	ColorSequenceKeypoint.new(0, Color3.fromRGB(40,40,44)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(14,14,16))
}

--================ TITLE =================
local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "F4X"
title.Font = Enum.Font.GothamBlack
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(245,245,245)

--================ INFO =================
local info = Instance.new("TextLabel", panel)
info.Position = UDim2.new(0,14,0,44)
info.Size = UDim2.new(1,-28,0,90)
info.BackgroundTransparency = 1
info.TextWrapped = true
info.TextXAlignment = Enum.TextXAlignment.Left
info.TextYAlignment = Enum.TextYAlignment.Top
info.Font = Enum.Font.Gotham
info.TextSize = 12
info.TextColor3 = Color3.fromRGB(190,190,190)
info.Text = "Click on a Part"

--================ HIGHLIGHT =================
local highlight = Instance.new("Highlight", gui)
highlight.FillColor = Color3.fromRGB(0,255,140)
highlight.OutlineColor = Color3.fromRGB(0,255,140)
highlight.FillTransparency = 0.85
highlight.OutlineTransparency = 0.15
highlight.Enabled = false

local selected

local function updateInfo()
	if not selected then
		info.Text = "Click on a Part"
		return
	end
	info.Text =
		"Name: "..selected.Name..
		"\nCollision: "..(selected.CanCollide and "ON" or "OFF")..
		"\nSize: "..math.floor(selected.Size.X)..", "..math.floor(selected.Size.Y)..", "..math.floor(selected.Size.Z)
end

local function setSelected(p)
	selected = p
	if p then
		highlight.Adornee = p
		highlight.Enabled = true
	end
	updateInfo()
end

--================ BUTTON CREATOR =================
local function button(txt, x, y, w, h, cb, getStateColor)
	local b = Instance.new("TextButton", panel)
	b.Size = UDim2.new(0, w, 0, h)
	b.Position = UDim2.new(0, x, 0, y)
	b.Text = txt
	b.Font = Enum.Font.GothamBold
	b.TextSize = 14
	b.TextColor3 = Color3.fromRGB(240,240,240)
	b.BackgroundColor3 = getStateColor and getStateColor() or Color3.fromRGB(55,55,60)
	b.BorderSizePixel = 0
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,12)

	local originalSize = b.Size

	b.MouseEnter:Connect(function()
		TweenService:Create(
			b,
			TweenInfo.new(0.15, Enum.EasingStyle.Quad),
			{Size = originalSize + UDim2.fromOffset(6,6)}
		):Play()
	end)

	b.MouseLeave:Connect(function()
		TweenService:Create(
			b,
			TweenInfo.new(0.15, Enum.EasingStyle.Quad),
			{Size = originalSize}
		):Play()
	end)

	b.MouseButton1Click:Connect(function()
		pcall(cb)
		if getStateColor then
			b.BackgroundColor3 = getStateColor()
		end
	end)

	return b
end

--================ MOVE =================
button("←", 40, 130, 60, 40, function() if selected then selected.CFrame *= CFrame.new(-1,0,0) end end)
button("→", 140,130,60,40,function() if selected then selected.CFrame *= CFrame.new(1,0,0) end end)
button("↑", 90, 100,60,40,function() if selected then selected.CFrame *= CFrame.new(0,0,-1) end end)
button("↓", 90, 180,60,40,function() if selected then selected.CFrame *= CFrame.new(0,0,1) end end)
button("Y +",220,100,70,40,function() if selected then selected.CFrame *= CFrame.new(0,1,0) end end)
button("Y -",220,180,70,40,function() if selected then selected.CFrame *= CFrame.new(0,-1,0) end end)

--================ SIZE =================
button("WIDTH +", 30,240,90,40,function()
	if selected then selected.Size += Vector3.new(1,0,1) updateInfo() end
end)

button("WIDTH -", 130,240,90,40,function()
	if selected then
		local s = selected.Size
		selected.Size = Vector3.new(math.max(1,s.X-1), s.Y, math.max(1,s.Z-1))
		updateInfo()
	end
end)

button("HEIGHT +", 30,290,90,40,function()
	if selected then selected.Size += Vector3.new(0,1,0) updateInfo() end
end)

button("HEIGHT -", 130,290,90,40,function()
	if selected then
		selected.Size = Vector3.new(
			selected.Size.X,
			math.max(0.5,selected.Size.Y-1),
			selected.Size.Z
		)
		updateInfo()
	end
end)

--================ CREATE / COLOR =================
button("NEW PART", 230,240,80,40,function()
	local char = lp.Character or lp.CharacterAdded:Wait()
	local hrp = char:WaitForChild("HumanoidRootPart")

	local p = Instance.new("Part")
	p.Anchored = true
	p.CanCollide = true
	p.Size = Vector3.new(12,1,12)
	p.Color = Color3.fromRGB(160,160,160)
	p.Position = hrp.Position + Vector3.new(0,4,0)
	p.Parent = workspace

	setSelected(p)
end)

button("COLOR", 230,290,80,40,function()
	if not selected then return end
	local colors = {
		Color3.fromRGB(255,80,80),
		Color3.fromRGB(80,255,120),
		Color3.fromRGB(80,160,255),
		Color3.fromRGB(255,255,120),
		Color3.fromRGB(255,120,255),
		Color3.fromRGB(240,240,240)
	}
	selected.Color = colors[math.random(#colors)]
end)

--================ COLLISION =================
button(
	"COLLISION",
	230,340,80,40,
	function()
		if selected then
			selected.CanCollide = not selected.CanCollide
			updateInfo()
		end
	end,
	function()
		if selected and selected.CanCollide then
			return Color3.fromRGB(70,160,90)
		else
			return Color3.fromRGB(160,70,70)
		end
	end
)

--================ CLICK =================
mouse.Button1Down:Connect(function()
	local m = UIS:GetMouseLocation()
	local p, s = panel.AbsolutePosition, panel.AbsoluteSize
	if m.X >= p.X and m.X <= p.X+s.X and m.Y >= p.Y and m.Y <= p.Y+s.Y then
		return
	end

	local ray = cam:ViewportPointToRay(m.X, m.Y)
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {lp.Character}
	params.IgnoreWater = true

	local res = workspace:Raycast(ray.Origin, ray.Direction * 5000, params)
	if res and res.Instance and res.Instance:IsA("BasePart") then
		setSelected(res.Instance)
	end
end)

--================ TIKTOK =================
local tt = Instance.new("TextButton", panel)
tt.Size = UDim2.new(1,-28,0,38)
tt.Position = UDim2.new(0,14,1,-40)
tt.Text = "@Xeno_Scripts"
tt.Font = Enum.Font.GothamBold
tt.TextSize = 15
tt.TextColor3 = Color3.fromRGB(220,220,220)
tt.BackgroundColor3 = Color3.fromRGB(26,26,28)
tt.BorderSizePixel = 0
Instance.new("UICorner", tt).CornerRadius = UDim.new(0,14)

local shine = Instance.new("UIGradient", tt)
shine.Color = ColorSequence.new{
	ColorSequenceKeypoint.new(0, Color3.fromRGB(70,70,70)),
	ColorSequenceKeypoint.new(0.5, Color3.fromRGB(160,160,160)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(70,70,70))
}
shine.Offset = Vector2.new(-1,0)

task.spawn(function()
	while tt.Parent do
		TweenService:Create(
			shine,
			TweenInfo.new(2,Enum.EasingStyle.Linear),
			{Offset = Vector2.new(1,0)}
		):Play()
		task.wait(2)
		shine.Offset = Vector2.new(-1,0)
	end
end)

tt.MouseButton1Click:Connect(function()
	if setclipboard then
		setclipboard("https://www.tiktok.com/@Xeno_Scripts")
	end
end)
