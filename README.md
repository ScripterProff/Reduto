-- Dragon Hub - Arsenal Aimbot + ESP + FOV (Mobile)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações
local Settings = {
    Aimbot = false,
    ESP = false,
    FOV = 120,
    AimPart = "Head",
    TeamCheck = true
}

-- Criar GUI
local gui = Instance.new("ScreenGui")
gui.Name = "DragonHub"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 220, 0, 250)
main.Position = UDim2.new(0.5, -110, 0.5, -125)
main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true

local uicorner = Instance.new("UICorner", main)

local function createButton(text, yPos, callback)
	local btn = Instance.new("TextButton", main)
	btn.Size = UDim2.new(0.9, 0, 0, 30)
	btn.Position = UDim2.new(0.05, 0, 0, yPos)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.Text = text
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.MouseButton1Click:Connect(callback)
	Instance.new("UICorner", btn)
end

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "Dragon Hub"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 16

createButton("Toggle Aimbot", 40, function()
	Settings.Aimbot = not Settings.Aimbot
end)

createButton("Toggle ESP", 80, function()
	Settings.ESP = not Settings.ESP
end)

createButton("Minimizar", 170, function()
	main.Visible = false
end)

createButton("Fechar", 210, function()
	gui:Destroy()
end)

-- FOV Visual
local fov = Drawing.new("Circle")
fov.Visible = true
fov.Radius = Settings.FOV
fov.Color = Color3.fromRGB(255, 255, 0)
fov.Thickness = 2
fov.Filled = false

-- ESP
local espTable = {}

local function createESP(player)
	local box = Drawing.new("Square")
	box.Visible = false
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Thickness = 2
	box.Filled = false

	local line = Drawing.new("Line")
	line.Visible = false
	line.Color = Color3.fromRGB(255, 0, 0)
	line.Thickness = 1.5

	espTable[player] = {Box = box, Line = line}
end

local function removeESP(player)
	if espTable[player] then
		espTable[player].Box:Remove()
		espTable[player].Line:Remove()
		espTable[player] = nil
	end
end

-- Criar ESP para jogadores já presentes
for _, p in ipairs(Players:GetPlayers()) do
	if p ~= LocalPlayer then
		createESP(p)
	end
end

Players.PlayerAdded:Connect(function(p)
	if p ~= LocalPlayer then
		createESP(p)
	end
end)

Players.PlayerRemoving:Connect(removeESP)

-- Aimbot
local function getClosestTarget()
	local closest, shortest = nil, math.huge
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(Settings.AimPart) then
			if not Settings.TeamCheck or player.Team ~= LocalPlayer.Team then
				local part = player.Character[Settings.AimPart]
				local pos, onscreen = Camera:WorldToViewportPoint(part.Position)
				local dist = (Vector2.new(pos.X, pos.Y) - Camera.ViewportSize / 2).Magnitude
				local realDist = (Camera.CFrame.Position - part.Position).Magnitude
				if onscreen and dist < Settings.FOV and realDist < shortest then
					closest = part
					shortest = realDist
				end
			end
		end
	end
	return closest
end

-- Input para mobile
local aiming = false
UserInputService.TouchStarted:Connect(function()
	aiming = true
end)
UserInputService.TouchEnded:Connect(function()
	aiming = false
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	fov.Position = Camera.ViewportSize / 2

	-- Aimbot
	if Settings.Aimbot and aiming then
		local target = getClosestTarget()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end

	-- ESP
	for player, drawings in pairs(espTable) do
		local char = player.Character
		local box, line = drawings.Box, drawings.Line
		if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") and player.Team ~= LocalPlayer.Team and Settings.ESP then
			local hrp = char.HumanoidRootPart
			local head = char.Head
			local pos, vis = Camera:WorldToViewportPoint(hrp.Position)
			if vis then
				local sizeY = 3.5 * (1 / (hrp.Position - Camera.CFrame.Position).Magnitude) * 1000
				box.Size = Vector2.new(sizeY * 0.6, sizeY)
				box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
				box.Visible = true

				local headPos = Camera:WorldToViewportPoint(head.Position)
				line.From = Camera.ViewportSize / 2
				line.To = Vector2.new(headPos.X, headPos.Y)
				line.Visible = true
			else
				box.Visible = false
				line.Visible = false
			end
		else
			box.Visible = false
			line.Visible = false
		end
	end
end)
