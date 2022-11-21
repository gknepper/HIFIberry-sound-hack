# HIFIberry-sound-hack


# Hack 1 - Use a non-hifiberry blessed card - Example using the HDMI output -  Can be used the analog or someother USB based sound card 

Notes to bypass the HIFIberry - HIFIberryOS sound cards:

Tested on a Raspberry pi 3 - via SSH

## Enable the onboard sound (HDMI and Headphones analog - optional if using a usb external card)

1 - Enable boot as read/write:
`mount -o remount,rw /dev/mmcblk0p1 /boot`

2 - Change the line on /boot/config.txt
from:
      `dtoverlay=vc4-fkms-v3d,audio=off`
to:
      `dtoverlay=vc4-fkms-v3d,audio=on`

3 - Restart the HIFIberryOS

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


# Hack 2 - Using a bluetooth speaker (Or whatever other bluetooth audio device) as the output:

Tested using a `Raspberry Pi 3 Model B Rev 1.2`

This setup is useful when you have a good bluetooth speaker without line-in, or you want to add the HifiBerryOS features to the bluetooth speaker. So the Hifiberry will work like a bridge receiving the requests and redirecting the audio via bluetooth to the speakers. 

1 - Enable boot as read/write:
`mount -o remount,rw /dev/mmcblk0p1 /boot`

2 - Include the line on /boot/config.txt to add bluetooth support - May not be necessary on other raspberries but it's required on P3 Model B
      `dtparam=krnbt=on`

3 - Create a file to add the bluetooth support on OS
      `touch /etc/features/bluetooth`

4 - Restart the Rasperry

## Once the bluetooth is enabled let's pair the audio device:

run `bluetoothctl`
then: 
```
[bluetooth]# scan on
Discovery started
[CHG] Controller B8:27:EB:B5:A5:F8 Discovering: yes
[NEW] Device 90:D9:94:2B:40:66 ZJ-YUPPIE
[bluetooth]# 
[bluetooth]# connect 90:D9:94:2B:40:66

Attempting to connect to 90:D9:94:2B:40:66
[CHG] Device 90:D9:94:2B:40:66 Connected: yes
[CHG] Device 90:D9:94:2B:40:66 Modalias: bluetooth:v05D6p000Ad0240
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 00001101-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 0000110b-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 0000110e-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 0000111e-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 00001124-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device 90:D9:94:2B:40:66 ServicesResolved: yes
[CHG] Device 90:D9:94:2B:40:66 Paired: yes
Connection successful

[bluetooth]# trust 90:D9:94:2B:40:66
```
Note: `trust` is necessary for bluetooth autoreconect 

Note: the bluetooth device address (90:D9:94:2B:40:66) is unique to each device, proceeed with the change acordingly to your device address. 

## Once the bluetooth is paired:

1 - Make the bluetooth audio the default alsa output
Overwrite the `/etc/asound.conf`

```
pcm.!default {
        type plug                                         
        slave.pcm {
                type bluealsa
                device "90:D9:94:2B:40:66"
                profile "a2dp"
        }
        hint {
                show on
                description "Bluetooth Audio ALSA Backend"
        }
}

ctl.!default {
        type plug                                         
        slave.pcm {
                type bluealsa
                device "90:D9:94:2B:40:66"
                profile "a2dp"
        }
        hint {
                show on
                description "Bluetooth Audio ALSA Backend"
        }
}
pcm.btheadset {
        type plug
        slave.pcm {
                type bluealsa
                device "90:D9:94:2B:40:66"
                profile "a2dp"
        }
        hint {
                show on
                description "Bluetooth Audio ALSA Backend"
        }
}
```
Note: the bluetooth device address (90:D9:94:2B:40:66) is unique to each device, proceeed with the change acordingly to your device address.

2 - Restart the Rasperry


Node: On my experiments the bluetooth volume didn't work therefore the volume has to be set on the bluetooth audio device itself.
--------------------------------------------------------------------------------------------------------------------------------------------------------
To Do - Automation
