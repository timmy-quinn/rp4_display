# Menuconfig
[tech a byte](https://www.youtube.com/watch?v=5BmEig-D2lA)


Menuconfig is a text-based menu driven user interface for configuring the 
Linux kernel. 

Selects which parts of the Linux kernel will be build, using a graphical 
interface. 
In linux source code tree, it can be started by running make menuconfig. In 
Yocto project it is a little different. 

```
bitbake -c menuconfig virtual/kernel
```

virtual/kernel is an abstraction. There may be multiple kernel recipes. 
Bitbake picks the default based on machine and configurations. 
You can check which one is used: 

```
bitbake -e virtual/kernel | grep ^PREFERRED_PROVIDER_virtual/kernel=
```

Or the recipe BitBake will build. 
```
bitbake -e virtual/kernel | grep ^PN=
```

## Make permanent 

1. Create kernel .bbappend file 
2. configure kernel via menuconfig
3. save kernel config 
4. copy config file to recipe folder 
5. build image 


### Kernel bbappend file 
create 
```
recipes-kernel/
    linux/
        linux-yocto/ 
        linux-yocto_5.15.bbappned 
```

### Configure kernel  
```
bitbake -c menuconfig virtual/kernel
```

We've configured kernel. If you clean state cache, it will be removed

### Save 

```
bitbake -c savedefconfig virtual/kernel
```
Saves it temp/work/.../linux-yocto/ 

### Copy the file to recipe folder 
Go to your linux-yocto and copy the newly built defconfig file to the 
linux-yocto directory you created earlier. 

add your defconfig to linux-yocto_5.15.bbappend

```
SRC_URIL += "file://defconfig

KCONFIG_MODE = "alldefconfig" 

```

````alldefconfig```` basically means that you add the configuration in 
defconfig to the existing configuration in poky.

### 
Build the kernel 

## Kernel config with config fragments (diffconfig)
1. Create Kernel .bbappend file 
2. Configure kernel via menuconfig
3. Create kernel config fragment 
4. Copy config fragment to recipe folder
5. edit kernel .bbappend file 

This prevents configurations from being deleted whenever a cleanstate is done. 

### create kernel .bbappend file 
```
recipes-kernel/
    linux/
        linux-yocto/ 
            old_defconfig
        linux-yocto_5.15.bbappend 
```

### configure via menuconfig
```
bitbake -c menuconfig virtual/kernel
```


### Create kernel config fragment 
```
bitbake -c diffconfig virtual/kernel
```

###  copy fragment to recipe folder 
fragment should be able to be found in 
````tmp/work/.../linux-yocto/.../fragment.cfg````

```
cp /path/to/fragment.cfg path/to/recipes-kernel/linux/linux-yocto/new-fragment-name.cfg
```
You should give the fragment a meaninful name. For example gpio.sysfs.cfg


### edit kernel .bbapppend file 

```linux-yocto_5.15.bbappend
SRC_URI += "file://config-name.cfg
```


### clean and build
if you do the following 
```
bitbake -c cleanstate virtual/kernel
bitbake -c menuconfig virtual/kernel
```
You will see that the menuconfig retains whatever settings you made when you 
first edited the menuconfig. 



