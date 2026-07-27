--[[
	CoolGui.lua
	LocalScript — colocar em StarterPlayer > StarterPlayerScripts

	GUI mobile-friendly com abas:
	- Aba 1: toggle de colisão do personagem
	- Aba 2: infos do jogador (idade da conta, presets de velocidade, shift lock)
	- Botão X fecha a GUI, seta ">" alterna entre as abas
	- Painel arrastável pelo título
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ===== Cores =====
local COLOR_BG        = Color3.fromRGB(18, 18, 20)
local COLOR_NEON      = Color3.fromRGB(255, 20, 40)
local COLOR_NEON_DIM  = Color3.fromRGB(120, 10, 20)
local COLOR_TEXT      = Color3.fromRGB(240, 240, 240)
local COLOR_BTN       = Color3.fromRGB(30, 30, 34)

-- ===== ScreenGui =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CoolGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = player:WaitForChild("PlayerGui")

-- ===== Painel principal =====
local panel = Instance.new("Frame")
panel.Name = "MainPanel"
panel.Size = UDim2.new(0, 280, 0, 220)
panel.Position = UDim2.new(0.5, -130, 0.15, 0)
panel.BackgroundColor3 = COLOR_BG
panel.BorderSizePixel = 0
panel.ClipsDescendants = true
panel.Parent = screenGui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 18)
panelCorner.Parent = panel

local panelStroke = Instance.new("UIStroke")
panelStroke.Color = COLOR_NEON
panelStroke.Thickness = 2
panelStroke.Transparency = 0.1
panelStroke.Parent = panel

-- ===== Barra superior (título + botões) =====
local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1, 0, 0, 44)
topBar.BackgroundTransparency = 1
topBar.Parent = panel

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -140, 1, 0)
title.Position = UDim2.new(0, 12, 0, 0)
title.BackgroundTransparency = 1
title.Text = "CoolGui"
title.Font = Enum.Font.GothamBlack
title.TextSize = 20
title.TextColor3 = COLOR_NEON
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = topBar

local titleStroke = Instance.new("UIStroke")
titleStroke.Color = COLOR_NEON
titleStroke.Thickness = 1
titleStroke.Transparency = 0.5
titleStroke.Parent = title

-- Botão de fechar (X)
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 40, 0, 40)
closeButton.Position = UDim2.new(1, -44, 0, 2)
closeButton.BackgroundColor3 = COLOR_BTN
closeButton.Text = "X"
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 18
closeButton.TextColor3 = COLOR_NEON
closeButton.AutoButtonColor = false
closeButton.Parent = topBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeButton

local closeStroke = Instance.new("UIStroke")
closeStroke.Color = COLOR_NEON
closeStroke.Thickness = 1
closeStroke.Parent = closeButton

closeButton.MouseButton1Click:Connect(function()
	screenGui:Destroy()
end)

-- Seta para próxima aba
local nextButton = Instance.new("TextButton")
nextButton.Name = "NextTabButton"
nextButton.Size = UDim2.new(0, 40, 0, 40)
nextButton.Position = UDim2.new(1, -88, 0, 2)
nextButton.BackgroundColor3 = COLOR_BTN
nextButton.Text = ">"
nextButton.Font = Enum.Font.GothamBold
nextButton.TextSize = 20
nextButton.TextColor3 = COLOR_NEON
nextButton.AutoButtonColor = false
nextButton.Parent = topBar

local nextCorner = Instance.new("UICorner")
nextCorner.CornerRadius = UDim.new(0, 8)
nextCorner.Parent = nextButton

local nextStroke = Instance.new("UIStroke")
nextStroke.Color = COLOR_NEON
nextStroke.Thickness = 1
nextStroke.Parent = nextButton

-- Botão de minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Name = "MinimizeButton"
minimizeButton.Size = UDim2.new(0, 40, 0, 40)
minimizeButton.Position = UDim2.new(1, -132, 0, 2)
minimizeButton.BackgroundColor3 = COLOR_BTN
minimizeButton.Text = "\226\128\148" -- —
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 20
minimizeButton.TextColor3 = COLOR_NEON
minimizeButton.AutoButtonColor = false
minimizeButton.Parent = topBar

local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 8)
minimizeCorner.Parent = minimizeButton

local minimizeStroke = Instance.new("UIStroke")
minimizeStroke.Color = COLOR_NEON
minimizeStroke.Thickness = 1
minimizeStroke.Parent = minimizeButton

