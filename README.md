-- 🚗✈️ AUTO VOLADOR + 🧍 VUELO PERSONAJE + 👻 NOCLIP
-- PC + CELULAR
-- MENÚ MOVIBLE + OCULTABLE

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local Player = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--------------------------------------------------
-- CONFIGURACIÓN ORIGINAL
--------------------------------------------------

local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

local FlyingPerson = true
local Speed = 70
local ColorHUD = Color3.fromRGB(0,255,255)

--------------------------------------------------
-- AUTO
--------------------------------------------------

local flyingCar = false
local vehicleSeat = nil
local vehicleModel = nil
local mainPart = nil

local bodyVelocityCar = nil
local bodyGyroCar = nil

local alturaObjetivo = nil

local velocidad = 120
local velocidadVertical = 70

--------------------------------------------------
-- NOCLIP
--------------------------------------------------

local Noclip = false

local function aplicarNoclip()

	if not Noclip then
		return
	end

	-- PERSONAJE
	local character = Player.Character

	if character then
		for _,obj in ipairs(character:GetDescendants()) do

			if obj:IsA("BasePart") then
				obj.CanCollide = false
			end

		end
	end

	-- AUTO
	if vehicleModel then

		for _,obj in ipairs(vehicleModel:GetDescendants()) do

			if obj:IsA("BasePart") then
				obj.CanCollide = false
			end

		end

	end
end

local function quitarNoclip()

	local character = Player.Character

	if character then

		for _,obj in ipairs(character:GetDescendants()) do

			if obj:IsA("BasePart") then
				obj.CanCollide = true
			end

		end

	end

	if vehicleModel then

		for _,obj in ipairs(vehicleModel:GetDescendants()) do

			if obj:IsA("BasePart") then
				obj.CanCollide = true
			end

		end

	end
end

--------------------------------------------------
-- ENCONTRAR AUTO
--------------------------------------------------

local function encontrarAuto()

	local character = Player.Character

	if not character then
		return false
	end

	local humanoid = character:FindFirstChildOfClass("Humanoid")

	if not humanoid then
		return false
	end

	local seat = humanoid.SeatPart

	if seat and seat:IsA("VehicleSeat") then

		vehicleSeat = seat
		vehicleModel = seat:FindFirstAncestorOfClass("Model")

		if vehicleModel then

			mainPart = vehicleModel.PrimaryPart

			if not mainPart then
				mainPart = seat
			end

			return true
		end
	end

	return false
end

--------------------------------------------------
-- INICIAR AUTO
--------------------------------------------------

local function iniciarVueloCar()

	if not encontrarAuto() then
		return false
	end

	if not mainPart then
		return false
	end

	alturaObjetivo = mainPart.Position.Y

	bodyVelocityCar = Instance.new("BodyVelocity")
	bodyVelocityCar.Name = "CarFlyVelocity"
	bodyVelocityCar.MaxForce = Vector3.new(
		1000000,
		1000000,
		1000000
	)
	bodyVelocityCar.P = 10000
	bodyVelocityCar.Velocity = Vector3.zero
	bodyVelocityCar.Parent = mainPart

	bodyGyroCar = Instance.new("BodyGyro")
	bodyGyroCar.Name = "CarFlyGyro"
	bodyGyroCar.MaxTorque = Vector3.new(
		1000000,
		1000000,
		1000000
	)
	bodyGyroCar.P = 10000
	bodyGyroCar.CFrame = mainPart.CFrame
	bodyGyroCar.Parent = mainPart

	return true
end

--------------------------------------------------
-- DETENER AUTO
--------------------------------------------------

local function detenerVueloCar()

	if bodyVelocityCar then
		bodyVelocityCar:Destroy()
		bodyVelocityCar = nil
	end

	if bodyGyroCar then
		bodyGyroCar:Destroy()
		bodyGyroCar = nil
	end

	alturaObjetivo = nil
end

--------------------------------------------------
-- GUI
--------------------------------------------------

local sg = Instance.new("ScreenGui")
sg.Name = "VueloJustinHUD"
sg.DisplayOrder = 999
sg.ResetOnSpawn = false
sg.Parent = Player:WaitForChild("PlayerGui")

--------------------------------------------------
-- MENÚ
--------------------------------------------------

