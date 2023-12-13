# Getting TFT35-SPI with IO2CAN working on a raspberry pi

We use the TFT35-SPI and IO2CAN from bigtreetech:
 - IO2CAN: https://github.com/bigtreetech/IO2CAN/tree/master
 - TFT35-SPI: https://github.com/bigtreetech/TFT35-SPI/tree/master

Please plug the IO2CAN into the gpio headers of the raspberry pi, connect the ribbon cable to the display and IO2CAN and set the switch on the IO2CAN to "CM4".

Side note:
You should also be able to use a JST cable instead of the IO2CAN. Just make sure to connect to the gpio pins that are also used by the IO2CAN.


## Setting up the display:

add to the end of /boot/config.txt:
```
dtparam=spi=on
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
```

to get console output, append to line in /boot/cmdline.txt after "rootwait":
```
 fbcon=map:10 fbcon=font:8x8 logo.nologo
```

reboot and check if you get output. You can use ```dmesg``` to see if the module was loaded correctly.

## Setting up touch:

install kernel headers:
```
sudo apt install git bc bison flex libssl-dev
sudo apt install 
```

install this repo:
```
git clone https://github.com/stadie/btt_tft35.git
cd btt_tft35
```

build the module using the sources from the CB1 kernel and build an overlay:
```
cd tsc2007
make
```

copy module to kernel modules:
```
make install
```

add to the end of /boot/config.txt:
```
dtoverlay=i2c-gpio,i2c_gpio_sda=27,i2c_gpio_scl=22,i2c_gpio_delay_us=3
dtoverlay=tft_touch
```


test after rerboot:
```
dmesg | grep tft
```
