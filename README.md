# CHIP-SDK
Everything needed to develop software for C.H.I.P.

While it is possible to install the SDK natively, currently the only supported way is to run it from a virtual machine.

*NOTE: the CHIP-SDK is updated regulary if you have an existing installation please have a look at the updating CHIP-SDK section below*

## System Requirements
You'll need VirtualBox and Vagrant.
For the virtual machine at least of free 1 GB RAM are necessary.
Up to 40 GB of disk space may be used.

## Installation

### VirtualBox
1. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. Install the [Oracle VM VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads) for the host - this is necessary to flash C.H.I.P from inside the virtual machine.
3. Operating system specific stuff:
   - If you are on Windows, you need to add the VirtualBox installation directory to your PATH.
   - In case of a Ubuntu host: add your user to the vboxusers group!

### Vagrant
You may need to install Vagrant. There are a couple options: 

* Download from [the Vagrant website](https://www.vagrantup.com/downloads.html)
* On OS X, you can use the [homebrew](http://brew.sh) package manager: 
    1. you'll need [Cask](http://caskroom.io), so if you don't have it: `brew install caskroom/cask/brew-cask`
    2. then `brew cask install vagrant`

### Git
Installation of Git depends on your operating system:
* On Windows, look at https://git-scm.com/download/win
* On a Debian based Linux you can do: `sudo apt-get install git`
* On Mac OS, the most convenient way is [homebrew](http://brew.sh): `brew install git`

### Clone the CHIP-SDK Git repository
Assuming you have `git` in your PATH, open up a terminal and type:

    git clone https://github.com/NextThingCo/CHIP-SDK

### Start up the virtual machine

In a shell on the host, change to the the CHIP-SDK directory and start up the virtual machine:

    cd CHIP-SDK
    vagrant up

A couple notes for the bleary eyed. If you get an error like:

    error: The guest machine entered an invalid state while waiting for it to boot.

This probably means your version of VirtualBox needs updating and/or needs the [Extension Pack](https://www.virtualbox.org/wiki/Downloads). Update as necessary and try `vagrant up` again.

If you get the error:

    error: Couldn't open file /Volumes/Satellite/gitbins/CHIP-SDK/base
    
that means you didn't `cd CHIP-SDK`.

### Login

In  the same shell on the host type the following:

    vagrant ssh

If everything went well you should see the following prompt:

    vagrant@vagrant-ubuntu-trusty-32:~$

Now you're ready to Flash a C.H.I.P. from your SDK!

## Prepare your C.H.I.P. for Flashing

First, prepare CHIP with a jumper wire between the **FEL** pin and **GND**. In other words, connect **Pin 7** and **Pin 39** on header **U14**.

Here's a diagram that labels the headers and pins assuming the components are facing you and the USB port is oriented up:

![Image of CHIP](https://nextthingco.zendesk.com/hc/en-us/article_attachments/203279037/CHIP_ALPHA_V02_Pinouts2.png "Image of jumpered CHIP")

And here's a photo with the jumper plugged in...

![Image of CHIP](https://nextthingco.zendesk.com/hc/en-us/article_attachments/203164668/DSCF2062.JPG "Image of jumpered CHIP")

It's worth noting that this jumper needs to be present only when you connect CHIP to power. If for some reason the wire becomes disconnected after you have powered CHIP, there is no problem or need to panic.

Now connect CHIP to your computer with a micro-USB→USB-B cable. The power LED will illuminate.

It's time to begin! Open a terminal on your computer, and let's start up a virtual machine.

    cd CHIP-SDK
    vagrant up
    vagrant ssh

If everything went well you should see the following prompt:

    vagrant@vagrant-ubuntu-trusty-32:~$
   
Now we're into C.H.I.P. and ready to flash an image.

## Help section

If you're familiar with the old version of the scripts, these options should sound pretty familiar to you, but some of the options have been moved around. So we added a help command to the flashing script:

    vagrant@vagrant-ubuntu-trusty-32:~$ ./chip-update-firmware.sh -h
    
    == Help ==

      -s  --  Server             [Debian + Headless]        
      -g  --  GUI                [Debian + XFCE]            
      -p  --  PocketCHIP         [CHIP on the go!]          
      -b  --  Buildroot          [Tiny, but powerful]       
      -f  --  Force clean        [re-download if applicable]
      -n  --  No limit           [enable greater power draw]
      -r  --  Reset              [reset device after flash] 
      -B  --  Branch             [eg. -B testing]           
      -N  --  Build#             [eg. -N 150]               
      -F  --  Format             [eg. -F Toshiba_4G_MLC]    
      -L  --  Local              [eg. -L ../img/buildroot/] 

## Flash a new C.H.I.P. with the NTC headless Debian image... 

If you want to flash C.H.I.P. with a custom image, scroll down the page...If you're cool with our current headless image, keep going!

With our virtual machine running, we'll start at our *trusty* prompt:

    vagrant@vagrant-ubuntu-trusty-32:~$

Now let's download the latest firmware (i.e. a Linux kernel, U-Boot and a root filesystem all built with buildroot) and flash it to CHIP.

    cd ~/CHIP-tools
    ./chip-update-firmware.sh
This may take a while - please be patient.

If everything went OK, you can now power up your CHIP again and connect by typing:

    screen /dev/ttyACM0 115200

You can login to CHIP as **chip** or if you feel more powerful as **root**. In both cases the password is **chip**. Now let's give it a quick hardware test...

    hwtest

If everything passed, your C.H.I.P. is ready to go!  Have fun!

## Flash a C.H.I.P. with Debian + GUI

Along with the instructions above, to flash our Debian image with the official C.H.I.P. GUI, you'll have to insert another option into the command. The `-g` flag will pull in Debian with a GUI-based version of the image.

    ./chip-update-firmware.sh -g


## Flash a C.H.I.P. with Buildroot

    ./chip-update-firmware.sh -b

You can login to CHIP as **chip** or if you feel more powerful as **root**. In both cases the password is **chip**.

## To Flash C.H.I.P. with your own custom buildroot image...

#### Start the build process

Logged in to the virtual machine again starting from our *trusty* prompt:

    vagrant@vagrant-ubuntu-trusty-32:~$

Lets' get in there and make something.

    cd ~/CHIP-buildroot
    make chip_defconfig
    make nconfig

From here, you can navigate the menu and select what you want to flash onto your C.H.I.P. and what you don't. Detailing custom buildroot images is outside the scope of this tutorial.  If you're curious, read Free Electrons wonderful [buildroot documentation](http://buildroot.uclibc.org/docs.html).

When you're finished with your selections, exit by hitting the F9 key, which will automatically save your custom buildroot to...  

    /home/vagrant/CHIP-buildroot/.config
   
**NOTE:** *You can save an alternate build by hitting the F6 key, but only the image save to the above path will flash to C.H.I.P.*

Now let's build your buildroot...

    make

This will take a while.  Depending on your computer, maybe an hour.  Maybe grab some coffee...


#### Flash your own buildroot image

Logged in to the virtual machine again starting from our *trusty* prompt:

    vagrant@vagrant-ubuntu-trusty-32:~$

type... 

    cd ~/CHIP-tools
    sudo ./chip-create-nand-images.sh ../CHIP-buildroot/output/build/uboot-nextthing_2016.01_next ../CHIP-buildroot/output/images/rootfs.tar my-new-images
    sudo chown -R vagrant:vagrant my-new-images
    ./chip-flash-nand-images.sh my-new-images

Ok, there's a lot of information there, but what's happening?

chip-create-nand-images.sh needs to know where to find the uboot binaries as well as the rootfs. From there, it formats these binaries and creates images appropriate for the different types of NAND memory on CHIP and CHIP Pro. That script needs to be run as root to maintain certain permissions in the rootfs. Afterwards though, we can chown the directory that your new images are in, and flash them with chip-flash-nand-images.sh.

## Shutdown

To log out of the virtual machine at anytime, type:

    exit
   
The virtual machine will still be running.  To shut it down, type:

    vagrant halt

## Troubleshooting'

In case you run into trouble because the kernel in the VM was updated and the shared vagrant folder can no longer be mounted, update the guest additions by typing the following in the CHIP-SDK directory on the host:

    vagrant plugin install vagrant-vbguest

Also look at [this blog post](http://kvz.io/blog/2013/01/16/vagrant-tip-keep-virtualbox-guest-additions-in-sync/)

In case you get the error

    `ERROR: You don't have permission to access Allwinner USB FEL device`
    
You'll need to run `./chip-update-firmware.sh` as `sudo`:

    sudo ./chip-update-firmware.sh

## Updating the CHIP-SDK

If you have an already existing installation and want to update it, follow these steps:

On you host operating system, pull the latest changes from our Git repository.
This can be done by changing into the CHIP-SDK directory and run git pull:

    cd ~/CHIP-SDK
    git pull

Make sure the virtual machine is shut down and re-provision:

    vagrant halt
    vagrant provision
    vagrant up

This should do the trick - ssh into the virtual machine:

    vagrant ssh

Once you see the trusty prompt, you can start developing!

    vagrant@vagrant-ubuntu-trusty-32:~$
