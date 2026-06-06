local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")

local plr = Players.LocalPlayer

local cfg = {
	farmActive = false,
	farmCancelled = false,
	tpDelay = 2,
	runDelay = 4,
	antiAfk = false,
}

local ouro = 0
local fase = 1

local function root()
	local c = plr.Character
	return c and c:FindFirstChild("HumanoidRootPart")
end

local function hum()
	local c = plr.Character
	return c and c:FindFirstChild("Humanoid")
end

-- anti afk

local function startAntiAfk()
	task.spawn(function()
		while cfg.antiAfk do
			pcall(function()
				VirtualUser:CaptureController()
				VirtualUser:ClickButton1(Vector2.new())
			end)
			task.wait(55)
		end
	end)
end

-- core farm

local function doRun()
	local char = plr.Character
	if not char then
		plr.CharacterAdded:Wait()
		char = plr.Character
	end

	local r = char:FindFirstChild("HumanoidRootPart")
	if not r then return end

	local stages = workspace:FindFirstChild("BoatStages")
		and workspace.BoatStages:FindFirstChild("NormalStages")

	if not stages then
		Rayfield:Notify({ Title = "Error", Content = "NormalStages not found in workspace!", Duration = 4, Image = "x" })
		cfg.farmActive = false
		return
	end

	for i = 1, 10 do
		if not cfg.farmActive or cfg.farmCancelled then return end

		local stage = stages:FindFirstChild("CaveStage" .. i)
		if stage then
			local dp = stage:FindFirstChild("DarknessPart")
			if dp then
				r.CFrame = dp.CFrame

				local floor = Instance.new("Part")
				floor.Anchored = true
				floor.CanCollide = true
				floor.Size = Vector3.new(10, 1, 10)
				floor.Transparency = 1
				floor.Position = r.Position - Vector3.new(0, 3, 0)
				floor.Parent = char

				task.wait(cfg.tpDelay)
				floor:Destroy()

				fase = i
			end
		end
	end

	if not cfg.farmActive or cfg.farmCancelled then return end

	local theEnd = stages:FindFirstChild("TheEnd")
	if theEnd then
		local chest = theEnd:FindFirstChild("GoldenChest")
		if chest then
			local trigger = chest:FindFirstChild("Trigger")
			if trigger then
				r.CFrame = trigger.CFrame

				local waited = 0
				local got = false
				while waited < 10 and cfg.farmActive do
					task.wait(0.5)
					waited += 0.5
					if Lighting.ClockTime ~= 14 then
						got = true
						break
					end
				end

				if got then
					ouro += 100
					Rayfield:Notify({
						Title = "Gold Collected!",
						Content = "Total: " .. ouro .. " gold",
						Duration = 2,
						Image = "check",
					})
				end
			end
		end
	end

	if not cfg.farmActive or cfg.farmCancelled then return end

	local spawned = false
	local conn
	conn = plr.CharacterAdded:Connect(function()
		spawned = true
		conn:Disconnect()
	end)

	local t = 0
	while not spawned and cfg.farmActive and t < 30 do
		task.wait(1)
		t += 1
	end

	if spawned then task.wait(2) end
	task.wait(cfg.runDelay)
end

local function farmLoop()
	while cfg.farmActive and not cfg.farmCancelled do
		local ok, err = pcall(doRun)
		if not ok then
			Rayfield:Notify({ Title = "Farm Error", Content = tostring(err), Duration = 4, Image = "x" })
			task.wait(3)
		end
	end
end

-- window

local Window = Rayfield:CreateWindow({
	Name = "alsk._. on discord",
	LoadingTitle = "Loading",
	LoadingSubtitle = "please wait",
	Theme = "Default",
	DisableRayfieldPrompts = false,
	DisableBuildWarnings = false,
})

local FarmTab = Window:CreateTab("Farm", "shovel")

FarmTab:CreateSection("Auto Farm")

FarmTab:CreateToggle({
	Name = "Auto Farm",
	CurrentValue = false,
	Flag = "farmToggle",
	Callback = function(v)
		cfg.farmActive = v
		cfg.farmCancelled = not v
		if v then
			task.spawn(farmLoop)
			Rayfield:Notify({ Title = "Farm", Content = "Started!", Duration = 2, Image = "play" })
		else
			Rayfield:Notify({ Title = "Farm", Content = "Stopped.", Duration = 2, Image = "square" })
		end
	end,
})

FarmTab:CreateSlider({
	Name = "Teleport Delay (sec)",
	Range = {0.1, 5},
	Increment = 0.1,
	Suffix = "s",
	CurrentValue = 2,
	Flag = "tpDelay",
	Callback = function(v) cfg.tpDelay = v end,
})

FarmTab:CreateSlider({
	Name = "Delay Between Runs (sec)",
	Range = {1, 15},
	Increment = 0.5,
	Suffix = "s",
	CurrentValue = 4,
	Flag = "runDelay",
	Callback = function(v) cfg.runDelay = v end,
})

FarmTab:CreateSection("Misc")

FarmTab:CreateToggle({
	Name = "Anti AFK",
	CurrentValue = false,
	Flag = "antiAfk",
	Callback = function(v)
		cfg.antiAfk = v
		if v then startAntiAfk() end
		Rayfield:Notify({
			Title = "Anti AFK",
			Content = v and "Enabled!" or "Disabled.",
			Duration = 2,
			Image = v and "check" or "x",
		})
	end,
})

Rayfield:Notify({
	Title = "alsk._. on discord",
	Content = "alsk._. on discord",
	Duration = 4,
	Image = "heart",
})
