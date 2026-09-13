local Players = game:GetService("Players")
local ServerStorage = game:GetService("ServerStorage")

local function buscarAutos()
	local autos = {}

	-- Buscar en ServerStorage
	for _, objeto in ipairs(ServerStorage:GetDescendants()) do
		if objeto:IsA("Model") and objeto:FindFirstChildWhichIsA("VehicleSeat", true) then
			autos[objeto.Name] = objeto
		end
	end

	-- Buscar también en Workspace
	for _, objeto in ipairs(workspace:GetDescendants()) do
		if objeto:IsA("Model") and objeto:FindFirstChildWhichIsA("VehicleSeat", true) then
			autos[objeto.Name] = objeto
		end
	end

	return autos
end

Players.PlayerAdded:Connect(function(player)

	player.CharacterAdded:Connect(function(character)

		local gui = Instance.new("ScreenGui")
		gui.Name = "MenuAutos"
		gui.ResetOnSpawn = false
		gui.Parent = player:WaitForChild("PlayerGui")

		local abrir = Instance.new("TextButton")
		abrir.Size = UDim2.new(0, 60, 0, 60)
		abrir.Position = UDim2.new(0, 10, 0.5, -30)
		abrir.Text = "🚗"
		abrir.TextScaled = true
		abrir.Parent = gui

		local menu = Instance.new("Frame")
		menu.Size = UDim2.new(0, 300, 0, 400)
		menu.Position = UDim2.new(0.5, -150, 0.5, -200)
		menu.Parent = gui

		local titulo = Instance.new("TextLabel")
		titulo.Size = UDim2.new(1, 0, 0, 45)
		titulo.Text = "🚗 BUSCADOR DE AUTOS"
		titulo.TextScaled = true
		titulo.Parent = menu

		local buscar = Instance.new("TextBox")
		buscar.Size = UDim2.new(1, -20, 0, 40)
		buscar.Position = UDim2.new(0, 10, 0, 55)
		buscar.PlaceholderText = "🔎 Buscar auto..."
		buscar.Text = ""
		buscar.TextScaled = true
		buscar.Parent = menu

		local lista = Instance.new("ScrollingFrame")
		lista.Size = UDim2.new(1, -20, 1, -110)
		lista.Position = UDim2.new(0, 10, 0, 105)
		lista.ScrollBarThickness = 8
		lista.Parent = menu

		local layout = Instance.new("UIListLayout")
		layout.Padding = UDim.new(0, 5)
		layout.Parent = lista

		local function actualizarLista()

			for _, objeto in ipairs(lista:GetChildren()) do
				if objeto:IsA("TextButton") then
					objeto:Destroy()
				end
			end

			local autos = buscarAutos()
			local textoBusqueda = string.lower(buscar.Text)

			for nombre, modelo in pairs(autos) do

				if textoBusqueda == ""
					or string.find(string.lower(nombre), textoBusqueda, 1, true) then

					local boton = Instance.new("TextButton")
					boton.Size = UDim2.new(1, -5, 0, 45)
					boton.Text = "🚘 " .. nombre
					boton.TextScaled = true
					boton.Parent = lista

					boton.Activated:Connect(function()

						local root =
							character:FindFirstChild("HumanoidRootPart")

						if not root then return end

						-- Buscar una copia del auto
						local copia = modelo:Clone()
						copia.Parent = workspace

						-- Aparecer delante del jugador
						copia:PivotTo(
							root.CFrame * CFrame.new(0, 3, -20)
						)

						-- Intentar sentar al jugador
						task.wait(0.5)

						local seat =
							copia:FindFirstChildWhichIsA(
								"VehicleSeat",
								true
							)

						local humanoid =
							character:FindFirstChildOfClass("Humanoid")

						if seat and humanoid then
							seat:Sit(humanoid)
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

		abrir.Activated:Connect(function()
			menu.Visible = not menu.Visible

			if menu.Visible then
				actualizarLista()
			end
		end)

		buscar:GetPropertyChangedSignal("Text"):Connect(
			actualizarLista
		)

		menu.Visible = false
	end)
end)
