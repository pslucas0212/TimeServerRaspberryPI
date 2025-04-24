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
│ Track (true, var)	  0.0, -4.0     deg ││GP 13    13   2.0 157.5  19.9  Y │
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

- Comment out other time servers:
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
# pool 2.fedora.pool.ntp.org iburst
# server 0.us.pool.ntp.org iburst
# server 1.us.pool.ntp.org iburst
# server 2.us.pool.ntp.org iburst
# server 3.us.pool.ntp.org iburst
```

- Add:
```
# Connect GPS USB
refclock SHM 0 refid GPS poll 2 precision 1e-3 offset 0.128
```
- Remove the hash in front of stratum preference setting
```
# Prefer servers with the lowest stratum regardless of their distance
stratumweight 2
```
- remove hash in front hwtimestamp *
```
# Enable hardware time stamping on all interfaces that support it.
hwtimestamp *
```

- In the Allow section add any subnets that need access to chronyd as a time source
```
# Allow NTP client access from local network.
allow x.x.x.x/24
allow x.x.x.x/24
```

- Finally update the firewall-cmd to allow clients to access the ntp service on your time server
```
# firewall-cmd --add-service=ntp --permanent 
# firewall-cmd --reload
# firewall-cmd --list-all
FedoraServer (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: end0
  sources: 
  services: cockpit dhcpv6-client dns mdns ntp ssh
  ports: 53/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
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




### References 
[Raspberry Pi Buster – GPS Dongle as a time source with Chrony & Timedatectl](https://photobyte.org/raspberry-pi-stretch-gps-dongle-as-a-time-source-with-chrony-timedatectl/)  
[Raspberry Pi 4 GPS Install](https://www.youtube.com/watch?v=isVHkovZuSM)  
[How to Setup GPS Tracker for Raspberry Pi](https://www.youtube.com/watch?v=A1zmhxcUOxw)

