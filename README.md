# Notes
[videos tutorials](https://www.youtube.com/watch?v=ltckiBV9FXg&list=PLRsiDZNEIs_T-AjQROqpwlzdLoJCkRab7&index=2)
[tutorial](https://medium.com/nerd-for-tech/build-your-own-linux-image-for-the-raspberry-pi-f61adb799652)

This sets up the environment, by default in a dir called ````build````

```
source src/poky/oe-init-build-env
```

Setup the build/local.conf 



For serial communication add the following to local.conf
```
# Tells meta-raspberrypi layer to add enable_uart=1
ENABLE_UART = "1"

# ensure Pi boots with console=serial0,115200
RPI_USE_SERIAL_CONSOLE = "1"
```

To build: 
```
bitbake core-image-base
```

### Flashing 
- bzip2 -d -f tmp/deploy/images/raspberrypi4/core-image-base-raspberrypi4.wic.bz2
- Run df -h command to see what devices are currently mounted.
- Insert your card into the card reader, and the card reader into your USB port.
- Make sure that the device is mounted; the easiest way to do that is to open it with your file manager.
- Run df -h again. The device that wasn't there last time is your SD card. The left column gives the device name of your SD card. It will be listed as something like /dev/mmcblk0p1 or /dev/sdd1. The last part (p1 or 1 respectively) is the partition number, but you want to write to the whole SD card, not just one partition, so you need to remove that part from the name (getting for example /dev/mmcblk0 or /dev/sdd) as the device for the whole SD card.
- Unmount the card with umount /dev/[sd_name] command, where [sd_name] is the name you got from the previous step.

***NOTE BE SUPER CAREFUL***

You can also shorten this process with 
```
bzcat tmp/deploy/images/raspberrypi4/image-name.bz2 | sudo dd of=/dev/disk
```

Or use balena etcher, which I have downloaded, cause it won't let you delete 
your system drive 

balena etcher also apparently decompresses a bz2 file before flashing it, so 
you don't even half to worry about it. 

### Adding additional meta layers

#### Error encountered
ERROR: User namespaces are not usable by BitBake, possibly due to AppArmor.
See https://discourse.ubuntu.com/t/ubuntu-24-04-lts-noble-numbat-release-notes/39890#unprivileged-user-namespace-restrictions for more information.

Seems to be to do with userspaces. More info found through link above. 

go to /etc/apparmod.d/

add the following in a new file names bitbake

```
    abi <abi/4.0>
    include <tunables/global>
    profile bitbake /**/bitbake/bin/bitbake flags=(unconfined) {
        userns,
    }
```

To load a profile into the kernel, use this command: 
```
sudo apparmor_parser -r /etc/apparmor.d/profile.name



``` 


#### License Error


ERROR: Nothing RPROVIDES 'linux-firmware-rpidistro-bcm43455' (but /home/timmy/proj/learning_linux/raspberry_pi/build/../src/poky/meta/recipes-core/packagegroups/packagegroup-base.bb RDEPENDS on or otherwise requires it)
linux-firmware-rpidistro RPROVIDES linux-firmware-rpidistro-bcm43455 but was skipped: Has a restricted license 'synaptics-killswitch' which is not listed in your LICENSE_FLAGS_ACCEPTED.

More info here: 
https://github.com/agherzan/meta-raspberrypi/issues/1111


The solution was to add to local.conf

```
LICENCSE_FLAGS_ACCEPTED='synaptics-killswitch'
```

#### Build crashes

On my computer the build continuously crashed
To fix that I lowered the number of threads being used. 
In local.conf: 
```
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
PARALLEL_MAKEINST = "-j 4"
```


#### Connecting to the Raspberry Pi  
#### USB 

Connect to raspberry pi 4 uart0 pins. 

ls /dev/ttyUSB* shows usb ports 
can check before and after connecting to the cable. 
default baud rate is 115200

It will ask for a login. The username is ```root``` by default, and it will not
ask for a password (by default).



#### SSH 
Add the following to build/local.conf 
```
EXTRA_IMAGE_FEATURES:append = " ssh-server-dropbear"
```
Yocto/Bitbake support overwrite style syntax with append, prepend, and remove. 
This allows you to add to the variable EXTRAM_IMAGE_FEATURES. 

We will add ssh-server-dropbear. This is a good lightweight ssh server 
perfect for embedded applications. 
We need to make sure there is a space at the beginning of the value, cause it 
is literally just being appended to EXTRA_IMAGE_FEATURES. 

- Connect raspberry pi via ethernet to router. 
- Connect over serial and log in 
- on Pi, run ````ip a````
    - below eth0, you should see inet: this will show the ip address
- on PC you can run ```ssh root@ip_address"



- Now you can navigate around. You won't have access to all of the commands/bins 
you would have on a normal linux system. However, Busybox does come installed 
by default. This gives you access to things like ls, cat, cd, grep and more. 
    - Busybox is a single binary that is symbolically linked to different 
    commands, which helps make it more lightweight. 
    - To use ls, cd into the root folder, with ````cd /````

## GPIO control 
By default, the raspberry pi build has GPIOs enabled. However, if you wanted 
to, I think that you could 

[sysfs interface](https://www.thegoodpenguin.co.uk/blog/stop-using-sys-class-gpio-its-deprecated/)
The sysfs interface is deprecated, but you can still use it. 
```
# Determine what the gpio number is. this will show the mappign between the 
# raspberry pi pinout. 
$ grep gpio /sys/kernel/debug/gpio

# enable the gpio pin to be controlled 
$ echo 21 > /sys/class/gpio/export  

# set the gpio to output 
$ echo out > /sys/class/gpio/gpio21/direction

# set the gpio valuethen use 
$ echo 1 > /sys/class/gpio/gpio21/value
```

To add ligpiod, you can use 

```
IMAGE_INSTALL:append = " libgpiod libgpiod-tools libgpiod-dev"
```
Which was found 
[here](https://stackoverflow.com/questions/63243453/how-to-install-libgpiod-binary-in-usr-bin-in-yocto-operating-system) 
since it was not immediately obvious


You can use the following commands: 
- gpioinfo: print info about GPIO lines
- gpiodetect: lists all GPIO chips detected
- gpioget: reads the current values of specified gpio lines
- gpiofind: locates the gpio chip name and line offset for a given line name 
- gpiomon: monitors gpio line for events 
- gpioset set values fo specified gpio lines, holding the lines until the 
process is killed or otherwise
[more info](https://libgpiod.readthedocs.io/en/latest/gpio_tools.html)

```
gpoiset GPIO4=1
```

