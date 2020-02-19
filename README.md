# Pi_config

Install log/protocol for NUSTEM's family of Pis

## Raspberry Pi NUSTEM distribution image creation log 2020-02

### Core burn:

Raspbian Buster dated **2020-02-05**. Burned using Etcher.

### Setup wizard

The defaults here are usually pretty good. Check keyboard settings -- most of NUSTEM's keyboards are UK-spec, though (annoyingly) not all. Skip the password change and updates for now.

### Initial updates

Connect to WiFi: WiFiGuest, then visit neverssl.com in Chromium to complete connection.

    sudo apt update
    
Only about 180 Mb of updates, as of 2020-02-11.

    sudo apt upgrade -y
    
Takes ages - incredible number of boot overlays. I've long thought that shoving all these under `/usr/share/rpikernelhack` must be alarming for newcomers.

    sudo apt autoclean
    sudo apt autoremove

### Network Manager

Let's get proper networking going first. This is also the most fragile part of the install, so let's get this out of the way early in case we break all the previous steps and have to start over.

The default Raspbian networking stack fails to connect to the corporate wifi networks. NetworkManager can fix that. Previously-used install approaches didn't quite work with the 2020-02-06 Buster release, but this did (on a clean install):

    sudo apt install network-manager network-manager-gnome

Check `/etc/network/interfaces`. Should be empty except for an include from `/etc/network/interfaces.d`.

Edit `/etc/dhcpcd.conf`, add line:
    denyinterfaces wlan0

`sudo systemctl restart dhcpcd`:

edit `/etc/NetworkManager/NetworkManager.conf`, amend to:

    [main]
    plugins=ifupdown,keyfile
    dhcp=internal
    
    [ifupdown]
    managed=true    

`sudo reboot`.

Connect to NU Simply Web from the Network Manager meny bar widget, using PEAP, 'no certificate required', NUSTEM account (ee071).

#### Previous Network Manager stuff

**This section retained for reference, for when things next change.**

https://davidxie.net/install-network-manager-on-raspbian was very useful, but now appears out-of-date""

    sudo apt install network-manager network-manager-gnome resolvconf --download-only
    sudo nano /etc/network/interfaces
        remove everything except:
            auto lo
            iface lo inet loopback
        ...only, there's nothing in that file except an include from /etc/network/interfaces.d/, which is in turn empty.
    sudo apt install network-manager network-manager-gnome resolvconf
    reboot
    sudo apt purge dhcpcd5
    sudo ln -sf /lib/systemd/resolv.conf /etc/resolv.conf
    sudo usermod -G netdev -a pi
    reboot

At this point the menu bar includes both Network Manager and whatever Raspbian uses as default, the latter disabled.

We still don't have routing. `sudo nano /lib/systemd/resolv.conf` to edit a file we shouldn't edit, and hard-wire the university's DNS in place:

    nameserver 192.168.60.50

Comment out everything else.

Then:

    sudo service systemd-resolved restart
    sudo systemctl restart networking

...and we have most networking. No `ping`, but `traceroute`, `apt` and web access all work as expected. Blimey.

Remove the old network widget from the menu bar and faff about with spacers until it looks a bit less rubbish.

### Wallpaper

Transferred via nustem web share (URL redacted from this public record -- ask Jonathan). Saved to ~/Pictures and set. 

`fbset -s` is a handy way of finding current display resolution.

### Misc

* Set time to Europe/London (appears default as of Buster)
* in `~/.bashrc`, turn on `ll`
* Preferences -> Raspberry Pi Configuration -> Interfaces: Enable Camera, SPI, I2C. SSH not enabled by default, since we run our Pis without a password.

Now restart.

### Reconfigure menu bar launchers

As of 2020 Mathematica no longer has a menu bar launcher by default (hurray). So just add Thonny to the launcher (right-click, select 'Application Launch Bar')

It's worth considering whether to purge Mathematica, Wolfram and LibreOffice. I haven't this time around, pending Pi 4 plans and on the assumption that we'll retire our 8Gb cards.

