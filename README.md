#Importante se no final quando executar o sudo svxlink der erro nos modulos como nao encontrado vide exemplo abaixo:

*** ERROR: Failed to load module ModuleEchoLink into logic SimplexLogic: /usr/bin/svxlink/ModuleEchoLink.so: cannot open shared object file: Not a directory

Verificar o caminho no modulo /etc/svxlink/svxlink.conf se esta o caminho abaixo ou /usr/bin/svxlink altere pro abaixo:

/usr/lib/arm-linux-gnueabihf/svxlink/

#SVXLINK (Echolink Linux) 

Instalando Echolink no Linux (SVXLINK)

sudo apt-get install g++ make libsigc++-1.2-dev libgsm1-dev libpopt-dev tcl8.5-dev libgcrypt-dev libspeex-dev libasound2-dev alsa-utils libqt4-dev git-core sigc++

Se der erro no comando abaixo como add-apt not found instale o seguinte:

sudo apt-get install software-properties-common python-software-properties

--- Essa parte e para CPU com LInux baseado em Debian/Ubuntu para Raspberry seguir passos abaixo do Raspberry

sudo add-apt-repository ppa:felix.lechner/hamradio

sudo apt-get update

sudo apt-get install svxlink-server

Echolink Client Se precisar

sudo apt-get install qtel

Arquivos para Editar

sudo nano /etc/svxlink/svxlink.conf 

sudo nano /etc/svxlink/svxlink.d/ModuleEchoLink.conf 

Diretorios de SOM

/usr/share/svxlink/sounds/en_US/

--- Essa parte e para raspberry

wget https://github.com/sm0svx/svxlink/archive/master.tar.gz

tar -xzvf master.tar.gz

cd svxlink-master

cd src

mkdir build

cd build

cmake -DCMAKE_INSTALL_PREFIX=/usr -DSYSCONF_INSTALL_DIR=/etc \
      -DLOCAL_STATE_DIR=/var ..

make

make doc

make install

ldconfig

---------------------OFICIAL DOCUMENT

Raspberry Pi

Tips for installing SvxLink on the Raspberry Pi.
Compilation

To make SvxLink run with lower CPU-load on the Raspberry Pi, set RELEASE_FLAGS in makefile.cfg to:

RELEASE_CFLAGS=-g -O2 -mfloat-abi=softfp -mfpu=vfp -mcpu=native

K9DWR simplex GPIO setup steps

This is an outline of the configuration I used to set up a Simplex node that I’m using as a remote link to a repeater. There are a few items here that are pretty specific to my setup, but the majority can be pretty easily translated to any Raspberry Pi. I’m using a RPi B (rev 2), but this has also been tested on a RPi 2.

The specific hardware being used is:
Raspberry Pi B (rev 2)
Easy-Digi sound interface
USB Soundcard (chipset CM108)
Motorola PM400 mobile radio
Powerswitch Tail II
Basic raspbian setup

    write image to SD card

    enable ssh

        connect to serial port https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/overview

        use raspi-config to turn on ssh

    basic RPi setup

        raspi-config

            expand filesystem

            set locale, timezone and keyboard

            GPU memory split = 16

            set hostname

    apt-get update && apt-get upgrade

    install vim and screen

    max_usb_current=1 in /boot/config.txt

    Set up tmpfs partitions - edit /etc/fstab

        edit /etc/fstab

        #temp filesystems for all the logging and stuff to keep it out of the SD card
        tmpfs    /tmp    tmpfs    defaults,noatime,nosuid,size=100m    0 0
        tmpfs    /var/tmp    tmpfs    defaults,noatime,nosuid,size=30m    0 0
        tmpfs    /var/log    tmpfs    defaults,noatime,nosuid,mode=0755,size=100m    0 0
        tmpfs    /var/spool/svxlink    tmpfs    defaults,noatime,nosuid,size=100m    0 0

        rm files from mount locations

        disable swap

            sudo swapoff --all

            sudo apt-get remove dphys-swapfile

        reboot

    run rpi-update for firmware updates - This is crucial for USB soundcard support

        noticed a funky problem with rpi-update. It will try to update the tool when it runs. This could be a problem, so tell it not to update itself when it runs:

        sudo UPDATE_SELF=0 rpi-update

        precheck (on my Pi B rev 2):

        /opt/vc/bin/vcgencmd version
        Feb 14 2015 22:17:16
        Copyright (c) 2012 Broadcom
        version 7789db485409720b0e523a3d6b86b12ed56fd152 (clean) (release)

        post check:

        pi@echolink1 ~ $ /opt/vc/bin/vcgencmd version
        Apr  6 2015 13:26:19
        Copyright (c) 2012 Broadcom
        version 52e98d667b8b6ef27a2df8d817807701057bb30d (clean) (release)

