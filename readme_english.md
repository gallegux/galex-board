# GALEX 8i1<br> an AMSTRAD CPC expansion


## Objectives

- Develop a digital electronics project.

- Equip the Amstrad CPC with features that users have missed over time, even though some of them could previously be obtained at high prices.

1- **Mass storage**. Currently, it is very difficult or impossible to find 3" floppy disks, and the few available are expensive and unreliable.

2- **RAM memory**, to run software that requires 128 Kb on the CPC 464 and 664, as well as software that makes use of extended memory beyond 128 Kb.

3- **ROM memory**, which adds functionality from the moment the computer is turned on, adding compilers, text editors, games, etc. Since it runs from memory, booting is instantaneous.

4- **Serial port**.

And since the microcontroller to be used has enough GPIO pins, we also add:

5- Reading of 3 **analog sensors**, as allowed by the microcontroller.

6- An **RTC** module to keep the time, in case it is of interest for any application (Symbos?). The RTC uses a CR927 battery to maintain the time, which can last about 4 years.

7- An **EEPROM** module (32 Kb) to save configurations, either for the expansion itself or for the CPC user. This is highly optional. It has been included because it is a very cheap component.

- A 0.96" or 1.3" OLED screen to display information, both during development/testing and ultimately to the user. (optional)

(These last three use the I2C protocol, so they only require 2 GPIO pins)

8- **WIFI** module (optional) using the ESP01s board.

<br>

### Board parts of interest

<img src="images/partes.png" width="80%" />

**A**: Power LED.

**B**: Reset button.

**C**: mSD card slot.

**D**: Jumper to supply power to the voltage regulator of the Wifi module.

**E**: Connector for the ESP01s Wifi module.

**F**: Status indicator LEDs for virtual floppy drives A and B:

- RED: no disk inserted.
- GREEN: disk inserted and write-unprotected.
- BLUE: disk inserted and write-protected.
If they are off, it means the virtual floppy drives are not active.

**G and H**: Buttons to manipulate the drives and disks. The top one is for drive/disk A and the bottom one is for B. A short press changes the disk and a long press toggles write protection.

**I**: Connector for the CR927 battery.

**J**: Connection pins for the serial connection and analog sensors.

**K**: Switch to enable/disable ROM functionality.

**L**: Switch to power the expansion board on and off.

**M**: Connector for the OLED screen (0.96" or 1.3"). The resistors on this module must be removed.

**N**: Pads to solder the voltage regulator for the Wifi module.

<br>

## The microcontroller

The RP2350B on the WeAct board was chosen because it exposes all the pins and allows for additional memory.

<img src="images/weactstudio.png" width="80%" />

To emulate RAM and ROM, we need more RAM than what comes with the microcontroller, since the 512 Kb of SRAM will be consumed by program variables and the libraries used. In the best-case scenario, we would have 31 banks of 16 Kb, which we would have to divide between RAM and ROM. To start with, a configuration with 8 ROM banks and 384 Kb of RAM wouldn't be bad (RAM is added in 64 Kb blocks), but we don't really know how much free memory we will have left.

To have more memory available, we solder a 16 Mbit (=2 Mbytes) PSRAM module to the back of the RP2350B board, giving us flexibility when configuring memory, for example:

- 1.5 MB of RAM and 32 ROMs
- 1.9 MB of RAM and 4 ROMs

<img src="images/weact-bottom.png" alt="WeAct Studio board bottom view" width="40%" /><i>WeAct board bottom view</i>

<img src="images/memory.png" alt="PSRAM chip" width="40%"/><i>PSRAM chip</i>

<img src="images/p-weact.png" alt="WeAct on the main board" width="80%"/><i>WeAct on the main board</i>

<br>

## WIFI Module

This module is completely optional. The ESP01S is powered at 3.3V from a voltage regulator connected directly to the 5V input. Since the voltage regulator consumes power, a jumper is provided to enable or disable its current. The reason for using a regulator is that the current the microcontroller can supply at 3.3V is insufficient.

