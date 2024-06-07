### useful commands

`sudo raspi-config`

Never run `sudo rpi-update`.

### installs

```bash
sudo apt update
sudo apt full-upgrade
sudo apt install thunar thunar-archive-plugin thunar-media-tags-plugin thunar-volman scite terminator vlc xbindkeys solaar viewnior tigervnc-viewer i3-wm i3blocks dmenu unclutter blueman
```

### adding additional partitions

1. After installing pi os, run it once to set everything up
2. Use a different os to access the drive, use gparted to shrink root partition, and create additional partitions (eg for var, home)
3. Temporarily mount root, var, home partitions:

```
mkdir temp
cd temp

mkdir -p root home var

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

### Fix keyboard (eg hash key is pound symbol)

Call `sudo raspi-config` => `Localisation Options` => `Keyboard`


### Reduce power when turned off

`sudo rpi-eeprom-config -e`

change `POWER_OFF_ON_HALT=0` to `POWER_OFF_ON_HALT=1`


### set speeds for temps

`sudo nano /boot/firmware/config.txt` 

Add to end (these are the defaults):

```
dtparam=fan_temp0=50000
dtparam=fan_temp0_hyst=5000
dtparam=fan_temp0_speed=75

dtparam=fan_temp1=60000
dtparam=fan_temp1_hyst=5000
dtparam=fan_temp1_speed=125

dtparam=fan_temp2=67500
dtparam=fan_temp2_hyst=5000
dtparam=fan_temp2_speed=175

