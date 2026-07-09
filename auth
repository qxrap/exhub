local Threads = {}
local Connections = {}

local Services = {
	Players         = game:GetService("Players"),
	Lighting        = game:GetService("Lighting"),
	MaterialService = game:GetService("MaterialService"),
}

local XMin, XMax = -560, -240

local ClothingClasses = {
	"Shirt","Pants","ShirtGraphic",
	"Accessory","Hat","HairAccessory",
	"FaceAccessory","NeckAccessory","ShoulderAccessory",
	"FrontAccessory","BackAccessory","WaistAccessory",
}

local function AddThread(f)
	table.insert(Threads, task.spawn(f))
end

local function AddConnection(c)
	table.insert(Connections, c)
end

local function SafeDestroy(obj)
	if obj.Name == "Overhead" then return end
	pcall(function() obj:Destroy() end)
end

local function IsClothing(obj)
	for _, c in ipairs(ClothingClasses) do
		if obj:IsA(c) then return true end
	end
end

local function IsCharacterPart(obj)
	for _, plr in ipairs(Services.Players:GetPlayers()) do
		if plr.Character and obj:IsDescendantOf(plr.Character) then
			return true
		end
	end
end

-- True only for descendants of the LOCAL player's character.
-- Used to fully preserve the local player's avatar (clothing, accessories, etc.)
local function IsLocalCharacterPart(obj)
	local lp = Services.Players.LocalPlayer
	if lp and lp.Character and obj:IsDescendantOf(lp.Character) then
		return true
	end
end

local function IsOutOfRange(obj)
	if obj:IsA("BasePart") then
		local x = obj.Position.X
		return x < XMin or x > XMax
	end
end

-- Keeps base lasers from being stripped into plain blocks or deleted by the X-range cleanup.
local function IsBaseLaser(obj)
	local p = obj
	local seenPlot = false
	while p and p ~= workspace do
		local n = p.Name:lower()
		if n == "plots" then seenPlot = true end
		if n:find("laser", 1, true) or n:find("lasers", 1, true) then
			return true
		end
		p = p.Parent
	end
	return seenPlot and obj.Name:lower():find("laser", 1, true) ~= nil
end

-- ============================
-- TRANSPARENT BASES
-- ============================
local BASE_NAMES = {
	"baseplate", "spawnlocation", "spawn location", "spawn",
}

local function IsBase(obj)
	if not obj:IsA("BasePart") then return false end
	local nameLower = obj.Name:lower()
	for _, n in ipairs(BASE_NAMES) do
		if nameLower:find(n, 1, true) then return true end
	end
	return false
end

-- Returns true if obj is a descendant of a base part (e.g. a name label on a spawn)
local function IsInBase(obj)
	local p = obj.Parent
	while p and p ~= workspace do
		if IsBase(p) then return true end
		p = p.Parent
	end
	return false
end

local function MakeTransparent(obj)
	pcall(function()
		if IsBase(obj) and not IsCharacterPart(obj) then
			obj.Transparency = 1
			obj.CastShadow   = false
		end
	end)
end

local function ApplyTransparentBases()
	for _, obj in ipairs(workspace:GetDescendants()) do
		MakeTransparent(obj)
	end
end

-- ============================
-- STRIP OBJECT
-- ============================
local function StripObject(obj)
	if IsBaseLaser(obj) then return end
	pcall(function()
		if obj:IsA("Texture") or obj:IsA("Decal") or obj:IsA("SpecialMesh") then
			SafeDestroy(obj)
		elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
			or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
			pcall(function() obj.Enabled = false end)
			SafeDestroy(obj)
		elseif obj:IsA("SurfaceAppearance") then
			SafeDestroy(obj)
		elseif obj:IsA("BasePart") then
			obj.CastShadow      = false
			obj.Material        = Enum.Material.Plastic
			obj.MaterialVariant = ""
			obj.Reflectance     = 0
		end
	end)
end

-- ============================
-- CLEAN OBJECT
-- ============================
local function CleanObject(obj)
	if IsBaseLaser(obj) then return end
	pcall(function()
		if obj:IsA("SurfaceAppearance") then
			SafeDestroy(obj)

		elseif obj:IsA("Decal") or obj:IsA("Texture") then
			if not (obj.Name == "face" and obj.Parent and obj.Parent.Name == "Head") then
				SafeDestroy(obj)
			end

		elseif obj:IsA("SpecialMesh") then
			obj.TextureId = ""

		elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then
			SafeDestroy(obj)

		elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
			SafeDestroy(obj)

		elseif obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("Explosion") then
			SafeDestroy(obj)

		elseif obj:IsA("Animation") or obj:IsA("AnimationController") then
			SafeDestroy(obj)

		elseif obj:IsA("BasePart") then
			obj.CastShadow      = false
			obj.Material        = Enum.Material.Plastic
			obj.MaterialVariant = ""
			obj.Reflectance     = 0
		end
	end)
