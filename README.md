# Configure your Raspberry PI 4 running Fedora 41 as Time Server for your local devices.

Documentation update in progress - 23 April 2025

Here are the steps I used in setting up a USB GPS with a Raspberry PI 4 to become a stratum 0 time server for my local networked devices.

System Specs:
- raspberrypi Raspberry Pi 4 Model B Rev 1.5
- Fedora Linux 41 (Server Edition)
- GlobalSat SiRF Star IV Serial GPS Receiver Bulk Head Mount (MR-350P-S4)

You may have success with other USB GPS devices.  Ideally you would want to set this up outside, but I have the USB GPS sitting on a first floor window sill facing in a SouthWest-ish direction and I receive strong enough signals for it to work.

### Software Installation  

We will use the Chrony package that comes with Fedora 41 for our time server

Install the GPS Daemon gpsd and the GPS Daemon client gpsd-clients
```
# dnf install -y gpsd gpsd-clients
```
### Configuring the software for the USB GPS

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

Update the gpsd file in the /etc/sysconfig directory.  Make sure your file looks like this...
```
# Options for gpsd, including serial devices
OPTIONS=""
# Set to 'true' to add USB devices automatically via udev
START_DAEMON="true"
USBAUTO="true"
DEVICES="/dev/ttyUSB0"
GPSD_OPTIONS="-n"
```

Restart the gpsd deamon
```
# systemctl restart gpsd
```

If no errors run cgps.   Ctrl-c to stop.  You should see a real time list of satellites your GPS USB fob is recieving
```
# cgps -s
┌─ssssssssssssssssssssssssssssssssssssssssss┐┌─222222222222222Seen 12/Used  9──┐
│ Time         2025-04-23T20:19:46.000Z ( 0)││GNSS  S PRN  Elev  Azim   SNR Use│
│ Latitude          42.16710814 N           ││GP  5     5  51.0 208.5  38.8  Y │
│ Longitude         87.78966890 W           ││GP  6     6  26.5  70.5  19.8  Y │
│ Alt (HAE, MSL)     871.887,    983.085 ft ││GP 11    11  58.0  51.0  20.4  Y │
│ Speed              0.00               mph ││GP 12    12  39.0 219.0  31.8  Y │
│ Track (true, var)	  0.0,     -4.0     deg ││GP 13    13   2.0 157.5  19.9  Y │
│ Climb              0.00            ft/min ││GP 19    19   7.5 124.5  18.5  Y │
│ Status          3D DGPS FIX (6 secs)      ││GP 20    20  84.0  90.0  26.3  Y │
│ Long Err  (XDOP, EPX)   0.79, +/-  9.7 ft ││GP 25    25  39.0 267.0  40.0  Y │
│ Lat Err   (YDOP, EPY)   1.82, +/- 22.4 ft ││SB133   133  25.5 232.5  36.5  Y │
│ Alt Err   (VDOP, EPV)   1.93, +/- 36.4 ft ││GP 18    18  43.5 213.0   0.0  N │
│ 2D Err    (HDOP, CEP)   1.60, +/- 24.9 ft ││GP 23    23  24.0 109.5   0.0  N │
│ 3D Err    (PDOP, SEP)   1.60, +/- 24.9 ft ││GP 26    26   8.0 135.0   1.5  N │
│ Time Err  (TDOP)        1.63              ││                                 │
│ Geo Err   (GDOP)        3.22              ││                                 │
│ Speed Err (EPS)            +/- 30.6 mph   ││                                 │
│ Track Err (EPD)         n/a               ││                                 │
│ Time offset             0.919352534     s ││                                 │
│ Grid Square             EN62ce50          ││                                 │
│ ECEF X, VX     599114.178 ft    0.000 ft/s││                                 │
│ ECEF Y, VY  -15522418.097 ft    0.000 ft/s││                                 │
│ ECEF Z, VZ   13974927.928 ft    0.000 ft/s││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
│                                           ││                                 │
└─aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa┘└─                                ┘

```

We want the chronyd service running as pre-req to running the gpsd service
Add 'Wants=chronyd.service' to the gpsd.service file found in the /etc/systemd/system.  Place the line before 'After=chronyd.service'.  Your file will look something like this...
```
[Unit]
Description=GPS (Global Positioning System) Daemon
Requires=gpsd.socket
# Needed with chrony SOCK refclock
Wants=chronyd.service
After=chronyd.service
```
The chronyd package comes installed and enabled by default in Fedora 41.  We 
Edit the /etc/chrony.conf file.

- Add:
```
# Connect GPS USB
refclock SHM 0 refid GPS poll 2 precision 1e-3 offset 0.128
```

- remove hash in front hwtimestamp *
```
# Enable hardware time stamping on all interfaces that support it.
hwtimestamp *
```
Make sure gpsd is stopped and then restart chronyd and gpsd.  If everything is setup correctly you should have startum 0 time server running

```
# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
#* GPS                           0   2   273     3  +4666us[+8304us] +/-   17ms
```

The #* by GPS indicates that chrony is using your USB GPS as its local time source. Note that this a stratum 0 time source.

You can also verify that the USB GPS is in use with the following:
```
# chronyc tracking
Reference ID    : 47505300 (GPS)
Stratum         : 1
Ref time (UTC)  : Wed Apr 23 20:13:40 2025
System time     : 0.000354952 seconds slow of NTP time
Last offset     : -0.001120362 seconds
RMS offset      : 0.000764950 seconds
Frequency       : 46.082 ppm fast
Residual freq   : -2.609 ppm
Skew            : 39.195 ppm
Root delay      : 0.000000001 seconds
Root dispersion : 0.008200806 seconds
Update interval : 12.0 seconds
Leap status     : Normal
```
Notice that the Referene refers to GPS which is our USB GPS


# <Old Doc - Schedule for deletion>
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
$ sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock
```

You can check the status gpsd
```
$ $ sudo systemctl status gpsd
● gpsd.service - GPS (Global Positioning System) Daemon
   Loaded: loaded (/lib/systemd/system/gpsd.service; enabled; vendor preset: ena
   Active: active (running) since Fri 2022-03-25 14:17:07 CDT; 1 months 4 days a
  Process: 659 ExecStart=/usr/sbin/gpsd $GPSD_OPTIONS $DEVICES (code=exited, sta
 Main PID: 667 (gpsd)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/gpsd.service
           └─667 /usr/sbin/gpsd -n -D 1 /dev/ttyUSB0
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
$ cgps -s
```


### Setting up chronyc as the time server using the GPS USB

Create and Configure GPSD file

```
$ cd /etc/default
$ vi gpsd
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

### Configure Chrony 

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

