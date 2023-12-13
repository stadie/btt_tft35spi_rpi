# Getting TFT35-SPI with IO2CAN working on a raspberry pi

We use the TFT35-SPI and IO2CAN from bigtreetech:
 - IO2CAN: https://github.com/bigtreetech/IO2CAN/tree/master
 - TFT35-SPI: https://github.com/bigtreetech/TFT35-SPI/tree/master

## Setting up the display:

add to the end of /boot/config.txt:
```
dtparam=spi=on
dtoverlay=fbtft,spi0-1,ili9486,width=320,height=480,dc_pin=17
dtparam=cpha=1,cpol=1,speed=24000000,fps=60
dtparam=bgr=1,rotate=90,txbuflen=307200
```

to get console output, append to line in /boot/command.txt
```

```


reboot and check if you get output. You can use ```dmesg``` to see if the module was loaded correctly.

## Setting up touch:

install kernel headers:
```
sudo apt install git bc bison flex libssl-dev
sudo apt install 
```

Enable build of module:
```
cd tsc2007
make
```

Copy module to kernel modules:
```

```

Build overlay:
```
dtc  -@ -I dts -O dtb -o tft_touch.dtbo tft_touch.dts
sudo cp tft_touch.dtbo /boot/overlays/.
```

Edit boot config:



Test after rerboot:
```
dmesg | grep tft
```
