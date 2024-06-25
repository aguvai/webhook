local httpService = game:GetService("HttpService")
local players = game:GetService("Players")
local datastoreService = game:GetService("DataStoreService")
local replicatedStorage = game:GetService("ReplicatedStorage")

local remotes = replicatedStorage.Remotes
local sendNotif = remotes.SendNotification

-- // SETUP DATA

local options = {
	preferences = {
		prefix = ";",
		merakiLogo = "https://cdn.discordapp.com/attachments/1201201471824330823/1242905241326981202/MerakiStampNoScroll.png?ex=664f8935&is=664e37b5&hm=45c0e91acc87abf28df08dbc675220f301c8cbc639a3eeaa544a113956b5d284&",
		defaultWebhook = "https://discord.com/api/webhooks/1242545091705573406/K71WXxqR9KU2Q-aEFT0cLdsAVUVIZFVoTKETN4bo861H8Fsm6K1tMsMFAcbYeDnDr1Ls"
	},
	
	control = {
		throwIfTooManyArguments = true
	}
}

local permissionLevels = {
	["Developer"] = {
		userIDs = {
			19233922,
			151160301,
			103729670
		},
		permissionLevel = 1
	},
	
	["Admin"] = {
		userIDs = {
			
		},
		permissionLevel = 2
	},
	
	["Moderator"] = {
		userIDs = {
			
		},
		permissionLevel = 3
	}
}

-- Reverse lookup table
local permissionLevelNames = {}
for role, info in pairs(permissionLevels) do
	permissionLevelNames[info.permissionLevel] = role
end

-- // HELPER FUNCTIONS

local function formatSeconds(seconds)
	local years = math.floor(seconds / (365.25 * 24 * 60 * 60))
	seconds = seconds - (years * 365.25 * 24 * 60 * 60)

	local months = math.floor(seconds / (30.44 * 24 * 60 * 60))
	seconds = seconds - (months * 30.44 * 24 * 60 * 60)

	local days = math.floor(seconds / (24 * 60 * 60))
	seconds = seconds - (days * 24 * 60 * 60)

	local hours = math.floor(seconds / (60 * 60))
	seconds = seconds - (hours * 60 * 60)

	local minutes = math.floor(seconds / 60)
	local seconds = seconds - (minutes * 60)

	local ageString = ""
	if years > 0 then
		ageString = ageString .. years .. " year" .. (years > 1 and "s" or "") .. ", "
	end
	if months > 0 then
		ageString = ageString .. months .. " month" .. (months > 1 and "s" or "") .. ", "
	end
	if days > 0 then
		ageString = ageString .. days .. " day" .. (days > 1 and "s" or "") .. ", "
	end
	if hours > 0 then
		ageString = ageString .. hours .. " hour" .. (hours > 1 and "s" or "") .. ", "
	end
	if minutes > 0 then
		ageString = ageString .. minutes .. " minute" .. (minutes > 1 and "s" or "") .. ", "
	end
	ageString = ageString .. seconds .. " second" .. (seconds > 1 and "s" or "")

	return ageString
end

function getTimeUTC()
	return (os.date("!%Y-%m-%dT%H:%M:%S.000Z", os.time()))
end

local thumbnailTypes = {
	Headshot = 1,
	Bust = 2,
	Full = 3
}
local function getPlayerThumbnail(playerID, type_)
	local s, url = pcall(function()
		if type_ == thumbnailTypes.Headshot then
			return httpService:JSONDecode(httpService:GetAsync("https://thumbnails.roproxy.com/v1/users/avatar-headshot?userIds=" .. playerID .. "&size=150x150&format=Png"))
		elseif type_ == thumbnailTypes.Bust then
			return httpService:JSONDecode(httpService:GetAsync("https://thumbnails.roproxy.com/v1/users/avatar-bust?userIds=" .. playerID .. "&size=150x150&format=Png"))
		elseif type_ == thumbnailTypes.Full then
			return httpService:JSONDecode(httpService:GetAsync("https://thumbnails.roproxy.com/v1/users/avatar?userIds=" .. playerID .. "&size=150x150&format=Png"))
		end
	end)
	
	if s and url then
		url = url.data[1].imageUrl
	else
		warn(string.format([[Roproxy API failed to grab player thumbnail "%s" in WebhookHandler]], type_) .. "\nTRACE: " .. debug.traceback())
		url = "https://t3.rbxcdn.com/9fc30fe577bf95e045c9a3d4abaca05d"
	end
	
	return url
end

