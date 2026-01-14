# How it started
Late 2025, I came across a Taobao listing that was selling a UHF RFID Reader for less than 10USD, this is fairly uncommon as most readers go for 15+usd onwards. I decided to buy one to add to my reader collection. (I ended up buying 6 units)<br>

<img src="./imgs/Taobao_listing.png" alt="drawing" width="600"><br>

This UHF RFID reader is sold under the model numbers below (with Taobao links)

- [CPH-F206](https://item.taobao.com/item.htm?id=904352296655&skuId=5763416697782)
- [TC-C206](https://item.taobao.com/item.htm?id=820590020134&skuId=5696639435762)
- [BYH-206](https://detail.tmall.com/item.htm?id=986978403603&skuId=5952377616135)

Searching online based on provided software, and screenshots, I was able to track down the company that made the software to [Cykeo](https://www.cykeorfid.com/), they are a company that specializes in making industrial rfid readers and antennas.

# Reader Software
The reader comes with its own software and SDK, and is able to output in multiple formats including Wiegand, Modbus, as well as emulating a USB Keyboard.<br>
<img src="./imgs/Reader_Software_1.png" alt="drawing" width="400"><br>
<img src="./imgs/Reader_Software_2.png" alt="drawing" width="400"><br>

# PCB Screenshot
Cracking open the reader, we are greeted with a 30mm x 30mm PCB soldered to a another PCB that provides a USB-C connector, a buzzer, and some led's, to keep the PCB compact, 0603 SMD components were used.<br>
<img src="./imgs/PCB_Screenshot.jpg" alt="drawing" width="400"><br>

Taking a closer look at the front, we see the MCU and an unknown RF chip has their markings lasered off, and the back has descriptions on what each pin does, what catches my eye is the SWDIO and SWCLK pins, this means theres Serial Wire Debug capabilities, and a chance to dump firmware.<br>
<img src="./imgs/UHF_PCB_Front.jpg" alt="drawing" width="400"><br>
<img src="./imgs/UHF_PCB_Back.jpg" alt="drawing" width="400"><br>

# MCU Identification & Firmware Dumping
Before we can attempt to dump the firmware, we'd need to try to figure out what MCU is used, and lucky for us when plugging in the reader the device identifies itself as a N32g43xCustom HID, this led me to believe that the MCU was a [N32G435](https://nsing.com.sg/product/General/cortexm4/N32G435/)

Connecting the SWD header to a J-LINK, setting up its parameters, and attempting to read the firmware, I was able to get a successful dump. Usually MCU's have code readout protection which prevents the downloading of flash contents, the manufacturer did not turn this feature on.

# Firmware Reversing
The firmware can now be thrown into IDA/Ghidra for analysis, using the strings within, around 70% of the firmware can be mapped back to the N32G43x SDK and example implementation found on github. From within the firmware, the RF chip can be identified, they are using an [Si4463](https://www.silabs.com/wireless/proprietary/ezradiopro-sub-ghz-ics/device.si4463?tab=specs) Transceiver chip that supports the ISM band.

<img src="./imgs/Firmware_IDA.png" alt="drawing" width="600"><br>
The reversing of the firmware is left as an exercise for the user and will not be covered here.

# PCB Reverse Engineering
The reversing of the PCB would not be possible if the firmware dump was not successful, now that we have the firmware, my next goal is to recreate the PCB but using larger 0805 SMD components.

With PCB reversing, the first steps are component identification, using a hot-plate, components were removed one by one, and their values recorded.<br>

<img src="./imgs/Component_Identification_1.jpg" alt="drawing" width="600"><br>

When dealing with small value components like inductors and capacitors, a Nano-VNA can be used to identify their values.<br>

<img src="./imgs/Component_Identification_2.jpg" alt="drawing" width="600"><br>

# Parts Identification
Identification of parts took a while as finding parts with the same name was challenging. Using part numbers, a multimeter, a component tester, an oscilloscope, image searching and pouring over datasheets helped find most of the components.<br>

<img src="./imgs/Deadbug.png" alt="drawing" width="600"><br>

| Marking | Name | Notes |
| --- | ---| --- |
| N32G435 | | MCU <br> [Datasheet](https://www.nsing.com.sg/uploads/DS/EN_DS_N32G435.pdf) |
| Si4463 | | RF Transceiver <br> [Datasheet](https://www.silabs.com/documents/public/data-sheets/Si4463-61-60-C.pdf)|
| 87t | Schottky barrier diode |  No exact match found, the closest is [BAT54S](https://assets.nexperia.com/documents/data-sheet/BAT54S.pdf) |
| 1F | 45 V, 100 mA NPN/NPN general-purpose transistor | [Datasheet](https://assets.nexperia.com/documents/data-sheet/BC847BS.pdf) |
| -V4 | 2-ch, 1.65-V to 5.5-V inverters | [SN74LVC2G04](https://www.ti.com/lit/ds/symlink/sn74lvc2g04.pdf) |
| AM | NPN switching transistor | [MMBT3904](https://www.onsemi.com/pdf/datasheet/mmbt3904wt1-d.pdf) |
| G4B | RF Switching Diode | No exact match found, the closest is [BAV70](https://assets.nexperia.com/documents/data-sheet/BAV70.pdf)
| N933 | 500mA uCap Ultra-Low Dropout, High PSRR LDO Regulator | [Datasheet](https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/DataSheets/500mA-uCap-Ultra-Low-Dropout-Regulator-with-High-PSRR-DS20005876B.pdf) |

For the Si4463 matching and filter network, the datasheet was used to identify the nominal values.

<img src="./imgs/SI4463_Matching_Components.png" alt="drawing" width="600"><br>

# Sanding PCB
After the components and their positions have been identified, the solder mask of the PCB is stripped using a 200-600 grit sandpaper to expose the front and back copper layers. This PCB is 4 layers and is 1mm thick.

<img src="./imgs/UHF_PCB_Bare_Front.jpg" alt="drawing" width="400"><br>
<img src="./imgs/UHF_PCB_Bare_Back.jpg" alt="drawing" width="400"><br>

# Tracing PCB
Next using high resolution images, Inkscape, and a multimeter, the traces of the PCB and its via's are drawn in separate layers<br>
This step also maps the MCU and RF pins, an overlay from the datasheet is added for pin identification. The position and values of components measured earlier are also noted, grounds and power lines (5V, 3V3) are color coded.<br>

<img src="./imgs/PCB_Trace.jpg" alt="drawing" width="400"><br>

<img src="./imgs/Inkscape_PCB_Tracing.png" alt="drawing" width="400"><br>

# Reversing PCB & Schematic
Using KiCad, the schematic is then redrawn with their component values. In this step, the component names from KiCad are mapped back to the PCB Trace in Inkscape. Some modifications were made around the USB-C and 3V3 power regulators circuit.

<img src="./imgs/Schematic.png" alt="drawing" width="800"><br>

Schematics are released in the schematic directory of this repo.

# Impedance Matching of RF Trace
When recreating the PCB, KiCad's Calculator Tools and JLCPCB's Capabilities are used to calculate trace width and spacing for the RF trace line.

<img src="./imgs/JLCPCB_4_Layer_Capability.png" alt="drawing" width="400"><br>
<img src="./imgs/KiCad_Calculator.png" alt="drawing" width="400"><br>

# Ordering & Testing PCB
After weeks of tracing and schematic drawing, PCB's are sent for fabrication. When placing components, i mimicked the layout of the original board, this was done to make circuit debugging easier<br>
<img src="./imgs/JLCPCB_PCB.png" alt="drawing" width="400"><br>
Below is a size comparison of the original and recreated PCB.<br>
<img src="./imgs/PCB_Size_Comparison.jpg" alt="drawing" width="400"><br>
After soldering all the components, the original MCU was soldered on for testing. In the first attempt, the reader was not picking up any tags.<br>
Going back to the drawing board, a few components had to be re-measured from another working reader and a few mistakes were identified. At this point an oscilloscope was used to debug the circuit using references from a functional reader. After fixing mis-typed components and much experimenting, the reader works!.<br>
<img src="./imgs/PCB_Recreated.jpg" alt="drawing" width="400"><br>
One area that needs to be studied more is the matching network and filter sections as the current PCB isn't as fast/sensitive enough at picking up tags vs the original reader.<br>
In v0.2 of the schematic, additional capacitors were added to the 5V rail. Also fixed incorrectly measured value for some components.<br>

