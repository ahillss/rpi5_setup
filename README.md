### useful commands

`sudo raspi-config`

Never run `sudo rpi-update`.

### installs

```
sudo apt update
sudo apt full-upgrade
sudo apt install thunar thunar-archive-plugin thunar-media-tags-plugin thunar-volman scite terminator vlc xbindkeys solaar viewnior tigervnc-viewer i3-wm i3blocks dmenu unclutter blueman gparted pavucontrol transmission-remote-gtk
```

### adding additional partitions

1. After installing pi os, run it once to set everything up
2. Use a different os to access the drive, use gparted to shrink root partition, and create additional partitions (eg for var, home)
3. Temporarily mount root, var, home partitions:

```
mkdir temp
cd temp

mkdir -p boot root var home

sudo mount /dev/nvme0n1p1 ./boot
sudo mount /dev/nvme0n1p2 ./root
sudo mount /dev/nvme0n1p3 ./var
sudo mount /dev/nvme0n1p4 ./home
```

4. Copy var, home to their new partitions eg:

```
sudo cp -a ./root/home/* ./home/
sudo cp -a ./root/var/* ./var/
```

5. Optionally backup old dirs

```
sudo mv ./root/var ./root/var2
sudo mv ./root/tmp ./root/tmp2
sudo mv ./root/home ./root/home2

sudo mkdir -p ./root/var ./root/tmp ./root/home
```

6. Call `sudo nano ./root/etc/fstab` and insert:

```
PARTUUID=YOUR_PART_UUID-03  /var    ext4    defaults,noatime    0 2
PARTUUID=YOUR_PART_UUID-04  /home   ext4    defaults,noatime    0 2

tmpfs /tmp tmpfs nodev,nosuid,mode=1777 0 0
tmpfs /var/tmp tmpfs nodev,nosuid,mode=1777 0 0

tmpfs /home/YOUR_USER_NAME/.cache tmpfs nodev,nosuid,mode=1777 0 0
```

7. Optionally add sd card mnts
```
sudo mkdir -p ./root/mnt/other_boot ./root/mnt/other_root
```

Add to fstab
```
/dev/mmcblk0p1  /mnt/other_boot   ext4    defaults,noatime,nofail,ro    0 2
/dev/mmcblk0p2  /mnt/other_root   ext4    defaults,noatime,nofail,ro    0 2
```

### Disable swap

`sudo nano /etc/dphys-swapfile`

Set: `CONF_SWAPSIZE=0`


### Fix keyboard (eg hash key is pound symbol)

Call `sudo raspi-config` => `Localisation Options` => `Keyboard`

### Reduce power when turned off

`sudo rpi-eeprom-config -e`

Set `POWER_OFF_ON_HALT=1`


### set speeds for temps

`sudo nano /boot/firmware/config.txt` 

Add to end (these are the defaults):

```
dtparam=fan_temp0=65000
dtparam=fan_temp0_hyst=3000
dtparam=fan_temp0_speed=75

dtparam=fan_temp1=70000
dtparam=fan_temp1_hyst=5000
dtparam=fan_temp1_speed=125

dtparam=fan_temp2=75000
dtparam=fan_temp2_hyst=5000
dtparam=fan_temp2_speed=175

dtparam=fan_temp3=80000
dtparam=fan_temp3_hyst=5000
dtparam=fan_temp3_speed=250
```

### manually control fan

disable: `pinctrl FAN_PWM op dh`

full: `pinctrl FAN_PWM op dl`

auto: `pinctrl FAN_PWM a0`

```
sudo bash -c "echo 'pinctrl FAN_PWM op dh' > /usr/local/bin/fan_off.sh"
sudo bash -c "echo 'pinctrl FAN_PWM a0' > /usr/local/bin/fan_on.sh"
sudo chmod +xr /usr/local/bin/fan_off.sh /usr/local/bin/fan_on.sh
```

### disable wifi, bluetooth

`sudo nano /boot/firmware/config.txt`

Add to end:

```
dtoverlay=disable-wifi
dtoverlay=disable-bt
```

### disable hdmi audio (untested)

`sudo nano /boot/firmware/config.txt`

Add `noaudio` to end of `dtoverlay=vc4-kms-v3d`: `dtoverlay=vc4-kms-v3d,noaudio`

### framebuffer 32 bit depth

`sudo nano /boot/firmware/cmdline.txt`

add to the end (for the first plug): ~~`video=HDMI-A-1:-32`~~ `video=HDMI-A-1:1920x1080M-32@60`

### disable ethernet leds

`sudo nano /boot/firmware/config.txt`

Add:

