# rpi0w-revival
This might not be the best place for instructions on how to setup a raspberry pi zero W to operate as kiosk displaying a webpage in 2026 but what else am I going to do?  Start paying someone to host a webpage for me?

## Rant
Single board computer prices are wildly increasing right now.  The former fame of the $35 raspberry pi is gone.  On august 16th 2026 it was $94 for a rpi 4b with 2GB memory, $209 for a rpi 5 with 8GB RAM and $399 for rpi 5 with 16GB RAM starter kit or $305 for the board by itself.  Rpi zero 2W is only $17... HAHA jokes on me... it's out of stock.  If I spend $89 I can get it at Canna as part of a kit with a stupid case, heat sink, and power supply.   Hmm no thanks to all of the above.<br>

I remember when I could spend somewhere around $20 and get a pi zero w then spend $20 more for any cables or maybe a power supply and try out a new os or new software project without taking out a loan or dipping into savings (I'm kidding).<br>

Fast forward to the age where even I can setup home lab software (self depricating but don't worry I have an ego too to keep in check).  I setup Home Assistant on my Rpi 5 and I like it... but now I want my Pi 5 back and really won't buy a replacement with these crazy prices.  So I have the Home Assistant project running on my RPi 3B+ (barely but it works).  Lights and power outlets can be turned on and off using matter, thread, and the ZBT-1 hardware for a thread border router. I even jumpered a capacitor to 5V and Gnd pins so to get rid of the constant under voltage or brownout problem.  Thanks Taulab project for making me learn about brownouts and under engineered power paths!<br>

Now I want to setup a Magic Mirror display (at first I thought of Dakboard, but why not self host MM2 instead).  I have two Pi Zero Ws left and I thought this HW should be sufficient.  Little did I know software bloat crept in, mainly with the browser, while I wasn't looking.  I tried using the latest 2025 Raspbian Trixie with chromium and then Firefox-esr and both failed to run due to a missing instruction set that doesn't exist on the Arm6 in the RPi Zero W.

## Problem Statement
New Raspberry Pis are heavily affected by the memory cost increase affecting all computers currently in 2026 and they are no longer a good value.  I'll be using older raspberry pi zero W hardware to build a simple Magic Mirror 2 display or possibly a Dakboard display.  The issue with using older hardware is modern web browser version will not run.

## Initial Configuration 260820
I have found promising configuration:
- 32-bit DietPi
- DRM/KMS
- Wayland Graphics packages
- Cage
- Cog
- WPE Webkit

The following were my setup and configuration steps:
- Using RPi Imager I flashed the current DietPi 32-bit RPi image released 2026-08-15 to an SD card.  Device > RPi Zero > OS > Other general-purpose OS > DietPi > DietPi OS (32-bit).
- Edit the dietpi.txt file on the SD card
```
# Select setup choices based on your location
AUTO_SETUP_KEYBOARD_LAYOUT=us
AUTO_SETUP_TIMEZONE=America/Los_Angeles

# Disable ethernet and enable wifi to match RPi Zero HW
AUTO_SETUP_NET_ETHERNET_ENABLED=0
AUTO_SETUP_NET_WIFI_ENABLED=1
AUTO_SETUP_NET_WIFI_COUNTRY_CODE=US

AUTO_SETUP_NET_HOSTNAME=myHostname

# Disable serial console (unless you want to debug with RS-232)
CONFIG_SERIAL_CONSOLE_ENABLE=0

# Enable Logind since it provides some setup steps we require
AUTO_UNMASK_LOGIND=1

# Use DietPi to install needed packages automatically
AUTO_SETUP_APT_INSTALLS=cage cog fonts-dejavu-core fontconfig ca-certificates

# Use Dropbear ssh to keep things diet
AUTO_SETUP_SSH_SERVER_INDEX=-1

# Disable auto apt checks since they nearly max out system memory
# apt can be run by disabling the display service (kiosk.service) then run apt updat && apt upgrade
CONFIG_CHECK_APT_UPDATES=0


# This must be set to actually run the automated install
AUTO_SETUP_AUTOMATED=1

# Improve security
SOFTWARE_DISABLE_SSH_PASSWORD_LOGINS=root
```

- Edit dietpi-wifi.txt and set your wifi ssid and password
```
aWIFI_SSID[0]='ssid-goes-here'
aWIFI_KEY[0]='wifi-password-goes-here'
```
- When the system is up, run sudo dietpi-launcher and make the following additional changes
  - Dietpi-display > adjust any screen resolution or rotation here
  - Dietpi-config > Display Options > KMS/DRM [On]
  - Dietpi-config > Display Options > GPU/RAM memory split > 64 Full GUI/Desktop/Default
  - Dietpi-config > Display Options > Rpi Codecs [On]
  - Dietpi-config > Display Options > adjust any display brightness here
  - Dietpi-config > Security Options > change any passwords as needed for security
- Create the necessary systemd unit file to create a kiosk service
  - sudo nano /etc/systemd/system/kiosk.service
```ini
[Unit]
Description=WebKit kiosk display
Requires=seatd.service
After=seatd.service systemd-user-sessions.service
Conflicts=getty@tty1.service
Before=getty@tty1.service

[Service]
Type=simple
User=dietpi
Group=dietpi
SupplementaryGroups=video render input
RuntimeDirectory=cage
RuntimeDirectoryMode=0700
Environment=XDG_RUNTIME_DIR=/run/cage
Environment=LIBSEAT_BACKEND=seatd
Environment=SEATD_SOCK=/run/seatd.sock
TTYPath=/dev/tty1
StandardInput=tty
StandardOutput=journal
StandardError=journal
TTYReset=yes
TTYVHangup=yes
TTYVTDisallocate=yes

ExecStart=/usr/bin/cage -- /usr/bin/cog https://wpewebkit.org/

Restart=always
RestartSec=3

[Install]
WantedBy=graphical.target
```

- Run the following commands to 1) add the necessary groups 2) enable the seatd service, 3) load our new kiosk service, 4) enable the kiosk service
```
sudo usermod -aG video,render,input dietpi
sudo systemctl enable --now seatd.service
sudo systemctl daemon-reload
sudo systemctl enable --now kiosk.service
```

