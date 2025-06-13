# Adding a second SD card on the Raspberry Pi

Following the excellent guide on [ralimtech](https://ralimtek.com/posts/2016/2016-12-10-raspberry_pi_secondary_sd_card/) I tried to add another SD Card reader to my Raspberry Pi 3B, via SPI. It failed since the syntax has changed in the device tree. The correct one was found in a thread on [the Raspberry Pi forum](https://forums.raspberrypi.com/viewtopic.php?t=335338).

This is my `mmc_spi.dts` file for anyone interested:  
```dts
/dts-v1/;
/plugin/;

/ {
   compatible = "brcm,bcm2835", "brcm,bcm2708"; 

   fragment@0 {
      status = "okay";
      target = <&spi0>;
      __overlay__ {
         status = "okay";
         #address-cells = <1>;
         #size-cells = <0>;
         spidev@0 {
            reg = <0>;
            compatible = "mmc-spi-slot";
	    voltage-ranges = <3000 3500>;
            spi-max-frequency = <8000000>;
         };
      };
   };
};
```

Compile with `dtc -@ -I dts -O dtb -o mmc_spi.dtbo mmc_spi.dts`, install with `sudo cp mmc_spi.dtbo /boot/overlays/` and be sure to add `dtoverlay=mmc_spi` to `/boot/firmware/cmdline.txt`

### Information that may be of interest

Raspberry Pi model: `Raspberry Pi 3 Model B Rev 1.2`  
Kernel version: `6.12.25+rpt-rpi-v8`
