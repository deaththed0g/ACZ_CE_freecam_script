[ENABLE]
//Hangar freecam by dead_d0g (death_the_d0g @ twitter)

{$lua}

--//Set search AoB and code-disabling camera function

function gameplayCamControl(toggle)
	if toggle then
		local s = [[
		HANGAR_CAM_CONTROL_PYR + 0x5D :
		db 90 90 90
		HANGAR_CAM_CONTROL_PYR + 0x12E :
		db 90 90 90
		luacall(playSound(findTableFile('Activate')))
		]]
		autoAssemble(s)
	end

	if not toggle then
		local s = [[
		HANGAR_CAM_CONTROL_PYR + 0x5D :
		db 0F 29 11
		HANGAR_CAM_CONTROL_PYR + 0x12E :
		db 0F 29 31
		unregistersymbol(HANGAR_CAM_CONTROL_PYR)
		]]
		autoAssemble(s)
	end
end

search_aob = [[
aobscan(HANGAR_CAM_CONTROL_PYR, 0F 28 02 0F 29 01 BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 09 0F 29 0A BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 12 0F 29 11 BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 81 C1 D0 01 00 00 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 19 0F 29 1A A1 ?0 ?? ?? 0? 83 C0 40 99 A3 ?0 ?? ?? 0? 89 15 ?4 ?? ?? 0? BA ?0 ?? ?? 0? 8B 0D ?0 A? ?? 0? 83 C1 70 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 22 0F 29 21 BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 29 0F 29 2A BA ?0 ?? ?? 0? 8B 0D ?0 ?? ?? 0? 83 E1 F0 89 C8 C1 E8 0C 8B 04 85 30 ?0 ?? ?? BB ?? ?? ?? 30 01 C1 0F 88 ?? ?? ?? D? 0F 28 32 0F 29 31)
registersymbol(HANGAR_CAM_CONTROL_PYR)
]]

--//Get process ID of any active PCSX2 instance then pause it

pcsx2_emu_id = getOpenedProcessID()
pause(pcsx2_emu_id)

--//Run search AoB. If successful then:
--//run he camera code disabler
--//disable control input
--//set camera control speed rates
--//disable hangar HUD graphics


