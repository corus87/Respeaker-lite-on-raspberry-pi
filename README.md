# Respeaker lite on Raspberry PI via I2S

Using the Respeaker lite on the Raspberry PI via I2S.


## Firmware
Make sure you have installed the I2S firmware, check out the [Respeaker repository](https://github.com/respeaker/ReSpeaker_Lite).


## Software
To be able to use the Respeaker lite over I2S we need a device tree overlay.


[This overlay does the job (Howto copied from the repo and updated some steps):](https://github.com/AkiyukiOkayasu/RaspberryPi_I2S_Slave?tab=readme-ov-file)

Compile on Raspberry Pi  
```bash
dtc -@ -H epapr -O dtb -o genericstereoaudiocodec.dtbo -Wno-unit_address_vs_reg genericstereoaudiocodec.dts
```

Copy i2smaster.dtbo to /boot/overlays  
```bash
sudo cp genericstereoaudiocodec.dtbo /boot/overlays
```

Edit /boot/firmware/config.txt  
Enable I2S / I2C and add i2smaster device tree overlay  
```/boot/firmware/config.txt   # Uncomment some or all of these to enable the optional hardware interface
dtparam=i2c_arm=on
dtparam=i2s=on
#dtparam=spi=on
dtoverlay=genericstereoaudiocodec
```

If you don't need HDMI audio output and Raspberry Pi's headphone output, comment out "dtparam=audio=on" by hash.  
like this.  
```/boot/firmware/config.txt
#dtparam=audio=on
```

## Wiring

| Raspberry | Respeaker Lite    |
|-----------|-------------------|
| 5V        | 5V                |
| GND       | GND               |
| GPIO18    | I2S_BCLK          |
| GPIO19    | I2S_LRCLK         |
| GPIO20    | I2S_DIN_XIAO(din) |
| GPIO21    | I2S_DIN_SEC(dout) |
| GPIO2     | SDA               |
| GPIO3     | SCL               |


### Note
Be careful soldering the pins to the Respeaker, it's not clear how but it's possible to break/brick the device. 
Check out [this thread for more information's.](https://forum.seeedstudio.com/t/3-broken-and-unusable-respeaker-lite/291165).
In my experience with 5 devices, be careful when soldering the pins, don't put to much heat into the device and keep the soldering point as it is after you are done (I broke/bricked one by just cleaning the solder points couple days later).


![Respeaker-Raspberry](images/Respeaker_Raspi.png?raw=true)



## Test
After a reboot you should be able to record with:

`arecord -D plughw:CARD=GenericStereoAu,DEV=1 -f S16_LE -r 48000 -c 1 -t wav -d 5 test.wav` 

And playback:

`aplay -D plughw:CARD=GenericStereoAu,DEV=0 test.wav`

## Alsa config

I've created an asoundrc config, I don't have much experience with Alsa but it works and does the following:

- Created two soft volumes mixer (playback and capture).
- Using dsnooper and dmixer to use the device by more then application. 
- Sets the the microphone and playback device as default.
- Sets the sample rate to 48khz (change this if you use the 16khz firmware)

Now you should be able to record and playback using the default devices:

`arecord -f S16_LE -t wav -d 5 test.wav` 

and 

`aplay test.wav`




## Note
If you have more information's about the Respeaker lite or Alsa/sound improvements, feel free to create PR or open an issue.