local Menu = Instance.new("Frame")
Menu.Size = UDim2.new(0,320,0,330)
Menu.Position = UDim2.new(0.5,-160,0.5,-165)
Menu.BackgroundColor3 = Color3.new(0,0,0)
Menu.BackgroundTransparency = 0.15
Menu.BorderSizePixel = 2
Menu.BorderColor3 = ColorHUD
Menu.Parent = sg

--------------------------------------------------
-- TÍTULO
--------------------------------------------------

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,45)
Title.BackgroundTransparency = 1
Title.TextColor3 = ColorHUD
Title.TextSize = 20
Title.Text = "🚗✈️ CONTROL DE VUELO"
Title.Parent = Menu

--------------------------------------------------
-- VUELO PERSONAJE
--------------------------------------------------

local btnToggle = Instance.new("TextButton")
btnToggle.Size = UDim2.new(0,150,0,50)
btnToggle.Position = UDim2.new(0,10,0,55)
btnToggle.BackgroundColor3 = Color3.new(0,0,0)
btnToggle.TextColor3 = ColorHUD
btnToggle.TextSize = 18
btnToggle.Text = "VUELO PERSONA: ON"
btnToggle.BorderSizePixel = 2
btnToggle.BorderColor3 = ColorHUD
btnToggle.Parent = Menu

--------------------------------------------------
-- VELOCIDAD +
--------------------------------------------------

local btnUp = Instance.new("TextButton")
btnUp.Size = UDim2.new(0,50,0,50)
btnUp.Position = UDim2.new(0,220,0,55)
btnUp.BackgroundColor3 = Color3.new(0,0,0)
btnUp.TextColor3 = ColorHUD
btnUp.TextSize = 25
btnUp.Text = "+"
btnUp.BorderSizePixel = 2
btnUp.BorderColor3 = ColorHUD
btnUp.Parent = Menu

--------------------------------------------------
-- VELOCIDAD -
--------------------------------------------------

local btnDown = Instance.new("TextButton")
btnDown.Size = UDim2.new(0,50,0,50)
btnDown.Position = UDim2.new(0,165,0,55)
btnDown.BackgroundColor3 = Color3.new(0,0,0)
btnDown.TextColor3 = ColorHUD
btnDown.TextSize = 25
btnDown.Text = "-"
btnDown.BorderSizePixel = 2
btnDown.BorderColor3 = ColorHUD
btnDown.Parent = Menu

--------------------------------------------------
-- VUELO AUTO
--------------------------------------------------

local btnCar = Instance.new("TextButton")
btnCar.Size = UDim2.new(1,-20,0,50)
btnCar.Position = UDim2.new(0,10,0,120)
btnCar.BackgroundColor3 = Color3.new(0,0,0)
btnCar.TextColor3 = ColorHUD
btnCar.TextSize = 18
btnCar.Text = "🚗 AUTO VOLADOR: OFF"
btnCar.BorderSizePixel = 2
btnCar.BorderColor3 = ColorHUD
btnCar.Parent = Menu

--------------------------------------------------
-- NOCLIP
--------------------------------------------------

local btnNoclip = Instance.new("TextButton")
btnNoclip.Size = UDim2.new(1,-20,0,50)
btnNoclip.Position = UDim2.new(0,10,0,185)
btnNoclip.BackgroundColor3 = Color3.new(0,0,0)
btnNoclip.TextColor3 = ColorHUD
btnNoclip.TextSize = 18
btnNoclip.Text = "👻 NOCLIP: OFF"
btnNoclip.BorderSizePixel = 2
btnNoclip.BorderColor3 = ColorHUD
btnNoclip.Parent = Menu

--------------------------------------------------
-- INFO
--------------------------------------------------

local info = Instance.new("TextLabel")
info.Size = UDim2.new(1,-20,0,45)
info.Position = UDim2.new(0,10,0,250)
info.BackgroundTransparency = 1
info.TextColor3 = ColorHUD
info.TextSize = 14
info.Text = "WASD / JOYSTICK = MOVER"
info.Parent = Menu

--------------------------------------------------
-- BOTÓN OCULTAR
--------------------------------------------------

local btnHide = Instance.new("TextButton")
btnHide.Size = UDim2.new(0,75,0,40)
btnHide.Position = UDim2.new(0.5,-37,0.5,-215)
btnHide.BackgroundColor3 = Color3.new(0,0,0)
btnHide.TextColor3 = ColorHUD
btnHide.TextSize = 13
btnHide.Text = "OCULTAR"
btnHide.BorderSizePixel = 2
btnHide.BorderColor3 = ColorHUD
btnHide.Parent = sg

