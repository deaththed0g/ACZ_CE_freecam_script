```lua
[ENABLE]
//Entity coordinates finder script by death_the_d0g (@ Twitter and GitHub)
//Locates all the entities that are loaded in a mission and creates a list with their XYZ coordinates.

{$lua}

--[[Start search for entities]]

ms = createMemScan()
ms.firstScan(soExactValue, vtByteArray, nil, "CC CC 4C 42",nil,0x20700000,0x21f00000,"",1,"4",true,nil,nil,nil)
ms.waitTillDone()
fl = createFoundList(ms)
fl.initialize()
nor = fl.Count
arraypos = 0

--[[Process found results and create cheat table]]
if (fl~=nil) then

	al = getAddressList()
	header = al.createMemoryRecord()

	header.Description = "Found entities"
	header.isGroupHeader = true

	for i = 1, nor do

		base_address = getAddress(fl[arraypos])

		m = al.createMemoryRecord()

		xpos = al.createMemoryRecord()
		zpos = al.createMemoryRecord()
		ypos = al.createMemoryRecord()
		yrotation = al.createMemoryRecord()
		zrotation = al.createMemoryRecord()
		xrotation = al.createMemoryRecord()
		hp = al.createMemoryRecord()

		idstring = (readString(base_address + 0x4, 16, false))
		xposoff = base_address - 0x120
		zposoff = xposoff + 0x4
		yposoff = zposoff + 0x4
		yrotoff = xposoff + 0x10
		zrotoff = yrotoff + 0x4
		xrotoff = zrotoff + 0x4
        entityhpoff = base_address + 0x18

		m.Description = idstring
		m.appendToEntry(header)
		m.isGroupHeader = true
		m.options = "[moHideChildren, moActivateChildrenAsWell, moDeactivateChildrenAsWell, moRecursiveSetValue, moAllowManualCollapseAndExpand, moManualExpandCollapse]"

		xpos.Description = "X coordinates"
		xpos.setAddress(xposoff)
		xpos.Type = vtSingle
		xpos.appendToEntry(m)

		zpos.Description = "Z coordinates"
		zpos.setAddress(zposoff)
		zpos.Type = vtSingle
		zpos.appendToEntry(m)

		ypos.Description = "Y coordinates"
		ypos.setAddress(yposoff)
		ypos.Type = vtSingle
		ypos.appendToEntry(m)

		yrotation.Description = "Pitch"
		yrotation.setAddress(yrotoff)
		yrotation.Type = vtSingle
		yrotation.appendToEntry(m)

		zrotation.Description = "Yaw"
		zrotation.setAddress(zrotoff)
		zrotation.Type = vtSingle
		zrotation.appendToEntry(m)

		xrotation.Description = "Roll"
		xrotation.setAddress(xrotoff)
		xrotation.Type = vtSingle
		xrotation.appendToEntry(m)

        hp.Description = "Entity hit points"
        hp.setAddress(entityhpoff)
        hp.Type = vtByte
        hp.appendToEntry(m)

		arraypos = arraypos + 1
	end

	fl.destroy()
	fl = nil

else
	--//If nothing was found just exit and print an error message
	ms.destroy()
	print("No results found")

end

{$asm}
[DISABLE]

{$lua}

--//Destroy created entries when disabling script
header.destroy()

{$asm}
```