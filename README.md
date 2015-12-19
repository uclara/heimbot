# heimbot

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [1. Initial Manual Steps](#1-initial-manual-steps)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 1. Initial Manual Steps

1. Download [Minibian](https://minibianpi.wordpress.com/download/)

1. Write Image to SD card
  cf. [eLinux Max Os X How-To](http://elinux.org/RPi_Easy_SD_Card_Setup#Using_command_line_tools_.282.29)

    1. `diskutil list`

    1. `diskutil unmount /dev/disk2`

    1. `dd bs=4194304 if=/Users/lukas/Downloads/2015-11-12-jessie-minibian.img | pv | sudo dd of=/dev/disk2`

    1. `diskutil -unmount /dev/disk2s1`

    1. `diskutil -eject /dev/disk2`

1. Boot Image

1. Resize SD
  cf. [Minibian How-To](https://minibianpi.wordpress.com/how-to/resize-sd)

    1. `ssh root@minibian`

    1. Change Partition Size

        1. `fdisk /dev/mmcblk0`

        1. `p` -- Show parition table.

        1. `d, 2` -- Delete active Linux partition.

        1. `n p 2` -- Create new and larger parition at same location as
           the delete one.

            1. Use the same sector as first sector the delete parition
               used; cf. above after running `p`.

            2. Use highest sector as last sector.

        1. `w` -- Write parition table.

    1. Reboot and SSH

    1. `resize2fs /dev/mmcblk0p2`