dtparam=fan_temp3=75000
dtparam=fan_temp3_hyst=5000
dtparam=fan_temp3_speed=250
```

### manually control fan

disable: `pinctrl FAN_PWM op dh`

full: `pinctrl FAN_PWM op dl`

auto: `pinctrl FAN_PWM a0`


### disable wifi, bluetooth (untested)

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

## nvme

### install nvme tools

`sudo apt install smartmontools nvme-cli`

### check nvme for errors

`sudo journalctl -b | grep -i nvme`

### check nvme sensors

`sudo nvme smart-log /dev/nvme0n1`

`sudo smartctl -a /dev/nvme0n1`

### get nvme lnkcap info

`lspci | grep -i nvme  | awk '{printf -s $1}' | sudo lspci -vv | grep -w LnkCap`

### disable unecessary nvme services

```
sudo systemctl disable nvmf-autoconnect.service
sudo systemctl disable nvmefc-boot-connections.service
```

### fix nvme disconnecting

#### disable power management

`sudo nano /boot/firmware/cmdline.txt`

add to end: `nvme_core.default_ps_max_latency_us=0 pcie_aspm=off`

#### update firmware

`sudo rpi-eeprom-update -a`

#### change max exit latency (maybe helps)

Get list of exit latency (Ex_Lat) values: 

`sudo smartctl -c /dev/nvme0n1`

Add to end:

`sudo nano /boot/firmware/cmdline.txt`

`nvme_core.default_ps_max_latency_us=YOUR_EX_LAT_VALUE` 

Get `Ex_Lat` value from list above (try second largest, then third etc if it keeps disconnecting).

#### change nvme gen 1  (maybe helps)

`sudo nano /boot/firmware/config.txt `

```
dtparam=pciex1
dtparam=pciex1_gen=1
```

## apps

### autostart

```
echo -e "\n[autostart]" >> $HOME/.config/wayfire.ini
echo "xbindkeys=xbindkeys -p" >> $HOME/.config/wayfire.ini
echo "solaar=solaar -w hide" >> $HOME/.config/wayfire.ini
echo "unclutter=unclutter -idle 2 -jitter 2 -root" >> $HOME/.config/wayfire.ini
echo "thunar=thunar --daemon" >> $HOME/.config/wayfire.ini
echo "blueman=blueman-applet" >> $HOME/.config/wayfire.ini
```

or:

```
echo "xbindkeys -p &" >> $HOME/autostart.sh
echo "solaar -w hide &" >> $HOME/autostart.sh
echo "unclutter -idle 2 -jitter 2 -root &" >> $HOME/autostart.sh
echo "thunar --daemon &" >> $HOME/autostart.sh
echo "blueman-applet &" >> $HOME/autostart.sh
sudo chmod +xr $HOME/autostart.sh
```

### shortcuts

```
echo -e '"chromium-browser"\nMod4+b\n' >> $HOME/.xbindkeysrc
echo -e '"thunar"\nMod4+f\n' >> $HOME/.xbindkeysrc
echo -e '"scite"\nMod4+s\n' >> $HOME/.xbindkeysrc
echo -e '"lxtask"\nControl+Shift+Escape\n' >> $HOME/.xbindkeysrc
echo -e '"moonlight-qt"\nMod4+m\n' >> $HOME/.xbindkeysrc	
echo -e '"scrot ~/Pictures/screenshot_$(date +%Y_%m_%d_%H_%M_%S_%3N).png"\nMod4+p\n' >> $HOME/.xbindkeysrc
echo -e '"scrot ~/Pictures/screenshot_$(date +%Y_%m_%d_%H_%M_%S_%3N).png"\nControl+p\n' >> $HOME/.xbindkeysrc
```

### gtk settings

```
mkdir -p  $HOME/.config/gtk-2.0 $HOME/.config/gtk-3.0
mkdir -p $HOME/Desktop $HOME/Documents $HOME/Downloads $HOME/Pictures $HOME/Videos
echo 'gtk-recent-files-max-age=0' >> $HOME/.config/gtk-2.0/gtkrc
echo -e '[Settings]\ngtk-recent-files-max-age=0\ngtk-recent-files-limit=0' > $HOME/.config/gtk-3.0/settings.ini
echo -e "file://$HOME/Documents Documents\nfile://$HOME/Downloads Downloads\nfile://$HOME/Pictures Pictures\nfile://$HOME/Videos Videos\nfile:///tmp tmp" >> $HOME/.config/gtk-3.0/bookmarks	
echo -e '[Filechooser Settings]\nLocationMode=path-bar\nShowHidden=true\nShowSizeColumn=true\nSortColumn=name\nSortOrder=ascending\nStartupMode=recent' > $HOME/.config/gtk-2.0/gtkfilechooser.ini
```

### viewnior

```
mkdir -p $HOME/.config/viewnior
echo -e '[prefs]\nzoom-mode=3\nfit-on-fullscreen=true\nshow-hidden=true\nsmooth-images=true\nconfirm-delete=true\nreload-on-save=true\nshow-menu-bar=false\nshow-toolbar=true\nstart-maximized=false\nslideshow-timeout=5\nauto-resize=false\nbehavior-wheel=2\nbehavior-click=0\nbehavior-modify=2\njpeg-quality=100\npng-compression=9\ndesktop=1\n' > $HOME/.config/viewnior/viewnior.conf
```

### thunar

```
mkdir -p $HOME/.config/xfce4/xfconf/xfce-perchannel-xml
echo -e '<?xml version="1.0" encoding="UTF-8"?>\n<channel name="thunar" version="1.0">\n\t<property name="last-show-hidden" type="bool"\nvalue="true"/>\n\t<property name="last-view" type="string" value="ThunarDetailsView"/>\n</channel>' >> $HOME/.config/xfce4/xfconf/xfce-perchannel-xml/thunar.xml
```

### vlc

```bash
mkdir -p $HOME/.config/vlc
echo -e "[qt4]\nqt-recentplay=0\nqt-privacy-ask=0\n\n[core]\nvideo-title-show=0\nplay-and-exit=1\none-instance-when-started-from-file=0\nsnapshot-path=$HOME/Pictures\nsnapshot-prefix=$N_[$T]_\nsnapshot-sequential=1\nkey-vol-up=Ctrl+Up\nkey-vol-down=Ctrl+Down\nkey-vol-mute=m\nkey-stop=\nkey-snapshot=s\nstats=0\nstereo-mode=1" > $HOME/.config/vlc/vlcrc
echo -e '[MainWindow]\nstatus-bar-visible=true' > $HOME/.config/vlc/vlc-qt-interface.conf
```

### terminator

```bash
mkdir -p $HOME/.config/terminator $HOME/.config/xfce4
echo -e '[global_config]\n  inactive_color_offset = 1.0\n[keybindings]\n  full_screen = ""\n   help = ""\n[profiles]\n [[default]]\n  show_titlebar = False\n  scrollbar_position = disabled' > $HOME/.config/terminator/config
echo 'TerminalEmulator=terminator' >> $HOME/.config/xfce4/helpers.rc
```

### i3wm

```
sudo sed -i 's/\(autologin-session=\)\(.*\)/#\1\2\n\1i3/g' /etc/lightdm/lightdm.conf
```

```
mkdir -p $HOME/.config/i3
cp -f /etc/i3/config $HOME/.config/i3/

sed -i 's/\(^exec --no-startup-id.*\)/#\1/g' $HOME/.config/i3/config
sed -i 's/\(bindsym Mod1+d exec\) \(dmenu_run\)/\1 --no-startup-id \2/g' $HOME/.config/i3/config
sed -i 's/\(exec i3-config-wizard\)/#\1/g' $HOME/.config/i3/config
sed -i 's/# \(bindsym Mod1+\)\(d exec --no-startup-id i3-dmenu-desktop\)/\1Shift+\2/g' $HOME/.config/i3/config
sed -i 's/\(set \$mod\) Mod4/\1 Mod1/g' $HOME/.config/i3/config
sed -i '/^bar {$/ a\\t#mode hide\n\t#hidden_state hide\n\tmodifier Mod1' $HOME/.config/i3/config
sed -i '/^bar {$/ a\\t#tray_output primary' $HOME/.config/i3/config
sed -i '/^# kill focused window/abindsym Mod1+Shift+x exec xdotool getwindowfocus windowkill' $HOME/.config/i3/config
sed -i 's/^\(font pango:.*\)/#\1\nfont pango:DejaVu Sans 18/g'  $HOME/.config/i3/config