READ ALL THE INSTALL DOCS FIRST: Installation Instructions
svxlink software setup

    add svxlink user:
    The svxlink user needs to have access to gpio and audio, and I also created mine so that it was not possible to log in (as a security item). You can still su - svxlink once you have logged into the Pi if you need to do things as the svxlink user.

    sudo useradd -c "Echolink user" -G gpio,audio -d /home/svxlink -m -s /sbin/nologin svxlink

    install needed libs (for debian-specific example see Debian)

    sudo apt-get install cmake libsigc++-2.0-dev libasound2-dev libpopt-dev libgcrypt11-dev tk-dev libgsm1-dev libspeex-dev libopus-dev groff

    grab source from GitHub

        download svxlink-master.zip from main Git Repo page

            There are actually a few options for this:

                Latest master from Git tree (what I used): https://github.com/sm0svx/svxlink

                Latest stable (but old): https://github.com/sm0svx/svxlink/releases/tag/15.11

                Latest zip/tgz from the main page: http://www.svxlink.org

        expand in /tmp

        follow INSTALL docs in extracted zip file with the following cmake options:

        cmake -DCMAKE_INSTALL_PREFIX=/usr -DSYSCONF_INSTALL_DIR=/etc -DLOCAL_STATE_DIR=/var -DUSE_OSS=NO -DUSE_QT=NO ..

        make && make doc

        as root: ldconfig && make install

    under /var/spool, tar up the svxlink directory since this is in memory and the contents will be removed on a reboot

    cd /var/spool
    tar zcvf svxlink.tgz svxlink

    we will add stuff to rc.local to put it back at boot time

    configure /etc/svxlink/svxlink.conf

    [GLOBAL] Section
    LOGICS=SimplexLogic

    [SimplexLogic] Section
    CALLSIGN

    [Rx1] Section
    AUDIO_DEV=alsa:plughw:1

    [Tx1] Section
    AUDIO_DEV=alsa:plughw:1
    PTT_TYPE=GPIO
    PTT_PORT=gpio17

    configure /etc/svxlink/svxlink.d/ModuleEchoLink.conf

    CALLSIGN
    PASSWORD

    set up /etc/rc.local to set gpio pin for PTT

    # Echolink GPIO settings
    # PTT GPIO17 (pin 11) -- I like using pin 11
    # because pin 9 is GND (and next to 11)
    echo 17 > /sys/class/gpio/export
    echo 'out' > /sys/class/gpio/gpio17/direction
    echo 0 > /sys/class/gpio/gpio17/value

    #replace /var/spool/svxlink since we put it in tmpfs.
    cd /var/spool
    tar zxvf svxlink.tgz

    install language pack

        You want the 16k sounds - https://github.com/sm0svx/svxlink-sounds-en_US-heather/releases

Hardware setup

Pinouts

Pi Power Button/LED

LED Plus (+)  -> P1-8 (GPIO14) - Red
LED Minus (-) -> P1-9 (GND)    - Black
Power C1      -> P1-6 (GND)    - Black
Power NO1     -> P1-5 (GPIO3)  - Blue
Power NC1     -> Not Used

Audio connections (polarity doesn’t matter at the Easy Digi end):

Mic
  Tip         -> Easy Digi "Audio from PC" - green
  Bottom Ring -> Easy Digi "Audio from PC" - white
  Middle Ring -> Not Used
