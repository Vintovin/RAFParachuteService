# Parachute Service Documentation
## **NOTE**
THIS SERVICE IS NOT PUBLICALLY AVALIABLE.
IF YOU WOULD LIKE ACCESS TO THIS, PLEASE DM ME ON DISCORD @bimbles_dev


## **Introduction**

This service is a centralised service to be used within BAF and RAF games for the purpose of parachute deployment of personnel and cargo.

The service in it's current state will only handle cargo drops, however, a soft deadline for 27/08/2023 has been set for the implementation of personnel drops under a static line system.

There is further plans for this system to be fitted with the ability to enable ripcord parachutes.

The cargo element of the system will be utilised as a part of the loadmaster system used by No.2 Group RAF.

## Pre-requisites
As of 23/08/2023:

 - Roblox LuaU
 - ReplicatedStorage
	 - Events
		 - ParachuteService
			 - NewChute:BinableEvent
			 - OpenChute:BindableEvent
	-	Components
		-	ParachuteService
			-	Chute:ParachuteCanopyPart
- ParachutePalletPart

### Roblox LuaU
This is the Roblox game environment, it's a given that this is required and I don't feel I need to explain this.

### Replicated Storage
This dictates all the required events and parts for the service.

#### BindableEvents
These are Server -> Server Event instances allowing server scripts to communicate with each other, upon the implementation of personnel parachutes, the use of Remote Events for Client -> Server and Server -> Client communications will be implemented

#### ParachuteCanopyPart
This is the canopy part that is pre-loaded with the attachment points and linearVelocity instance that allows the parachute to operate. These are all named accordingly and are REQUIRED

#### ParachutePalletPart
This is the part in which the parachute will attached to. There is no need for specific names , however, this is the instance that is passed and requires an attachment named "attachment" in order for the system to work. This attachment should be placed at the central to the pallet, and near the top. This is where the rope constraints will be attached to. 

## Installation
Upon importing this service into your game, it should be located within the serverScriptService, I would not recommend putting this anywhere exposed to the client as it can be abused to manipulate active parachutes, or to break the service itself.

Upon installing, the service should be initialised, using the :Init() method.

### Example
```lua
local ServerScriptService = game:GetService("ServerScriptService")
local Services = ServerScriptService.Services
local ParachuteService = require(Services.ParachuteService)

ParachuteService:Init()
```

No functions need to be passed, however, you may want to listen for the callback of `true` from the service to verify that it is operational, if there are any errors, it will return `nil`

### Error check example

```lua
local res = ParachuteService:Init()
if res then
	print("[ParachuteService] - Initialised Successfully")
else
	warn("[ParachuteService] - Error Initiliasing")
end
```
## References

### ParachuteModule.Defaults
The Defaults datatype is split into 2 identical sub-structures


| Name | DataType | Description |
|--|--|--|
| FinalVelocity | Number | This is the final velocity of the parachute, and should be negative |
| BeginningForce| Number | This is the initial force given to LinearVelocity |
| ForceMax| Number | This is the maximum force given to LinearVelocity |
| ForceInciment| Number | This is the increment of the force given to LinearVelocity |
| Interval| Number | This is the time between increments |
| rope| table| This is the table holding parachute cord values |
| rope.Thickness| Number | This is the rope thickness |
| rope.Color| BrickColor| This is the rope colour |
| rope.Visible | Boolean | This is the rope's visible property |
| rope.Length | Number | This is the rope's Length property |


### ParachuteModule.Chutes
The Chute Reference for each parachute instance


| Name | DataType | Description |
|--|--|--|
| Type| String | This is the chute type |
| Main| Instance | This is the main part of the parachute, where the parachute will connect |
| Chute| Instance| This is the parachute canopy part |
| Ropes| table| This is a table of all rope constraints |
| Constraints| ParachuteModule.Defaults| These are the constraints to be used during operation|

## API

## ParachuteService:Init() -> {success:bool}
This is the ParachuteService's Initialisation function, it will return a Boolean value of `true` if it has been successfully initialised or `nil` if there was an error while initialising.


### Example Code:
```lua
local ServerScriptService = game:GetService("ServerScriptService")
local Services = ServerScriptService.Services
local ParachuteService = require(Services.ParachuteService)

local res = ParachuteService:Init()
if res then
	print("[ParachuteService] - Initialised Successfully")
else
	warn("[ParachuteService] - Error Initiliasing")
end
```

## ParachuteService:NewChute(Type:String,Main:Instance) -> {}
This method will create a new parachute reference and setup the ingame instances for the parachute.

| Input/Output| DataType | Description |
|--|--|--|
| Type| String | This is the chute type |
| Main| Instance | This is the main part of the parachute, where the parachute will connect |

### Example Code:
The code below will fire the NewChute event from the server, which will then fire ParachuteService:NewChute
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Components = ReplicatedStorage.Components.ParachuteService
local Events = ReplicatedStorage.Events.ParachuteService

Events.NewChute:Fire("Cargo",script.Parent.DROP)
```


## ParachuteService:OpenChute(Type:String,Main:Instance) -> {}
This method will open the parachute that is referenced.

| Input/Output| DataType | Description |
|--|--|--|
| Type| String | This is the chute type |
| Main| Instance | This is the main part of the parachute, where the parachute will connect |

### Example Code:
The code below will fire the OpenChute event from the server, which will then fire ParachuteService:OpenChute
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Components = ReplicatedStorage.Components.ParachuteService
local Events = ReplicatedStorage.Events.ParachuteService

Events.OpenChute:Fire("Cargo",script.Parent.DROP)
```


## ParachuteService:KillChute(chuteRef:Table) -> {}

| Input/Output| DataType | Description |
|--|--|--|
| chuteRef| Table| This is the reference table for the Parachute being cut |

### Example Code:
The code below is an extract from the OpenChute method, where the parachute will be cut when the main part hits the ground.
```lua
local GroundHitListener = chuteRef.Main.Touched:Connect(function(Hit)
		if Hit and Hit.Parent then
			if (Hit.Name ~= chuteRef.Main.Name or Hit.Parent.Name == chuteRef.Main.Parent.Name) and Hit ~= game.Workspace.Terrain then
				if #Hit:GetTouchingParts() > 0 then
					ParachuteModule:KillChute(chuteRef)
				end
			elseif Hit == game.Workspace.Terrain then
				ParachuteModule:KillChute(chuteRef)
			end
		end
	end)
```
