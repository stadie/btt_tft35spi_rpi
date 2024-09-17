#
# A little Makefile to build the patched kernel module for the touch driver of BTT TFT35-SPI
# author: Hartmut Stadie  Dec 2023
#
OLD_KVER := $(shell echo "`uname -r | cut -d'.' -f 1,2` - 6.3 < 0" | /usr/bin/bc -l)
# Build options
obj-m += tsc2007.o

ifeq ($(OLD_KVER), 0)
tsc2007-objs :=  tsc2007_core.o tsc2007_iio.o
else
tsc2007-objs :=  tsc2007_core.o
endif

PWD := $(CURDIR)
BTT_CODE := https://raw.githubusercontent.com/bigtreetech/CB1-Kernel/kernel-5.16/kernel

all: get_code module 

module:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
	rm -f tft_touch.btbo

update: clean_code get_code

clean_code: clean
	rm -f *.c *.h

get_code:  tsc2007_core.c  tsc2007.h tsc2007_iio.c

tsc2007_core.c: tsc2007_core_patch.diff
	wget $(BTT_CODE)/drivers/input/touchscreen/tsc2007_core.c
	@patch < tsc2007_core_patch.diff

tsc2007.h:
	wget $(BTT_CODE)/drivers/input/touchscreen/tsc2007.h

tsc2007_iio.c:
	wget $(BTT_CODE)/drivers/input/touchscreen/tsc2007_iio.c


install: tsc2007.ko 
	sudo cp tsc2007.ko  /lib/modules/$(shell uname -r)/kernel/drivers/input/touchscreen/tsc2007.ko
	sudo depmod