```
dtparam=eth_led0=4
dtparam=eth_led1=4
```

### disable power led?

`sudo nano /boot/firmware/config.txt`

Add:

```

#dtparam=pwr_led_trigger=none
#dtparam=act_led_trigger=default-on

dtparam=pwr_led_trigger=default-on
dtparam=act_led_trigger=none

dtparam=pwr_led_activelow=off
dtparam=act_led_activelow=off
```

## nvme

### install nvme tools

`sudo apt install smartmontools nvme-cli`

### check nvme for errors

`sudo journalctl -b | grep -i nvme`

### check nvme sensors

`sudo nvme smart-log /dev/nvme0n1`

`sudo smartctl -a /dev/nvme0n1`

### get nvme model

`sudo nvme list`

### get nvme lnkcap info

`lspci | grep -i nvme  | awk '{printf -s $1}' | sudo lspci -vv | grep -w LnkCap`

### disable unecessary nvme services

```
sudo systemctl disable nvmf-autoconnect.service
sudo systemctl disable nvmefc-boot-connections.service
```

### manually run trim

`sudo fstrim -v -a`

### fix nvme disconnecting

#### disable power management

`sudo nano /boot/firmware/cmdline.txt`

add to end: `pcie_aspm=off nvme_core.default_ps_max_latency_us=0`

#### update firmware

`sudo rpi-eeprom-update -a`

#### set firmware to latest (not sure if can do from here)

sudo nano `/etc/default/rpi-eeprom-update`

Set `FIRMWARE_RELEASE_STATUS="latest"`

#### change nvme gen 1

`sudo nano /boot/firmware/config.txt `

Add:


```
dtparam=pciex1
dtparam=pciex1_gen=1
```

#### change max exit latency (maybe helps)

Instead of disabling it, as that will also increase the temps.

Get list of exit latency (Ex_Lat) values: 

`sudo smartctl -c /dev/nvme0n1`

Add to end:

`sudo nano /boot/firmware/cmdline.txt`

`nvme_core.default_ps_max_latency_us=YOUR_EX_LAT_VALUE` 

Get `Ex_Lat` value from list above (try second largest, then third etc if it keeps disconnecting).

## apps

### autostart

This doesn't work from normal desktop image, only on the full desktop image.

```
echo -e "\n[autostart]" >> $HOME/.config/wayfire.ini
echo "0=xbindkeys -p" >> $HOME/.config/wayfire.ini
echo "1=solaar -w hide" >> $HOME/.config/wayfire.ini
echo "2=thunar --daemon" >> $HOME/.config/wayfire.ini
echo "3=blueman-applet" >> $HOME/.config/wayfire.ini

#echo "unclutter=unclutter -idle 2 -jitter 2 -root" >> $HOME/.config/wayfire.ini
```

### change dpi

```
echo -e "\n[screen]" >> $HOME/.config/wayfire.ini
echo "scale = 1.5" >> $HOME/.config/wayfire.ini
echo 'export QT_SCALE_FACTOR=0.7' >> $HOME/.profile
```

### chromium

```
sudo cp /etc/chromium/master_preferences /etc/chromium/master_preferences.old
```

```
sudo sed -i 'N;/\n}/{s//,&/;P;s/".*,/"bookmark_bar":\{"show_on_all_tabs": true\}/};P;D' /etc/chromium/master_preferences
sudo sed -i 'N;/\n}/{s//,&/;P;s/".*,/"session" : \{ "restore_on_startup" : 1 \}/};P;D' /etc/chromium/master_preferences
sudo sed -i 'N;/\n}/{s//,&/;P;s/".*,/"browser" : \{ "theme": \{ "color_scheme": 2 \} \}/};P;D' /etc/chromium/master_preferences
sudo sed -i 'N;/\n}/{s//,&/;P;s/".*,/"first_run_tabs":["chrome:\/\/newtab"]/};P;D' /etc/chromium/master_preferences
```

```
mkdir -p /$HOME/.config/chromium/Default
echo '{"browser":{"enabled_labs_experiments":["enable-force-dark@6"],"first_run_finished":true}}' > "/$HOME/.config/chromium/Local State"
```

### xbox one controller

```
sudo apt install --no-install-recommends dkms
sudo apt-get -y install cabextract

#sudo ./uninstall.sh
git clone https://github.com/medusalix/xone
cd xone
sudo ./install.sh
sudo xone-get-firmware.sh
```

### moonlight

```
wget "https://dl.cloudsmith.io/public/moonlight-game-streaming/moonlight-qt/setup.deb.sh"
distro=raspbian sudo -E bash setup.deb.sh
sudo apt install moonlight-qt
```