end

-- ============================
-- STOP ANIMATIONS
-- ============================
local function StopAnimations(animator)
	pcall(function()
		for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
			local isChar = false

			for _, plr in ipairs(Services.Players:GetPlayers()) do
				if plr.Character and animator:IsDescendantOf(plr.Character) then
					isChar = true
					break
				end
			end

			if not isChar then
				track:Stop()
			end
		end
	end)
end

-- ============================
-- OPTIMIZE CHARACTER
-- ============================
local function OptimizeCharacter(char)
	if not char then return end
	-- Skip local player's own character to preserve their avatar appearance
	local lp = Services.Players.LocalPlayer
	if lp then
		if lp.Character == char then return end
		-- also check by parent in case CharacterAdded fires before .Character is set
		if char.Parent == lp then return end
		for _, plr in ipairs(Services.Players:GetPlayers()) do
			if plr == lp and (plr.Character == char or char.Parent == plr) then return end
		end
	end

	task.spawn(function()
		task.wait(0.3)
		if not char or not char.Parent then return end

		local n = 0
		for _, obj in ipairs(char:GetDescendants()) do
			n = n + 1
			if IsClothing(obj) then
				SafeDestroy(obj)
			else
				CleanObject(obj)
			end
			-- Rejoin freeze fix: spread heavy character cleanup across frames.
			if n % 60 == 0 then task.wait() end
		end
	end)
end

-- ============================
-- GREY SKY (blank sky fast flag)
-- ============================
local function ApplyGreySky()
	pcall(function()
		-- Remove any existing Sky instances first
		for _, obj in ipairs(Services.Lighting:GetChildren()) do
			if obj:IsA("Sky") then
				obj:Destroy()
			end
		end

		-- Insert a blank Sky - renders as a flat grey void
		local sky        = Instance.new("Sky")
		sky.SkyboxBk     = ""
		sky.SkyboxDn     = ""
		sky.SkyboxFt     = ""
		sky.SkyboxLf     = ""
		sky.SkyboxRt     = ""
		sky.SkyboxUp     = ""
		sky.CelestialBodiesShown = false
		sky.Parent       = Services.Lighting
	end)
end

-- ============================
-- LIGHTING (remove effects + grey sky + dim slightly)
-- ============================
local function OptimizeLighting()
	local L = Services.Lighting

	L.GlobalShadows            = false
	L.FogEnd                   = 9e9
	L.FogStart                 = 9e9
	L.EnvironmentDiffuseScale  = 0
	L.EnvironmentSpecularScale = 0

	-- Dim the scene down just a touch
	L.Brightness               = 1.5   -- default is 2; slightly darker
	L.Ambient                  = Color3.fromRGB(60, 60, 60)  -- subtle dim ambient

	for _, v in ipairs(L:GetChildren()) do
		if v:IsA("BloomEffect") or v:IsA("BlurEffect")
		or v:IsA("ColorCorrectionEffect") or v:IsA("SunRaysEffect")
		or v:IsA("DepthOfFieldEffect") or v:IsA("Atmosphere")
		or v:IsA("Clouds") then
			v:Destroy()
		end
	end

	ApplyGreySky()
end

-- ============================
-- TERRAIN
-- ============================
local function ApplyTerrain()
	pcall(function()
		local T = workspace.Terrain
		T.Decoration        = false
		T.WaterWaveSize     = 0
		T.WaterWaveSpeed    = 0
		T.WaterReflectance  = 0
		T.WaterTransparency = 1
	end)
end

-- ============================
-- RENDER / PHYSICS SETTINGS
-- ============================
pcall(function()
	settings().Rendering.QualityLevel        = Enum.QualityLevel.Level01
	settings().Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level01

	workspace.Terrain.WaterWaveSize     = 0
	workspace.Terrain.WaterWaveSpeed    = 0
	workspace.Terrain.WaterReflectance  = 0
	workspace.Terrain.WaterTransparency = 1
	workspace.Terrain.Decoration        = false

	settings().Physics.AllowSleep = true
	settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Skip
end)

