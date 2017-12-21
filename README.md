# Pi_config
Install log/protocol for NUSTEM's family of Pis


## Raspberry Pi image 2017-10-09

### Core burn:
Raspbian Stretch dated 9-9-2017

### Initial updates
Connect to Wifi Guest, 
    sudo apt update
    sudo apt upgrade
    sudo apt autoclean
    sudo apt autoremove

### Wallpaper
Transferred via nustem share. Saved to ~/Pictures and set. `fbset -s` handy way of finding display resolution.

### Misc
* Set time to Europe/London
* in `~/.bashrc`, turn on `ll`
* Enable Camera, SPI, I2C, SSH

Now restart

### Reconfigure menu bar launchers
* Remove Wolfram and Mathematica
* Add Thonny

(might wish to purge Wolfram, Mathematica and LibreOffice for space and update speed reasons, though as long as cards are 8Gb or greater I'm not convinced that's worthwhile. `apt upgrade` is only likely to be done as a re-imaging of all cards?)

### Pi-Top install
    cd Downloads
    git clone https://github.com/rricharz/pi-top-install
    cd pi-top-install
    chmod +x install*
    ./install-brightness
    ./install-poweroff

Insert into `/home/pi/.config/openbox/lxde-pi-rc.xml`:  
(**NB. Omit this stage - left as reference since this applies to PiTop lapptops*)
        <keybind key="0xC7">
          <action name="Execute">
            <command>brightness increase</command>
          </action>
        </keybind>
        <keybind key="0xC6">
          <action name="Execute">
            <command>brightness decrease</command>
          </action>
        </keybind>
...which was a bit pointless because those keys aren't on the keyboard for a Ceed. Durr. Oh, well. Onwards!

### Keyboard layout
Set to US, for most of our keyboards. Which was a mistake, since they're mostly UK.

### Python libraries, etc.

#### Pimoroni
`sudo apt install pimoroni`, then in Pimoroni installer choose:

- Explorer HAT
- Unicorn HAT (needed for Node Red stuff later, even though we only have Unicorn HAT HD)
- Unicorn HAT HD
- Scroll pHAT
- Scroll pHAT HD
- MicroDot pHAT

Awesome, this puts examples into a `~/Pimoroni` directory.

#### Sense HAT
    sudo apt install sense-hat
Already present with recent Raspbian

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

#### Paho-MQTT library
    pip install paho-mqtt

### Node Red updates
Referring to [Node-RED documentation](https://nodered.org/docs/hardware/raspberrypi), update Node via:

    bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)

Blimey. That's a bit on the brutal side.

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
    wget https://code.headmelted.com/installers/apt.sh
    chmod +x apt.sh
    sudo ./apt.sh

Surprisingly quick.

Launch, and install Python extension. Then install pylint as recommended.

### Tidying up
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

    sudo npm install -g tldr

Copy the `config` file from `https://github.com/tldr-pages/tldr-node-client` to `.tldrrc`, set theme to base16.

#### howdoi

    sudo apt install libxml2-dev python3-lxml python-lxml libxslt-dev python-dev
    pip install howdoi

Add to `.profile`:

    if [ -d "$HOME/.local/bin" ] ; then
        PATH="$HOME/.local/bin:$PATH"
    fi
    
### Password

Reset using Pi GUI tool (not `passwd` on the command line, which does sanity checking for rubbish passwords).

New password: **nustem**

### Things to include next time around

* Set up Pylint within Visual Studio Code. Oops.
* Arduino
* OpenCV? Ouchie.
* Browser defaults:
    * neverssl.com
    * microbit.org
    * Node Red (what is that link?)
    * projects.raspberrypi.org
    * any more?
* Pimoroni modules & example code
* I quite like Neofetch (which I think is an `apt install`?); added to `.bashrc`, splurges a nice logo and system info when a new terminal is opened. Which is a helpful reminder of which system you're interacting with.


### Deploy

Make `.iso` image (I think I did this using Disk Utility on the Mac), then process through [PiShrink](https://github.com/Drewsif/PiShrink) so the image is as small as possible for bulk writing. I'm pretty sure I ran PiShrink via an Ubuntu VM.

Ensure each written card has booted at least once, to expand the filesystem to fill the card.
