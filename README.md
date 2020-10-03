# gun-
idkkk
--[[Variables]]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
local tool = script.Parent -- get tool
local player = game:GetService("Players").LocalPlayer -- get player
local mouse = player:GetMouse() -- getting the mouse
local sound = tool:WaitForChild("GunFire") -- gunshot sound
local torso = "" -- nothing
local reloading = false -- checks if reloading or not
local contextActionService = game:GetService("ContextActionService")
local bodyType = nil -- nil for now but will check whether player is R6 or R15
local difference = 0 -- space between head and mouse
local bullets = tool:WaitForChild("Bullets")
local replicatedStorage = game:GetService("ReplicatedStorage")
local events = replicatedStorage:WaitForChild("Events")
local functions = replicatedStorage:WaitForChild("Functions")
local akEvents = events:WaitForChild("AKEvents")
local akFunctions = functions:WaitForChild("AKFunctions")
local gunGui = tool:WaitForChild("AKGUI")
local reloadTime = "3"
local down = false
-- Remote Events --
local equipAnimation = akEvents:WaitForChild("EquipAnimation")
local headshot = akEvents:WaitForChild("Headshot")
local reloadTwo = akEvents:WaitForChild("Reload")
local shootEvent = akEvents:WaitForChild("ShootEvent")
local unequipAnimation = akEvents:WaitForChild("UnequipAnimation")
-- Remote Functions --
local checkBodyType = akFunctions:WaitForChild("CheckBodyType")
local fetchBulletsLeft = akFunctions:WaitForChild("FetchBulletsLeft")
--[[Find Body Type]]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
function findBodyType()
	bodyType = checkBodyType:InvokeServer(tool)
	print(bodyType)
end
--[[Reload Function]]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
function reload()
	if bullets.Value < 31 then
	reloading = true
	reloadTwo:FireServer(tool.Reload)

	player.PlayerGui:WaitForChild("AKGUI").Bullets.Text = "RELOADING"
	wait(reloadTime)
	bullets.Value = 31
	player.PlayerGui:WaitForChild("AKGUI").Bullets.Text = "AK-47 - " .. bullets.Value.. "/31"
	mouse.Icon = game.ReplicatedStorage.MouseValueDeagle.Value
	equipAnimation:FireServer(tool.Shoot)
	reloading = false
	end
end
--[[Main Firing Script]]---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

tool.Equipped:Connect(function(theMouse)

	gunGui:Clone().Parent = player.PlayerGui
	findBodyType()
	equipAnimation:FireServer(tool.Shoot)
	local img = game.StarterGui.ImageLoader.DeagleHair.Image
	mouse.Icon = img
	mouse.Button1Down:Connect(function()
down = true
	while down == true do
		wait(.1)

			print("working")
		if bullets.Value <= 0 or reloading == true then
			--Don't do anything
		else
			
			local head = game.Workspace[player.Name].Head.CFrame.lookVector
			local theMouse = CFrame.new(game.Workspace[player.Name].Head.Position,theMouse.Hit.p).lookVector
			difference = (head - theMouse)
			local ray = Ray.new(tool.Handle.CFrame.p, (player:GetMouse().Hit.p - tool.Handle.CFrame.p).unit*250)
			local part,position = game.Workspace:FindPartOnRay(ray,player.Character,false, true)
			
			--if difference.magnitude < 1.33 then
				sound:Play()
				bullets.Value = bullets.Value - 1
				shootEvent:FireServer(tool, position, part)
			
mouse.Button1Up:connect(function()
	down = false
end)
				end
		end
		--end
	end)

	

	
	local reloadMobileButton = contextActionService:BindAction("ReloadBtn",reload,true,"r")
	contextActionService:SetPosition("ReloadBtn",UDim2.new(0.72,-25,0.20,-25))
	contextActionService:SetImage("ReloadBtn","http://www.roblox.com/asset/?id=10952419")
end)
--[[Tool Unequipped Event]]--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
tool.Unequipped:Connect(function()
	mouse.Icon = ""
	unequipAnimation:FireServer(tool.Shoot)
	player.PlayerGui.AKGUI:Destroy()
	contextActionService:UnbindAction("ReloadBtn")
end)
