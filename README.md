# Getting TFT35-SPI with IO2CAN working on a raspberry pi

We use the TFT35-SPI and IO2CAN from bigtreetech:
 - IO2CAN: https://github.com/bigtreetech/IO2CAN/tree/master
 - TFT35-SPI: https://github.com/bigtreetech/TFT35-SPI/tree/master

## Setting up the display:

in /boot/config.txt:

```
dtparam=spi=on
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
```

## Setting up touch:

install rpi-source:
```
sudo apt install git bc bison flex libssl-dev
sudo wget https://raw.githubusercontent.com/RPi-Distro/rpi-source/master/rpi-source -O /usr/local/bin/rpi-source && sudo chmod +x /usr/local/bin/rpi-source && /usr/local/bin/rpi-source -q --tag-update
```

Download the sources of the current kernel:
```
rpi-source
```

Enable build of module:
```
cd linux
make menuconfig
```
Go to "Device Drivers" -> "Input Device Support" -> "Touchscreens"; then press "m" for "TSC2007 based touchscreens" and exit.

Build the modules:
```
make prepare
make modules -j 4
```
