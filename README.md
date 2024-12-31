# Getting TFT35-SPI working on a raspberry pi


I have used the TFT35-SPI with IO2CAN from bigtreetech:
 - IO2CAN: https://github.com/bigtreetech/IO2CAN/tree/master
 - TFT35-SPI: https://github.com/bigtreetech/TFT35-SPI/tree/master
Please plug the IO2CAN into the gpio headers of the raspberry pi, connect the ribbon cable to the display and IO2CAN and **set the switch on the IO2CAN to "CM4"**.

**However, this works also with wires directly connecting the JST pins on the display with the gpio pins or with a FCC cable to the "SPI screen" connector on a Manta board with CM4.**

Pin out:
| TFT35-SPI | IOCAN gpio pin | Manta FCC gpio pin |
| -- | --- | --- |
| +5 V | any 5 V | comes from board |
| GND | any GND | comes from board |
| MISO_LCD (SPI MISO) | 9 | 9 |
| MOSI_LCD (SPI MOSI) | 10 | 10 |
| SCK_LCD (SPI SCLK) | 11 | 11 |
| NSS_LCD (SPI CS) | 7 | 4 | 
| RS_LCD | 17 | 17 |
| SDA_NS2009 (I2C SDA) | 27  | 27 |
| SCL_NS2009 (I2C SCL) | 22 | 22 |

The gpio pins are set up to use the *spi0* interface. In case of the CM4 with the Manta board, a different chip select (CS) pin is used while with IO2CAN the CS pin is mapped to the default *cs1 pin* of *spi0*. 


> [!NOTE]
> - With IO2CAN, the display uses the spi0-1 interface. However, the display seems to interfere with the CAN interface and I got corrupted packets and timeouts in klipper when homing. Lowering the bandwidth for the dispay did not help, so using CAN and the display does not seem to work reliably. Moving the display to other gpio pins is also not easy as the spi mode it uses is not supported on the other spi interfaces of rpi zero 2 or rpi 3.
> - You will have to re-build the kernel module for the tfttouch part every time your kernel changes.


## Setting up the display:

- add to the end of /boot/config.txt or on bookworm and higher /boot/firmware/config.txt:

with IO2CAN:
```
dtparam=spi=on
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
```
to setup the SPI interface and load the display driver.

_Please note that this uses spi0-1 and dc_pin=17 unlike written on the TFT35-SPI git._

with Manta/CM4:
 ```
dtoverlay=spi0-2cs,cs0_pin=23,cs1_pin=4,
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
``` 
On the CB1 spi0-0 is reserved for CAN with the Manta, so it might be useful to set it up as well.

- to get console output, append to line in /boot/cmdline.txt or on bookworm and higher /boot/firmware/cmdline.txt after "rootwait":
```
 fbcon=map:10 fbcon=font:8x8 logo.nologo
```

- reboot and check if you get output. You can use ```dmesg``` to see if the module was loaded correctly.

## Setting up touch:

- update system and install the build tools:
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt install git bc bison flex libssl-dev
```

- install the kernel headers based on your system
with rpi OS bookworm:
for 64 bit:
```
sudo apt install linux-headers-rpi-v8
``` 
or for 32 bit:
```
sudo apt install linux-headers-rpi-{v6,v7,v7l}
```

with rpi OS bullseye:
```
sudo apt install --reinstall raspberrypi-kernel-headers
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

- add to the end of  /boot/config.txt or on bookworm and higher /boot/firmware/config.txt:
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

> [!IMPORTANT]
> If the kernel version changes you will have to build and install the module again:
> ```
> cd btt_tft35/tsc2007
> make clean
> make
> make install
> ```

## Debugging

- check if the module or overlay could be loaded:
```
dmesg
sudo vcdbg log msg
```

## Acknowledgements
I like to thank Stenberggg on github for trying this setup on a raspberry pi 5. Fragmon@Crydteam on discord tried this on a CM4 and a Manta board and connect the display to the FCC port. He has also helped a lot in figuring out the correct pins. 