## Additional Configuration 260821
```
sudo apt install kms++-utils
kmsprint 
kmsprint -m
```

Edit /etc/modules-load.d/modules.conf and add one line i2c-dev
```
sudo nano /etc/modules-load.d/modules.conf
i2c-dev
```
verify with lsmod | grep i2c[-_]dev

Now its possible to run ddc utilities
```
sudo ddcutil detect
sudo ddcutil probe
```

Usefull commands are to run cat on the files in /sys/class/drm/card1-HDMI-A-1/
```cat /sys/class/drm/card1-HDMI-A-1/modes```

Set the display brightness in half and watch the power usage drop.  For me it went from 4.5W -> 3.2W
```sudo ddcutil setvcp 10 50```

## Changing the launch of the kiosk to a dedicated system user
```
sudo useradd  --system --create-home --home-dir /var/lib/kiosk --shell /bin/bash kiosk
sudo passwd --lock kiosk
sudo nano /etc/pam.d/cage
auth       required     pam_unix.so nullok
account    required     pam_unix.so
session    required     pam_unix.so
session    required     pam_systemd.so
```
Edit the kiosk.service file to change the user and a reboot should do it.

## Issues, Enhancements, and Things to Work On
- The system memory usage is pushed to the max.  I think the automatically run apt update is crashing the web browser.  Need to try an apt update routine that first unloads the wpe webit to free memory, displays something to the user to say maintenance, and runs apt.
  - apt qq list command (or something like it ran) with a subprocess of wc -l
  - cpu and men usage was at 100% 
- When I reviewed dmesg after a crash it showed one OOM error.  I need to capture log info better
- 

