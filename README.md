script_name("GameFixer")
script_author("sagazx wg")
script_version("1.0")
script_description("Adaptado com funcionalidades semelhantes à versão para PC, embora nem todas estejam incluídas. Pode conter erros, pois é a primeira versão..")

local imgui = require("mimgui")
local addons = require("ADDONS")
local sf = require('sampfuncs')
local sampEvents = require('samp.events')
local check_sampev, sampev = pcall(require, "samp.events")
local widgets = require("widgets")
local faicons = require("fAwesome6_solid")
local inicfg = require("inicfg")
local ffi = require("ffi")
local gta = ffi.load("GTASA")
local hook = require('monethook')
local SaMemory = require("SAMemory")
SaMemory.require("CCamera")
local Camera = SaMemory.camera

---------------------- [ cfg ]----------------------
local configFile  = "GameFixer.ini"
local configData = inicfg.load(inicfg.load({
	main = {
		sensitivity = 100,
		croshair = false;
		hitmarker = false,
		hitefects = false,
		Fov = 70,
		miraposx = 0.5185,
		miraposy = 0.438,
		auto_miraposx = 0.5185,
    	auto_miraposy = 0.438,
		renderDistance = 900,
		aspectratio = 1,
		auto_crosshair = true 
	},
	settings = {
		theme = 0,
		imguiItemsScale = 0.84
	}, 
	colors = {
		hit1r = 1,
		hit1g = 1,
		hit1b = 1,
		hit1a = 1,
		hit2r = 1,
		hit2g = 1,
		hit2b = 1,
		hit2a = 1
	},
	Weather_time = {
		Weather = -1,
        TimeHours = -1,
        TimeMinutes = -1
	},
	commands = {
		openmenu = "/gmenu",
		settime = "/st",
		setweather = "/sw",
		resettime = "/rt",
		resetweather = "/rw",
		clearchat = "/cc",
		togglechat = "/tc",
		reconect = "/rec",
		ugenrl = "/ugenrl",
		uds = "/uds",
		uss = "/uss",
		ums = "/ums",
		urs = "/urs",
		umps = "/umps",
		usmg = "/usmg",
		uaks = "/uaks",
		usps = "/usps",
		u9ms = "/u9ms",
		uedc = "/uedc",
		uhits = "/uhits",
		upns = "/upns"
	},
	ugenrl_main = {
        enable = true,
		autonosounds = true,
    	hit = true,	
        pain = true,
        weapon = true,
		enemyWeapon = true,
        selectedHit = 0,
		selectedPain = 0,
		selectedDK = 0,
		selectedM4 = 0,
		selectedShotgun = 0,
		selectedRifle = 0,
		selectedAK = 0,
		selectedMP5 = 0,
		selectedSniper = 0,
		selectedEdc = 0,
		selectedGun = 0,
		selectedSmg = 0
    },
	ugenrl_volume = {
		hit = 0.30,
		pain = 0.30,
		weapon = 0.30
	},
	ugenrl_sounds = {
	  hitPath = getWorkingDirectory() .. "/gamefixer/genrl/Hits/bell_01.mp3",
	  painPath = getWorkingDirectory() .. "/gamefixer/genrl/Pain/pain_01.mp3",
	  dkPath = getWorkingDirectory() .. "/gamefixer/genrl/Deagle/deagle_01.mp3",
	  m4Path = getWorkingDirectory() .. "/gamefixer/genrl/M4/m4_01.mp3",
	  shotgunPath = getWorkingDirectory() .. "/gamefixer/genrl/Shotgun/escopeta_01.mp3",
	  riflePath = getWorkingDirectory() .. "/gamefixer/genrl/Rifle/rifle_01.mp3",
	  akPath = getWorkingDirectory() .. "/gamefixer/genrl/AK-47/ak47_01.mp3",
	  mp5Path = getWorkingDirectory() .. "/gamefixer/genrl/MP5/mp5_01.mp3",
	  sniperPath = getWorkingDirectory() .. "/gamefixer/genrl/Sniper/sniper_01.mp3",
	  edcPath = getWorkingDirectory() .. "/gamefixer/genrl/Edc/edc_01.mp3",
	  gunPath = getWorkingDirectory() .. "/gamefixer/genrl/9mm/9mm_01.mp3",
	  smgPath = getWorkingDirectory() .. "/gamefixer/genrl/Smg/smg_01.mp3"
	}
}, configFile ))

inicfg.save(configData, configFile )

function openLink(url)
	gta._Z12AND_OpenLinkPKc(url)
end

local function setSensitivity(value)
    gta.AIMWEAPON_STICK_SENS = 0.001 + value / 3000.0
end

function setCameraRenderDistance(distance)
    Camera.pRwCamera.farplane = distance
end

function fileExists(filePath)
	local file = io.open(filePath, "r")
	
	if file then
		io.close(file)
		
		return true
	else
		return false
	end
end

local imguiNew = imgui.new

-------------------------- [window] --------------------------
local toggleMenu = imguiNew.bool(false)
local menuConfig = {
	opened = imguiNew .bool(false),
	selected = {
		[0] = " Main"
	},
	tabs = {
		[faicons.WRENCH] = " Correction ",
		[faicons.GEAR] = " Settings",
		[faicons.ROCKET] = " Boost FPS",
		[faicons.KEYBOARD] = " Commands",
		[faicons.MUSIC] = " UGENRL",
		[faicons.BARS] = " Main"
	}
}

-------------------------- [index] --------------------------
local selectedIndexes = {
    hit = imgui.new.int(configData.ugenrl_main.selectedHit),
    pain = imgui.new.int(configData.ugenrl_main.selectedPain),
    dk = imgui.new.int(configData.ugenrl_main.selectedDK),
    m4 = imgui.new.int(configData.ugenrl_main.selectedM4),
    shotgun = imgui.new.int(configData.ugenrl_main.selectedShotgun),
    rifle = imgui.new.int(configData.ugenrl_main.selectedRifle),
    ak = imgui.new.int(configData.ugenrl_main.selectedAK),
    mp5 = imgui.new.int(configData.ugenrl_main.selectedMP5),
    sniper = imgui.new.int(configData.ugenrl_main.selectedSniper),
    edc = imgui.new.int(configData.ugenrl_main.selectedEdc),
    gun = imgui.new.int(configData.ugenrl_main.selectedGun),
    smg = imgui.new.int(configData.ugenrl_main.selectedSmg)
}



-------------------------- [sliders] --------------------------
local sliders = {
	opacityLevel = imguiNew.float(1),
	fontSizeScale = imguiNew.float(configData.settings.imguiItemsScale),
	miraPosX = imguiNew.float(configData.main.miraposx),
	miraPosY = imguiNew.float(configData.main.miraposy),
	renderdist = imguiNew.float(configData.main.renderDistance),
	aspectRatio = imguiNew.float(configData.main.aspectratio),
	fovValue = imguiNew.float(configData.main.Fov),
	sensitivityValue = imguiNew.float(configData.main.sensitivity),
	weatherValue = imguiNew.int(configData.Weather_time.Weather == -1 and 0 or configData.Weather_time.Weather),
	hourValue = imguiNew.int(configData.Weather_time.TimeHours == -1 and 0 or configData.Weather_time.TimeHours),
	minutesValue = imguiNew.int(configData.Weather_time.TimeMinutes == -1 and 0 or configData.Weather_time.TimeMinutes),
	hitVolume = imguiNew.float(configData.ugenrl_volume.hit),
    painVolume = imguiNew.float(configData.ugenrl_volume.pain),
    weaponVolume = imguiNew.float(configData.ugenrl_volume.weapon)
}

--------------------------- [checkboxes] ---------------------------
local checkboxes = {
	enableCustomCrosshair = imguiNew.bool(configData.main.croshair),
	enabledHitMarker = imguiNew.bool(configData.main.hitmarker),
	enabledHitEfects = imguiNew.bool(configData.main.hitefects),
	ugenrl_enable = imguiNew.bool(configData.ugenrl_main.enable),
	autonosounds = imguiNew.bool(configData.ugenrl_main.autonosounds),
	weapon_checkbox = imguiNew.bool(configData.ugenrl_main.weapon),
	enemyweapon_checkbox = imguiNew.bool(configData.ugenrl_main.enemyWeapon),
	hit_checkbox = imguiNew.bool(configData.ugenrl_main.hit),
	pain_checkbox = imguiNew.bool(configData.ugenrl_main.pain),
	weapon_checkbox = imguiNew.bool(configData.ugenrl_main.weapon),
	autoCrosshair = imgui.new.bool(configData.main.auto_crosshair)  -- Nuevo checkbox
}

-------------------------- [buffer] --------------------------
local buffers = {
	cmd_openmenu = imguiNew.char[128](configData.commands.openmenu),
	cmd_settime = imguiNew.char[128](configData.commands.settime),
	cmd_setweather = imguiNew.char[128](configData.commands.setweather),
	cmd_resettime = imguiNew.char[128](configData.commands.resettime),
	cmd_resetweather = imguiNew.char[128](configData.commands.resetweather),
	cmd_clearchat = imguiNew.char[128](configData.commands.clearchat),
	cmd_togglechat = imguiNew.char[128](configData.commands.togglechat),
	cmd_reconect = imguiNew.char[128](configData.commands.reconect),
	cmd_ugenrl = imguiNew.char[128](configData.commands.ugenrl),
	cmd_uds = imguiNew.char[128](configData.commands.uds),
	cmd_uss = imguiNew.char[128](configData.commands.uss),
	cmd_ums = imguiNew.char[128](configData.commands.ums),
	cmd_urs = imguiNew.char[128](configData.commands.urs),
	cmd_umps = imguiNew.char[128](configData.commands.umps),
	cmd_usmg = imguiNew.char[128](configData.commands.usmg),
	cmd_uaks = imguiNew.char[128](configData.commands.uaks), 
    cmd_usps = imguiNew.char[128](configData.commands.usps),  
    cmd_u9ms = imguiNew.char[128](configData.commands.u9ms), 
    cmd_uedc = imguiNew.char[128](configData.commands.uedc),
    cmd_uhits = imguiNew.char[128](configData.commands.uhits),
    cmd_upns = imguiNew.char[128](configData.commands.upns)
}

-------------------------- [paths] --------------------------
local paths = {
	hitPath = configData.ugenrl_sounds.hitPath or getWorkingDirectory() .. "/gamefixer/genrl/Hits/default.mp3",
	painPath = configData.ugenrl_sounds.painPath or getWorkingDirectory() .. "/gamefixer/genrl/Pain/default.mp3",
	dkPath = configData.ugenrl_sounds.dkPath or getWorkingDirectory() .. "/gamefixer/genrl/Deagle/default.mp3",
	m4Path = configData.ugenrl_sounds.m4Path or getWorkingDirectory() .. "/gamefixer/genrl/M4/default.mp3",
	shotgunPath = configData.ugenrl_sounds.shotgunPath or getWorkingDirectory() .. "/gamefixer/genrl/Shotgun/default.mp3",
	riflePath = configData.ugenrl_sounds.riflePath or getWorkingDirectory() .. "/gamefixer/genrl/Rifle/default.mp3",
	akPath = configData.ugenrl_sounds.akPath or getWorkingDirectory() .. "/gamefixer/genrl/AK-47/default.mp3",
	mp5Path = configData.ugenrl_sounds.mp5Path or getWorkingDirectory() .. "/gamefixer/genrl/MP5/default.mp3",
	sniperPath = configData.ugenrl_sounds.sniperPath or getWorkingDirectory() .. "/gamefixer/genrl/Sniper/default.mp3",
	edcPath = configData.ugenrl_sounds.edcPath or getWorkingDirectory() .. "/gamefixer/genrl/Edc/default.mp3",
	gunPath = configData.ugenrl_sounds.gunPath or getWorkingDirectory() .. "/gamefixer/genrl/9mm/default.mp3",
	smgPath = configData.ugenrl_sounds.smgPath or getWorkingDirectory() .. "/gamefixer/genrl/Smg/default.mp3"
}