pcall(function() setfpscap(999) end)

-- ============================
-- MAIN INIT THREAD
-- ============================
AddThread(function()
	if not game:IsLoaded() then game.Loaded:Wait() end
	task.wait(2)

	OptimizeLighting()
	ApplyTerrain()

	local __initN = 0
	for _, obj in ipairs(workspace:GetDescendants()) do
		__initN = __initN + 1
		if __initN % 120 == 0 then task.wait() end
		if IsBase(obj) then
			MakeTransparent(obj)

		elseif IsLocalCharacterPart(obj) then
			-- skip entirely - preserve local player's avatar appearance

		elseif IsClothing(obj) then
			SafeDestroy(obj)

		elseif IsInBase(obj) then
			-- skip - preserve name labels and other children on bases

		elseif IsCharacterPart(obj) then
			-- skip

		elseif IsBaseLaser(obj) then
			-- skip - keep base lasers from becoming plain blocks

		elseif IsOutOfRange(obj) then
			SafeDestroy(obj)

		else
			CleanObject(obj)
			StripObject(obj)

			if obj:IsA("Animator") then
				StopAnimations(obj)
			end
		end
	end

	ApplyTransparentBases()
end)

-- ============================
-- LIGHTING EFFECT GUARD THREAD
-- ============================
AddThread(function()
	while true do
		task.wait(2)
		pcall(function()
			local L = Services.Lighting
			for _, obj in ipairs(L:GetChildren()) do
				if obj:IsA("Atmosphere") or obj:IsA("Clouds")
				or obj:IsA("PostEffect") then
					SafeDestroy(obj)
				end
			end
		end)
	end
end)

-- ============================
-- WORKSPACE WATCHER
-- ============================
-- Rejoin freeze fix: batch workspace additions instead of spawning one task per descendant.
local __MH_pendingClean = {}
local __MH_cleanQueued = false
local function __MH_processAddedObject(obj)
	if not obj or not obj.Parent then return end
	if IsBase(obj) then
		MakeTransparent(obj)
		return
	end

	if IsLocalCharacterPart(obj) then
		-- skip entirely - preserve local player's avatar appearance
	elseif IsClothing(obj) then
		SafeDestroy(obj)
	elseif IsInBase(obj) then
		-- skip - preserve name labels and other children on bases
	elseif IsCharacterPart(obj) then
		-- skip
	elseif IsBaseLaser(obj) then
		-- skip - keep base lasers from becoming plain blocks
	elseif IsOutOfRange(obj) then
		SafeDestroy(obj)
	else
		CleanObject(obj)
		StripObject(obj)
		if obj:IsA("Animator") then StopAnimations(obj) end
	end
end

AddConnection(workspace.DescendantAdded:Connect(function(obj)
	__MH_pendingClean[#__MH_pendingClean + 1] = obj
	if __MH_cleanQueued then return end
	__MH_cleanQueued = true
	task.defer(function()
		while #__MH_pendingClean > 0 do
			local batch = __MH_pendingClean
			__MH_pendingClean = {}
			for i, item in ipairs(batch) do
				pcall(__MH_processAddedObject, item)
				if i % 35 == 0 then task.wait() end
			end
			task.wait()
		end
		__MH_cleanQueued = false
	end)
end))

-- ============================
-- LIGHTING GUARD
-- ============================
AddConnection(Services.Lighting.DescendantAdded:Connect(function(obj)
	if obj:IsA("Atmosphere") or obj:IsA("Clouds")
	or obj:IsA("PostEffect") then
		SafeDestroy(obj)
	end
end))

-- ============================
-- MATERIAL SERVICE GUARD
-- ============================
AddConnection(Services.MaterialService.DescendantAdded:Connect(function(obj)
	SafeDestroy(obj)
end))

-- ============================
-- PLAYER HOOKS
-- ============================
for _, plr in ipairs(Services.Players:GetPlayers()) do
	OptimizeCharacter(plr.Character)
	AddConnection(plr.CharacterAdded:Connect(OptimizeCharacter))
end

AddConnection(Services.Players.PlayerAdded:Connect(function(plr)
	AddConnection(plr.CharacterAdded:Connect(OptimizeCharacter))
end))

-- ============================
-- GC THREAD
-- ============================
AddThread(function()
	while true do
		task.wait(15)
		pcall(function() collectgarbage("collect") end)
	end
end)

print("[Optimizer] Loaded ✓")
