# Problems with the Raspberry Pi 5

## No sleep or hibernate

* unlikely to be ever added

## No boot menu at startup

* if the boot has been set to nvme, you have to physically remove the drive to get back to the sd card

* better to use boot from sd card setting and remove it to boot from the nvme

## Bright red led when off

* haven't been able to turn it off

## Cooler fan spins at high speed on boot

* while normal desktop computers do ths too, the noise from the small fan is loud

* doesn't seem to always happen (maybe when the pi has been turned off at the wall?)

## Geekworm X1002 (nvme board)

* when `nvme_core.default_ps_max_latency_us` is enabled, the board emits a high pitched whining (due to fluctuating power levels?)

* four spots on the side of the board where if touched making a weird metal brush sound (instead of the smooth plastic feel you'd expect)

* the ribbon cable (copper coloured one) will short against metal cases, so you need to place electrical tape between them (they are aware of the problem, might have a fix by now)

* no replacement PCIe FFC cable available for sale

* pain to access/remove nvme drive, have to take whole thing out of the case

## Geekworm P580 (case)

* doesn't come with any kind of a button, only a cut out (annoying to use)

* green/red power led from pi and the blue led from the nvme board shine too brightly through the case (could use white electrical tape over the leds to diffuse them?)

* slight gap between sides of the lid and case, but not a big deal

* sd card can be inserted but not remove, have to take the pi out of the case to do so

* the case is a bit wider than necessary (older pi form factor?)

* for the ribbon cable between the pi and nvme, there is an adequate gap, but the cable going into the pi had to bend around a bit, but seems to work fine

* on the inside of the case there are slight spots where they missed with the paint

## Geekworm Active Cooler H505 (Armor Cooler)

* fan gives off a plastic smell

# Other

## NVME Drives

* better to use cheaper ones in case you have compatibility problems

### WD Blue SN580

* with power saving enabled can idle at around 40 degrees, but when off it idles at 60 (sometimes up to 65 degrees?)

* had disconnecting issues (when nvme gen not set to 1?)

### Patriot P300 (non us version)

* with power savings disabled, idles around 40 degrees

## Top hat nvme

* heat from the nvme can escape more easily

* more cases support top hats

* ribbon cable from the pi doesn't need to bend so much

* the Geekworm x1001 and M901 has a gap for heat and air to pass through from/to the pi

* not sure how bad the full covering top hats are for dealing with heat from the pi

* easier access to nvme drive, can just take lid off?

## Sound Blaster Play! 4

* works without any setup

* USB 3 can cause static/buzz? (use USB 2 instead)

* about 5% of the time it fails to initialise on boot, but can fix with: `sudo usbreset "Sound Blaster Play! 4"`


## Misc

* had to change vlc video output to xvideo xcb for it work in fullscreen

* wayfire's autostart only works when installing from `Raspberry Pi OS with desktop and recommended software` and not `Raspberry Pi OS with desktop`

## Good

* 2500mhz is adequate for web browsing, youtube, moonlight streaming at 1080p 60 fps

* 8gb is also adequate

* when running moonlight, idles at 60 degrees with the cooler fans off, can also have the fans running at low speed, which is quiet