-- ===== Botão flutuante (GUI minimizada) =====
local miniButton = Instance.new("ImageButton")
miniButton.Name = "MiniButton"
miniButton.Size = UDim2.new(0, 56, 0, 56)
miniButton.Position = UDim2.new(0.5, -28, 0.15, 0)
miniButton.BackgroundColor3 = COLOR_BG
miniButton.Image = "rbxassetid://84142615535715"
miniButton.ScaleType = Enum.ScaleType.Crop
miniButton.Visible = false
miniButton.Parent = screenGui

local miniCorner = Instance.new("UICorner")
miniCorner.CornerRadius = UDim.new(0, 16)
miniCorner.Parent = miniButton

local miniStroke = Instance.new("UIStroke")
miniStroke.Color = COLOR_NEON
miniStroke.Thickness = 2
miniStroke.Parent = miniButton

local function minimizeGui()
	panel.Visible = false
	miniButton.Visible = true
end

local function restoreGui()
	panel.Visible = true
	miniButton.Visible = false
end

minimizeButton.MouseButton1Click:Connect(minimizeGui)

-- arraste do botão flutuante, com detecção de "tap" (sem arrastar) pra restaurar a GUI
local miniDragging = false
local miniDragInputObj
local miniDragStart
local miniStartPos
local miniMoved = false

miniButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		miniDragging = true
		miniMoved = false
		miniDragStart = input.Position
		miniStartPos = miniButton.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				miniDragging = false
				if not miniMoved then
					restoreGui()
				end
			end
		end)
	end
end)

miniButton.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then
		miniDragInputObj = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == miniDragInputObj and miniDragging then
		local delta = input.Position - miniDragStart
		if delta.Magnitude > 6 then
			miniMoved = true
		end
		miniButton.Position = UDim2.new(
			miniStartPos.X.Scale, miniStartPos.X.Offset + delta.X,
			miniStartPos.Y.Scale, miniStartPos.Y.Offset + delta.Y
		)
	end
end)

-- ===== Arraste (drag) — usa o título como alça, funciona com toque e mouse =====
title.Active = true

local dragging = false
local dragInput
local dragStart
local startPos

local function updateDrag(input)
	local delta = input.Position - dragStart
	panel.Position = UDim2.new(
		startPos.X.Scale, startPos.X.Offset + delta.X,
		startPos.Y.Scale, startPos.Y.Offset + delta.Y
	)
end

title.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = panel.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

title.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		updateDrag(input)
	end
end)

-- ===== Fábrica de toggle reutilizável (trilho + bolinha) =====
local function createToggleSwitch(parent, yPos, initialState, onChanged)
	local track = Instance.new("Frame")
	track.Name = "ToggleTrack"
	track.Size = UDim2.new(0, 70, 0, 32)
	track.Position = UDim2.new(0.5, -35, 0, yPos)
	track.BackgroundColor3 = initialState and COLOR_NEON or COLOR_NEON_DIM
	track.BorderSizePixel = 0
	track.Parent = parent

	local trackCorner = Instance.new("UICorner")
	trackCorner.CornerRadius = UDim.new(1, 0)
	trackCorner.Parent = track

	local trackStroke = Instance.new("UIStroke")
	trackStroke.Color = COLOR_NEON
	trackStroke.Thickness = 1.5
	trackStroke.Parent = track

	local knob = Instance.new("Frame")
	knob.Name = "Knob"
	knob.Size = UDim2.new(0, 26, 0, 26)
	knob.Position = initialState and UDim2.new(1, -29, 0.5, -13) or UDim2.new(0, 3, 0.5, -13)
	knob.BackgroundColor3 = COLOR_TEXT
	knob.BorderSizePixel = 0
	knob.Parent = track

	local knobCorner = Instance.new("UICorner")
	knobCorner.CornerRadius = UDim.new(1, 0)
	knobCorner.Parent = knob

	-- área tocável maior que o visual (melhor para mobile)
	local hitArea = Instance.new("TextButton")
	hitArea.Name = "HitArea"
	hitArea.Text = ""
	hitArea.BackgroundTransparency = 1
	hitArea.Size = UDim2.new(1, 20, 1, 20)
	hitArea.Position = UDim2.new(0, -10, 0, -10)
	hitArea.Parent = track

	local state = initialState

	local function updateVisual()
		local knobGoal = state and UDim2.new(1, -29, 0.5, -13) or UDim2.new(0, 3, 0.5, -13)
		local colorGoal = state and COLOR_NEON or COLOR_NEON_DIM

		TweenService:Create(knob, TweenInfo.new(0.18, Enum.EasingStyle.Quad), {
			Position = knobGoal
		}):Play()

		TweenService:Create(track, TweenInfo.new(0.18, Enum.EasingStyle.Quad), {
			BackgroundColor3 = colorGoal
		}):Play()
	end

	hitArea.MouseButton1Click:Connect(function()
		state = not state
		updateVisual()
		onChanged(state)
	end)

	return track
