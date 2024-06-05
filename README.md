# rpi5 setup

## settings

### misc

```bash
sudo raspi-config
sudo rpi-eeprom-update
```

### adding additional partitions

1. After installing pi os, run it once to set everything up
2. Use a different os to access the drive, use gparted to shrink root partition, and create additional partitions (eg for var, home)
3. Mount root and additional partitions eg:

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
```

6. Add entries to fstab:

```
PARTUUID=YOUR_PART_UUID-03  /var         ext4    defaults,noatime    0 2
PARTUUID=YOUR_PART_UUID-04  /home           ext4    defaults,noatime    0 2
```

### framebuffer 32 bit depth

In `/boot/firmware/cmdline.txt`, add (for first plug) to the end: 

~~`video=HDMI-A-1:-32`~~ 

`video=HDMI-A-1:1920x1080M-32@60`

### disable hdmi audio (untested)

In `/boot/firmware/config.txt`, add `noaudio` to end of `dtoverlay=vc4-kms-v3d`:

`dtoverlay=vc4-kms-v3d,noaudio`
 
### disable wifi, bluetooth (untested)

At the end of `/boot/firmware/config.txt` add:

```
dtoverlay=disable-wifi
dtoverlay=disable-bt
```

### manually control fan

disable: `pinctrl FAN_PWM op dh`

full: `pinctrl FAN_PWM op dl`

auto: `pinctrl FAN_PWM a0`


### set speeds for temps (not working?)

At the end of `/boot/firmware/config.txt` (these are the defaults [from here](file:///boot/firmware/overlays/README)):

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

### ramdisks

```
# for /etc/fstab

tmpfs /tmp tmpfs nodev,nosuid,mode=1777 0 0
tmpfs /var/tmp tmpfs nodev,nosuid,mode=1777 0 0

tmpfs /home/someone/.cache tmpfs nodev,nosuid,mode=1777 0 2

#tmpfs /home/someone/.cache/thumbnails tmpfs nodev,nosuid,mode=1777 0 0
#tmpfs /home/someone/.cache/vlc tmpfs nodev,nosuid,mode=1777 0 0
#tmpfs /home/someone/.cache/chromium tmpfs nodev,nosuid,mode=1777 0 0
```

~~`mkdir -p ./home/someone/.cache/thumbnails ./home/someone/.cache/vlc ./home/someone/.cache/chromium`~~

## apps

### installs

```bash
sudo apt update
sudo apt full-upgrade
sudo apt install thunar thunar-archive-plugin thunar-media-tags-plugin thunar-volman scite terminator vlc xbindkeys solaar viewnior tigervnc-viewer i3-wm i3blocks dmenu unclutter
```

### autostart

```bash
echo -e "\n[autostart]" >> $HOME/.config/wayfire.ini
echo "xbindkeys=xbindkeys -p" >> $HOME/.config/wayfire.ini
echo "solaar=solaar -w hide" >> $HOME/.config/wayfire.ini
echo "unclutter=unclutter -idle 2 -jitter 2 -root" >> $HOME/.config/wayfire.ini
echo "thunar=thunar --daemon" >> $HOME/.config/wayfire.ini
```

### shortcuts

```bash
echo -e '"chromium-browser"\nMod4+b\n' >> $HOME/.xbindkeysrc
echo -e '"thunar"\nMod4+f\n' >> $HOME/.xbindkeysrc
echo -e '"scite"\nMod4+s\n' >> $HOME/.xbindkeysrc
echo -e '"lxtask"\nControl+Shift+Escape\n' >> $HOME/.xbindkeysrc
echo -e '"moonlight-qt"\nMod4+m\n' >> $HOME/.xbindkeysrc	
echo -e '"scrot ~/Pictures/screenshot_$(date +%Y_%m_%d_%H_%M_%S_%3N).png"\nMod4+p\n' >> $HOME/.xbindkeysrc
```

### gtk settings

```bash
mkdir -p  $HOME/.config/gtk-2.0 $HOME/.config/gtk-3.0
mkdir -p $HOME/Desktop $HOME/Documents $HOME/Downloads $HOME/Pictures $HOME/Videos
echo 'gtk-recent-files-max-age=0' >> $HOME/.config/gtk-2.0/gtkrc
echo -e '[Settings]\ngtk-recent-files-max-age=0\ngtk-recent-files-limit=0' > $HOME/.config/gtk-3.0/settings.ini
echo -e "file://$HOME/Documents Documents\nfile://$HOME/Downloads Downloads\nfile://$HOME/Pictures Pictures\nfile://$HOME/Videos Videos\nfile:///tmp tmp" >> $HOME/.config/gtk-3.0/bookmarks	
echo -e '[Filechooser Settings]\nLocationMode=path-bar\nShowHidden=true\nShowSizeColumn=true\nSortColumn=name\nSortOrder=ascending\nStartupMode=recent' > $HOME/.config/gtk-2.0/gtkfilechooser.ini
```

### viewnior

```bash
mkdir -p $HOME/.config/viewnior
echo -e '[prefs]\nzoom-mode=3\nfit-on-fullscreen=true\nshow-hidden=true\nsmooth-images=true\nconfirm-delete=true\nreload-on-save=true\nshow-menu-bar=false\nshow-toolbar=true\nstart-maximized=false\nslideshow-timeout=5\nauto-resize=false\nbehavior-wheel=2\nbehavior-click=0\nbehavior-modify=2\njpeg-quality=100\npng-compression=9\ndesktop=1\n' > $HOME/.config/viewnior/viewnior.conf
```

### thunar

```bash
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