Headphone
  Tip         -> Easy Digi "Audio to PC" - blue
  Bottom Ring -> Easy Digi "Audio to PC" - red
  Middle Ring -> Not Used

Line connections:

P1-11 (GPIO17) -> Easy Digi RTS - Yellow
P1-13 (GPIO27) -> CSQ Enable    - Red (see note #1)
P1-14 (GND)    -> Easy Digi GND - Black
P1-22 (GPIO25) -> Powerswitch Tail II + - Orange
P1-20 (GND)    -> Powerswitch Tail II - - Black

Radio Pinout:

Pin2 - Ext Mic Audio - White      -> Easy Digi Mic In
Pin3 - Ext Mic PTT   - Brown/Blue -> Easy Digi PTT Hi
Pin7 - Ground        - Black      -> Easy Digi GND
Pin11 - RX Audio out - Green      -> Easy Digi Rx Audio
Pin14 - CSQ Enable   - Red        -> Pi P1-13 (GPIO27) (see note #1)

NOTE #1: This is a passthru on Red from the radio directly to the
    RPi through the Easy Digi.  It's not actually connected to anything
    inside the Easy Digi.  It's purely for cleanliness of the wires.

Software setup

The power button was inspired by the need to gracefully shut down the Pi to avoid corrupting the SD card, and have a visual indicator to know if it was turned on or not. To that end, I borrowed from the Adafruit retrocade project; specifically an adaptation I found on philtopia.com: http://www.philtopia.com/?p=1823

I have the retrocade code (renamed to button-monitor) loading in rc.local and monitoring GPIO3:

# Enable the button monitor for graceful shutdown
/root/button-monitor &

It allows me to do a long press to trigger a shutdown (actually calling a bash script called system-down.sh) that flashes the LED and then calls a shutdown. A trick with GPIO3 is if it’s jumpered to Ground when the system is off, it will start the Pi, so the one button can be used for both startup and shutdown. The code I’m using in the shell script is:

#!/bin/bash
#
# system-down.sh
#
# Shell script that is called by button-monitor to gracefully
# shut down the pi.  This way, we can make changes to the
# env, look for error codes, etc and fix things without
# recompiling all the time.

# Change LED to be an output
echo 14 > /sys/class/gpio/export
echo 'out' > /sys/class/gpio/gpio14/direction

for blink in {1..7}
do
    echo 0 > /sys/class/gpio/gpio14/value
    sleep .2
    echo 1 > /sys/class/gpio/gpio14/value
    sleep .2
done

shutdown -h now

A few notes about finding hardware info and testing:
First off, figuring out the actual HW. If you want to list the output devices, use aplay; if you want the input devices, use arecord:

root@echolink1:/tmp# aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 0: ALSA [bcm2835 ALSA], device 1: bcm2835 ALSA [bcm2835 IEC958/HDMI]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Device [C-Media USB Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0

root@echolink1:/tmp# arecord -l
**** List of CAPTURE Hardware Devices ****
card 1: Device [C-Media USB Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0

The important part is the card # and the device #. In this case, 1 and 0. That’s where the -D value for arecord comes from (eg, -D hw:1,0). Since it’s the default capture, just hw:1 is also sufficient. A different sub-device would be necessary only if the sound card had more than one mic input. For another example, card 0 in my config has two output devices and NO capture devices. hw:0,0 is the 3.5mm jack and hw:0,1 is the HDMI audio output on the Raspberry Pi. Card 1 is the USB soundcard that we are using for in/out.

Using arecord, I was finally able to figure out an incantation to test that audio is making it into the device. All three of these will produce the same output:

arecord -vv -fdat -c1 -D hw:1,0 /tmp/test.wav
arecord -vv -fdat -c1 -D hw:1 /tmp/test.wav
arecord -vv -f S16_LE -r48000 -D hw:1 /tmp/test.wav

-vv - make the display very verbose and include a VU meter at the bottom of the screen
-fdat - sets the format. shorthad for [-f S16_LE -c2 -r48000] (16 bit little endian, 48000, stereo)
-c1 - sets number of channels to 1 (mono instead of stereo). Overriding the -fdat+
-D hw:1 or hw:1,0 - uses the default capture device on card 1
-r48000 - bitrate of 48,000Hz

Any of these lines will result in the following recording settings:

Recording WAVE '/tmp/test.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
Hardware PCM card 1 'C-Media USB Audio Device' device 0 subdevice 0
Its setup is:
  stream       : CAPTURE
  access       : RW_INTERLEAVED
  format       : S16_LE
  subformat    : STD
  channels     : 1
  rate         : 48000
  exact rate   : 48000 (48000/1)
  msbits       : 16
  buffer_size  : 24000
  period_size  : 6000
  period_time  : 125000
  tstamp_mode  : NONE
  period_step  : 1
  avail_min    : 6000
  period_event : 0
  start_threshold  : 1
  stop_threshold   : 24000
  silence_threshold: 0
  silence_size : 0
  boundary     : 1572864000
  appl_ptr     : 0
  hw_ptr       : 0

svxlink can’t be running while you are testing because the sound device will be busy and arecord will fail: "arecord: main:682: audio open error: Device or resource busy"
Problems (and fixes) along the way.

This is a collection of issues that I hit during the setup process. I tried to document the specific issue and also the fix to the issue.

**Compile Problems:
NOTE: Most of these are not an issue if you READ THE INSTALL DOCS COMPLETELY FIRST.

it puts everything in /usr/local - if this isn’t what you want (probably not), then change cmake:

cmake -DCMAKE_INSTALL_PREFIX=/usr -DSYSCONF_INSTALL_DIR=/etc -DLOCAL_STATE_DIR=/var -DUSE_QT=NO ..

missing libs - svxlink: error while loading shared libraries: libasynccpp.so.1.3:

cannot open shared object file: No such file or directory

checking with ldd:

root@echolink1:/usr/local/bin# ldd ./svxlink
/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so (0xb6ef4000)
libsigc-2.0.so.0 => /usr/lib/arm-linux-gnueabihf/libsigc-2.0.so.0 (0xb6ed8000)
libpopt.so.0 => /lib/arm-linux-gnueabihf/libpopt.so.0 (0xb6ec5000)
libgsm.so.1 => /usr/lib/arm-linux-gnueabihf/libgsm.so.1 (0xb6eb3000)
libtcl8.5.so.0 => /usr/lib/libtcl8.5.so.0 (0xb6dbc000)
libgcrypt.so.11 => /lib/arm-linux-gnueabihf/libgcrypt.so.11 (0xb6d3c000)
libdl.so.2 => /lib/arm-linux-gnueabihf/libdl.so.2 (0xb6d31000)
libasynccpp.so.1.3 => not found
libasyncaudio.so.1.3 => not found
libasynccore.so.1.3 => not found
librt.so.1 => /lib/arm-linux-gnueabihf/librt.so.1 (0xb6d21000)
libpthread.so.0 => /lib/arm-linux-gnueabihf/libpthread.so.0 (0xb6d02000)
libspeex.so.1 => /usr/lib/arm-linux-gnueabihf/libspeex.so.1 (0xb6ce4000)
libasound.so.2 => /usr/lib/arm-linux-gnueabihf/libasound.so.2 (0xb6c20000)
libstdc++.so.6 => /usr/lib/arm-linux-gnueabihf/libstdc++.so.6 (0xb6b4e000)
libm.so.6 => /lib/arm-linux-gnueabihf/libm.so.6 (0xb6adc000)
libgcc_s.so.1 => /lib/arm-linux-gnueabihf/libgcc_s.so.1 (0xb6ab4000)
libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6983000)
/lib/ld-linux-armhf.so.3 (0xb6f01000)
libgpg-error.so.0 => /lib/arm-linux-gnueabihf/libgpg-error.so.0 (0xb6978000)

a simple ldconfig after the compile fixed this one

missing QT - You have two options for this one, install qt4-dev-tools if you need it, or add -DUSE_QT=NO to your cmake flags if you don’t. If you aren’t going to use the GUI client Qtel, you don’t need it. Removing it will probably save some compile time, too.

make doc fails - install groff first. Might need Doxygen too, but it compiles without it (and Doxygen is 1GB of stuff you don’t need on the SD card, anyway)

TK errors - cmake generates the following. It’s unclear if it’s a problem (it compiles fine). This shouldn’t happen if you install the tcl-dev and tk-dev packages
 — Could NOT find TCLTK (missing: TK_LIBRARY TK_INCLUDE_PATH)
 — Could NOT find TK (missing: TK_LIBRARY TK_INCLUDE_PATH)

Audio HISS in repeater RF audio - This one is really making me nuts. For whatever reason, the PM400 has a very noticeable hiss at around 5kHz. I’m attempting to install the alsaequal equalizer plugin to see if I can pre-process the signal coming in through the alsa device. I have no idea if I can make it work. Steps to configure - apt-get install libasound2-plugin-equal edit .asoundrc file

+

ctl.equal {
type equal;
}
pcm.plugequal {
type equal;
slave.pcm "plughw:1,0";
}
pcm.equal {
type plug;
slave.pcm plugequal;
}

+ This appears to work backwards: pcm.equal → pcm.plugequal → ctl.equal I still need to understand exactly how the plugin arch works. Getting closer. figured out that the syntax for referencing the plugin is alsa:equal, not alsa:plughw:equal but using it for Rx1 results in a hung system after the first time audio comes in. Another angle is to use a hardware Eq. I’ll probably try running it through my old stero Eq as a test.

UPDATE: Still not fixed, but usable. At this point, just accepting that there’s some noise and living with it.

Audio beep during TX - This appears to be the same issue outlined here: http://svxlink.996268.n3.nabble.com/Beeps-in-received-audio-td3088.html "It’s a new feature which was newly added. A beep is added when a remote EchoLink transmission stops. You probably are experiencing dropped UDP packets from the remote station. If SvxLink does not receive audio for 200ms it will interpret this as if the remote transmission has ended." The fix for this is to do a local overlay replacement for the is_receiving proc in the EchoLink Tcl module in /usr/share/svxlink/events.d

    variable last_carrier_loss 0;

    #
    # Executed when a transmission from an EchoLink station is starting
    # or stopping
    #
    proc is_receiving {rx} {
       variable last_carrier_loss;

       if {$rx == 0} {
         set now [clock clicks -milliseconds];
         if { ($now - $last_carrier_loss) > 2500 } {
           playSilence 100;
           playTone 1500 100 100;
           playTone 1000 100 100;
           playSilence 100;
         }

         set last_carrier_loss $now;
       }
    }

audio coming into the Pi from the radio, but not going out the internet - No idea what’s up with this issue yet. doing a tcpdump shows the software isn’t even trying to send. Everything else works (inet → repeater, inet → inet).

FIXED - Use SimplexLogic instead of RepeaterLogic. The basic issue is RepeaterLogic assumes a hard-linked repeater connection rather than a remote-link connection as described on the Echolink site. As a result, svxlink wants to retransmit back out to the repeater, rather than send the received audio out the internet.

stopping TX on open squelch - I don’t want svxlink keying up over top of someone on the repeater and doubling. Need to see if there’s an option to disable this. I know something exists for one type of usage, but I don’t know if that’s globally available.

FIXED - since we are switching to SimplexLogic instead of RepeaterLogic, the default is to do this for us already. There’s going to be doubling, but nothing worse than what you already deal with. What’s important is that it doesn’t step on someone that’s already TXing on the repeater. The variable that addresses this is MUTE_TX_ON_RX=1 (which is enabled by default).

System goes into endless TX when svxlink is shut down - We need to be careful to turn off the link radio when the software is shut down. The voltage on RTS triggers an endless PTT otherwise. We might need to script something to monitor the status (ie, if no svxlink, set the RTS pin).

FIXED - Use GPIO instead. When the software shuts down, it doesn’t reset the pins the same way the serial connection does, so we don’t have the same risk. It’s still a good idea to script it so it disables the port explicitly and also that we set up a watchdog so that if the pin is high and svxlink is not running it shuts it down as well. That will help protect against crashes of the software while it’s TXing.

Incoming audio is clipped. - The squelch is too sensitive and cuts off RF audio whenever someone pauses to breathe. Need to find a setting to make it more forgiving.

FIXED - Since we are using SQL_DET=VOX for RF audio coming in, there’s no reason to be picky about the VOX_THRESHOLD. As a result, I just turned the threshold down (currently VOX_THRESHOLD=200) so that it fires when anything comes in (including what’s basically dead air). In other words, if there’s any RF audio at all, we want to relay it.

REALLY FIXED - I switched to using SQL_DET=GPIO and am reading the CSQ pin from the radio to tell when a signal is coming in. This is a lot more reliable than using VOX. It does mean, however, that the link will trigger on the squelch tail now. I have an issue opened to get SQL_START_DELAY to apply to CSQ over GPIO to fix this (issue137).

spurious connection issues - It’s common for incoming connections to have issues and throw messages about mismatched IP addresses. This is a caching issue with how mobile clients change their IP addresses because most of them use relays and don’t connect directly. As a result, they often end up connecting from a new address every time they start up. A possible fix may be to script to restart if no connections exist to flush the cached data.

FIXED (in testing) - git issue93 covers this with a software fix that isn’t in the master branch yet. I’ve been running on the issue93 branch which successfully deals with the problem. This should work once it’s put in the master branch.

REALLY FIXED - The code change has been put into the master branch and has worked great. It sometimes takes a few extra seconds to connect, but it does work.

audio connection failing (locking up) - This looks like it is probably the issue of not enough power to the USB sound card. I’ve switched to using the Pi2 and GPIO for PTT instead of the Serial port. I’ve also added the following to the /boot/config.txt, which is supposed to supply more power to the USB ports:
max_usb_current=1

FIXED (ish) Update: Power doesn’t seem to be the problem. I’ve run on the Pi2, used an external powered USB hub and that hasn’t helped. What did help, though was disabling the QSORecorder. It’s not clear yet why this makes a difference, but I’ve had no issues for several days after turning off recording. For now, it’s just going to have to be a voodoo thing and keep it disabled.

REALLY FIXED: This turned out to be an issue with running out of disk space for storing the wav files. Two things fixed this: 1. more closely monitoring disk and not letting it fill up. 2. code updates in the source that more gracefully handle if the disk does fill up.

Using GPIO has its own challenges: You need to enable the port, set the direction of flow and match it on the svxlink config side (the useradd -G gpio takes care of this): add svxlink user to the gpio group

/etc/rc.local entries:

        # Echolink GPIO settings
        # PTT GPIO17 (pin 11) -- I like using pin 11 because pin 9 is GND
        echo 17 > /sys/class/gpio/export
        echo 'out' > /sys/class/gpio/gpio17/direction
        echo 0 > /sys/class/gpio/gpio17/value

        # SQL_DET GPIO27 (pin 13)
        echo 27 > /sys/class/gpio/export
        echo 'in' > /sys/class/gpio/gpio27/direction

        #replace /var/spool/svxlink
        cd /var/spool
        tar zxvf svxlink.tgz

        # enable Powerswitch Tail II for radio power control
        # The PST II is a Hardware AC interrupt device that can be used to
        # remove power to the radio.  I put this in place as a failsafe
        # way to kill runaway transmissions.
        # GPIO25 (pin P1-22)
        echo 25 > /sys/class/gpio/export
        echo 'out' > /sys/class/gpio/gpio25/direction
        echo 1 > /sys/class/gpio/gpio25/value

        # Enable the button monitor for graceful shutdown
        /root/button-monitor &

        # Start svxlink
        /root/el.sh

/etc/svxlink/svxlink.conf
    PTT_TYPE=GPIO
    PTT_PIN=gpio17

I actually prefer using GPIO because you have better control over the PTT (see System goes into endless TX when svxlink is shut down)

The default svxlink.conf has an incorrect variable that the system complains about on startup:

*** WARNING: The DTMF_TONE_AMP configuration variable set in transmitter
Tx1 is deprecated. Use DTMF_DIGIT_PWR instead. To get the same output level
using the new configuration variable, add 3dB to the old value.
This has been reported to the developer and updated in the build info

FIXED: The current master branch has this updated

