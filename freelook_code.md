[ENABLE]
//Hangar freecam by dead_d0g (death_the_d0g @ twitter)

{$lua}


base_address = 0x20765231

al = getAddressList()

header = al.createMemoryRecord()
header.Description = "FREE LOOK script"
header.isGroupHeader = true

ky_mssg1 = al.createMemoryRecord()
ky_mssg2 = al.createMemoryRecord()
ky_mssg3 = al.createMemoryRecord()
ky_mssg4 = al.createMemoryRecord()
ky_mssg5 = al.createMemoryRecord()
ky_mssg6 = al.createMemoryRecord()
ky_mssg7 = al.createMemoryRecord()
ky_mssg8 = al.createMemoryRecord()
ky_mssg9 = al.createMemoryRecord()
ky_mssg10 = al.createMemoryRecord()
ky_mssg11 = al.createMemoryRecord()
ky_mssg12 = al.createMemoryRecord()
ky_mssg13 = al.createMemoryRecord()
ky_mssg14 = al.createMemoryRecord()
ky_mssg15 = al.createMemoryRecord()
ky_mssg16 = al.createMemoryRecord()

ky_mssg1.Description = "TRIANGLE = move up"
ky_mssg1.appendToEntry(header)

ky_mssg2.Description = "CROSS = move down"
ky_mssg2.appendToEntry(header)

ky_mssg3.Description = "CIRCLE = move right"
ky_mssg3.appendToEntry(header)

ky_mssg4.Description = "SQUARE = move left"
ky_mssg4.appendToEntry(header)

ky_mssg5.Description = "R1 = move forwards"
ky_mssg5.appendToEntry(header)

ky_mssg6.Description = "L1 = move backwards"
ky_mssg6.appendToEntry(header)

ky_mssg7.Description = "L2 = yaw left"
ky_mssg7.appendToEntry(header)

ky_mssg8.Description = "R2 = yaw right"
ky_mssg8.appendToEntry(header)

ky_mssg9.Description = "L-STICK UP = pitch up"
ky_mssg9.appendToEntry(header)

ky_mssg10.Description = "L-STICK DOWN = pitch down"
ky_mssg10.appendToEntry(header)

ky_mssg11.Description = "L-STICK LEFT = roll left"
ky_mssg11.appendToEntry(header)

ky_mssg12.Description = "L-STICK RIGHT = roll right"
ky_mssg12.appendToEntry(header)

ky_mssg13.Description = "D-PAD LEFT = rotate 90° left"
ky_mssg13.appendToEntry(header)

ky_mssg14.Description = "D-PAD RIGHT = rotate 90° right"
ky_mssg14.appendToEntry(header)

ky_mssg15.Description = "D-PAD UP = set rotation to 0º"
ky_mssg15.appendToEntry(header)

ky_mssg16.Description = "D-PAD DOWN = rotate 90° down"
ky_mssg16.appendToEntry(header)

writeBytes(base_address, 0)

{$asm}


[DISABLE]

{$lua}
writeBytes(base_address, 2)
header.destroy()
{$asm}
