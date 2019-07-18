# Pi_config

Install log/protocol for NUSTEM's family of Pis

## Raspberry Pi NUSTEM distribution image creation log 2019-07-16

### Core burn:

Raspbian Buster dated **2019-07-10**. Burned using Etcher.

### Setup wizard

The defaults here are usually pretty good. Check keyboard settings -- most of NUSTEM's keyboards are UK-spec, though (annoyingly) not all. Skip the updates and password change for now.

### Initial updates

Connect to WiFi: WiFiGuest, then visit neverssl.com in Chromium to complete connection.

    sudo apt update

(with the 2019-06-20 Buster release, this requires agreement to a weird 'repository changed its Suite value from testing to stable' alert. This appears to be fixed with 2019-07-10.)

As of 2019-07-12, 437Mb of updates already, including pretty much all of Node, raspberrypi-kernel, raspberrypi-ui-mods, raspberrypi-bootloader, and more. Blimey. The big files are mostly openjdk stuff, I think. Again, things have settled down massively as of 2019-07-16.

    sudo apt upgrade
    sudo apt autoclean
    sudo apt autoremove


### Wallpaper

Transferred via nustem web share (URL redacted from this public record -- ask Jonathan). Saved to ~/Pictures and set. 

`fbset -s` is a handy way of finding current display resolution.

### Misc

* Set time to Europe/London (appears default as of Buster)
* in `~/.bashrc`, turn on `ll`
* Preferences -> Raspberry Pi Configuration -> Interfaces: Enable Camera, SPI, I2C. SSH not enabled by default, since we run our Pis without a password.

Now restart

### Reconfigure menu bar launchers

Do this by right-clicking the application launch area, for access to the standard (LXDE?) configuration GUI.

* Remove Wolfram and Mathematica
* Add Thonny

(Might wish to purge Wolfram, Mathematica and LibreOffice for space and update speed reasons, though as long as cards are 8Gb or greater I'm not convinced that's worthwhile. As of launch Buster release, Wolfram/Mathematica have been removed pending compatability; as of 2019-07-10 release they seem to be back, albeit unannounced.)

#### CPU Usage Monitor

Via Panel Applet preferences - right-click on the panel strip. Add, with percentage gauge. I can believe this isn't of much use on a Pi4, but on a 3 it's very handy to see if Chrome is spinning plates in the background.

### Pi-Top install

We're working mostly on Pi-Top CEED desktop units. It's worth installing the Pi-Top software to allow brightness control and soft power-down.

    sudo apt install pt-device-manager

...gets you the `pt-brightness` command. `pt-brightness -h` offers help. Full brightness: `pt-brightness -b 10`. `pt-brightness --timeout 0` doesn't appear to do what one would hope, expect, or is documented. If memory serves, the way to turn idle screen blanking off on a Raspberry Pi is to install x-screensaver, then configure it so it's switched off. Yes, really.

### Python libraries, etc.

#### Pimoroni

Normally, I'd `sudo apt install pimoroni` to install the Pimoroni Dashboard, which would then be in the Accessories group (or `pimoroni-dashboard` at a prompt). However, as of 2019-07-11 it doesn't seem to work with Buster. The version in the repo looks like it's old, but I also can't install the more recent .deb that's tucked away in Github (but still a couple of years old). Hmm.