if autoAssemble(search_aob) then

	--[[Toggle code]]
	gameplayCamControl(true)

	 --[[Set global hotkey delay]]
	setGlobalKeyPollInterval(0)

	 --[[Disable controller input]]
	WriteBytes(0x203F70B8, 00, 00, 00 ,00)

	--[[
	Set movement rates here. The higher the value the faster the camera will move.
	This parameter uses floating point numbers only.
	TODO:Find a way to implement an input box (as CE supports them) so the user can input their speed values
	on the fly and avoid the hassle of editing the script every time they want to modify them.
	]]
	move_cam_rate = 5.5 --- X,Z,Y coordinates movement speed
	move_analog_cam_rate = 0.098125 --- PITCH/YAW/ROLL movement speed

	--[[Start search for the camera coordinates]]
	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "00 00 20 44 00 00 ?? 43 00 00 00 44 00 00 80 3F" , nil, 0x20700000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]
	if (fl~=nil) then
		al = getAddressList()
		header = al.createMemoryRecord()

		header.Description = "Hangar freecam values"
		header.isGroupHeader = true
		for i = 1, 1 do
			base_address = getAddress(fl[0])

			cont_inpt_dbg = al.createMemoryRecord()
			hngr_cam_xpos = al.createMemoryRecord()
			hngr_cam_ypos = al.createMemoryRecord()
			hngr_cam_zpos = al.createMemoryRecord()
			hngr_cam_p = al.createMemoryRecord()
			hngr_cam_y = al.createMemoryRecord()
			hngr_cam_r = al.createMemoryRecord()
			hngr_enty_vis = al.createMemoryRecord()
			hngr_plcmnt = al.createMemoryRecord()

			controller_input_status = 0x203F70B8
			hangar_camera_xpos = base_address + 0x30
			hangar_camera_ypos = base_address + 0x34
			hangar_camera_zpos = base_address + 0x38
			hangar_camera_pitch = base_address + 0x40
			hangar_camera_yaw = base_address + 0x44
			hangar_camera_roll = base_address + 0x48
			hangar_aircraft_placement_type = base_address + 0x1826
			hangar_aircraft_placement_type_old = readBytes(hangar_aircraft_placement_type)
			hangar_entity_visibility = base_address + 0x1827


			cont_inpt_dbg.Description = "[debug] HANGAR controller input status"
			cont_inpt_dbg.setAddress(controller_input_status)
			cont_inpt_dbg.Aob.Size = 04
			cont_inpt_dbg.ShowAsHex = true
			cont_inpt_dbg.appendToEntry(header)

			hngr_cam_xpos.Description = "HANGAR camera X pos"
			hngr_cam_xpos.setAddress(hangar_camera_xpos)
			hngr_cam_xpos.Type = vtSingle
			hngr_cam_xpos.appendToEntry(header)

			hngr_cam_zpos.Description = "HANGAR camera Z pos"
			hngr_cam_zpos.setAddress(hangar_camera_zpos)
			hngr_cam_zpos.Type = vtSingle
			hngr_cam_zpos.appendToEntry(header)

			hngr_cam_ypos.Description = "HANGAR camera Y pos"
			hngr_cam_ypos.setAddress(hangar_camera_ypos)
			hngr_cam_ypos.Type = vtSingle
			hngr_cam_ypos.appendToEntry(header)

			hngr_cam_p.Description = "HANGAR camera pitch"
			hngr_cam_p.setAddress(hangar_camera_pitch)
			hngr_cam_p.Type = vtSingle
			hngr_cam_p.appendToEntry(header)

			hngr_cam_y.Description = "HANGAR camera yaw"
			hngr_cam_y.setAddress(hangar_camera_yaw)
			hngr_cam_y.Type = vtSingle
			hngr_cam_y.appendToEntry(header)

			hngr_cam_r.Description = "HANGAR camera roll"
			hngr_cam_r.setAddress(hangar_camera_roll)
			hngr_cam_r.Type = vtSingle
			hngr_cam_r.appendToEntry(header)

			hngr_plcmnt.Description = "HANGAR aircraft placement type"
			hngr_plcmnt.setAddress(hangar_aircraft_placement_type)
			hngr_plcmnt.Type = vtByte
			hngr_plcmnt.appendToEntry(header)

			hngr_enty_vis.Description = "HANGAR entity visiblity"
			hngr_enty_vis.setAddress(hangar_entity_visibility)
			hngr_enty_vis.Type = vtByte
			hngr_enty_vis.appendToEntry(header)

			--[[Set hotkeys]]

			mrhk_hngr_cam_xpos = al.getMemoryRecordByDescription("HANGAR camera X pos")
			mrhk_hngr_cam_ypos = al.getMemoryRecordByDescription("HANGAR camera Y pos")
			mrhk_hngr_cam_zpos = al.getMemoryRecordByDescription("HANGAR camera Z pos")
			mrhk_hngr_cam_p = al.getMemoryRecordByDescription("HANGAR camera pitch")
			mrhk_hngr_cam_y = al.getMemoryRecordByDescription("HANGAR camera yaw")
			mrhk_hngr_cam_r = al.getMemoryRecordByDescription("HANGAR camera roll")

			key1 = mrhk_hngr_cam_xpos.createHotkey({VK_D},mrhIncreaseValue,move_cam_rate,"HANGAR camera X pos")
			key2 = mrhk_hngr_cam_xpos.createHotkey({VK_A},mrhDecreaseValue,move_cam_rate,"HANGAR camera X pos")
			key1 = mrhk_hngr_cam_ypos.createHotkey({VK_E},mrhIncreaseValue,move_cam_rate,"HANGAR camera Y pos")
			key2 = mrhk_hngr_cam_ypos.createHotkey({VK_Q},mrhDecreaseValue,move_cam_rate,"HANGAR camera Y pos")
			key1 = mrhk_hngr_cam_zpos.createHotkey({VK_S},mrhIncreaseValue,move_cam_rate,"HANGAR camera Z pos")
			key2 = mrhk_hngr_cam_zpos.createHotkey({VK_W},mrhDecreaseValue,move_cam_rate,"HANGAR camera Z pos")

			key1 = mrhk_hngr_cam_p.createHotkey({VK_I},mrhIncreaseValue,move_analog_cam_rate,"HANGAR camera pitch")
			key2 = mrhk_hngr_cam_p.createHotkey({VK_K},mrhDecreaseValue,move_analog_cam_rate,"HANGAR camera pitch")
			key1 = mrhk_hngr_cam_y.createHotkey({VK_J},mrhIncreaseValue,move_analog_cam_rate,"HANGAR camera yaw")
			key2 = mrhk_hngr_cam_y.createHotkey({VK_L},mrhDecreaseValue,move_analog_cam_rate,"HANGAR camera yaw")
			key1 = mrhk_hngr_cam_r.createHotkey({VK_O},mrhIncreaseValue,move_analog_cam_rate,"HANGAR camera roll")
			key2 = mrhk_hngr_cam_r.createHotkey({VK_U},mrhDecreaseValue,move_analog_cam_rate,"HANGAR camera roll")
		end
		fl.destroy()
		fl = nil
	else
		ms.destroy()
	end
	ms.destroy()


	--[[Start search for HUD elements and disable them]]

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil,"00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 61 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 B8 3D 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 0C 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 8C 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 9E 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 BA 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 E8 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 F6 3E 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 0C 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 23 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 28 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 3A 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 3E 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 51 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 54 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 3F 00 00 80 3F 00 00 80 3F 00 00 69 3F 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00", nil, 0x20700000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]

	if (fl~=nil) then
		al = getAddressList()
		for i = 1, 1 do
			base_address = getAddress(fl[0])

			hngr_hud_stff1 = al.createMemoryRecord()
			hngr_hud_stff2 = al.createMemoryRecord()
			hngr_hud_stff3 = al.createMemoryRecord()

			hangar_hud_stuff1 = base_address + 0xC
			hangar_hud_stuff2 = base_address + 0x4CC
			hangar_hud_stuff3 = base_address + 0x4EC

			writeBytes(hangar_hud_stuff1, 0, 0, 0, 0)
			writeBytes(hangar_hud_stuff2, 0, 0, 0, 0)
			writeBytes(hangar_hud_stuff3, 0, 0, 0, 0)

			hngr_hud_stff1.Description = "Hangar HUD stuff 1"
			hngr_hud_stff1.setAddress(hangar_hud_stuff1)
			hngr_hud_stff1.Aob.Size = 04
			hngr_hud_stff1.ShowAsHex = true
			hngr_hud_stff1.appendToEntry(header)

			hngr_hud_stff2.Description = "Hangar HUD stuff 2"
			hngr_hud_stff2.setAddress(hangar_hud_stuff2)
			hngr_hud_stff2.Aob.Size = 04
			hngr_hud_stff2.ShowAsHex = true
			hngr_hud_stff2.appendToEntry(header)

			hngr_hud_stff3.Description = "Hangar HUD stuff 3"
			hngr_hud_stff3.setAddress(hangar_hud_stuff3)
			hngr_hud_stff3.Aob.Size = 04
			hngr_hud_stff3.ShowAsHex = true
			hngr_hud_stff3.appendToEntry(header)

		end
		fl.destroy()
		fl = nil
	else
		ms.destroy()
	end
	ms.destroy()

	--[[Start search for the display values of that pie-chart thingy and disable it]]

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "40 99 3D 00" , nil, 0x20700000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]

	if (fl~=nil) then
		al = getAddressList()
		for i = 1, 1 do
			base_address = getAddress(fl[0])

			pie_chrt = al.createMemoryRecord()
			pie_chart_graphic = base_address - 0x8
			writeBytes(pie_chart_graphic, 0, 0, 176, 27)

			pie_chrt.Description = "Hangar HUD stuff 4"
			pie_chrt.setAddress(pie_chart_graphic)
			pie_chrt.Aob.Size = 04
			pie_chrt.ShowAsHex = true
			pie_chrt.appendToEntry(header)

		end

		fl.destroy()
		fl = nil

	else
		ms.destroy()
	end

	ms.destroy()

	--[[Start search for entities coordinates and ground reflection effect]]
	--//Note about ground reflection effect:
	--This value is only available for specific hangars.

	ms = createMemScan()
	ms.firstScan(soExactValue, vtByteArray, nil, "00 00 CA 42 00 00 DC C2 00 00 A0 42 00 00 A0 42 00 00 9E 42 00 00 F0 C0 00 00 04 42 7F 7F 7F 7F 7F 7F 7F 7F 00 00 00 46 00 00 57 43 04 00 00 00 00 00 00 40 00 00 20 C1 00 00 F0 41 00 00 F0 C1 00 00 3E 43 00 00 02 43 00 00 0C 42 00 00 20 41 00 00 08 43 00 00 48 C3 00 00 B6 42 00 00 AA 42 00 00 A0 42 00 00 F0 C0 00 00 B6 42 7F 7F 7F 7F 7F 7F 7F 7F 00 00 40 46 00 00 07 43 04 00 00 00 00 00 00 40 00 00 82 43 00 00 F0 41 00 00 B4 C2 00 00 61 43 00 00 40 42 00 00 96 C2 00 00 20 41 00 00 0C 43 00 00 52 C3 00 00 B6 42 00 00 A4 42 00 00 A0 42 00 00 F0 C0 00 00 B6 42 7F 7F 7F 7F 7F 7F 7F 7F 00 00 00 46 00 00 61 43 04 00 00 00 00 00 00 40 00 80 93 C3 00 00 48 42 00 00 00 41 00 00 2F C3 00 00 C8 42 00 00 D2 C2 00 00 20 41 00 00 C8 42 33 33 47 C2 00 00 A0 42 00 00 9E 42 00 00 9C 42 00 00 F0 C0 00 00 E0 41 7F 7F 7F 7F 7F 7F 7F 7F 00 00 00 46 00 00 57 43 04 00 00 00 00 00 00 40 00 00 20 42 00 00 F0 41 00 00 F0 C1 00 00 70 43 00 00 02 43 00 00 0C 42 00 00 20 41", nil, 0x20700000,0x21f00000,"",1,"4",true,nil,nil,nil)
	ms.waitTillDone()
	fl = createFoundList(ms)
	fl.initialize()

	--[[Process found results and create cheat table]]

	if (fl~=nil) then
		al = getAddressList()
		for i = 1, 1 do
			base_address = getAddress(fl[0])

			hngr_plyr_pos0 = al.createMemoryRecord()
			hngr_wngmn_pos0 = al.createMemoryRecord()
			hngr_plyr_pos1 = al.createMemoryRecord()
			hngr_wngmn_pos1 = al.createMemoryRecord()
			hngr_plyr_pos2 = al.createMemoryRecord()
			hngr_wngmn_pos2 = al.createMemoryRecord()
			hngr_plyr_pos3 = al.createMemoryRecord()
			hngr_wngmn_pos3 = al.createMemoryRecord()
			grnd_eff = al.createMemoryRecord()

			hangar_wingman_position0 = base_address
			hangar_player_position0 = base_address + 0x4

			hangar_wingman_position1 = base_address + 0x50
			hangar_player_position1 = base_address + 0x54

			hangar_wingman_position2 = base_address + 0xA0
			hangar_player_position2 = base_address + 0xA4

			hangar_wingman_position3 = base_address + 0xF0
			hangar_player_position3 = base_address + 0xF4

			ground_reflection = base_address + 0x11D

			--writeBytes(ground_reflection, 1) --This option should be left to the user to modify it

			grnd_eff.Description = "Hangar ground reflection effect"
			grnd_eff.setAddress(ground_reflection)
			grnd_eff.Type = vtByte
			grnd_eff.appendToEntry(header)

			hngr_wngmn_pos0.Description = "WINGMAN position in hangar type 0"
			hngr_wngmn_pos0.setAddress(hangar_wingman_position0)
			hngr_wngmn_pos0.Type = vtSingle
			hngr_wngmn_pos0.appendToEntry(header)

			hngr_plyr_pos0.Description = "PLAYER position in hangar 0"
			hngr_plyr_pos0.setAddress(hangar_player_position0)
			hngr_plyr_pos0.Type = vtSingle
			hngr_plyr_pos0.appendToEntry(header)

			hngr_wngmn_pos1.Description = "WINGMAN position in hangar type 1"
			hngr_wngmn_pos1.setAddress(hangar_wingman_position1)
			hngr_wngmn_pos1.Type = vtSingle
			hngr_wngmn_pos1.appendToEntry(header)

			hngr_plyr_pos1.Description = "PLAYER position in hangar 1"
			hngr_plyr_pos1.setAddress(hangar_player_position1)
			hngr_plyr_pos1.Type = vtSingle
			hngr_plyr_pos1.appendToEntry(header)

			hngr_wngmn_pos2.Description = "WINGMAN position in hangar type 2"
			hngr_wngmn_pos2.setAddress(hangar_wingman_position2)
			hngr_wngmn_pos2.Type = vtSingle
			hngr_wngmn_pos2.appendToEntry(header)

			hngr_plyr_pos2.Description = "PLAYER position in hangar 2"
			hngr_plyr_pos2.setAddress(hangar_player_position2)
			hngr_plyr_pos2.Type = vtSingle
			hngr_plyr_pos2.appendToEntry(header)

			hngr_wngmn_pos3.Description = "WINGMAN position in hangar type 3"
			hngr_wngmn_pos3.setAddress(hangar_wingman_position3)
			hngr_wngmn_pos3.Type = vtSingle
			hngr_wngmn_pos3.appendToEntry(header)

			hngr_plyr_pos3.Description = "PLAYER position in hangar 3"
			hngr_plyr_pos3.setAddress(hangar_player_position3)
			hngr_plyr_pos3.Type = vtSingle
			hngr_plyr_pos3.appendToEntry(header)

		end

		fl.destroy()
		fl = nil

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