end

-- ===== Container das abas =====
local pagesContainer = Instance.new("Frame")
pagesContainer.Name = "Pages"
pagesContainer.Size = UDim2.new(1, 0, 1, -44)
pagesContainer.Position = UDim2.new(0, 0, 0, 44)
pagesContainer.BackgroundTransparency = 1
pagesContainer.Parent = panel

-- ===== Aba 1: colisão =====
local page1 = Instance.new("Frame")
page1.Name = "Page1_Collision"
page1.Size = UDim2.new(1, 0, 1, 0)
page1.BackgroundTransparency = 1
page1.Visible = true
page1.Parent = pagesContainer

local toggleLabel = Instance.new("TextLabel")
toggleLabel.Name = "ToggleLabel"
toggleLabel.Size = UDim2.new(1, -40, 0, 20)
toggleLabel.Position = UDim2.new(0, 20, 0, 16)
toggleLabel.BackgroundTransparency = 1
toggleLabel.Text = "Colisão"
toggleLabel.Font = Enum.Font.GothamMedium
toggleLabel.TextSize = 16
toggleLabel.TextColor3 = COLOR_TEXT
toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
toggleLabel.Parent = page1

-- Lógica do toggle de colisão
local collisionOff = false
local cachedParts = {}
local heartbeatConn = nil
local descendantAddedConn = nil

local function rebuildCache(character)
	table.clear(cachedParts)
	for _, part in ipairs(character:GetDescendants()) do
		if part:IsA("BasePart") then
			table.insert(cachedParts, part)
		end
	end
end

local function applyOnce(enabled)
	for _, part in ipairs(cachedParts) do
		if part.Parent then
			part.CanCollide = enabled
		end
	end
end

local function startEnforcing()
	if heartbeatConn then return end
	heartbeatConn = RunService.Heartbeat:Connect(function()
		applyOnce(false)
	end)
end

local function stopEnforcing()
	if heartbeatConn then
		heartbeatConn:Disconnect()
		heartbeatConn = nil
	end
end

local function applyCollisionState()
	if collisionOff then
		startEnforcing()
	else
		stopEnforcing()
		applyOnce(true)
	end
end

createToggleSwitch(page1, 48, collisionOff, function(newState)
	collisionOff = newState
	applyCollisionState()
end)

-- ===== Aba 2: infos do jogador =====
local page2 = Instance.new("Frame")
page2.Name = "Page2_PlayerInfo"
page2.Size = UDim2.new(1, 0, 1, 0)
page2.BackgroundTransparency = 1
page2.Visible = false
page2.Parent = pagesContainer

local accountAgeLabel = Instance.new("TextLabel")
accountAgeLabel.Name = "AccountAgeLabel"
accountAgeLabel.Size = UDim2.new(1, -24, 0, 22)
accountAgeLabel.Position = UDim2.new(0, 12, 0, 6)
accountAgeLabel.BackgroundTransparency = 1
accountAgeLabel.Text = "Conta: " .. player.AccountAge .. " dias"
accountAgeLabel.Font = Enum.Font.GothamMedium
accountAgeLabel.TextSize = 15
accountAgeLabel.TextColor3 = COLOR_TEXT
accountAgeLabel.TextXAlignment = Enum.TextXAlignment.Left
accountAgeLabel.Parent = page2

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(1, -24, 0, 20)
speedLabel.Position = UDim2.new(0, 12, 0, 34)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade"
speedLabel.Font = Enum.Font.GothamMedium
speedLabel.TextSize = 14
speedLabel.TextColor3 = COLOR_TEXT
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = page2

local speedPresets = {
	{ name = "Normal", value = 16 },
	{ name = "Rápido", value = 28 },
	{ name = "Turbo",  value = 45 },
}

local function setWalkSpeed(value)
	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.WalkSpeed = value
	end
end

local buttonWidth = 72
local spacing = 8
for i, preset in ipairs(speedPresets) do
	local btn = Instance.new("TextButton")
	btn.Name = "Speed_" .. preset.name
	btn.Size = UDim2.new(0, buttonWidth, 0, 40)
	btn.Position = UDim2.new(0, 12 + (i - 1) * (buttonWidth + spacing), 0, 60)
	btn.BackgroundColor3 = COLOR_BTN
	btn.Text = preset.name
	btn.Font = Enum.Font.GothamMedium
	btn.TextSize = 13
	btn.TextColor3 = COLOR_TEXT
	btn.AutoButtonColor = false
	btn.Parent = page2

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = btn

	local stroke = Instance.new("UIStroke")
	stroke.Color = COLOR_NEON
	stroke.Thickness = 1
	stroke.Parent = btn

	btn.MouseButton1Click:Connect(function()
		setWalkSpeed(preset.value)
	end)
