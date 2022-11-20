# HIFIberry-sound-hack

Notes to bypass the HIFIberry - HIFIberryOS sound cards:

Tested on a Raspberry pi 3 - via SSH

## Enable the onboard sound (HDMI and Headphones analog - optional if using a usb external card)

1 - Enable boot as read/write:
`mount -o remount,rw /dev/mmcblk0p1 /boot`

2- Change the line 
from:
      `dtoverlay=vc4-fkms-v3d,audio=off`
to:
      `dtoverlay=vc4-fkms-v3d,audio=on`

3- Restart the HIFIberryOS

## Enable HIFIberry to use other card then the HifiBerry

1 - Open `/opt/hifiberry/bin/reconfigure-players`

The routine below defines what audio output will be used:

```
# Find the correct input and output hardware devices
# this is important if additional USB hardware is connected
detect_hw () {
 HW=`aplay -l | grep hifiberry | awk -F: '{print $1}'`
 HW_SHORT=`echo $HW | awk '{print $2}'`
 fix_asound_hw
}
```

the *hifiberry* has to match the output card to be used  ``` HW=`aplay -l | grep hifiberry | awk -F: '{print $1}' ``` 

Example for the HDMI output - check the avaiable cards with:

```
# aplay -l | grep card
card 0: b1 [bcm2835 HDMI 1], device 0: bcm2835 HDMI 1 [bcm2835 HDMI 1]
card 1: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
card 2: sndrpihifiberry [snd_rpi_hifiberry_dac], device 0: HifiBerry DAC HiFi pcm5102a-hifi-0 [HifiBerry DAC HiFi pcm5102a-hifi-0]
```
IMPORTANT: The grep has to return a unique card. test the output like that

Good string - *HDMI* :
```
# aplay -l | grep HDMI           
card 0: b1 [bcm2835 HDMI 1], device 0: bcm2835 HDMI 1 [bcm2835 HDMI 1]
```
Bad string - *bcm2835*:
```
# aplay -l | grep bcm2835
card 0: b1 [bcm2835 HDMI 1], device 0: bcm2835 HDMI 1 [bcm2835 HDMI 1]
card 1: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
```

2 - Change the string - In case a USB sound card, the board has to appear on aplay output, otherwise wasn't detected by the kernel and will not gonna work.

3 - Restart the HIFIberryOS

--------------------------------------------------------------------------------------------------------------------------------------------------------

To Do 1 - Automation
To Do 2 - Bluetooth audio output
