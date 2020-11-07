# arm64 Debian - ADSB Display Setup for FlightAware

#### 1.)  Follow the instructions in the file `arm64 Debian Basic Install for Rock64` or `arm64 Debian Basic Install for PineH64B`, a 16GB eMMC module is sufficient as the whole setup requires only 2.5GB in space when finished.

#### 2.)  Install sudo    (log-in as root)

`apt install sudo`

`nano /etc/sudoers`     scroll down to `User privilege specification` copy `root` for your specific username and save

#### 3.)  Set network to static IP address

`sudo nano /etc/network/interfaces`

     # This file describes the network interfaces available on your system
     # and how to activate them. For more information, see interfaces(5).
     
     source /etc/network/interfaces.d/*
     
     # The loopback network interface
     auto lo
     iface lo inet loopback
     
     # The primary network interface
     auto eth0
     allow-hotplug eth0
     iface eth0 inet static
               address 192.168.1.xxx
               netmask 255.255.255.0
               gateway 192.168.1.xxx
               dns-nameservers 192.168.1.xxx

replace `xxx` with your relevant/desired subaddress

`sudo nano /etc/resolv.conf`

`nameserver 192.168.1.xxx`      `xxx` should match your DNS Server

#### 4.) Install system monitoring

`sudo apt install hddtemp lm-sensors glances python3-pip htop screenfetch inxi`

`sudo nano /etc/glances/glances.conf`     comment / uncomment various sections as required

`sudo nano /lib/systemd/system/glances.service`

    [Unit]
    Description=Glances
    After=network.target
    
    [Service]
    ExecStart=/usr/bin/glances -w
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target

`sudo systemctl status glances.service`     check that glances is running and pressing ‘q’ returns to console

`sudo reboot`     to restart the machine and all systemd services, once machine is up again check `192.168.1.xxx:61208` if glances is running correctly.

`senors`      shows all data gathered by lm-sensors

`glances`     shows a variety of system/machine data which can be configured by changing `/etc/glances/glances.conf`

#### 5.)  Install xfce4 Desktop Environment

`sudo apt install xfce4 xfce4-power-manager-plugins`

`sudo systemctl disable lightdm.service`

`sudo apt purge lightdm`

`sudo apt autoremove`

`sudo apt autoclean`

#### 6.)  Hide xfce panel for Firefox full screen (kiosk mode)

`startxfce4`

then click Applications → Settings → Panel
Tab → Display, Section → General
Automatically hide the panel → Always

Do the above for Panel 1 and Panel 2 and don’t forget the Display Power Management settings ;-)

Close xfce4 and return to concole.

#### 7.)  Console auto log-in and start XFCE at boot/log-in

`nano /home/youruser/.bashrc`     add the following to the END of file

    if [ "$(tty)" = "/dev/tty1" ]; then
                  startxfce4
    fi

`sudo nano /etc/systemd/logind.conf`      uncomment `#NAutoVTs=6` and set to `NAutoVTs=2`

`sudo mkdir /etc/systemd/system/getty@tty1.service.d`

`sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf`

    [Service]
    ExecStart=
    ExecStart=-/usr/sbin/agetty --autologin youruser --noclear %I $TERM
    Type=simple

`sudo systemctl enable getty@tty1`

`sudo reboot`

#### 8.)  Make Firefox full screen and autostart

`sudo apt install firefox-esr webext-ublock-origin-firefox webext-privacy-badger`

and then install `“Auto Fullscreen”` Add-on by `tazeat`

`mkdir .config/autostart`

`nano .config/autostart/FlightAware.desktop`

    [Desktop Entry]
    Name=FlightAware
    Exec=firefox http://192.168.1.xxx:8080
    StartupNotify=true
    Hidden=false
    Terminal=false
    Type=Application

`xxx` should match your FlightAware Feeder Address → see my repo `arm64 Debian - ADSB Receiver Setup (FlightAware)` for details

`sudo reboot`

Done, enjoy your new ADSB display !