So... do this the clunky way:

    Blinkt!         curl https://get.pimoroni.com/blinkt | bash             -- tested
    Displayotron    curl https://get.pimoroni.com/blinkt | bash             -- 
    Enviro pHAT     curl https://get.pimoroni.com/envirophat | bash         -- tested
    Explorer HAT    curl https://get.pimoroni.com/explorerhat | bash        --
    MicroDot pHAT   curl https://get.pimoroni.com/microdotphat | bash       -- tested
    Piano HAT       curl https://get.pimoroni.com/pianohat | bash           -- doesn't work on pHAT Stack, but tested directly.
    Scroll pHAT     curl https://get.pimoroni.com/scrollphat | bash         -- tested
    Scroll pHAT HD  curl https://get.pimoroni.com/scrollphathd | bash       -- tested
    Unicorn HAT     curl https://get.pimoroni.com/unicornhat | bash         --
    Unicorn HAT HD  curl https://get.pimoroni.com/unicornhathd | bash       -- tested
    Speaker pHAT    not installed in default image -- resets Pi audio configuration
    Skywriter       curl get.pimoroni.com/skywriter | bash
    Inky pHAT       sudo pip3 install inky                                  -- tested (doesn't work on pHAT Stack)

* ScrollpHATHD throws an smbus error on install but appears to work.
* Inky pHAT test code (which isn't installed via pip; the script installers are better in this sense) uses font Hanken, which isn't installed in Buster. `pip3 install font-hanken-grotesk`. Bug report filed. Also `pip3 install font-intuitive`. 

Notes:

* Choose a 'full install' in each case, and the script will put examples into a `~/Pimoroni` directory. Explorer HAT installer also covers the pHAT.
* Speaker pHAT not installed -- disables on-board audio.

Getting the Skywriter HAT to work is hilarious. Mouse control needs autopy, which needs setuptools-rust, which needs Rust, which needs:

    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

...which works, but then autopy fails to compile with a rustc compiler error E0554. Turns out it needs Rust nightly build; code uses a `feature` structure which is regarded as unstable-only. Whaaaaaat?!

...I've not bothered doing that on this pass.



#### Paho-MQTT library

    pip install paho-mqtt

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

#### Node-red modules

    cd ~/.node-red
    npm install node-red-contrib-explorerhat
    npm install node-red-node-pi-neopixel
    npm install node-red-dashboard
    npm install node-red-node-pi-sense-hat

### VS Code editor

VS Code runs well on the Pi, *but*... recent releases use an Electron version which doesn't support ARM, and hence fail to launch. The only solution is: 

    wget https://code.headmelted.com/installers/apt.sh
    chmod +x apt.sh
    sudo ./apt.sh

This fails, but persevere:

    wget https://packagecloud.io/headmelted/codebuilds/gpgkey
    sudo apt-key add gpgkey

    sudo apt update
    sudo apt install code-oss=1.29.0-1539702286
    sudo apt-mark hold code-oss

Launch, and install Python extension. 

* Install pylint as recommended. (or `pip3 install pylint`, which I think is what vscode does behind the scenes)
* Set integrated terminal to fallback (it'll prompt soon after launch, if you futz around a bit)
* Turn off Minimap view.

> Old, not done, left for reference: Still sort-of needs `/home/pi/.local/bin` in PATH.

## Fonts

    sudo apt install fonts-hack fonts-inconsolata

Turns out both Inconsolata and Hack render rather oddly on the Ceed display. Stick with Monospace 10. Terminal size 96x30.

    npm install git://github.com/adobe-fonts/source-code-pro.git#release

(doesn't seem to have worked)

### Tidying up

> Not needed in 2019 install -- left in for reference

Somehow, we've picked up a bunch of sources.list errors.
    sudo apt --fix-broken install
...removes picap install, and upgrades python[3]-rpi.gpio back to Stretch from Jessie.
    sudo apt upgrade
...needed a couple of passes to pick everything up.

### Shell comfort zone

#### locate

    sudo apt install mlocate
    sudo updatedb

#### tldr

See [TLDR reference](https://tldr.sh).

> Didn't have to do any of the npm version stuff in more recent Buster release, left here for reference. Looks like the npm version is still old, though.

Installing involves us fixing an npm issue; the npm version is too old for the default node version (?!). Uninstalling npm should force a fall-back to the node-installed version, but it seems simplest / safest to:

    sudo npm i npm@latest -g

Which jumps us from npm 5.8.0 to npm ... oh, no it doesn't. Blimey.

    sudo npm uninstall -g npm

Well, that's removed the npm which was in /usr/local/bin/, but the one in /usr/bin/ is still 5.8.0. Looks like we'll have to reinstall node.

...and that doesn't work (see the nodeRED stuff above). However, it did _uninstall_ Node, so:

    sudo apt install node npm

...might get us back running.

    sudo npm install -g tldr

> Back into config as performed:

Copy the `config` file from `https://github.com/tldr-pages/tldr-node-client` to `.tldrrc`, set theme (on last line) to base16. Then run `tldr tar` to cache pages.

#### howdoi

    sudo apt install libxml2-dev python3-lxml python-lxml libxslt-dev python-dev
    pip install howdoi

Add to `.profile`: // Not done on this pass ; already in PATH

    if [ -d "$HOME/.local/bin" ] ; then
        PATH="$HOME/.local/bin:$PATH"
    fi

#### neofetch

    sudo apt install neofetch

add to `.profile`:
    
    neofetch

### Misc. modules etd.

These are mostly to support our existing codebases.

`python-osc` already installed (?!). 

    pip3 install fuzzywuzzy python-Levenshtein termcolor
    pip3 install pyqrcode pypng reportlab 
    


### GUIzero

Already installed in Buster release. Interesting.

    pip3 install guizero

### PyGameZero

Already installed in Buster. Iiiinteresting.

    pip install pgzero

### GraspIO

Not installed in default image -- uses a bespoke OS image.

## Arduino

Oof. OK, let's get the installer... `https://arduino.cc`, then head to Software -> Downloads. Get Linux ARM 32bit (Raspberry Pi user). Double-click downloaded package to uncompress (takes a while). 

(not done - running out of space on 8Gb card)

## Processing

    curl https://processing.org/download/install-arm.sh | sudo sh


### Set audio output

...to Analogue.

### Password

Reset using Pi GUI tool (not `passwd` on the command line, which does sanity checking for rubbish passwords).

New password: **nustem**

### Things to include next time around

* Set up Pylint within Visual Studio Code. Oops. @done
* Arduino 
* OpenCV? Ouchie.
* Browser defaults:
  * neverssl.com
  * microbit.org
  * Node Red (127.0.0.1:1880)
  * projects.raspberrypi.org
  * any more?
* Pimoroni modules & example code @done
* I quite like Neofetch (which I think is an `apt install`?); added to `.bashrc`, splurges a nice logo and system info when a new terminal is opened. Which is a helpful reminder of which system you're interacting with. @done

### Network Manager

Following https://davidxie.net/install-network-manager-on-raspbian:

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
    
Need to add user `pi` to network group:

    sudo usermod -G netdev -a pi
    reboot

At this point the menu bar includes both Network Manager and whatever Raspbian uses as default, the latter disabled.

Connect to NU Simply Web using PEAP, 'no certificate required', NUSTEM account (ee071).

We still don't have routing. `sudo nano /lib/systemd/resolv.conf` to edit a file we shouldn't edit, and hard-wire the university's DNS in place:

    nameserver 192.168.60.50

Comment out everything else.

Then:

    sudo service systemd-resolved restart
    sudo systemctl restart networking

...and we have most networking. No `ping`, but `traceroute`, `apt` and web access all work as expected. Blimey.

Remove the old network widget from the menu bar and faff about with spacers until it looks a bit less rubbish.


### Deploy

We're heavily out of space on an 8Gb card - not going to be able to install Arduino or Processing. Boo!

Make `.iso` image:

* Make an image from the SD Card reader, save it as a `.dmg`
* Convert image (Images menu) to 'CD/DVD master'. Resave.
* Change new file name from `.cdr` to `.iso`

Process through [PiShrink](https://github.com/Drewsif/PiShrink) so the image is as small as possible for bulk writing. I'm running PiShrink via an Ubuntu virtual machine and a shared folder (VirtualBox, which is clunky but works.)

Boot from card, run `sudo raspi-config`, select 'Advanced options`, resize root file system, reboot.




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