```bash
sudo sed -i 's/\(autologin-session=\)\(.*\)/#\1\2\n\1i3/g' /etc/lightdm/lightdm.conf
```

```bash
mkdir -p $HOME/.config/i3
cp -f /etc/i3/config $HOME/.config/i3/

sed -i 's/\(^exec --no-startup-id.*\)/#\1/g' $HOME/.config/i3/config

sed -i 's/\(bindsym Mod1+d exec\) \(dmenu_run\)/\1 --no-startup-id \2/g' $HOME/.config/i3/config
sed -i 's/\(exec i3-config-wizard\)/#\1/g' $HOME/.config/i3/config
sed -i 's/# \(bindsym Mod1+\)\(d exec --no-startup-id i3-dmenu-desktop\)/\1Shift+\2/g' $HOME/.config/i3/config
sed -i 's/\(set \$mod\) Mod4/\1 Mod1/g' $HOME/.config/i3/config
sed -i '/^bar {$/ a\\t#mode hide\n\t#hidden_state hide\n\tmodifier Mod1' $HOME/.config/i3/config
sed -i '/^bar {$/ a\\t#tray_output primary' $HOME/.config/i3/config
#sed -i 's/^\(font pango:\).*/\1Ubuntu Mono 14/g' $HOME/.config/i3/config

sed -i '/^font pango.*/a#font pango:Ubuntu Mono 18' $HOME/.config/i3/config

echo 'bindsym Mod1+Shift+h bar mode toggle' >> $HOME/.config/i3/config
echo -e '\n#\nworkspace_layout tabbed\ndefault_orientation vertical' >> $HOME/.config/i3/config
echo 'for_window [window_role="pop-up"] floating enable' >> $HOME/.config/i3/config
echo 'for_window [title="File Operation Progress"] floating enable' >> $HOME/.config/i3/config
echo '#for_window [class="Chromium"] floating disable' >> $HOME/.config/i3/config	
sed -i '/^# kill focused window/abindsym Mod1+Shift+x exec xdotool getwindowfocus windowkill' $HOME/.config/i3/config	
echo 'bindsym Mod1+Control+Shift+s exec systemctl suspend' >> $HOME/.config/i3/config
echo 'bindsym Mod1+Control+Shift+h exec systemctl hibernate' >> $HOME/.config/i3/config
```

### i3 blocks

```bash
sed -i 's/\(status_command \)i3status/\1i3blocks/g' $HOME/.config/i3/config

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

```bash
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
```

```bash
echo -e 'selection.back=#CCBDFF\nselection.alpha=50\nselection.layer=1\n' >> $HOME/.SciTEUser.properties
echo -e 'caret.line.back=#CCDDFF\ncaret.fore=#FFFFFF\n' >> $HOME/.SciTEUser.properties
echo -e 'highlight.current.word=1\nhighlight.current.word.indicator=style:straightbox,colour:#FFBBDD,fillalpha:255,under\nstyle.*.34=back:#51DAEA\n' >> $HOME/.SciTEUser.properties