local function getPlayerWebAddress(playerID, returnURLOnly)
	if returnURLOnly then
		return string.format("https://www.roblox.com/users/%s/profile", playerID)
	else
		local username = players:GetNameFromUserIdAsync(playerID)
		return string.format("[%s](https://www.roblox.com/users/%s/profile)", username, playerID)
	end
end

local function getProperCommandUsage(command):string
	local formattedArguments = "No arguments"
	if #command.args > 0 then
		formattedArguments = ""
		for i, v in ipairs(command.args) do
			local optionalText = ""
			if v.optional then optionalText = ", optional" end
			formattedArguments = formattedArguments .. "**[" .. i .. optionalText.. "]** " .. v.name .. " *(" .. v.type_ .. ")*"
			if i < #command.args then
				formattedArguments = formattedArguments .. " "
			end
		end
	end
	
	return string.format([[
		%s%s %s]], options.preferences.prefix, command.name, formattedArguments)
end


-- // WEBHOOK HANDLING

-- Embed handling

--[[
	Returns an embed containing information about the user who executed the command
--]]
local function getLogEmbed(userPacket)
	local embed = {
		author = {
			name =  string.format( [[%s used "%s"]], userPacket[1], userPacket[5]),
			url = getPlayerWebAddress(userPacket[2], true),
			icon_url = userPacket[4][1]
		},
		--fields = {
		--	{
		--		name = "Profile",
		--		value = getPlayerWebAddress(userPacket[2]),
		--		inline = true
		--	},

		--	{
		--		name = "Time",
		--		value = getTimeUTC(),
		--		inline = true
		--	}
		--},
		--thumbnail = { url = userPacket[4][2] },
		color = 14142879,
		footer = {
			text = "Meraki Interactive Webhook Application",
			icon_url = options.preferences.merakiLogo
		},
		timestamp = getTimeUTC()
	}
	return embed
end

--[[
	Returns an embed containing information on proper command usage and syntax
--]]
local function getCommandUsageEmbed(commandData)
	local embed = {
		title = "Proper command usage",
		description = getProperCommandUsage(commandData),
		color = 11130783
	}
	return embed
end

--[[
	Returns a general embed containing information returned by the command
--]]
local function getGeneralEmbed(content, title)
	title = title or ""
	local embed = {{
		title = tostring(title:upper()),
		description = tostring(content)
	}}
	return embed
end

local messageTypes = {
	Error = 1,
	Normal = 2,
	Info = 3,
	Success = 4
}

--[[
	Sends client sided notification to command user
--]]
local function notify(player, message, type_, location)
	type_ = type_ or messageTypes.Normal
	location = location or "upper"

	if type_ == messageTypes.Normal then
		sendNotif:FireClient(player, "normal", message, location)
	elseif type_ == messageTypes.Info then
		sendNotif:FireClient(player, "info", message, location)
	elseif type_ == messageTypes.Error then
		sendNotif:FireClient(player, "error", message, location)
	elseif type_ == messageTypes.Success then
		sendNotif:FireClient(player, "success", message, location)
	end
end

-- Main
--[[
	Sends data to Discord webhook
--]]
local function sendWebhook(playerPacket, data)	
	local success, result = pcall(function()
		httpService:PostAsync(options.preferences.defaultWebhook, httpService:JSONEncode(data))
		notify(playerPacket[6], "Successfully sent command result to webhook!")
	end)

	if not success then
		warn("Failed to post to webhook: " .. tostring(result) .. "\nTRACE: " .. debug.traceback())
		notify(playerPacket[6], "Failed to sent command result to webhook!", messageTypes.Error)
	end
end

--[[
	Formats command information and prepares it for Discord webhook
--]]
local function formatAndSend(userPacket, type_, embedsTable, commandData, content : string, title : string)
	content = content or ""
	
	-- Build content, if present
	if title and content ~= "" then
		content = ">>> ## ".. string.upper(title) .. "\n" .. content
	elseif content ~= "" then
		content = ">>> " .. content
	end
	
	-- Build embeds
	local embeds = {}
	
	if embedsTable then
		for _, v in ipairs(embedsTable) do
			table.insert(embeds, v)
		end
	end
	
	-- If it's an error message, also include info about how to properly use the attempted command
	if type_ == messageTypes.Error then
		table.insert(embeds, getCommandUsageEmbed(commandData))
	end
	
	table.insert(embeds, getLogEmbed(userPacket))
	
	-- Webhook data
	local username = "Battle Builders"
	local avatar_url = options.preferences.merakiLogo
	
	local data = {
		username = username,
		avatar_url = avatar_url,
		content = content,
		embeds = embeds
	}
	
	sendWebhook(userPacket, data)
end


-- // COMMAND FUNCTIONS

-- [[ FETCH-INFO ]]

