# short-pi-boot-script

#### Run your own startup scripts on Raspbian, without ever having touched the Linux partition.

This is a 100% script-based way to run commands at boot (even at first boot). It only uses the FAT32 */boot* partition on the SD card or the .img file. You don't need to change anything about the ext4 (Linux) partition.

This is useful if:

* you want to do unattended configuration at the first boot of the new SD card. For example, to configure networking, hostname, localisation and SSH.
* you broke something in the operating system, and just need to remove that *one* file to get it running again
* you write a lot of SD cards and don't want to spend time configuring each one
* you prefer to use the pristine Linux partition, rather than a modified version from someone you don't know
* you prefer a readable and editable shell script over a compiled program.

Doing it only with the */boot* partition is attractive because Macs and Windows PCs don't easily write to the Linux partition. Who wants to install yet another program on their computer?

## Usage
Each of these can be done on an SD card with Raspbian or on the downloaded *.img* disk image, which you can then flash to any number of SD cards. Most computers auto-mount the */boot* partition when you insert the SD card or double-click the *.img* file.

### Configure the files for your configuration
This requires the *unattended* file and a change to the file *cmdline.txt* on the boot partition.

1. Download the file [unattended](./unattended) from this project.
2. Open the file for editing. Look at section 2: modify as needed \*
3. Save the file and put it on the boot partition. Now open *cmdline.txt*, which is already on that partition \*\*
4. Remove the item with `init=` (if it is there) and put the following at the end of the line:
```
init=/bin/bash -c "mount -t proc proc /proc; mount -t sysfs sys /sys; mount /boot; source /boot/unattended"
```
5. Save the file
6. Download the file [wpa_supplicant.conf](./wpa_supplicant.conf) from this project
7. Open the file for editing and modify YOURWIFINETWORKNAME and WIFIPASSWORD to match your wifi network
8. Save the file and put it on the boot partition.
5. Eject the SD card or .img file, and you're done.

\* when your commands run, the PATH is `/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:.`; the working directory is `/`; the entire Linux partition is available; systemd isn't up yet so there are no services, the network is unavailable, and the system thinks it's January 1st, 1970.

\*\* the [cmdline.txt](./cmdline.txt) in this project is from Raspberry Pi OS with Desktop from 2020-12-02. If you happen to have that version, you can drop my cmdline.txt in.

## Warnings and recovery
You probably wouldn't do this sort of thing to an SD card that holds all your most important files, or that is urgently needed in a production situation. Remember that these scripts are all-powerful: they run as the administrator, so `rm -rf /` will *really* erase everything. To state the obvious, ***I don't accept any responsibility for what you do to your system using this***. Also, it's advisable to test it before using it when it matters.

If you have overwritten *cmdline.txt* on the boot partition with another version and the Pi doesn't boot from that card, copy the original cmdline.txt from the *.img* file. Or if you don't have that, correct the partition UUID in cmdline.txt:

* find the disk's UUID for partitions (distinct from 'disk UUID' and 'filesystem UUID'), a 32-bit integer saved in little-endian order at offset 0x1b8 from the start of the SD card or .img, with a command like

```
sudo dd if=/dev/disk2 bs=4 skip=110 count=1 | hexdump -e '1/4 "%02x"'
```
which emits the integer in hexadecimal notation, like `402e4a57` (example value, for 2017-04-10-raspbian-jessie).

* append `-02` for the root partition, and put the result in *cmdline.txt* in the form `root=PARTUUID=402e4a57-02`

## References
These files are largely based on the scripts from Jim Danner, you can find his gitlab repo [here](https://gitlab.com/JimDanner/pi-boot-script). My version here is only his basic version with a few additions. If you want to do more advanced configuration, his version is hugely more capable.
