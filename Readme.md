# RaspiBackup.sh
Script to backup a Raspberry Pi OS installation to an .img file. 
The resulting file can be installed to an SD card or a USB device, as per following guide: 
https://www.raspberrypi.org/documentation/installation/installing-images/README.md  


## Author / Origin:

This script is inpired by user `jinx`.


## Usage

* RaspiBackup.sh _COMMAND_ _OPTION_ sdimage

E.g.:
* RaspiBackup.sh start [-cslzdf] [-L logfile] sdimage
* RaspiBackup.sh mount [-c] sdimage [mountdir]
* RaspiBackup.sh umount sdimage [mountdir]
* RaspiBackup.sh gzip [-df] sdimage
* RaspiBackup.sh showdf sdimage
### Commands:

* *start* - starts complete backup of RPi's SD Card to 'sdimage'
* *mount* - mounts the 'sdimage' to 'mountdir' (default: /mnt/'sdimage'/)
* *umount* - unmounts the 'sdimage' from 'mountdir'
* *gzip* - compresses the 'sdimage' to 'sdimage'.gz
* *showdf* - Shows allocation of image
### Options:

* -c creates the SD Image if it does not exist
* -l writes rsync log to 'sdimage'-YYYYmmddHHMMSS.log
* -z compresses the SD Image (after backup) to 'sdimage'.gz
* -d deletes the SD Image after successful compression
* -f forces overwrite of 'sdimage'.gz if it exists
* -L logfile writes rsync log to 'logfile'

### Examples:

Start backup to `backup.img`, creating it if it does not exist:
```
RaspiBackup.sh start -c /path/to/backup.img
```


Refresh (incremental backup) of `backup.img`. You can only refresh a noncompressed image. 
```
RaspiBackup.sh start /path/to/backup.img
```


Mount the RPi's SD Image in `/mnt/backup.img`:
```
RaspiBackup.sh mount /path/to/backup.img /mnt/backup.img
```

Unmount the SD Image from default mountdir (`/mnt/backup.img/`):
```
RaspiBackup.sh umount /path/to/backup.img
```

show allocation of SD Image:
```
RaspiBackup.sh showdf /path/to/backup.img
```


### Pros of this version

The current script doesn't require you to specify a source device. This is useful when the system is on a different device from the SD card, so either all on a pendrive/external disk, or with /boot on the SD card, and / on the USB device.
The code will extract PARTUUID's from /etc/fstab, and use those to correctly identify the underlying devices (so keep your /etc/fstab current).
Another useful benefit for those running a system from a USB drive, is that the size of the image will be calculated as the real size of the root partition (not the size of the whole device... like 30GB for a pendrive where only 4GB worth of root partition resides), plus 256MB which is the recommended /boot size by the RPi Foundation.

### Caveat:

This script takes a backup while the source partitions are mounted and in use. The resulting imagefile will be inconsistent!

To minimize inconsistencies, you should terminate as many services as possible before starting the backup. An example is provided as daily.sh.

The image produced will always be immediately functional after restore (no need to correct /boot/cmdline.txt not /etc/fstab) if you plan to use a single device for /boot and / (SD card, or USB boot), otherwise if you need /boot to reside on the SD card while / is on USB drive, you will have to do as following:

1. Flash the image on the planned device for root partition (which will also contain a 256MB unused boot partition)
2. Format an SD with FAT32 to be your boot device, and copy the content of the FAT32 boot partition on the USB drive, to this SD
3. Boot the system normally
4. Run `sudo blkid` to check the PARTUUID of the FAT32 partition on the SD card, and copy its value to clipboard
5. Run `sudo nano /etc/fstab` and edit the PARTUUID in the /boot row, changing it to the PARTUUID you just copied from the output of the previous command
6. Reboot your system, and run `df` to verify that the row for the /boot mount actually points to `/dev/mmcblk0p1` (meaning that firmware updates will be installed to the correct partition)
