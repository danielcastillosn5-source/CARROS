local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

--------------------------------------------------
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "MenuAutos"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- BOTÓN PARA ABRIR
local abrir = Instance.new("TextButton")
abrir.Size = UDim2.new(0,60,0,60)
abrir.Position = UDim2.new(0,10,0.5,-30)
abrir.Text = "🚗"
abrir.TextScaled = true
abrir.Parent = gui

-- MENÚ
local menu = Instance.new("Frame")
menu.Size = UDim2.new(0,320,0,430)
menu.Position = UDim2.new(0.5,-160,0.5,-215)
menu.BackgroundTransparency = 0.1
menu.Parent = gui

-- TÍTULO
local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1,-50,0,45)
titulo.Text = "🚗 BUSCADOR DE AUTOS"
titulo.TextScaled = true
titulo.Parent = menu

-- BOTÓN OCULTAR
local ocultar = Instance.new("TextButton")
ocultar.Size = UDim2.new(0,45,0,45)
ocultar.Position = UDim2.new(1,-45,0,0)
ocultar.Text = "✕"
ocultar.TextScaled = true
ocultar.Parent = menu

-- BUSCADOR
local buscar = Instance.new("TextBox")
buscar.Size = UDim2.new(1,-20,0,40)
buscar.Position = UDim2.new(0,10,0,55)
buscar.PlaceholderText = "🔎 Buscar auto..."
buscar.Text = ""
buscar.TextScaled = true
buscar.Parent = menu

-- LISTA
local lista = Instance.new("ScrollingFrame")
lista.Size = UDim2.new(1,-20,0,315)
lista.Position = UDim2.new(0,10,0,105)
lista.ScrollBarThickness = 8
lista.Parent = menu

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0,5)
layout.Parent = lista

--------------------------------------------------
-- ABRIR / OCULTAR
--------------------------------------------------

abrir.Activated:Connect(function()
	menu.Visible = true
end)

ocultar.Activated:Connect(function()
	menu.Visible = false
end)

--------------------------------------------------
-- BUSCAR VEHÍCULOS
--------------------------------------------------

local function buscarAutos()

	local autos = {}

	for _, objeto in ipairs(workspace:GetDescendants()) do

		if objeto:IsA("Model") then

			local asiento =
				objeto:FindFirstChildWhichIsA("VehicleSeat", true)

			if asiento then
				autos[objeto.Name] = objeto
			end

		end
	end

	return autos
end

--------------------------------------------------
-- MOSTRAR AUTOS
--------------------------------------------------

local function actualizar()

	for _, objeto in ipairs(lista:GetChildren()) do
		if objeto:IsA("TextButton") then
			objeto:Destroy()
		end
	end

	local autos = buscarAutos()
	local texto = string.lower(buscar.Text)

	for nombre, modelo in pairs(autos) do

		if texto == ""
			or string.find(
				string.lower(nombre),
				texto,
				1,
				true
			) then

			local boton = Instance.new("TextButton")
			boton.Size = UDim2.new(1,-5,0,45)
			boton.Text = "🚘 " .. nombre
			boton.TextScaled = true
			boton.Parent = lista

			boton.Activated:Connect(function()

				local personaje = player.Character
				if not personaje then return end

				local root =
					personaje:FindFirstChild("HumanoidRootPart")

				if not root then return end

				-- COPIAR AUTO
				local copia = modelo:Clone()

				copia.Parent = workspace

				copia:PivotTo(
					root.CFrame *
					CFrame.new(0,3,-20)
				)

				-- BUSCAR ASIENTO
				task.wait(0.3)

				local asiento =
					copia:FindFirstChildWhichIsA(
						"VehicleSeat",
						true
					)

				local humanoid =
					personaje:FindFirstChildOfClass(
						"Humanoid"
					)

				if asiento and humanoid then
					asiento:Sit(humanoid)
				end

			end)
		end
	end

	task.wait()

	lista.CanvasSize = UDim2.new(
		0,
		0,
		0,
		layout.AbsoluteContentSize.Y + 10
	)
end

buscar:GetPropertyChangedSignal("Text"):Connect(actualizar)

--------------------------------------------------
-- MENÚ MOVIBLE
-- PC + CELULAR
--------------------------------------------------

local moviendo = false
local inicio
local posicionInicial

titulo.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		moviendo = true
		inicio = input.Position
		posicionInicial = menu.Position
	end
end)

titulo.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		moviendo = false
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if not moviendo then return end

	if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then

		local diferencia =
			input.Position - inicio

		menu.Position = UDim2.new(
			posicionInicial.X.Scale,
			posicionInicial.X.Offset + diferencia.X,

			posicionInicial.Y.Scale,
			posicionInicial.Y.Offset + diferencia.Y
		)
	end
end)

--------------------------------------------------
-- INICIO
--------------------------------------------------

menu.Visible = false
actualizar()
