# provisioning-of-raspberry-pi-4-with-maas
https://discourse.maas.io/t/provisioning-of-raspberry-pi-4-with-maas/2243

## 3.1 Setting-up rpi4 for network boot

This procedure explains how to flash the latest EEPROM firmware and to configure the bootloader to start network boot when the SD card boot fails

- [ ] Insert a micro-sd card (>16GB) ready in the sd-card reader slot
- [ ] Install Raspberry Pi Imager

```
sudo snap install rpi-imager
```

- [ ] Choose OS -> Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32-bit)
- [ ] Choose SD Card, then WRITE
- [ ] Boot your rpi, find the ip address (looking for lease in dhcp server log, in your router web page, etc.), then ssh with password: ‘raspberry’, then update and upgrade, then reboot

```
ssh pi@<ip_addr_of_your_rpi>
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

- [ ] Check the latest stable eeprom fw on github, project raspberrypi, rpi-eeprom/tree/master/firmware/stable (at the time of writing this doc, it was pieeprom-2020-07-16.bin and set the variable accordingly

```
PI_EEPROM_VERSION=pieeprom-2020-07-16
```

- [ ] Update eeprom with latest fw version, get the configuration, set the boot order and update again

```
wget https://github.com/raspberrypi/rpi-eeprom/blob/master/firmware/stable/${PI_EEPROM_VERSION}.bin
sudo rpi-eeprom-config ${PI_EEPROM_VERSION}.bin > bootconf.txt
sed -i 's/BOOT_ORDER=.*/BOOT_ORDER=0x21/g' bootconf.txt
sudo rpi-eeprom-config --out ${PI_EEPROM_VERSION}-netboot.bin --config bootconf.txt ${PI_EEPROM_VERSION}.bin
sudo rpi-eeprom-update -d -f ./${PI_EEPROM_VERSION}-netboot.bin
sudo reboot
```

`NOTE!` For some reasons, the wget command on Raspberry Pi OS results in an incomplete file (about 78KB against 524KB). In this case I had to download to my laptop first and then scp to rpi.

- [ ] Check version and boot parameters are as expected

```
$ vcgencmd bootloader_version
Jul 16 2020 16:15:46
version 45291ce619884192a6622bef8948fb5151c2b456 (release)
timestamp 1594912546
vcgencmd bootloader_config
[all]
BOOT_UART=0
WAKE_ON_GPIO=1
POWER_OFF_ON_HALT=0
DHCP_TIMEOUT=45000
DHCP_REQ_TIMEOUT=4000
TFTP_FILE_TIMEOUT=30000
ENABLE_SELF_UPDATE=1
DISABLE_HDMI=0
BOOT_ORDER=0x21
```

`Note:` BOOT_ORDER should be 0x21 to allow network boot.