{$asm}


[DISABLE]

{$lua}
--//Restore values to default

--[[Restore controller input]]
WriteBytes(0x203F70B8, 48, 125, 105, 00)

--[[Restore controller input. TODO]]
--writeBytes(ground_reflection, 0)

--[[Restore disabled HUD elements]]
writeBytes(hangar_hud_stuff1, 0, 0, 97, 63)
writeBytes(hangar_hud_stuff2, 0, 0, 24, 62)
writeBytes(hangar_hud_stuff3, 0, 0, 24, 62)
writeBytes(pie_chart_graphic, 1, 0, 176, 27)

--[[Restore defaul aircraft positions on the hangar]]

writeBytes(hangar_aircraft_placement_type, hangar_aircraft_placement_type_old)
writeFloat(hangar_wingman_position0, 101)
writeFloat(hangar_player_position0, -110)
writeFloat(hangar_wingman_position1, 136)
writeFloat(hangar_player_position1, -200)
writeFloat(hangar_wingman_position2, 140)
writeFloat(hangar_player_position2, -210)
writeFloat(hangar_wingman_position3, 100)
writeFloat(hangar_player_position3, -49.79999924)

--[[Destroy entries when disabling]]
header.destroy()

--[[Restore NOP'd code]]
gameplayCamControl(false)

{$asm}