--------------------------------------------------
-- BOTÓN MOSTRAR
--------------------------------------------------

local btnShow = Instance.new("TextButton")
btnShow.Size = UDim2.new(0,80,0,40)
btnShow.Position = UDim2.new(0,10,0.8,0)
btnShow.BackgroundColor3 = Color3.new(0,0,0)
btnShow.TextColor3 = ColorHUD
btnShow.TextSize = 14
btnShow.Text = "MOSTRAR"
btnShow.BorderSizePixel = 2
btnShow.BorderColor3 = ColorHUD
btnShow.Visible = false
btnShow.Parent = sg

--------------------------------------------------
-- OCULTAR
--------------------------------------------------

btnHide.MouseButton1Click:Connect(function()

	Menu.Visible = false
	btnHide.Visible = false
	btnShow.Visible = true

end)

--------------------------------------------------
-- MOSTRAR
--------------------------------------------------

btnShow.MouseButton1Click:Connect(function()

	Menu.Visible = true
	btnHide.Visible = true
	btnShow.Visible = false

end)

--------------------------------------------------
-- HACER MENÚ MOVIBLE
--------------------------------------------------

local dragging = false
local dragStart
local startPos

Menu.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPos = Menu.Position

		input.Changed:Connect(function()

			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end

		end)

	end

end)

UserInputService.InputChanged:Connect(function(input)

	if dragging and (
		input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch
	) then

		local delta = input.Position - dragStart

		Menu.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)

	end

end)

--------------------------------------------------
-- VUELO PERSONA ON/OFF
--------------------------------------------------

btnToggle.MouseButton1Click:Connect(function()

	FlyingPerson = not FlyingPerson

	if FlyingPerson then

		btnToggle.Text = "VUELO PERSONA: ON"
		btnToggle.TextColor3 = ColorHUD

	else

		btnToggle.Text = "VUELO PERSONA: OFF"
		btnToggle.TextColor3 = Color3.new(1,0,0)

	end

end)

--------------------------------------------------
-- VELOCIDAD +
--------------------------------------------------

btnUp.MouseButton1Click:Connect(function()

	Speed = Speed + 10
	print("Velocidad: "..Speed)

end)

--------------------------------------------------
-- VELOCIDAD -
--------------------------------------------------

btnDown.MouseButton1Click:Connect(function()

	Speed = math.max(10,Speed - 10)
	print("Velocidad: "..Speed)

end)

--------------------------------------------------
-- AUTO ON/OFF
--------------------------------------------------

btnCar.MouseButton1Click:Connect(function()

	if not flyingCar then

		if iniciarVueloCar() then

			flyingCar = true
			btnCar.Text = "🚗 AUTO VOLADOR: ON"

		end

	else

		flyingCar = false
		btnCar.Text = "🚗 AUTO VOLADOR: OFF"

		detenerVueloCar()

	end

end)

--------------------------------------------------
-- NOCLIP ON/OFF
--------------------------------------------------

btnNoclip.MouseButton1Click:Connect(function()

	Noclip = not Noclip

	if Noclip then

		btnNoclip.Text = "👻 NOCLIP: ON"
		btnNoclip.TextColor3 = ColorHUD

		aplicarNoclip()

	else

		btnNoclip.Text = "👻 NOCLIP: OFF"
		btnNoclip.TextColor3 = Color3.new(1,0,0)

		quitarNoclip()

	end

end)

--------------------------------------------------
-- VUELO DEL PERSONAJE
--------------------------------------------------

local bv = Instance.new("BodyVelocity")
bv.Name = "PlayerFlyVelocity"
bv.MaxForce = Vector3.new(1000000,1000000,1000000)
bv.Parent = Root

local bg = Instance.new("BodyGyro")
bg.Name = "PlayerFlyGyro"
bg.MaxTorque = Vector3.new(1000000,1000000,1000000)
bg.Parent = Root

--------------------------------------------------
-- ACTUALIZACIÓN
--------------------------------------------------

