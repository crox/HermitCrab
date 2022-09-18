# How to use Klipper on HermitCrab Canbus version

## Pinout
### HermitCrab Canbus pinout
<img src=Images/canbus_pinout.jpg width="400" /><br/>

## Wiring diagram
### Raspberry pi communicates with the hotend board via USB
<img src=Images/wiring_usb.png width="1200" /><br/>

### Raspberry pi communicates with the hotend board via Canbus
<img src=Images/wiring_canbus.png width="1200" /><br/>

## Generate the firmware.bin 

You can either get precompiled binaries (1) or build your own firmware (2):
1. Get precompiled firmware

   * [firmware-F072-Canbus.bin](./firmware-F072-Canbus.bin) is the firmware that communicates with raspberry pi through Canbus



## Update the firmware.bin on the HermitCrab Hotend
1. Connect the USB of the board to your Computer  through microUSB port(Note: The USB of the board cannot supply power to the MCU, so an external 12/24V is required to supply power to the board).  Note: this affects step 3 below.
2. Press and hold the `Boot` button on the back, then click the `Reset` button, and then release the `boot` button. And The MCU has entered DFU mode now.
   <br/><img src=Images/boot.png width="400" /><br/>
3. You can update the firmware through your Computer (i) or use the raspberry pi (ii)
  * Update the firmware through Computer
    * Download and install the required `STM32CubeProgrammer` directly from the ST website:  https://www.st.com/en/development-tools/stm32cubeprog.html
    * Download `firmware.bin` into MCU with `STM32CubeProgrammer`, and then click `Reset` button to enter normal working mode.
 
## Configure the printer parameters on the RPi
1. [HermitCrab_Canbus_pins.cfg](./HermitCrab_Canbus_pins.cfg) is the configuration file of klipper which contains all pinouts of Canbus hotend board
    ```
3. If you use Canbus to communicate with raspberry pi, you need to modify the following files by logging into the RPi (with SSH or similar). And wiring reference [here](#raspberry-pi-communicates-with-the-hotend-board-via-canbus)<br/>
   Refer to [klipper's official Canbus config](https://www.klipper3d.org/CANBUS.html) to set Canbus
   * Input `sudo nano /boot/config.txt` command, input the password (default: raspberry), Then add the following content to the `config.txt` file
     ```
     dtparam=spi=on
     dtoverlay=mcp2515-can0,oscillator=12000000,interrupt=25,spimaxfrequency=1000000
     ```
     Save(`Ctrl + S`) and Exit(`Ctrl + X`) after modification, input `sudo reboot` to restart raspberry pi
   * Wait for RPi to boot again, then login and input the `dmesg | grep -i '\(can\|spi\)'` command to test whether MCP2515 has been connected normally after the restart is completed.<br/>
     The normal response should be as follows
     ```
     [ 8.680446] CAN device driver interface
     [ 8.697558] mcp251x spi0.0 can0: MCP2515 successfully initialized.
     [ 9.482332] IPv6: ADDRCONF(NETDEV_CHANGE): can0: link becomes ready
     ```
     <img src=Images/can_connected.png width="800" /><br/>
   * Input `sudo nano /etc/network/interfaces.d/can0` command to create a new file named `can0`, and write the following content
     ```
     auto can0
     iface can0 can static
         bitrate 250000
         up ifconfig $IFACE txqueuelen 1024
     ```
     Set the Canbus speed to 250K (consistent with the speed set in [firmware-F072-Canbus.bin](./firmware-F072-Canbus.bin)), and set the `txqueuelen` to 1024 bytes. Save(`Ctrl + S`) and Exit(`Ctrl + X`) after modification, input `sudo reboot`to restart raspberry pi

   * Each micro-controller on the CAN bus is assigned a unique id based on the factory chip identifier encoded into each micro-controller. To find each micro-controller device id, make sure the hardware is powered and wired correctly, and then run:<br/>
     `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`<br/>
     If uninitialized CAN devices are detected the above command will report lines like the following:<br/>
     `Found canbus_uuid=0e0d81e4210c`<br/>
     set the correct ID number in `printer.cfg`<br/>
     ```
     [mcu HermitCrab]
     canbus_uuid: 0e0d81e4210c
     ```
