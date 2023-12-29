# Getting TFT35-SPI with IO2CAN working on a raspberry pi

We use the TFT35-SPI and IO2CAN from bigtreetech:
 - IO2CAN: https://github.com/bigtreetech/IO2CAN/tree/master
 - TFT35-SPI: https://github.com/bigtreetech/TFT35-SPI/tree/master

Please plug the IO2CAN into the gpio headers of the raspberry pi, connect the ribbon cable to the display and IO2CAN and **set the switch on the IO2CAN to "CM4"**.

[!NOTE]
- You should also be able to use a 10-pin JST connector and wires to the gpio pins instead of the IO2CAN. Just make sure to connect to the gpio [pins that are also used by the IO2CAN for CM4](https://github.com/bigtreetech/IO2CAN/blob/master/Hardware/BIGTREETECH%20IO2CAN%20V1.0_IO.pdf).
- The display uses the spi0-1 interface. With the IO2CAN spi0-0 is used for CAN. However, the display seems to interfere with the CAN interface and I got corrupted packets and timeouts in klipper when homing. Lowering the bandwidth for the dispay did not help, so using CAN and the display does not seem to work reliably. Moving the display to other gpio pins is also not easy as the spi mode it uses is not supported on the other spi interfaces of rpi 2 zero or rpi 3.
- You will have to re-build the kernel module for the tfttouch part every time your kernel changes.


## Setting up the display:

- add to the end of /boot/config.txt:
```
dtparam=spi=on
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
```
_Please note that this uses spi0-1 and dc_pin=17 unlike written on the TFT35-SPI git._

- to get console output, append to line in /boot/cmdline.txt after "rootwait":
```
 fbcon=map:10 fbcon=font:8x8 logo.nologo
```

- reboot and check if you get output. You can use ```dmesg``` to see if the module was loaded correctly.

## Setting up touch:

- install the build tools:
```
sudo apt install git bc bison flex libssl-dev
```

- install the kernel headers based on your system
for 64 bit:
```
sudo apt install linux-headers-rpi-v8
``` 
or for 32 bit:
```
sudo apt install linux-headers-rpi-{v6,v7,v7l}
```

- install this repo:
```
git clone https://github.com/stadie/btt_tft35spi_rpi.git btt_tft35
cd btt_tft35
```

- build the module using two source files for the tsc2007 module that were modified for the [CB1 kernel](https://github.com/bigtreetech/CB1-Kernel) and build an overlay that will load the driver:
```
cd tsc2007
make
```

- copy module to kernel modules and the new overlay to boot:
```
make install
```

- add to the end of /boot/config.txt:
```
dtoverlay=i2c-gpio,i2c_gpio_sda=27,i2c_gpio_scl=22,i2c_gpio_delay_us=3
dtoverlay=tft_touch
```
This configures an i2c connection with the pins used by IO2CAN and loads the overlay for the touch driver.

- after a reboot
```
sudo reboot now
```
you should have a working touch display.

[!IMPORTANT]
If the kernel version changes you will have to build and install the module again:
```
cd btt_tft35/tsc2007
make clean
make
make install
```

## Debugging

- check if the module or overlay could be loaded:
```
dmesg
sudo vcdbg log msg
```
