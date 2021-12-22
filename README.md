# Setting Up a Raspberry PI 4 to Be a Time Server

Last updated: 12/22/2021

Instructions for Setting Up GPS USB as a stratum 0 time server for you network environment.

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

Start by installing chrony
```
$ sudo apt-get install chrony
```

Edit the chrony conf file to use our USB GPS
```
$ sudo vi /etc/chrony/chrony.conf
```

Add the following line to the end of the file
```
# USB GPS
refclock SHM 0 offset 0.5 delay 0.2 refid NMEA prefer
```

Start and enable chrony
```
$ sudo systemctl start chrony
$ sudo systemctl enable chrony
```

Check to see if chrony is using your USB GPS for the primariy time source. 
```
$ chronyc sources -v
210 Number of sources = 5

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
#* NMEA                          0   4     0   29m  -1053us[ -842us] +/-  144ms
^- t2.time.bf1.yahoo.com         2  10   377   673    -86ms[  -86ms] +/-   73ms
^- static-72-87-88-202.prvd>     2  10   377   659    -85ms[  -85ms] +/-   64ms
^- ntp05.cymru.com               3  10   377   672    -88ms[  -88ms] +/-  110ms
^- 38.229.52.9                   2   9   377   514    -87ms[  -87ms] +/-  109ms
pslucas@ns01:/etc/chrony $ 
```
The #* by NMEA indicates that chrony is using your USB GPS as its local time source. Note that this a stratum 0 time source.

You can also verify that the USB GPS is in use with the following:
```
$ sudo chronyc tracking
Reference ID    : 4E4D4541 (NMEA)
Stratum         : 1
Ref time (UTC)  : Wed Dec 22 17:47:44 2021
System time     : 0.000000009 seconds fast of NTP time
Last offset     : +0.000210558 seconds
RMS offset      : 0.046488557 seconds
Frequency       : 8.279 ppm fast
Residual freq   : +25.147 ppm
Skew            : 0.123 ppm
Root delay      : 0.200000003 seconds
Root dispersion : 0.102880307 seconds
Update interval : 16.0 seconds
Leap status     : Normal
```

Notice that the Referene refers to NMEA which is our USB GPS


### References 
[Raspberry Pi Buster – GPS Dongle as a time source with Chrony & Timedatectl](https://photobyte.org/raspberry-pi-stretch-gps-dongle-as-a-time-source-with-chrony-timedatectl/)
[Raspberry Pi 4 GPS Install](https://www.youtube.com/watch?v=isVHkovZuSM)  
[How to Setup GPS Tracker for Raspberry Pi](https://www.youtube.com/watch?v=A1zmhxcUOxw)