echo 'bindsym Mod1+Shift+h bar mode toggle' >> $HOME/.config/i3/config
echo -e '\n#\nworkspace_layout tabbed\ndefault_orientation vertical' >> $HOME/.config/i3/config
echo -e '\n#' >> $HOME/.config/i3/config
echo 'for_window [window_role="pop-up"] floating enable' >> $HOME/.config/i3/config
echo 'for_window [title="File Operation Progress"] floating enable' >> $HOME/.config/i3/config
echo -e '\n#' >> $HOME/.config/i3/config
echo 'assign [class="Moonlight"] 3' >> $HOME/.config/i3/config
echo 'assign [class="Chromium"] 1' >> $HOME/.config/i3/config

echo -e '\n#\n#exec --no-startup-id ~/runstart.sh' >> $HOME/.config/i3/config
```

### i3 blocks

```
sed -i 's/\(status_command \)i3status/\1i3blocks/g' $HOME/.config/i3/config
```

```
mkdir -p $HOME/.config/i3blocks
echo -n '' >  $HOME/.config/i3blocks/config

echo -e '\n[cpu_load]\ncolor=#FFFFFF\ncommand=mpstat -P ALL 1 1 |  awk '"'"'/Average:/ && $2 ~ /[0-9]/ {printf "%.0f\\x25 ",100-$12}'"'"'|xargs\ninterval=10' >> $HOME/.config/i3blocks/config
echo -e '\n[cpu_hertz]\ncolor=#FFBB66\ncommand=find /sys/devices/system/cpu/cpu[0-3]/cpufreq/scaling_cur_freq  -type f |xargs cat | awk '"'"'{printf "%.1f ",$1/1000000}'"'"'|xargs\ninterval=5' >> $HOME/.config/i3blocks/config
echo -e '\n[memory_free]\ncolor=#EEFF88\ncommand=awk '"'"'/MemAvailable/ {printf("%d\\xcb\\x96\\n", ($2/1000))}'"'"' /proc/meminfo\ninterval=2' >> $HOME/.config/i3blocks/config
echo -e '\n#[memory_used]\n#color=#FFDDCC\n#command=awk '"'"'/MemTotal|MemAvailable/ {print $2}'"'"' /proc/meminfo | paste -sd'"'"' '"'"' | awk '"'"'{printf "%d\\xcb\\x97\\n",($1-$2)/1000}'"'"'\n#interval=2' >> $HOME/.config/i3blocks/config
echo -e '\n#[swap_free]\n#command=awk '"'"'/SwapTotal|SwapFree/ {print $2}'"'"' /proc/meminfo | paste -sd'"'"' '"'"' | awk '"'"'{printf "<span color=\\"%s\\">%d\\xcb\\x96</span>\\n",$1=="0"?"#555555":"#FFDDCC",$2/1000}'"'"'\n#interval=2\n#markup=pango' >> $HOME/.config/i3blocks/config
echo -e '\n[swap_used]\ncommand=awk '"'"'/SwapTotal|SwapFree/ {print $2}'"'"' /proc/meminfo | paste -sd'"'"' '"'"' | awk '"'"'{printf "<span color=\\"%s\\">%d\\xcb\\x97</span>\\n",$1=="0"?"#555555":"#FFDDCC",($1-$2)/1000}'"'"'\ninterval=2\nmarkup=pango' >> $HOME/.config/i3blocks/config
echo -e '\n[temp]\ncolor=#85C1E9\ncommand=cat /sys/class/thermal/thermal_zone*/temp | awk '"'"'$1 {printf "%.0f\\xc2\\xb0 ",$1/1000}'"'"'|awk '"'"'$1=$1'"'"'\ninterval=2' >> $HOME/.config/i3blocks/config
echo -e '\n[fan]\ncolor=#85E9C1\ncommand=cat /sys/devices/platform/cooling_fan/hwmon/*/fan1_input | xargs\ninterval=2' >> $HOME/.config/i3blocks/config