RunService.RenderStepped:Connect(function()

	--------------------------------------------------
	-- NOCLIP
	--------------------------------------------------

	if Noclip then
		aplicarNoclip()
	end

	--------------------------------------------------
	-- VUELO PERSONAJE
	--------------------------------------------------

	if FlyingPerson then

		bg.CFrame = Camera.CFrame

		bv.Velocity =
			Camera.CFrame.LookVector * Speed

	else

		bv.Velocity = Vector3.zero

	end

	--------------------------------------------------
	-- AUTO
	--------------------------------------------------

	if flyingCar and bodyVelocityCar and bodyGyroCar and mainPart then

		local direccion = Vector3.zero

		--------------------------------------------------
		-- PC
		--------------------------------------------------

		if UserInputService.KeyboardEnabled then

			if UserInputService:IsKeyDown(Enum.KeyCode.W) then
				direccion += Camera.CFrame.LookVector
			end

			if UserInputService:IsKeyDown(Enum.KeyCode.S) then
				direccion -= Camera.CFrame.LookVector
			end

			if UserInputService:IsKeyDown(Enum.KeyCode.A) then
				direccion -= Camera.CFrame.RightVector
			end

			if UserInputService:IsKeyDown(Enum.KeyCode.D) then
				direccion += Camera.CFrame.RightVector
			end

		end

		--------------------------------------------------
		-- CELULAR
		--------------------------------------------------

		local character = Player.Character
		local humanoid = character and
			character:FindFirstChildOfClass("Humanoid")

		if humanoid and humanoid.MoveDirection.Magnitude > 0 then

			direccion = humanoid.MoveDirection

		end

		--------------------------------------------------
		-- VELOCIDAD
		--------------------------------------------------

		if direccion.Magnitude > 0 then
			direccion = direccion.Unit * velocidad
		end

		--------------------------------------------------
		-- MANTENER ALTURA
		--------------------------------------------------

		local diferenciaAltura =
			alturaObjetivo - mainPart.Position.Y

		local movimientoVertical =
			math.clamp(
				diferenciaAltura * 5,
				-velocidadVertical,
				velocidadVertical
			)

		bodyVelocityCar.Velocity =
			Vector3.new(
				direccion.X,
				movimientoVertical,
				direccion.Z
			)

		--------------------------------------------------
		-- GIRAR AUTO
		--------------------------------------------------

		if direccion.Magnitude > 0.1 then

			bodyGyroCar.CFrame = CFrame.lookAt(
				mainPart.Position,
				mainPart.Position + direccion
			)

		end

	end

end)

--------------------------------------------------
-- AUTO: SUBIR / BAJAR
--------------------------------------------------

local subir = Instance.new("TextButton")
subir.Size = UDim2.new(0,80,0,55)
subir.Position = UDim2.new(1,-90,0,250)
subir.Text = "⬆️ SUBIR"
subir.TextScaled = true
subir.Visible = false
subir.Parent = sg

local bajar = Instance.new("TextButton")
bajar.Size = UDim2.new(0,80,0,55)
bajar.Position = UDim2.new(1,-90,0,315)
bajar.Text = "⬇️ BAJAR"
bajar.TextScaled = true
bajar.Visible = false
bajar.Parent = sg

btnCar.MouseButton1Click:Connect(function()

	if flyingCar then

		subir.Visible = true
		bajar.Visible = true

	else

		subir.Visible = false
		bajar.Visible = false

	end

end)

subir.Activated:Connect(function()

	if flyingCar and mainPart and alturaObjetivo then

		alturaObjetivo = alturaObjetivo + 20

	end

end)

bajar.Activated:Connect(function()

	if flyingCar and mainPart and alturaObjetivo then

		alturaObjetivo = alturaObjetivo - 20

	end

end)

--------------------------------------------------
-- SI SE BAJA DEL AUTO
--------------------------------------------------

RunService.Heartbeat:Connect(function()

	if flyingCar then

		if not encontrarAuto() then

			flyingCar = false

			btnCar.Text = "🚗 AUTO VOLADOR: OFF"

			subir.Visible = false
			bajar.Visible = false

			detenerVueloCar()

		end

	end

end)

--------------------------------------------------
-- RESPALDAR PERSONAJE AL MORIR/REAPARECER
--------------------------------------------------

Player.CharacterAdded:Connect(function(char)

	Character = char
	Humanoid = char:WaitForChild("Humanoid")
	Root = char:WaitForChild("HumanoidRootPart")

	bv.Parent = Root
	bg.Parent = Root

end)
