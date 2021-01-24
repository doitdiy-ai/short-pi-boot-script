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
This requires the *cmdline.txt*, *unattended*, *pipassword* and *wpa_supplicant.conf* files, which will need to be modified as follows.

1. Download the files as [zip file](https://github.com/doitdiy-ai/short-pi-boot-script/archive/master.zip) from this project and unzip
2. Insert a freshly imaged SD card in your SD card reader and open it in a file browser
3. Open *cmdline.txt* that is on the SD card, and take note of the PARTUUID section, it'll look like this:
```
PARTUUID=904a3764-02
```
4. Open a text editor like Notepad on your PC and copy the string `904a3764-02` (this string may vary based on the OS image that was put on the SD card)
5. Close the *cmdline.txt*. Next open the downloaded *cmdline.txt* (should be in the `short-pi-boot-script` folder)
6. Replace the string right after `PARTUUID=` with what you just copied into Notepad. Save and close *cmdline.txt*.
7. Open the downloaded file *unattended* with an editor like Notepad
8. Replace the test YOURHOSTNAMEHERE with the hostname that you'd like your raspberry pi to have.
9. If you want to change the default password of the pi user, find the line that starts with `#usermod` and remove the `#`
10. Save and close the file.
11. If you want to change the default password of the pi user, you'll also have to open *pipassword* with an editor like Notepad
12. Replace the text YOURPASSWORDHERE with the password that you'd like the pi user to have, and save and close
13. Open the file *wpa_supplicant.conf* with an editor like Notepad
14. Modify YOURWIFINETWORKNAME and WIFIPASSWORD to match your wifi network, and save and close the file
15. Copy all 4 files: *cmdline.txt*, *unattended*, *pipassword* and *wpa_supplicant.conf* to the SD card
16. Eject the SD card, insert in your raspberry pi and boot it up

These scripts will enable ssh, vnc and a picamera. They'll set the hostname, the pi user's password, the screen resolution, locale, timezon and keyboard type. Feel free to modify, but make sure you enter valid values. If you're running headless, or even with a screen attached, bugs are hard to track down.

\* when your commands run, the PATH is `/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:.`; the working directory is `/`; the entire Linux partition is available; systemd isn't up yet so there are no services, the network is unavailable, and the system thinks it's January 1st, 1970.

\*\* the [cmdline.txt](./cmdline.txt) in this project is from Raspberry Pi OS with Desktop from 2021-01-11. If you happen to have that version, you can drop my cmdline.txt in (no need to modify the string after `PARTUUID=`).

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
