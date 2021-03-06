#
#  xsvftool-xpcu - An (X)SVF player for the Xilinx Platform Cable USB
#
#  Copyright (C) 2011  RIEGL Research ForschungsGmbH
#  Copyright (C) 2011  Clifford Wolf <clifford@clifford.at>
#  
#  Permission to use, copy, modify, and/or distribute this software for any
#  purpose with or without fee is hereby granted, provided that the above
#  copyright notice and this permission notice appear in all copies.
#  
#  THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
#  WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
#  MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
#  ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
#  WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
#  ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
#  OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
#

USE_PREP_FIRMWARE = 1
USE_PREP_HARDWARE = 1

LIBXSVFDIR=..

CC = gcc
CFLAGS = -Wall -Wextra -Werror -Os -ggdb -I$(LIBXSVFDIR) -MD
LDFLAGS = -L$(LIBXSVFDIR)
LDLIBS = -lusb -lreadline -lxsvf

SDCC = sdcc
SDCFLAGS = -mmcs51 --xram-loc 0x2000

all: xsvftool-xpcu

xsvftool-xpcu: filedata.h hardware_cksum_c.inc $(LIBXSVFDIR)/libxsvf.a xsvftool-xpcu.o fx2usb-interface.o
	$(CC) $(LDFLAGS) xsvftool-xpcu.o fx2usb-interface.o $(LDLIBS) -o $@

hardware.svf erasecpld.svf: hardware.sh hardware.ucf hardware.v hardware_cksum_vl.inc
ifeq ($(USE_PREP_HARDWARE),1)
	cp prep_hardware.svf hardware.svf
	cp prep_erasecpld.svf erasecpld.svf
else
	bash hardware.sh
endif

firmware.ihx: firmware.c
ifeq ($(USE_PREP_FIRMWARE),1)
	cp prep_firmware.ihx firmware.ihx
else
	cpp -MD -MF $(basename $<).d -MT $(basename $<).ihx -o /dev/null $<
	$(SDCC) $(SDCFLAGS) $<
endif

firmware.ihx: gpifprog_fixed.c
gpifprog_fixed.c: gpifprog.c
	sed 's/ xdata / /g;' < $< > $@

$(LIBXSVFDIR)/libxsvf.a:
	$(MAKE) -C $(LIBXSVFDIR) libxsvf.a

hardware_cksum_vl.inc hardware_cksum_c.inc: hardware.sh hardware.ucf hardware.v
	echo "'h$$(cat $^ | md5sum | cut -c1-6 | tr a-z A-Z)" > hardware_cksum_vl.inc
	echo "\"$$(cat $^ | md5sum | cut -c1-6 | tr a-z A-Z)\"" > hardware_cksum_c.inc

filedata.h: hardware.svf erasecpld.svf firmware.ihx
	{ echo "unsigned char hardware_svf[] = { " && perl -pe 's/(.)/ord($$1).","/sge' hardware.svf && echo "};" && \
	echo "unsigned char erasecpld_svf[] = { " && perl -pe 's/(.)/ord($$1).","/sge' erasecpld.svf && echo "};" && \
	echo "unsigned char firmware_ihx[] = { " && perl -pe 's/(.)/ord($$1).","/sge' firmware.ihx && echo "};"; } | \
	perl -pe 's/(.{70}.*?,)/$$1\n/g' > filedata.h_new
	mv filedata.h_new filedata.h

prep:
	make clean
	sed -i '/^USE_PREP_/ s/1/0/;' Makefile
	make hardware.svf firmware.ihx
	cp hardware.svf prep_hardware.svf
	cp erasecpld.svf prep_erasecpld.svf
	cp firmware.ihx prep_firmware.ihx
	sed -i '/^USE_PREP_/ s/0/1/;' Makefile
	make clean

clean:
	rm -f firmware.asm firmware.lnk firmware.lst firmware.map
	rm -f firmware.mem firmware.rel firmware.rst firmware.sym
	rm -f hardware.bld hardware_build.xml hardware.chk hardware.cmd
	rm -f hardware.log hardware.lso hardware.mfd hardware.ngc hardware.ngd
	rm -f hardware_ngdbuild.xrpt hardware.ngr hardware.pad hardware_pad.csv
	rm -f hardware.pnx hardware.prj hardware.rpt hardware.svf hardware.syr
	rm -f hardware.vm6 hardware.xml hardware.xst hardware_xst.xrpt
	rm -f hardware.cxt hardware.gyd hardware.jed _impactbatch.log tmperr.err
	rm -f hardware_cksum_vl.inc hardware_cksum_c.inc gpifprog_fixed.c
	rm -rf hardware_html xilinx xlnx_auto_0_xdb _xmsgs
	rm -f filedata.h firmware.ihx erasecpld.cmd erasecpld.svf
	rm -f xsvftool-xpcu core *.o *.d

-include *.d

