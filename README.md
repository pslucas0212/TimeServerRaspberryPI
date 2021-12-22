# Setting Up a Raspberry PI 4 to Be a Time Server

Last updated: 12/22/2021

Instructions for Setting Up GPS USB for Time Server Support

### Software Installation  

Install the GPS Daemon gpsd and the GPS Daemon client gpsd-clients
```
$ sudo apt-get install gpsd gpsd-clients python-gps
```
Plug in your USB GPS to your Raspberry PI. 

You can see what the latest serial device that was added to your Pi typing:
```
$ ls -l /dev/tty*
...
crw--w---- 1 root tty       4,  9 Dec 22 10:17 /dev/tty9
crw-rw---- 1 root dialout 204, 64 Dec 22 10:17 /dev/ttyAMA0
crw------- 1 root root      5,  3 Dec 22 10:17 /dev/ttyprintk
crw-rw---- 1 root dialout 188,  0 Dec 22 10:39 /dev/ttyUSB0
```
The newest device will show up last.  You may see something else then ttyUSB0

Stop the gpsd service to bind the GPS to the service
```
$ sudo systemctl stop gpsd
Warning: Stopping gpsd.service, but it can still be activated by:
  gpsd.socket
```
Disable gpsd from starting automaticatlly.

```
$ sudo systemctl disable gpsd
Synchronizing state of gpsd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable gpsd
```

Next edit the /lib/systemd/system/gpsd.socket file and change this line ListenStream=127.0.0.1:2947 to ListenStream=0.0.0.0:2947
```
$ sudo vi /lib/systemd/system/gpsd.socket
```

Double check that no gpsd processes are running.
```
$ sudo killall gpsd
gpsd: no process found
```

Now let's bind our USB GPS to gpsd.  And of course this starts our gpsd daemon.
```
$ sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.socket
```

You can check the status gpsd
```
$ sudo systemctl status gpsd
```

Enable gpsd to start automatically on reboot
```
$ sudo systemctl enable gpsd
[sudo] password for xxxxx: 
Synchronizing state of gpsd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable gpsd
Created symlink /etc/systemd/system/multi-user.target.wants/gpsd.service → /lib/systemd/system/gpsd.service.
Created symlink /etc/systemd/system/sockets.target.wants/gpsd.socket → /lib/systemd/system/gpsd.socket.
```

We can run gpsmon to see if our GPS device is working with our PI.  Ctrl-c to stop 
```
$ gpsmon
```

Another tool for same information - CGPS.   Ctrl-c to stop
```
# cgps -s
```


Setting up chronyc as the time server using the GPS USB



Create and Configure GPSD file

```
cd /etc/default
vi gpsd
```

gpsd file contents
```
START_DAEMON="true"

USBAUTO="true"

DEVICES="/dev/ttyUSB0"

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
[Raspberry Pi 4 GPS Install](https://www.youtube.com/watch?v=isVHkovZuSM)  
[How to Setup GPS Tracker for Raspberry Pi](https://www.youtube.com/watch?v=A1zmhxcUOxw)