--[[
	Retrieves a specific player's datastore profile
--]]
function getPlayerData(userId)
	local orderedDataStore = datastoreService:GetOrderedDataStore("STUDIODATA5/"..userId)
	local dataStore = datastoreService:GetDataStore("STUDIODATA5/"..userId)

	local pages = orderedDataStore:GetSortedAsync(false, 1)
	local data = pages:GetCurrentPage()

	if data[1] ~= nil then -- Check if you even received an entry
		return dataStore:GetAsync(data[1].key) -- Get first entry with first key
	end
	return nil
end

--[[
	Retrieves all non-tabular data fields within a player's datastore profile
--]]
local function getNonTabularFields(playerData)
	local fields = {}
	
	for name, value in pairs(playerData) do
		if type(value) ~= "table" then
			table.insert(fields, {
					name = tostring(name),
					value = tostring(value),
					inline = true
				})
		end
	end
	
	return fields
end

--[[
	Recursively unpacks and formats a specified tabular data field within a player's datastore profile
--]]
local function unpackTabularData(index, value, embedDescription, indentationLevel)
	local indentation = string.rep("--", indentationLevel)
	
	embedDescription = embedDescription .. string.format("\n**%s> %s**", indentation, index)

	for key, val in pairs(value) do
		if type(val) == "table" then
			embedDescription = unpackTabularData(key, val, embedDescription, indentationLevel + 1)
		else
			embedDescription = embedDescription .. string.format("\n%s> **%s:** %s", string.rep("--", indentationLevel + 1), key, tostring(val))
		end
	end

	return embedDescription
end

local needleTypes = {
	General = 1,
	Singular = 2,
	Tabular = 3
}

--[[
	Scalable embedding method
--]]
local function getDataEmbed(userID, embedDescription, playerData, needle, indexOfDataValue)
	local fieldsData
	
	if needle == needleTypes.General then
		fieldsData = getNonTabularFields(playerData)
	elseif needle == needleTypes.Singular then
		fieldsData = {{
				name = indexOfDataValue,
				value = tostring(playerData[indexOfDataValue]),
				inline = true
		}}
	elseif needle == needleTypes.Tabular then
		embedDescription = unpackTabularData(indexOfDataValue, playerData[indexOfDataValue], embedDescription, 0)
	end
	
	local embed = {
		author = {
			name = string.format([[Data fetch for %s (%s)]], players:GetNameFromUserIdAsync(userID), userID),
			icon_url = getPlayerThumbnail(userID, thumbnailTypes.Headshot)
		},
		thumbnail = { url = getPlayerThumbnail(userID, thumbnailTypes.Full) },
		description = embedDescription,
		fields = fieldsData
	}
	
	return embed
end

