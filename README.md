# Setting Raspberry PI to Be a Time Server

Instructions for Setting Up GPS USB for Time Server Support

### Software Installation  

Install GPS?
```
# apt install gpsd
# apt install gpsd-clients
```


Determine device by serial.  You will use this for the DEVICES setting in the gpsd configuration file that you will create in the next step.
```
# ls -l /dev/serial/by-id/  
total 0
lrwxrwxrwx 1 root root 13 Oct 24 20:17 usb-Prolific_Technology_Inc._USB-Serial_Controller-if00-port0 -> ../../ttyUSB0
```

Create and Configure GPSD file

```
cd /etc/default
vi gpsd
```
GPSD file contents
```
START_DAEMON="true"

USBAUTO="true"

DEVICES="/dev/ttyACM0"

GDSP_OPTIONS="-n -D 1"

ENABLED="yes"

GPS_BAUD=9600
```

Now start gpsd and check status.

```
# systemctl start gpsd
# systemctl status gpsd
```

















```
