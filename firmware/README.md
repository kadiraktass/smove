# Firmware challenge solution

[Desired system](https://github.com/itatsmove/smovechallenge/blob/master/challenges/firmware.md) is implemented by two scripts. [One](https://github.com/kadiraktass/smove/blob/master/firmware/uno_slave.ino) (uno_slave.ino) for Arduino and [one](https://github.com/kadiraktass/smove/blob/master/firmware/rpi_master.py) (rpi_master.py) for Raspberry Pi. 

## Installation instructions

### RPi part: 

You need to activate I2C interface on your RPi. Open a terminal in RPi. Run:

    sudo raspi-config
    
Go to **Interfacing Options**. Select **I2C** and select **Yes**.  
Also, you need to install libraries for the script if they are not installed yet. In the terminal, run:

    sudo apt-get install python-smbus i2c-tools  
    sudo pip install pyserial
    
Lastly, you need to make RPi run the script on startup.  
Open a terminal and go to the directory of your script:
  
    cd /path/to/rpi_master.py
    
 Make the script executable: 
 
    chmod +x rpi_master.py 

Next, add the line **/path/to/rpi_master.py &** to the file **/etc/rc.local** before the **exit 0** line.   
    
### Arduino part: 

Simply [upload](https://www.arduino.cc/en/Guide/ArduinoUno) the script (uno_slave.ino) to your Arduino. 

    
## Usage 

Setup the hardware and switch on the power.  
Send commands to RPi via serial interface.   

## RPi script 

In the main loop, RPi gets the commands from the user via serial interface. Then it process the command by either ordering Arduino to perform a task or getting the last fuel sensor read value. And, it responses to the user accordingly. 

In another thread, RPi reads the fuel sensor value periodically by sending a data request to Arduino in every second. 

Communication between RPi and Arduino is established with I2C as RPi master. A simple command list is defined for the purpose of message exchange between these two device. RPi sends a number to Arduino and Arduino performs tasks in respect to the number it receives.  

Used libraries in RPi script:  
[Pyserial](https://pythonhosted.org/pyserial/) (For the serial interface)  
[Smbus](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/Documentation/i2c/smbus-protocol) (For I2C)  
[Threading](https://docs.python.org/3/library/threading.html) (For periodic check)   

## Arduino script  

In the main loop, Arduino waits for an I2C message from master RPI. When it receives the message, it controls the relays or updates the fuel sensor reading. Also, if RPi makes a data request, it sends the fuel sensor value.  

And, if Arduino does not receive a message for 10 seconds, it restarts RPi. 

Used libraries in Arduino script:  
[Wire](https://www.arduino.cc/en/Reference/Wire) (For I2C)