echo -e '\n[time]\ncommand=date "+%a %d %b, %I:%M %p"\ninterval=5' >> $HOME/.config/i3blocks/config
echo -e '\n#[battery]\n#color=#58D68D\n#command=cat /sys/class/power_supply/BAT1/status /sys/class/power_supply/BAT1/capacity | tr "\\n" " " | awk '"'"'$1 $2 {printf "<span color=\\"%s\\">\\xe2\\x9a\\xa1</span>%s%%", $1=="Charging"?"yellow":"light grey",$2}'"'"'\n#interval=2\n#markup=pango' >> $HOME/.config/i3blocks/config
echo -e '\n[volume]\ncommand=amixer -c 0 -M -D pulse get Master | sed "s/[][%]//g" | awk '"'"'/Front Left:.+/ {printf "<span color=\\"%s\\">%s\\xE2\\x99\\xAA</span>\\n",$6=="off"?"#333333":"#FFFFFF",$5}'"'"'\ninterval=5\nmarkup=pango' >> $HOME/.config/i3blocks/config
```

### scite

```
echo '' > $HOME/.SciTEUser.properties

echo -e 'check.if.already.open=1\nload.on.activate=1\nquit.on.close.last=1\n' >> $HOME/.SciTEUser.properties
echo -e 'open.filter=$(all.files)\n' >> $HOME/.SciTEUser.properties
echo -e 'statusbar.visible=1\ntitle.full.path=1\ntoolbar.visible=0\n' >> $HOME/.SciTEUser.properties
echo -e 'line.margin.visible=1\nline.margin.width=1+\noutput.wrap=1\nwrap=1\n' >> $HOME/.SciTEUser.properties
echo -e 'save.session=1\nsave.recent=0\nsave.find=1\nsave.position=1\n' >> $HOME/.SciTEUser.properties
echo -e '\nindent.size=4\ntabsize=4\nuse.tabs=0\n' >> $HOME/.SciTEUser.properties
echo -e 'use.tabs.$(file.patterns.make)=1\n' >> $HOME/.SciTEUser.properties
echo -e '\nstatusbar.text.1=pos=$(CurrentPos),li=$(LineNumber), co=$(ColumnNumber) [$(EOLMode)]\next.lua.startup.script=$(SciteUserHome)/.SciTEStartup.lua\n' >> $HOME/.SciTEUser.properties
echo -e 'function OnUpdateUI() props["CurrentPos"]=editor.CurrentPos end' > $HOME/.SciTEStartup.lua

echo -e '\n###darkmode' >> $HOME/.SciTEUser.properties
echo -e '\nimports.exclude=shell markdown conf cmake cpp lisp lua css html json python rust tcl yaml' >> $HOME/.SciTEUser.properties
echo -e '\nselection.back=#227733\nselection.alpha=50\nselection.layer=1' >> $HOME/.SciTEUser.properties
echo -e '\ncaret.line.back=#444444\ncaret.fore=#FFFFFF\ncaret.period=0\ncaret.width=2\n#caret.style=2' >> $HOME/.SciTEUser.properties
echo -e '\nhighlight.current.word=1\nhighlight.current.word.indicator=style:straightbox,colour:#777777,fillalpha:255,under\nstyle.*.34=back:#22AAFF' >> $HOME/.SciTEUser.properties
echo -e '\nstyle.*.32=$(font.base),back:#101010,fore:#BBBBDD\nstyle.*.33=$(font.base),back:#101010' >> $HOME/.SciTEUser.properties
echo -e '\nfont.base=font:Verdana,size:16\nfont.small=font:Verdana,size:14\nfont.comment=font:Georgia,size:16' >> $HOME/.SciTEUser.properties

echo -e '\n###lightmode' >> $HOME/.SciTEUser.properties
echo -e '\n#selection.back=#CCBDFF\n#selection.alpha=50\n#selection.layer=1' >> $HOME/.SciTEUser.properties
echo -e '\n#caret.line.back=#CCDDFF\n#caret.fore=#FFFFFF' >> $HOME/.SciTEUser.properties
echo -e '\n#highlight.current.word=1\n#highlight.current.word.indicator=style:straightbox,colour:#FFBBDD,fillalpha:255,under\n#style.*.34=back:#51DAEA' >> $HOME/.SciTEUser.properties
```

### xbox one controller

#### xone

```bash
sudo apt install --no-install-recommends dkms
sudo apt-get -y install cabextract

#sudo ./uninstall.sh
git clone https://github.com/medusalix/xone
cd xone
sudo ./install.sh
sudo xone-get-firmware.sh
```

#### xpadneo
```
sudo apt install --no-install-recommends dkms
sudo apt install raspberrypi-kernel-headers

git clone https://github.com/atar-axis/xpadneo.git
cd xpadneo
sudo ./install.sh
```

### moonlight

```
wget "https://dl.cloudsmith.io/public/moonlight-game-streaming/moonlight-qt/setup.deb.sh"
distro=raspbian sudo -E bash setup.deb.sh
sudo apt install moonlight-qt
```
