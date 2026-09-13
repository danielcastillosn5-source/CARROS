local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local spawnEvent = ReplicatedStorage:WaitForChild("SpawnVehicle")

local gui = Instance.new("ScreenGui")
gui.Name = "VehicleMenu"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

--------------------------------------------------
-- BOTÓN ABRIR
--------------------------------------------------

local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0, 60, 0, 60)
openButton.Position = UDim2.new(0, 10, 0.5, -30)
openButton.Text = "🚗"
openButton.TextScaled = true
openButton.Parent = gui

--------------------------------------------------
-- MENÚ
--------------------------------------------------

local menu = Instance.new("Frame")
menu.Size = UDim2.new(0, 320, 0, 430)
menu.Position = UDim2.new(0.5, -160, 0.5, -215)
menu.Parent = gui

--------------------------------------------------
-- TÍTULO
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -45, 0, 45)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "🚗 BUSCADOR DE AUTOS"
title.TextScaled = true
title.Parent = menu

--------------------------------------------------
-- OCULTAR
--------------------------------------------------

local hideButton = Instance.new("TextButton")
hideButton.Size = UDim2.new(0, 45, 0, 45)
hideButton.Position = UDim2.new(1, -45, 0, 0)
hideButton.Text = "✕"
hideButton.TextScaled = true
hideButton.Parent = menu

hideButton.Activated:Connect(function()
	menu.Visible = false
end)

openButton.Activated:Connect(function()
	menu.Visible = true
end)

--------------------------------------------------
-- BUSCADOR
--------------------------------------------------

local search = Instance.new("TextBox")
search.Size = UDim2.new(1, -20, 0, 40)
search.Position = UDim2.new(0, 10, 0, 55)
search.PlaceholderText = "🔎 Buscar auto..."
search.Text = ""
search.TextScaled = true
search.Parent = menu

--------------------------------------------------
-- LISTA
--------------------------------------------------

local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -20, 0, 315)
list.Position = UDim2.new(0, 10, 0, 105)
list.ScrollBarThickness = 8
list.Parent = menu

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 5)
layout.Parent = list

--------------------------------------------------
-- OBTENER VEHÍCULOS
--------------------------------------------------

local function getVehicles()

	local vehicles = {}

	-- Busca modelos con VehicleSeat
	-- en todo Workspace.
	for _, obj in ipairs(workspace:GetDescendants()) do

		if obj:IsA("Model")
			and obj:FindFirstChildWhichIsA("VehicleSeat", true) then

			vehicles[obj.Name] = true
		end
	end

	return vehicles
end

--------------------------------------------------
-- ACTUALIZAR LISTA
--------------------------------------------------

local function updateList()

	for _, child in ipairs(list:GetChildren()) do
		if child:IsA("TextButton") then
			child:Destroy()
		end
	end

	local vehicles = getVehicles()
	local text = string.lower(search.Text)

	for name in pairs(vehicles) do

		if text == ""
			or string.find(
				string.lower(name),
				text,
				1,
				true
			) then

			local button = Instance.new("TextButton")

			button.Size = UDim2.new(1, -5, 0, 45)
			button.Text = "🚘 " .. name
			button.TextScaled = true
			button.Parent = list

			button.Activated:Connect(function()
				spawnEvent:FireServer(name)
			end)
		end
	end

	task.wait()

	list.CanvasSize = UDim2.new(
		0,
		0,
		0,
		layout.AbsoluteContentSize.Y + 10
	)
end

search:GetPropertyChangedSignal("Text"):Connect(updateList)

--------------------------------------------------
-- MENÚ MOVIBLE PC + CELULAR
--------------------------------------------------

local dragging = false
local dragStart
local startPos

local function startDrag(input)

	dragging = true
	dragStart = input.Position
	startPos = menu.Position

end

local function endDrag()

	dragging = false

end

local function moveDrag(input)

	if not dragging then
		return
	end

	local delta = input.Position - dragStart

	menu.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)

end

title.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		startDrag(input)
	end

end)

title.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		endDrag()
	end

end)

UserInputService.InputChanged:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then

		moveDrag(input)
	end

end)

--------------------------------------------------
-- INICIAR
--------------------------------------------------

menu.Visible = false
