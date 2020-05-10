## Goals

Reverse-engeneering and documenting Microsoft Catapult FPGA board (v2: Pikes Peak) so one can use it as a readily available and cheap development kit for Intel Stratix V FPGA.

## Overview of project Catapult

[Project Catapult (microsoft.com)](https://www.microsoft.com/en-us/research/project/project-catapult/)  
[Large-Scale Reconfigurable Computing in a Microsoft Datacenter (PDF)](https://www.microsoft.com/en-us/research/uploads/prod/2014/06/HC26.12.520-Recon-Fabric-Pulnam-Microsoft-Catapult.pdf)  
[The Catapult Project - An FPGA view of the Data Center (PDF)](http://www.prime-project.org/wp-content/uploads/sites/206/2018/02/Talk-7-Dan-Fay-The-Catapult-Project-%E2%80%93-An-FPGA-view-of-the-Data-Center.pdf)  

There's several versions of Catapult board. The one we're working here is the OpenCloud Mezzanine Card (aka v2 "Pikes Peak", Microsoft p/n `X900563-001`):
![](/docs/pics/board_top.jpeg)
![](/docs/pics/board_bottom.jpeg)

Known p/n for this board:
- `X900563-001` (Microsoft OEM p/n)
- `834147-001 PCA, WCS FPGA PCIe Mezz Card`, also `838874-001` (HP p/n)
- `Dell CRD CTL FPGA PKPK WCS K5K73 AIRFLOW 0K5K73` (Dell p/n)


## Overview of Catapult FPGA board v2 "Pikes Peak"

### Main features

- Intel Stratix V GS (DSP-optimized), p/n 5SGSKF40I3LNAC (*see note 1)
- 2x QSFP 40G cages
- 9x Skhynix H5TC4G83BFR 8-bit 4Gb DDR3L, 72-bit bus (64 bit + ECC)
- 2x PCIe Gen3 8x, routed to the Samtec connector

Note 1: See "FPGA part mystery".

### FPGA part number mystery

Judging by Microsoft papers, it’s supposed to be 5SGSD5, but the actual part (5SGSKF40I3LNAC) p/n not following Intel [part numbering scheme for Stratix V](https://www.intel.com/content/www/us/en/programmable/documentation/sam1403476018909.html#sam1403476002556). There's hardly any info for this p/n online, here's the only Intel document with this p/n: [PDN2007](https://www.intel.com/content/dam/www/programmable/us/en/pdfs/literature/pcn/pdn2007.pdf) (thanks to [@FPGA_VHDL](https://twitter.com/FPGA_VHDL) for pointning this out).  
There's also no support for such part in Quartus Prime 18.0. JTAG IDCODE for the actual part is `0x029070dd`, which corresponds to 5SGSMD5H1/2/3 and 5SGSMD5K1/2/3. Actual part working well with bitstreams, generated for 5SGSMD5H* and 5SGSMD5K*, so it's unclear at the moment what's the difference between them.

### Form factor

Catapult v2 "Pikes Peak" designed as Open CloudServer Tray Mezzanine Card in compliance with [Open CloudServer OCS Chassis Specification Version 2.0](https://www.opencompute.org/documents/microsoft-ocs-v2-chassis).

### Markings
Here's the markings for my board (HP branded) for reference:

Heatsink:
- X900563-001 (sticker, I believe it's Microsoft OEM part number for this board)
- HP P/N:834147-001; Replace with Spare: PCA, WCS FPGA PCIe Mezz Card; [838874-001] (sticker)
- CCIFBT6M2390105BC3A039A (sticker, s/n?) 

PCB:
- P/N DAT6MTHMECO Rev: C (top, silkscreen)
- NLJ54708033 (top, sticker)
- C3A; 3KT6MMA0090 (top, sticker)
- Microsoft (bottom, silkscreen)
- NLJ54708033 (bottom, sticker)

### Mechanical

Unfortunately, there's no mechanical specifications for Tray Mezzanine Card in Open CloudServer standard, so, here's my initial measurements.  
- Dimensions: approx. 154x87x29mm  
- Weight: approx. 210g  

### BOM

See [bom.xlsx](/docs/bom.xlsx).

### FPGA pins

See [fpga_io.xlsx](/docs/fpga_io.xlsx).

Twitter user [@occamlab](https://twitter.com/occamlab) shared with us [some information about FPGA pins designations](http://virtlab.occamlab.com/home/zapisnik/microsoft-catapult-v2) (especially PCIe and QSFP Tx), but I didn't have a chance to check them.

### USB

USB connector `J3` pinout (thanks to [Jan Marjanovič](https://twitter.com/janmarjanovic) for sharing [his RE efforts](https://j-marjanovic.io/stratix-v-accelerator-card-from-ebay.html)):
![](/docs/pics/usb.png)

There's some vague mentions online that one can use FT232H as a ByteBlaster by changing its VID/PID by modifying driver files or installing external ROM chip to the board (U12).

Right now you can use it with OpenOCD to check IDCODE:

![](/docs/pics/openocd.png)

OpenOCD config file for FT232H:
```
# TCK:  D0
# TDI:  D1
# TDO:  D2
# TMS:  D3
# TRST: D4
# SRST: D5

interface ftdi
ftdi_vid_pid 0x0403 0x6014
ftdi_layout_init 0x0078 0x017b
adapter_khz 1000
ftdi_layout_signal nTRST -ndata 0x0010 -noe 0x0040
ftdi_layout_signal nSRST -ndata 0x0020 -noe 0x0040
# change this to 'transport select swd' if required
transport select jtag
```

### JTAG

I've been able to connect an actual USB ByteBlaster after soldering to following pins:
![](/docs/pics/jtag_points.png)

## Additional resources

- Jan Marjanovič's blog page covering Catapult board RE: https://j-marjanovic.io/stratix-v-accelerator-card-from-ebay.html  
- [@occamlab](https://twitter.com/occamlab)'s blog page: http://virtlab.occamlab.com/home/zapisnik/microsoft-catapult-v2
