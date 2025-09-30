# How to re-install libreboot on a already librebooted X60

Assume a stock Kali Linux.

:warning: **WARNING**

:warning: **If you decide to follow this guide, you do so AT YOUR OWN RISK! If you are not comfortable, do not proceed.**

Ok, with that out of the way:

Follow this closely, do not skip any commands! Also, please note dots and slashes, they are important.

Make sure system is up to date. For using sudo you may have to enter your password again.

```
sudo apt update && sudo apt -y full-upgrade
```

Open terminal and install git (Usually it is already there)

```
sudo apt install git
```

Configure git with some dummy values (or put real ones, IDC):

```
git config --global user.name "No Honeypot"
git config --global user.email screwthensa@example.com
```

Get the latest libreboot files from codeberg and move into the folder that was created:

```
git clone https://codeberg.org/libreboot/lbmk
cd lbmk
```

Set the number of threads to 4, to efficiently utilize your processor:

```
export XBMK_THREADS=4
```

Install all the dependencies needed for building. 

!Please note that this will possible take a long time, as a lot of stuff needs to be built!

As Kali is based on Debian, use:

```
sudo ./mk dependencies debian
```

Build the program that will do the actual flashing:

```
./mk -b flashprog
```

Find the model of your laptop in the list that is generated when you type:

```
./mk -b coreboot list
```

There are two images for the x60: ```x60``` and ```x60_16mb```.

You most likely have the regular ```x60```, to be sure, you can run:

```
sudo ./elf/flashprog/flashprog -p internal
```

Study the output of this command and look for any mention of chip size. 

Again, you most likely have the regular, unless the person who sold it to you also upgraded the BIOS EEPROM.

In short: if you see no mention of 16mb, use the exact commands here

:warning: **If flashprog shows 16mb, everywhere you read ```x60``` should be ```x60_16mb```**

:exclamation: **If flashprog talks about a /dev/mem related error, refer to troubleshooting below**

Now build the image for your laptop:

```
./mk -b coreboot x60
```

(One last time: if you have the 16mb, this command should read ```./mk -b coreboot x60_16mb```)

This can take a loooooong time, depending on your internet speed and computer speed (easily 30 mins+).

Longest task may be ```building GCC <version> for target```. Please be patient and do not turn off your laptop.

Final message should be something like ```Creating new libreboot image```

Once this has finished, you have a couple of images.

I think the nicest one is the one with a splash-screen, it will be called
```seagrub_x60_libgfxinit_corebootfb_usqwerty.rom```

Again, if you selected 16mb it will be called ```seagrub_x60_mb_libgfxinit_corebootfb_usqwerty.rom```

To make it easier, you can move into the folder that holds these roms and copy the flashprog program to this same folder

```
cd bin/x60
cp ../../elf/flashprog/flashprog .
```

And, now, if you're ready, you can write this ROM:

```
sudo ./flashprog -p internal -w seagrub_x60_libgfxinit_corebootfb_usqwerty.rom
```

Once all done, reboot your system and enjoy your new libreboot!

## troubleshooting /dev/mem errors
You are here because the flashprog command gave a /dev/mem related error.
Before you can proceed building libreboot, you need to fix this by:

Open the grub config and find a line containing ```GRUB_CMDLINE_LINUX=""```
Change this so it reads ```GRUB_CMDLINE_LINUX="iomem=relaxed"```

Use the text editor like so:
```
sudo nano /etc/default/grub
```

CTRL + O writes the changes, CTRL + X exits the text editor.
Now, you need to update the changes so they are persisted:

```
sudo update-grub
```

And reboot. Then, after logging in, opening the terminal and moving into lbmk directory, start again with the command ```sudo ./elf/flashprog/flashprog -p interal```, which should now be more informative
