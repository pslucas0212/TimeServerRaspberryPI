# Setting Raspberry PI to Be a Time Server

Instructions for Setting Up GPS USB for Time Server Support

### Software Installation  

Install the GPS Daemon gpsd and the GPS Daemon client gpsd-clients
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

gpsd file contents
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

Stop gpsd to manage GPS reciever "manually".  Ctrl-c to stop
```
# systemctl stop gpsd
```
Start GSP monitoring tool
```
# gpsmon /dev/<USB >
```

Another tool for same information - CGPS.   Ctrl-c to stop
```
# cgps -s
```

Make NTP configuration

Install NTP
```
# apt install ntp
```

Edit ntpd.conf - /etc.  

Disable the pool servers that come as default.

### References
[How to connect an usb GPS receiver with a Raspberry PI?[(https://gpswebshop.com/blogs/tech-support-by-os-linux/how-to-connect-an-usb-gps-receiver-with-a-raspberry-pi)
[Raspberry Pi 4 GPS Install](https://www.youtube.com/watch?v=isVHkovZuSM)
[How to Setup GPS Tracker for Raspberry Pi](https://www.youtube.com/watch?v=A1zmhxcUOxw)