#### CPU Usage Monitor

Via Panel Applet preferences - right-click on the panel strip. Add, with percentage gauge. I can believe this isn't of much use on a Pi4, but on a 3 it's very handy to see if Chrome is spinning plates in the background.

### Pi-Top install

We're working mostly on Pi-Top CEED desktop units. It's worth installing the Pi-Top software to allow brightness control and soft power-down.

    sudo apt install pt-device-manager

...gets you the `pt-brightness` command. `pt-brightness -h` offers help. Full brightness: `pt-brightness -b 10`. `pt-brightness --timeout 0` doesn't appear to do what one would hope, expect, or is documented.

### Python libraries, etc.

#### Pimoroni

Pimoroni install scripts rely on checking the network connection by pinging 8.8.8.8 (Google DNS). That's blocked on my local network, so the scripts think they're offline and refuse to proceed. But it turns out the libraries are all preinstalled in current Raspbian anyway (!), so the only thing I'm missing is the code examples. Ignore for now.

##### Pimoroni -- held back for now.

`sudo apt install pimoroni` to install the Pimoroni Dashboard, which is then be in the Accessories group (or `pimoroni-dashboard` at a prompt). While this has been minimally patched for Buster, it's not up-to-date with more recent Pimoroni HATs/pHATs. It's also extremely slow, since it checks everything every time. Ugh ugh ugh.

Ended up installing via the following scripts anyway:

    Blinkt!         curl https://get.pimoroni.com/blinkt | bash             -- 
    Displayotron    curl https://get.pimoroni.com/displayotron | bash       -- 
    Enviro pHAT     curl https://get.pimoroni.com/envirophat | bash         -- 
    Explorer HAT    curl https://get.pimoroni.com/explorerhat | bash        --
    MicroDot pHAT   curl https://get.pimoroni.com/microdotphat | bash       -- 
    Piano HAT       curl https://get.pimoroni.com/pianohat | bash           -- doesn't work on pHAT Stack, but tested directly.
    Scroll pHAT     curl https://get.pimoroni.com/scrollphat | bash         -- 
    Scroll pHAT HD  curl https://get.pimoroni.com/scrollphathd | bash       -- 
    Unicorn HAT     curl https://get.pimoroni.com/unicornhat | bash         --
    Unicorn HAT HD  curl https://get.pimoroni.com/unicornhathd | bash       -- 
    Speaker pHAT    not installed in default image -- resets Pi audio configuration
    Skywriter       curl get.pimoroni.com/skywriter | bash                  -- not installed
    Inky pHAT       sudo pip3 install inky                                  -- doesn't work on pHAT Stack

