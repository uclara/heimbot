# heimbot

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [1. Initial Manual Steps](#1-initial-manual-steps)
- [2. Automatic Steps using Ansible](#2-automatic-steps-using-ansible)
  - [2.1. Bootstrap](#21-bootstrap)
  - [2.2 Set Up Services](#22-set-up-services)
- [3. Teaching-in of Devices](#3-teaching-in-of-devices)
  - [3.1 HomeMatic](#31-homematic)
- [4. CUL Flushing](#4-cul-flushing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 1. Initial Manual Steps

1. Download [Minibian](https://minibianpi.wordpress.com/download/)

1. Write Image to SD card -- cf. [eLinux Max Os X How-To](http://elinux.org/RPi_Easy_SD_Card_Setup#Using_command_line_tools_.282.29)

    1. `diskutil list`

    1. `diskutil unmount /dev/disk2`

    1. `dd bs=4194304 if=/Users/lukas/Downloads/2015-11-12-jessie-minibian.img | pv | sudo dd of=/dev/disk2`

    1. `diskutil -unmount /dev/disk2s1`

    1. `diskutil -eject /dev/disk2`

1. Boot Image

1. Resize SD -- cf. [Minibian How-To](https://minibianpi.wordpress.com/how-to/resize-sd)

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

1. Install Python

1. In case the CUL USB transmitter are not yet flushed please see CUL Flushing

## 2. Automatic Steps using Ansible


### 2.1. Bootstrap

1. Adapt `ansible.cfg` to your installation

1. Run bootstrapping

    `ansible-playbook -u root -k bootstrap.yml`

### 2.2 Set Up Services

`ansible-playbook site.yml`

The site playbook supports a few parameters which you can pass via `--extra-vars "parameter=value"`:

* `apt_update=[True|False]` -- `base`: Updates Apt cache and runs an upgrade.

* `clean_fhem_saved_state=[True|False]` -- `fhem`: Clears Fhem saved state.

* `clean_homebridge_identifier_cache=[True|False]` -- `homebridge`: Clears Homebridge identifier cache and restarts homebridge.


## 3. Teaching-in of Devices

### 3.1 HomeMatic

In order to control a new HomeMatic device, you need to _pair_ it with Fhem first -- cf. [IO-Device in den Pairing-Modus versetzen](http://www.fhemwiki.de/wiki/HomeMatic_Devices_pairen#IO-Device_in_den_Pairing-Modus_versetzen) for details.

1. `set CUL_868 hmPairForSec 60` -- Set Fhem into Teaching-in mode accepting pairing requests of HomeMatic device for 60 s.

1. `define autocreate autocreate` -- Optionally set Fhem to autocreate new devices.

1. Follow the instruction from the manual of your HomeMatic device to initiate a pairing attempt. For a switch like the `HM-ES-PMSw1-Pl`, hold the button for at least 5 s until the LED start to blink orange.

In case of a successful paring, you will see the similar lines in the Fhem log:

```
2015.12.23 17:12:00 2: CUL_HM Unknown device HM_3705C3 is now defined
2015.12.23 17:12:00 2: autocreate: define HM_3705C3 CUL_HM 3705C3
2015.12.23 17:12:00 2: autocreate: define FileLog_HM_3705C3 FileLog ./log/HM_3705C3-%Y-%m.log HM_3705C3
2015.12.23 17:12:00 3: Device HM_3705C3 added to ActionDetector with 000:10 time
2015.12.23 17:12:00 3: CUL_HM pair: HM_3705C3 powerMeter, model HM-ES-PMSw1-Pl serialNr
2015.12.23 17:12:04 3: CUL_HM set HM_3705C3 getConfig
2015.12.23 17:12:05 3: Device HM_3705C3 added to ActionDetector with 000:10 time
```


## 4. CUL Flushing

Usually, the CUL USB transmitter come without a firmware and need to be flashed first. You can use your Raspberry Pi to do this. Just set `FHEM.firware_flash_support` to `Yes` in `host_vars/heimbot.yml` and rerun the FHEM role. This will install the necessary tools. Then follow the next steps. For detailed instructions see [culfw](http://culfw.de) and [CUL am Raspberry Pi flashen](http://www.fhemwiki.de/wiki/CUL_am_Raspberry_Pi_flashen#Raspberry_Pi_mit_Raspbian).

1. Download Firmware -- cf. [culfw - Source (and compiled firmware)](http://culfw.de/culfw.html#Links)

1. Unpack the tar ball and change into the directory `Devices/CUL`

1. Insert a CUL USB transmitter into an USB port _while pressing the little button on the back_. This sets the transmitter into flush mode.

1. In the directory `Devices/CUL` run `make usbprogram`. This will show you which programm may be specified. For a CUL V3.4 use `usbprogram_v3`

1. Run the flush programm, e.g., `make usbprogram_v3`

    ```
       root@heimbot:/tmp/culfw/CUL_VER_161/Devices/CUL# make usbprogram_v3
       dfu-programmer atmega32u4 erase || true
       dfu-programmer atmega32u4 flash CUL_V3.hex
       Validating...
       23220 bytes used (80.98%)
       dfu-programmer atmega32u4 start
    ```

1. `dmesg -T` should show this

    ```
       [Tue Dec 22 10:25:46 2015] cdc_acm 1-1.2:1.0: ttyACM0: USB ACM device
       [Tue Dec 22 10:25:50 2015] usb 1-1.3: new full-speed USB device number 19 using dwc_otg
       [Tue Dec 22 10:25:51 2015] usb 1-1.3: New USB device found, idVendor=03eb, idProduct=204b
       [Tue Dec 22 10:25:51 2015] usb 1-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
       [Tue Dec 22 10:25:51 2015] usb 1-1.3: Product: CUL868
       [Tue Dec 22 10:25:51 2015] usb 1-1.3: Manufacturer: busware.de
       [Tue Dec 22 10:25:51 2015] cdc_acm 1-1.3:1.0: ttyACM1: USB ACM device
    ```

1. Repeat from step 3 for all CUL USB transmitters.


