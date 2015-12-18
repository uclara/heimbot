# heimbot

## Manual Steps

1. Download [Minibian](https://minibianpi.wordpress.com/download/)

1. Write Image to SD card

    1. `diskutil list`

    1. `diskutil unmount /dev/disk2`

    1. `dd bs=4194304 if=/Users/lukas/Downloads/2015-11-12-jessie-minibian.img | pv | sudo dd of=/dev/disk2`

    1. `diskutil -unmount /dev/disk2s1`

    1. `diskutil -eject /dev/disk2`

1. Boot Image

## Automatic Steps using Ansible

1. Bootstrap