* ScrollpHATHD throws an smbus error on install but appears to work.
* Inky pHAT test code (which isn't installed via pip; the script installers are better in this sense) uses font Hanken, which isn't installed in Buster. `pip3 install font-hanken-grotesk`. Bug report filed. Also `pip3 install font-intuitive`. 

Notes:

* Choose a 'full install' in each case, and the script will put examples into a `~/Pimoroni` directory. Explorer HAT installer also covers the pHAT.
* Speaker pHAT not installed -- disables on-board audio.

###### Note re: Skywriter HAT

This left over from previous installs:

> Getting the Skywriter HAT to work is hilarious. Mouse control needs autopy, which needs setuptools-rust, which needs Rust, which needs:

>     curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

> ...which works, but then autopy fails to compile with a rustc compiler error E0554. Turns out it needs Rust nightly build; code uses a `feature` structure which is regarded as unstable-only. Whaaaaaat?!

...I've not bothered doing that on this pass.


#### Paho-MQTT library

    pip3 install paho-mqtt

### VS Code editor

This wasn't being compiled for quite a long time, but the (third-party) builds are now up again.

    wget https://code.headmelted.com/installers/apt.sh
    chmod +x apt.sh
    sudo ./apt.sh
    
    wget https://packagecloud.io/headmelted/codebuilds/gpgkey
    sudo apt-key add gpgkey

Launch, and install Python extension. This was problematic in Feb 2020 - download via Chrome failed rather a lot, and the website makes it tricky to find the direct download link. I think the network was just being flaky; it eventually installed.

* Install pylint as recommended. (or `pip3 install pylint`, which I think is what vscode does behind the scenes) (2020-02-11: pylint already installed)
* Turn off Minimap view.


#### VS Code -- legacy

In the old days:

    sudo apt update
    sudo apt install code-oss=1.29.0-1539702286
    sudo apt-mark hold code-oss
    
...installed a working version. Then:

    sudo apt-mark unhold code-css

... got us back to current head.

### Fonts

    sudo apt install fonts-hack fonts-inconsolata

Turns out both Inconsolata and Hack render rather oddly on the Ceed display / with whatever LCD hinting Raspbian uses. Stick with Monospace 10. Terminal size 96x30.

### Shell comfort zone

#### locate

    sudo apt install mlocate
    sudo updatedb

#### tldr

There are issues here with `npm`, which (as installed) doesn't seem to work with the version of node that's installed. 
    
    sudo npm i npm@latest -g
    sudo npm install -g tldr

Copy the `config` file from `https://github.com/tldr-pages/tldr-node-client` to `.tldrrc`, set theme (on last line) to `base16`. Then run `tldr tar` to cache pages (or any other tldr lookup).

#### howdoi

    sudo apt install libxml2-dev python3-lxml python-lxml libxslt-dev python-dev
    pip3 install howdoi
    
`.profile` includes the installation directory, but needs `source .profile` (or a new bash session) to pick it up.

#### neofetch

    sudo apt install neofetch

add to `.bashrc`:
    
    neofetch

### Misc. modules etc.

These are mostly to support our existing codebases.

    pip3 install fuzzywuzzy python-Levenshtein termcolor
    pip3 install pyqrcode pypng reportlab 
    
#### Things we previously installed which now seem to be standard, and can therefore ignore

These were all under pip/pip3:

    python-osc
    guizero
    pgzero

## Arduino

Oof. OK, let's get the installer... `https://arduino.cc`, then head to Software -> Downloads. Get Linux ARM 32bit (Raspberry Pi user). Double-click (or tar xvf) downloaded package to uncompress (takes a while). cd into the directory and `sudo ./install.sh`.

Launch (interesting: 1.8.11 editor and menu fonts *aren't* cruddingly awful on Raspbian, which is weirdly not the case on Ubuntu x64 of late), do the 'additional board manager' thing, with:
    `http://arduino.esp8266.com/stable/package_esp8266com_index.json`.
    
Boards Manager -> ESP8266. 'Verifying archive integrity' takes ages.



## Processing

    curl https://processing.org/download/install-arm.sh | sudo sh
    
This doesn't appear to be documented as previously on the Processing site, but still seems to work as of 2020-02-19. Installs 3.5.3, which bleats that 3.5.4 is available, only it's not (it's not been compiled for ARM).

### Set audio output

...to Analogue. (right-click on menu bar speaker icon)

### Password

Reset using Pi GUI tool (not `passwd` on the command line, which does sanity checking for rubbish passwords).

New password: **nustem**

### Browser quick links

* nustem.uk
* connect.nustem.uk
* microbit.org
* projects.raspberrypi.org @done

* Node Red (127.0.0.1:1880) @done

* any more?

### Deploy

We're heavily out of space on an 8Gb card - not going to be able to install Arduino or Processing. Boo!

Make `.iso` image:

* Make an image from the SD Card reader, save it as a `.dmg`
* Convert image (Images menu) to 'CD/DVD master'. Resave.
* Change new file name from `.cdr` to `.iso`

Process through [PiShrink](https://github.com/Drewsif/PiShrink) so the image is as small as possible for bulk writing. I'm running PiShrink via an Ubuntu virtual machine and a shared folder (VirtualBox, which is clunky but works.)

    sudo pishrink.py [source] [dest]

It's worth omitting the `[dest]` if it's already a converted copy of the `.dmg`, as the first step is a copy... which takes some time for a 32Gb card.

(note: I'm working from a host/guest shared folder in a VirtualBox Ubuntu 18.10 machine. To read/write the shared folder without sudo permissions, I need to (in the Ubuntu guest OS):

    sudo usermod -aG vboxsf <someuserID>
    
and log out/in. )

Boot from card, run `sudo raspi-config`, select 'Advanced options`, resize root file system, reboot.


### Last bits

    pip3 install rpi_ws281x
    pip3 install pyowm pyquery requests
    sudo pip3 install adafruit-circuitpython-neopixel
    

---

### TODO: Next time around

* Install the current Adafruit CircuitPython & NeoPixel libraries (NB. they're actually serious about sudo for this!) (also note that Adafruit's 'NeoPixels on a Raspberry Pi' page is five years old as of 2019-07-24, and doesn't work on Buster. Sigh. Looks like we just need `rpi-ws281x-python`.)
  * This mostly worked: https://thepihut.com/blogs/raspberry-pi-tutorials/using-neopixels-with-the-raspberry-pi
  * 


---

## Ommitted from 2019 build:

#### PiCap **OMMITTED**

**NB. as of October 2017 the PiCap distribution did gnarly things to other bits and I ended up starting over. Caution!**

Basic `apt install picap` fails with a mosquitto dependency. Pending fixes:

        apt-get download picap
        sudo apt-get install wiringpi, libsdl2-mixer-dev python-liblo python-pygame liblo-dev libav-tools espeak
        sudo dpkg -i --ignore-depends=python-mosquitto --ignore-depends=libmosquitto-dev picap_1.3.0_armhf.deb

Ugh.

        picap-setup

Example code installed into ~/PiCapExamples. Node installed under NVM.

        picap-intro

...to run the tour.

#### Nano syntax highlighting

> Already included in Buster install -- retained here for reference.

In `~/.nanorc`:

    include /usr/share/nano/css.nanorc
    include /usr/share/nano/python.nanorc
    include /usr/share/nano/html.nanorc
    include /usr/share/nano/json.nanorc
    include /usr/share/nano/sh.nanorc
    include /usr/share/nano/javascript.nanorc


### Node Red updates

> Ommited 2019-07-10, Node install version is 10.15.2, which is recent enough.
> Actually... 2019-07-11 I might be doing it anyway, as I later ran into npm version issues.
> ...but this doesn't work as of 2019-07-11 on Buster: latest update to the script was 20 hours ago
> so looks like it's being worked on currently. 
> https://github.com/node-red/raspbian-deb-package/commits/master/resources/update-nodejs-and-nodered

Referring to [Node-RED documentation](https://nodered.org/docs/hardware/raspberrypi), update Node via:

    bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)

Blimey. That's a bit on the brutal side.

> Doesn't work 2019-07-10: `nvm` package not found

    sudo apt install nvm

Following for reference, not necessary if one proceeds as above:
> Installer throwing all sorts of errors; log file suggests it's not happy about the unmet dependency in the PiCap install, and that's also done something a bit odd with node versions (albeit within nvm). I seem to have got it working with `nvm use system`. Bit worried it may be broken, though.
> 
> Then
>     nvm use lts/*
>     
> as requested. Only, that complains about not having version N/A. `nvm list` lists a bunch of things, and yes, `lts` is redirected to N/A. Huh. Set to `nvm use stable` since that seems to be more recent than `system`.

Enable as system service at boot time:
    sudo systemctl enable nodered.service

### Uninstalling Pimoroni Speaker-pHAT

Bit annoying, but see https://github.com/pimoroni/speaker-phat/issues/17

    copy `/etc/asound.conf` somewhere, delete.

Then in `/boot/config.txt` set lines:

    dtparam=audio=on
    #dtoverlay=i2s-mmap
    #dtoverlay=hifiberry-dac

#### Node-red modules

    cd ~/.node-red
    npm install node-red-contrib-explorerhat
    npm install node-red-node-pi-neopixel
    npm install node-red-dashboard
    npm install node-red-node-pi-sense-hat
