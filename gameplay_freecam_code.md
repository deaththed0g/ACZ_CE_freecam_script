```asm
[ENABLE]
//Gameplay freecam by dead_d0g (death_the_d0g @ Twitter/Github)
//A "freecam" script for ACZ.
```

{$lua}

```lua
--//Set search AoB and code-disabling camera function

function gameplayCamControl(toggle)
	if toggle then
		local s = [[
		GAMEPLAY_CAM_CONTROL + 0xC0:
		db 90 90 90
		luacall(playSound(findTableFile('Activate')))
		]]
		autoAssemble(s)
	end

	if not toggle then
		local s = [[
		GAMEPLAY_CAM_CONTROL + 0xC0:
		db 0F 29 11
		unregistersymbol(GAMEPLAY_CAM_CONTROL)
		]]
		autoAssemble(s)
	end
end

search_aob = [[
aobscan(GAMEPLAY_CAM_CONTROL, 0F 28 32 0F 29 31 BA ?0 ?? ?? 0? 8B 0D ?0 A? ?? 0? 83 C1 60 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 39 0F 29 3A BA ?0 ?? ?? 0? 8B 0D ?0 A? ?? 0? 81 C1 90 00 00 00 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 02 0F 29 01 BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 09 0F 29 0A BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 12 0F 29 11 8B 0D ?0 ?? ?? 0? 81 C1 4E 01 00 00 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F B6 01) // should be unique
registersymbol(GAMEPLAY_CAM_CONTROL)
]]

--//Get process ID of any active PCSX2 instance then pause it

pcsx2_emu_id = getOpenedProcessID()
pause(pcsx2_emu_id)

--//Run search AoB. If successful then:
--//run he camera code disabler
--//disable control input
--//set camera control speed rates
--//set hotkeys and store previous values
--//fix right-stick analog issue when disabling the script

if autoAssemble(search_aob) then

	--[[Toggle code]]
	gameplayCamControl(true)

	--[[Set global hotkey delay]]
	setGlobalKeyPollInterval(0)

	--[[Disable controller input]]
	writeBytes(0x203F70BC, 00, 00, 00 ,00)

	--[[Set Pause game flag]]
	writeBytes(0x207651E8, 05)

	--[[Disable non-pause HUD]]
	writeBytes(0x203FFBCF, 00)

	--[[
	Set movement rates here. The higher the value the faster the camera will move.
	This parameter uses floating point numbers only.
	]]

	move_cam_rate = 5.5 --- X,Z,Y coordinates movement speed
	move_analog_cam_rate = 0.098125 --- PITCH/YAW/ROLL movement speed

	--[[Start search for the camera coordinates]]

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "00 00 ?? 44 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 ?? ?? ?? 00 00 00 00 00 02 C0 01 00 00 80 3F FF FF 7F 4B 00 00 00 00 00 02 C0 01 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 3F ?? ?? ?? 43 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ?? ?? ?? ?? 00 00 00 00 00 00 00 00 00 00 00 C5 00 00 00 C5 ?? ?? ?? ?? 00 00 80 BF 00 00 00 00 00 00 00 00 ?? ?? ?? ?? 00 00 00 00 ?? ?? ?? 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ?? ?? ?? ?? 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ?? ?? 80 BF 00 00 80 BF 00 00 00 00 00 00 00 00 ?? ?? ?? ?? 00 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 00 45 00 00 00 45 CD CC ?? ?? ?? ?? ?? 3F 03 00 00 00 ?? ?? ?? ?? 01 00 00 ?? ?? ?? ?? ?? 00 00 00 15 00 01 ?? ?? ?? ?? ?? ?? ?? ?? ?? ?? 00 00 ?? 44 00 00 ?? ??" , nil, 0x20800000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]
	if (fl~=nil) then
		al = getAddressList()

		header = al.createMemoryRecord()

		header.Description = "GAMEPLAY freecam script"
		header.isGroupHeader = true

		for i = 1, 1 do
			base_address = getAddress(fl[0])

			tps_cam_xpos1 = al.createMemoryRecord() ---///third person view camera
			tps_cam_ypos1 = al.createMemoryRecord()
			tps_cam_zpos1 = al.createMemoryRecord()
			tps_cam_xpos2 = al.createMemoryRecord()
			tps_cam_ypos2 = al.createMemoryRecord()
			tps_cam_zpos2 = al.createMemoryRecord()

			as_cam_pitch = al.createMemoryRecord() ---///analog stick movement
			as_cam_yaw = al.createMemoryRecord()
			as_cam_roll = al.createMemoryRecord()

			p_state = al.createMemoryRecord()
			c_state = al.createMemoryRecord()
			h_state = al.createMemoryRecord()

			third_person_view_camera_xpos1 = base_address + 0xB30
			third_person_view_camera_ypos1 = base_address + 0xB34
			third_person_view_camera_zpos1 = base_address + 0xB38
			third_person_view_camera_xpos2 = base_address + 0xB3C
			third_person_view_camera_ypos2 = base_address + 0xB40
			third_person_view_camera_zpos2 = base_address + 0xB44

			pause_state = 0x207651E8
			controller_state = 0x203F70BC
			hud_state = 0x203FFBCF

			analog_stick_camera_movement_pitch = base_address + 0xC20
			analog_stick_camera_movement_yaw = base_address + 0xC24
			analog_stick_camera_movement_roll = base_address + 0xC28

			third_person_view_camera_xpos1_old = readFloat(third_person_view_camera_xpos1)
			third_person_view_camera_ypos1_old = readFloat(third_person_view_camera_ypos1)
			third_person_view_camera_zpos1_old = readFloat(third_person_view_camera_zpos1)

			third_person_view_camera_xpos2_old = readFloat(third_person_view_camera_xpos2)
			third_person_view_camera_ypos2_old = readFloat(third_person_view_camera_ypos2)
			third_person_view_camera_zpos2_old = readFloat(third_person_view_camera_zpos2)

			tps_cam_xpos1.Description = "tpscam_player_axis_x"
			tps_cam_xpos1.setAddress(third_person_view_camera_xpos1)
			tps_cam_xpos1.Type = vtSingle
			tps_cam_xpos1.appendToEntry(header)

			tps_cam_ypos1.Description = "tpscam_player_axis_y"
			tps_cam_ypos1.setAddress(third_person_view_camera_ypos1)
			tps_cam_ypos1.Type = vtSingle
			tps_cam_ypos1.appendToEntry(header)

			tps_cam_zpos1.Description = "tpscam_player_axis_zoom"
			tps_cam_zpos1.setAddress(third_person_view_camera_zpos1)
			tps_cam_zpos1.Type = vtSingle
			tps_cam_zpos1.appendToEntry(header)

			tps_cam_xpos2.Description = "tpscam_camera_axis_x"
			tps_cam_xpos2.setAddress(third_person_view_camera_xpos2)
			tps_cam_xpos2.Type = vtSingle
			tps_cam_xpos2.appendToEntry(header)

			tps_cam_ypos2.Description = "tpscam_camera_axis_y"
			tps_cam_ypos2.setAddress(third_person_view_camera_ypos2)
			tps_cam_ypos2.Type = vtSingle
			tps_cam_ypos2.appendToEntry(header)

			tps_cam_zpos2.Description = "tpscam_camera_axis_zoom"
			tps_cam_zpos2.setAddress(third_person_view_camera_zpos2)
			tps_cam_zpos2.Type = vtSingle
			tps_cam_zpos2.appendToEntry(header)

			as_cam_pitch.Description = "Analog stick camera pitch"
			as_cam_pitch.setAddress(analog_stick_camera_movement_pitch)
			as_cam_pitch.Type = vtSingle
			as_cam_pitch.appendToEntry(header)

			as_cam_yaw.Description = "Analog stick camera yaw"
			as_cam_yaw.setAddress(analog_stick_camera_movement_yaw)
			as_cam_yaw.Type = vtSingle
			as_cam_yaw.appendToEntry(header)

			as_cam_roll.Description = "Analog stick camera roll"
			as_cam_roll.setAddress(analog_stick_camera_movement_roll)
			as_cam_roll.Type = vtSingle
			as_cam_roll.appendToEntry(header)

			p_state.Description = "Pause state"
			p_state.setAddress(pause_state)
			p_state.Type = vtByte
			p_state.appendToEntry(header)

			c_state.Description = "[DEBUG] Controller input state"
			c_state.setAddress(controller_state)
			c_state.Aob.Size = 04
			c_state.ShowAsHex = true
			c_state.appendToEntry(header)

			h_state.Description = "HUD state"
			h_state.setAddress(hud_state)
			h_state.Type = vtByte
			h_state.appendToEntry(header)

			--[[Set hotkeys]]

			mrhk_tps_cam_xpos1 = al.getMemoryRecordByDescription("tpscam_player_axis_x")
			mrhk_tps_cam_ypos1 = al.getMemoryRecordByDescription("tpscam_player_axis_y")
			mrhk_tps_cam_zpos1 = al.getMemoryRecordByDescription("tpscam_player_axis_zoom")

			mrhk_tps_cam_xpos2 = al.getMemoryRecordByDescription("tpscam_camera_axis_x")
			mrhk_tps_cam_ypos2 = al.getMemoryRecordByDescription("tpscam_camera_axis_y")
			mrhk_tps_cam_zpos2 = al.getMemoryRecordByDescription("tpscam_camera_axis_zoom")

			mrhk_as_cam_pitch = al.getMemoryRecordByDescription("Analog stick camera pitch")
			mrhk_as_cam_yaw = al.getMemoryRecordByDescription("Analog stick camera yaw")
			mrhk_as_cam_roll = al.getMemoryRecordByDescription("Analog stick camera roll")

			key1 = mrhk_tps_cam_xpos1.createHotkey({VK_A},mrhIncreaseValue,move_cam_rate,"tpscam_player_axis_x")
			key2 = mrhk_tps_cam_xpos1.createHotkey({VK_D},mrhDecreaseValue,move_cam_rate,"tpscam_player_axis_x")
			key1 = mrhk_tps_cam_ypos1.createHotkey({VK_S},mrhIncreaseValue,move_cam_rate,"tpscam_player_axis_y")
			key2 = mrhk_tps_cam_ypos1.createHotkey({VK_W},mrhDecreaseValue,move_cam_rate,"tpscam_player_axis_y")
			key1 = mrhk_tps_cam_zpos1.createHotkey({VK_E},mrhIncreaseValue,move_cam_rate,"tpscam_player_axis_zoom")
			key2 = mrhk_tps_cam_zpos1.createHotkey({VK_Q},mrhDecreaseValue,move_cam_rate,"tpscam_player_axis_zoom")

			key1 = mrhk_tps_cam_xpos2.createHotkey({VK_F},mrhIncreaseValue,move_cam_rate,"tpscam_camera_axis_x")
			key2 = mrhk_tps_cam_xpos2.createHotkey({VK_H},mrhDecreaseValue,move_cam_rate,"tpscam_camera_axis_x")
			key1 = mrhk_tps_cam_ypos2.createHotkey({VK_G},mrhIncreaseValue,move_cam_rate,"tpscam_camera_axis_y")
			key2 = mrhk_tps_cam_ypos2.createHotkey({VK_T},mrhDecreaseValue,move_cam_rate,"tpscam_camera_axis_y")
			key1 = mrhk_tps_cam_zpos2.createHotkey({VK_Y},mrhIncreaseValue,move_cam_rate,"tpscam_camera_axis_zoom")
			key2 = mrhk_tps_cam_zpos2.createHotkey({VK_R},mrhDecreaseValue,move_cam_rate,"tpscam_camera_axis_zoom")

			key1 = mrhk_as_cam_pitch.createHotkey({VK_K},mrhIncreaseValue,move_analog_cam_rate,"Analog stick camera pitch")
			key2 = mrhk_as_cam_pitch.createHotkey({VK_I},mrhDecreaseValue,move_analog_cam_rate,"Analog stick camera pitch")
			key1 = mrhk_as_cam_yaw.createHotkey({VK_L},mrhIncreaseValue,move_analog_cam_rate,"Analog stick camera yaw")
			key2 = mrhk_as_cam_yaw.createHotkey({VK_J},mrhDecreaseValue,move_analog_cam_rate,"Analog stick camera yaw")
			key1 = mrhk_as_cam_roll.createHotkey({VK_O},mrhIncreaseValue,move_analog_cam_rate,"Analog stick camera roll")
			key2 = mrhk_as_cam_roll.createHotkey({VK_U},mrhDecreaseValue,move_analog_cam_rate,"Analog stick camera roll")

			--[[Panic key]]

			key3 = mrhk_tps_cam_xpos1.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_xpos1_old,"tpscam_player_axis_x")
			key3 = mrhk_tps_cam_ypos1.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_ypos1_old,"tpscam_player_axis_y")
			key3 = mrhk_tps_cam_zpos1.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_zpos1_old,"tpscam_player_axis_zoom")
			key3 = mrhk_tps_cam_xpos2.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_xpos2_old,"tpscam_camera_axis_x")
			key3 = mrhk_tps_cam_ypos2.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_ypos2_old,"tpscam_camera_axis_y")
			key3 = mrhk_tps_cam_zpos2.createHotkey({VK_SPACE},mrhSetValue,third_person_view_camera_zpos2_old,"tpscam_camera_axis_zoom")
      
			key3 = mrhk_as_cam_pitch.createHotkey({VK_SPACE},mrhSetValue,0,"Analog stick camera pitch")
      key3 = mrhk_as_cam_yaw.createHotkey({VK_SPACE},mrhSetValue,0,"Analog stick camera yaw")
			key3 = mrhk_as_cam_roll.createHotkey({VK_SPACE},mrhSetValue,0,"Analog stick camera roll")



		end
		fl.destroy()
		fl=nil
	else
		ms.destroy()
	end
	ms.destroy()

	--[[Disable menu and its HUD graphics]]
	--//Start search for the Pause menu transparency value

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "B0 C5 3C 00 01 01 00 ??", nil, 0x20900000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]
	if (fl~=nil) then
		al = getAddressList()

		for i = 1, 1 do
			zaddress = getAddress(fl[0])
			mntrsnprncy = al.createMemoryRecord()

			menutransparency = zaddress + 0x4

			writeBytes(menutransparency, 00)

			mntrsnprncy.Description = "Pause menu transparency"
			mntrsnprncy.setAddress(menutransparency)
			mntrsnprncy.Type = vtByte
			mntrsnprncy.appendToEntry(header)
		end
		fl.destroy()
		fl=nil
	else
		ms.destroy()
	end
	ms.destroy()

	--[[Right-stick analog fix]]
	--//This fixes an issue with the right-analog stick (the stick that control the camera in-game)
	--//becoming unresponsive when disabling the freecam script

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "3C 00 0? 0? ?? ?? FF FF 00 00 00 ?? ?? ?? ?? ?? ?? ?? ?? ??" , nil, 0x20800000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()
	if (fl~=nil) then
		al = getAddressList()
		for i = 1, 1 do
			xaddress = getAddress(fl[0])
			anlg_fix = al.createMemoryRecord()

			analogfix = xaddress + 0x3

			anlg_fix.Description = "Right-stick analog fix"
			anlg_fix.setAddress(analogfix)
			anlg_fix.Type = vtByte
			anlg_fix.appendToEntry(header)

		end
		fl.destroy()
		fl=nil
	else
		ms.destroy()
	end
	ms.destroy()

	--//Unpause the emulator once the search is done, the values are set and the code was successfully injected
	unpause(pcsx2_emu_id)

else
	--//If search was a failure then print a error message and unpause the emulator
	print("Unable to run the script.")
    print("The script will not work if you loaded a savestate.")
    print("If you haven't used any savestates but you're still getting this message please contact me.")
	unpause(pcsx2_emu_id)
end
```
```asm
{$asm}

[DISABLE]
```
```lua
{$lua}
--//Restore values to default

--[[Restore controller input]]
writeBytes(0x203F70BC, 192, 125, 105 ,0)

--[[Restore pause menu visibility]]
writeBytes(menutransparency, 01)

--[[Restore non-pause HUD visibility]]
writeBytes(0x203FFBCF, 01)

--[[Restore default camera position values]]
writeFloat(third_person_view_camera_xpos1, third_person_view_camera_xpos1_old)
writeFloat(third_person_view_camera_ypos1, third_person_view_camera_ypos1_old)
writeFloat(third_person_view_camera_zpos1, third_person_view_camera_zpos1_old)
writeFloat(third_person_view_camera_xpos2, third_person_view_camera_xpos2_old)
writeFloat(third_person_view_camera_ypos2, third_person_view_camera_ypos2_old)
writeFloat(third_person_view_camera_zpos2, third_person_view_camera_zpos2_old)

--[[Right-analog stick fix]]
writeBytes(analogfix, 04)

--[[Unpause game]]
writeBytes(0x207651E8, 04)

--Destroy entries
header.destroy()

--restored previously NOP'd code
gameplayCamControl(false)
```
{$asm}