#echo -e 'selection.back=#227733\nselection.alpha=50\nselection.layer=1\n' >> $HOME/.SciTEUser.properties
#echo -e 'caret.line.back=#444444\n' >> $HOME/.SciTEUser.properties
#echo -e 'highlight.current.word=1\nhighlight.current.word.indicator=style:straightbox,colour:#777777,fillalpha:255,under\nstyle.*.34=back:#22AAFF\n' >> $HOME/.SciTEUser.properties
#echo -e 'style.*.32=$(font.base),back:#101010,fore:#BBBBDD\nstyle.*.33=$(font.base),back:#101010\n' >> $HOME/.SciTEUser.properties
#echo -e 'font.base=font:Verdana,size:16\nfont.small=font:Verdana,size:14\nfont.comment=font:Georgia,size:16\n' >> $HOME/.SciTEUser.properties
```

```bash
mkdir -p /usr/local/share/applications
echo -e '[Desktop Entry]\nName=SciTE New Window\nType=Application\nExec=SciTE -check.if.already.open=0 %F\nIcon=Sci48M\nMimeType=text/plain;' > /usr/local/share/applications/scitenew.desktop
echo -e '[Desktop Entry]\nName=SciTE as Root\nType=Application\nExec=gksudo -k "SciTE -check.if.already.open=0 %F"\nIcon=Sci48M\nMimeType=text/plain;' > /usr/local/share/applications/sciteroot.desktop
sed -i 's/rust //g' /usr/share/scite/SciTEGlobal.properties
```

### tigervnc

```bash
echo -e '#!/bin/bash\n\nPASSWD_FILE=~/.vnc/passwd\n\nif [ $1 ]; then\n\techo $1 | vncpasswd -f > $PASSWD_FILE\nfi\n\nvncviewer passwd=$PASSWD_FILE' > /usr/local/bin/tigervnc.sh
chmod +xr /usr/local/bin/tigervnc.sh
```

```bash
mkdir -p $HOME/.vnc
echo -e 'TigerVNC Configuration file Version 1.0\n\nDotWhenNoCursor=1\nRemoteResize=1\nMenuKey=' > $HOME/.vnc/default.tigervnc
```

### xbox one controller

```bash
#sudo apt install xboxdrv
sudo apt install --no-install-recommends dkms
sudo apt-get -y install cabextract

#sudo ./uninstall.sh
git clone https://github.com/medusalix/xone
cd xone
sudo ./install.sh
sudo xone-get-firmware.sh
```

### moonlight

```bash
wget "https://dl.cloudsmith.io/public/moonlight-game-streaming/moonlight-qt/setup.deb.sh"
distro=raspbian sudo -E bash setup.deb.sh
sudo apt install moonlight-qt
```

## nvme

### fix nvme disconnecting

in `/boot/firmware/cmdline.txt` try adding at the end of the line, either or both: 

`nvme_core.default_ps_max_latency_us=0 pcie_aspm=off`

### fix nvme power management

Helps if nvme keeps disconnecting with default value, or if your ssd is running hot with it disabled (above).

Get list of exit latency (Ex_Lat) values: 

`sudo smartctl -c /dev/nvme0n1`

In `/boot/firmware/cmdline.txt` add:

`nvme_core.default_ps_max_latency_us=YOUR_EX_LAT_VALUE`

Change `YOUR_EX_LAT_VALUE` to one of the `Ex_Lat` values eg try second largest value.

### nvme tools
```
sudo apt install smartmontools nvme-cli 
```

### check nvme for errors

`sudo journalctl -b | grep -i nvme`

### check nvme sensors

`sudo nvme smart-log /dev/nvme0n1`

`sudo smartctl -a /dev/nvme0n1`

### get nvme lnkcap info

`lspci | grep -i nvme  | awk '{printf -s $1}' | sudo lspci -vv | grep -w LnkCap`

### change nvme gen (eg gen 1)

Add to `/boot/firmware/config.txt ` (untested):

```
dtparam=pciex1
dtparam=pciex1_gen=1
```

### disable unecessary nvme services

```
sudo systemctl disable nvmf-autoconnect.service
sudo systemctl disable nvmefc-boot-connections.service
```