local script_name = "{73b461}[GameFixer]"
local toggled_sm = false
local zoom_offset = 0
local is_fov_modified = false
local hitMarkerColor = imguiNew.float[4](configData.colors.hit1r, configData.colors.hit1g, configData.colors.hit1b, configData.colors.hit1a)
local hitEfectsColor = imguiNew.float[4](configData.colors.hit2r, configData.colors.hit2g, configData.colors.hit2b, configData.colors.hit2a)
local selectedTab = 1
local colorListNumber = imguiNew.int(configData.settings.theme)
local colorList = {"Rojo","Verde","Azul", "Oscuro", "Cian", "Morado Neon", "Cyberpunk"}
local colorListBuffer = imguiNew["const char*"][#colorList](colorList)
local showCustomCrosshairSettings = imgui.new.bool(false)


---------------------- [UGENRL]----------------------

local function listFilesInDirectory(directoryPath)
	local fileListProcess = io.popen("ls \"" .. directoryPath .. "\"")
	local fileList = {}
	for fileName in fileListProcess.lines(fileListProcess) do
	  table.insert(fileList, fileName)
	end
	fileListProcess:close()
	return fileList
end
  
  local function getAvailableSounds(directory)
    local directoryPath = getWorkingDirectory() .. "/gamefixer/genrl/" .. directory .. "/"
    local fileNames = listFilesInDirectory(directoryPath)
    local soundFiles = {}

    for _, fileName in ipairs(fileNames) do
        if fileName:match("%.mp3$") or fileName:match("%.wav$") then
            table.insert(soundFiles, directoryPath .. fileName)
        end
    end

    return soundFiles
end

local availableHitsSounds = getAvailableSounds("Hits")
local availablePainSounds = getAvailableSounds("Pain")
local availableDKSounds = getAvailableSounds("Deagle")
local availableM4Sounds = getAvailableSounds("M4")
local availableShotgunSounds = getAvailableSounds("Shotgun")
local availableRifleSounds = getAvailableSounds("Rifle")
local availableAkSounds = getAvailableSounds("AK-47")
local availableMp5Sounds = getAvailableSounds("MP5")
local availableSniperSounds = getAvailableSounds("Sniper")
local availableEdcSounds = getAvailableSounds("Edc")
local availableGunSounds = getAvailableSounds("9mm")
local availableSmgSounds = getAvailableSounds("Smg")

function clearSound(audio)
    lua_thread.create(function()
        while getAudioStreamState(audio) == 1 do wait(50) end
        collectgarbage()
    end)
end

  ---------------------- [FFI]----------------------
ffi.cdef[[
	typedef struct RwRect RwRect;
	typedef struct RwCamera RwCamera;
	void _Z10CameraSizeP8RwCameraP6RwRectff(RwCamera *camera, RwRect *rect, float unk, float aspect);
    float _ZN7CCamera22m_f3rdPersonCHairMultXE;
    float _ZN7CCamera22m_f3rdPersonCHairMultYE;
    void _ZN4CHud14DrawCrossHairsEv();
    void _Z12AND_OpenLinkPKc(const char* link);
	float AIMWEAPON_STICK_SENS;
]]

---------------------- [Main Menu]----------------------
local mainMenuFrame = imgui.OnFrame(function()
	return toggleMenu[0]
	end, function(arg_12_0)
	local screenWidth, screenHeight = getScreenResolution()
	local menuWidth = 1266
	local menuHeight = 775

	imgui.GetStyle().Alpha = sliders.opacityLevel[0]

	imgui.SetNextWindowSize(imgui.ImVec2(menuWidth, menuHeight), imgui.Cond.FirstUseEver)
	imgui.BeginWin11Menu(" ", toggleMenu, false, menuConfig.tabs, menuConfig.selected, menuConfig.opened, 90, 130)
	imgui.SetWindowFontScale(sliders.fontSizeScale[0])

	local windowSize = imgui.GetWindowSize()
	imgui.SetCursorPos(imgui.ImVec2(windowSize.x - 43, 10))
	addons.CloseButton("##closemenu", toggleMenu, 33, 5)
	
	if menuConfig.selected[0] == " Correction " then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Spacing()
		imgui.Spacing()
		imgui.Text("Corección")
		imgui.PopFont()
		imgui.SetCursorPosY(90)
		imgui.PushFont(font2)

		if not showCustomCrosshairSettings[0] then
			if imgui.Checkbox("Corrección automática de la mira", checkboxes.autoCrosshair) then
				if checkboxes.autoCrosshair[0] then
					checkboxes.enableCustomCrosshair[0] = false
					configData.main.croshair = false
					adjustCrosshairToAspectRatio()
				end
				configData.main.auto_crosshair = checkboxes.autoCrosshair[0]
				inicfg.save(configData, configFile)
			end

			imgui.SameLine(800)
			imgui.PopFont()
			imgui.PushFont(nil)

			local buttonWidth = 100
			local buttonHeight = 40
			local windowWidth = imgui.GetWindowSize().x
			local buttonX = windowWidth - buttonWidth - 10

			imgui.SetCursorPosX(buttonX)
			if imgui.Button(faicons.CROSSHAIRS .. "##customizeCrosshair", imgui.ImVec2(buttonWidth, buttonHeight)) then
				showCustomCrosshairSettings[0] = true
			end

		else
			imgui.PushFont(nil)
			if imgui.Button(faicons.ARROW_LEFT .. "##back", imgui.ImVec2(100, 40)) then
				showCustomCrosshairSettings[0] = false
			end
			imgui.PopFont()

			imgui.Spacing()
			imgui.PushFont(font1)
			imgui.Text("Ajustar posición de la mira")  
			imgui.PushFont(font2)
			imgui.Spacing()
			imgui.Text("Si la posición de la mira no es la adecuada con la corrección automática,\npuede ajustarla manualmente activando esta opción.")  
			imgui.Spacing()

			if imgui.Checkbox("Activar mira personalizada", checkboxes.enableCustomCrosshair) then
				if checkboxes.enableCustomCrosshair[0] then
					checkboxes.autoCrosshair[0] = false
					configData.main.auto_crosshair = false
				end
				configData.main.croshair = checkboxes.enableCustomCrosshair[0]
				inicfg.save(configData, configFile)
			end
			imgui.PopFont()

			if checkboxes.enableCustomCrosshair[0] then	
				imgui.Spacing()
				imgui.PushFont(font2)
				imgui.Text("Posición horizontal")
				imgui.PopFont()
				imgui.PushFont(font1)
				local totalWidth = imgui.GetContentRegionAvail().x
				local sliderWidth = totalWidth - (50 + 50 + 8 + 16)
				imgui.PushItemWidth(sliderWidth)
				
				
				local isXEdited = imgui.SliderFloat("##miraposx", sliders.miraPosX, 0.371, 0.671, "%.3f")
				if isXEdited then
					configData.main.miraposx = sliders.miraPosX[0]
					inicfg.save(configData, configFile)
				end
				local isXActive = imgui.IsItemActive() 
				
				imgui.SameLine()
				imgui.PopFont()
				imgui.PushFont(nil)
				if imgui.Button("\xef\x81\xa7##miraposx_plus", imgui.ImVec2(50, 45)) then
					sliders.miraPosX[0] = sliders.miraPosX[0] + 0.001
					if sliders.miraPosX[0] > 0.671 then
						sliders.miraPosX[0] = 0.671
					end
					configData.main.miraposx = sliders.miraPosX[0]
					inicfg.save(configData, configFile)
				end
				imgui.SameLine()
				if imgui.Button(faicons.MINUS .. "##miraposx_minus", imgui.ImVec2(50, 45)) then
					sliders.miraPosX[0] = sliders.miraPosX[0] - 0.001
					if sliders.miraPosX[0] < 0.371 then
						sliders.miraPosX[0] = 0.371
					end
					configData.main.miraposx = sliders.miraPosX[0]
					inicfg.save(configData, configFile)
				end
				imgui.PopFont()
				imgui.Spacing()
				imgui.PushFont(font2)
				imgui.Text("Posición vertical")
				imgui.PopFont()
				
				local isYEdited = imgui.SliderFloat("##miraposy", sliders.miraPosY, 0.285, 0.585, "%.3f")
				if isYEdited then
					configData.main.miraposy = sliders.miraPosY[0]
					inicfg.save(configData, configFile)
				end
				
				local isYActive = imgui.IsItemActive()
				
				imgui.PushFont(nil)
				imgui.SameLine()
				if imgui.Button("\xef\x81\xa7##miraposy_plus", imgui.ImVec2(50, 45)) then
					sliders.miraPosY[0] = sliders.miraPosY[0] + 0.001
					if sliders.miraPosY[0] > 0.585 then
						sliders.miraPosY[0] = 0.585
					end
					configData.main.miraposy = sliders.miraPosY[0]
					inicfg.save(configData, configFile)
				end
				imgui.SameLine()
				if imgui.Button(faicons.MINUS .. "##miraposy_minus", imgui.ImVec2(50, 45)) then
					sliders.miraPosY[0] = sliders.miraPosY[0] - 0.001
					if sliders.miraPosY[0] < 0.285 then
						sliders.miraPosY[0] = 0.285
					end
					configData.main.miraposy = sliders.miraPosY[0]
					inicfg.save(configData, configFile)
				end
				imgui.PopFont()
				imgui.PushFont(font2)
				if isXActive or isYActive then
					sliders.opacityLevel[0] = 0.02
				else
					sliders.opacityLevel[0] = 1.0
				end
				imgui.Spacing()
				imgui.Spacing()
				if imgui.Button("Restaurar ajustes por defecto", imgui.ImVec2(-1, 40)) then
					configData.main.miraposx = 0.5185
					configData.main.miraposy = 0.438
					inicfg.save(configData, configFile)
					sliders.miraPosX[0] = configData.main.miraposx
					sliders.miraPosY[0] = configData.main.miraposy
				end
			end
			
		end

		elseif menuConfig.selected[0] == " Boost FPS" then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Text("Aumentar FPS")
		imgui.PopFont()
		imgui.SetCursorPosY(90)
		imgui.PushFont(font2)
		imgui.Spacing()
		imgui.Spacing()
		imgui.TextDisabled("Distancia de dibujado")
		imgui.Spacing()
		imgui.Spacing()
		imgui.PushItemWidth(-1)
		if imgui.SliderFloat("##renderDistance", sliders.renderdist, 50, 900, "%.0f") then
			configData.main.renderDistance = sliders.renderdist[0]
			inicfg.save(configData, configFile)
		end
		imgui.PopItemWidth()
		
		elseif menuConfig.selected[0] == " Settings" then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Text("Configuración")
		imgui.PopFont()
		imgui.SetCursorPosY(90)
		imgui.PushFont(font2)
		imgui.Spacing()
		imgui.Spacing()
		imgui.TextDisabled("Opacidad")
		imgui.PushItemWidth(-1)
		imgui.SliderFloat("##alpha", sliders.opacityLevel, 0.1, 1, "%.2f")
		imgui.Spacing()
		imgui.Spacing()
		imgui.TextDisabled("Tamaño de los items")
		if imgui.SliderFloat("##imguiItemsScale", sliders.fontSizeScale, 0.5, 1, "%.2f") then 
		configData.settings.imguiItemsScale = sliders.fontSizeScale[0]
		inicfg.save(configData, configFile)
		end
		imgui.Spacing()
		imgui.Spacing()
		imgui.TextDisabled("Seleccionar tema")
		imgui.PushStyleVarVec2(imgui.StyleVar.ItemSpacing, imgui.ImVec2(5, 15))
		if imgui.Combo(" ", colorListNumber, colorListBuffer, #colorList) then
			configData.settings.theme = colorListNumber[0]
			inicfg.save(configData, configFile)
			theme[colorListNumber[0]+1].change()
		end
imgui.PopItemWidth()
imgui.PopStyleVar()
imgui.SetCursorPosY(windowSize.y - 35)
imgui.PopFont()
imgui.PushFont(font1)

imgui.Text("Autor:")
imgui.SameLine()

-- Cinza
imgui.PushStyleColor(imgui.Col.Text, imgui.ImVec4(0.6, 0.6, 0.6, 1.0))
imgui.Text("Sagazx_wg")
imgui.PopStyleColor()
	
		if imgui.IsItemClicked() then
			openLink("")
		end
		
		imgui.PushStyleColor(imgui.Col.Button, imgui.ImVec4(0.0, 0.00, 0.0, 0.0))
		imgui.SetCursorPos(imgui.ImVec2(windowSize.x - 116, windowSize.y - 70))
		imgui.PushFont(nil)
		if imgui.Button(faicons.ROTATE_RIGHT .. "##reload", imgui.ImVec2(100, 60)) then
			thisScript():reload()
		end
		imgui.PopStyleColor()

		elseif menuConfig.selected[0] == " Commands" then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Text("Comandos")
		imgui.PopFont()
		imgui.PushFont(font2)
		imgui.SetCursorPosY(90)
		imgui.BeginChild("##MainScroll", imgui.ImVec2(-1), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
		imgui.Spacing()
		imgui.Spacing()
		imgui.PushItemWidth(120)

		local function drawCommandInput(label, buffer, configKey)
			if imgui.InputText(label, buffer, ffi.sizeof(buffer)) then
				configData.commands[configKey] = ffi.string(buffer)
				inicfg.save(configData, configFile)
				reloadCommands()
			end

			if ffi.string(buffer):sub(1, 1) ~= "/" then
				imgui.SameLine()
			
				imgui.PushFont(nil)  
				imgui.TextColored(imgui.ImVec4(1, 0, 0, 1), "") 
				imgui.PopFont()
				imgui.SameLine()
				imgui.PushFont(font2)  
				imgui.TextColored(imgui.ImVec4(1, 0, 0, 1), "debe empezar con /")
				imgui.PopFont()
			end
			
			imgui.Spacing()
			imgui.Spacing()
		end

		drawCommandInput(" Comando para abrir el menú", buffers.cmd_openmenu, "openmenu")
		drawCommandInput(" Ajustar la hora", buffers.cmd_settime, "settime")
		drawCommandInput(" Modificar el clima", buffers.cmd_setweather, "setweather")
		drawCommandInput(" Sincronizar la hora con el servidor", buffers.cmd_resettime, "resettime")
		drawCommandInput(" Sincronizar el clima con el servidor", buffers.cmd_resetweather, "resetweather")
		drawCommandInput(" Limpiar el chat", buffers.cmd_clearchat, "clearchat")
		drawCommandInput(" Mostrar/Ocultar mensajes del chat", buffers.cmd_togglechat, "togglechat")
		drawCommandInput(" Reconectarse al servidor", buffers.cmd_reconect, "reconect")
		
		drawCommandInput(" Encender/Apagar Ultimate Genrl", buffers.cmd_ugenrl, "ugenrl")
		drawCommandInput(" Cambiar sonido de la desert", buffers.cmd_uds, "uds")
		drawCommandInput(" Cambiar sonido de la m4", buffers.cmd_ums, "ums")
		drawCommandInput(" Cambiar sonido de la escopeta", buffers.cmd_uss, "uss")
		drawCommandInput(" Cambiar sonido del rifle", buffers.cmd_urs, "urs")
		drawCommandInput(" Cambiar sonido de la mp5", buffers.cmd_umps, "umps")
		drawCommandInput(" Cambiar sonido de la uzi/tec9", buffers.cmd_usmg, "usmg")
		drawCommandInput(" Cambiar sonido de la ak47", buffers.cmd_uaks, "uaks")
		drawCommandInput(" Cambiar sonido de la 9mm", buffers.cmd_u9ms, "u9ms")
		drawCommandInput(" Cambiar sonido de la edc", buffers.cmd_uedc, "uedc")
		drawCommandInput(" Cambiar sonido del sniper", buffers.cmd_usps, "usps")
		drawCommandInput(" Cambiar sonido del HiSound", buffers.cmd_uhits, "uhits")
		drawCommandInput(" Cambiar sonido de dolor", buffers.cmd_upns, "upns")
		imgui.PopItemWidth()
		
		elseif menuConfig.selected[0] == " UGENRL" then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Text("UGENRL")
		imgui.PopFont()
		imgui.SetCursorPosY(90)
		imgui.BeginChild("##MainScroll", imgui.ImVec2(-1), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
		imgui.PushFont(font2)
		imgui.Spacing()
		imgui.Spacing()
		if imgui.Checkbox("Activar Ultimate Genrl", checkboxes.ugenrl_enable) then
			configData.ugenrl_main.enable = checkboxes.ugenrl_enable[0]
			inicfg.save(configData, configFile)
		end		
		
		if checkboxes.ugenrl_enable[0] then
			imgui.Spacing()
			imgui.Spacing()
			if imgui.Checkbox("Sonidos de disparo del jugador", checkboxes.weapon_checkbox) then
				configData.ugenrl_main.weapon = checkboxes.weapon_checkbox[0]
				inicfg.save(configData, configFile)
			end
			imgui.SameLine()
			if imgui.Checkbox("Sonidos de disparo del enemigo", checkboxes.enemyweapon_checkbox) then
				configData.ugenrl_main.enemyWeapon = checkboxes.enemyweapon_checkbox[0]
				inicfg.save(configData, configFile)
			end
			imgui.Spacing()
			imgui.Spacing()
			if imgui.Checkbox("Sonidos de golpe del jugador", checkboxes.hit_checkbox) then
				configData.ugenrl_main.hit = checkboxes.hit_checkbox[0]
				inicfg.save(configData, configFile)
			end
			imgui.SameLine()
			if imgui.Checkbox("Sonidos de dolor", checkboxes.pain_checkbox) then
				configData.ugenrl_main.pain = checkboxes.pain_checkbox[0]
				inicfg.save(configData, configFile)
			end
			imgui.Spacing()
			imgui.Spacing()
			imgui.Separator()
			function drawWeaponSoundList(label, x, y, availableSounds, selectedIndex, pathKey, configKey, volumeSlider)
				imgui.SetCursorPos(imgui.ImVec2(x, y))
				imgui.Text(label)
				imgui.SetCursorPos(imgui.ImVec2(x, y + 35))
				imgui.BeginChild(label .. "List", imgui.ImVec2(230, 220), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
				
				for index, filePath in ipairs(availableSounds) do
					local isSelected = (selectedIndex[0] == index - 1)
					if imgui.Selectable(filePath:match("[^/]+$"), isSelected) then
						selectedIndex[0] = index - 1
						paths[pathKey] = filePath
						configData.ugenrl_sounds[pathKey] = filePath
						configData.ugenrl_main[configKey] = selectedIndex[0]
			
						local audioStream = loadAudioStream(filePath)
						setAudioStreamVolume(audioStream, volumeSlider[0])
						setAudioStreamState(audioStream, 1)
						inicfg.save(configData, configFile)
					end
				end
				imgui.EndChild()
			end
			
			local soundLists = {
				{label = "Sonido deagle:", sounds = availableDKSounds, index = selectedIndexes.dk, pathKey = "dkPath", configKey = "selectedDK", volumeSlider = sliders.weaponVolume},
				{label = "Sonido escopeta:", sounds = availableShotgunSounds, index = selectedIndexes.shotgun, pathKey = "shotgunPath", configKey = "selectedShotgun", volumeSlider = sliders.weaponVolume},
				{label = "Sonido m4:", sounds = availableM4Sounds, index = selectedIndexes.m4, pathKey = "m4Path", configKey = "selectedM4", volumeSlider = sliders.weaponVolume},
				{label = "Sonido ak47:", sounds = availableAkSounds, index = selectedIndexes.ak, pathKey = "akPath", configKey = "selectedAK", volumeSlider = sliders.weaponVolume},
				{label = "Sonido mp5:", sounds = availableMp5Sounds, index = selectedIndexes.mp5, pathKey = "mp5Path", configKey = "selectedMP5", volumeSlider = sliders.weaponVolume},
				{label = "Sonido sniper:", sounds = availableSniperSounds, index = selectedIndexes.sniper, pathKey = "sniperPath", configKey = "selectedSniper", volumeSlider = sliders.weaponVolume},
				{label = "Sonido edc:", sounds = availableEdcSounds, index = selectedIndexes.edc, pathKey = "edcPath", configKey = "selectedEdc", volumeSlider = sliders.weaponVolume},
				{label = "Sonido 9mm:", sounds = availableGunSounds, index = selectedIndexes.gun, pathKey = "gunPath", configKey = "selectedGun", volumeSlider = sliders.weaponVolume},
				{label = "Sonido Uzi/Tec9:", sounds = availableSmgSounds, index = selectedIndexes.smg, pathKey = "smgPath", configKey = "selectedSmg", volumeSlider = sliders.weaponVolume},
				{label = "Sonido Rifle:", sounds = availableRifleSounds, index = selectedIndexes.rifle, pathKey = "riflePath", configKey = "selectedRifle", volumeSlider = sliders.weaponVolume},
				{label = "Sonido al infligir daño:", sounds = availableHitsSounds, index = selectedIndexes.hit, pathKey = "hitPath", configKey = "selectedHit", volumeSlider = sliders.hitVolume},
				{label = "Sonido de dolor:", sounds = availablePainSounds, index = selectedIndexes.pain, pathKey = "painPath", configKey = "selectedPain", volumeSlider = sliders.painVolume},
			}
			
			local totalWidth = imgui.GetWindowSize().x
			local listWidth = 230
			local listHeight = 220
			local spacingY = 30
			local minMargin = 10

			local columns = math.floor((totalWidth + minMargin) / (listWidth + minMargin))
			columns = math.max(1, columns)

			local usedWidth = columns * listWidth
			local remainingSpace = totalWidth - usedWidth
			local margin = remainingSpace / (columns + 1)


			
			for i, entry in ipairs(soundLists) do
				local col = (i - 1) % columns
				local row = math.floor((i - 1) / columns)
			
				local x = margin + col * (listWidth + margin)
				local y = 210 + row * (listHeight + 90)
			
				drawWeaponSoundList(entry.label, x, y, entry.sounds, entry.index, entry.pathKey, entry.configKey, entry.volumeSlider)
			end
			imgui.Spacing()
			imgui.Spacing()
			imgui.Spacing()
			imgui.Spacing()
			imgui.Separator()
			imgui.Spacing()
			imgui.Spacing()
			imgui.Spacing()
			imgui.Spacing()

			imgui.PushItemWidth(300)
			if imgui.SliderFloat("##hitVolume", sliders.hitVolume, 0, 1, "%.2f") then
				configData.ugenrl_volume.hit = tonumber(string.format("%.2f", sliders.hitVolume[0]))
				inicfg.save(configData, configFile)
			end		
			imgui.PopItemWidth()
			imgui.SameLine()
			imgui.Text("Volumen de golpe")
			imgui.Spacing()
			imgui.PushItemWidth(300)
			if imgui.SliderFloat("##painVolume", sliders.painVolume, 0, 1, "%.2f") then
				configData.ugenrl_volume.pain = tonumber(string.format("%.2f", sliders.painVolume[0]))
				inicfg.save(configData, configFile)
			end
			imgui.PopItemWidth()
			imgui.SameLine()
			imgui.Text("Volumen de dolor")
			imgui.Spacing()
			imgui.PushItemWidth(300)
			if imgui.SliderFloat("##weapon", sliders.weaponVolume, 0, 1, "%.2f") then
				configData.ugenrl_volume.weapon = tonumber(string.format("%.2f", sliders.weaponVolume[0]))
				inicfg.save(configData, configFile)
			end
			imgui.PopItemWidth()
			imgui.SameLine()
			imgui.Text("Volumen del disparos")
			imgui.Spacing()
		end
		elseif menuConfig.selected[0] == " Main" then
		imgui.PushFont(font1)
		imgui.SetCursorPosY(18)
		imgui.Text("Principal")
		imgui.PopFont()
		imgui.PushFont(font2)
		imgui.SetCursorPosY(90)
		imgui.BeginChild("##MainScroll", imgui.ImVec2(-1), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
		imgui.TextDisabled("Configuraciones Generales")
		imgui.Spacing()
		imgui.Separator()
		imgui.Spacing()
		imgui.TextDisabled("Resolución:")
		imgui.SameLine(150)
		local asptext = (sliders.aspectRatio[0] == 1) and "Predeterminado" or "%.1f"
		imgui.PushItemWidth(-1)
		if imgui.SliderFloat("##aspect", sliders.aspectRatio, 1, 3, asptext) then
			configData.main.aspectratio = sliders.aspectRatio[0]
			inicfg.save(configData, configFile)
			if checkboxes.autoCrosshair[0] then
				adjustCrosshairToAspectRatio()
			end
		end
		
		imgui.PopItemWidth()
		imgui.Spacing()
		imgui.TextDisabled("Campo de Visión (FOV):")
		imgui.SameLine()
		local fovtext = (sliders.fovValue[0] == 70) and "Predeterminado" or "%.0f"
		imgui.PushItemWidth(-1)
		if imgui.SliderFloat("##fov", sliders.fovValue, 70, 120, fovtext) then
			configData.main.Fov = sliders.fovValue[0]
			inicfg.save(configData, configFile)
		end
		imgui.PopItemWidth()
		imgui.Spacing()
		imgui.TextDisabled("Sensibilidad:")
		imgui.SameLine(140)
		imgui.PushItemWidth(-1)
		if imgui.SliderFloat("##sensitivity", sliders.sensitivityValue, 0, 100, "%.0f") then
			configData.main.sensitivity = sliders.sensitivityValue[0]
			inicfg.save(configData, configFile)
			setSensitivity(configData.main.sensitivity)
		end
		imgui.PopItemWidth()
		imgui.Spacing()
		imgui.Separator()
		imgui.Spacing()
		if imgui.CollapsingHeader("Editor de Hora y Clima", true, imgui.TreeNodeFlags.DefaultOpen) then
			imgui.Spacing()
			imgui.Spacing()
			imgui.Spacing()
		
			imgui.TextDisabled("Clima:")
			imgui.SameLine(140)
			imgui.PushItemWidth(600)
			if imgui.SliderInt("##weather", sliders.weatherValue, 0, 45, configData.Weather_time.Weather == -1 and "Predeterminado" or "%d") then
				configData.Weather_time.Weather = sliders.weatherValue[0]
				inicfg.save(configData, configFile)
				forceWeatherNow(configData.Weather_time.Weather)
			end
			imgui.PopItemWidth()
		
			imgui.Spacing()
		
			imgui.TextDisabled("Hora:")
			imgui.SameLine(100)
			imgui.PushItemWidth(280)
			if imgui.SliderInt("##hour", sliders.hourValue, 0, 23, configData.Weather_time.TimeHours == -1 and "Predeterminado" or "%d") then
				configData.Weather_time.TimeHours = sliders.hourValue[0]
				inicfg.save(configData, configFile)
				setTimeOfDay(configData.Weather_time.TimeHours, configData.Weather_time.TimeMinutes)
			end
			imgui.PopItemWidth()
		
			imgui.SameLine(400)
		
			imgui.TextDisabled("Minutos:")
			imgui.SameLine(500)
			imgui.PushItemWidth(280)
			if imgui.SliderInt("##minutes", sliders.minutesValue, 0, 59, configData.Weather_time.TimeMinutes == -1 and "Predeterminado" or "%d") then
				configData.Weather_time.TimeMinutes = sliders.minutesValue[0]
				inicfg.save(configData, configFile)
				setTimeOfDay(configData.Weather_time.TimeHours, configData.Weather_time.TimeMinutes)
			end
			imgui.PopItemWidth()
		
			imgui.Spacing()
			imgui.Spacing()

			imgui.SetCursorPosX((imgui.GetWindowWidth() - 300) / 2)
			if imgui.Button("Sincronizar con el servidor", imgui.ImVec2(300, 35)) then
				configData.Weather_time.Weather = -1
				configData.Weather_time.TimeHours = -1
				configData.Weather_time.TimeMinutes = -1
				inicfg.save(configData, configFile)
				forceWeatherNow(0)
				setTimeOfDay(12, 0)
			end
		end
		imgui.Spacing()
		imgui.Spacing()
		imgui.Spacing()
		imgui.Separator()
		imgui.Spacing()
	
		if imgui.CollapsingHeader("Damager Informer", true, imgui.TreeNodeFlags.DefaultOpen) then
			if imgui.Checkbox("Activar marcador de impacto", checkboxes.enabledHitMarker) then
				configData.main.hitmarker = checkboxes.enabledHitMarker[0]
				inicfg.save(configData, configFile)
			end
			imgui.Spacing()
	
			imgui.TextDisabled("Color del marcador")
			imgui.SameLine(200)
			if imgui.ColorEdit4("##hitMarkerColor", hitMarkerColor, imgui.ColorEditFlags.NoInputs) then
				configData.colors.hit1r = tonumber(string.format("%.2f", hitMarkerColor[0]))
				configData.colors.hit1g = tonumber(string.format("%.2f", hitMarkerColor[1]))
				configData.colors.hit1b = tonumber(string.format("%.2f", hitMarkerColor[2]))
				configData.colors.hit1a = tonumber(string.format("%.2f", hitMarkerColor[3]))
				inicfg.save(configData, configFile)
			end
	
			imgui.Spacing()
			if imgui.Checkbox("Activar Hit efects", checkboxes.enabledHitEfects) then
				configData.main.hitefects = checkboxes.enabledHitEfects[0]
				inicfg.save(configData, configFile)
			end
			imgui.Spacing()
			imgui.TextDisabled("Color de los numeros")
			imgui.SameLine(200)
			if imgui.ColorEdit4("##hitEfectsColor", hitEfectsColor, imgui.ColorEditFlags.NoInputs) then
				configData.colors.hit2r = tonumber(string.format("%.2f", hitEfectsColor[0]))
				configData.colors.hit2g = tonumber(string.format("%.2f", hitEfectsColor[1]))
				configData.colors.hit2b = tonumber(string.format("%.2f", hitEfectsColor[2]))
				configData.colors.hit2a = tonumber(string.format("%.2f", hitEfectsColor[3]))
				inicfg.save(configData, configFile)
			end
		end
	
		imgui.EndChild()
	
	end

	imgui.EndWin11Menu()
end)

----------------------------------------------------- [Functions] ----------------------------------------------------------

---------------------- [Change sound command]----------------------
function changeWeaponSound(param, availableSounds, weaponKey, command, iniKey)
    param = tonumber(param)

    if not param or param < 1 or param > #availableSounds then
        local selectedIndex = iniKey and configData.ugenrl_main[iniKey] or configData.ugenrl_main["selected" .. weaponKey]
        local selectedText = selectedIndex and (selectedIndex + 1)

        sampAddChatMessage(script_name .. " {FFFFFF}Utilice: {dc4747}" .. command .. " [1-" .. #availableSounds .. "]", -1)
        sampAddChatMessage(script_name .. " {FFFFFF}Parametro actual: {dc4747}" .. selectedText, -1)
        return
    end

    local soundPath = availableSounds[param]
    paths[weaponKey .. "Path"] = soundPath
    configData.ugenrl_sounds[weaponKey .. "Path"] = soundPath
    
    if iniKey then
        configData.ugenrl_main[iniKey] = param - 1
    else
        configData.ugenrl_main["selected" .. weaponKey] = param - 1
    end

    selectedIndexes[weaponKey][0] = param - 1  

    inicfg.save(configData, configFile)

    local audioStream = loadAudioStream(soundPath)
    setAudioStreamVolume(audioStream, sliders.weaponVolume[0])
    setAudioStreamState(audioStream, 1)
	clearSound(audioStream)

    local armas_femeninas = {["m4"] = true, ["mp5"] = true, ["smg"] = true, ["shotgun"] = true, ["edc"] = true, ["gun"] = true, ["dk"] = true}
    local weaponNames = {
        dk = "desert",
        m4 = "m4",
        shotgun = "Escopeta",
        rifle = "rifle",
        mp5 = "mp5",
        smg = "uzi/tec9",
        ak = "ak-47",
        gun = "9mm",
        edc = "edc",
        sniper = "sniper"
    }
    local nombreArma = weaponNames[weaponKey] or weaponKey
    local articulo = armas_femeninas[weaponKey] and "de la" or "del"
    sampAddChatMessage(script_name .. " {FFFFFF}El sonido " .. articulo .. " {BABABA}" .. nombreArma .. " {FFFFFF} ha sido configurado en: {dc4747}" .. param, -1)
end

---------------------- [Autocroshair]----------------------
function adjustCrosshairToAspectRatio()
    local aspect = configData.main.aspectratio

	local crosshairPositions = {
		[1.00] = { x = 0.5185, y = 0.438 },
		[1.10] = { x = 0.5185, y = 0.448 },
		[1.20] = { x = 0.5185, y = 0.452 },
		[1.30] = { x = 0.5185, y = 0.455 },
		[1.40] = { x = 0.5185, y = 0.460 },
		[1.50] = { x = 0.5185, y = 0.463 },
		[1.60] = { x = 0.5185, y = 0.465 },
		[1.70] = { x = 0.5185, y = 0.467 },
		[1.80] = { x = 0.5185, y = 0.470 },
		[1.90] = { x = 0.5185, y = 0.470 },
		[2.00] = { x = 0.5185, y = 0.472 },
		[2.10] = { x = 0.5185, y = 0.474 },
		[2.20] = { x = 0.5185, y = 0.475 },
		[2.30] = { x = 0.5185, y = 0.476 },
		[2.40] = { x = 0.5185, y = 0.477 },
		[2.50] = { x = 0.5185, y = 0.478 },
		[2.60] = { x = 0.5185, y = 0.478 },
		[2.70] = { x = 0.5185, y = 0.478 },
		[2.80] = { x = 0.5185, y = 0.479 },
		[2.90] = { x = 0.5185, y = 0.480 },
		[3.00] = { x = 0.519, y = 0.481 }
	}

    local pos = crosshairPositions[aspect]
    
    if pos then
        configData.main.auto_miraposx = pos.x
        configData.main.auto_miraposy = pos.y
    else
        local closestAspect = 1.00
        for key, _ in pairs(crosshairPositions) do
            if math.abs(aspect - key) < math.abs(aspect - closestAspect) then
                closestAspect = key
            end
        end
        configData.main.auto_miraposx = crosshairPositions[closestAspect].x
        configData.main.auto_miraposy = crosshairPositions[closestAspect].y
    end
    inicfg.save(configData, configFile)
end


---------------------- [Samp Events]----------------------

function sampEvents.onSendBulletSync(bulletSync)
    -- hitMarker
    if bulletSync.target then
        bulletX, bulletY, bulletZ = bulletSync.target.x, bulletSync.target.y, bulletSync.target.z
    end

 	-- WeaponSounds
    if not checkboxes.ugenrl_enable[0] then return end 

    local weaponSounds = {
    	[22] = paths.gunPath,     
        [24] = paths.dkPath,      
        [25] = paths.shotgunPath, 
        [26] = paths.shotgunPath,
        [27] = paths.edcPath,     
        [28] = paths.smgPath,    
        [32] = paths.smgPath,
        [29] = paths.mp5Path,
        [30] = paths.akPath,      
        [31] = paths.m4Path,      
        [33] = paths.riflePath,
        [34] = paths.sniperPath
    }

    if checkboxes.weapon_checkbox[0] and weaponSounds[bulletSync.weaponId] then
        local audioStream = loadAudioStream(weaponSounds[bulletSync.weaponId])
        setAudioStreamVolume(audioStream, sliders.weaponVolume[0])
        setAudioStreamState(audioStream, 1)
		clearSound(audioStream)
    end
end

function sampEvents.onBulletSync(playerId, bulletSync)
	if not checkboxes.ugenrl_enable[0] then return end

    local enemySounds = {
        [22] = paths.gunPath, [24] = paths.dkPath, [25] = paths.shotgunPath,
        [26] = paths.shotgunPath, [27] = paths.edcPath, [28] = paths.smgPath,
        [32] = paths.smgPath, [29] = paths.mp5Path, [30] = paths.akPath,
        [31] = paths.m4Path, [33] = paths.riflePath, [34] = paths.sniperPath
    }

    if not checkboxes.enemyweapon_checkbox[0] or not enemySounds[bulletSync.weaponId] or not fileExists(enemySounds[bulletSync.weaponId]) then
        return
    end

    local charIsExist, enemyPed = sampGetCharHandleBySampPlayerId(playerId)
    if not charIsExist or not enemyPed or not doesCharExist(enemyPed) then
        return
    end

    local ex, ey, ez = getCharCoordinates(enemyPed)
    local px, py, pz = getCharCoordinates(PLAYER_PED)
    if not ex or not px then return end

    local distance = getDistanceBetweenCoords3d(px, py, pz, ex, ey, ez)

    local minDistance = 8.0  
    local midDistance = 20.0 
    local maxDistance = 60.0 
    local minVolume = 0.05

    local volumeFactor
    if distance <= minDistance then
        volumeFactor = 0.7 
    elseif distance <= midDistance then
        volumeFactor = 0.5 + 0.2 * (1 - (distance - minDistance) / (midDistance - minDistance)) 
    elseif distance <= maxDistance then
        volumeFactor = math.max(minVolume, 0.05 + 0.3 * (1 - (distance - midDistance) / (maxDistance - midDistance))) 
    else
        volumeFactor = 0.0 
    end

    local audioStream = loadAudioStream(enemySounds[bulletSync.weaponId])
    if audioStream then
        local baseVolume = sliders.weaponVolume[0] 
        local adjustedVolume = baseVolume * volumeFactor
        setAudioStreamVolume(audioStream, adjustedVolume)
        setAudioStreamState(audioStream, 1)
		clearSound(audioStream)
    end
end


function sampEvents.onSendGiveDamage(playerId, damage, weapon, bodypart)
    -- Hitmarker
    if checkboxes.enabledHitMarker[0] and weapon > 21 and weapon < 35 then
        lua_thread.create(renderHitMarker, {bulletX, bulletY, bulletZ, playerId, math.ceil(damage)})
    end

	-- Hiteffects
    if checkboxes.enabledHitEfects[0] and weapon > 21 and weapon < 35 then
        lua_thread.create(renderHitEffect, {bulletX, bulletY, bulletZ, playerId, math.ceil(damage)})
    end

    -- Hitsound
	if not checkboxes.ugenrl_enable[0] then return end
    if checkboxes.hit_checkbox[0] and paths.hitPath and fileExists(paths.hitPath) then
        local audioStream = loadAudioStream(paths.hitPath)
        setAudioStreamVolume(audioStream, sliders.hitVolume[0])
        setAudioStreamState(audioStream, 1)
		clearSound(audioStream)
    end
end

function sampEvents.onSendTakeDamage()
	if not checkboxes.ugenrl_enable[0] then return end
	if checkboxes.pain_checkbox[0] and paths.painPath and fileExists(paths.painPath) then
	  local audioStream = loadAudioStream(paths.painPath)
	  setAudioStreamVolume(audioStream, sliders.painVolume[0])
	  setAudioStreamState(audioStream, 1)
	  clearSound(audioStream)
	end
end

---------------------- [Adjusts croshair]----------------------
function drawCrosshairHook()
    local save1 = gta._ZN7CCamera22m_f3rdPersonCHairMultXE
    local save2 = gta._ZN7CCamera22m_f3rdPersonCHairMultYE

    if configData.main.croshair then
        gta._ZN7CCamera22m_f3rdPersonCHairMultXE = configData.main.miraposx
        gta._ZN7CCamera22m_f3rdPersonCHairMultYE = configData.main.miraposy
    elseif configData.main.auto_crosshair then
        gta._ZN7CCamera22m_f3rdPersonCHairMultXE = configData.main.auto_miraposx
        gta._ZN7CCamera22m_f3rdPersonCHairMultYE = configData.main.auto_miraposy
    else
        gta._ZN7CCamera22m_f3rdPersonCHairMultXE = 0.5185
        gta._ZN7CCamera22m_f3rdPersonCHairMultYE = 0.438
    end

    local r = drawCrosshairHook()

    gta._ZN7CCamera22m_f3rdPersonCHairMultXE = save1
    gta._ZN7CCamera22m_f3rdPersonCHairMultYE = save2

    return r
end

drawCrosshairHook = hook.new('void(*)()', drawCrosshairHook, ffi.cast('uintptr_t', ffi.cast('void*', gta._ZN4CHud14DrawCrossHairsEv)))

---------------------- [Adjusts AspectRatio]----------------------
function aspectHook(camera, rect, unk, aspect)
	if sliders.aspectRatio[0] == 1 then
		return aspectHook(camera, rect, unk, aspect)
	end

	return aspectHook(camera, rect, unk, 2.1 / sliders.aspectRatio[0])
end

aspectHook = hook.new("void(*)(RwCamera *camera, RwRect *rect, float unk, float aspect)", aspectHook, ffi.cast("uintptr_t", ffi.cast("void*", gta._Z10CameraSizeP8RwCameraP6RwRectff)))

---------------------- [RGB colors to hexadecimal]----------------------
function RGBToHex(red, green, blue, alpha)
	if red < 0 or red > 255 or green < 0 or green > 255 or blue < 0 or blue > 255 or alpha and (alpha < 0 or alpha > 255) then
		return nil
	end

	if alpha then
		return tonumber(string.format("0x%.2X%.2X%.2X%.2X", alpha, red, green, blue))
	else
		return tonumber(string.format("0x%.2X%.2X%.2X", red, green, blue))
	end
end

---------------------- [Chat]----------------------
function toggleChatMessages()
    toggled_sm = not toggled_sm
    sampAddChatMessage(script_name..": {FFFFFF}Chat "..(toggled_sm and "{dc4747}Desactivado" or "{73b461}Activado"), -1)
end

function blockChatMessages(color, text)
    if toggled_sm then return false end
end

function clearChat()
    for i = 1, 15 do
        sampAddChatMessage("", -1)
    end
end

---------------------- [Hits]----------------------
local color = 0xFFFFA500
local fontFlag = require('moonloader').font_flag
local hitMarkerFont = renderCreateFont('Segoe UI', 15, fontFlag.BOLD + fontFlag.SHADOW)
math.randomseed(os.time())
local hitEffectFont = renderCreateFont("Consolas", 12, fontFlag.BOLD + fontFlag.BORDER + fontFlag.SHADOW)
math.randomseed(os.time())


function renderHitMarker(data)
    if not data or not data[1] or not data[2] or not data[3] then return end

    local startTime = os.clock()
    local x, y, z = data[1], data[2], data[3] 
    local myX, myY, myZ = getCharCoordinates(PLAYER_PED)
    local charIsExist, ped = sampGetCharHandleBySampPlayerId(data[4])

    if not charIsExist then return end

    local r = math.max(0, math.min(255, math.floor(hitMarkerColor[0] * 255)))
    local g = math.max(0, math.min(255, math.floor(hitMarkerColor[1] * 255)))
    local b = math.max(0, math.min(255, math.floor(hitMarkerColor[2] * 255)))
    local a = math.max(0, math.min(255, math.floor(hitMarkerColor[3] * 255)))
    local hitMarkerHex = RGBToHex(r, g, b, a)

    if not hitMarkerHex then return end

    while os.clock() - startTime < 5.75 do
        wait(0)

        if isPointOnScreen(x, y, z, 1) then
            local elapsedTime = os.clock() - startTime
            local alpha = math.floor(decreaseAlpha(hitMarkerHex, math.ceil(bringFloatTo(0, 255, elapsedTime, 5.75))))
            local alphaColor = (alpha * 0x1000000) + (hitMarkerHex % 0x1000000)

            local backX, backY = convert3DCoordsToScreen(x, y, z)
            if backX and backY then
                renderFontDrawText(hitMarkerFont, "x", backX, backY, string.format("0x%08X", alphaColor))
            end
        end
    end
end

function renderHitEffect(data)
    if not data or not data[1] or not data[2] or not data[3] or not data[5] then return end

    local startTime = os.clock()
    local x, y, z = data[1], data[2], data[3] 
    local damage = data[5]
    local myX, myY, myZ = getCharCoordinates(PLAYER_PED)
    local charIsExist, ped = sampGetCharHandleBySampPlayerId(data[4])

    if not charIsExist then return end

    local r = math.max(0, math.min(255, math.floor(hitEfectsColor[0] * 255)))
    local g = math.max(0, math.min(255, math.floor(hitEfectsColor[1] * 255)))
    local b = math.max(0, math.min(255, math.floor(hitEfectsColor[2] * 255)))
    local a = math.max(0, math.min(255, math.floor(hitEfectsColor[3] * 255)))
    local hitEffectHex = RGBToHex(r, g, b, a)

    if not hitEffectHex then return end

    while os.clock() - startTime < 2.5 do
        wait(0)

        if isPointOnScreen(x, y, z, 1) then
            local elapsedTime = os.clock() - startTime
            local alpha = math.floor(decreaseAlpha(hitEffectHex, math.ceil(bringFloatTo(0, 255, elapsedTime, 2.5))))
            local alphaColor = (alpha * 0x1000000) + (hitEffectHex % 0x1000000)

            local backX, backY = convert3DCoordsToScreen(x, y, z)
            if backX and backY then
                local moveUp = math.ceil(bringFloatTo(0, 200, elapsedTime, 2.5))
                renderFontDrawText(hitEffectFont, tostring(damage), backX, backY - moveUp, string.format("0x%08X", alphaColor))
            end
        end
    end
end

function decreaseAlpha(color, decreaseAmount)
    local alpha = math.floor(color / 0x1000000)
    local newAlpha = math.max(0, alpha - decreaseAmount)
    return newAlpha
end

function bringFloatTo(from, to, elapsedTime, duration)
    local progress = elapsedTime / duration
    local newValue = from + (progress * (to - from))
    return newValue
end

---------------------- [Function reconect]----------------------
function reconnect(arg)
    lua_thread.create(function()
        arg = tonumber(arg) or 0
        local ms = 500 + arg * 1000
        if ms <= 0 then
            ms = 100
        end

        while ms > 0 do
            if ms <= 500 then
                local bs = raknetNewBitStream()
                raknetBitStreamWriteInt8(bs, sf.PACKET_DISCONNECTION_NOTIFICATION)
                raknetSendBitStreamEx(bs, sf.SYSTEM_PRIORITY, sf.RELIABLE, 0)
                raknetDeleteBitStream(bs)
            end

            printStringNow("Espera: ~g~" .. tostring(ms) .. "ms", 100)
            wait(100)
            ms = ms - 100
        end

        local bs = raknetNewBitStream()
        raknetEmulPacketReceiveBitStream(sf.PACKET_CONNECTION_LOST, bs)
        raknetDeleteBitStream(bs)
    end)
end

---------------------- [Function Time_Weather]----------------------
function extract_value(i, j)
    if j == nil then j = '%s' end
    local k = {}
    for l in string.gmatch(i, '([^' .. j .. ']+)') do
        table.insert(k, l)
    end
    return k
end

function onWeatherChange(p)
    if p == nil or p == '' then
        sampAddChatMessage(script_name .. "{FFFFFF} Utilice: {dc4747}" .. configData.commands.setweather .. " [0-45]", -1)
        sampAddChatMessage(string.format(script_name.." {FFFFFF}Parametro actual: {dc4747}%d", configData.Weather_time.Weather), -1)
        return
    end

    local weather = tonumber(p)
    
    if weather == nil or weather < 0 or weather > 45 then
        sampAddChatMessage(script_name .. "{FFFFFF} Utilice: {dc4747}" .. configData.commands.setweather .. " [0-45]", -1)
        sampAddChatMessage(string.format(script_name.." {FFFFFF}Parametro actual: {dc4747}%d", configData.Weather_time.Weather), -1)
		return
    end

    if configData.Weather_time.Weather == weather then
        sampAddChatMessage(script_name .. "{FFFFFF} El clima ya esta establecido en: {dc4747}" .. weather, -1)
        return
    end

    configData.Weather_time.Weather = weather
    inicfg.save(configData, configFile)
    forceWeatherNow(weather)
    sampAddChatMessage(string.format(script_name.." {FFFFFF}El clima se ha establecido a: {dc4747}%d", weather), -1)
end

function onResetWeatherCommand()
    configData.Weather_time.Weather = -1
    forceWeatherNow(0)
    inicfg.save(configData, configFile)
    sampAddChatMessage(script_name.." {FFFFFF}El clima ha sido restablecido a la configuracion original.", -1)
end


function onTimeChange(p)
    if p == nil or p == '' then
        sampAddChatMessage(script_name .. "{FFFFFF} Utilice: {dc4747}" .. configData.commands.settime .. " [0-23 - horas]:[0-59 - minutos]", -1)
		sampAddChatMessage(string.format(script_name .. "{FFFFFF} Parametro actual: {dc4747}%02d:%02d", configData.Weather_time.TimeHours, configData.Weather_time.TimeMinutes), -1)
        return
    end

    local k = extract_value(p, ':')
    local r, s
    if k[1] == nil then 
        r = tonumber(p) 
        s = 0
    else 
        r = tonumber(k[1])
        s = tonumber(k[2]) or 0
    end

    if r == nil or r < 0 or r > 23 or s < 0 or s > 59 then
        sampAddChatMessage(script_name .. "{FFFFFF} Utilice: {dc4747}" .. configData.commands.settime .. " [0-23 - horas] [0-59 - minutos]", -1)
        sampAddChatMessage(string.format(script_name .. "{FFFFFF} Parametro actual: {dc4747}%02d:%02d", configData.Weather_time.TimeHours, configData.Weather_time.TimeMinutes), -1)
        return
    end

    if configData.Weather_time.TimeHours == r and configData.Weather_time.TimeMinutes == s then
        sampAddChatMessage(script_name .. "{FFFFFF} La hora ya esta establecida en {dc4747}" .. string.format("%02d:%02d", r, s), -1)
        return
    end

    configData.Weather_time.TimeHours = r
    configData.Weather_time.TimeMinutes = s
    inicfg.save(configData, configFile)
    sampAddChatMessage(string.format(script_name.." {FFFFFF}La Hora ha sido fijada en: {dc4747}%02d:%02d", r, s), -1)
end

function onResetTimeCommand()
    configData.Weather_time.TimeHours = -1
    configData.Weather_time.TimeMinutes = -1
    setTimeOfDay(12, 0)
    inicfg.save(configData, configFile)
    sampAddChatMessage(script_name.." {FFFFFF}La hora ha sido restablecida a la configuracion original", -1)
end

if check_sampev then
    function sampev.onSetWeather(z)
        if configData.Weather_time.Weather ~= -1 then return false end
        return {z}
    end

    function sampev.onSetPlayerTime(r, s)
        if configData.Weather_time.TimeHours ~= -1 then return false end
        return {r, s}
    end
end

---------------------- [Commands]----------------------
local lastRegisteredCommands = {}

function registerCommand(command, callback)
    if command:sub(1, 1) ~= "/" then
        return
    end

    local cmd = command:sub(2)
    if lastRegisteredCommands[cmd] then
        sampUnregisterChatCommand(lastRegisteredCommands[cmd])
    end

    sampRegisterChatCommand(cmd, callback)
    lastRegisteredCommands[cmd] = cmd
end

function reloadCommands()
    for cmd, _ in pairs(lastRegisteredCommands) do
        sampUnregisterChatCommand(cmd)
    end
    lastRegisteredCommands = {}  

    registerCommand(ffi.string(buffers.cmd_openmenu), function()
        toggleMenu[0] = not toggleMenu[0]
    end)

    registerCommand(ffi.string(buffers.cmd_setweather), function(arg)
        onWeatherChange(arg)
    end)

    registerCommand(ffi.string(buffers.cmd_settime), function(arg)
        onTimeChange(arg)
    end)

    registerCommand(ffi.string(buffers.cmd_resettime), function(arg)
        onResetTimeCommand()
    end)

    registerCommand(ffi.string(buffers.cmd_resetweather), function(arg)
        onResetWeatherCommand()
    end)

    registerCommand(ffi.string(buffers.cmd_clearchat), function()
        clearChat()
    end)

    registerCommand(ffi.string(buffers.cmd_togglechat), function()
        toggleChatMessages()
    end)
	if check_sampev then
        function sampev.onServerMessage(color, text)
            return blockChatMessages(color, text)
        end
    end

    registerCommand(ffi.string(buffers.cmd_reconect), function(arg)
        reconnect(arg)
    end)

	registerCommand(ffi.string(buffers.cmd_ugenrl), function()
		checkboxes.ugenrl_enable[0] = not checkboxes.ugenrl_enable[0]
		configData.ugenrl_main.enable = checkboxes.ugenrl_enable[0]
		inicfg.save(configData, configFile)

		local estado = checkboxes.ugenrl_enable[0] and "{73b461}Activado" or "{dc4747}Desactivado"
		sampAddChatMessage(script_name.." {FFFFFF}Ultimate Genrl " .. estado, -1)
	end)

	local soundCommands = {
		{cmd = buffers.cmd_uds, sounds = availableDKSounds, key = "dk", iniKey = "selectedDK"},
		{cmd = buffers.cmd_ums, sounds = availableM4Sounds, key = "m4", iniKey = "selectedM4"},
		{cmd = buffers.cmd_uss, sounds = availableShotgunSounds, key = "shotgun", iniKey = "selectedShotgun"},
		{cmd = buffers.cmd_urs, sounds = availableRifleSounds, key = "rifle", iniKey = "selectedRifle"},
		{cmd = buffers.cmd_umps, sounds = availableMp5Sounds, key = "mp5", iniKey = "selectedMP5"},
		{cmd = buffers.cmd_usmg, sounds = availableSmgSounds, key = "smg", iniKey = "selectedSmg"},
		{cmd = buffers.cmd_uaks, sounds = availableAkSounds, key = "ak", iniKey = "selectedAK"},
		{cmd = buffers.cmd_u9ms, sounds = availableGunSounds, key = "gun", iniKey = "selectedGun"},
		{cmd = buffers.cmd_uedc, sounds = availableEdcSounds, key = "edc", iniKey = "selectedEdc"},
		{cmd = buffers.cmd_usps, sounds = availableSniperSounds, key = "sniper", iniKey = "selectedSniper"},
		{cmd = buffers.cmd_uhits, sounds = availableHitsSounds, key = "hit", iniKey = "selectedHit"},
		{cmd = buffers.cmd_upns, sounds = availablePainSounds, key = "pain", iniKey = "selectedPain"}
	}
	
	for _, data in ipairs(soundCommands) do
		registerCommand(ffi.string(data.cmd), function(cmd)
			local param = cmd:match("%d+")
			changeWeaponSound(param, data.sounds, data.key, ffi.string(data.cmd), data.iniKey)
		end)
	end
	
end

---------------------- [Function main]----------------------
function main()
	local fileName1 = getWorkingDirectory() .. "/GameFixerBETA"
	local fileName2 = getWorkingDirectory() .. "/GameFixer.lua"
	local fileName3 = getWorkingDirectory() .. "/GameFixer 1.0.luac"
	local fileName4 = getWorkingDirectory() .. "/GameFixer 1.0.luac"

	if fileExists(fileName1) or fileExists(fileName2) or fileExists(fileName3) or fileExists(fileName4) then
		print("cargado correctamente")
	else
		print("No renombres el mod")
		thisScript():unload()
	end
	
	while true do
		if isWidgetSwipedRight(WIDGET_RADAR) then
		toggleMenu[0] = not toggleMenu[0]
	end
		
	setSensitivity(configData.main.sensitivity)
	reloadCommands()
	drawCrosshairHook()
	setCameraRenderDistance(sliders.renderdist[0])
		
	---------------------- [weather-time]----------------------
	
	if configData.Weather_time.TimeHours ~= -1 then
		setTimeOfDay(configData.Weather_time.TimeHours, configData.Weather_time.TimeMinutes)
	end
	
	if configData.Weather_time.Weather ~= -1 then
		forceWeatherNow(configData.Weather_time.Weather)
	end
	
	---------------------- [Fov]----------------------
	if sliders.fovValue[0] == 70 then
	is_fov_modified = false
	else
	is_fov_modified = true
	end

	if is_fov_modified then
	if isCurrentCharWeapon(PLAYER_PED, 34) and isCharPlayingAnim(PLAYER_PED, "gun_stand") then
		if isWidgetPressed(WIDGET_ZOOM_IN) then
			zoom_offset = zoom_offset + 6

			if zoom_offset > 10 + sliders.fovValue[0] then
				zoom_offset = 10 + sliders.fovValue[0]
			end
		elseif isWidgetPressed(WIDGET_ZOOM_OUT) then
			zoom_offset = zoom_offset - 4.5

			if zoom_offset < 0 then
				zoom_offset = 0
			end
		end

		cameraSetLerpFov(sliders.fovValue[0] - zoom_offset, sliders.fovValue[0] - zoom_offset, 1000, true)
	else
		reduction_factor = zoom_offset / 10
		zoom_offset = zoom_offset - reduction_factor

		if zoom_offset < 0 then
			zoom_offset = 0
		end

		cameraSetLerpFov(sliders.fovValue[0] - zoom_offset, sliders.fovValue[0] - zoom_offset, 1000, true)
	end
	end

	wait(0)
	end
end

---------------------- [Load fonts/theme]----------------------
imgui.OnInitialize(function()
	faicons.Init()
	theme[colorListNumber[0]+1].change()
	
	local glyph_ranges  = imgui.GetIO().Fonts:GetGlyphRangesDefault()
	local fontPath = getWorkingDirectory() .. "/gamefixer/fonts/"
	
	font1 = imgui.GetIO().Fonts:AddFontFromFileTTF(fontPath .. "sgame.ttf", 30, _, glyph_ranges )
	font2 = imgui.GetIO().Fonts:AddFontFromFileTTF(fontPath .. "Verdana.ttf", 25, _, glyph_ranges )
end)

---------------------- [Menu style windows 11]----------------------
function imgui.BeginWin11Menu(title, isOpen, toggleButton, tabList, selectedTab, menuState, menuWidth, contentWidth, extraFlags)
    imgui.PushStyleVarVec2(imgui.StyleVar.WindowPadding, imgui.ImVec2(0, 0))
    imgui.Begin(title, isOpen, imgui.WindowFlags.NoTitleBar + (extraFlags or 0))

    local windowSize = imgui.GetWindowSize()
    local windowPos = imgui.GetWindowPos()
    local drawList = imgui.GetWindowDrawList()
    local buttonSize = menuWidth - 10

	imgui.SetCursorPos(imgui.ImVec2(menuWidth, menuWidth))
    local menuPos = imgui.GetCursorScreenPos()

	drawList:AddRectFilled(menuPos, imgui.ImVec2(menuPos.x + windowSize.x - menuWidth, menuPos.y + windowSize.y - menuWidth), imgui.GetColorU32Vec4(imgui.GetStyle().Colors[imgui.Col.ChildBg]), imgui.GetStyle().WindowRounding, 9)
    imgui.SetCursorPos(imgui.ImVec2(0, 0))
    local backgroundPos = imgui.GetCursorScreenPos()
	
	drawList:AddRectFilled(backgroundPos, imgui.ImVec2(backgroundPos.x + (menuState[0] and contentWidth or menuWidth), backgroundPos.y + windowSize.y), imgui.GetColorU32Vec4(imgui.GetStyle().Colors[imgui.Col.WindowBg]), imgui.GetStyle().WindowRounding, 5)
    imgui.SetCursorPos(imgui.ImVec2(buttonSize + 10, menuWidth / 2 - imgui.CalcTextSize(title).y / 2))
    imgui.Text(title)
    imgui.SetCursorPosY(5)

    if toggleButton then
        imgui.SetCursorPosX(5)
        if imgui.Button(toggleButton, imgui.ImVec2(buttonSize, buttonSize)) then
            menuState[0] = not menuState[0]
        end
    else
        imgui.SetCursorPosY(25)
    end

    imgui.SetCursorPosX(5)
	
	local orderedTabs = {
        faicons.BARS,
        faicons.WRENCH,
        faicons.ROCKET,
        faicons.MUSIC,
        faicons.KEYBOARD,
        faicons.GEAR
    }

    for _, tabIcon in ipairs(orderedTabs) do
        local tabName = tabList[tabIcon]

        if tabName then 
            imgui.SetCursorPosX(5)
            imgui.PushStyleColor(imgui.Col.Button, selectedTab[0] == tabName 
                and imgui.GetStyle().Colors[imgui.Col.ButtonActive] 
                or imgui.GetStyle().Colors[imgui.Col.WindowBg])

            imgui.PushStyleVarVec2(imgui.StyleVar.ButtonTextAlign, imgui.ImVec2(menuState[0] and 0.1 or 0.5, 0.5))
            imgui.SetWindowFontScale(1.5)

            local iconColor = imgui.GetStyle().Colors[imgui.Col.NavHighlight] or imgui.ImVec4(1.0, 1.0, 1.0, 1.0)
            imgui.PushStyleColor(imgui.Col.Text, iconColor)

            if imgui.Button(menuState[0] and tabIcon .. " " .. tabName or tabIcon, 
                imgui.ImVec2(menuState[0] and contentWidth - 10 or buttonSize, buttonSize)) then
                selectedTab[0] = tabName
            end

            imgui.PopStyleColor()
            imgui.SetWindowFontScale(1.0)
            imgui.PopStyleVar()
            imgui.PopStyleColor()
        end
    end
	imgui.SetCursorPos(imgui.ImVec2(menuWidth, 0))
	imgui.PushStyleVarVec2(imgui.StyleVar.WindowPadding, imgui.ImVec2(15, 15))
	imgui.BeginChild(title .. "::mainchild", imgui.ImVec2(windowSize.x - menuWidth, windowSize.y), true, imgui.WindowFlags.NoScrollbar)

end

---------------------- [Finish script]----------------------
function imgui.EndWin11Menu()
	imgui.EndChild()
	imgui.End()
	imgui.PopStyleVar(2)
end

---------------------- [Themes]----------------------
function apply_monet()
	imgui.SwitchContext()
	local style = imgui.GetStyle()
	local colors = style.Colors
	local clr = imgui.Col
	local ImVec4 = imgui.ImVec4
	local generated_color = monet.buildColors(ini.cfg.color, 1.0, true)
	colors[clr.Text] = ColorAccentsAdapter(generated_color.accent2.color_50):as_vec4()
	colors[clr.TextDisabled] = ColorAccentsAdapter(generated_color.neutral1.color_600):as_vec4()
	colors[clr.WindowBg] = ColorAccentsAdapter(generated_color.accent2.color_900):as_vec4()
	colors[clr.ChildBg] = ColorAccentsAdapter(generated_color.accent2.color_800):as_vec4()
	colors[clr.PopupBg] = ColorAccentsAdapter(generated_color.accent2.color_700):as_vec4()
	colors[clr.Border] = ColorAccentsAdapter(generated_color.accent1.color_200):apply_alpha(0xcc):as_vec4()
	colors[clr.Separator] = ColorAccentsAdapter(generated_color.accent1.color_200):apply_alpha(0xcc):as_vec4()
	colors[clr.BorderShadow] = imgui.ImVec4(0.00, 0.00, 0.00, 0.00)
	colors[clr.FrameBg] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x60):as_vec4()
	colors[clr.FrameBgHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x70):as_vec4()
	colors[clr.FrameBgActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x50):as_vec4()
	colors[clr.TitleBg] = ColorAccentsAdapter(generated_color.accent2.color_700):apply_alpha(0xcc):as_vec4()
	colors[clr.TitleBgCollapsed] = ColorAccentsAdapter(generated_color.accent2.color_700):apply_alpha(0x7f):as_vec4()
	colors[clr.TitleBgActive] = ColorAccentsAdapter(generated_color.accent2.color_700):as_vec4()
	colors[clr.MenuBarBg] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x91):as_vec4()
	colors[clr.ScrollbarBg] = imgui.ImVec4(0,0,0,0)
	colors[clr.ScrollbarGrab] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x85):as_vec4()
	colors[clr.ScrollbarGrabHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.ScrollbarGrabActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xb3):as_vec4()
	colors[clr.CheckMark] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xcc):as_vec4()
	colors[clr.SliderGrab] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xcc):as_vec4()
	colors[clr.SliderGrabActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0x80):as_vec4()
	colors[clr.Button] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xcc):as_vec4()
	colors[clr.ButtonHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.ButtonActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xb3):as_vec4()
	colors[clr.Tab] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xcc):as_vec4()
	colors[clr.TabActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xb3):as_vec4()
	colors[clr.TabHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.Header] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xcc):as_vec4()
	colors[clr.HeaderHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.HeaderActive] = ColorAccentsAdapter(generated_color.accent1.color_600):apply_alpha(0xb3):as_vec4()
	colors[clr.ResizeGrip] = ColorAccentsAdapter(generated_color.accent2.color_700):apply_alpha(0xcc):as_vec4()
	colors[clr.ResizeGripHovered] = ColorAccentsAdapter(generated_color.accent2.color_700):as_vec4()
	colors[clr.ResizeGripActive] = ColorAccentsAdapter(generated_color.accent2.color_700):apply_alpha(0xb3):as_vec4()
	colors[clr.PlotLines] = ColorAccentsAdapter(generated_color.accent2.color_600):as_vec4()
	colors[clr.PlotLinesHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.PlotHistogram] = ColorAccentsAdapter(generated_color.accent2.color_600):as_vec4()
	colors[clr.PlotHistogramHovered] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.TextSelectedBg] = ColorAccentsAdapter(generated_color.accent1.color_600):as_vec4()
	colors[clr.ModalWindowDimBg] = ColorAccentsAdapter(generated_color.accent1.color_200):apply_alpha(0x26):as_vec4()