end

-- Shift lock: trava a câmera/mouse no centro e faz o personagem olhar
-- sempre na direção da câmera (ajuda quando a câmera solta em alguns jogos)
local divider = Instance.new("Frame")
divider.Name = "Divider"
divider.Size = UDim2.new(1, -24, 0, 1)
divider.Position = UDim2.new(0, 12, 0, 100)
divider.BackgroundColor3 = COLOR_NEON_DIM
divider.BorderSizePixel = 0
divider.Parent = page2

local shiftLockLabel = Instance.new("TextLabel")
shiftLockLabel.Name = "ShiftLockLabel"
shiftLockLabel.Size = UDim2.new(1, -40, 0, 20)
shiftLockLabel.Position = UDim2.new(0, 20, 0, 108)
shiftLockLabel.BackgroundTransparency = 1
shiftLockLabel.Text = "Shift Lock"
shiftLockLabel.Font = Enum.Font.GothamMedium
shiftLockLabel.TextSize = 16
shiftLockLabel.TextColor3 = COLOR_TEXT
shiftLockLabel.TextXAlignment = Enum.TextXAlignment.Left
shiftLockLabel.Parent = page2

local shiftLockOn = false
local shiftLockRenderConn = nil

-- Mira central que aparece na tela enquanto o shift lock está ativo
local crosshair = Instance.new("Frame")
crosshair.Name = "ShiftLockCrosshair"
crosshair.Size = UDim2.new(0, 26, 0, 26)
crosshair.AnchorPoint = Vector2.new(0.5, 0.5)
crosshair.Position = UDim2.new(0.5, 0, 0.5, 0)
crosshair.BackgroundTransparency = 1
crosshair.Visible = false
crosshair.Parent = screenGui

local crosshairCorner = Instance.new("UICorner")
crosshairCorner.CornerRadius = UDim.new(1, 0)
crosshairCorner.Parent = crosshair

local crosshairStroke = Instance.new("UIStroke")
crosshairStroke.Color = COLOR_NEON
crosshairStroke.Thickness = 2
crosshairStroke.Parent = crosshair

local function updateCharacterFacing()
	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")
	local root = character and character:FindFirstChild("HumanoidRootPart")
	if humanoid and root then
		local lookVector = camera.CFrame.LookVector
		local flatLook = Vector3.new(lookVector.X, 0, lookVector.Z)
		if flatLook.Magnitude > 0.01 then
			root.CFrame = CFrame.new(root.Position, root.Position + flatLook)
		end
	end
end

local function setShiftLock(enabled)
	shiftLockOn = enabled
	crosshair.Visible = enabled

	local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")

	if enabled then
		-- LockCenter só tem efeito com mouse (PC); no mobile é ignorado sem
		-- causar problema, o giro do personagem abaixo funciona nos dois casos
		UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
		if humanoid then
			humanoid.AutoRotate = false
		end
		if shiftLockRenderConn then shiftLockRenderConn:Disconnect() end
		shiftLockRenderConn = RunService.RenderStepped:Connect(updateCharacterFacing)
	else
		UserInputService.MouseBehavior = Enum.MouseBehavior.Default
		if humanoid then
			humanoid.AutoRotate = true
		end
		if shiftLockRenderConn then
			shiftLockRenderConn:Disconnect()
			shiftLockRenderConn = nil
		end
	end
end

-- reaplica o estado do shift lock quando o personagem renasce
player.CharacterAdded:Connect(function(character)
	local humanoid = character:WaitForChild("Humanoid", 5)
	if humanoid and shiftLockOn then
		humanoid.AutoRotate = false
	end
end)

createToggleSwitch(page2, 132, shiftLockOn, function(newState)
	setShiftLock(newState)
end)

-- ===== Reconstrói o cache de colisão quando o personagem (re)nasce =====
player.CharacterAdded:Connect(function(character)
	character:WaitForChild("HumanoidRootPart", 5)
	rebuildCache(character)
	applyCollisionState()

	if descendantAddedConn then
		descendantAddedConn:Disconnect()
	end

	descendantAddedConn = character.DescendantAdded:Connect(function(part)
		if part:IsA("BasePart") then
			table.insert(cachedParts, part)
			if collisionOff then
				part.CanCollide = false
			end
		end
	end)
end)

if player.Character then
	rebuildCache(player.Character)
end

-- ===== Alternância entre abas =====
local pages = { page1, page2 }
local currentPage = 1

nextButton.MouseButton1Click:Connect(function()
	pages[currentPage].Visible = false
	currentPage = (currentPage % #pages) + 1
	pages[currentPage].Visible = true
end)