<img src="images/regulador.jpg" alt="Voltage regulator" width="25%" /><i>Voltage regulator</i>

<img src="images/esp01s.jpg" alt="ESP01S" width="30%" /><i>ESP01S</i>

To install the voltage regulator, bend the pins straight and solder it so that the silkscreen faces downward and the AMS1117 faces upward. It does not interfere with the WeAct Studio board.

<img src="images/p-wifi.png" alt="Regulator and ESP01S in place" width="80%" /><i>Regulator and ESP01S in place</i>

<br>

## OLED Screen
There are several sizes with different pinouts. The one we use is VDD-GND-SCK-SDA.

<img src="images/p-full.png" alt="Full expansion" width="80%" /><i>Full expansion</i>

This module requires longer pins.

<br>

## RSX Commands (not final)

**|HD**  enables mass storage from the mSD card, acting like a hard drive with directory support

**|IDRIVE**  enables the CPC's built-in disk drive

**|DSK**  enables disk emulation (.dsk files)

### HD Mode Commands

**|CD**   change directory

**|CPE,"source","destination"**  copy a file to the emulated disk

**|CPF,"source","destination"**  copy a file to the drive's disk

**|CP,"source","destination"**  copy a file to the flash drive

**|MV,"source","destination"**  change file location

**|RM,"file/directory"**  delete a file or directory

**|TREE**  directory structure

**|CATA**  extended CAT

### Disk Emulation Commands

(A disk set is a selection of .dsk files)

**|LSTA**  shows the disks in set A (floppy drive A)

**|LSTB**  shows the disks in set B (floppy drive B)

**|SETA**  sets the disks for set A

**|SETB**  sets the disks for set B

**|CLA**  clears set A

**|CLB**  clears set B

**|NEXTA**  next disk in set A

**|NEXTB**  next disk in set B

**|PROTA,1**   protect disk A

**|PROTA,0**   unprotect disk A

**|PROTB,1**   protect disk B

**|PROTB,0**   unprotect disk B

**|CP,"source","destination"**  copy a file

**|CPF,"source","destination"**  copy a file to the drive's disk

**|CPH,"source","destination"**  copy a file to the flash drive


### Commands for Real Disks

**|CP,"source","destination"**  copy a file

**|CPH,"source","destination"**  copy a file to the flash drive


### ROM Expansion Commands

**|ROM,slot,"file.rom"**  sets a ROM in a slot

**|ROM,slot**  removes the ROM from the slot

**|ROMLST**  shows installed ROMs

Use slot 255 to refer to the LOWER ROM.


### Commands for RAM Configuration

**|RAM,64k-blocks**  sets the extra 64 Kbyte blocks

**|RAM**  shows the amount of RAM in the expansion


### Other Commands

**|BURNDSK,"file.dsk","drive"**  copies a .dsk file to a physical disk, drive=A|B

**|GETDSK,"drive","file.dsk"**  creates a disk image and saves it as a file


### RTC Module Commands

**|DATE**  Returns the date in yyyy-mm-dd format

**|DATE,"y-m-d"**  Sets the date

**|TIME**  Returns the time in hh:mm:ss format (24h)

**|SETTIME,"h:m:s"**  Sets the time


### EEPROM Module Commands

**|EEPROM,address,byte**  saves a byte to the specified address

**|EEPROM,address**   reads the byte from the specified address


### WIFI Module Commands

**|PING,"ip-or-hostname"**  pings a host

**|WLAN**  scans for connectable WLAN networks

**|WIFI,"ssid","password"**  WiFi configuration using DHCP

**|WIFI,"ssid","password","ip","mask","gw","dns1","dns2"**  WiFi configuration with static IP

**|WGET,"url","file"**  downloads a file from the internet


### Serial Communication Commands

**|SERIAL,"serial-comm-parameters"**  configures serial communication

Bytes will be sent and received using OUT and IN instructions.


### Analog Sensor Commands

**|ADC,sensor-num**  gets an 8-bit value from the selected ADC (sensor=1..3)