--[[
	Main command processing for fetch-info
--]]
local function fetchInfo(userPacket, userID, filter)
	filter = filter or "general"
	local playerData = getPlayerData(userID)
	
	print(playerData)
	
	if playerData then
		local embed = {}
		
		local embedDescription = string.format([[### %s Data Profile | Filter: *%s*]], getPlayerWebAddress(userID), filter)
		
		if filter == "general" then
			table.insert(embed, getDataEmbed(userID, embedDescription, playerData, needleTypes.General))
			formatAndSend( userPacket, messageTypes.Normal, embed )
		else 
			-- Data filter
			for index, value in pairs(playerData) do
				if string.lower(index) == filter then
					if type(value) == "table" then
						table.insert(embed, getDataEmbed(userID, embedDescription, playerData, needleTypes.Tabular, index))
					else
						table.insert(embed, getDataEmbed(userID, embedDescription, playerData, needleTypes.Singular, index))
					end
					
					formatAndSend( userPacket, messageTypes.Normal, embed )
					return
				end
			end
			print(getGeneralEmbed(("No data found for type: " .. filter), "fetch-info failed"))
			formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(("No data found for type: " .. filter), "fetch-info failed"))
		end
		
	else
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(string.format([[No data found for user: %s (%s) ]], getPlayerWebAddress(userID), userID), "fetch-info empty"))
	end
end


-- [[ GET-SERVER-INFO ]]

local serverCreationTime = os.time()
local function getServerInfo(userPacket, embedOnly)
	local placeID = game.PlaceId
	local gameID = game.GameId
	local jobID = game.JobId
	local currentTime = os.time()
	local ageSeconds = currentTime - serverCreationTime
	local serverAge = formatSeconds(ageSeconds)
	local serverVersion = game.PlaceVersion
	local activePlayers = #players:GetPlayers()
	local serverSize = players.MaxPlayers
	
	local content = string.format([[### Server info:
		**Place ID:** %s
		**Game ID:** %s
		**Job ID:** %s
		**Server age:** %s
		**Running on:** version %s 
		**Active players:** %s/%s 
		
		To join this server, open the console on your web browser (this is typically done by pressing F12). Then, copy and paste the following into your console:
		**Roblox.GameLauncher.joinGameInstance(%s, "%s")**
		]], placeID, gameID, jobID, serverAge, serverVersion, activePlayers, serverSize, placeID, jobID)
	
	local embed = getGeneralEmbed(content, "Server info")
	
	if embedOnly then
		return embed
	else
		formatAndSend(userPacket, messageTypes.Normal, embed)
	end
end

-- [[ KICK, BAN ]]

--[[
	Retrieves player from unique identifier
--]]
local function getPlayerFromIdentifier(identifier)
	local player
	local userID

	local function getPlayerByName(name)
		for _, v in ipairs(players:GetPlayers()) do
			if string.lower(string.sub(v.Name, 1, #name)) == string.lower(name) then
				player = v
				userID = v.UserId
				break
			end
		end
	end

	if tonumber(identifier) then
		local success, result = pcall(function()
			return players:GetPlayerByUserId(identifier)
		end)
		if success then
			player = result
			userID = identifier
		else
			player = getPlayerByName(identifier)
		end
	else
		getPlayerByName(identifier)
	end
	
	return player, userID
end

--[[
	Removes player from server
--]]
local function kick(userPacket, identifier, reason)
	local player, userID = getPlayerFromIdentifier(identifier)
	
	if player then
		player:Kick(reason)
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(string.format([[Successfully kicked user: %s (%s) from the server.]], getPlayerWebAddress(userID), userID), "Kicked user"))
	else
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(string.format([[Could not find user with filter: **"%s"**.]], identifier), "Kick failed"))
	end
end

--[[
	Bans player from server
--]]
local function ban(userPacket, identifier, reason, time)
	local player, userID = getPlayerFromIdentifier(identifier)
	
	if player then
		player:Kick(reason)
		
		-- TODO KODY: add datastore ban here
		
		
		
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(string.format([[Successfully banned user: %s (%s).]], getPlayerWebAddress(userID), userID), "Banned user"))
	else
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed(string.format([[Could not find user with filter: **"%s"**.]], identifier), "Ban failed"))
	end
end


-- // COMMAND INPUT HANDLING