end

function applyBaseStyles()
	local style = imgui.GetStyle()
    style.WindowPadding = imgui.ImVec2(8, 8)
    style.WindowRounding = 4
    style.WindowBorderSize = 1
    style.WindowMinSize = imgui.ImVec2(32, 32)
    style.WindowTitleAlign = imgui.ImVec2(0, 0.5)
    style.ChildRounding = 4
    style.ChildBorderSize = 1
    style.PopupRounding = 2
    style.PopupBorderSize = 1
    style.FramePadding = imgui.ImVec2(6, 4)
    style.FrameRounding = 2
    style.FrameBorderSize = 0
    style.ItemSpacing = imgui.ImVec2(8, 4)
    style.ItemInnerSpacing = imgui.ImVec2(4, 4)
    style.IndentSpacing = 21
    style.ScrollbarSize = 20
    style.ScrollbarRounding = 4
    style.GrabMinSize = 20
    style.GrabRounding = 2
    style.TabRounding = 2
    style.ButtonTextAlign = imgui.ImVec2(0.5, 0.5)
    style.SelectableTextAlign = imgui.ImVec2(0, 0)
end

function applyModernStyles()
    local style = imgui.GetStyle()
	style.WindowPadding = imgui.ImVec2(14, 14)
	style.WindowRounding = 12
	style.WindowBorderSize = 3
	style.WindowMinSize = imgui.ImVec2(50, 50)
	style.WindowTitleAlign = imgui.ImVec2(0.5, 0.5)
	style.ChildRounding = 10
	style.ChildBorderSize = 2
	style.PopupRounding = 10
	style.PopupBorderSize = 2
	style.FramePadding = imgui.ImVec2(12, 9)
	style.FrameRounding = 8
	style.FrameBorderSize = 0
	style.ItemSpacing = imgui.ImVec2(14, 5)
	style.ItemInnerSpacing = imgui.ImVec2(8, 8)
	style.IndentSpacing = 30
	style.ScrollbarSize = 18
	style.ScrollbarRounding = 10
	style.GrabMinSize = 20
	style.GrabRounding = 10
	style.TabRounding = 10
	style.ButtonTextAlign = imgui.ImVec2(0.5, 0.5)
	style.SelectableTextAlign = imgui.ImVec2(0, 0)
end

theme = {
	------------ [RED] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
			
			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.8, 0.5, 0.5, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.15, 0, 0, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.2, 0.05, 0.05, 0.3)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.25, 0.07, 0.07, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.8, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0.4, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.6, 0.15, 0.15, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.7, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.8, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.3, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.2, 0.05, 0.05, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.3, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.2, 0.05, 0.05, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.7, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(0.8, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.8, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.9, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.7, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.7, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.7, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.6, 0.2, 0.2, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.7, 0.3, 0.3, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.6, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.7, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.8, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.3, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.5, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.8, 0.3, 0.3, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.8, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)			
		end		
	},
	------------ [GREEN] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
		
			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.6, 0.6, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.02, 0.15, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.02, 0.2, 0.02, 0.3)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.02, 0.25, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.1, 0.8, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0.05, 0.05, 0.05, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.1, 0.4, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.2, 0.9, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.05, 0.3, 0.05, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.1, 0.9, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.02, 0.2, 0.02, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.02, 0.2, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.02, 0.1, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.1, 0.6, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.2, 0.8, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(0.4, 1.0, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.1, 0.8, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.05, 0.9, 0.05, 1)
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.1, 0.8, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.1, 0.8, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.1, 0.8, 0.1, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.2, 1.0, 0.2, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.1, 0.8, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.3, 1.0, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.02, 0.2, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.1, 0.4, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.2, 1.0, 0.2, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.2, 1.0, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)
		end		
	},
	------------ [BLUE] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()

			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.6, 0.6, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.02, 0.2, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.05, 0.3, 0.5, 0.3)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.07, 0.35, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.2, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0.1, 0.2, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.15, 0.5, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.2, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.05, 0.3, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.1, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.02, 0.25, 0.5, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.05, 0.3, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.02, 0.2, 0.5, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.2, 0.5, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.3, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.4, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.3, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.2, 0.5, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.2, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.35, 0.75, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.2, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.35, 0.75, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.2, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.35, 0.75, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.2, 0.6, 1.0, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.3, 0.7, 1.0, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.35, 0.75, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.2, 0.6, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.35, 0.75, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.05, 0.3, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.1, 0.4, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.3, 0.7, 1.0, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.3, 0.7, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)
		end		
	},
	------------ [DARK] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
		
			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.6, 0.6, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.02, 0.02, 0.02, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.1, 0.1, 0.1, 0.5)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.12, 0.12, 0.12, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.3, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0, 0, 0, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.18, 0.18, 0.18, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.25, 0.25, 0.25, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.08, 0.08, 0.08, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.15, 0.15, 0.15, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.08, 0.08, 0.08, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.1, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.05, 0.05, 0.05, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.2, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.3, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.5, 0.5, 0.5, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.7, 0.7, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.2, 0.2, 0.2, 1.00)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.4, 0.4, 0.4, 1.00)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.8, 0.8, 0.8, 1.00)			
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.2, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.3, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.2, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.3, 0.3, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.2, 0.2, 0.2, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.3, 0.3, 0.3, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.15, 0.15, 0.15, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.25, 0.25, 0.25, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.35, 0.35, 0.35, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.1, 0.1, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.2, 0.2, 0.2, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.4, 0.4, 0.4, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.4, 0.4, 0.4, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)
		end		
	},
	------------ [CIAN] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
		
			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.6, 0.6, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.02, 0.1, 0.12, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.05, 0.15, 0.18, 0.5)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.08, 0.2, 0.22, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0, 0, 0, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.1, 0.4, 0.45, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.15, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.02, 0.12, 0.15, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.02, 0.12, 0.15, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.02, 0.15, 0.18, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.02, 0.1, 0.12, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.15, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.2, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.15, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.2, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.3, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.3, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.1, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.3, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.1, 0.6, 0.7, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.2, 0.7, 0.8, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.3, 0.8, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.08, 0.5, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.15, 0.6, 0.7, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.1, 0.4, 0.5, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.15, 0.5, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.2, 0.7, 0.8, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.2, 0.7, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)
		end		
	},
	------------ [NEON] ------------
	{
		change = function()
			local ImVec4 = imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
		
			imgui.GetStyle().Colors[imgui.Col.Text] = ImVec4(1, 1, 1, 1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled] = ImVec4(0.6, 0.6, 0.6, 1)
			imgui.GetStyle().Colors[imgui.Col.WindowBg] = ImVec4(0.05, 0.02, 0.1, 1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg] = ImVec4(0.1, 0.03, 0.2, 0.3)
			imgui.GetStyle().Colors[imgui.Col.PopupBg] = ImVec4(0.12, 0.04, 0.25, 1)
			imgui.GetStyle().Colors[imgui.Col.Border] = ImVec4(0.5, 0.1, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.BorderShadow] = ImVec4(0.2, 0.05, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg] = ImVec4(0.4, 0.1, 0.8, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBg] = ImVec4(0.2, 0.05, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgActive] = ImVec4(0.4, 0.1, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.TitleBgCollapsed] = ImVec4(0.1, 0.03, 0.2, 0.5)
			imgui.GetStyle().Colors[imgui.Col.MenuBarBg] = ImVec4(0.15, 0.04, 0.35, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarBg] = ImVec4(0.08, 0.02, 0.25, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab] = ImVec4(0.4, 0.1, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive] = ImVec4(0.5, 0.15, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Button] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive] = ImVec4(0.8, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Header] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive] = ImVec4(0.8, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Separator] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorHovered] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.SeparatorActive] = ImVec4(0.8, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip] = ImVec4(0.6, 0.2, 1.0, 0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered] = ImVec4(0.7, 0.3, 1.0, 0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive] = ImVec4(0.8, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.Tab] = ImVec4(0.6, 0.2, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabActive] = ImVec4(0.8, 0.4, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocused] = ImVec4(0.2, 0.05, 0.3, 1)
			imgui.GetStyle().Colors[imgui.Col.TabUnfocusedActive] = ImVec4(0.4, 0.1, 0.9, 1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg] = ImVec4(0.7, 0.3, 1.0, 0.5)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget] = ImVec4(0.7, 0.3, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight] = ImVec4(1.0, 1.0, 1.0, 1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg] = ImVec4(0, 0, 0, 0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg] = ImVec4(0, 0, 0, 0.33)
		end			
	},
	------------ [Cyberpunk] ------------
	{
		change=function()
			local ImVec4=imgui.ImVec4
			imgui.SwitchContext()
			applyModernStyles()
			imgui.GetStyle().Colors[imgui.Col.WindowBg]=ImVec4(0.05,0.05,0,1)
			imgui.GetStyle().Colors[imgui.Col.ChildBg]=ImVec4(0.1,0.1,0,0.3)
			imgui.GetStyle().Colors[imgui.Col.PopupBg]=ImVec4(0.12,0.12,0,1)
			imgui.GetStyle().Colors[imgui.Col.Text]=ImVec4(1,1,1,1)
			imgui.GetStyle().Colors[imgui.Col.TextDisabled]=ImVec4(0.7,0.7,0,1)
			imgui.GetStyle().Colors[imgui.Col.Border]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.Separator]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.FrameBg]=ImVec4(0.3,0.3,0,1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgHovered]=ImVec4(0.5,0.5,0,1)
			imgui.GetStyle().Colors[imgui.Col.FrameBgActive]=ImVec4(0.7,0.7,0,1)
			imgui.GetStyle().Colors[imgui.Col.Header]=ImVec4(0.5,0.5,0,1)
			imgui.GetStyle().Colors[imgui.Col.HeaderHovered]=ImVec4(0.6,0.6,0,1)
			imgui.GetStyle().Colors[imgui.Col.HeaderActive]=ImVec4(0.7,0.7,0,1)
			imgui.GetStyle().Colors[imgui.Col.Button]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.ButtonHovered]=ImVec4(1,0.9,0,1)
			imgui.GetStyle().Colors[imgui.Col.ButtonActive]=ImVec4(1,0.8,0,1)
			imgui.GetStyle().Colors[imgui.Col.Tab]=ImVec4(0.5,0.5,0,1)
			imgui.GetStyle().Colors[imgui.Col.TabHovered]=ImVec4(0.6,0.6,0,1)
			imgui.GetStyle().Colors[imgui.Col.TabActive]=ImVec4(0.7,0.7,0,1)
			imgui.GetStyle().Colors[imgui.Col.CheckMark]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrab]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.SliderGrabActive]=ImVec4(1,0.9,0,1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrab]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabHovered]=ImVec4(1,0.9,0,1)
			imgui.GetStyle().Colors[imgui.Col.ScrollbarGrabActive]=ImVec4(1,0.8,0,1)
			imgui.GetStyle().Colors[imgui.Col.ResizeGrip]=ImVec4(1,1,0,0.7)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripHovered]=ImVec4(1,0.9,0,0.8)
			imgui.GetStyle().Colors[imgui.Col.ResizeGripActive]=ImVec4(1,0.8,0,1)
			imgui.GetStyle().Colors[imgui.Col.TextSelectedBg]=ImVec4(1,1,0,0.3)
			imgui.GetStyle().Colors[imgui.Col.DragDropTarget]=ImVec4(1,1,0,1)
			imgui.GetStyle().Colors[imgui.Col.NavHighlight]=ImVec4(1,1,0.5,1)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingHighlight]=ImVec4(0,0,0,0.33)
			imgui.GetStyle().Colors[imgui.Col.NavWindowingDimBg]=ImVec4(0,0,0,0.33)
			imgui.GetStyle().Colors[imgui.Col.ModalWindowDimBg]=ImVec4(0,0,0,0.33)
		end		
	},

	{
		change = function()
			apply_monet()
		end
	},
}