--[[
	Argument validation
--]]
local function checkArguments(args, commandData, commandName, userPacket)
	local numProvidedArgs = #args - 1
	
	-- Check for absent required arguments
	local requiredArgsCount = 0
	for _, arg in ipairs(commandData.args) do
		if not arg.optional then
			requiredArgsCount = requiredArgsCount + 1
		end
	end

	if (numProvidedArgs) < requiredArgsCount then
		local missingArgs = "### Missing required arguments:\n"

		for i, v in ipairs(commandData.args) do
			if not v.optional and args[i + 1] == nil then
				missingArgs = missingArgs .. "**" .. tostring(i) .. ":** " .. v.name .. ", *" .. v.type_ .. "*\n"
			end
		end

		formatAndSend(userPacket, messageTypes.Error, getGeneralEmbed(missingArgs, ("ERROR | Missing required arguments for " .. commandName .. " command")), commandData)
		return false
	elseif (numProvidedArgs) > #commandData.args and options.control.throwIfTooManyArguments then
		formatAndSend(userPacket, messageTypes.Error, getGeneralEmbed(string.format([[Too many arguments provided in **"%s"** command. Received **%s** arguments, expected **%s**.]], commandName, numProvidedArgs, #commandData.args), ("ERROR | Too many arguments")), commandData)
		return false
	end

	-- Type checking for non-string arguments
	local content = "### Invalid data types received in the following arguments:\n"
	local dataTypeMismatch = false
	for i, v in ipairs(commandData.args) do
		local parallelArgument = args[i + 1]
		-- Skip type checking for missing optional arguments
		if parallelArgument == nil and v[3] then
			break
		end

		-- Type checking logic
		if v.type_ == "boolean" then
			if parallelArgument ~= "true" and parallelArgument ~= "false" then
				content = content .. "**" .. tostring(i) .. ":** " .. v.name .. ' - got "' .. parallelArgument .. '", expected *' .. v.type_ .. "*\n"
				dataTypeMismatch = true
			end
		elseif v.type_ == "number" then
			if not tonumber(parallelArgument) then
				content = content .. "**" .. tostring(i) .. ":** " .. v.name .. ' - got "' .. parallelArgument .. '", expected *' .. v.type_ .. "*\n"
				dataTypeMismatch = true
			end
		end
	end

	if dataTypeMismatch then
		formatAndSend(userPacket, messageTypes.Error, getGeneralEmbed(content, ("ERROR | Datatype mismatch in command: " .. commandName)), commandData)
		return false
	end

	return true
end

--[[
	Command library
--]]
local commands = {
	["fetch-info"] = {
		name = "fetch-info",
		func = fetchInfo,
		args = {
			{ name = "UserID", type_ = "number", optional = false },
			{ name = "Filter", type_ = "string", optional = true }
		},
		permissionLevel = permissionLevels.Developer.permissionLevel
	},
	
	["get-server-info"] = {
		name = "get-server-info",
		func = getServerInfo,
		args = {},
		permissionLevel = permissionLevels.Developer.permissionLevel
	},
	
	["kick"] = {
		name = "kick",
		func = kick,
		args = {
			{ name = "Identifier", type_ = "UserID or username", optional = false },
			{ name = "Reason", type_ = "string", optional = true }
		},
		permissionLevel = permissionLevels.Moderator.permissionLevel
	},
	
	["ban"] = {
		name = "ban",
		func = ban,
		args = {
			{ name = "Identifier", type_ = "UserID or username", optional = false },
			{ name = "Reason", type_ = "string", optional = true },
			{ name = "Time", type_ = "number", optional = true}
		},
		permissionLevel = permissionLevels.Admin.permissionLevel
	}
}

--[[
	Confirms a user's permission to run a given command
--]]
local function checkPermissions(userPacket, commandData, commandName)
	local permissionLevel = commandData.permissionLevel
	
	if permissionLevel < userPacket[3][2] then

		local content = string.format([[
			User does not have permission to run command "%s".
				
			Required permission level: **%s, (%s)**. %s's permission level: **%s, (%s)**.
			]], commandName, permissionLevelNames[permissionLevel], permissionLevel, userPacket[1], userPacket[3][1], userPacket[3][2])
		formatAndSend(userPacket, messageTypes.Error, getGeneralEmbed(content, "ERROR | Permission denied"), commandData)

		return false
	end
	
	return true
end

--[[
	Runs checks on the user and input structure, then runs the specified command
--]]
local function processCommand(message, userPacket)
	local args = string.split(string.sub(message, 2, #message), " ")
	local commandName = args[1]
	local commandData = commands[commandName]
	
	userPacket[5] = message

	if commandData then	
		-- Ensure user has permission to run this command
		if not checkPermissions(userPacket, commandData, commandName) then return end

		-- Validate arguments
		if not checkArguments(args, commandData, commandName, userPacket) then return end

		-- Run command
		local success, result = pcall(commandData.func, userPacket, unpack(args, 2))
		if not success then
			formatAndSend(userPacket, messageTypes.Error, getGeneralEmbed(('Error executing command "' .. commandName .. '": *' .. result .. "*"), "ERROR | Command execution failure"), commandData)
		end
	else
		formatAndSend(userPacket, messageTypes.Normal, getGeneralEmbed("Unrecognized command: " .. commandName))
	end
end


--[[
	Validates a user's permission levels.
	Returns a user packet structure if the user is authorized to use this system. Otherwise, returns false.
--]]
local function validateUser(player)
	--[[
		USER PACKET STRUCTURE
			[1] Name
			[2] UserId
			[3] Role data
				[1] Role name
				[2] Role permission level
			[4] Thumbnails
				[1] Avatar headshot image url
				[2] Avatar full body image URL
			[5] Last command used
			[6] Player object
	--]]
	
	local userID = player.UserId
	
	for role, info in pairs(permissionLevels) do
		for _, id in ipairs(info.userIDs) do
			if id == userID then
				-- This user is validated, build their packet
				local userPacket = {}
				
				table.insert(userPacket, player.Name)
				table.insert(userPacket, userID)
				table.insert(userPacket, {role, permissionLevels[role].permissionLevel})
				table.insert(userPacket, {getPlayerThumbnail(userID, thumbnailTypes.Headshot), getPlayerThumbnail(userID, thumbnailTypes.Full)})
				table.insert(userPacket, "")
				table.insert(userPacket, player)
				
				return userPacket
			end
		end
	end
	
	return false
end

players.PlayerAdded:Connect(function(player)
	local userPacket = validateUser(player)
	
	if userPacket ~= false then
		player.Chatted:Connect(function(message)
			local message = string.lower(message)
			local commandStart = string.sub(message, 1, 1) == options.preferences.prefix

			-- Check for prefix
			if commandStart then
				processCommand(message, userPacket)
			end
		end)
	end
end)
